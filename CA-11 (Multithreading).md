
# Multithreading

## 1. What Problem Does Multithreading Solve?

A pipelined CPU stalls when it encounters a **cache miss** — the data isn't in cache, so it must be fetched from RAM, which takes many cycles. If no independent instruction is available to execute in the meantime, every functional unit sits idle.

Multithreading keeps those functional units busy by maintaining **multiple threads** in the CPU concurrently. When one thread stalls, the CPU switches to another thread and executes its instructions instead.

> A **thread** is an independent stream of instructions with its own program counter and register state. It is not a separate process — it is a lightweight unit of execution within a program (or across programs).

**Key hardware requirement:** Each thread needs its own private Program Counter and its own set of registers. Thread switches must also be far cheaper than process switches — a process switch (handled by the OS in software) takes hundreds to thousands of cycles; a thread switch in hardware must be nearly zero-cost.

---

## 2. Fine-Grained Multithreading

**Definition:** The CPU switches between threads at **every instruction**, regardless of whether the current thread would have stalled or not. The scheduling policy is round-robin.

The CPU cycles through threads — Thread A's instruction, then Thread B's, then Thread C's, then back to Thread A, and so on. Each thread occupies one slot per round-trip.

**Key property:** In a $k$-stage pipeline with at least $k$ threads, there is never more than one instruction from any single thread in the pipeline at any moment. This means:
- No RAW/WAR/WAW hazards within the pipeline (instructions from the same thread are never adjacent in the pipeline).
- The pipeline never stalls due to data dependencies.

**Timing diagram example (5-stage pipeline, 5 threads A–E):**

| Cycle | IF | ID | EX | MEM | WB |
|-------|----|----|-----|-----|-----|
| 1 | A1 | | | | |
| 2 | B1 | A1 | | | |
| 3 | C1 | B1 | A1 | | |
| 4 | D1 | C1 | B1 | A1 | |
| 5 | E1 | D1 | C1 | B1 | A1 |
| 6 | A2 | E1 | D1 | C1 | B1 |

Each thread's successive instruction (A1, A2, A3…) enters the pipeline 5 cycles apart — never overlapping with another of its own instructions inside the pipeline.

**Limitations:**
- Even threads that are not stalling get delayed — a thread's instructions are spaced $k$ cycles apart regardless.
- If the number of threads is fewer than $k$ (the pipeline depth), some slots will still go unused.
- Thread switching must happen with **zero overhead**. In practice, it is very small but never truly zero.

---

## 3. Coarse-Grained Multithreading

**Definition:** The CPU switches to another thread **only when the running thread causes a stall** (e.g., a cache miss). It does not switch preemptively.

When a stall is detected, the CPU switches to the next available thread. When that thread stalls, it switches again. This continues until either the original thread's stall resolves or a new thread takes over.

**Key contrast with fine-grained:**
- Fewer context switches → lower switching overhead when threads are running smoothly.
- A **single-cycle stall** is still wasted: the switch itself takes at least one cycle, and the pipeline is flushed at that point. So coarse-grained cannot hide short stalls.
- Better suited for hiding long-latency stalls (e.g., a main memory access of 100+ cycles), where the cost of switching is negligible compared to the wait.

**Comparison table:**

| Property | Fine-Grained | Coarse-Grained |
|---|---|---|
| Switch trigger | Every instruction | Only on stall |
| Short stall handling | Hides it completely | Still wastes at least 1 cycle |
| Long stall handling | Handles, if enough threads | Handles well |
| Overhead per switch | Near-zero required | Affordable if infrequent |
| ILP exploitation | Poor (each thread proceeds slowly) | Slightly better per-thread progress |

---

## 4. Medium-Grained Multithreading

**Definition:** A switch is performed when the running thread is **about to issue an instruction** that is likely to cause a long stall — such as a load from uncached memory or a branch. The instruction is issued, and then the CPU immediately switches to another thread.

This avoids even the one-cycle waste of coarse-grained switching, since the switch overlaps with the stall cycle itself. It is not commonly labelled separately in all textbooks but represents a practical middle ground.

---

## 5. Thread Identification in the Pipeline

For the pipeline to operate correctly, it must know which thread each instruction belongs to.

**Fine-grained MT:** Each instruction carries a **thread ID tag** that travels with it through the pipeline stages.

**Coarse-grained MT:** Two options:
1. Thread ID tag (same as above), or
2. **Flush the pipeline** at each switch — after a switch, all instructions in the pipeline belong to the new thread. This works only if switches are infrequent enough that the flush overhead is acceptable.

**Important constraint:** All active threads' instructions should ideally reside in the instruction cache. If a context switch itself causes an I-cache miss, the benefit of multithreading is destroyed.

---

## 6. Simultaneous Multithreading (SMT)

**Definition:** SMT exploits both **ILP (Instruction-Level Parallelism)** and **TLP (Thread-Level Parallelism)** simultaneously. In a superscalar (multiple-issue) processor, multiple instructions are issued per cycle — and those instructions can come from **different threads**.

The key insight: a single thread may not generate enough independent instructions to fill all available issue slots. But instructions from different threads are almost certainly independent of each other, so they can be issued together without hazard concerns.

**Comparison of scheduling approaches** (each slot = one issue slot in one cycle):

| Approach | Description |
|---|---|
| Superscalar (no MT) | Multiple slots per cycle, all from one thread. Slots wasted if thread stalls or has low ILP. |
| Coarse-grained MT | One thread at a time; switch on long stall. Still limited by each thread's ILP. |
| Fine-grained MT | Round-robin across threads each cycle. One slot per cycle used; wastes the other slots in a superscalar. |
| SMT | Multiple slots per cycle, drawn from multiple threads. Both ILP and TLP exploited. |

**Requirements for SMT to work:**
- A large rename register file (to support multiple architectural register sets simultaneously via renaming).
- A ROB per thread (or logically partitioned): each thread's instructions must retire independently and in order relative to that thread.
- Sufficient reservation stations to hold instructions from multiple threads.
- An I-cache capable of feeding instructions from multiple threads without excessive miss rate.

**Limitation:** Even SMT cannot always fill all issue slots every cycle, due to: limited functional units, limited reservation station entries, I-cache bandwidth, or simply too few threads.

---

## 7. Intel's Hyper-Threading (SMT Implementation)

Intel introduced SMT as **Hyper-Threading** — first in the Xeon (2002), then in the Pentium 4 at 3.06 GHz. It supports exactly **2 logical threads** on one physical core.

**Design motivation:** An estimated 5% increase in chip area enables a second thread, using functional units that would otherwise be idle. Intel reported a 25–30% performance improvement from this.

**To the OS:** The two logical threads appear as two separate CPUs sharing caches and RAM.

### 7.1 Resource Sharing Strategies

Intel uses four distinct strategies for managing processor resources between two threads:

**Replication:** Some state must be fully duplicated — one copy per thread. Each thread gets:
- Its own Program Counter
- Its own register rename mapping table (ISA architectural registers → physical rename registers)
- Its own large-page ITLB (instruction TLB)

This replication is what accounts for the ~5% chip area increase.

**Partitioning:** Some resources are statically split 50/50 between threads. Each thread gets exactly half, regardless of whether it uses it. Applies to:
- Load buffers (24 entries each, in Pentium 4 with 48 total)
- Store buffers (12 entries each)
- ROB / Retirement queue (63 entries each, from 126 total)
- Small-page ITLB

Downside: if one thread doesn't use its partition, the other thread cannot access it.

**Competitive sharing (full sharing):** The resource is not divided — whichever thread needs it first gets it. Applies to:
- Reservation stations
- Caches (L1/L2 cache lines)
- Data TLBs
- 2nd-level TLB

Upside: no waste from unused partitions. Downside: one thread can starve the other if it consumes the entire resource.

**Threshold sharing:** A thread can dynamically use the resource up to a set percentage. A floor is reserved for the other thread. Applies to:
- The scheduler that dispatches micro-ops (µops) to reservation stations.

This prevents one thread from monopolizing the scheduler while still being flexible.

**Summary table:**

| Resource | Strategy | Rationale |
|---|---|---|
| Program Counter | Replicated | Each thread has independent instruction pointer |
| Rename tables (RSB) | Replicated | Independent register namespaces |
| Large-page ITLB | Replicated | Independent fetch state |
| Load/Store buffers | Partitioned | Bounded; predictable worst-case sharing |
| ROB | Partitioned | Independent retirement per thread |
| Reservation stations | Competitive | Available in large quantities; starvation unlikely |
| Caches | Competitive | Available in large quantities |
| Data TLBs | Competitive | Available in large quantities |
| Scheduler dispatch | Threshold | Prevents monopolization |
| Execution units | Unaware | Fully shared; threads are unaware of each other here |

### 7.2 Thread Selection Points

The pipeline has multiple **thread selection points** — places where the hardware chooses which thread to serve:
```

Predict/Fetch → [IQ] → Decode → [IQ] → Alloc → [RS] → Schedule → EX → [ROB] → Retire ★ ★ ★ ★

```

Decisions made at each star:
- **Predict/Fetch:** Which thread's instructions to fetch next
- **Decode:** Which thread's instruction to decode
- **Alloc:** Which thread's µop to allocate into the ROB/RS
- **Retire:** Which thread's instruction to retire (commit)
- **Memory pipeline:** Scheduling of MOB (Memory Order Buffer) entries

### 7.3 Applications That Benefit Most from SMT

SMT yields the greatest benefit for workloads with:
- **Complex memory access patterns** — frequent cache misses mean one thread stalls often, giving the second thread room to run.
- **Mix of instruction types** — e.g., one thread doing integer operations while another does floating-point; different functional units are used, reducing contention.

### 7.4 Performance Data

**Benchmark: retired instructions per clock (hypothetical 8-context SMT processor):**

SMT consistently outperforms the equivalent superscalar (no multithreading) across workloads — SPECint95, Apache, OLTP, DSS, SPECint2000.

**Intel Core i7 — SMT enabled vs. disabled performance gain:**

| Workload | Gain |
|---|---|
| Floating Point | 7% |
| 3dsMax | 10% |
| Integer | 13% |
| Cinebench 10 | 16% |
| POV-Ray 3.7 | 29% |
| 3DMark Vantage CPU | 34% |

**Key observation:** Workloads with diverse instruction mixes or high memory latency benefit more.

### 7.5 Cache Sharing Problem

When two threads both have large working sets, competitive cache sharing can cause both to thrash each other's cache lines. Each thread would generate far fewer cache misses running alone. In this scenario, coarse-grained multithreading might actually be more efficient than SMT, since only one thread's working set is active at a time.

---

## 8. Other Processors Using Multithreading

| Processor | Year | Config |
|---|---|---|
| Intel Xeon | 2002 | First with Hyper-Threading (SMT ×2) |
| Intel Pentium 4 (3.06 GHz) | 2002 | SMT ×2 (Hyper-Threading) |
| IBM Power 5 | 2004 | Dual-core, SMT ×2 per core |
| IBM Power 7 | 2010 | 2–8 cores, SMT ×4 per core |
| UltraSPARC T2 (Niagara) | — | Fine-grained MT, 8 threads per core |
| UltraSPARC T3 (Rainbow Fall) | — | Fine-grained MT, 8 threads per core |
| Intel Nehalem (re-introduced) | 2008 | SMT ×2, with on-die memory controller |

---

## 9. PYQ Coverage

### 2025, Q27
> The idea behind Simultaneous Multithreading (SMT) is to:
> **(b) utilize functional units with independent operations from the same or different threads**

SMT fills issue slots using instructions from multiple threads. Instructions across threads are independent by definition (no register sharing), so they can be issued together without hazard checking between them.

### 2025, Q28
> The unit responsible for retrieving blocks of macro-instructions from memory and translating them into micro-ops and buffering them for downstream stages is the:
> **(d) Front-End Pipeline (FEP)**

In Intel's Nehalem microarchitecture (covered more fully in the Nehalem PDF), the FEP handles instruction fetch, decode into µops, and buffering before the out-of-order execution engine. In the Pentium 4 pipeline discussed here: the Trace Cache + Fetch queue + Decode stages form the equivalent front end.

### 2024, Q27
> Simultaneous Multithreading (SMT):
> **(a) allows multiple independent threads to execute simultaneously on the same core**

Note: option (b) says "two threads to use the same functional unit on a single core" — this is wrong. Threads use the same functional units *at different times* or on *different* functional units, not the same one simultaneously.

### CT-II 2021, Q1
> If a processor writes into a block in its local cache, all other caches also obtain the new content immediately in the:
> **(b) write-update protocol**

This is a cache coherence question (covered in the MSI/MESI notes). Included here for completeness — multithreading itself does not resolve cache coherence; that is a separate hardware protocol.