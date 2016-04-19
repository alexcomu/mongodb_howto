# Arbiter on Replica Set

Arbiters are **mongod** instances that are part of a replica set but do not hold data. Arbiters participate in elections in order to break ties. If a replica set has an even number of members, add an arbiter.

Arbiters have minimal resource requirements and do not require dedicated hardware. You can deploy an arbiter on an application server or a monitoring host.

**IMPORTANT** Do not run an arbiter on the same system as a member of the replica set.


## Add an arbiter on a new machine

Sequence of operation to configure a new arbiter for our replica sets.

### 1 - Create data directory

Create a data directory for the arbiter. The mongod instance uses the directory for configuration data, will not hold the data set.

    mkdir /data/arb

### 3 - Start the Arbiter

Start the arbiter specifying the data directory and the replica set name. In our case:

    mongod --port 30000 --dbpath /data/arb --replSet rs0

### 4 - Add the arbiter to the set

Connect to the primay node and add the arbiter to the replica set using **rs.addArb()**:

    rs.addArb("arbiter1.alexcomu:30000")


