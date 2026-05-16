# Computer Architecture — Energy Efficiency & Green Computing
### (PYQ-Targeted Notes)

---

## 1. Power Dissipation in CMOS Devices

Every question about power gating, clock gating, leakage, or DVFS traces back to the fundamental CMOS power equation. You must know this cold.

### Total Average Power

$$P_{avg} = P_{dynamic} + P_{leakage}$$

### Dynamic (Switching) Power

$$P_{dynamic} = A \cdot C \cdot V^2 \cdot f$$

Where:
- $A$ = **Activity factor** — how frequently wires switch between 0 and 1. If a wire never switches, it contributes zero dynamic power.
- $C$ = **Node capacitance** — depends on wire lengths and transistor sizes.
- $V$ = **Supply voltage**
- $f$ = **Clock frequency**

Dynamic power is only dissipated **when switching happens**. When a circuit is idle (no switching), dynamic power goes to zero.

### Leakage Power

Leakage is **continuous** — it is dissipated even when the transistor is supposed to be off.

The physical cause: transistors do not switch perfectly. Below their threshold voltage $V_{th}$, they still allow a small current to flow — this is called **sub-threshold leakage**, and it is the dominant leakage component.

**What determines leakage power:**

$$P_{leakage} \propto f(V,\ V_{th},\ T,\ \text{transistor size})$$

The four factors are:
1. **Supply voltage $V$** — higher voltage increases leakage
2. **Switching threshold voltage $V_{th}$** — higher threshold reduces leakage but also slows the transistor
3. **Temperature $T$** — higher temperature exponentially increases leakage (sub-threshold current is thermally activated)
4. **Transistor size** — smaller transistors have less physical material to block leakage, so at sub-10nm nodes, leakage dominates

> **PYQ 2025 Q44:** "In a CMOS device, the supply voltage V, the switching threshold voltage, temperature, and transistor size determine..."
> **Answer: (b) leakage power.**
> Dynamic power is determined by $A, C, V, f$ — not threshold voltage or temperature. Leakage depends on exactly those four: $V$, $V_{th}$, temperature, and transistor size.

---

## 2. Clock Gating

**What it does:** Removes (gates) the clock signal from a functional unit when that unit is not being used.

**What it saves:** Dynamic power. When the clock is removed, the flip-flops in that unit stop switching. Since $P_{dynamic} = A \cdot C \cdot V^2 \cdot f$, by setting the effective activity factor $A = 0$ for idle units, switching power drops to zero.

**What it does NOT do:** It does not cut the supply voltage $V_{dd}$ to the unit. The circuit is still powered. Therefore, **leakage current continues to flow**.

**Where it applies:** Any granularity — individual flip-flops, execution units, entire subsystems (L2 cache, memory controller).

**Cost:** Extra logic must be added to detect idleness and gate the clock. This adds area and a small amount of complexity.

**Summary:**
- Clock gating → eliminates dynamic power during idle periods
- Leakage power continues → circuit is still powered, just not clocking

---

## 3. Power Gating

**What it does:** Physically disconnects the supply voltage $V_{dd}$ from a functional unit using a **header or footer transistor** (sometimes called a "sleep transistor").

**Mechanism:**
- A "sleep" signal is asserted → the transistor turns off → voltage to the block is cut
- When the block is needed again → "sleep" is de-asserted → voltage is restored

**What it saves:** Both dynamic power (no switching) AND leakage power (no supply voltage means no sub-threshold current path).

**Cost:**
- The state of the gated unit is **lost** when power is cut. The processor must save any essential state before power gating, and restore it on wake-up.
- There is a **latency penalty** and **energy overhead** for the power-on/off transitions themselves.
- Requires special **state-retention flip-flops** if some state must be preserved at very low power.

**Where it applies:** Larger functional units where the idle period is long enough to justify the save/restore overhead. Examples: entire CPU cores, graphics units, memory controllers.

> **PYQ 2025 Q41:** "The technique to reduce Power Consumption by shutting off the current to blocks of the circuit that are not in use is called..."
> **Answer: (c) Power Gating.**
>
> - Clock Gating = removes the clock, not the supply current. Leakage continues.
> - DVFS = scales voltage and frequency, but does not shut off current.
> - Power Gating = shuts off the supply current entirely using a transistor switch.

### Comparison Table

| Property | Clock Gating | Power Gating |
|---|---|---|
| What is removed | Clock signal | Supply voltage $V_{dd}$ |
| Dynamic power saved | Yes (no switching) | Yes |
| Leakage power saved | No | Yes |
| State preserved | Yes | No (must save/restore) |
| Wake-up latency | Very low | Higher |
| Granularity | Fine-grained | Coarser units |

---

## 4. Dynamic Voltage and Frequency Scaling (DVFS)

From $P_{dynamic} = A \cdot C \cdot V^2 \cdot f$:
- Voltage and frequency can be **scaled together** at runtime based on workload demand.
- Reducing $V$ by half and $f$ by half reduces dynamic power by a factor of $2^2 \times 2 = 8$ (cubic relationship).
- Frequency and voltage are coupled: higher $f$ requires higher $V$ to maintain reliable switching.

**DVFS** is distinct from clock gating or power gating — it does not idle the circuit. It runs the circuit at a lower operating point when full performance is not required.

---

## 5. System-Level Power Management: Race-to-Idle vs Crawl-to-Halt

These are two opposing philosophies for how to handle a workload when the system is not at full load.

### Race-to-Idle (also called Race-to-Halt)

**Philosophy:** Run at the **highest** possible frequency/voltage. Complete the task as fast as possible. Then immediately drop to the **lowest** power state (power gate cores, deep idle).

**Why this saves energy:**
- The high-power active period is very short.
- Static/leakage power accumulates over time. If you finish in 1 second instead of 3 seconds, you spend 2 fewer seconds paying leakage.
- Net energy = (high power × short time) + (near-zero power × long sleep) is often less than (medium power × long time).

**When it works best:** When static/leakage power is significant relative to dynamic power, and when deep sleep states are accessible quickly.

> **PYQ 2025 Q45:** "A System level power and thermal management scheme keeps the system in its highest operating state in order to complete the workload as fast as possible and then go to sleep at its lowest operating state. Such a scheme is called..."
> **Answer: (d) Race-to-idle.**

### Crawl-to-Halt

**Philosophy:** Run at the **lowest** possible frequency/voltage to extend battery life (e.g., IoT, wearables). The workload runs slower, but power draw is minimized continuously.

**When it works best:** Battery-constrained devices where extending usage time matters more than completing tasks quickly.

**Key insight:** Most modern systems use a **combination** of these two strategies depending on the workload and system constraints.

---

## 6. GPU Architecture: Streaming Multiprocessors and ALU Density

### GPU Internal Structure

A GPU is composed of many small processors. In **NVIDIA GPUs**, the processing units are called **Streaming Multiprocessors (SMs)**.

Inside each SM:
- Many **CUDA cores** (arithmetic units / ALUs) — typically 32 to 128 per SM depending on architecture.
- A shared L1 cache and shared memory.
- A warp scheduler (dispatches groups of 32 threads simultaneously).
- No out-of-order execution logic, no branch predictor per-thread (the hardware hides latency through thread switching instead).

> **PYQ 2025 Q42:** "For NVIDIA GPUs, the cores are called..."
> **Answer: (b) streaming multiprocessors.**
> Note: AMD calls their equivalent units "compute units."

### Why GPUs Outperform CPUs on Parallel Workloads

A CPU devotes significant die area to: out-of-order execution logic, branch predictors, large caches, prefetchers, speculative execution hardware. This control logic is designed to speed up **single-thread sequential** performance.

A GPU makes the opposite tradeoff: it devotes the majority of die area to **ALUs (arithmetic logic units)**, and uses much simpler control — no per-thread branch prediction, no out-of-order engine per thread.

The result:
- A modern GPU can have thousands of ALUs working in parallel.
- For workloads that are embarrassingly parallel (matrix multiply, convolution, rendering), the GPU's raw arithmetic throughput far exceeds a CPU.
- For workloads with lots of branches, data-dependent control flow, or serial dependencies — CPUs win.

> **PYQ 2025 Q43:** "GPUs can obtain improved performance over superscalar out-of-order CPUs on highly parallel workloads by dedicating a larger fraction of their die area to..."
> **Answer: (a) arithmetic logic units.**
> The key tradeoff is: CPUs allocate die area to control logic (for ILP and speculation). GPUs allocate die area to ALUs (for throughput). The simplicity of GPU control logic is not a weakness — it is what frees up die area for more compute.

---

## 7. The Greenhouse Gas (GHG) Protocol — Carbon Emission Scopes

This is the industry-standard accounting framework for carbon emissions. Every major tech company (AMD, Apple, Google, Intel, Microsoft, TSMC) reports using this framework.

Emissions are divided into three scopes based on where they originate.

### Scope 1 — Direct Emissions

These are emissions that **come directly from the organization's own operations**.

Sources:
- Fuel combustion (diesel generators, natural gas for heating)
- Refrigerants in offices and data centers (refrigerant leakage has high GHG impact)
- Transportation using company-owned vehicles
- **Burning of chemicals and gases in semiconductor manufacturing** — this is the key one for chip fabs. TSMC reports that burning perfluorocarbons (PFCs) and other chemicals during wafer etching contributes nearly 30% of Scope 1 emissions.

For **chip manufacturers** (TSMC, Intel, GlobalFoundries): Scope 1 is a large fraction because of the chemicals used in fabrication.
For **mobile vendors** and **data center operators**: Scope 1 is a small fraction.

### Scope 2 — Indirect Emissions from Purchased Energy

These are emissions from the **electricity or heat** that the organization buys.

- Semiconductor fabs consume enormous electricity → Scope 2 is critical for fabs. Energy consumption produces over 63% of emissions from manufacturing 12-inch wafers at TSMC.
- Data centers consume massive electricity → Scope 2 is historically their largest category.
- The magnitude depends on **carbon intensity** of the local energy grid (grams of CO₂ per kWh):
  - Coal: ~820 g CO₂/kWh
  - Wind: ~11 g CO₂/kWh
  - Solar: ~41 g CO₂/kWh
- When data centers switch to renewable energy, Scope 2 drops dramatically.

### Scope 3 — Supply Chain Emissions (Upstream + Downstream)

Everything else — the full supply chain in both directions.

Upstream (things the company buys):
- **Hardware manufacturing** — when a data center buys thousands of CPUs, the carbon from making those chips counts as the data center operator's Scope 3.
- Raw material extraction (cobalt, lithium, copper, etc.)
- Construction of facilities
- Employee commuting

Downstream (things the company sells):
- Use of sold products (e.g., Apple counts the electricity users consume running iPhones as Apple's Scope 3 downstream)
- End-of-life recycling

For technology companies, Scope 3 is the **dominant** category once renewable energy is adopted for operations:
- Google (2018): Scope 3 was 21× larger than Scope 2
- Facebook (2019): Scope 3 was 23× larger than Scope 2

> **PYQ 2025 Q35:** "Direct emissions coming from fuel combustion, refrigerants in offices and data centres, transportation, and the use of chemicals and gases in semiconductor manufacturing are categorized by the Greenhouse Gas Protocol as..."
> **Answer: (a) Scope 1.**
> The question lists exactly the definition of Scope 1: these are all direct emissions from the organization's own activities.

### Scope Summary Table

| Scope | Type | Key Sources for Tech Companies |
|---|---|---|
| Scope 1 | Direct | PFC burning in fabs, diesel generators, refrigerant leaks, company vehicles |
| Scope 2 | Purchased energy | Electricity for fabs, data centers, offices |
| Scope 3 | Supply chain | Hardware manufacturing (bought CPUs, servers), construction, employee commute, product use by customers |

---

## 8. Reducing Carbon from Hardware Manufacturing

### The Key Insight

Historically, most carbon from computing came from **operational energy** (running the hardware). As systems become more energy efficient and data centers adopt **renewable energy**, Scope 2 (operational) emissions collapse.

The result: **hardware manufacturing becomes the dominant carbon source**.

Example: iPhone carbon breakdown:
- iPhone 3GS (2008): 49% manufacturing, 51% use
- iPhone 11 (2019): 86% manufacturing, 14% use

This shift happened because energy efficiency improved, but manufacturing still requires the same complex, energy-intensive processes.

### How to Reduce Manufacturing Carbon

> **PYQ 2025 Q36:** "Carbon emissions from hardware manufacturing can be reduced by..."
> **Answer: (b) using renewable energy for semiconductor fabrication units.**

The reasoning: Semiconductor fabrication consumes enormous amounts of electricity (Scope 2) and burns chemicals (Scope 1). The largest lever for reducing manufacturing carbon is powering fabs with renewable energy — solar, wind, hydropower — which have carbon intensities orders of magnitude lower than coal or gas.

**Quantitative example from TSMC data:** A 64× improvement in renewable energy penetration reduces overall wafer manufacturing carbon by approximately 2.7×. Even with 100% wind energy replacing coal, manufacturing carbon does not go to zero because Scope 1 (PFC burning, chemicals) and material extraction remain.

**Other approaches mentioned in the literature:**
- Novel fabrication materials that require less chemical processing
- Yield improvement (fewer wasted wafers means less energy per good chip)
- Extending hardware lifetime (amortizing the manufacturing carbon over more years of use)
- Leaner hardware designs (fewer transistors = less manufacturing energy)

---

## 9. PYQ Answer Summary

| Year | Q# | Topic | Answer |
|---|---|---|---|
| 2025 | 35 | GHG Scope 1 definition | (a) Scope 1 |
| 2025 | 36 | Reduce manufacturing carbon | (b) Renewable energy for fabs |
| 2025 | 41 | Technique that shuts off current to idle blocks | (c) Power Gating |
| 2025 | 42 | NVIDIA GPU core name | (b) Streaming Multiprocessors |
| 2025 | 43 | Why GPUs beat CPUs on parallel work | (a) ALUs (more die area to arithmetic) |
| 2025 | 44 | What determines leakage power in CMOS | (b) leakage power |
| 2025 | 45 | Run fast, finish, then sleep deeply | (d) Race-to-idle |