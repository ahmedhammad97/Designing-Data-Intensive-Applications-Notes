# Designing-Data-Intensive-Applications-Notes

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
