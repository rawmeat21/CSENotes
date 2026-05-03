## MIMD Architectures

### What is MIMD?

MIMD stands for **Multiple Instruction, Multiple Data**. The idea is to achieve higher performance by running multiple cooperating programs (threads) on multiple processors simultaneously. Each processor executes its own instruction stream on its own data — hence "multiple instruction, multiple data."

The threads need to **exchange information** with each other to cooperate toward a goal. This communication happens in one of two ways: through **shared memory** or through **messages**.

---

### The Two Fundamental Architectures

Every MIMD machine is built around one of two memory models. This is the most important classification.

![[Pasted image 20260503115406.png]]

#### (1) Distributed Memory (Message-Passing)

Each processing element (PE) consists of a processor and its own **local memory**. No PE can directly access the memory of another PE. Whenever interaction among PEs is necessary, they **send messages** to each other over an interconnection network.

These systems are commonly called **multicomputers**.

```
PE0         PE1        ...   PEn
[P0][M0]  [P1][M1]        [Pn][Mn]
      \        |             /
       Interconnection Network
```

#### (2) Shared Memory

All processors are connected to a set of memory modules via an interconnection network. Any processor can directly access **any memory module**. The set of all memory modules defines a **global address space** which is completely visible to all processors.

These systems are commonly called **multiprocessors**.

```
M0    M1   ...   Mk
  \    |         /
   Interconnection Network
  /    |         \
P0    P1   ...   Pn
```

---

### Interconnection Networks

The interconnection network is what connects processors to memory (or to each other). Its quality has a decisive impact on the speed, size, and cost of the entire machine.

Networks are first classified by **topology** as either static or dynamic.

**Static networks:** the connections between switching units are fixed — typically realized as direct point-to-point connections. Also called **direct networks**. Multicomputers are typically based on static networks.

**Dynamic networks:** communication links can be reconfigured by setting the active switching units of the system. Dynamic networks are mainly employed in **multiprocessors**.

Dynamic interconnection networks break down further into two types:

**Shared path networks:** provide a continuous connection among processors and memory blocks. That connection is **shared** among all processors, which must **compete** for its use. Examples: single bus, multiple buses.

**Switching networks:** do not provide a continuous connection. A switching mechanism enables processors to be **temporarily connected** to memory blocks on demand. Examples: crossbar, multistage networks.

```
Dynamic interconnection networks
         /                \
Shared path           Switching
networks              networks
   /    \             /        \
Single  Multiple   Crossbar  Multistage
bus     buses               networks
```

A **crossbar** is a switching network. A **multistage network** is also a switching network. Both appear heavily in exams.

---

### Distributed Memory Systems: Advantages and Disadvantages

**Advantages:**

Since processors work with their own attached local memory most of the time, the **contention problem** (multiple processors fighting for the same memory) is not as severe as in shared memory systems. As a result, distributed memory multicomputers are **highly scalable** and are good architectural candidates for building **massively parallel computers**.

Message passing solves not only communication but **synchronization** as well — when a process receives a message, it implicitly knows the sending process has reached a certain point in its execution. Sophisticated synchronization constructs like monitors or semaphores are not needed.

**Disadvantages:**

To achieve high performance in multicomputers, special attention must be paid to **load balancing** — distributing work evenly across processors.

Message-passing based communication and synchronization can lead to **deadlock** situations.

Although there is no architectural bottleneck, message passing requires **physical copying of data structures** between processes. Intensive data copying can result in significant performance degradation.

---

### Shared Memory Systems: Advantages and Disadvantages

**Advantages:**

There is no need to partition either the code or the data. Therefore, **uniprocessor programming techniques can easily be adapted** to the multiprocessor environment. Neither new programming languages nor specialized compilers are needed to exploit shared memory systems.

There is **no need to physically move data** when processes communicate. The consumer process can access the data from the place where the producer stored it. Communication between processes is therefore very efficient.

**Disadvantages:**

Synchronized access to shared data structures requires **special synchronizing constructs** such as semaphores, conditional critical regions, monitors, and so on. Usually, message-passing synchronization is simpler to understand and apply.

The main disadvantage of shared memory systems is **lack of scalability** due to the contention problem. When several processors want to access the same memory module, they must compete for the right to do so — the winner proceeds while the losers wait. The larger the number of processors, the higher the probability of memory contention. Beyond a certain number of processors, this probability is so high that adding a new processor will not increase performance.

---

### Solutions to Low Scalability in Shared Memory

Three approaches are used:

**1.** Use a **high-throughput, low-latency interconnection network** — this reduces the time wasted waiting for memory access.

**2.** Use **cache memories** — processors keep recently used data locally in cache, reducing the frequency of trips to main memory over the interconnection network.

**3.** Implement the logically shared memory as a **collection of local memories** — this is called **virtual shared memory** or **distributed shared memory** architecture. Any processor can access the local memory of any other processor through the interconnection network, but local access is faster than remote access.

This third solution leads to the Distributed Shared Memory classification described below.

---

### Distributed Shared Memory Systems: Classification

When logically shared memory is physically implemented as local memories at each node, access times are non-uniform — accessing your local memory is faster than accessing a remote node's memory. This gives rise to three classes:

#### (i) NUMA — Non-Uniform Memory Access

![[Pasted image 20260503115439.png]]


The shared memory is divided into as many blocks as there are processors. Each block is attached to a processor as local memory with a direct bus connection. Whenever a processor addresses the part of the shared memory that is its local memory, access is fast. Accessing a remote memory block is slower — hence **non-uniform**.

```
P0  PE0    P1  PE1       Pn  PEn
    M0         M1             Mn
         \      |       /
          Interconnection Network
```

#### (ii) CC-NUMA — Cache-Coherent Non-Uniform Memory Access

![[Pasted image 20260503115528.png]]


CC-NUMA is a compromise between NUMA and COMA. Like NUMA, the shared memory is constructed as a set of local memory blocks. However, to reduce traffic on the interconnection network, each processor node is supplied with a **large cache memory block**. Because multiple caches now hold copies of the same memory block, a **cache coherence protocol** is needed — hence the "CC" prefix.

The **Stanford DASH** architecture is the canonical example of CC-NUMA. It consists of multiple microprocessor clusters connected through a scalable, low-latency interconnection network. Physical memory is distributed among the processing nodes in various clusters. The distributed memory forms a global address space. For each memory block, the **directory** keeps track of which remote nodes are caching it. The directory memory and remote access cache facilitate prefetching and the directory-based coherence protocol.

```
Cluster 1                    Cluster n
[P][Cache] ... [P][Cache]    [P][Cache] ... [P][Cache]
[Directory][Memory]          [Directory][Memory]
[Remote Access Cache]        [Remote Access Cache]
              \                      /
               Interconnection Network
```

#### (iii) COMA — Cache-Only Memory Access

![[Pasted image 20260503115513.png]]


COMA is a special case of NUMA in which the distributed main memories are **converted entirely to caches** at each processor node. There is no conventional memory hierarchy at each node — just a cache and a directory. All the caches together form a global address space. Remote cache access is assisted by the distributed cache directories.

```
P0       P1           Pn
C0       C1           Cn    (Cache)
D0       D1           Dn    (Directory)
         Interconnection Network
```

---

![[Pasted image 20260503115720.png]]

---
### Shared Memory MIMD: Design Issues

The distinguishing feature of shared memory systems is that no matter how many memory blocks are used and how those memory blocks are connected to the processors, the address spaces of all memory blocks are **unified into a global address space completely visible to all processors**.

The three main design issues for increasing scalability of shared memory systems are:

**1. Organization of memory** — whether UMA, NUMA, CC-NUMA, or COMA.

**2. Design of interconnection networks** — crossbar, multistage, buses, etc.

**3. Design of cache coherent protocols** — MSI, MESI, directory-based, etc. (covered in a later chapter).

Shared memory systems are primarily classified by their memory organization since this is the most fundamental design issue.

---

### PYQ Questions for This Chapter

**From 2019 paper Q6:**

> A NUMA machine is a (a) distributed memory system (b) shared memory system. → **(b) shared memory system.** NUMA is a type of distributed _shared_ memory system — the memory is physically distributed but forms a logically shared global address space.

**From 2021 paper Q13, Q15, Q16:**

> - Massively parallel computers are usually (a) multiprocessors (b) multicomputers → **(b) multicomputers** (distributed memory, highly scalable).
> - There is no need to physically move data between communicating processes in (a) multiprocessors (b) multicomputers → **(a) multiprocessors** (shared memory — the consumer reads directly from where the producer wrote).
> - In a distributed shared memory system, (a) direct access to local memory of a remote processor is prohibited (b) any processor can access the local memory of any other processor → **(b).** This is the defining feature of distributed shared memory (e.g. NUMA, CC-NUMA).

**From 2021 paper Q17:**

> In a COMA machine, remote cache access is achieved through (a) distributed cache directories → **(a).** As stated in the notes, COMA uses distributed cache directories to assist remote cache access.

**From 2021 paper Q18:**

> In the Stanford DASH architecture, the directory keeps track of (b) remote nodes that have a copy of each memory block → **(b).** For each memory block, the directory keeps track of which remote nodes are caching it.

**From 2019 paper Q5 (home assignment):**

> A crossbar is a (a) switching network → **(a).** Crossbar is a switching network, not a shared path network.

**From 2023 paper Q2:**

> A NUMA machine is a (a) shared memory system (b) distributed memory system (c) distributed shared memory system → **(c).** NUMA is specifically a _distributed shared memory_ system — the memory is physically distributed but logically shared with a global address space.

**From 2021 paper Q19:**

> A multistage network is a (a) shared path network (b) switching network → **(b).** A multistage network is a switching network.

**From 2019 paper Q20–Q21:**

> A significant advantage of distributed memory systems: **(b) contention problem is not severe.** A disadvantage of shared memory systems: **(b) lack of scalability.**

**From 2022 paper Q27:**

> An interconnection network is a "programmable system" — it is a system because it is composed of many components: buffers, channels, switches, and controls that work together to deliver data → **(c).** This is exactly the definition from the notes on interconnection networks as a system.