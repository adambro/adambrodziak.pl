---
title: "Log retention in ELK stack"
datePublished: Sun May 07 2023 15:32:41 GMT+0000 (Coordinated Universal Time)
cuid: clhdkoxfo000j09mq8fv2f6w6
slug: log-retention-in-elk-stack
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1683473248840/9abb5384-14aa-4bd0-a99f-38e84defc5e1.jpeg
tags: logstash, elasticsearch, elk, logging

---

Developers kept complaining that they can't find recent logs in Kibana. It happened before for many reasons (worthy of another post), but this time was different. There was no evident problem with the log structure or FluentD log shipper anymore.

We've noticed that one app went haywire and started sending logs like crazy. Because of that disks on Elasticsearch nodes got full and ES started to reject new logs. when the disk gets full Elasticsearch switches all indexes into read-only mode.

## Curator for Logstash

Curator is a solution that allows you to set how many days you want to keep your log indexes in Elasticsearch. That is the most popular configuration.

Keeping the last 2 weeks of logs using Curator was our setup. Normally it worked just fine. With predictable log influx, it can be managed with Curator. You can calculate how many days of logs to keep not to overflow the storage on ES nodes.

The problem is when more events are arriving due to some problem with the app or some kind of DoS attack for example. In such case, fresh logs are the most valuable to detect the attack progress or how an outage is spreading across the system.

The other option is to set to remove (or apply any other supported action) if an index grows to a certain size in gigabytes. That gets closer to an ideal scenario where we maximize disk space utilization. That is if you know your disk size for the whole cluster upfront and want to manage those values across clusters (dev, test, prod).

What I wanted was simple. Keep *as much logs* as available space allows, but *do not drop log events* when disks are full (more on that later).

Curator does not have such mode, unfortunately. I've been looking around for alternatives but found nothing.

## The solution is Bash script

Fortunately checking for disk usage on nodes is fairly easy in Elasticsearch API. So with a few `curl` calls and a sprinkle of bash scripting here's the solution to avoid lost data because of full disk.

```bash
#!/bin/bash
# Newline\tab as only separator, required for for loop
IFS=$'\n\t'
# Fail on first error
set -euo pipefail

ELASTIC_URL=${ELASTIC_URL:=localhost:9200}
# At 90% usage ES will try to move shards to other nodes. See `disk.watermark.high` in docs.
DISK_WATERMARK=88

NODES_UTILIZATION=$(curl --fail-with-body -s -X GET "$ELASTIC_URL/_cat/allocation?h=disk.percent&pretty")
for DISK_USAGE in $NODES_UTILIZATION; do
    if (($DISK_USAGE > $DISK_WATERMARK)); then
        OLDEST_INDEX="$(curl --fail-with-body -s -X GET "$ELASTIC_URL/_cat/indices/logstash-*?h=index&s=index" | head -n 1)"
        curl --fail-with-body -s -X DELETE "$ELASTIC_URL/$OLDEST_INDEX"
        exit 0
    fi
done
```

As you can see it is pretty straightforward. One caveat: it's using `--fail-with-body` param added to curl 7.76.0 version, so it might not be available in older Linux distributions. That is just to show the error response from ES server for debugging.

## Run script periodically

Logstash indexes are created daily. Actually, the index name follows `logstash-YYYY-MM-DD` format by default. This is also the assumption in the script above in `_cat/indices/logstash-*` GET query.

However, to make the script efficient it should be run more often than once a day. The reason is some app could go haywire with logging and fill up storage in the evening. In such cases we have lost data on what happened around that failure.

The solution is simple. Make the script **run by cron every hour**. It worked for us flawlessly.

## Why such disk usage values?

Why delete an index when circa 90% disk usage is reached? It is related to how Elasticsearch behaves where very little storage space is left.

The [official Elasticsearch docs](https://www.elastic.co/guide/en/elasticsearch/reference/8.7/modules-cluster.html#disk-based-shard-allocation) are not very clear, so let me briefly explain what happens when usage reaches a given level for default values.

Assuming on any given node disk is being filled:

* at 85% - ES will stop allocating shards to that node, see `disk.watermark.low` setting.
    
* at 90% - ES will try to re-allocate shards to other nodes, see `disk.watermark.high` setting.
    
* at 95% - ES enforces read-only index block, see `disk.watermark.flood_stage` setting.
    

Preventing reaching 90% is the goal here, but even that could not help. Imagine one node disk is over 90%, so ES will try to move shards, but it will fail. Most likely other nodes will be over 85% already, so allocating is blocked. That is for equal disk sizes and shards being spread evenly - something to strive for anyway.

Let's assume Elasticsearch could move a shard from a node that is filling up to another one. Now think of the load that moving a giant slab of data (shards with logs are pretty big) from one ES node to another. Such operation can grind the cluster to a halt. We don't want that.

To be on the safe side we target 85% disk usage then. If that level is reached nothing active is being done, the node is just cordoned (to use Kubernetes lingo). Elasticsearch will not try to shuffle shards around and we have some room to spare before it does.

## Is it that simple?

Well yes, but actually no ;) The idea behind it is so brilliantly simple that I was sure somebody has implemented it. However, I have found nothing, not even a post on some obscure blog ;)

On the other hand log aggregation in ELK stack is not an easy job. You have to define log even structure, decide what is indexed and what is not, and create a template for indexes. That is, if you have control over logging clients in apps, if not it gets much worse.

On top of that shard replicas, hot and cold indexes, and archived indexes are probably on your mind too. That's a lot and something that deserves another blog post. Let me know if you're interested.

Make your logs like Pokemons. Gotta Catch 'Em All