#HOW TO - MONGODB

Quick and dirty guide on "how to MongoDB".

## INSTALL

**Import the public key used by the package management system**

    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927

**Create a list file for MongoDB**

    echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

**Reload local package database**

    sudo apt-get update

**Install the MongoDB packages**

    sudo apt-get install -y mongodb-org

## RUN

**START / STOP / RELOAD MONGODB**

    service mongod start
    service mongod stop
    service mongod restart


# Deploy Replica Set

A replica set in **MongoDB** is a group of mongod processes that maintain the same data set. Replica sets provide redundancy and high availability, and are the basis for all production deployments. This section provides tutorials for common tasks related to replica sets.

## Requirements

Before you can deploy a replica set, you must install MongoDB on each system that will be part of your replica set (we meed at least 3 instances). If you have not already installed MongoDB, see the [installation tutorials](https://docs.mongodb.org/manual/installation/#tutorial-installation "Installation").

All members of a replica set must be able to connect to every other member of the set to support replication. Always verify connections in both “directions.” Networking topologies and firewall configurations can prevent normal and required connectivity, which can block replication.

To configure the network bind edit the mongo config file (/etc/mongod.conf under Unix systems):

    # network interfaces
    net:
        port: 27017
        bindIp: PRIVATE-IP

Be sure to setup internal DNS name to reach all the different nodes from one virtual machine to an other. You can simply modify the **/etc/hosts** file and add your DNS resolution, for example:

    192.168.0.1     mongo1.alexcomu
    192.168.0.2     mongo2.alexcomu
    192.168.0.3     mongo3.alexcomu

So, to check the bidirectional test of networking try from **mongo1**:

    `mongo --host mongo2.alexcomu --port 27017`
    `mongo --host mongo3.alexcomu --port 27017`

From **mongo2**:

    `mongo --host mongo1.alexcomu --port 27017`
    `mongo --host mongo3.alexcomu --port 27017`

From **mongo3**:

    `mongo --host mongo1.alexcomu --port 27017`
    `mongo --host mongo2.alexcomu --port 27017`

If any connection, in any direction fails, check your networking and firewall configuration and reconfigure your environment to allow these connections.

## Configuration Procedure

The following procedure outlines the steps to deploy a replica set when access control is disabled.

### 1 - Start each member of the replica set with appropriate options

For each member, start a **mongod** and specify the replica set option (**replSet**). If the application connects to more than one replica set, each set should have a dinstict name. In our case we'll use the same replica set name, for instance:

    mongod --replSet "rs0"

We can also specify the replica set name in the configuration file (be careful, YAML syntax does not support indentation, use space!):

    replication:
       oplogSizeMB: <int>
       replSetName: <string>
       secondaryIndexPrefetch: <string>
       enableMajorityReadConcern: <boolean>

### 2 - Connect a mongo shell to a replica set member

Connect your shell to Mongo:

    mongo --host 192.168.0.1 --port 27017

### 3 - Initiate the replica set

On one and one only member of the replica set, use **rs.initiate()** to initiates a set that consists of the current member and that uses the default replica set configuration.

    rs.initiate()

### 4 - Verify the initial replica set configuration

Use **rs.status** to display the replica set configuration object:

    rs.status()

The replica set configuration object will looks like:

    rs1:PRIMARY> rs.conf()
        {
            _id" : "rs1",
            "version" : 1,
            "protocolVersion" : NumberLong(1),
            "members" : [
                {
                    "_id" : 0,
                    "host" : "192.168.0.1:27017",
                    "arbiterOnly" : false,
                    "buildIndexes" : true,
                    "hidden" : false,
                    "priority" : 1,
                    "tags" : {
                        },
                    "slaveDelay" : NumberLong(0),
                    "votes" : 1
                }
            ]
        }

### 5 - Add the remaining members to the replica set.

We have to add the remaining members to our replica set using **rs.add()**. You must be connected to the **primary** member to add the others, in our case is **mongo1.alexcomu** instance.

    rs.add("mongo2.alexcomu")
    rs.add("mongo3.alexcomu")

Check the status to identify the primary in replica set. And we're done! Congrats!

### 5 Enable read-mode.

From a secondary node is possible to enable the read mode using this simple command:

    rs.slaveOk()
