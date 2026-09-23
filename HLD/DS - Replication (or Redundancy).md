
##### Why replicate data?

-> To keep data geographically close to your users (and thus reduce latency)

-> To allow the system to continue working even if some of its parts have failed (and thus increase availability)
-> To scale out the number of machines that can serve read queries (and thus increase read throughput)

**NOTE:** For this entire article (and it's parts, if any), assume that each machine stores ALL data of your app. 

**Storing immutable data in multiple nodes is easy, but if data changes, that's when you have a problem.**

How to make sure the data ends up on all replicas (machines which contain duplicate data)?

## Leaders and Followers

1. One of the replicas is designated the leader (also known as master or primary). When clients want to write to the database, they must send their requests to the leader, which first writes the new data to its local storage.

2. The other replicas are known as followers. Whenever the leader writes new data to its local storage, it also sends the data change to all of its followers as part of a replication log or change stream. Each follower takes the log from the leader and updates its local copy of the database accordingly, by applying all writes in the same order as they were processed on the leader.

3. When a client wants to read from the database, it can query either the leader or any of the followers. However, **writes are only accepted on the leader** (the followers are read only from the client’s point of view).

![[Pasted image 20260917115622.png]]

![[Pasted image 20260917115656.png]]


### Synchronous and asynchronous replication

![[Pasted image 20260917120847.png]]

Replication to follower 1 is synchronous - The leader waits for an OK from the machine before letting the client know the write is done.

Replication to follower 2 is asynchronous: The leader sends the message, but doesn’t wait for a response from the follower.

The diagram shows that there is a substantial delay before follower 2 processes the
message:

Normally, replication is quite fast: most database systems apply changes to
followers in less than a second. **However, there is no guarantee of how long it might take**. 

There are circumstances when followers might fall behind the leader by several minutes or more; for example, if a follower is recovering from a failure, if the system is operating near maximum capacity, or if there are network problems between the nodes.

**Why use synchronous replication?** - Follower is guaranteed to have an up-to-date copy of the data that is consistent with the leader. If the leader suddenly fails, we can be sure that the data is still available on the follower.

**Problem with synchronous replication**:  If the synchronous follower doesn’t respond (because it has crashed, or there is a network fault, or for any other reason), the write cannot be processed. **The leader must block all writes and wait until the synchronous replica is available again.**

**That is why you shouldn't make all your followers synchronous!**

---> In practice, if you enable synchronous replication on a database, it usually means that **one of the followers is synchronous, and the others are asynchronous.**

---> If the synchronous follower becomes unavailable or slow, one of the asynchronous followers is made synchronous. This guarantees that you have an up-to-date copy of the data on at least two nodes: the leader and one synchronous follower. This configuration is sometimes also called **semi-synchronous**.

---> Often, leader-based replication is configured to be **completely asynchronous**. 

--> In this case, if the leader fails and is not recoverable, any writes that have not yet been replicated to followers are lost. This means that a write is **not guaranteed to be durable**,
even if it has been confirmed to the client (it may not happen even if leader said so).

--> However, a fully asynchronous configuration has the advantage that the leader can continue processing writes, even if all of its followers have fallen behind.


### How to setup new followers?

How to add a new follower? 

**Simply copying data files from one node to another is typically not sufficient**: clients are constantly writing to the database, and the data is always in flux, so a standard file copy would see different parts of the database at different points in time. The result might not make any sense. 

~ You could make the files on disk consistent by locking the database (making it unavailable for writes), but that would go against our goal of high availability.

The method is as follows:

1. Take a consistent **snapshot** of the leader’s database at some point in time, if possible, without taking a lock on the entire database. Most databases have this feature, as it is also required for backups. (In some cases, third-party tools are needed, such as innobackupex for MySQL).
2. Copy the snapshot to the new follower node.
3. The follower connects to the leader and requests all the data changes that have happened since the snapshot was taken. This requires that the snapshot is associated with an exact position in the leader’s replication log. (That position has various names: for example, PostgreSQL calls it the log sequence number, and MySQL calls it the binlog coordinates).
4. When the follower has processed the backlog of data changes since the snapshot, we say it has caught up. It can now continue to process data changes from the leader as they happen.


### How to handle node outages?

There are many things that could potentially go wrong: crashes, power outages, network issues, and more.

#### Follower failure - Use catch-up recovery

-> On its local disk, each follower keeps a log of the data changes it has received from the leader. 

-> If a follower crashes and is restarted, or if the network between the leader and the follower is temporarily interrupted, the follower can recover quite easily: from its log, it knows the last transaction that was processed before the fault occurred. So, the follower can connect to the leader and request all the data changes that occurred during the time when the follower was disconnected. When it has applied these changes, it has caught up to the leader and can continue receiving a stream of data changes as before.


#### Leader failure - Failover

-> One of the followers needs to be promoted to be the new leader

-> Clients need to be reconfigured to send their writes to the new leader

-> Other followers need to start consuming data changes from the **new leader**. 

This process is called **failover**.

This can be done manually. 

**How about automatically?** 

The steps are:

1. **Determining that the leader has failed:** Most systems simply use a timeout: nodes frequently bounce messages back and forth between each other, and if a node doesn’t respond for some period of time (say 30s), it is assumed to be dead. (If the leader is deliberately taken down for planned maintenance, this doesn’t apply.)

2.  **Choosing a new leader:** This could be done through an election process (where the leader is chosen by a majority of the remaining replicas), or a new leader could be appointed by a previously elected **controller node**. The best candidate for leadership is usually the replica with the most up-to-date data changes from the old leader (to minimize any data loss). 
	**Note:** Getting all the nodes to agree on a new leader is a consensus problem.

3. **Reconfiguring the system to use the new leader**: Clients now need to send their write requests to the new leader. If the old leader comes back, it might still believe that it is the leader, not realizing that the other replicas have forced it to step down. The system needs to ensure that the old leader becomes a follower and recognizes the new leader.


##### Problems with failover

![[Pasted image 20260917124635.png]]
![[Pasted image 20260917124644.png]]


There are no easy solutions to these problems. For this reason, some operations teams prefer to perform failovers manually, even if the software supports automatic failover.


### How does leader based replication work though?

#### Statement-based replication

The leader logs every write request (statement) that it executes and sends that statement log to its followers.

For a relational database, this means that every INSERT, UPDATE, or DELETE statement is forwarded to followers, and each follower parses and executes that SQL statement as if it had been received from a client.

Problems with this:

- Any statement that calls a nondeterministic function, such as NOW() to get the current date and time or RAND() to get a random number, is likely to generate a different value on each replica.
-  If statements use an autoincrementing column, or if they depend on the existing data in the database (`e.g., UPDATE … WHERE <some condition>`), they must be executed in exactly the same order on each replica, or else they may have a different effect. This can be limiting when there are multiple concurrently executing transactions.
- Statements that have side effects (e.g., triggers, stored procedures, user-defined functions) may result in different side effects occurring on each replica, unless the side effects are absolutely deterministic.


#### Write-ahead log (WAL) shipping

-  In the case of a log-structured storage engine (see “SSTables and LSM-Trees” on page 76), this log is the main place for storage. Log segments are compacted and garbage-collected in the background.
- In the case of a B-tree (see “B-Trees” on page 79), which overwrites individual disk blocks, every modification is first written to a write-ahead log so that the index can be restored to a consistent state after a crash.

We can use the exact same log to build a replica on another node: besides writing the log to disk, the leader also sends it across the network to its followers.

When the follower processes this log, it builds a copy of the exact same data structures as found on the leader.

This method of replication is used in PostgreSQL and Oracle, among others.

Problem: The log describes the data on a very low level: a WAL contains details of which bytes were changed in which disk blocks. This makes replication closely coupled to the storage engine. If the database changes its storage format from one version to another, it is typically not possible to run different versions of the database software on the leader and the followers.


#### Logical (row-based) log replication

Use different log formats for replication and for the storage engine, which allows the replication log to be decoupled from the storage engine internals. This kind of replication log is called a logical log, to distinguish it from the storage engine’s (physical) data representation.

A logical log for a relational database is usually a sequence of records describing writes to database tables at the granularity of a row:
-  For an inserted row, the log contains the new values of all columns.
-  For a deleted row, the log contains enough information to uniquely identify the row that was deleted. Typically this would be the primary key, but if there is no primary key on the table, the old values of all columns need to be logged.
-  For an updated row, the log contains enough information to uniquely identify the updated row, and the new values of all columns (or at least the new values of all columns that changed).

MySQL’s binlog (when configured to use row-based replication) uses this approach.

-> A logical log is decoupled from the storage engine internals, it can more easily be kept backward compatible, allowing the leader and the follower to run different versions of the database software, or even different storage engines.

-> A logical log format is also easier for external applications to parse. This aspect is useful if you want to send the contents of a database to an external system, such as a data warehouse for offline analysis, or for building custom indexes and caches (called _change data capture).


#### Trigger-based replication

If there are cases like:

-> you want to only replicate a subset of the data
-> want to replicate from one kind of database to another
-> you need conflict resolution logic

Then you may need to do replication at the application layer.

 Use these features that are available in many relational databases: **triggers and stored procedures**

A trigger lets you register custom application code that is automatically executed when a data change (write transaction) occurs in a database system. 

The trigger has the opportunity to log this change into a separate table, from which it can be read by an external process. That external process can then apply any necessary application logic and replicate the data change to another system.

**Databus for Oracle and Bucardo for Postgres work like this, for example.**

Problem:  Trigger based replication typically has greater overheads than other replication methods, and is more prone to bugs and limitations than the database’s built-in replication.


## Replication Lag

**Replication lag means the delay between a write happening on a leader and being reflected on its follower.**


Leader based replication is a _read scaling architecture_.

First of all, you cannot have all synchronous nodes.

**Eventual consistency** 

If an application reads from an asynchronous follower, it may see outdated information if the follower has fallen behind. 

This leads to apparent inconsistencies in the database: if you run the same query on the leader and a follower at the same time, you may get different results, because not all writes have been reflected in the follower. 

This inconsistency is just a temporary state, if you stop writing to the database and wait a while, the followers will eventually catch up and become consistent with the leader. For that reason, this effect is known as eventual consistency.


Some problems which can be caused due to a lag is given:

### Reading Your Own Writes

![[Pasted image 20260922105704.png]]

The user updates the database, then tries to read from a lagging follower, and sees no update is made.


Fix: read-after-write consistency (also known as read-your-writes consistency) 

How to implement this?

-> **When reading something that the user may have modified, read it from the leader; otherwise, read it from a follower**. This requires that you have some way of knowing whether something might have been modified, without actually querying it. For example, user profile information on a social network is normally only editable by the owner of the profile, not by anybody else. Thus, a simple rule is: always read the user’s own profile from the leader, and any other users’ profiles from a follower.  If most things in the application are potentially editable by the user, that approach won’t be effective, as most things would have to be read from the leader (negating the benefit of read scaling).

-> **Track other things to decide whether to read from leader or not.** For example, you could track the time of the last update and, for one minute after the last update, make all reads from the leader. You could also monitor the replication lag on followers and prevent queries on any follower that is more than one minute behind the leader.

->  **The client can remember the timestamp of its most recent write. Then the system can ensure that the replica serving any reads for that user reflects updates at least until that timestamp.** If a replica is not sufficiently up to date, either the read can be handled by another replica or the query can wait until the replica has been updated. The timestamp could be a logical timestamp (something that indicates ordering of writes, such as the log sequence number) or the actual system clock (in which case clock synchronization becomes critical).


Another problem is when user wants to access your service through multiple devices. 

In this case you may want to provide cross-device read-after-write consistency: if the user enters some information on one device and then views it on another device, they should see the information they just entered.

Some issues:

-> Approaches that require remembering the timestamp of the user’s last update become more difficult, because the code running on one device doesn’t know what updates have happened on the other device. This metadata needs to be centralized.

-> If your replicas are distributed across different datacenters, there is no guarantee that connections from different devices will be routed to the same datacenter. (For example, if the user’s desktop computer uses the home broadband connection and their mobile device uses the cellular data network, the devices’ network routes may be completely different.) If your approach requires reading from the leader, you may first need to route requests from all of a user’s devices to the same datacenter.

**1. All nodes in one datacenter (single-DC replication)**
```
        Datacenter A
   +---------------------+
   |  [Leader]            |
   |     |  \             |
   |  [Follower] [Follower]|
   +---------------------+
```

**2. One leader-follower group spread across multiple datacenters**

```
   Datacenter A              Datacenter B
  +-----------+             +-----------+
  |  [Leader]  |---(WAN)--->| [Follower] |
  |     |      |             +-----------+
  | [Follower] |
  +-----------+
```
**3. Multiple independent leader-follower groups, all sharing one datacenter**

```
        Datacenter A
   +----------------------------+
   |  [Leader: OrdersDB]         |
   |  [Follower: OrdersDB]       |
   |                              |
   |  [Leader: UsersDB]           |
   |  [Follower: UsersDB]         |
   +----------------------------+
```

### Monotonic Reads

**When reading from asynchronous followers it’s possible for a user to see things moving backward in time.** (what?)

This can happen if a user makes several reads from different replicas, first to a follower with little lag, then to a follower with greater lag. (This scenario is quite likely if the user
refreshes a web page, and each request is routed to a random server!).

![[Pasted image 20260922111207.png]]

The fix is to use **monotonic reads**. Monotonic reads only means that if one user makes several reads in sequence, they will not see time go backward, i.e., they will not read older data after having previously read newer data.

-> One way of achieving monotonic reads is to make sure that each user always makes their reads from the same replica (different users can read from different replicas).

For example, the replica can be chosen based on a hash of the user ID, rather than
randomly. However, if that replica fails, the user’s queries will need to be rerouted to
another replica.


### Consistent Prefix Reads

Consider the following conversation on a chat app:

```
Mr. Poons
: How far into the future can you see, Mrs. Cake?
Mrs. Cake
: About ten seconds usually, Mr. Poons.
```

Imagine a third person is listening to this conversation through followers. The
things said by Mrs. Cake go through a follower with little lag, but the things said by
Mr. Poons have a longer replication lag: 

![[Pasted image 20260922111738.png]]

The observer sees:

```
Mrs. Cake
: About ten seconds usually, Mr. Poons.
Mr. Poons
: How far into the future can you see, Mrs. Cake?
```

Preventing this kind of anomaly requires another type of guarantee: **consistent prefix reads**

This guarantee says that if a sequence of writes happens in a certain order, then anyone reading those writes will see them appear in the same order.


This is a problem in partitioned / sharded databases. 

If the database always applies writes in the same order, reads always see a consistent prefix, so this anomaly cannot happen. 

However, in many distributed databases, different partitions operate independently, so there is no global ordering of writes: when a user reads from the database, they may see some parts of the database in an older state and some in a newer state.



## Multi-Leader Replication

-> Leader-based replication has one major downside: there is only one leader, and all
writes must go through it. If you can’t connect to the leader for any reason, for
example due to a network interruption between you and the leader, you can’t write to
the database.

The fix? use multiple leaders obv.

We call this a **multi-leader configuration** (also known as master–master or active/active replication).

In this setup, each leader simultaneously acts as a follower to the other leaders.

**It rarely makes sense to use a multi-leader setup within a single datacenter, because the benefits rarely outweigh the added complexity.**

### Use cases

#### Multi-datacenter operation

Imagine you have a database with replicas in several different datacenters (perhaps so
that you can tolerate failure of an entire datacenter, or perhaps in order to be closer to your users). 

With a normal leader-based replication setup, the leader has to be in one of the datacenters, and all writes must go through that datacenter.

**In a multi-leader configuration, you can have a leader in each datacenter.**

![[Pasted image 20260922113220.png]]


#### Clients with offline operation

Multi-leader replication is appropriate is if you have an application that needs to continue to work while it is disconnected from the internet.

Consider the calendar apps on your mobile phone, your laptop, and other devices. You need to be able to see your meetings (make read requests) and enter new meetings (make write requests) at any time, regardless of whether your device currently has an internet connection. 

If you make any changes while you are offline, they need to be synced with a server and your other devices when the device is next online.


In this case, every device has a local database that acts as a leader (it accepts write requests), and there is an asynchronous multi-leader replication process (sync) between the replicas of your calendar on all of your devices. The replication lag may be hours or even days, depending on when you have internet access available.


#### Collaborative editing

Real-time collaborative editing applications allow several people to edit a document simultaneously. For example, Etherpad and Google Docs allow multiple
people to concurrently edit a text document or spreadsheet.


When one user edits a document, the changes are instantly applied to their local replica (the
state of the document in their web browser or client application) and asynchronously
replicated to the server and any other users who are editing the same document.

If you want to guarantee that there will be no editing conflicts, the application must obtain a lock on the document before a user can edit it. If another user wants to edit the same document, they first have to wait until the first user has committed their changes and released the lock. This collaboration model is equivalent to single-leader replication with transactions on the leader.


However, for faster collaboration, you may want to make the unit of change very small (e.g., a single keystroke) and avoid locking. This approach allows multiple users to edit simultaneously, but it also brings all the challenges of multi-leader replication, including requiring conflict resolution.


### Problem with multi leader replication


![[Pasted image 20260923095011.png]]


 Consider a wiki page that is simultaneously being edited by two users. User 1 changes the title of the page from A to B, and user 2 changes the title from A to C at the same time. Each user’s change is successfully applied to their local leader. However, when the changes are asynchronously replicated, a conflict is detected.

#### Synchronous vs asynchrounous

![[Pasted image 20260923095254.png]]

#### Conflict avoidance

If there is 1 leader -> all writes go through it -> conflicts cannot occur.

Better to avoid conflicts than handle them.

In an application where a user can edit their own data, you can ensure that requests from a particular user are always routed to the same datacenter and use the leader in that datacenter for reading and writing. 

Different users may have different “home” datacenters (perhaps picked based on geographic proximity to the user), but from any one user’s point of view the configuration is essentially single-leader.

However, sometimes you might want to change the designated leader for a record - perhaps because one datacenter has failed and you need to reroute traffic to another datacenter, or perhaps because a user has moved to a different location and is now closer to a different datacenter. 

In this situation, conflict avoidance breaks down, and you have to deal with the possibility of concurrent writes on different leaders.


#### Converging toward a consistent state

-> A single-leader database applies writes in a sequential order: if there are several updates to the same field, the last write determines the final value of the field.

-> In a multi-leader configuration, there is no defined ordering of writes, so it’s not clear what the final value should be.

In previous figure, at leader 1 the title is first updated to B and then to C; at leader 2 it is first updated to C and then to B. Neither order is “more correct” than the other.

**If each replica simply applied writes in the order that it saw the writes, the database would end up in an inconsistent state**: the final value would be C at leader 1 and B at leader 2

The conflict must be resolved in a convergent way. 

Some ways to handle this conflict:

- Give each write a unique ID (e.g., a timestamp, a long random number, a UUID, or a hash of the key and value), pick the write with the highest ID as the winner, and throw away the other writes. If a timestamp is used, this technique is known as **last write wins** **(LWW)**. Although this approach is popular, **it is dangerously prone to data loss.**

- Give each replica a unique ID, and let writes that originated at a higher-numbered replica always take precedence over writes that originated at a lower-numbered replica. This approach also implies data loss.

-  Somehow merge the values together, ex: order them alphabetically and then concatenate them (in previous figure, the merged title might be something like “B/C”).

- Record the conflict in an explicit data structure that preserves all information, and write application code that resolves the conflict at some later time (perhaps by prompting the user).


#### Custom conflict resolution

Most multi-leader replication tools let you write conflict resolution logic using application code. That code may be executed on write or on read:

**On write**

As soon as the database system detects a conflict in the log of replicated changes, it calls the conflict handler.

For example, Bucardo allows you to write a snippet of Perl for this purpose.

 This handler typically cannot prompt a user—it runs in a background process and it must execute quickly.

**On read**

When a conflict is detected, all the conflicting writes are stored. The next time the data is read, these multiple versions of the data are returned to the application. 

The application may prompt the user or automatically resolve the conflict, and write the result back to the database. 

CouchDB works this way, for example.



![[Pasted image 20260923101357.png]]


#### What is a conflict?

Consider a meeting room booking system: it tracks which room is booked by which group of people at which time.  This application needs to ensure that each room is only booked by one group of people at any one time.

In this case, a conflict may arise if two different bookings are created for the same room at the same time. Even if the application checks availability before allowing a user to make a booking, there can be a conflict if the two bookings are made on two different leaders.


### Topologies

![[Pasted image 20260923101806.png]]

The most general topology is all-to-all, in which every leader sends its writes to every other leader.

MySQL by default supports only a circular topology, in which each node receives writes from one node and forwards those writes (plus any writes of its own) to one other node.

In circular and star topologies, a write may need to pass through several nodes before it reaches all replicas. Therefore, nodes need to forward data changes they receive from other nodes.

**How to prevent infinite replication loops?**

Use something like a visited array concept.

Each node is given a unique identifier, and in the replication log, each write is tagged with the identifiers of all the nodes it has passed through. When a node receives a data change that is tagged with its own identifier, that data change is ignored, because the node knows that it has already been processed.

In circular and star topologies, a node can fail, and this may disrupt replication. 

The fault tolerance of a more densely connected topology (such as all-to-all) is better because it allows messages to travel along different paths, avoiding a single point of failure.


all-to-all topologies can have issues too. In particular, some network links may be faster than others (e.g., due to network congestion), with the result that some replication messages may “overtake” others.

![[Pasted image 20260923102345.png]]

You might think we should use a clock, but clocks cannot be trusted to be sufficiently in sync.

To order these events correctly, a technique called **version vectors** can be used.


## Leaderless replication

![[Pasted image 20260923102912.png]]


### Writing to the Database When a Node Is Down

In leader based configuration, when leader is down, we need to do a failover. 

Here, failover doesn't exist.

![[Pasted image 20260923103151.png]]


The client (user 1234) sends the write to all three replicas in parallel, and the two available replicas accept the write but the unavailable replica misses it.

Let’s say that it’s sufficient for two out of three replicas to acknowledge the write: after user 1234 has received two ok responses, we consider the write to be successful.

**The client simply ignores the fact that one of the replicas missed the write.**

Now imagine that the unavailable node comes back online, and clients start reading from it. Any writes that happened while the node was down are missing from that node. Thus, if you read from that node, you may get stale (outdated) values as responses.

So, when a client reads from the database, it doesn’t just send its request to one replica: **read requests are also sent to several nodes in parallel.** The client may get different responses from different nodes; i.e., the up-to-date value from one node and a stale value from another. 

Version numbers are used to determine which value is newer.


#### Read repair and anti-entropy

After an unavailable node comes back online, how does it catch up on the writes that it missed?

2 mechanisms are popular:

- **Read repair**: When a client makes a read from several nodes in parallel, it can detect any stale responses. So, we basically update values in the stale node as we read them.
- **Anti-entropy process**: Some datastores have a background process that constantly looks for differences in the data between replicas and copies any missing data from one replica to another. Unlike the replication log in leader-based replication, this anti-entropy process does not copy writes in any particular order, and there may be a significant delay before data is copied.


#### Quorums for reading and writing

In the previous example, what if instead of 2, only 1 recieved the new image?

In general,

If there are n replicas, every write must be confirmed by w nodes to
be considered successful, and we must query at least r nodes for each read. (In our
example, n = 3, w = 2, r = 2.) 

As long as w + r > n, we expect to get an up-to-date value when reading, because at least one of the r nodes we’re reading from must be up to date. Reads and writes that obey these r and w values are called quorum reads and writes.

![[Pasted image 20260923104248.png]]

