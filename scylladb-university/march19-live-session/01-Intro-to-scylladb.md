# Intro to Scylla DB - March 19, 2024

https://university.scylladb.com
https://docs.scylladb.com
https://forum.scylladb.com

Guy Shtub: guy@scylladb.com
https://www.linkedin.com/in/guy-shtub

* Works in multiple datacenters
* Scales horizontally, can add/remove nodes
* Scales vertically, makes use of larger machines.
* Single digit latency read/write, high throughput

Design Decisions:
* Shard Per Core architecture.
* Each core gets its own resources (I/O, Networking, CPU) and is resposible for a portion of data.  Avoids context switching
* Translates to lower latency and better performance.

Ecosystem Compatibility
* Drivers in Go, Rust, Java, C#, etc
* Knows how to work Spark, Kafka, and other systems.
* Fully compatible with Apache Cassandra drivers.

Flavors:
* Open Source (Git)
* Enterprise -with Support
* ScyllaDB Cloud --- where is it hosted?

* All nodes in ScyllaDB created equally, each node can serve requests.
* No Master/Slave

Cluster - Node Ring
* Collection of nodes that work together
* At least 3 nodes, can scale up to hundreds of nodes
* Can add more nodes, and Scylla will repartition the data and stream
* For expected peak traffic, can pre-increase the number of nodes
* Traffic between nodes is Peer to Peer

Data Replication
* Uses replication factor (RF).  Number of nodes that hold a copy of the data.
* For example, if 5 nodes, RF 3 - each copy of data will be replicated to 3 nodes.  So 3 copies of the data.
* Works with HA.  If one node goes down, can still serve requests.
* Partitioner Hash Function - determines which nodes have the data.

Consistency Level
* CL:  # of nodes that must acknowledge read/write request.
* Per operation, tunable consistency - can define a different level per query.
* Quorum - majority of replicats have to acknowledge
* All - all replicas must acknowledge
* Trade off - low consistency increases availability but lowers consistency
* Possible to set local quorum for a query - data will be replicated within a single datacenter, then asynchronously replicated to the other datacenters.

Multiple Data Centers
* Topology aware
* Can have same cluster in multiple datacenters for performance, availability

## Read/Write Path

Cluster Level Write 1
* Client writes to a node (Coordinator Node)
* Can set RF and CL per query.
* Data is then written to the other nodes.  The other nodes will send an acknowledgement back to the coordinator node.
* If two of 3 nodes in RF3 acknowledge write, it is success (because you have quorum)
* If CL = All, and one node fails to write, then it would be considered a failure. 

## High Availability Lab



## Q&A

Q: How does the client know which request their node will be sent to? Is this abstracted to the client, or does it need to be network aware of all the nodes? 
A: You provide contact points to the client and it retrieves the node information from Scylla and will distribute connections across nodes (and shards for shard aware drivers)