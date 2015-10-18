---
title: "Setting up replication on MySQL"
posted: 2012-01-26T00:00
tags: [MySQL, replication, backup, howto, technical, video]
vimeo_video: 126037265
track: Technical
---
Using replication is a fast and easy way to keep a continuous backup of data in MYSQL. It cannot guard against user error and accidental deletes, and therefore should not be the only tool in use for backups; however, when recovering from a server outage it is nice to know you will not loose more than a second or two of data.

Of course for my sites, this means that I have lost nothing because my data does not change very often, but for sites with a large volume of transactions every day, this is a great way to minimize any data loss. The best part about setting up replication is just how easy it is! In this video walkthrough I take you through setting up two brand new MySQL instances for replication. I start with two brand new servers with nothing but MySQL installed on fedora.

## Setup Server

First, we need to setup two servers.  One for our master and one for our slave.  I used a basic Fedora 15 server from Rackspace cloud.  Rackspace offers cheap servers especially at the low end for testing and are easy to turn on and off.  Once I had my servers spun up and running I installed emacs, nmap, and MySQL.

I used the following command on a fresh fedora 15 installation:

`yum install emacs nmap MySQL MySQL-server`

## Master Setup

Next, we need to setup the master server for replication.  To do this we need to edit the my.cnf file on the master, and tell the MySQL server to use a binlog, give it a serverId, flush its binlog on each tx commit, and finally to sync its binlogs with slaves.

my.cnf on the master looks like this:

```
log-bin=MySQL-bin
server-id=1
innodb_flush_log_at_trx_commit=1
sync_binlog=1
```

Now lets create a database, I called mine testdb, and ran the following command to create it:

`mysqladmin create testdb`

Now lets create a test table to store some data, I did mine with the following command:

```
CREATE TABLE `TestTable`(
`id` INTEGER AUTO_INCREMENT NOT NULL PRIMARY KEY,
`created` DATETIME NOT NULL,
`value1` VARCHAR(255) DEFAULT NULL,
`value2` VARCHAR(255) DEFAULT NULL
)ENGINE=InnoDB;
```

Now, lets insert some data into the table. I inserted data with the following commands:

```
INSERT INTO TestTable (created, value1, value2) VALUES (NOW(), '1','2');
INSERT INTO TestTable (created, value1, value2) VALUES (NOW(), '3','4');
INSERT INTO TestTable (created, value1, value2) VALUES (NOW(), '5','6');
INSERT INTO TestTable (created, value1, value2) VALUES (NOW(), '7','8');
INSERT INTO TestTable (created, value1, value2) VALUES (NOW(), '9','10');
```

Next, we will need to create a user for the slave to connect as, I granted access with the following command:

```
CREATE USER 'repl'@'%' IDENTIFIED BY 'slavepass';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
```

Now we are ready to do some real work.  First we need to lock the tables for a moment while we make the location of the binlogs and also get a database dump at that precise moment in the binlog.

First, I flushed and locked the table with the following command:

`FLUSH TABLES WITH READ LOCK;`

Next, I got the exact place in the binlog by looking at the master status. I got the status of the master with the following command (be sure to take a note of the file and log position):

`SHOW MASTER STATUS`

Now that I know where I am currently in the binlog I get a mysqldump at this exact time in the binlog.  Remember no more data is being written, because I have locked the database. I did a MySQL dump with the following command:

mysqldump testdb > testdb_dump.sql

At this point, I have a mysqldump and I know where I was in the binlogs when this mysqldump happened.  So it is safe to unlock the tables and allow data to continue to be written.  I unlocked the tables with the following command:

`UNLOCK TABLES;`

We can assume that data will continue to be written to the database at this point.  To test this, lets write a few more rows before we continue.  I used the following command:

```
INSERT INTO TestTable (created, value1, value2) VALUES (NOW(), '21','22');
INSERT INTO TestTable (created, value1, value2) VALUES (NOW(), '23','24');
INSERT INTO TestTable (created, value1, value2) VALUES (NOW(), '25','26');
INSERT INTO TestTable (created, value1, value2) VALUES (NOW(), '27','28');
INSERT INTO TestTable (created, value1, value2) VALUES (NOW(), '29','30');
```

## Slave Setup

Now we are all setup on the master, so just a few things I need to do on the slave.  First I need to copy the mysqldump to the slave.  Then once there, create the database and import the mysqldump.  I ran the following commands to do this:

```
mysqladmin create testdb
mysql testdb < testdb.sql
```

At this point I have a copy of the master but the way it looked like when the mysqldump was made, but nothing since then, and of course nothing new is being copied over yet.

The first thing I need to do is modify the my.cnf file.  The file on my slave is a lot simpler and need only contain a server-id.  The server-if MUST be different than the master and different than any other slaves I might happen to have also reading from this master.

my.cnf on my slave looks like this: `server-id=2`

The only thing left to do now is to tell the slave to start replicating from the master.  To do this we tell the slave the host, username, password, as well as the binlog filename and the log position to start from.  Remember this is not the current position in the binlog, rather, this is the position of the binlog when we made the mysqldump earlier.  Remember we ran the command to show the master status and made a not of the log file position and filename?  We will use it here.  This way MySQL will start replicating not from the current moment, but from the moment the mysqldump was created.

I ran the following command on the slave to make it point to the master:

```
CHANGE MASTER TO
MASTER_HOST='50.57.234.6',
MASTER_USER='repl',
MASTER_PASSWORD='slavepass',
MASTER_LOG_FILE='MySQL-bin.000001',
MASTER_LOG_POS= 107;
```

And we are done!  First thing we will see is the rows that were written after the mysqldump was made show up to the slave.  Once that happens we know replication is working.  We can test it out by adding a few more rows to the master and watching them get copied over to the slave.

I ran the following command on the slave to add a few more rows:

```
INSERT INTO TestTable (created, value1, value2) VALUES (NOW(), '31','32');
INSERT INTO TestTable (created, value1, value2) VALUES (NOW(), '33','34');
INSERT INTO TestTable (created, value1, value2) VALUES (NOW(), '35','36');
INSERT INTO TestTable (created, value1, value2) VALUES (NOW(), '37','38');
INSERT INTO TestTable (created, value1, value2) VALUES (NOW(), '39','40');
```