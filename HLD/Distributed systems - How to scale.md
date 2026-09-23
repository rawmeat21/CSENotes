
## Why distribute data over many machines

1. **Scalability** - If your data volume, read load, or write load grows bigger than a single machine can handle, you can potentially **spread the load** across multiple machines.
2. **Fault tolerance/high availability** - If your application needs to continue working even if one machine goes down, you can use multiple machines to give you redundancy. When one fails, another one can take over.
3. **Latency** - If you have users around the world, you might want to have servers at various locations worldwide so that each user can be served from a datacenter that is geographically close to them.


## How to scale to higher load

### Vertical scaling

Which means - upgrade the damn hardware.

**Shared-memory architecture** - Many CPUs, many RAM chips, and many disks can be joined together under one operating system, and a fast interconnection network allows any CPU to access any part of the memory or disk.

Problem:

-> cost grows faster than linearly: a machine with twice as many CPUs, twice as much RAM, and twice as much disk capacity as another typically costs significantly more than twice as much. 

-> Due to bottlenecks, a machine twice the size cannot necessarily handle twice the load.

-> It is also limited to a single geogrphical location.

**Shared-disk architecture** - uses several machines with independent CPUs and RAM, but stores data on an array of disks that is shared between the machines, which are connected via a fast network. This architecture is used for some data warehousing workloads.

Problem: Contention and the overhead of locking limit scalability.


### Horizontal scaling (Shared-Nothing Architectures)

-> Each machine or VM running the database software is called a node. Each node uses its CPUs, RAM, and disks independently. Any coordination between nodes is done at the software level, using a conventional network.

-> No special hardware is required by a shared-nothing system, so you can use whatever machines have the best price/performance ratio.

#### How to distribute data across multiple nodes

**Replication** - Keeping a copy of the same data on several different nodes, potentially in different locations. Replication provides redundancy: if some nodes are unavailable,
the data can still be served from the remaining nodes. Replication can also help
improve performance. 

**Partitioning** - Splitting a big database into smaller subsets called partitions so that different partitions can be assigned to different nodes (also known as sharding).

![Pasted image 20260917114053](../assets/Pasted%20image%2020260917114053.png)

