# Designing Data Intensive Applications Notes

Although being one of the most important books for the software industry, as it bridges the gap between distributed systems theory and practical engineering, I struggled to find a good summarized reading notes that covers up all the key points of the book, so here it is, I hope.

## Part I: Foundation of Data Systems

### Chapter 1: Reliable, Scalable, and Maintainable Applications

CPU power is rarely the limiting factor anymore, it is the data size that is.

There are many technology options out there, and our task is to figure out the most appropriate tools and approaches for the task.

We need to ensure that the data remains correct and complete, provide good performance, and scale to handle load increase, despite any internal failures, or system degradations.

#### Reliability
***System should continue to work correctly, even in the face of faults and human errors***.

A fault is a one component of the system deviating from its specs, while failure is the when the system as a whole stops working. It's impossible to prevent faults, but we should try to prevent faults from causing failures by designing fault-tolerance mechanisms.

Hardware redundancy is the first line of defense against hardware faults. It was sufficient for a long time, but as computing demand increase, there is a move toward systems that can tolerate the loss of entire machines by using software fault tolerance as well.

The reason behind software faults is making some kind of assumptions about the environment, this assumptions are usually true, until the moment they are not. There is no quick solution to the problem, but the software can constantly check itself while running for discrepancy.

Some approaches for making reliable systems, in spite of unreliable human actions include:
- Design abstractions that are minimal and easy to achieve one thing with, but not too restrictive for people to work around them.
- Provide fully featured sandbox environments with real data for testing, without affecting real users.
- Test throughly at all levels, from unit tests, to whole system integration tests.
- Make it fast to roll back configuration changes, and provide tools to re-compute data.
- Use proper monitoring that shows early warnings signals of faults.


#### Scalability
***As system grows, there should be reasonable ways for dealing with that growth***.

The first step in scaling a system is to define the system's loads parameters (eg. requests, read to write ratio, etc.)

Throughput is usually the most important metric in batch processing systems, while *response time* is the most important metrics for online systems.

A common performance metric is percentile, where `Xth percentile = Y ms` means that `X%` of the requests will perform better than `Y ms`. It's important to optimize for a high percentile, as customers with slowest requests often have the most data (eg. purchases). However, over optimizing (eg. 99.999th) might be too expensive.

It's important to measure response times on client side against realistic traffic size.

Elastic system is useful if load is highly unpredictable, but manually scaled systems are simpler and have fewer operational surprises.

In an early stage startup, it's usually more important to be able to iterate quickly on product features than to scale to some hypothetical future load.


#### Maintainability
***Different people who works on the system should all be able to work on it productively***

The majority of the cost of the software is in the ongoing maintenance and not the initial development.

A good system should be operable, which means making routine tasks easy. This can be done by:
- Good monitoring
- Avoiding dependency on individual machines
- Good documentation
- Providing good default behavior, while giving administrators the option to override
- Self-healing, while giving administrators a manual control

A good system should be simple, this can be done by reducing complexity, which does not necessarily mean reducing its functionality, but rather by making **abstractions**.

Simple and easy to understand systems are usually easier to modify than complex ones.

A good system should be evolvable, which means making it easily adapt the changes. Agile is one of the best working patterns for maintaining evolvable systems.



### Chapter 2: Data Models and Query Languages

#### Data Models

Data models are perhaps the most important part of developing software, because they have such a profound effect on the way we think about the problems we're solving.

Relational model is the best known data model today, because of the way it hides the implementation details behind a cleaner interface. It turned out to generalize very well when computers were used for increasingly diverse purposes.

NoSQL databases have been adopted quickly and easily because:
- It provided a better scaling mechanism, including very high write throughput than relational databases
- It was free and open source
- It supported few specific query operations better than relational databases
- It provided more dynamic and expressive data model than relational databases

Relational databases will continue to be used alongside a broad variety of non-relational datastores (Polyglot Persistence).

Relational databases receives common criticism as an *awkward translation layer* is required between the application code objects, and the database model. ORM frameworks reduce the overhead but they don't eliminate it.

Relational databases deal with the **one-to-many** relationship in one of three ways:
- The common normalized way is to put the *many* values in a separate table, with foreign key reference to the *one*
- Later versions of SQL allowed multi-valued data be stored in a single row, with support for querying inside them
- The least favorable option is to store them as encoded JSON or XML, and let the application do the internal query

Document-oriented databases on the other hand supports **one-to-many** relationship natively, and provides better *locality* for the data object, thanks to the self-contained nature of JSON.

Having standardized lists for users to choose from rather than plain typing is easier for updating, better for styling consistency, better for localization support, and easier for searching. This standardized lists should be using an underlying IDs for the values, as anything that is meaningful to humans may need to change sometime in the future.

Relational databases deal with **many-to-one** relationship by referring to rows in tables by ID, as joins are easy. However, Document databases doesn't nicely support **many-to-one** relationships. Instead, the application code would need to go through the overhead of simulating the join itself, which can cache it all in memory if it's small and slow-changing. 

Relational databases overpowered older models such as *hierarchical model* and *network model* on the long run, thanks to the *query optimizer* that made it easier for relational model to add new features to the applications.

When comparing document model to relational model, arguments in favor of document model are schema flexibility, better performance, and more-matching data structure with the application, while Relational model provides better support for joins, and **many-to-one** and **many-to-many** relationships.

Document databases are not *schemaless*, but rather have *schema on read* in oppose to relational model's *schema on write*. It is not enforced by the database, but more easier to change and modify.

For document databases to benefit from *locality*, documents have to be relatively small in size.

A hybrid of relational and document models might be the future of databases, as they are becoming more similar over time.


#### Query Languages

SQL is attractive to people due to its *declarative* nature, it specifies the pattern of the resulted data instead of the way of querying it. Also, declarative code is easier to parallelize across multiple machines.

MapReduce is fairly low-level programming model for distributed execution, but it doesn't have a monopoly on distributed query execution.

Graph data model is usually the most suitable model for data with a lot of **many-to-many** relationships. There are many well-known algorithms which can operate on graphs, and some good *declarative* query languages such as Cypher for efficient querying.

Graph data model is different from network model in the way it gives much greater flexibility for applications to adapt.



### Chapter 3: Storage and Retrieval

#### Data Structures That Powers Your Database

The simplest and best efficient write operation is simply appending to a file.

Databases doesn't usually index everything by default, but require you to choose indexes manually.

Hash Index is an in-memory key-value store that maps every key to its byte offset in the data file. New records are appended to a *segment* of certain size which is being merged and compacted by a background thread, allowing for old segments to be deleted. For reliability and concurrency, only one thread is used for writing, and a snapshot of segment's hashmap is written to disk regularly.

Append-only logs allow faster sequential writes opposed to random writes, it have much simpler concurrency and crash recovery scenarios, and the merging mechanism avoids files getting fragmented over time. However, the hash table must fit in memory, and it doesn't support range queries.

Log-Structured Merge-Tree (LSM-Tree) uses a similar concept to the Hash Index, but rather using an in-memory balanced tree (eg. AVL, Red-Black) to keep records sorted. Writes are very fast as it is an append-only operations, while for reads it first checks the *memory table*, then the most recent file segment, then the next-older, and so on. If the database crashes, then the most recent writes are lost, to avoid that, every write is immediately appended to secondary disk log file.

LSM-Trees provides a segment merging mechanism that is simple and efficient, it no longer need to keep and index for all keys in memory (thanks to sorting), and it is possible to group and compress records before writing to disk. However it can be very slow when looking for keys that does not exist (A *bloom filter* might help with that).

B-Tree index is the most widely used indexing structure. It's the standard index for almost all relational databases. It breaks the database down into fixed-size pages (4 KB in size) which is the same size as the disk's page.

B-Tree is balanced, so `n` keys always have depth of `O(log n)`. It also have a branching factor of several hundreds typically.

B-Trees implement crash resilience mechanism by having an append-only log file in the disk, and handle concurrency by using latches.

Some optimizations to B-Trees include writing modified page to different location with the parent pointing to the new location, storing an abbreviation of the key to save space, and referencing sibling nodes in the leaf layer.

Advantages of LSM-Trees over B-Trees:
- Faster writes
- Higher write throughput
- Compresses better
- Lower write amplification

Disadvantages for LSM-Trees over B-Trees:
- Slower reads
- Compaction process might interfere with the overall performance
- Performance is less predictable
- Disk bandwidth can be easily consumed
- Un-merged segments might keep growing until filling up the disk space
- Same key might exist in multiple segments


In addition to *primary key index*, databases can have non-unique *secondary indexes*, that are often crucial for joins in relational databases. Both B-Trees and LSM-Trees can have *secondary indexes*.

Storing all row data within the index (*clustered index*) can allow some queries to be answered using index alone.

Multi-column indexes are helpful in querying several columns at once (eg. Geo-spatial data) as it appends several fields as one key. Standard B-Trees or LSM-Trees cannot answer such queries but they rather convert multi-dimensions into one value.

Fuzzy indexes can help in querying *similar keys* when the exact key is unknown, it's useful in document classification and machine learning.

In-memory databases are much faster but less durable and more expensive. It thus needs to write to disk asynchronously in case it had to restart. It is great for small datasets, and for modeling complex data models (eg. queues, sets) 


#### Transaction Processing or Analytics?

Businesses usually have two types databases, *online transaction processing* (OLTP) for every-day transactions, and *online analytic processing* for analytics purposes (Also known as *Data Warehouse*).

Data Warehouse's goal is to provide the same data in the *transaction processing* database, for *business analytics* and *data science engineers* to use without affecting the performance of the transaction database. This happens either through periodic data dump, or continuous stream of updates. In both cases the data is transformed first before being loaded in the warehouse.

Big advantage of having a separate warehouse is that it can be optimized for analytic access patterns, specially that typical OLTP indexing engines don't perform well in the case of analytic queries, but it's usually relational because SQL is good for analytic queries.


#### Column-Oriented Storage

Typical data warehouse tables are very wide, and typical warehouse queries only access few of them at a time. So instead of storing each row's data together, it is more efficient to store each column's data together. This can work for both relational and non-relational models.

Column-based datastores can benefit of data repetition by compression, and from CPU cycles by *vectorized processing*, but this makes writes more difficult (eg. in-place update is not possible). We can solve this by using in-memory structure same as LSM-Trees, which sorted and accumulated before it's written to disk.

A helpful technique for data warehouse is *materialized aggregates*, which caches some of the aggregated data that are used most often. In relational model, this can be created using *materialized views*.



### Chapter 4: Encoding and Evolution

Changes to the application's features also requires a change to data that it stores, this means that multiple versions of code and multiple data formats may coexist at the same time. Thus, *backward compatibility* and *forward compatibility* should be maintained.

#### Formats of Encoding Data

Moving data between local memory and network requires *encoding* and *decoding* processes.

Language-specific encoding formats can be easier in decoding, however, it's usually tied for one programming language, it exposes the code to some security vulnerabilities, and it often have bad performance, so using JSON, XML, or Binary formats are usually better.

Binary encoding can save a lot of space for huge datasets, however it might be not worth the loss of human-readability for small ones.

Some binary encoding libraries (eg. Thrift, Protocol Buffers) come with code generation tools the produces schema classes given a schema. They can be more compact than textual data formats, and always have an up-to-date documentation.


#### Models of Dataflow

The most common ways for data flow between processes are using a database, service call (REST, RPC, etc.), or async message passing.

Dataflow through database is like the process is sending a future message to itself, it requires both *backward compatibility* and *forward compatibility*. A workaround the compatibility issue can be through rewriting the whole database into new schema every time it changes, however it's an expensive thing to do for large databases.

Similar to web dataflow paradigms, a server can act as a client to another service, the concept that led to *service oriented architecture*, where services are easier to change as they are independently deployable and evolvable.

Service calls dataflow only exposes specific apis, unlike database which can be queried for any available data. But again, we should expect old and new versions of services to run at the same time.

REST is a design philosophy that emphasizes simple data formats and uses URLs for identifying resources. SOAP on the other hand is an XML-based protocol for which clients can access a remote service using local classes and methods calls.

*Remote Procedure Call* (RPC) is a dataflow model that tries to make a request to a remote service looks like a local function call. However, it is *flawed* due to the difference between network call and local call:
- Network call is unpredictable, client would have to retry failed requests for example
- Network calls can return without a result due to a time out
- Retrying might cause an action to be performed twice (unless idempotence is used)
- Network latency is wildly variable
- Calling local function with memory reference might be hard to translate to a network call

REST seems to be the predominant style for public APIs, but RPC is often used on requests between services within the same datacenter.

RPC only needs *backward compatibility* on requests, and *forward compatibility* on responses.

When service is upgraded, it usually cannot force its clients to upgrade as well, but rather maintain multiple versions of APIs.

Async message passing requires an intermediary temporary storage called message broker or message queue. It has several advantages over RPC:
- It acts as a buffer when recipient is unavailable.
- It automatically retry sending the message to prevent it from being lost
- It avoids the sender the need to know the recipient's IP and port number
- One message can be sent to multiple recipients
- It decouples the sender from the receiver

One downside is that is is one-way communication, so for the recipient to reply it might have to use another channel.

Message brokers usually have one or more set of topics, and when a message is sent to a specific topic, the broker pass it all topic's subscribers. This creates a freedom for processes to publish messages to another topics upon receiving a message.



## Part II: Distributed Data

### Chapter 5: Replication

If the data doesn't change, replication would be as simple as copying it once to other replicas, but the difficulty lies in handling changes if it does, because we usually have to deal with many trade-offs such as sync vs async replication, and how to handle failed replicas.

#### Leaders and Followers

*Leader-based* replication allows writes to only go through the leader, which then sends it to all other *follower replicas*. The clients then can query any replica including the leader. Although its limitation, it's one of the most used replication algorithms in databases and message brokers.

Synchronous replication guarantees the followers to have and up-to-date copy of the data, but if one follower didn't respond, the write cannot proceed, that's why in practice only one of few other replicas are synchronous, while other are async. Even that sync replication normally is quite fast, there is not guarantee about its latency, that's why async replication is widely used.

To add a new follower to a *leader-based* system, the leader has to take consistent snapshots of its database, the latest snapshot is then copied to the new follower, and when the follower connects to the leader, it requests all the changes that happened after the snapshot. It becomes ready afterwards.

In case of a follower failure (which is usually detected through *timeout*), the follower has to keep a *log* of its data changes, and request all data changes happened during its disconnection form the leader after it wakes up. However, in case of leader failure, one of the followers has to elected as a new leader, and other followers has to sync data changes with it instead.

Downsides for automated leader-failure recovery are:
- The new leader may not have received all updates from older leader, discarding these updates is very dangerous.
- Old leader might wake up thinking it is the leader, resulting in *split brain* (two nodes believe they are the leader)

Writes that are sent to followers can take many forms, some are:
- **Statement based replication**, where the SQL statement is sent as is, however environment related variables (eg. randomness, time, etc.) will cause inconsistency, it can be replaced by static values from the leader, but it's generally not preferred
- **Write-ahead log**, where followers receives append-only logs, but this makes replication closely coupled to the storage engine
- **Logical log replication**, where different log formats are used to allow the log to decouple from the engine's internals. It also allows *backward compatibility*
- **Trigger-based replication**, it gives much flexibility for the application code to determine what to replicate. It's useful when flexibility is needed, but it's prone to more bugs and limitations, as well as greater overhead


#### Problems with Replication Lag

*Leader-based* replication suits workloads with mostly reads and small percentage of writes, but realistically it has to be asynchronous.

**Read-after-write consistency** guarantees that the users always see any updates they submit themselves immediately, this is done through restricting the read of *self-written* data only from the leader. This can be known from the most recent write *logical timestamp*. The downsides are that any request of that kind must be routed to the leader's datacenter, and that user might be using multiple devices at once.

**Monotonic reads consistency** guarantees that users don't see newer writes before old ones, for this to happen, the client must always make its reads from the same replica. However, if the replica fails, rerouting must take place.

**Consistent prefix reads** guarantees that any sequence of writes that happens in a certain order, must be read in the same order. This happens by making sure casually related writes are written to the same partition, and are written in the same order.

When using *eventual consistency*, if we cannot live with a lag that increases to several minutes, a stronger guarantee should be used.


#### Multi-Leader Replication

When a process is replicated across many datacenters, we can have a leader in *each* datacenter, this provides better performance due to hidden network delays, as well as failure tolerance as each datacenter can continue operating independently, and also better tolerance for network problems due to asynchronous replication. However, data may be concurrently modified in two different datacenters, so those write conflicts must be resolved.

Some retrofitted features in databases such as auto-incrementing keys, triggers, and integrity constrains are problematic, that's why *multi-leader* replication is considered dangerous and should be avoided if possible.

Since handling conflicts can be very tricky, avoiding them is the recommended approach. However when resolution is needed, some ways would be to give each write a unique ID and the highest ID wins, give each replica a unique ID and the higher takes precedence, merge conflicting values together, or record the conflicts in an explicit data structure.

*Multi-leader replication* comes in various topologies, the most common are *all-to-all*, *circular*, and *star* topologies. A problem with circular and star topologies is that they have a single point of failure. All-to-all topology doesn't have this problem, but it may have problems with causality as some network links might be faster than others.


#### Leaderless Replication

When our system is *write-intensive*, leader(s) may act as a bottleneck, that's when *leader-less replication* comes in handy, also known as *Dynamo-style*. The clients send their writes and reads to several replicas in parallel.

Reading the same key from different replicas can help detect inconsistency, so the client can re-write inconsistent field with the majority's value. Another mechanism is to have a background process that constantly check for differences and amend them asynchronously.

The numbers of replicas to read from (***r***), and to write to (***w***) should be configured to follow the *quorums* formula: ***w + r > n***, where (***n***) is the number of replicas. Smaller values of ***w*** or ***r*** results in more *stale* read values, but provides lower latency and higher availability.

When two writes occur concurrently, with no way of finding which happened first, the only safe solution is to merge the writes.

*Dynamo-style* is usually optimized for *eventual consistency* use cases, more stronger guarantees require transactions or consensus.

Leaderless replication is suitable for multi-datacenter operation.

The problem with *eventual consistency* is that if each node simply overwrote the value of the key whenever it receives it, the nodes would become permanently inconsistent, some solutions include:
- **Last write wins**, as long as we have a way of determining which write was the recent, replicas would end-up being consistent, this solution achieves convergence but at the cost of durability
- **Happens-before**, is a mechanism of detecting whether two operations are concurrent, only then, a conflict needs resolution
- **Merge values**, it guarantees the we don't lose any of the conflicting data, and then it's up to the application how to show them
- **Version vectors (version clock)**, which is assigning a version number per replica, and each replica has to increment it's number after every write, if an array of all version numbers is passed alongside the operations, it would be easy to detect inconsistency


### Chapter 6: Partitioning

For huge datasets that doesn't fit in a single database, or for scalability purposes, partitioning (sharding) is the solution. Typically each piece of data belongs to exactly one partition, and a single operation might need to touch multiple partitions at once. Thus, complex queries should be parallelized across many nodes.

#### Partitioning and Replication

Replication and partitioning usually works together hand-in-hand, so partitions are usually stored on several nodes for fault-tolerance. However, the choice of partitioning and replication schemes are mostly independent.

#### Partitioning of Key-Value Data

The goal of partitioning is to spread data evenly across nodes, and more importantly to avoid having *skewed* partitions with most of the load (resulting in *hot spots*).

There are two main ways of partitioning keys:
- **Partitioning by key range**, that is by sorting the keys we have, and assigning boundaries between ranges. This is suitable for range queries, but requires continuous boundaries adaptation, with the risk of still having *hot-spots*
- **Partitioning by hash key**, that is by using a non-cryptographic hash function (eg. MD5), and partition using range of hashes instead of keys. This provides more randomization, and also can be used in partitioning by *compound primary key* which enables *one-to-many* relationships, but we lose the range queries ability.

One problem still is when most reads and writes are for the same key (eg. celebrity account ID), it is usually left for the application to handle this skew, typically by assigning random bytes at the beginning/end of this key to scatter it across all the replicas, however, this requires extra bookkeeping, as well as requests to all the replicas when reading.


#### Partitioning of Secondary Indexes

Secondary indexes are very important to all relational databases, and some document databases as well. However, they don't map neatly to partitions.

One option is to add an extra secondary index inside every partition, this index would cover-up only the keys in the partition. However, if the client needs to find all fields with a common secondary field, it has to query all partitions.

Another option is to have a global secondary index which can also be partitioned, but using *term* instead of *document*. Every partition would keep a secondary index of some of these terms, this makes reads more efficient, but writes are slower and complicated. However, in practice updates to global secondary indexes are asynchronous and very fast.


#### Rebalancing Partitions

Over time, data has to be moved from one node to another, this *rebalancing* process is usually expected to meet some minimum requirements:
- After rebalancing, the load should be shared fairly between nodes
- While rebalancing, the database should continue accepting reads and writes
- Data shouldn't be moved between nodes more than necessary

A top of mind strategy for rebalancing might be using `mod` operation for the hash of the key, but this turns out to be very bad because if the number of nodes changes, most keys will need to be moved.

A good alternative is to have a **fixed number of partitions**, by creating many more partitions than there are nodes, and assign several partitions to each node, and when a node is added to the cluster, it steals some partitions as are from older nodes. This can allow us to assign more partitions to nodes that are more powerful, but since the number is fixed, partitions can get very large, thus making rebalancing and recovery from failures very expensive, and if they are too small, it is too much overhead.

Another alternative is **dynamic partitioning**, which acts similarly to the top level of a B-tree, when partitions exceeds a configured size, it's split into two partitions, and when shrinks when goes below another configured size, then partitions can be moved to different nodes for rebalancing the load. The big advantage is that the number of partitions adapts to the total volume.

Another option is **Partitioning proportionally to nodes**, which is to have a fixed number of partitions per node. When a new node joins, it picks fixed number of random partitions to split and take half of them. However, the randomization might produce unfair splits.

Having a completely automated rebalancing process can be very unpredictable, so it's good to have a human in the loop for rebalancing.


#### Request Routing

Similar to all network systems, *service discovery* problem needs to be approached, for which we need a mechanism of mapping keys to partition and their hosting nodes. This mechanism can be placed inside the nodes, routing tier (load balancer), or clients themselves.

Many distributed systems rely on a coordination service such as **Zoo-Keeper** for solving this problem, that is by keeping track of cluster's metadata including partitioning.

### Chapter 7: Transactions

A transaction is a way of an application to group several reads and writes together into a logical unit, which either succeeds or fails entirely.

#### The Slippery Concept of a Transaction

There are wrong misconceptions that any large system should abandon transactions in order to maintain good performance and high availability, and that transactions are essential to database vendors, but these viewpoints are pure hyperbole.

Transactions are usually associated with the ACID properties:
- **Atomicity**: something that cannot be broken down into smaller parts, so when a client wants to make several writes, but a fault occurs, the whole transaction aborts and the client can safely retry
- **Consistency**: that the database is being in a "good-state", which is not something the application should guarantee, not the database
- **Isolation**: means that concurrently executing transactions are isolated from each other
- **Durability**: the promise that once a transaction has committed, its data will not be forgotten, for single node, this only means writing to a hard drive, but for replicated databases, it requires copying the data to some number of nodes

Storage engines almost universally aim to provide *atomicity* and *isolation* on the level of single object. Atomicity can be implemented using log for crash recovery, and *isolation* using a lock on each object.

Multi-object transactions might be needed when a foreign key references to a row in another table, or when de-normalized information and secondary indexes needs to be updated. However, many distributed datastores abandoned multi-object transactions because they are difficult to implement across partitions.

Leaderless replicated datastores won't undo something it has already done, so it's the application's responsibility to recover from errors.

Retrying aborted transactions isn't prefect because if the network failed while the server is trying to acknowledge the successful commit, the transaction might get performed twice, or when the error is permanent (eg. constraint violation), retrying would be pointless. 

#### Weak Isolation Levels

Concurrency bugs are hard to find by testing, for that, databases have long tried to hide it by providing *transaction isolation*, especially *serializable isolation*. However, due to its performance cost, systems use weaker levels of isolations more commonly, which protect against *some* concurrency issues, but not all.

Many popular relational databases that are considered "ACID" use weak isolation themselves.

The most basic level of transaction isolation is **read committed**, which simply just prevents dirty reads and dirty writes.

**Dirty reads** is when a transaction can read another transaction's write that has not been committed yet, this is useful because if the transaction needs to update several objects, other transactions might see some updates but not others, and if the transaction aborts, any writes would need to rollback. To prevent dirty reads, a lock might be used, however, this harms the response time, so a better approach is to remember both the old and new values, and give the old value to any read request before commitment.

**Dirty writes** is when a transaction write overwrites another transaction's uncommitted write, this is useful because if the transaction updates multiple objects, dirty writes can lead to bad outcome, however, preventing it still doesn't prevent some other race conditions. Most databases prevents dirty writes by using row-level locks.

Read committed isolation does not protect against **read skew**, where a transaction reads different parts from the database in different points of time, this can be cause issues for situations like backups, analytic queries, or integrity checks.

The solution to read skews is **snapshot isolation**, where each transaction reads from a consistent snapshot of the database, as if it was frozen at a particular point in time. It is usually implemented using write locks, but readers doesn't require any locks.

The difference between snapshot isolation reads and read committed, is that snapshot isolation keeps *several* versions of an object instead of just two.

Indexing might sound like a problem for snapshot isolation, but one solution is to have the index point to *all* versions of an object, while another approach is to use an append-only/copy-on-write variant that doesn't overwrite pages in the underlying tree.

A common pattern in databases is *read-modify-write*, which might lead to **lost update** problem. This problem occur when two concurrent transactions perform *read-modify-write* cycle, and one of the updates  was overridden and lost.

There are variety of solutions that has been developed for solving the lost update problem:
- Atomic writes, which is usually the best solution if the code can be expressed in terms of operations, that is like `UPDATE counters SET value = value + 1 WHERE ...`. Unfortunately not all writes can be expressed this way, and it's easy with ORMs to miss things up
- Explicit locking, where the application explicitly locks objects that are being updated: `SELECT * FROM ... WHERE ... FOR UPDATE`
- Automatically deleting lost updates, where the database allows the transaction to execute in parallel, and detects a lost update if happened, abort the transaction and force it to retry. An advantage to this is that the database performs the check efficiently in conjunction with snapshot isolation, and the detection happens automatically and is thus less error-prone
- Compare and set, where only to allow the update to happen only if the value has not changed since last read: `UPDATE ... SET ... WHERE id = ... AND content = "same old content"`. However this might fail if the database allows reading from old snapshots.
- Conflict resolution, as in replicated databases, techniques based on locks and compare-set doesn't apply, so one approach is to allow concurrent writes to create several conflicting versions, and let the application code to resolve (using *last write wins*) or merge them

Two other race conditions that can still happen are write skew and phantom reads.

**Write skew** is when two transactions are updating two different objects, yet their result is different than performing them serially. This can be fixed only through serializable isolation by configuring some constraints with triggers or materialized views as they involve multiple objects, or by explicitly lock the rows that the transaction depends on using `FOR UPDATE`.

**Phantom reads** occurs when, in the course of a transaction, new rows are added or removed by another transaction to the records being read, and there is no way to put locks on rows that might not be existing yet. This can be solved using *materialized conflicts* which is more like an artificial lock to the database. But, serializable isolation is always preferred over this approach.


#### Serializability

Isolation levels are hard to understand, you cannot tell if an application code is safe just by looking, and there are no good tools to help with that. So, the simple solution is just to use *serializable isolation*, which is the strongest isolation level and prevents *all* race conditions.

*Serializable isolation* can be implemented through different techniques, actual serial execution, two phase locking, and optimistic concurrency.

The easiest is **actual serial execution**, which to actually execute only one transaction at a time on a single thread. This approach wasn't feasible until recently, as RAM become more cheaper, and database designers realized OLTP transactions usually makes a small number of reads and writes (in contrast with log-running analytical queries that should use snapshot isolation). Single thread execution can sometimes perform better than concurrent systems, however, the throughput is limited to single CPU.

To enhance the performance of serial executions, the application must submit the entire transaction code ahead of time as a *stored procedure*, which makes the performance reasonable, especially for databases with general-purpose programming languages. Partitioning can still be used with serial executions, especially when most transactions only uses one partition.

**Two Phase Locking** is similar to dirty writes, but with more stronger requirements on the lock, where two modes of locks are provided, *shared mode* for readers, and *exclusive mode* for writes. Deadlocks might result from multiple transaction holding locks, then the database automatically detects deadlocks and aborts one of the transactions.

The big downside of two-phase locking is the performance, which is much worse compared to weak isolation, also it can have unstable latencies, and can be very slow at high percentiles.

**Serializable Snapshot Isolation (SSI)** is an optimistic concurrency control technique that is fairly new but fast enough becoming the new default in the future. Instead of blocking if something potentially dangerous happens, transactions continue anyways, and the database checks for conflicts only when transaction commits, this reduces the number of unnecessary aborts. It perform badly if there is high contention, but this contention can be reduced with commutative atomic operations. Also, same as two phase locking, SSI can be partitioned and distributed which doesn't tie the performance to a single CPU core.
