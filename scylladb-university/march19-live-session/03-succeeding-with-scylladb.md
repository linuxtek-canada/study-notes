# Succeeding with ScyllaDB

Speaker:  
* Patrick Bossman
* Email: patrick.bossman@scylladb.com
* LinkedIn: https://www.linkedin.com/in/bossman/

## Choose your Platform

ScyllaDB Cloud:
* AWS and GCP currently supported
* Azure is coming

Production:
* Scylla Cloud
* Public/Private Clouds
* Kubernetes Operator: https://operator.docs.scylladb.com/stable/generic.html

Fully Managed:
* Not on t2micro instances, as they are not production, don't have backup/repair
* Meant to be used and destroyed
* Manages scale out/scale in, backups
* Focus on creating keyspaces and application

## ScyllaDB AMI
* Machine Images for AWS, GCP, Azure
* Pre-packaged image with dependencies, kernel, configuration
* If use recommended instance types, optimizations is done already.
* Lot of people using these instances, "well worn" path
* https://opensource.docs.scylladb.com/stable/getting-started/install-scylla/index.html

## Download and Install
* OSS or Enterprise
* Run on any cloud on own hardware
* Important to run scylla_setup script to let auto-tune
* Optimize IO, CPU, Clock source, clock synchronization, mount options

## Configure cpuset
* If not using AMI, make sure to tune CPUs!
* Recommend to optimize use of CPUs between Scylla and NIC, changing later is painful
* When CPU 8 or higher, start to allocate some for ScyllaDB, and some for network traffic.  
* The larger instance type, if tuning not run, will have some queries running on shards/CPUs competing with network traffic, can impact performance.
* Have to reshard on each node to fix (painful), rewrite SS tables, takes time. 
* AMI already has this done.  Have to set during setup script

## Deployment Recommendations
* Design for HA - snitch, racks, replication settings
* Good mindset for any database to design for capacity, what state are you sizing for - disaster situation
* If mission critical looking for HA, capacity planning has to consider how to operatate in a disaster situation

## Capacity Planning
* Size so that "surviving domains" in a domain failure can carry the load (rack, region, etc)

## Supported Snitch Types (1)
* https://opensource.docs.scylladb.com/stable/operating-scylla/system-configuration/snitch.html
* https://opensource.docs.scylladb.com/stable/operating-scylla/system-configuration/snitch.html#ec2multiregionsnitch
* Snitch - how Scylla distributes the data in the cluster.
* Defines where the replicas will be stored.

SimpleSnitch
* Default, not for production use

RackInferringSnitch
* Binds nodes to DCs and Racks according to Broastcast IPs

GossipingPropertyFileSnitch - Use in Production
* Multi cluster deployments and nodes in various datacenters
* Can explicitly define which DC and Rack a specific node belongs to.
* Reads the config from a cassandra-rackdc.properties file located in /etc/scylla

## Supported Snitch Types (2)

Ec2Snitch (AMI Deployment Default)
* EC2 Single Cluster Deployments 
* All nodes in same region

EC2MultiRegionSnitch (use in Production)
* EC2 multi-cluster deployments across different regions

* Scylla reads underlying AWS EC2 metadata to determine this.

## Racks Awareness
Racks
* Set number of racks to number of replicas, desire odd number of racks/replicas (usually 3)
* If 2 of 3 replicas, one goes down, still have quorum, avoid split-brain problems
* When rack comes back online, can run repair and redistribute the data
* Can do rolling restarts, take down part of the database.  

## Racks Awareness (2)
* Some regions may not have multiple availability zones
* Want to have separate racks in separate AZ/DC to add resiliency.
* Try to isolate racks in separate failure domains.
* Want to have odd number of racks and replicas to maintain quorum
* Two racks in one AZ = Risky!
* Still worth having 3 nodes in same AZ to support online rolling maintenance

## Capacity Considerations
Designing for HA
* Surviving Nodes should be able to handle mission critical work - business should be able to keep running
* Try to size with some extra capacity
* Replacing dead nodes would stream from surviving nodes

Surviving Nodes
* Up nodes if a node fails
* Surviving racks in an AZ failure
* Surviving region in a region failure

## Monitoring and Manager
Current and Historial Metrics
* Reads, writes, latencies
* CQL/Alternator transaction mix - Alternator is Amazon DynamoDB Compatible API - https://www.scylladb.com/alternator/
* Utilization patterns
* Disk and CPU usage patterns
* Cluster/DC/Instance/Shard Level - zoom in/out to diagnose issues
* Can look at load over time - on/off business hours
* Look at workload mix - Alternator Vs CQL

## Monitoring (2)
* Scheduling Backup/Repair
* Try not to overlap - know if there is an off-peak window to run maintenance
* Know how long repairs run
* Know if they will impact application latency
* Tuning - repair/compaction - customers who have latency sensitive application, sub ms latency requirements
* If reduce parallelism and intensity - do they complete in reasonable time

## Manager
Ease of management
* Repairs scheduled with cron
* Broken into pieces to run separately, knows if failure and will retry chunks at the end.
* See progress/history by table.
* Easy to speed up/down - tune to throttle while application running, find optimal spot to not impact latency
* Can see progress during repair
* See final progress report, history of runs for future reference.

## Backups 
* Backups deduplicated on upload.  First time has to send everything - be aware!
* When SS tables no longer needed, will expire/delete data.
* [Tombstones](https://university.scylladb.com/courses/scylla-operations/lessons/scylla-manager-repair-and-tombstones/topic/tombstones/)

## Tuning Repairs

Actions Taken
* Schedule during off peak
* Lower impact by slowing down repairs - tune parallel and intensity
* Can reduce throughput to MB per sec - stream_io_throughput_mb_per_sec
* sctool repair update 
* Can take longer to finish repair, but less impact to live applications

## Avoid Common Pitfalls

Not designing keyspaces for availability
* Use a rack aware snitch, and NetworkTopologyStrategy to be AZ aware
* Don't use SimpleStrategy- will round robin, not rack aware
* Run repair after altering replication factor - procedure
  * The alter creates empty replica and new writes start going there
  * Repair is needed to move existing data
* Don't use RF=1.  Can get auth error if that node goes down.

New guardrails to enforce this:
```
scylla --help
--restrict-replication-simplestrategy
--minimum-replication-factor-fail-threshold
```

## Pitall 2
* Non-graceful Scylla Shutdown
* Don't just stop/kill
* Do a nodetooldrain, systemctl stop scylla-server
* Drains memtables, gracefully stops listening to clients/nodes
* Stops customer connections to that node.

## Pitfall 3
0 out of 2 replicas replied
* Using CL=LOCAL_*, data is not replicated to local DC

Solutions:
* Replicate data to local DC
* Connect client to other DC

Clocks not synchronized (NTP/Chrony):
* Fix clock synchronization between nodes

## Pitfall 4
Retry storm
* Client Timeout < Server Timeout + Retry Policy
* Query reaches client timeout, cancelled and submits a retry
* Client concurrency controls assume the server stopped working on it
* But server does not timeout that request until server timeout

Solution:
* Set server timeout < client timeout
* Utilize "USING TIMEOUT xxxms" to override server timeout within a query

## Pitfall 5 - Garbage Collection
Repairs not scheduled to complete within gc_grace_period
* Default gc_grace_period 10 days, default repair 7 days
* Ressurection risk - tombstones may be garbage collected before all replicas receive the tombstone.  Data on that node may be "resurrected"

* Ensure repair completes within gc_grace_sections to guarantee tombstone replicated to all replicas
* Alter table to utilize mode repair
* [Tombstone Article](https://www.scylladb.com/2022/06/30/preventing-data-resurrection-with-repair-based-tombstone-garbage-collection/)

Tombstone Details
* Separate writes for INSERT, UPDATE, DELETE
* If 3 writes in SS tables to delete, the tombstone says data needs to be removed.

## Q&A 

Q: How to determine sizing for ScyllaDB Clusters \
A: Cloud Calculator, can enter number instances, size/instance type, etc: 
* [Pricing](https://www.scylladb.com/product/scylla-cloud/get-pricing/)
* [Sizing Info](https://www.scylladb.com/pricing/)

Q: Incremental Backups
A: In general, backups are done via ScyllaDB Manager
* Snapshots your tables and upload to your configured object storage of choice. 
* From there, you restore it and choose a snapshot. 
* A snapshot is deduplicated from the first full backup you performed.

Q: ScyllaDB on Kubernetes, ensure graceful shutdown? \
A: Done with [ScyllaDB Operator](https://www.scylladb.com/product/scylladb-operator-kubernetes/)

## Resources and Links
* https://cloud.scylladb.com
* https://university.scylladb.com/courses/scylla-operations/
* https://www.scylladb.com/2023/10/02/introducing-database-performance-at-scale-a-free-open-source-book/
* https://lp.scylladb.com/database-performance-book-offer (this seems broken?)
* [Alternative Article](https://www.scylladb.com/2023/12/05/database-performance-at-scale-free-book-masterclass/) - [PDF Book Link](https://link.springer.com/content/pdf/10.1007/978-1-4842-9711-7.pdf?pdf=button)