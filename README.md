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
