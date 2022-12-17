---
title:  "Grafana Dashboard for Pi-hole Stats"
description: "Excessive Pi-hole monitoring with pretty graphs!"
author: avojak
image: https://s3.amazonaws.com/avojak.com/2022-12-15-pihole-influxdb-monitor/dashboard.png
tags:
  - software
  - app
  - evergreen
---

One of the most important parts of my home network and homelab is my [Pi-hole](https://pi-hole.net), because who likes internet ads?
I also enjoy pretty graphs, so I recently started looking for a way to add Pi-hole stats to my Grafana dashboard. Unfortunately all
the other monitoring tools that I came across had a fatal flaw in how they recorded the number of queries throughout the day:
they simply plotted the "queries today" value at each poll interval. However, Pi-hole tracks queries in 10-minute rolling windows.
Each value represents the number of queries for the past 23 hours and 50 minutes, plus the number of queries in the current 10-minute
window. Charting this over time looks like a sawtooth, and isn't what we want. My goal was to replicate the same chart that the Pi-hole
dashboard shows.

Using the [REST API](https://discourse.pi-hole.net/t/pi-hole-api/1863), I simply poll the `overTimeData10mins` endpoint and record those
datapoints in InfluxDB. Sure, there will be a lot of duplicate data, but those datapoints will simply be overwritten in the database.
Once I got this working, I made quick work of populating all the other data that the API provides.

```python
# Ads over time
for timestamp,count in ads_over_time.items():
    points.append(Point.from_dict(
        {
            "measurement": "over_time_data",
            "tags": tags,
            "fields": {
                "ads_over_time": count
            },
            "time": int(timestamp)
        },
        WritePrecision.S
    ))
```

The last major challenge was displaying the tables of Top Queries, Top Ads, and Top Sources. For most other datapoints I was able to simply
take the JSON data from the API and drop it in as the field data for the database record. However, attempting to group multiple records by time,
and sort them turned into a massive headache, and failed in several edge cases. 
To make querying the data easier, I converted the JSON data from the REST API into a comma-separated 
string. This way there would always be a single, latest record in the database with the top items.

```python
points.append(Point.from_dict(
    {
        "measurement": "top_queries",
        "tags": tags,
        "fields": {
            "top_10": self._json_to_csv(stats.pop('top_queries'))
        },
        "time": now_seconds
    },
    WritePrecision.S
))
```

In the end I tossed together a little Grafana dashboard to display all the data I collected:

<figure class="constrained" markdown=1>
![Grafana dashboard](https://s3.amazonaws.com/avojak.com/2022-12-15-pihole-influxdb-monitor/dashboard-full.png)
</figure>

The script is available as a Docker image on DockerHub, and can be run easily:

```bash
docker run -d \
    -e PIHOLE_ALIAS="pihole" \
    -e PIHOLE_ADDRESS="http://pi.hole" \
    -e PIHOLE_TOKEN="pihole_api_token" \
    -e INFLUXDB_ADDRESS="http://influxdb:8086" \
    -e INFLUXDB_ORG="my-org" \
    -e INFLUXDB_TOKEN="super_secret_token" \
    -e INFLUXDB_BUCKET="pihole" \
    avojak/pihole-influxdb:latest
```

Alternatively you can run the Python application directly from the command-line:

```bash
python3 pihole-influxdb.py \
    --pihole-alias "pihole" \
    --pihole-address "http://pi.hole" \
    --pihole-token "pihole_api_token" \
    --influxdb-address "http://influxdb:8096" \
    --influxdb-org "my-org" \
    --influxdb-token "super_secret_token" \
    --influxdb-bucket "pihole"
```

Check out the full source over on my GitHub:

{% include github-card.html
  user="avojak"
  repository="pihole-influxdb-monitor"
%}