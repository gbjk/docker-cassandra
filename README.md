Cassandra in Docker
===

This repository is a clone of the spotify cassandra docker images. It provides everything you need to run Cassandra in Docker, and is tuned for fast
container startup.

Why?
---
While naive Cassandra images take around 30 seconds to start, our containers take only a few seconds.
Optimizations include:

* Disabling vnodes. We don't use them at Spotify, and Cassandra starts much faster without them
  (~10 sec).
* Disabling something called "waiting for gossip to settle down" because there is no gossip in a
  one-node cluster (another ~10 sec).

Examples
--------
# Get some tokens
docker run gbjk/cassandra:2.0.12 token-generator 3
# Run the primary node
docker run -d --env CASSANDRA_TOKEN=0 -p 127.0.10.1:9042:9042 --name cdb01 gbjk/cassandra:cluster
# Now run two more, linked to the first
docker run -d --env CASSANDRA_TOKEN=$SECOND_TOKEN -p 127.0.10.1:9042:9042 --link cdb01:seed --name cdb02 gbjk/cassandra:cluster
docker run -d --env CASSANDRA_TOKEN=$THIRD_TOKEN -p 127.0.10.1:9042:9042 --link cdb01:seed --name cdb03 gbjk/cassandra:cluster

In the box
---
* **gbjk/cassandra**

  This is probably the image you want, it runs a one-node Cassandra cluster.
  Built from the `cassandra` directory.

* **gbjk/cassandra:cluster**

  Runs a Cassandra cluster. Expects `CASSANDRA_SEEDS` and `CASSANDRA_TOKEN` env variables to be set.
  If `CASSANDRA_SEEDS` is not set, node acts as its own seed, unless linked to a docker container named seed. If `CASSANDRA_TOKEN` is not set, the
  container will not run. Built from the `cassandra-cluster` directory.

* **gbjk/cassandra:base**

  The base image with an unconfigured Cassandra installation. You probably don't want to use this
  directly. Built from the `cassandra-base` directory.

Notes
---
Things are still under heavy development:
* Only Cassandra 2.0 with almost-generic config (miles away from what we actually run Cassandra
  with) is supported so far.
* There's nothing to help you with tokens and stuff.
