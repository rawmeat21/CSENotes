
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




