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


### Shards

Shards store the data. To provide high availability and data consistency, in a production sharded cluster, each shard is a **replica set**. For more information on replica sets, see the chapter *02_replicaset*.

### Query Routes

Query Routers, or **mongos** instances, interface with client applications and direct operations to the appropriate shard or shards. A client sends requests to a mongos, which then routes the operations to the shards and returns the results to the clients.

### Config Servers

Config servers store the cluster’s metadata. This data contains a mapping of the cluster’s data set to the shards. The query router uses this metadata to target operations to specific shards.

To have more information on Sharding concepts please visit the [MongoDB WebSite](https://docs.mongodb.org/manual/core/sharding-introduction/).


## Deploy a Sharded Cluster

Starting from MongoDB 3.2, config servers for sharded clusters can be deployed as a replica set. The config servers store the sharded cluster’s metadata. The following steps deploy a three member replica set for the config servers.

### Deploy Config Server Replica Set

To run a config server store we have to apply the following steps. First of all we have to modify the config file and update the replica set and sharding info:

    sharding:
       clusterRole: configsvr
    replication:
       replSetName: configReplSet
    net:
       port: <port>
    storage:
       dbpath: <path>

Then, connect to a mongo shell to one of the config servers and run **rs.initiate()** to initiate the replica set. After that, add the others replica set nodes using **rs.add("mongo2.alexcomu")**.

### Start the mongos instances

The **mongos** instances are lightweight and do not require data directories. You can run a mongos instance on a system that runs other cluster components, such as on an application server or a server running a mongod process. By default, a mongos instance runs on port 27017.

When you start the mongos instance, specify the config servers, using either the sharding.configDB setting in the configuration file or the --configdb command line option.

In our case, from mongo3.alexcomu and mongo2.alexcomu, run this command:

    mongos --configdb configReplSet/mongo1.alexcomu:27017 --port 27018

### Add Shards to the Cluster

A shard can be a standalone mongod or a replica set. In a production environment, each shard should be a replica set. Use the procedure in Deploy a Replica Set to deploy replica sets for each shard.

Connect from a mongo shell to a **mongos** instance:

    mongo --host mongo3.alexcomu --port 27018

Add each shard to the cluster using **sh.addShard()** method:

    sh.addShard("rs1/mongo2.alexcomu:27017")

Show the status using **sh.status()**

    mongos> sh.status()
    --- Sharding Status ---
      sharding version: {_id : 1,
    	"minCompatibleVersion" : 5,
    	"currentVersion" : 6,
    	"clusterId" : ObjectId("57178ced7d839494c0462caf")
    }
      shards:
    	{_id : "rs1",  "host" : "rs1/192.168.0.2:27017, mongo4.alexcomu:27017" }
      active mongoses:
    	"3.2.5" : 2
      balancer:
    	Currently enabled:  yes
    	Currently running:  no
    	Failed balancer rounds in last 5 attempts:  5
    	Last reported error:  could not find host matching read preference { mode: "primary" } for set rs1
    	Time of Reported error:  Wed Apr 20 2016 18:04:15 GMT+0200 (CEST)
    	Migration Results for the last 24 hours:
    		No recent migrations
      databases:


### Enable Sharding for a Database

Create a DB on your primary node:

    use alexcomu
    db.createCollection(user)
    db.user.insert({"name" : "Alex", "surname" : "Comu", "address" : "casamia", "phone" : "0123", "email" : "alex.comunian@gmail.com"})
    db.user.insert({"name":"Ciccio", "surname":"Pasticcio", "address" : "casaSua", "phone" : "456789", "email" : "ciccio.pasticcio@gmail.com"})

Then, connect to a **mongos** instance as before:

    mongo --host mongo3.alexcomu --port 27018

Use the command **sh.enableSharding()** to enable sharding mode on a database.

    sh.enableSharding("alexcomu")
    { "ok" : 1 }

From the status command:

    ...
    databases:
    {_id : "alexcomu",  "primary" : "rs1",  "partitioned" : true }

### Enable Sharding for a Collection

We have to determine what you will use for the shard key. Your selection of the shard key affects the efficiency of sharding. If the collection already contains data you must create an index on the shard key using **createIndex()**. If the collection is empty then MongoDB will create the index as part of the sh.shardCollection() step.

So, in this case we have to create an Index:

    db.user.createIndex({email: 1})
    {
    	"createdCollectionAutomatically" : false,
    	"numIndexesBefore" : 1,
    	"numIndexesAfter" : 2,
    	"ok" : 1
    }

Only after this operation we can set the shardCollection:

    sh.shardCollection("alexcomu.user", { "email": 1} )
    { "collectionsharded" : "alexcomu.user", "ok" : 1 }

In this case the **user** collection in the **alexcomu** database using the shard key {"emal": 1}. This shard key distributes documents by the value of the email field.





a
