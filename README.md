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

So, to check the bidirectional test of networking try from **node1**:

    `mongo --host node2.com --port 27017`
    `mongo --host node3.com --port 27017`

From **node2**:

    `mongo --host node1.com --port 27017`
    `mongo --host node3.com --port 27017`

From **node3**:

    `mongo --host node1.com --port 27017`
    `mongo --host node2.com --port 27017`

If any connection, in any direction fails, check your networking and firewall configuration and reconfigure your environment to allow these connections.
