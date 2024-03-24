# ScyllaDB Core Concepts
* Attila Toth - Developer Advocate, ScyllaDB
* LinkedIn: https://www.linkedin.com/in/attilapm


## Creating ScyllaDB Cloud using Terraform
* [Setting up ScyllaDB using Terraform](https://www.scylladb.com/2023/11/29/set-up-your-own-1m-ops-benchmark-with-scylladb-cloud-terraform/)
* [Zipf’s Law in Python](https://www.codedrome.com/zipfs-law-in-python/)
* [ScyllaDB Cloud](https://cloud.scylladb.com)

## Relational vs Non-relational Databases

Relational DB:
* Analyze data first, create data model
* Create application with queries to reference data schema

NoSQL:
* (Not Just) SQL
* Look at application first, data needed, work backwards to create data model that makes sense.
* Trying to be performant, create data model that makes sense not from the data perspective, but the query perspective.

## Data Modelling Terminology
* Cluster - Collection of ScyllaDB nodes.
* Node - machine/computer running ScyllaDB software
* Keyspace:
  * Multiple keyspaces in a cluster. Set some attributes on a per-keyspace basis.  
  * Different replication factors for datsets.

## Tables
* How ScyllaDB stores data
* Data partitioned according to partition key
* Partition stored on a node and replicated accross multiple nodes based on the RF
* Row - Unit how ScyllaDB stores data within a table.
* Column-Name/Value Pair - Key to get to value

Within Table:
* Multiple partitions, rows
* Partition Key in table definition used to create the partitions (rows)
* Can use partition key to get to row, and query other columns in that row.

## Replication Factor, Hashing
* RF in Partition Key defines how many copies of same data is stored in ScyllaDB nodes
* Partition Hash Function

## Primary Key
* Partition Key first column in the primary key
* Second can be the clustering key - important for data modelling.  Defines the sorting order within a partition
* For example, can set the Partition Key as ID, and then sort the rows by Time

## What is CQL?
* Cassandry Query Language - main language to interact with ScyllaDB
* Can also use Dynamo DB API
* Similar to SQL, but more limited - cannot do relational queries.  Optimized for performance.
* Data Definition (DDL) - Create/Delete/Alter, keyspaces, tables, 
* Data Manipulation - select, insert, update, delete, patch
* [CQL Native Data Types](https://opensource.docs.scylladb.com/stable/cql/types.html)
* Can connect with scylla-cqlsh - Python based tool:

```
pip install scylla-cqslsh
cqlsh <node ip>
```

## Collections
* Sets
* Lists
* Maps
* UDT - User Defined Types - create your own - can use to store things like phone numbers

## IOT Example
[IOT Project Page](https://iot.scylladb.com/stable/)

`docker exec -it scyllaU nodetool status`

* Node Status - UN = Up, Normal
* Nodetool Status 

Connect to ScyllaDB node: `docker exec -it scyllaU csqlsh`

* Can create table, insert row, etc.

* When defining table, can use `WITH CLUSTERING ORDER BY (key DESC)` to ensure the data is ordered properly.

## Choosing a Partition Key

Plan For:
* High Cardinality - want column in table with a lot of unique values
* Even Distribution - want to query all values within the column.  If small subset of IDs (for example), much better if all of the partitions get queried evenly.

Avoid:
* Hot Partition - Partition receiving most of the queries.
* Large Partition - Contains majority of the data, much larger in size than other partitions.

Good Examples:
* Username, UserID, UserID+Time, SensorID, SensorID+Time
* Any timeseries data, need to filter by time, good idea to add

Bad Examples:
* State, Age, Random stuff
* May be common value that will be frequently set
* Want unique values

## Choosing a Clustering Key
* Allow useful range queries.  For example, filter by time but not all queries

## Drivers
Native Driver + Recap:
* Client connect to nodes in cluster as the Coordinator Node
* ScyllaDB finds all the nodes with the data you are querying.
* Problem if Coordinator Node doesn't have the data - ScyllaDB needs to go to other nodes to get the data

Improvement - Token Aware Clients
* Routes queries to the right nodes, directly to a Coordinator node that  has the data replicata.
* Coordinator node can communicate with other nodes

Shard Aware Clients/Driver
* Route queries to the right nodes + Core
* Goes to the correct CPU core to deliver the data in a performant manner

Scylla Drivers:
* Shard Aware - Java, Go, Rust, C/C++, Python
* Token Aware - C#, PHP, Ruby, Node.JS, etc

## Q&A
Q: Is there sharding? \
A: Shard per core architecture.  
* When making a query, go directly to the CPU that can efficiently return the data.  Done automatically.
* Making better use of the CPU, machine, overall resources

Q:  Does the data type of the partition key affect performance? \
A:  Shouldn't be any performance difference between the type.

Q:  How to list all tables? \

A: Quick lookup:
* If you want to see the tables of a specific keyspace, run the following from the CQL Shell: \
`use keyspace_name; DESCRIBE tables;` 
* To list all the tables in all keyspaces, run:
`DESCRIBE tables;` \
Without choosing a keyspace first Note that this will also list system tables.
* You can also use `DESC keyspace your_keyspace_name` to get the entire schema

Q: When you insert a new row with the same primary key does it delete the old row and add a new one or does it just simply add a new row and this has to be cleaned up later? \
A:  
* If the table only has the partition key, it will override the previous data. 
* But if you add a clustering key, and these cluster are different from the previous one, you will generate a new row.
* For example: user_id: string partition_key name: string last_seen: ts clustering
* Every time the user does a new login, it will create a new now because the timestamp is different.

Q:  When you created the Docker container with ScyllaDB and then used csqlsh no username/password was prompted.  Did this bypass database access restriction? \
A:  You can enable auth on the Docker instances.  They're disabled by default.

Q: Does scylladb support secondary indexes or does clustering key perform that function? \
A: [Supports matrerialized Views and secondary indexes](https://university.scylladb.com/courses/data-modeling/lessons/materialized-views-secondary-indexes-and-filtering/)
* [Talk on index, filters, and other animals - explains the options](https://resources.scylladb.com/videos/indexes-filters-and-other-animals).

Q: Non CLI options \
A: ScyllaDB Monitoring - Toolkit for monitoring clusters, Set of Grafana dashboards prebuilt.  View into caching performs.
* Available for ScyllaDB Cloud and Open Source

Q: Incremental Backup/Restore? \
A: Panel Question - Better to discuss here.