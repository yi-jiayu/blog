---
title: "Running Airflow Webserver on Cloud Run"
date: 2021-07-10T12:23:48+08:00
---

[Apache Airflow](https://airflow.apache.org/) is, according to its website, a
platform created by the community to programmatically author, schedule and
monitor workflows. Airflow is deployed in several several components, two of
which are the Scheduler and Webserver.

[DAG
Serialization](https://airflow.apache.org/docs/apache-airflow/stable/dag-serialization.html)
allows the Webserver to read DAGs entirely from the database instead of having
to parse DAG files. This makes it possible to deploy onto a serverless platform
like Cloud Run, where the Webserver does not need to be running when no one is
accessing it. However, there are some hoops which need to be jumped through to
actually do that.

## Build a custom Airflow container image

Airflow generates a
[`webserver_config.py`](https://github.com/apache/airflow/blob/main/airflow/config_templates/default_webserver_config.py)
config file by default, which is used to configure features like authentication
or mail sending. However, on a serverless platform like Cloud Run which uses
ephemeral containers, the base Airflow container image will constantly
regenerate this file.

In order to customise `webserver_config.py`, it should be built directly into
the container. Below is a sample Dockerfile which installs
[Authlib](https://authlib.org/), which is needed for the `AUTH_OAUTH`
authentication method in Airflow, then copies in a preconfigured
`airflow_webserver.py`. This container can then be passed to Cloud Run and will
use the predefined config.

```Dockerfile
FROM apache/airflow:2.1.1-python3.8

RUN pip install --no-cache-dir Authlib

COPY webserver_config.py .
```

## Disable entrypoint database checks

Cloud Run connects to Cloud SQL via Cloud SQL Proxy, which mounts a Unix socket
into the container for the application to connect to. SQLAlchemy, which is used
by Airflow, supports this. However, the official Airflow container image also
tries to verify the database connection in its [entrypoint
script](https://github.com/apache/airflow/blob/2.1.1/scripts/in_container/prod/entrypoint_prod.sh)
but assumes that the database connection is always over TCP, so this check will
always fail. We can bypass this check by setting the
`CONNECTION_CHECK_MAX_COUNT` environment variable to 0.

## Configure Airflow

Airflow is configured with an `airflow.cfg` config file by default, however
every config option can also be set using environment variables. The full
configuration reference and corresponding environment variables can be found on
the [Configuration
Reference](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html)
page.

The `webserver_config.py` file can also be configured to read environment
variables, for example to set OAuth credentials.
