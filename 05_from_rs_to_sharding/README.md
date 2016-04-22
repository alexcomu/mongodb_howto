# From Replica Set to Sharding

This quick and simple tutorial converts a single three-member replica set to a sharded cluster with two shards. Each shard is an independent three-member replica set. We'll cover the following steps:

* Create the initial three-member replica set and insert data into a collection
* Start the config servers and a **mongos**
* Add the initial replica set as a shard
* Create a second shard and add to the cluster
* Shard the desired collection

## Requirements

We need 10 servers: one server for the **mongos** and three servers each for the first / second replica set and for the config replica set.

Each server must have a resolvable domain, hostname, or IP address within your system.

## Setup Initial Replica Set

Edit the config file of **mongo1**, **mongo2** and **mongo3** as the following:

    replication:
     replSetName: "rs0"

Connect to a mongo shell, start the replica set and add the secondaries:

    mongo --host IP-HOST
    rs.initiate()
    rs.add("mongo2")
    rs.add("mongo3")

## Create and populate a Collection

Use this simple script to create and populate a new db "test" with collection "test_collection":

    use test
    var bulk = db.test_collection.initializeUnorderedBulkOp();
    people = ["Marc", "Bill", "George", "Eliot", "Matt", "Trey", "Tracy", "Greg", "Steve", "Kristina", "Katie", "Jeff"];
    for(var i=0; i<1000000; i++){
       user_id = i;
       name = people[Math.floor(Math.random()*people.length)];
       number = Math.floor(Math.random()*10001);
       bulk.insert( { "user_id":user_id, "name":name, "number":number });
    }
    bulk.execute();


## Configure The Config Server Replica Set

Start **mongo7**, **mongo8** and **mongo9** as config replica set:

    mongod --configsvr --replSet configReplSet

Connect to a shell to mongo7 with port 27019 (default) and run rs.initiate():

    mongo --host mongo7 --port 27019
    rs.initiate( {
       _id: "configReplSet",
       configsvr: true,
       members: [
          { _id: 0, host: "mongo7:27019" },
          { _id: 1, host: "mongo8:27019" },
          { _id: 2, host: "mongo9:27019" }
       ]
    } )

## Start Mongos Instances and Init the Shard

Connect to **mongo6** ad start **mongos** process. So, from mongo6:

    mongos --configdb configReplSet/mongo7:27019,mongo8:27019,mongo9:27019 --chunkSize 1

Connect to a mongos shell:

    mongo mongo6:27017/admin
    use admin
    sh.addShard("rs0/mongo1:27017,mongo2:27017,mongo3:27017")

## Add second Shard

Start **mongo4**, **mongo5** and **mongo10** as replica set:

    mongod --replSet "rs1"

Connect to **mongo4** and run:

    rs.initiate()
    rs.add("mongo5")
    rs.add("mongo10")

Then connect to **mongo6** to add the new shard:

    sh.addShard("rs1/mongo4:27017,mongo5:27017,mongo10:27017")

## Shard a collection

Then connect to **mongo6** and enable sharding for a database:

    sh.enableSharding( "test" )

Now we have to determine the shard key. The shard key determines how MongoDB distributes the documents between shards. Good shard keys:

* have values that are evenly distributed among all documents
* group documents that are often accessed at the same time into contiguous chunks
* allow for effective distribution of activity among shards

Be careful because you **cannot change** the shard key! In our case this procedure will use the number field as the shard key for test_collection.

So, we have a no empty collection, we have to create an Index!

    db.test_collection.createIndex( { number : 1 } )

    {
    	"raw" : {
    		"rs0/mongo1:27017,mongo2:27017,mongo3:27017" : {
    			"createdCollectionAutomatically" : false,
    			"numIndexesBefore" : 1,
    			"numIndexesAfter" : 2,
    			"ok" : 1,
    			"$gleStats" : {
    				"lastOpTime" : Timestamp(1461313737, 1),
    				"electionId" : ObjectId("7fffffff0000000000000004")
    			}
    		}
    	},
    	"ok" : 1
    }

Now we are ready to shard the collection:

    use test
    sh.shardCollection( "test.test_collection", { "number" : 1 } )

    { "collectionsharded" : "test.test_collection", "ok" : 1 }

The **balancer** will redistribute chunks of documents when it next runs. As clients insert additional documents into this collection, the **mongos** will route the documents between the shards.

## Confirm the shard is balancing

To confirm balancing activity, run **db.stats()** or db.printShardingStatus() in the test database. The result will be something like:

    {
    	"raw" : {
    		"rs0/mongo1:27017,mongo2:27017,mongo3:27017" : {
    			"db" : "test",
    			"collections" : 1,
    			"objects" : 966779,
    			"avgObjSize" : 70.83201331431485,
    			"dataSize" : 68478903,
    			"storageSize" : 48963584,
    			"numExtents" : 0,
    			"indexes" : 2,
    			"indexSize" : 24526848,
    			"ok" : 1,
    			"$gleStats" : {
    				"lastOpTime" : Timestamp(0, 0),
    				"electionId" : ObjectId("7fffffff0000000000000004")
    			}
    		},
    		"rs1/mongo10:27017,mongo4:27017,mongo5:27017" : {
    			"db" : "test",
    			"collections" : 1,
    			"objects" : 40426,
    			"avgObjSize" : 70.83001038935339,
    			"dataSize" : 2863374,
    			"storageSize" : 741376,
    			"numExtents" : 0,
    			"indexes" : 2,
    			"indexSize" : 471040,
    			"ok" : 1,
    			"$gleStats" : {
    				"lastOpTime" : Timestamp(0, 0),
    				"electionId" : ObjectId("7fffffff0000000000000001")
    			}
    		}
    	},
    	"objects" : 1007205,
    	"avgObjSize" : 70,
    	"dataSize" : 71342277,
    	"storageSize" : 49704960,
    	"numExtents" : 0,
    	"indexes" : 4,
    	"indexSize" : 24997888,
    	"fileSize" : 0,
    	"extentFreeList" : {
    		"num" : 0,
    		"totalSize" : 0
    	},
    	"ok" : 1
    }


The result of **db.printShardingStatus()**:

    --- Sharding Status ---
    sharding version: {
    	"_id" : 1,
    	"minCompatibleVersion" : 5,
    	"currentVersion" : 6,
    	"clusterId" : ObjectId("5718f84b011eb032a533a670")
    }
      shards:
    	{  "_id" : "rs0",  "host" : "rs0/mongo1:27017,mongo2:27017,mongo3:27017" }
    	{  "_id" : "rs1",  "host" : "rs1/mongo10:27017,mongo4:27017,mongo5:27017" }
      active mongoses:
    	"3.2.5" : 1
      balancer:
    	Currently enabled:  yes
    	Currently running:  yes
    		Balancer lock taken at Fri Apr 22 2016 08:33:06 GMT+0000 (UTC) by mongo6:27017:1461311107:1782615144:Balancer
    	Collections with active migrations:
    		test.test_collection started at Fri Apr 22 2016 08:33:05 GMT+0000 (UTC)
    	Failed balancer rounds in last 5 attempts:  0
    	Migration Results for the last 24 hours:
    		8 : Success
      databases:
    	{  "_id" : "test",  "primary" : "rs0",  "partitioned" : true }
    		test.test_collection
    			shard key: { "number" : 1 }
    			unique: false
    			balancing: true
    			chunks:
    				rs0	126
    				rs1	8
    			too many chunks to print, use verbose if you want to force print

To view the active shards:

    mongos> use admin
        switched to db admin
    mongos> db.runCommand({listShards:1})
        {
        "shards" : [
            {
                "_id" : "rs0",
                "host" : "rs0/mongo1:27017,mongo2:27017,mongo3:27017"
            },
            {
                "_id" : "rs1",
                "host" : "rs1/mongo10:27017,mongo4:27017,mongo5:27017"
            }
        ],
        "ok" : 1
        }


Run these commands for a second time to demonstrate that chunks are migrating from rs0 to rs1. And we are done! :)
