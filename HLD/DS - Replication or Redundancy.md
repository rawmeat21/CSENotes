
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




