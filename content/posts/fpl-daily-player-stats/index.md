---
title: "FPL daily player statistics dataset"
subtitle: "Player value, ownership, form, ICT index and more over time"
date: 2019-03-25 00:31:56+08:00
---

On the [Fantasy Premier League](https://fantasy.premierleague.com/) website, there's an endpoint
called [`bootstrap-static`](https://fantasy.premierleague.com/drf/bootstrap-static) which returns a
JSON document containing data on various parts of the game.

The bulk of the data is a list of player objects under the `elements` key. Each player object
contains detailed information about a player, for example:

<details open>
  <summary>A player object</summary>
```json
{
  "id": 257,
  "photo": "92217.jpg",
  "web_name": "Firmino",
  "team_code": 14,
  "status": "d",
  "code": 92217,
  "first_name": "Roberto",
  "second_name": "Firmino",
  "squad_number": 9,
  "news": "Ankle injury - 50% chance of playing",
  "now_cost": 92,
  "news_added": "2019-02-24T17:31:21Z",
  "chance_of_playing_this_round": 50,
  "chance_of_playing_next_round": 50,
  "value_form": "0.2",
  "value_season": "13.3",
  "cost_change_start": -3,
  "cost_change_event": 0,
  "cost_change_start_fall": 3,
  "cost_change_event_fall": 0,
  "in_dreamteam": false,
  "dreamteam_count": 2,
  "selected_by_percent": "15.5",
  "form": "2.0",
  "transfers_out": 2518790,
  "transfers_in": 1463787,
  "transfers_out_event": 52014,
  "transfers_in_event": 758,
  "loans_in": 0,
  "loans_out": 0,
  "loaned_in": 0,
  "loaned_out": 0,
  "total_points": 122,
  "event_points": 0,
  "points_per_game": "4.5",
  "ep_this": "1.5",
  "ep_next": "1.5",
  "special": false,
  "minutes": 2058,
  "goals_scored": 9,
  "assists": 6,
  "clean_sheets": 16,
  "goals_conceded": 11,
  "own_goals": 0,
  "penalties_saved": 0,
  "penalties_missed": 0,
  "yellow_cards": 0,
  "red_cards": 0,
  "saves": 0,
  "bonus": 17,
  "bps": 499,
  "influence": "580.2",
  "creativity": "503.5",
  "threat": "952.0",
  "ict_index": "202.9",
  "ea_index": 0,
  "element_type": 4,
  "team": 12
}
```
</details>

Despite the "static" in its name, the `bootstrap-static` endpoint data is actually updated fairly
frequently---"static" seems to refer to not being specific to the logged-in user. By recording the
`bootstrap-static` endpoint over time, we can create a time series of FPL player data!

In fact, I've been saving the information from this endpoint twice daily since September 2018, using
a scheduled AWS Lambda function and saving the data in gzip-compressed format to a S3 bucket. The
bucket is publicly-accessible and available here:

{{< linkpreview title="FPL 2018-19 data" description="Twice-a-day snapshots of the FPL bootstrap-static endpoint since 12 September 2018" url="http://fpl-2018-19-data.s3.amazonaws.com/" >}}

## Getting the data

Using the [AWS CLI](https://aws.amazon.com/cli/), you can synchronise the contents of the bucket to
your local machine:

```console
$ aws s3 sync s3://fpl-2018-19-data fpl-2018-19-data
download: s3://fpl-2018-19-data/bootstrap-static-2018-09-12T0851Z.json.gz to fpl-2018-19-data/bootstrap-static-2018-09-12T0851Z.json.gz
download: s3://fpl-2018-19-data/bootstrap-static-2018-09-15T0856Z.json.gz to fpl-2018-19-data/bootstrap-static-2018-09-15T0856Z.json.gz
download: s3://fpl-2018-19-data/bootstrap-static-2018-09-13T0220Z.json.gz to fpl-2018-19-data/bootstrap-static-2018-09-13T0220Z.json.gz
download: s3://fpl-2018-19-data/bootstrap-static-2018-09-12T1452Z.json.gz to fpl-2018-19-data/bootstrap-static-2018-09-12T1452Z.json.gz
download: s3://fpl-2018-19-data/bootstrap-static-2018-09-12T0852Z.json.gz to fpl-2018-19-data/bootstrap-static-2018-09-12T0852Z.json.gz
download: s3://fpl-2018-19-data/bootstrap-static-2018-09-13T0856Z.json.gz to fpl-2018-19-data/bootstrap-static-2018-09-13T0856Z.json.gz
...
```

This will download the contents of the bucket to the `fpl-2018-19-data` directory.

Here is a short shell script to uncompress the data to a sibling `data` directory:

```sh
#!/bin/sh -

destdir=data
for f in fpl-2018-19-data/*; do
	dest="$destdir/$(basename -s .gz $f)"
	if [ ! -f $dest ]; then
		echo $dest
		gunzip -c $f > $dest
	fi
done
```

Each file contains the timestamp it was downloaded at in the filename:

```console
$ ls data
bootstrap-static-2018-09-12T0851Z.json
bootstrap-static-2018-09-12T0852Z.json
bootstrap-static-2018-09-12T1452Z.json
bootstrap-static-2018-09-13T0220Z.json
bootstrap-static-2018-09-13T0856Z.json
bootstrap-static-2018-09-13T2056Z.json
bootstrap-static-2018-09-14T0856Z.json
bootstrap-static-2018-09-14T2056Z.json
bootstrap-static-2018-09-15T0856Z.json
bootstrap-static-2018-09-15T2056Z.json
...
```

I was able to extract the date from the filename using the following `strptime` pattern in Python:

```python
>>> from datetime import datetime
>>> def extract_timestamp(filename):
...     return datetime.strptime(filename, 'bootstrap-static-%Y-%m-%dT%H%MZ.json')
... 
>>> extract_timestamp('bootstrap-static-2018-09-15T2056Z.json')
datetime.datetime(2018, 9, 15, 20, 56)
```

## Example visualisations

As an experiment to see what I could do with the data, I loaded it into Elasticsearch and made some
graphs with Kibana (I'll write about how I did this in a future post):

### Average team value over time

{{< figure src="average-team-value-over-time.png" caption="Average team value over time" >}}

Manchester City players are far and away the most expensive---although their value seems to have
taken a hit recently---followed by Liverpool as a distant second. The unlucky club with ID 3 just
before the tight pack at the bottom that seems to be dropping consistently in value is Arsenal.

### Player value over time (top 10)

{{< figure src="player-value-over-time.png" caption="Player value over time (top 10)" >}}

The undisputed top two most valuable players are Mohamed Salah and Harry Kane, with Sergio Agüero in
tenuous third place.

### Player ownership over time (top 5)

{{< figure src="player-ownership-over-time.png" caption="Player ownership over time (top 5)" >}}

Fantasy managers are fickle, as even the most valuable players see their popularity fluctuate
wildly. The subject of the precipitous drop in the middle of the graph is Sergio Agüero falling from
being owned by more than half of all teams to just a third in a single week.

### Salah's ownership vs value over time

{{< figure src="salah-ownership-value-over-time.png" caption="Salah's ownership and value over time" >}}

While the FPL pricing algorithm is not fully understood, it's heavily influenced by demand. Here, we
can see how Salah's price tightly tracks the percentage of teams he is owned by, and this trend
applies for most players.

