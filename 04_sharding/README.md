# Sharding

Sharding is a method for storing data across multiple machines. MongoDB uses sharding to support deployments with very large data sets and high throughput operations.

Database systems with large data sets and high throughput applications can challenge the capacity of a single server. High query rates can exhaust the *CPU* capacity of the server. Larger data sets exceed the storage capacity of a single machine. Finally, working set sizes larger than the system’s *RAM* stress the *I/O* capacity of disk drives.

To address these issues of scales, database systems have two basic approaches: **vertical scaling** and **sharding**.

## Vertical scaling

Vertical scaling adds more CPU and storage resources to increase capacity. This solution has limitations: large number of *CPUs* and large amount of *RAM* are **more expensive** than smaller systems.

## Sharding

Sharding, or **hotizontal scaling**, divides the data set and distributes the data over multiple servers, or **shards**. Each shard is an indipendent database, and collectively, the shards make up a single logical database.

<p align="center">
  <img src="sharded-collection.png" alt="Sharded Collection"/>
</p>

Why use sharding:

* Sarding reduces the number of operations each shard handles. Each shard processes fewer operations as the cluster grows. As a result, a cluster can encrease capacity and throughput *horizontally*

* Sharding reduces the amount of data that each server needs to store. Each shard stores less data as the cluster grows.

## Sharding with MongoDB

Shared cluster has the following components:
* Shards
* Query routers
* Config servers

<p align="center">
  <img src="sharded-cluster-production-architecture.png" alt="Sharded Collection"/>
</p>
