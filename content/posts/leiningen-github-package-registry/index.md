---
title: "Deploying Leiningen projects to GitHub Package Registry"
subtitle: "Set up Leiningen to deploy Clojure libraries to GitHub Package Registry as Apache Maven packages"
date: 2019-09-04T11:42:19+05:30
tags: ["clojure", "github"]
---

I recently got access to [GitHub Package Registry](https://github.com/features/package-registry),
which is currently in beta.

According to its
[documentation](https://help.github.com/en/articles/about-github-package-registry#supported-clients-and-formats),
GitHub Package Registry supports npm, RubyGems, Maven, Docker and NuGet packages at the moment.

Out of curiousity, I decided to try to publish a Clojure library to GitHub Package Registry, since
Clojure code is often distributed in the Maven repository format as well.

## Creating a new Clojure library

I created a new project using [Leiningen](https://leiningen.org), a common build tool for Clojure
projects:

```
$ lein new clj-left-pad
Generating a project called clj-left-pad based on the 'default' template.
The default template is intended for library projects, not applications.
To see other templates (app, plugin, etc), try `lein help new`.
```

By default, `lein new` creates a library project. `lein new app` can be used to create an
application project using the `app` template---but this post is about publishing a library. You can
find out more about creating a project with Leiningen in the
[tutorial](https://github.com/technomancy/leiningen/blob/master/doc/TUTORIAL.md#creating-a-project).

## Setting up GitHub Package Registry as a Maven repository

It seems like packages in GitHub Package Registry are repository-specific---a repository needs to
exist before you can publish a package for it. So I created a repository for my new library:

{{< linkpreview title="yi-jiayu/clj-left-pad" description="Trying out GitHub Package Registry"
url="https://github.com/yi-jiayu/clj-left-pad" >}}

### Telling Leiningen about GitHub Package Registry

GitHub provides [some
documentation](https://help.github.com/en/articles/configuring-apache-maven-for-use-with-github-package-registry)
about deploying Maven packages to GitHub Package Registry, however it only describes working with
Maven's XML configuration (`~/.m2/settings.xml` and `pom.xml`).

Leiningen, on the other hand, uses its own configuration file with Clojure syntax
(`project.clj`). However, it still uses Maven under the hood, so we can roughly map `project.clj`
properties to `pom.xml` ones by referring to the sample `project.clj` with all available keys:

{{< linkpreview
title="leiningen/sample.project.clj at f39bdb6ac4b998577d0c0c4b50cfdbc14cb06f55"
description="Automate Clojure projects without setting your hair on fire. - technomancy/leiningen"
url="https://github.com/technomancy/leiningen/blob/f39bdb6ac4b998577d0c0c4b50cfdbc14cb06f55/sample.project.clj"
>}}

In short, we just need to add a single `:repositories` key to `project.clj`:

```diff
 (defproject clj-left-pad "0.1.0-SNAPSHOT"
             :description "String left pad"
             :url "https://github.com/yi-jiayu/clj-left-pad"
             :license {:name "EPL-2.0 OR GPL-2.0-or-later WITH Classpath-exception-2.0"
                       :url "https://www.eclipse.org/legal/epl-2.0/"}
+            :repositories [["github" {:url "https://maven.pkg.github.com/yi-jiayu" :creds :gpg}]]
             :dependencies [[org.clojure/clojure "1.10.0"]]
             :repl-options {:init-ns clj-left-pad.core})
```

### Specifying credentials

The `:creds :gpg` entry in the new repository we added previous tells Leiningen that it can find
GPG-encrypted credentials for the repository. For GitHub Package Registry, use your GitHub username
as your username and a GitHub [personal access token
(PAM)](https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line)
with the `read:packages` and `write:packages` scopes as your password. I also used
`https://maven.pkg.github.com/yi-jiayu*` as the URL (replace my username with your own). Then follow
the Leiningen guide for providing credentials with GPG:
https://github.com/technomancy/leiningen/blob/master/doc/DEPLOY.md#gpg.

## Deploying!

Once your repository and credentials are set up, you can deploy to GitHub Package Registry with
`lein deploy github`:

```console
$ lein deploy github
Created /Users/yijiayu/git/github.com/yi-jiayu/clj-left-pad/target/clj-left-pad-0.3.1.jar
Wrote /Users/yijiayu/git/github.com/yi-jiayu/clj-left-pad/pom.xml
Need to sign 2 files with GPG
[1/2] Signing /Users/yijiayu/git/github.com/yi-jiayu/clj-left-pad/target/clj-left-pad-0.3.1.jar with GPG
[2/2] Signing /Users/yijiayu/git/github.com/yi-jiayu/clj-left-pad/pom.xml with GPG
Sending clj-left-pad/clj-left-pad/0.3.1/clj-left-pad-0.3.1.jar (9k)
    to https://maven.pkg.github.com/yi-jiayu/
Sending clj-left-pad/clj-left-pad/0.3.1/clj-left-pad-0.3.1.pom (3k)
    to https://maven.pkg.github.com/yi-jiayu/
Sending clj-left-pad/clj-left-pad/0.3.1/clj-left-pad-0.3.1.jar.asc (1k)
    to https://maven.pkg.github.com/yi-jiayu/
Could not transfer artifact clj-left-pad:clj-left-pad:jar.asc:0.3.1 from/to github (https://maven.pkg.github.com/yi-jiayu): Access denied to: https://maven.pkg.github.com/yi-jiayu/clj-left-pad/clj-left-pad/0.3.1/clj-left-pad-0.3.1.jar.asc, ReasonPhrase: Forbidden.
Sending clj-left-pad/clj-left-pad/0.3.1/clj-left-pad-0.3.1.pom.asc (1k)
    to https://maven.pkg.github.com/yi-jiayu/
Could not transfer artifact clj-left-pad:clj-left-pad:pom.asc:0.3.1 from/to github (https://maven.pkg.github.com/yi-jiayu): Access denied to: https://maven.pkg.github.com/yi-jiayu/clj-left-pad/clj-left-pad/0.3.1/clj-left-pad-0.3.1.pom.asc, ReasonPhrase: Forbidden.
Failed to deploy artifacts: Could not transfer artifact clj-left-pad:clj-left-pad:jar.asc:0.3.1 from/to github (https://maven.pkg.github.com/yi-jiayu): Access denied to: https://maven.pkg.github.com/yi-jiayu/clj-left-pad/clj-left-pad/0.3.1/clj-left-pad-0.3.1.jar.asc, ReasonPhrase: Forbidden.
```

For some reason, there are errors on the console. However, visiting the packages page on the GitHub
repository itself shows that the package was successfully deployed:

{{< figure src="deployed-package.png" alt="GitHub repository packages section for the newly-deployed package." >}}

¯\\_(ツ)_/¯
