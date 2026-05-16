# Performance & Benchmarking

> Source: CIS 571, UPenn — Devietti, Martin, Roth

---

## 1. What is Computer Architecture?

> **PYQ CT-II 2021 Q18**: *"The present-day definition of 'Computer Architecture' refers to — (d) all of the above"* ✅

The modern definition of **Computer Architecture** encompasses all three of:
- **Instruction Set Architecture (ISA)** — the programmer-visible interface: instruction set design, data types, addressing modes
- **Organization (Microarchitecture)** — how the ISA is implemented: pipelines, caches, functional units
- **Hardware** — the physical implementation: circuits, technology, physical design

---

## 2. Performance Metrics: Latency vs. Throughput

**Latency (Execution Time):** Time to finish one fixed task. Units: seconds.

**Throughput (Bandwidth):** Number of tasks completed per unit time. Units: tasks/second.

These are different and often in conflict. Parallelism improves throughput but does not necessarily improve latency.

**Which to use:** Match metric to goal.
- Scientific program (finish one job fast) → optimize **latency**
- Web server (serve many users) → optimize **throughput**

**Example: Car vs. Bus moving people 10 miles**

| | Car (capacity=5, speed=60mph) | Bus (capacity=60, speed=20mph) |
|-|-------------------------------|-------------------------------|
| **Latency** | 10 min | 30 min |
| **Throughput** | 15 people/hour | 60 people/hour |

Car wins on latency, bus wins on throughput.

---

## 3. The Basic CPU Performance Equation

$$\text{Execution Time} = \frac{\text{Instructions}}{\text{Program}} \times \frac{\text{Cycles}}{\text{Instruction}} \times \frac{\text{Seconds}}{\text{Cycle}}$$

Or equivalently:

$$T = N \times \text{CPI} \times t_{clk} = \frac{N \times \text{CPI}}{f}$$

Where:
- $N$ = **Dynamic instruction count** — depends on: program, compiler, ISA
- $\text{CPI}$ = **Cycles per Instruction** — depends on: program, compiler, ISA, microarchitecture
- $t_{clk} = 1/f$ = **Clock period** — depends on: microarchitecture, technology

To improve performance, reduce any of these three factors.

### 3.1 CPI and IPC

**CPI** = average cycles per instruction.

**IPC** = $1/\text{CPI}$ = instructions per cycle. Used more often than CPI ("bigger is better").

Different instruction types have different costs. CPI is the weighted average:

$$\text{CPI} = \sum_{i} (\text{fraction}_i \times \text{cycles}_i)$$

**Example:** A program with equal fractions of integer (1 cycle), memory (2 cycles), FP (3 cycles) instructions:

$$\text{CPI} = \frac{1}{3}(1) + \frac{1}{3}(2) + \frac{1}{3}(3) = 2$$

**Example — Which optimization helps more?**

Instruction mix: Integer ALU 50% (1 cycle), Load 20% (5 cycles), Store 10% (1 cycle), Branch 20% (2 cycles)

$$\text{Base CPI} = 0.5(1) + 0.2(5) + 0.1(1) + 0.2(2) = 0.5 + 1.0 + 0.1 + 0.4 = 2.0$$

Option A — branch prediction reduces branch cost to 1 cycle:
$$\text{CPI}_A = 0.5(1) + 0.2(5) + 0.1(1) + 0.2(1) = 0.5 + 1.0 + 0.1 + 0.2 = 1.8$$

Option B — faster memory reduces load cost to 3 cycles:
$$\text{CPI}_B = 0.5(1) + 0.2(3) + 0.1(1) + 0.2(2) = 0.5 + 0.6 + 0.1 + 0.4 = 1.6$$

**Option B gives more improvement** (CPI 2.0 → 1.6 vs 2.0 → 1.8) because loads have a higher frequency and a larger absolute cycle reduction.

### 3.2 Clock Frequency is Not the Whole Story

Clock frequency alone is a misleading metric:

| Processor | CPI | Clock | Execution Time (relative) |
|-----------|-----|-------|--------------------------|
| A | 2 | 5 GHz | $2/5 = 0.4$ ns/instr |
| B | 1 | 3 GHz | $1/3 = 0.33$ ns/instr |

Despite A having a higher clock, **B is faster** because lower CPI more than compensates. Always evaluate the full equation.

---

## 4. Comparing Performance — Speedup

**Speedup of A over B:**

$$\text{Speedup} = \frac{\text{Latency}(B)}{\text{Latency}(A)} = \frac{\text{Throughput}(A)}{\text{Throughput}(B)}$$

**"A is X% faster than B":**

$$X = \left(\frac{\text{Latency}(B)}{\text{Latency}(A)} - 1\right) \times 100$$

**Example:** Program A runs in 200 cycles, program B in 350 cycles.
- Speedup of A over B: $350/200 = 1.75\times$
- A is $(1.75 - 1) \times 100 = 75\%$ faster than B
- % decrease in cycles from B to A: $(350-200)/350 \times 100 = 42.3\%$

Note: **% increase ≠ % decrease**. The 75% and 42.3% above are both valid — they just have different denominators.

---

## 5. Averaging Performance Numbers

Use the **right mean** for the right quantity:

| Mean Type | When to Use | Formula |
|-----------|------------|---------|
| **Arithmetic** | Quantities proportional to time (latencies) | $\frac{1}{N}\sum P_i$ |
| **Harmonic** | Quantities inversely proportional to time (rates, throughputs) | $\frac{N}{\sum \frac{1}{P_i}}$ |
| **Geometric** | Unitless ratios (speedups) | $\sqrt[N]{\prod P_i}$ |

You can **add latencies**, but you **cannot add throughputs/rates**.

**Example — Why harmonic mean for rates:**

Drive 2 miles: first mile at 30 mph, second mile at 90 mph. Average speed?

$$\text{Time}_1 = \frac{1}{30}\ \text{hr},\quad \text{Time}_2 = \frac{1}{90}\ \text{hr}$$
$$\text{Total time} = \frac{1}{30} + \frac{1}{90} = \frac{3+1}{90} = \frac{4}{90}\ \text{hr}$$
$$\text{Average speed} = \frac{2 \text{ miles}}{4/90 \text{ hr}} = 45\ \text{mph}$$

The arithmetic mean ($60$ mph) is wrong. The harmonic mean of 30 and 90 gives $\frac{2}{\frac{1}{30}+\frac{1}{90}} = 45$ mph. ✅

**SPECmark** uses the **geometric mean** of speedup ratios over all benchmarks, because speedup is a unitless ratio.

---

## 6. Amdahl's Law

> **PYQ CT-II 2021 Q19**: *"Processor 10× faster, computation=40%, I/O=60%, overall speedup = (d) 1.56"* ✅

Amdahl's Law gives the **maximum overall speedup** when only a fraction $P$ of execution time is improved by factor $S$:

$$\text{Speedup} = \frac{1}{(1-P) + \dfrac{P}{S}}$$

Where:
- $P$ = fraction of execution time affected by the optimization
- $S$ = speedup of that fraction
- $(1-P)$ = fraction of execution time **not** affected (the bottleneck)

### Solving the PYQ:

Processor is 10× faster. It only affects the **computation** portion (40%). The I/O portion (60%) is unaffected.

$$P = 0.4,\quad S = 10,\quad (1-P) = 0.6$$

$$\text{Speedup} = \frac{1}{0.6 + \dfrac{0.4}{10}} = \frac{1}{0.6 + 0.04} = \frac{1}{0.64} \approx 1.5625 \approx \mathbf{1.56}$$

### Key Implications of Amdahl's Law:

The **serial (unaffected) fraction dominates**. Even with infinite speedup of the parallel part:

$$\text{Speedup}_{\max} = \lim_{S \to \infty} \frac{1}{(1-P) + \frac{P}{S}} = \frac{1}{1-P}$$

Examples:
- If 25% of code is optimized with $S = 2\times$: Speedup $= 1/(0.75 + 0.125) = 1.14\times$
- If 25% of code is optimized with $S = \infty$: Speedup $= 1/0.75 = 1.33\times$
- If 10% is serial (90% parallel), max speedup with $N$ threads: $1/(0.1 + 0.9/N)$
  - At $N = \infty$: max speedup = $10\times$
- If 1% is serial: max speedup = $100\times$

### Amdahl's Law for Parallelization:

$$\text{Speedup} = \frac{1}{(1-P) + \dfrac{P}{N}}$$

Where $P$ = fraction of parallelizable code, $N$ = number of threads.

**Strong scaling:** Same problem, more processors → runs faster. Gets harder as serial portion becomes the bottleneck.

**Weak scaling:** Increase problem size proportionally with processors. Works in many scientific/internet domains.

---

## 7. Little's Law

A fundamental result from queuing theory, applicable to any **steady-state system** (average arrival rate = average departure rate):

$$L = \lambda W$$

Where:
- $L$ = average number of items in the system
- $\lambda$ = average arrival rate (items/second)
- $W$ = average wait time (latency) per item

No assumptions needed about arrival/service distributions or service order (FIFO, LIFO, etc.).

**To get high throughput $\lambda$, you need either:**
- Small $W$ (low latency per request), or
- Large $L$ (many requests in flight simultaneously — parallelism)

**Applications in computing:**
- Sizing the queue of L1/L2/L3 cache misses in flight
- Sizing outstanding network requests per machine or datacenter
- Calculating average latency from throughput and concurrency measurements

**Example:** A web server handles 1,000 requests/second ($\lambda = 1000$) and has 50 requests in flight on average ($L = 50$). Average latency: $W = L/\lambda = 50/1000 = 0.05$ seconds.

---

## 8. Benchmarks

**Why benchmarks?** "Performance" of a chip means nothing without an associated workload.

**Benchmark:** A standardized workload used to compare performance across machines. Must be representative of actual programs people run.

**Micro-benchmark:** Tiny program isolating one aspect of performance (e.g., binary tree search, towers of hanoi). Not representative of real application complexity.

### SPECmark (SPEC CPU 2006/2017)

- Industry-standard CPU benchmark suite
- **Latency SPECmark procedure:**
  1. Run each benchmark, take odd number of samples, choose **median**
  2. Compute speedup = reference machine time / your machine time
  3. Aggregate across all benchmarks using **geometric mean** of speedups
- **Throughput SPECmark:** Run multiple benchmarks in parallel on a multi-processor system

### Why Geometric Mean for SPECmark?

Speedup ratios are unitless. The geometric mean is the correct average for ratios because it ensures consistency regardless of which machine is chosen as the reference. The arithmetic mean of ratios is skewed by the choice of reference.

---

## 9. Roofline Model

Used to determine the **current bottleneck** of a kernel: is it compute-bound or memory-bound?

**Operational Intensity** = Flops performed per byte of memory traffic (Flops/byte).

The model plots **attainable performance** (GFlops/sec) vs. **operational intensity**:

$$\text{Attainable GFlops/s} = \min(\text{Peak Compute},\ \text{Peak Memory BW} \times \text{Operational Intensity})$$

- **Left of ridge point** (low operational intensity) → **memory-bound**: performance scales with memory bandwidth
- **Right of ridge point** (high operational intensity) → **compute-bound**: performance capped by peak FP performance
- **Ridge point** = operational intensity where memory bandwidth × intensity = peak compute

A kernel plotted below the roofline has optimization headroom; one touching the roofline is at the hardware limit.

---

## 10. Performance Rules of Thumb

**Peak ≠ Actual:** "Peak performance is the performance you are guaranteed not to exceed." Real programs fall short due to cache misses, branch mispredictions, limited ILP, etc. Design for sustained performance.

**Bandwidth is easier to buy than latency:** Adding more buses/channels scales bandwidth; reducing the fundamental latency of a component is much harder.

**Build a balanced system:** Don't over-optimize one component. System performance is often determined by the slowest component (this is just Amdahl's Law applied to systems).

---

## PYQ Answer Summary

| Q | Answer | Key derivation |
|---|--------|---------------|
| CT-II 2021 Q18 | **(d)** all of the above | Architecture = ISA + Organization + Hardware |
| CT-II 2021 Q19 | **(d)** 1.56 | $1/(0.6 + 0.4/10) = 1/0.64 = 1.5625$ |