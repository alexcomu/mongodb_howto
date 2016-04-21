# From Replica Set to Sharding

This quick and simple tutorial converts a single three-member replica set to a sharded cluster with two shards. Each shard is an independent three-member replica set. We'll cover the following steps:

* Create the initial three-member replica set and insert data into a collection
* Start the config servers and a **mongos**
* Add the initial replica set as a shard
* Create a second shard and add to the cluster
* Shard the desired collection

## Requirements

We need 10 servers: one server for the **mongos** and three servers each for the first / second replica set and for the config replica set.
