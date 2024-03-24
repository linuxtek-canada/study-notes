# Expert Panel Discussion - Q&A

## Articles
* [User Stories](https://resources.scylladb.com/scylladb-user-stories)
* [Workload Prioritization](https://university.scylladb.com/courses/scylla-operations/lessons/workload-prioritization/)

## Q&A

Q:  
* Would like to hear a bit about running ScyllaDB on Kubernetes. 
* Someone linked the ScyllaDB operator document. 
* What is the status for this on EKS? It's listed as experimental.
* More info on the Scylla Operator and how it manages the ScyallDB cluster would be great

A:  Customers running it in production on EKS - look for a [talk on this](https://www.scylladb.com/presentations/scylladb-kubernetes-operator-goes-global/).

Q:  Tiered storage ETA? \
A:  No ETA, working for support for S3 to store slower tier on object storage.  
* [Check newsletter on forums](https://forum.scylladb.com/t/last-week-in-scylladb-git-master-issue-222-2024-03-17/1387)
* Some disks in AWS instance tiers use hard disks vs NVME, so they are slower.  
* ScyllaDB meant to work with faster storage.
* [Article about using S3](https://www.scylladb.com/2024/03/04/avi-kivity-and-dor-laor-chat/)

Q: How to handle upgrades/maintenance for ScyllaDB clusters with less downtime? \
A: Operations on ScyllaDB done in rolling manner.  Upgrade node, restart, etc.

Q:  Emre Besirik - What can you tell us about ScyllaDB's ability to resist data corruption, ability to recover, backup and return to point in time capable? And general disaster recover capabilities? \
A:  
* Data is replicated across nodes. Tradeoff between availability, cost, performance.
* [Article - OVH Datacenter Fire](https://www.scylladb.com/2021/03/23/kiwi-com-nonstop-operations-with-scylla-even-through-the-ovhcloud-fire/)

Q: Any concerns when upgrading about having a portion of the ScyllaDB nodes running on different versions? Are they recommended to stay within a certain version to avoid drift? \
A: Follow supported upgrade path.  Upgrade in sequence.
* Upgrading between larger versions not tested.  Stay within one major release. 
* Individual upgrades 5.0 -> 5.1 -> 5.2 -> 5.3
* [Reference Article](https://opensource.docs.scylladb.com/stable/upgrade/)

Q: Logging Options - Performance? \
A: Idle disk/cpu can be used for logging systems.  Real time analytics on workloads
* [Article](https://www.scylladb.com/presentations/real-time-or-analytics-workloads-why-not-both/)
* Requirement for high performance, throughput, consistent low latency - ScyllaDB
* Does have logging options for different verbosity.
* Standard syslog, configurable, can use remote syslog to forward logging
* More specialized logging - audit logging in Enterprise version - data definition language, can log modifications (generates too many logs)

Q: Can you talk a bit about some of the memory caching systems that can be used with ScyllaDB, and the pros/cons? \
A: Don't need any external cache, don't recommend placing a cache in front of DB.  Adds complexity, latency.
* ScyllaDB manages how hot data is made available
* Customers have replaced various caches with ScyllaDB
* [Article - Internal Cache](https://www.scylladb.com/2024/01/08/inside-scylladbs-internal-cache/)
* [Article - Replacing Cache Registration](https://lp.scylladb.com/wbn-replacing-your-cache-registration)
* [Article - 7 Reasons Not to Put a Cache in Front of Your Database](https://www.scylladb.com/2017/07/31/database-caches-not-good/)

Q: What are some ideal use cases for the ALLOW FILTERING keyword? \
A: If has high selectivity can be good to use.  If low selectivity, bad performance. 
* Good for analytics.  Design schema in a way that queries don't require filtering.
* If you optimize for OTP, other things could be neglected.  Other queries might not be as natural.
* Without filtering would be less efficient, but analytics aren't as latency sensitive.  
* Scans are not inefficient, don't require indexes.
* Analytics can process a lot of data, ALLOW FILTERING can remove the data at the DB level rather than at the client layer
* Less efficient to pull the data across network and filter at client level.

Q: If you got the modeling for a scylla table wrong and data is now in it, is there any way to re-model in production? \
A: Recommended to create a new table and migrate the data.
* One reason data modelling so important.  Think about queries from query driven approach.
* Limited support for altering a database.
* Can't change primary keys, need to get data model correct.
* Ways to recover, but require a lot of work.  Create new tables, populate them in parallel with old tables, then move workload over.
* If can afford downtime, go down, migrate data, restart application - don't like to do this.
* [Article - Data Modeling](https://university.scylladb.com/courses/data-modeling/)

Q: What integrations and architectures do you recommend for search capability? Ingesting patterns, etc. \
A: Depends on use case. 
* Could write to ScyllaDB then go to Elasticsearch.
* Could put messages into Kafka Topic, then go to ScyllaDB and Elasticsearch in parallel.
* If low latency and ingest lots of writes, and search capabilty consistency - would want to place ScyllaDB in front of writers.

Q: Security Features? \
A: Compliance, Encryption,
* Compliance - SOC2 for ScyllaDB Cloud, ESOC Compliant.
* Encryption - Can encrypt data in transit (passing between nodes) and at rest.
* Support own keys with Scylla Cloud, using Key Servers - don't have to store keys on server itself
* Amazon KMS or equivalent, Scylla can retrieve, encrypt/decrypt data, won't be stored on disks themselves
* RBAC - Prevent users from accessing data they shouldn't, limit according to roles
* Audit
* [Certifications Page](https://www.scylladb.com/trust-center/)

Q: Are there any considerations or trade-offs to keep in mind when dealing with eventual consistency in analytical use cases? \
A: Depends on use case:
* May not be good fit for banking, where the balances HAVE to be up to date everywhere.
* Can guarantee data is absolutely up to date using QUORUM to get last data wrote.
* Do support limited asset with lightweight transactions.  On single partitions.
* Low consistency levels could get outdated information, but sometimes this is ok (ie. Hits to webpages)
* Usually start with Quorum read/write, and can go to more or less consistency if they don't need the guarantees and value performance.
* [Article - Lightweight Transactions](https://university.scylladb.com/courses/data-modeling/lessons/lightweight-transactions/)

Q: Please speak on 6.0 ETA. We're getting anxious \
A: Almost feature complete, lots working on it.

* ScyllaDB In Action book - By Bo Ingram (Discord) - Get a power user’s perspective on everything you need to know about ScyllaDB, from your very first queries to running it in a production environment 
* [Book Offer Link](http://lp.scylladb.com/scylladb-in-action-book-offer)