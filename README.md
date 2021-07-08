# Designing Data Intensive Applications Notes

Although being one of the most important books for the software industry, as it bridges the gap between distributed systems theory and practical engineering, I struggled to find a good summarized reading notes that covers up all the key points of the book, so here it is, I hope.

This reading notes are biased towards *what to do* rather than *how it works*. The main goal behind it is to be a quick one page look-up for people wishing to remember some of details on the fly, or for someone who wish to recap the highlights of the whole book in less than an hour. However, a fair amount of *how it works* explanation details are included.

The book is available for purchase [here](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/).

## Table of Content
- [Part I: Foundation of Data Systems](#p1)
	- [Chapter 1: Reliable, Scalable, and Maintainable Applications](#ch1)
	- [Chapter 2: Data Models and Query Languages](#ch2)
	- [Chapter 3: Storage and Retrieval](#ch3)
	- [Chapter 4: Encoding and Evolution](#ch4)
- [Part II: Distributed Data](#p2)
	- [Chapter 5: Replication](#ch5)
	- [Chapter 6: Partitioning](#ch6)
	- [Chapter 7: Transactions](#ch7)
	- [Chapter 8: The Trouble with Distributed Systems](#ch8)
	- [Chapter 9: Consistency and Consensus](#ch9)
- [Part III: Derived Data](#p3)
	- [Chapter 10: Batch Processing](#ch10)
	- [Chapter 11: Stream Processing](#ch11)
	- [Chapter 12: The Future of Data Systems](#ch12)
- [More Reading Notes](#more)

## <a name="p1">Part I: Foundation of Data Systems</a>

### <a name="ch1">Chapter 1: Reliable, Scalable, and Maintainable Applications</a>

CPU power is rarely the limiting factor anymore, it is the data size that is.

There are many technology options out there, and our task is to figure out the most appropriate tools and approaches for the task; that is to ensure that the data remains correct and complete, provide good performance, and scale to handle load increase, despite any internal failures, or system degradations.

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

Throughput is the most important metric in batch processing systems, while *response time* is the most important metrics for online systems.

A common performance metric is percentile, where `Xth percentile = Y ms` means that `X%` of the requests will perform better than `Y ms`. It's important to optimize for a high percentile, as customers with slowest requests often have the most data (eg. purchases). However, over optimizing (eg. 99.999th) might be too expensive.

It's important to measure response times on client side against realistic traffic size.

Elastic system is useful if load is highly unpredictable, but manually scaled systems are simpler and have fewer operational surprises.

In an early stage startup, it's more important to be able to iterate quickly on product features than to scale to some hypothetical future load.

#### Maintainability

***Different people who works on the system should all be able to work on it productively***

The majority of the cost of the software is in the ongoing maintenance and not the initial development.

A good system should be operable, which means making routine tasks easy. This can be done by:
- Good monitoring
- Avoiding dependency on individual machines
- Good documentation
- Providing good default behavior, while giving administrators the option to override
- Self-healing, while giving administrators a manual control

A good system should be simple, this can be done by reducing complexity, which doesn't necessarily mean reducing its functionality, but rather by making **abstractions**.

Simple and easy to understand systems are usually easier to modify than complex ones.

A good system should be evolvable, which means making it easily adapt the changes. Agile is one of the best working patterns for maintaining evolvable systems.

### <a name="ch2">Chapter 2: Data Models and Query Languages</a>

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

### <a name="ch3">Chapter 3: Storage and Retrieval</a>

#### Data Structures That Powers Your Database

The simplest and best efficient write operation is simply appending to a file.

Databases doesn't usually index everything by default, but require us to choose indexes manually.

Hash Index is an in-memory key-value store that maps every key to its byte offset in the data file. New records are appended to a *segment* of certain size which is being merged and compacted by a background thread, allowing for old segments to be deleted. For reliability and concurrency, only one thread is used for writing, and a snapshot of segment's hash map is written to disk regularly.

Append-only logs allow faster sequential writes opposed to random writes, it have much simpler concurrency and crash recovery scenarios, and the merging mechanism avoids files getting fragmented over time. However, the hash table must fit in memory, and it doesn't support range queries.

Log-Structured Merge-Tree (LSM-Tree) uses a similar concept to the Hash Index, but rather using an in-memory balanced tree (eg. AVL, Red-Black) to keep records sorted. Writes are very fast as it is an append-only operations, while for reads it first checks the *memory table*, then the most recent file segment, then the next-older, and so on. If the database crashes, then the most recent writes are lost, to avoid that, every write is immediately appended to secondary disk log file.

LSM-Trees provides a segment merging mechanism that is simple and efficient, it no longer need to keep and index for all keys in memory (thanks to sorting), and it is possible to group and compress records before writing to disk. However it can be very slow when looking for keys that doesn't exist (A *bloom filter* might help with that).

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

### <a name="ch4">Chapter 4: Encoding and Evolution</a>

Changes to the application's features also requires a change to data that it stores, this means that multiple versions of code and multiple data formats may coexist at the same time. Thus, *backward compatibility* and *forward compatibility* should be maintained.

#### Formats of Encoding Data

Moving data between local memory and network requires *encoding* and *decoding* processes.

Language-specific encoding formats can be easier in decoding, however, it's usually tied for one programming language, it exposes the code to some security vulnerabilities, and it often have bad performance, so using JSON, XML, or Binary formats are usually better.

Binary encoding can save a lot of space for huge datasets, however it might be not worth the loss of human-readability for small ones.

Some binary encoding libraries (eg. Thrift, Protocol Buffers) come with code generation tools the produces schema classes given a schema. They can be more compact than textual data formats, and always have an up-to-date documentation.

#### Models of Dataflow

The most common ways for dataflow between processes are using a database, service call (REST, RPC, etc.), or async message passing.

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

## <a name="p2">Part II: Distributed Data</a>

### <a name="ch5">Chapter 5: Replication</a>

If the data doesn't change, replication would be as simple as copying it once to other replicas, but the difficulty lies in handling changes if it does, because we usually have to deal with many trade-offs such as sync vs async replication, and how to handle failed replicas.

#### Leaders and Followers

*Leader-based* replication allows writes to only go through the leader, which then sends it to all other *followers*. The clients then can query any replica including the leader. Although its limitation, it's one of the most used replication algorithms in databases and message brokers.

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

### <a name="ch6">Chapter 6: Partitioning</a>

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

Secondary indexes are important to all relational databases, and also some document databases. However, they don't map neatly to partitions.

One option is to add an extra secondary index inside every partition, this index would cover-up only the keys in the partition. However, if the client needs to find all fields with a common secondary field, it has to query all partitions.

Another option is to have a global secondary index which can also be partitioned, but using *term* instead of *document*. Every partition would keep a secondary index of some of these terms, this makes reads more efficient, but writes are slower and complicated. However, in practice updates to global secondary indexes are asynchronous and very fast.

#### Rebalancing Partitions

Over time, data has to be moved from one node to another, this *re-balancing* process is expected to meet some minimum requirements:
- After re-balancing, the load should be shared fairly between nodes
- While re-balancing, the database should continue accepting reads and writes
- Data shouldn't be moved between nodes more than necessary

A top of mind strategy for re-balancing might be using `mod` operation for the hash of the key, but this turns out to be very bad because if the number of nodes changes, most keys will need to be moved.

A good alternative is to have a **fixed number of partitions**, by creating many more partitions than there are nodes, and assign several partitions to each node, and when a node is added to the cluster, it steals some partitions as are from older nodes. This can allow us to assign more partitions to nodes that are more powerful, but since the number is fixed, partitions can get very large, thus making re-balancing and recovery from failures very expensive, and if they are too small, it is too much overhead.

Another alternative is **dynamic partitioning**, which acts similarly to the top level of a B-tree, when partitions exceeds a configured size, it's split into two partitions, and when shrinks when goes below another configured size, then partitions can be moved to different nodes for re-balancing the load. The big advantage is that the number of partitions adapts to the total volume.

Another option is **Partitioning proportionally to nodes**, which is to have a fixed number of partitions per node. When a new node joins, it picks fixed number of random partitions to split and take half of them. However, the randomization might produce unfair splits.

Having a completely automated re-balancing process can be very unpredictable, so it's good to have a human in the loop for re-balancing.

#### Request Routing

Similar to all network systems, *service discovery* problem needs to be approached, for which we need a mechanism of mapping keys to partition and their hosting nodes. This mechanism can be placed inside the nodes, routing tier (load balancer), or clients themselves.

Many distributed systems rely on a coordination service such as **Zoo-Keeper** for solving this problem, that is by keeping track of cluster's metadata including partitioning.

### <a name="ch7">Chapter 7: Transactions</a>

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

**Dirty reads** is when a transaction can read another transaction's write that hasn't been committed yet, this is useful because if the transaction needs to update several objects, other transactions might see some updates but not others, and if the transaction aborts, any writes would need to rollback. To prevent dirty reads, a lock might be used, however, this harms the response time, so a better approach is to remember both the old and new values, and give the old value to any read request before commitment.

**Dirty writes** is when a transaction write overwrites another transaction's uncommitted write, this is useful because if the transaction updates multiple objects, dirty writes can lead to bad outcome, however, preventing it still doesn't prevent some other race conditions. Most databases prevents dirty writes by using row-level locks.

Read committed isolation doesn't protect against **read skew**, where a transaction reads different parts from the database in different points of time, this can be cause issues for situations like backups, analytic queries, or integrity checks.

The solution to read skews is **snapshot isolation**, where each transaction reads from a consistent snapshot of the database, as if it was frozen at a particular point in time. It is usually implemented using write locks, but readers doesn't require any locks.

Snapshot isolation reads differ from read committed in that it keeps *several* versions of an object instead of just two.

Indexing might sound like a problem for snapshot isolation, but one solution is to have the index point to *all* versions of an object, while another approach is to use an append-only/copy-on-write variant that doesn't overwrite pages in the underlying tree.

A common pattern in databases is *read-modify-write*, which might lead to **lost update** problem. This problem occur when two concurrent transactions perform *read-modify-write* cycle, and one of the updates  was overridden and lost.

There are variety of solutions that has been developed for solving the lost update problem:
- Atomic writes, which is usually the best solution if the code can be expressed in terms of operations, that is like `UPDATE counters SET value = value + 1 WHERE ...`. Unfortunately not all writes can be expressed this way, and it's easy with ORMs to miss things up
- Explicit locking, where the application explicitly locks objects that are being updated: `SELECT * FROM ... WHERE ... FOR UPDATE`
- Automatically deleting lost updates, where the database allows the transaction to execute in parallel, and detects a lost update if happened, abort the transaction and force it to retry. An advantage to this is that the database performs the check efficiently in conjunction with snapshot isolation, and the detection happens automatically and is thus less error-prone
- Compare and set, where only to allow the update to happen only if the value hasn't changed since last read: `UPDATE ... SET ... WHERE id = ... AND content = "same old content"`. However this might fail if the database allows reading from old snapshots.
- Conflict resolution, as in replicated databases, techniques based on locks and compare-set doesn't apply, so one approach is to allow concurrent writes to create several conflicting versions, and let the application code to resolve (using *last write wins*) or merge them

Two other race conditions that can still happen are write skew and phantom reads.

**Write skew** is when two transactions are updating two different objects, yet their result is different than performing them serially. This can be fixed only through serializable isolation by configuring some constraints with triggers or materialized views as they involve multiple objects, or by explicitly lock the rows that the transaction depends on using `FOR UPDATE`.

**Phantom reads** occurs when, in the course of a transaction, new rows are added or removed by another transaction to the records being read, and there is no way to put locks on rows that might not be existing yet. This can be solved using *materialized conflicts* which is more like an artificial lock to the database. But, serializable isolation is always preferred over this approach.

#### Serializability

Isolation levels are hard to understand, we cannot tell if an application code is safe just by looking, and there are no good tools to help with that. So, the simple solution is just to use *serializable isolation*, which is the strongest isolation level and prevents *all* race conditions.

*Serializable isolation* can be implemented through different techniques, actual serial execution, two phase locking, and optimistic concurrency.

The easiest is **actual serial execution**, which to actually execute only one transaction at a time on a single thread. This approach wasn't feasible until recently, as RAM become more cheaper, and database designers realized OLTP transactions usually makes a small number of reads and writes (in contrast with log-running analytical queries that should use snapshot isolation). Single thread execution can sometimes perform better than concurrent systems, however, the throughput is limited to single CPU.

To enhance the performance of serial executions, the application must submit the entire transaction code ahead of time as a *stored procedure*, which makes the performance reasonable, especially for databases with general-purpose programming languages. Partitioning can still be used with serial executions, especially when most transactions only uses one partition.

**Two Phase Locking** is similar to dirty writes, but with more stronger requirements on the lock, where two modes of locks are provided, *shared mode* for readers, and *exclusive mode* for writes. Deadlocks might result from multiple transaction holding locks, then the database automatically detects deadlocks and aborts one of the transactions.

The big downside of two-phase locking is the performance, which is much worse compared to weak isolation, also it can have unstable latencies, and can be very slow at high percentiles.

**Serializable Snapshot Isolation (SSI)** is an optimistic concurrency control technique that is fairly new but fast enough becoming the new default in the future. Instead of blocking if something potentially dangerous happens, transactions continue anyways, and the database checks for conflicts only when transaction commits, this reduces the number of unnecessary aborts. It perform badly if there is high contention, but this contention can be reduced with commutative atomic operations. Also, same as two phase locking, SSI can be partitioned and distributed which doesn't tie the performance to a single CPU core.

### <a name="ch8">Chapter 8: The Trouble with Distributed Systems</a>

The main difference between a single computer and a distributed system, is that in distributed systems there are lots of ways for things to go wrong, and we should assume that it *will* go wrong.

#### Faults and Partial Failures

A program that runs on a single computer is *deterministic*, it usually either fully function or entirely break, while in a distributed system, some parts can break in some unpredictable *nondeterministic* way, even though other parts of the system are working fine.

If we want to make distributed systems work, we must accept the possibility of partial failure and build fault-tolerance mechanisms into the software. This is achieved by knowing what behavior to expect from the software in the case of fault, consider wide range of possible faults, and artificially create such situations in our testing environment to see what happens.

#### Unreliable Networks

*Shared-nothing* distributed system are becoming the dominant approach for building internet services, because it make use of commoditized cloud computing services, and can achieve high reliability through redundancy. However, the network is its only way of communication.

The *asynchronous packet networks* have a wide variety of downsides, as the packet might get lost or queued for a long time, and the receiver node may fail or pause temporarily. The usual way of handling this is using **timeouts**, for which it isn't necessarily to tolerate the fault after, but showing an error message can be also a valid approach.

Many systems need to automatically detect faulty nodes, such as load balancers to stop sending requests to a dead node, or a when a leader fails in a single-leader replication. We might be able to get some feedback from the network protocols such as RST or FIN packets, or configure the machine's operating system to start a script when the process crashes, but these approaches doesn't gives strong guarantees as compared to receiving feedback from the application itself.

The value of the configured timeout is very critical. Short timeouts detects faults faster, but carries higher risk of falsely declaring a dead node, such as an action being performed twice, or transfer its load the another overloaded node, causing cascading failure.

Theoretically, a reasonable timeout value is `2d + r`, where `d` is the maximum delay for a packet, and `r` is the node's processing time. However, these values are hardly bounded in practice. So, choosing the timeouts by continuous experimental measurements is usually better.

Some latency-sensitive applications, use UDP rather than TCP, as it's a good choice in situations where delayed data is worthless.

#### Unreliable Clocks

It's hard to define the time inside a distributed system, as each machine has its own notion of time, which maybe slightly faster or slower than others. Network Time Protocol (NTP) is commonly used to solve this problem.

Modern computers have at least two different kinds of clocks:
- **Time-of-delay clock**, which is usually synchronized with NTP to return the current date and time, but it is unsuitable for measuring elapsed time.
- **Monotonic clock**, which is guaranteed to always move forward, therefore suitable for measuring duration (eg. timeouts), but has a meaningless absolute value. It also may use NTP to adjust its frequency: how fast it moves forward.

Unfortunately, our methods for getting a clock to tell the correct time aren't nearly as reliable or accurate. However, we can manage to get a good enough accuracy using GPS receivers, Precision Time Protocol (PTP), and careful deployment and monitoring. Such monitoring ensures that we notice broken clocks before they cause too much damage.

Robust software needs to be prepared to deal with incorrect clocks.

NTP synchronization can't insure correct ordering of events in distributed systems. Thus, an additional causality tracking mechanisms, such as logical clocks (eg. version vectors), is safer alternative.

The best possible time accuracy in practice is probably to the tens of milliseconds, so it might be better to define time within a range of lower to higher possible values. This uncertainty bound can be calculated based on the time source.

Threads can pause for long period of time for multiple reasons (eg. garbage collection), in this period it loses sense of time. So a node in a distributed system must expect such pause even in a middle of a function, and encounter for it.

#### Knowledge, Truth, and Lies

In a distributed system, a node cannot know anything for sure, but we can state the assumptions we are making about the behavior (*the system model*), and algorithms can be proved to function correctly within certain system models.

A node cannot trust its own judgment, and must abide by the voting (*quorum*) decision of other nodes, even if it only effects itself.

When using lock or lease to protect access to some resource, a mechanism such as *fencing* should be enforced to prevent a node that falsely believe it has the access, from disrupting the rest of the system. It's unwise for a service to assume that its clients will always behave well.

Distributed systems problems become much harder if there is a risk that nodes may lie, such a behavior is known as *Byzantine fault*, and a system is *byzantine fault-tolerant* if it continues to operate correctly even if some nodes are malfunctioning or under malicious attack.

*Byzantine fault-tolerant* algorithms are quite complicated and costly to deploy, making them impractical, especially when all nodes are running inside the companies datacenters, but it might make sense in a peer-to-peer network.

Even if we trust our nodes, there is still a weak form of lying, such as hardware issues, software bugs, or misconfiguration. Luckily, we can tolerate this using checksum on TCP or application level for example, and by input validation.

System models with regards to timing includes:
- **Synchronous model**, which is not realistic as it assumes delays never exceeds an upper bound
- **Partially synchronous model** however assumes the system to behave synchronously only for most of the time, which is realistic
- **Asynchronous model** which makes no assumptions about timings, but it's very restrictive

And from node failure perspective, system models include:
- **Crash-stop-faults**, where node only fails when it crashes, thereafter its gone forever
- **Crash-recovery-faults**, where the node can crash at any moment, but perhaps respond again after some unknown time
- **Byzantine-faults**, where nodes can do anything including trying to trick other nodes

The most useful model in real systems is the *partially synchronous model* with *crash-recovery*.

It's important to distinguish between two kind of properties, *safety* and *liveness*, because it is common to require that safety properties always hold, while with liveness properties we are allowed to make caveats.

We do have to make some assumptions about faults that can happen. However, real implementation might still have to handle impossible cases, even by just firing an error message.

### <a name="ch9">Chapter 9: Consistency and Consensus</a>

The simplest way of handling system faults is to simply let the entire system fail, and then show an error message. But, the best way is to have a general-purpose abstraction with useful guarantees that we can implement.

Consensus is one of the tricky abstractions that helps with getting all of the nodes to agree on something.

#### Consistency Guarantees

Most replicated databases provide at least *eventual consistency*, with such a weak guarantee we need to be aware of its limitations, and not to assume too much, as these limitations only  appear when there is a fault in the system.

Systems with stronger guarantees usually have worse performance, but they are more appealing as they are more easy to use correctly.

#### Linearizability

Linearizability is a strict model of consistency  that makes a system appear as if only one copy of the data, and all operations on it are atomic.

To achieve linearization, we have to add a constraint that when any read returns a new value, all following reads on any client must also return the new value. It's important to note that this model doesn't assume any transaction isolation though.

It's possible to test whether a system is linearizable by recording the timings of all requests and responses and check whether they are sequential.

Linearizability is often confused with serializability, but they are two quite different guarantees. Serializability is an isolation property, it guarantees that transactions behaves the same as if they were executed serially. Linearizability however is a recency guarantee on reads and writes. A database may provide both guarantees, two-phase locking and actual serial execution are typically linearizable, while serial snapshot isolation is not (by design).

Linearizability is important in few areas such as leader election using a lock, constraints and uniqueness guarantees, and cross-channel timing dependencies. Without such recency guarantees, race conditions can happen, and linearization might be the easiest way to avoid it. However, some constraints such as foreign key or attribute constraints doesn't require linearization.

Systems with a single copy of data are linearizable, but not fault-tolerant. So to implement linearizability with fault-tolerance we need either a single-leader replicated system, or to use consensus algorithms. Systems with multi-leader replication are not linearizable, while leaderless replicated systems (Dynamo-style) are hardly linearizable at cost of reduced performance (not recommended).

According to CAP theorem, applications that don't require linearizability, can be more tolerant of network problems, because it can remain available in the face of them.

Although linearizability is a useful guarantee, few systems are actually linearizable in practice, as it is always slow even when there is no network fault, which decreases the performance significantly.

#### Ordering Guarantees

Ordering helps preserve causality, and a system that obeys the order imposed by causality is called *causally consistent* (eg. snapshot isolation provides causal consistency).

Causality uses *partial order*, where concurrent operations may be processed in any order, but non-concurrent operations must be ordered. This is weaker than linearizability which uses *total order*, where it allows any two elements to be compared and ordered.

Linearizability doesn't have concurrent operations, however, it is one of the ways of preserving causality.

Causal consistency is the strongest possible consistency model that doesn't slow down or fail due to network failures or delays.

Causal consistency needs to track causal dependencies across the entire database, not just for a single key, so *version vectors* can be used for that. However, keeping track of all dependencies can become impractical, so a better way could be to use *sequence numbers* or *timestamps* (from a logical clock) to order events instead. These numbers are compact and provide a total order.

The best known way of generating *sequence numbers* for causal consistency is **Lamport timestamps**, where every node and every client keeps track of the *maximum* counter value it has seen so far, and then includes it on every request.

The difference between Lamport timestamps and version vectors, is that version vectors can distinguish whether two operations are concurrent or weather one is causally dependent on the other, whereas Lamport timestamps always enforce total ordering. Lamport timestamps are more compact, but we cannot use it to tell whether two operations are concurrent or casually dependent.

In order to use total ordering between multiple nodes, we should use *total order broadcast*, which is a message exchanging protocol that guarantees reliability (no messages are lost), and total ordered delivery of messages to all nodes.

Total order broadcast is used in database replication, serializable transactions, creating messages log, and lock for fencing tokens.


#### Distributed Transactions and Consensus

The goal of consensus is to get several nodes to agree on something, it's very useful in leader election, and atomic commits.

Theoretically, consensus in impossible in an unreliable asynchronous system, but it's made possible in practice when timeouts and crash-detection are allowed.

Atomic commit is easy on a single node, as it just depends on the order in which data is durably written to disk, but it's quite challenging when performed across multiple nodes, as it's not sufficient to send a commit request to all nodes independently. Most NoSQL datastores don't support such transactions, but various relational systems do.

The most common algorithm for solving atomic commit across multiple nodes is *two-phase commit* (2PC), where applications read and write data on multiple database nodes as normal, and when each is ready to commit, the coordinator begins a *prepare* phase asking each node whether it's ready to commit, after which the coordinator sends to them either a *commit* or *abort* messages.

The 2PC protocol contains two crucial points of "no return", when a participant votes yes in the *prepare* phase, and when the coordinator decides the decision. The decision is irrevocable, but could be undone by another *compensating transaction*.

Participants of 2PC can safely abort if the coordinator failed before the *prepare phase*, but if the coordinator failed after that, the participant can do nothing but to wait for it to recover using it's log. A 3PC algorithm can solve this issue in theory but not in practice.

Distributed transactions provides important safety guarantees, but they are criticized for operational problems, and killing performance (up to 10 times slower). It can be implemented either as an internal protocol were all nodes runs the same software, which allow for optimizations, or as a heterogeneous protocol that requires all nodes to be implementing the same standard (eg. XA).

The problem with transaction that is stuck waiting for coordinator, is that it cannot release the locks it's holding, which can cause larger parts of the application to become unavailable. This can be fixed by either a human manual interaction, or using an automated *heuristic decisions*.

Consensus algorithm in a formalized form should satisfy 4 properties: all nodes should agree on the same decision, no node decides twice, if a node decides a value, then the value must have been proposed by some node, and every non-crashed node should eventually decide some value. However, for the last property to be guaranteed, the algorithm requires at least the majority of the nodes to be functioning correctly.

The best known fault-tolerance consensus algorithms are Viewstamped Replication, Paxos, Raft, and Zab. All of them (except Paxos) implement total order broadcast directly, as total order broadcast is equivalent to repeated rounds of consensus.

All consensus protocols use a leader internally, but don't guarantee the leader uniqueness, instead a weaker guarantee is to only guarantee the leader uniqueness within each *epoch* (lifetime of a leader). Thus, we end-up having two round of voting, once to choose a leader, and a second to vote on leader's proposal.

Some limitations of consensus algorithms include: it requires a strict majority to operate, it assumes a fixed set of nodes to participate in voting, it generally relies on timeouts to detect failed nodes, and it's sensitive to network problems.

One of the most important applications of fault-tolerant consensus algorithms are membership and coordination services (eg. ZooKeeper), they are used in linearizable atomic operations, total ordering of operations, failure detection, change notifications, allocating work to nodes, and service discovery. However, they're usually used indirectly through other services.

## <a name="p3">Part III: Derived Data</a>

Systems often consist if multiple databases, indexes, caches, and analytics systems, thus, it needs to implement mechanisms for moving data from one store to another.

Systems that stores and process data can be grouped into two broad categories:
- System of records: acts as a source of truth, and holds the authoritative version of any new coming data, while being represented only once, and usually normalized.
- Derived data systems: the result of transforming or processing data from other source. It can easily be recreated from its original source if lost. Its commonly denormalized and redundant, but is essential for good read performance.

### <a name="ch10">Chapter 10: Batch Processing</a>

All systems can fit into three main categories:
- Services (online), where it waits for requests, tries to handle them as quickly as possible, and sends back a response.
- Batch Processing Systems (offline), where it runs a job to process a large amount of *bounded* input, and produce some output data. It is often scheduled periodically, and can take up to several days as no user is typically waiting for it to finish.
- Stream Processing Systems (near-real-time), where is consumes *unbounded* input shortly after its available, processes it, and produces output.

#### Batch Processing with Unix Tools

Many data analysis can be done in few minutes using some combination of Unix commands `awk`, `sed`, `grep`, `sort`, `uniq`, and `xargs`.

Unix shell lets us easily compose these small programs (commands) into powerful data processing jobs, for this to happen, all these programs should have the same input/output interface. In Unix, this interface is a just a file. And with the help of pipes, these files can be treated as a sequence of bytes flowing from one program to another.

Reasons that Unix tools are so successful includes that the input files are treated as immutable, we can end the pipeline are any point, and we can write the output of intermediate stages to files for fault-tolerance.

The biggest limitation of Unix tools is that it can only run on a single machine, that's where tools like Hadoop come in.

#### MapReduce and Distributed File Systems

Same as Unix tools, MapReduce job doesn't modify the input, but it can be distributed across thousands of machines, and instead of using local file system, it reads and writes to distributed file system (eg. HDFS).

To create a MapReduce job, we need to implement a *mapper* callback function, which is called once for every input record and has the job of extracting key-value pair(s) from the record, and a *reducer* callback function, that collects all the values belonging to the same key, and use them to produce a number of output records, which is configured by the user.

MapReduce jobs cannot have any randomness, and the range of problems we can solve with single MapReduce job is limited, so commonly jobs are chained together to form a *workflow*, which is done implicitly using directory names.

Joins are necessary whenever we need to access records on both sides of an association, but MapReduce has no concept of indexes to help in the join operation, it reads the entire content of the file. One solution is to query the file needed for join over the network, however such an approach is most likely to suffer from poor performance, and also can lead to inconsistency if the data changes over the time of round-trip. So, a better approach would be to clone the other database into the distributed system.

MapReduce can perform joins through several ways, such as *sort-merge joins*, *broadcast hash joins*, and *partitioned hash joins*.

Google initially used MapReduce to build the index of its search engine, and even though it later moved away from it, MapReduce reamins a good way of building indexes for Lucene/Solr, as it's effective in performing full-text search.

The output of batch jobs are often some kind of database, and having a client library inside the mapper or reducer to write to a database can be a bad idea for many reasons, instead, we can build a new database inside the batch job, and write its files to the distributed file system.

The Unix-like philosophy in MapReduce has many advantages, as we can simply rollback any changes with the guarantee of the producing the same output when the job runs again, feature development can proceed more quickly, it transparently retries failed tasks without affecting the application logic, it provides a separation of concerns, and the same set of files can be used as inputs for different jobs.

Distributed file systems open the possibility of dumping data in file systems, and making data available quickly even if it was in raw format, which can then be consumed by *Data Warehouses* or other services after it is cleaned up and transformed.

MapReduce approach is more appropriate for large jobs that takes a long time, and likely to experience at least one failure.

#### Beyond MapReduce

MapReduce's approach is fully materialized, which means to eagerly compute results of some operations and write them out rather than computing them on demand. This prevents any job from starting until all its preceding jobs are completed, mappers are often redundant, and this extra intermediate storage have to be replicated which wastes a lot of resources.

MapReduce also has some other limitations apart from materialization, where implementing complex job using the raw MapReduce APIs is quite hard, and it provides poor performance for some kind of processing.

Dataflow engines (eg. Spark, Tex, Flink, etc.) overcome MapReduce disadvantages by handling and entire workflow as one job, rather than small independent sub-jobs. It also provides more flexible callback functions (*operations*) rather than only map and reduce.

We can use dataflow engines to implement the same computations as MapReduce, but it executes significantly faster. However, because they dismiss the intermediate materialization, they have to to recompute most of the data when the job fails. However, the re-computation overhead of dataflow engines makes it challenging to use it with small data or CPU-intensive computations, so materialization would be cheaper.

Higher-level languages and APIs (eg. Hive, Pig) has the advantages of requiring less code, while also allowing interactive use. Moreover, it also improves the job execution efficiency at the machine level.

One of the biggest advantages of batch processing jobs compared to typical databases, is the freedom of writing arbitrary code inside the callback functions, which is usually in high level languages.

Batch processing has an increasingly important applications in statistical and numerical algorithms, machine learning, recommendation systems, and computing spatial algorithms as well.

### <a name="ch11">Chapter 11: Stream Processing</a>

Some problems of batch processing are that it has to read the entire input before processing, and that the input is reflected to output after a very long period (up to days sometimes), stream processing come into place as a solution for these two problems, where processing is run over small chunks of input for a very small time frames (usually less than a second).

#### Transmitting Event Streams

Stream is usually a sequence of *events*: a small, self-contained, immutable objects with a timestamp. Related events are usually grouped together into a *topic*.

A database is sufficient to connect producers and consumers. However, continuous polling is expensive, so its better for consumers to be notified when new events appear, the behavior that usually requires specialized tools such as messaging systems.

Messaging systems allows multiple producers nodes to send messages to the same topic and allows multiple consumer nodes to receive messages in a topic.

When a producer sends messages faster than the consumer can process, consumers can either drop the messages, buffer them in a queue, or block the producer from sending more messages. And for durability, a combination of writing messages to disk and having replication might be used, but with a cost of lower throughput and higher latency.

One option for a messaging systems is direct network communication, such as UDP multi-cast, broker-less messaging libraries, or direct HTTP or RPC requests. However, their biggest drawback is that they require applications to be aware of loss possibility.

Another more widely used option is communication via a *message broker* or *message queue*, which acts as a server that both producers and consumers connects to, it automatically deletes a message after delivery, it supports some way of subscribing to a subset of topics, and it notifies clients when data changes. Message Broker can decide to distribute the event load among consumers, or deliver all messages to all consumers, or a combination of both.

A variation of message brokers called *log-based brokers*, uses a combination of the durable storage approach of databases, and the low-latency notification facilities of messaging. It is able to achieve high throughput while partitioning across multiple machines, and providing fault tolerance. It is appropriate for stream processing systems that consume input streams and generate derived state or derived output streams.

Log-based messages brokers are suitable in situations with high message throughput, fast processing, and when message ordering is important, while typical JMS/AMQP brokers are typically used when messages are expensive to process, and parallelization is needed.

#### Database and Streams

A database can be represented as a stream, where an *event* can be something that was written to a database, it can be captured, stored, and processed. This representation opens up powerful opportunities for integrating systems.

As the same data appears in several different places, they need to be kept in sync. One option is to use *dual writes*, however, unless there is a good concurrency detection mechanism, it might cause race conditions, and it also requires atomic commits.

A better approach for data sync is *change data capture* (CDC), which is the process of observing all data changes written to a database and extracting them in a form in which they can be replicated to other systems (eg. search index). It allows the database to act as a leader to other followers. It is usually implemented by parsing the replication log of the database, which relies on taking consistent snapshots regularly and *log compaction* to avoid running out of space. 
a
Another approach similar to CDC is *event sourcing*, which records the user's actions (commands) as immutable events rather than the effects of the action. It allows that new side effects easily be chained off the existing events, but it also requires the application to log events and transform it deterministically into application state, thus requiring log compaction as well.

What makes both CDC and event sourcing powerful is the principle of immutability. especially that mutable state and append-only log of immutable events don't contradict each other. This makes it easier to diagnose bugs, help capture more information than just the current state, and makes it easier to evolve the application over time. Also we gain a great flexibility by separating the form of writing from the form of reading data, thus, allowing multiple read views.

The biggest downside of CDC and event sourcing is that the consumers of the event log are usually asynchronous, which might lead to failure in *reading your own writes*. One solution is perform updates on read view synchronously, but a better approach might be to implement linearizable storage using total order broadcast. However, if the event log and application state are partitioned in the same way, then a single-threaded log consumer needs no concurrency control for writes.

The limitations of immutability is that immutable history may grow very large, causing the system to perform poorly. Also, for administrative reasons, data must be completely deleted in some cases, which is surprisingly hard.

#### Processing Streams

A stream can be processed in three ways, either write it to a datastore (eg. database, cache, search index) which can be queried by the clients later, push it as an event to the users, or process it and produce another output stream.

Stream processing acts like batch processing in the way that is consumes input in read-only fashion, and writes output in append-only fashion. However, streams never ends, so operations such as sorting becomes impossible. Also, restarting a failed job is not an option.

Stream Processing is useful in monitoring systems such as fraud detection, financial markets, and factory machines status.

*Complex event processing* (CEP) is an approach that stores queries in the long-term, and try to continuously match it with input streams.

Analytics are usually aggregated over a period of time, this time interval is known as *window*. Many stream processing frameworks use the local system clock for determining windowing, but it breaks down if there is any significant processing lag. So usually 

The problem with defining the *window* interval, is that we can never be sure whether some events are still to come or not. Usually, this is solved by a timeout after not seeing any new events for a while, and then either ignore any straggler event, or publish  a correction for it.

For events to have a reliable timestamp, we have to log three timestamps: the event occurrence time, the event sending time, the event receiving time. This three timestamps can help in estimating the offset between the device clock and server clock.

There are four types of windows:
- Trumbling window: which has a fixed length, and every event belong to exactly one window
- Hopping window: which has a fixed length, but allow windows overlap
- Sliding window: which contain all events that fits in any some interval, and allows overlapping by definition
- Session window: which has no fixed duration, but contains events that occur closely together in time

Same as batch processing, stream processing needs to do joins, either by joining a stream to another stream , or a stream to a table, or two tables. The three types of joins requires the stream processor to maintain some state based on the join input, and a query that state on messages from the other join, which makes the ordering guarantees an important matter, and it is often addressed by using a unique identifier for a particular version of the joined record.

In order to provide fault-tolerance to stream processing, we might want to break the stream into small blocks, and treat each block as a batch (batch size is typically around a second). Also, atomic commits might be necessary to avoid causing side effects twice, and luckily, the overhead of transaction protocols can be amortized by processing several messages within a single transaction.

An alternative for transactions is *idempotence writes*, which are operations that can be performed multiple times and still has the effect as if it was performed once. Any operation can be made idempotent with some extra metadata.

### <a name="ch12">Chapter 12: The Future of Data Systems</a>

#### Data Integration

There is unlikely to be one piece of software that is suitable for all the different use cases, trying to do everything in one piece of software almost guarantees poor implementation, so applications need to compose several different pieces to serve its goals.

When copies of the same data need to be maintained in several storage systems to satisfy different access patterns, some concurrency problems occurs (eg. conflicting writes), so the best approach (when possible) is to funnel all input into a single system (system of records) that decides the ordering of all writes, then it becomes more easier for other systems to derive data from it.

Distributed transactions have been always considered the classic way of keeping different data systems consistent, their biggest advantage opposed to asynchronous derived data systems is that they provide linearizability, but this always comes with some limitations and overheads. However, a middle-ground approach of **log-based derived data** might be the most promising data system right now.

Both batch processing and streaming processing has a quite strong functional flavor which is good for fault tolerance, and for reasoning about the dataflows inside an organization.

Derived views allow *gradual evolution*, which means we can maintain two (old and new) schemas side by side independently. The beauty of this is that we always have a working system to go back to.

Some systems which need a quickly approximated data through stream processing, as well as correct and reliable version of the data later through batch processing, usually use **lambda architecture**, which records incoming data as immutable events to an always-growing dataset, and runs both systems in parallel, where each uses a derived view. The only down side is the operational complexity of debugging and maintaining two different systems.

#### Unbundling Databases

Database is consisted of different interacting components that we usually take for granted to work synchronously to achieve the desired storage role. This traditional synchronous actions require distributed transactions with all its overheads, so an asynchronous event-log (with Idempotence) might be a much more robust and practical approach, which leads us to the concept of *unbundling the database*.

Unbundling the database means building systems that abstractly acts like a database, but it in fact consists of a loosely coupled components. This has the advantages of making the system more robust to outages or performance degradation of individual components.

The goal of unbundling is to allow to combine several different databases in order to achieve good performance for much wider range of workloads that no single piece of software can satisfy them all.

It might make sense to have some parts of a system that specialize in durable data storage, and other parts that specialize in running application code. The two can interact while still remaining independent.

The difference between dataflow systems compared to microservices is that it has a one-directional, asynchronous communication mechanism, rather than synchronous request/response interaction, so instead of RPC we have a stream join between events.

The ideas of stream processing and messaging and not restricted to datacenters, but we can extend them all the way to the end-user devices.

#### Aiming for Correctness

Transactions have been the choice for building correct applications for more than four decades by now, and while in some areas they have been completely abandoned for their overheads, they are not going away, but also correctness can be achieved in the context of dataflow.

Data systems that provide strong safety properties (eg. serializable transactions) are not guaranteed to be free from data loss or corruption. However, it would be easier to recover from such mistakes by preventing faulty code from destroying good (immutable) data. One of the most effective approaches to achieve this is to make all operations *idempotent*.

Two-phase commits are not sufficient to ensure that the transaction will be executed once, so to make an operation idempotent, we need to consider *end-to-end flow* of the whole operation. However, we don't have such an abstraction yet.

The most common way of achieving consensus is by having a single leader node, but also the unbundled database approach with log-based messaging have a similar approach to enforce uniqueness constraint.

Traditionally, executing transactions across multiple partitions requires an atomic commit, but equivalent correctness can be achieved with partitioned logs as follows:
- The request is given a unique ID by the client, and atomically appended to the log partitioned based on its ID
- A stream processor reads the log for requests, and emits message(s) with the request ID to output streams
- Further processors consumes the output streams

Consistency conflates two different requirements, which are *timeliness* and *integrity*. Violation of timeliness is eventual consistency, whereas violation of integrity is *perpetual consistency* and can be catastrophic!

Event-based dataflow systems decouples timeliness and integrity, there is no guarantee of timeliness, but integrity can be achieved through:
- Representing the content of the atomic write operation as a single message
- Deriving all other state updates from the single message using deterministic derivation functions
- Passing a client-generated request ID through all these levels of processing, enabling end-to-end duplicate suppression
- Making messages immutable

It is always a good idea not to just blindly trust the guarantees given by a software, no matter how widely used it is, because bugs can always creep in. We should have a way of finding out (preferably automatically and continually) if the data has been corrupted so that we can fix it and track down the source of error. This is known as *auditing*.

Event-based systems can provide better auditability than transaction-based systems, as it gives a clear picture of why the mutations were performed. Also, a deterministic and well-defined dataflow makes it easier to debug and trace the execution of the system.

It would be better if we can check that the entire derived data pipeline is correct *end-to-end*, which can give us confidence about the correctness of any disks, networks, services, and algorithms along the path.

#### Doing the Right Thing

Every system is built for a purpose which has both intended and unintended consequences. We are responsible to carefully consider those consequences.

Predictive analytics systems which usually rely on machine learning can be very misleading. If there is a systematic bias in the input, the system will most likely learn and amplify that bias in the output.

We should not retain data forever, but purge it as soon as it is no longer needed. This contradicts with the idea of immutability, but a promising approach might be to enforce access control through cryptographic protocols, rather than merely by policy.

## <a name="more">More Reading Notes</a>
- [Software Architecture Patterns](https://github.com/ahmedhammad97/Software-Architecture-Patterns-Notes)
- [Clean Code](https://github.com/ahmedhammad97/Clean-Code-Do-And-Dont)
- [Principles of Package Desgin](https://github.com/ahmedhammad97/Principles-of-Package-Design-Reading-Notes)
