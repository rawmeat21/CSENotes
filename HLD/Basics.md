Data systems: databases, caches, search indexes, stream processing, batch processing.

![[Pasted image 20260915102116.png]]

When you combine several tools in order to provide a service, the service’s interface or application programming interface (API) usually hides those implementation details from clients.


### Reliability

The system should continue to work correctly (performing the correct function at the desired level of performance) even in the face of adversity (hardware or software faults, and even human error).

Basically, system can handle when things go wrong.

The things that can go wrong are called faults, and systems that anticipate faults and can cope with them are called fault-tolerant or resilient. 

Fault != failure. Failure is when the system as a whole stops providing the required service to the user.

#### Hardware Faults

example- Hard disks crash, RAM becomes faulty, the power grid has a blackout, someone unplugs the wrong network cable.

How to handle? - Redundancy is a common approach. This approach cannot completely prevent hardware problems from causing failures, but it is well understood and can often keep a machine running uninterrupted for years.

But it may not be enough:

![[Pasted image 20260915103752.png]]

### Software errors

Errors within the system. Some examples:

 - A software bug that causes every instance of an application server to crash when given a particular bad input. For example, consider the leap second on June 30, 2012, that caused many applications to hang simultaneously due to a bug in the Linux kernel.
 - A runaway process that uses up some shared resource like CPU time, memory, disk space, or network bandwidth.
 - A service that the system depends on that slows down, becomes unresponsive, or starts returning corrupted responses.
 - Cascading failures, where a small fault in one component triggers a fault in another component, which in turn triggers further faults.

### Human errors

![[Pasted image 20260915104231.png]]
![[Pasted image 20260915104239.png]]

### Scalability

As the system grows (in data volume, traffic volume, or complexity), there should be reasonable ways of dealing with that growth.

It is a system’s ability to cope with increased load (many users).

#### What is load?

Load can be described with a few numbers which we call **load parameters**. The best choice of parameters depends on the architecture of your system.

It can be requests per second to a web server, the ratio of reads to writes in a database, the number of simultaneously active users in a chat room, the hit rate on a cache, etc.


##### Twitter example

![[Pasted image 20260915104953.png]]
![[Pasted image 20260915105007.png]]
![[Pasted image 20260915105016.png]]

![[Pasted image 20260915105047.png]]
![[Pasted image 20260915105244.png]]

#### How to describe performance of system

• When you increase a load parameter and keep the system resources (CPU, memory, network bandwidth, etc.) unchanged, how is the performance of your system affected?

• When you increase a load parameter, how much do you need to increase the resources if you want to keep performance unchanged?

-> Throughput- the number of records we can process per second, or the total time it takes to run a job on a dataset of a certain size.

-> Response time- the time between a client sending a request and receiving a response.

-> Latency- the duration that a request is waiting to be handled.

Even if you only make the same request over and over again, you’ll get a slightly different response time on every try. 

In practice, in a system handling a variety of requests, the response time can vary a lot. We therefore need to think of response time not as a single number, but as a **distribution of values** that you can measure.

![[Pasted image 20260915110501.png]]

**What causes high response time?** - expensive requests, random additional latency introduced by a context switch to a background process, the loss of a network packet and TCP retransmission, a garbage collection pause, a page fault forcing a read from disk, mechanical vibrations in the server rack, or many other causes.


average response time = mean of all response times.

Better is median response time (middle value). The median is also known as the 50th percentile, and sometimes abbreviated as _p50_.

For checking how bad the outliers (requests which take up more time for some reason), check higher percentiles, like 95-99th. For example, if the 95th percentile response time is 1.5 seconds, that means 95 out of 100 requests take less than 1.5 seconds, and 5 out
of 100 requests take 1.5 seconds or more. 

High percentiles of response times, also known as **tail latencies**, are important
because they directly affect users’ experience of the service. 

For example, Amazon describes response time requirements for internal services in terms of the 99.9th percentile, even though it only affects 1 in 1,000 requests. 

This is because the customers with the slowest requests are often those who have the most data on their accounts because they have made many purchases, that is, they’re the most valuable customers.

On the other hand, optimizing the 99.99th percentile (the slowest 1 in 10,000 requests) was deemed too expensive and to not yield enough benefit for Amazon’s purposes. 

**Reducing response times at very high percentiles is difficult because they are easily affected by random events outside of your control, and the benefits are diminishing.**

For example, percentiles are often used in service level objectives (SLOs) and service level agreements (SLAs), contracts that define the expected performance and availability of a service. An SLA may state that the service is considered to be up if it has a median response time of less than 200 ms and a 99th percentile under 1 s (if the response time is longer, it might as well be down), and the service may be required to be up at least 99.9% of the time. 

These metrics set expectations for clients of the service and allow customers to demand a refund if the SLA is not met.

**Queuing delays**- Queueing delays often account for a large part of the response time at high percentiles. As a server can only process a small number of things in parallel (limited, for example, by its number of CPU cores), it only takes a small number of slow requests
to hold up the processing of subsequent requests. This is called head-of-line blocking. 

Even if those subsequent requests are fast to process on the server, the client will see a slow overall response time due to the time waiting for the prior request to complete. **Due to this effect, it is important to measure response times on the client side.**



![[Pasted image 20260915112328.png]]

Even if you make the calls in parallel, the end-user request still needs to wait for the slowest of the parallel calls to complete.

Even if only a small percentage of backend calls are slow, the chance of getting a slow call increases if an end-user request requires **multiple backend calls**, and so a higher proportion of end-user requests end up being slow (an effect known as **tail latency amplification**).

#### How to handle load?

scaling up (vertical scaling, moving to a more powerful machine) 
scaling out (horizontal scaling, distributing the load across multiple smaller machines). 

(Distributing load across multiple machines is also known as **shared-nothing architecture**).

**Elastic systems** can automatically add computing resources when they detect a load increase, whereas other systems are scaled manually (a human analyzes the capacity and decides to add more machines to the system). An elastic system can be useful if load is highly unpredictable. 


Distributing stateless services across multiple machines is fairly straightforward. Stateful data systems from a single node to a distributed setup can introduce a lot of additional complexity. 

For this reason, common wisdom until recently was to keep your database on a single node (scale up) until scaling cost or highavailability requirements forced you to make it distributed.


The architecture of systems that operate at large scale is usually highly specific to the
application. There is no such thing as a generic, one-size-fits-all scalable architecture (informally known as magic scaling sauce). The problem may be the volume of reads, the volume of writes, the volume of data to store, the complexity of the data, the response time requirements, the access patterns, or (usually) some mixture of all of these plus many more issues.

For example, a system that is designed to handle 100,000 requests per second, each 1 kB in size, looks very different from a system that is designed for 3 requests per minute, each 2 GB in size, even though the two systems have the same data throughput.

So what can we do? - We can use general ideas and patterns. 


### Maintainability 

Over time, many different people will work on the system (engineering and operations, both maintaining current behavior and adapting the system to new use cases), and they should all be able to work on it productively.

Majority of the cost of software is not in its initial development, but in its ongoing maintenance, like fixing bugs, keeping its systems operational, investigating failures, adapting it to new platforms, modifying it for new use cases, repaying technical debt, and adding new features.

3 design principles which are important:

1. **Operability**: Make it easy for operations teams to keep the system running smoothly.

2. **Simplicity**: Make it easy for new engineers to understand the system, by removing as much complexity as possible from the system. (not the same as simplicity of the UI)

3. **Evolvability**: Make it easy for engineers to make changes to the system in the future, adapting it for unanticipated use cases as requirements change. Also known as extensibility, modifiability, or plasticity.


#### Principles of a good operations team

![[Pasted image 20260915114008.png]]
![[Pasted image 20260915114015.png]]


