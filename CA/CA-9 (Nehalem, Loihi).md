# Intel Nehalem & Intel Loihi

> **PYQ scope:** Nehalem questions are about SMT, pipeline stages (FEP, ILD, Scheduler, Retirement), the on-die integration benefit, and SMT speedup benchmarks. Loihi is a single question — the core count.

---

## Intel Nehalem

### Why Nehalem matters architecturally

Before Nehalem (e.g., in the earlier Core 2 architecture), the **memory controller and cache coherence logic lived on a separate chip** called the North Bridge. Every memory request from a core had to travel off-die, cross a bus to the North Bridge, and come back. This added latency and capped bandwidth.

Nehalem moved the **DRAM controller, the shared L3 cache, and the QPI ports all onto the same silicon die as the four cores**. This is what the PYQ directly tests:

> *"The fact that the DRAM controller, the L3 cache, and QPI ports are all housed within the same silicon die as the four cores... is the principal cause of the high performance of the Nehalem."*

The memory hierarchy is:
- **L1:** Private per-core (32 KB instruction + 32 KB data)
- **L2:** Private per-core (256 KB)
- **L3:** Shared across all cores, on-die (up to 8 MB in some configurations), inclusive

### Quick Path Interconnect (QPI)

QPI is a **point-to-point** link used for chip-to-chip communication (e.g., in multi-socket systems, one Nehalem chip talks to another via QPI). It replaced the old Front Side Bus. Each direction is independent (full-duplex), running at 6.4 GT/s, giving 12.8 GiB/s in each direction.

---

### The Front-End Pipeline (FEP)

The FEP is the part of the pipeline responsible for **getting instructions out of memory and converting them into micro-ops** that the rest of the machine can execute. It does not execute anything — it feeds the execution engine.

The pipeline stages in order:

#### 1. Instruction Fetch / Pre-fetch
Instructions are fetched from the L1 instruction cache (or pre-fetch buffers) in **16-byte chunks** per cycle.

#### 2. Instruction Length Decoder (ILD)
The ILD takes those 16 bytes and figures out **where each instruction starts and ends**. x86 instructions are variable length (1 to 15 bytes), so this is non-trivial — the ILD has to scan the byte stream and mark boundaries. This is what the PYQ tests:

> *"The unit which accepts 16 bytes from the L1 instruction cache or pre-fetch buffers and prepares the Intel64 instructions found there for instruction decoding downstream is the **Instruction Length Decoder**."*

#### 3. Instruction Decoder
Takes the bounded x86 instructions and **translates each one into one or more micro-ops (µops)**. Simple instructions (e.g., `ADD reg, reg`) → 1 µop. Complex instructions → multiple µops.

#### 4. Micro-fusion
Some pairs of µops can be **fused back together** into a single µop for the purpose of rename/dispatch. This saves RS and ROB slots.

#### 5. Scheduler (Reservation Stations)
Holds µops waiting for their operands. When all operands for a µop are ready, the scheduler **dispatches it to the appropriate execution port**. This is out-of-order execution — µops don't wait for earlier µops that aren't ready; they execute as soon as their own inputs are ready.

> PYQ: *"The unit responsible for retrieving blocks of macro-instructions from memory and translating them into micro-ops and buffering them for the downstream stages is the **Front-End Pipeline (FEP)**."*

#### 6. Execution Units
Actual functional units: ALUs, FPUs, load/store units. Multiple ports, multiple units per port. µops execute here.

#### 7. Retirement Unit (ROB)
The **Re-Order Buffer (ROB)** holds µops that have completed execution but haven't yet committed their results to the architectural state (registers, memory). Instructions retire **in-order** — even though they execute out-of-order, results are committed in program order. This is what preserves the illusion of sequential execution and handles exceptions correctly.

---

### Simultaneous Multithreading (SMT) — Hyper-Threading

SMT is the technique of running **multiple hardware threads on a single physical core at the same time**, sharing the core's functional units.

The PYQ definition is direct:

> *"The idea behind SMT is to utilize functional units with independent operations from the same or different threads."*

The key insight: a single thread cannot keep all functional units busy all the time (cache misses, data dependencies, etc. leave units idle). SMT fills those idle slots with µops from a **different thread**. The core's execution resources are shared; the architectural state (registers, PC, etc.) is **replicated per thread**.

Nehalem supports **2 hardware threads per core** (HT = Intel's brand name for SMT).

#### SMT Speedup — What the PYQ benchmarks mean

The 2025 paper gives speedup numbers for a 4-core Nehalem with and without SMT (HT). You need to be able to read the table and answer questions from it. The two benchmarks are **PARSEC** (parallel compute-heavy workload) and **Java** (managed runtime, more memory-latency sensitive).

The benchmark table from the paper (values you need):

| Config | PARSEC speedup | Java speedup |
|---|---|---|
| 4-core, no SMT | ~3.04 | ~3.916 |
| 4-core, with SMT | (efficiency 75%) | (efficiency 97%) |

**Speedup efficiency** = $\dfrac{\text{actual speedup}}{\text{maximum possible speedup}} \times 100\%$

For 4-core + SMT, the maximum possible speedup is 8 (4 cores × 2 threads). But that's rarely the framing used — the efficiency is typically computed relative to the ideal linear case.

> PYQ Q31: 4-core, no SMT, PARSEC speedup ≈ **3.04**
> PYQ Q34: 4-core, no SMT, Java speedup ≈ **3.916** (closest answer: 3.916)
> PYQ Q33: 4-core, with SMT, PARSEC speedup efficiency ≈ **75%**
> PYQ Q32: 4-core, with SMT, Java speedup efficiency ≈ **97%**

The reason Java benefits more from SMT than PARSEC: Java workloads have more memory latency (GC pauses, pointer chasing), so the second thread finds more idle cycles to fill. PARSEC is compute-bound, so the second thread competes for the same functional units rather than filling idle slots.

---

## Intel Loihi (Neuromorphic Processor)

For the exam, the only directly tested fact is:

> *"The number of neuromorphic cores in the Intel Loihi chip is **128**."*

Loihi is fabricated in Intel's 14-nm process, with a die area of 60 mm². It contains 128 neuromorphic cores plus 3 embedded x86 cores. The neuromorphic cores implement spiking neural networks (SNNs) — computation using discrete spike events rather than continuous activation values, inspired by biological neurons.

If descriptive questions on Loihi ever appear (they haven't in PYQs yet), the key ideas are:
- Each neuromorphic core handles up to 1,024 neuron compartments
- Cores communicate via an asynchronous Network-on-Chip (NoC) using spike messages
- The chip supports on-chip learning (modifiable synaptic weights) via programmable STDP rules
- Cores synchronize using a barrier mechanism, not a global clock

---

## Summary — PYQ Answer Key

| Q          | Topic                         | Answer                                                                    |
| ---------- | ----------------------------- | ------------------------------------------------------------------------- |
| Q27 (2025) | SMT idea                      | Utilize functional units with independent ops from same/different threads |
| Q28 (2025) | FEP role                      | Front-End Pipeline                                                        |
| Q29 (2025) | On-die integration            | Principal cause of Nehalem's high performance                             |
| Q30 (2025) | ILD                           | Instruction Length Decoder                                                |
| Q31 (2025) | PARSEC, 4-core no SMT         | 3.04                                                                      |
| Q32 (2025) | Java, 4-core SMT efficiency   | 97%                                                                       |
| Q33 (2025) | PARSEC, 4-core SMT efficiency | 75%                                                                       |
| Q34 (2025) | Java, 4-core no SMT           | 3.916                                                                     |
| Q39 (2025) | Loihi core count              | 128                                                                       |
