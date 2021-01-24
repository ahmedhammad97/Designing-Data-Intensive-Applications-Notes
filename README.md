# Designing Data Intensive Applications Notes

Although being one of the most important books for the software industry, as it bridges the gap between distributed systems theory and practical engineering, I struggled in finding a good summarized reading notes that covers up all the key points of the book, so here it is, I hope.

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

