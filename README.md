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


