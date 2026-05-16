# Domain-Specific Hardware Accelerators (DSAs)

> **Paper**: Dally, Turakhia, Han — ACM July 2020
> **Core thesis**: DSAs gain *efficiency* from specialization and *performance* from parallelism.

---

## 1. Why DSAs Exist — The Motivation

General-purpose CPUs execute sequences of simple instructions. The energy required to **fetch and interpret** one instruction is **10× to 4000×** more than the energy to actually **perform** the underlying operation (e.g., an ADD).

This overhead was tolerable when Moore's Law held — transistor density doubled every ~2 years, so performance scaled automatically. **Moore's Law has largely ended.** DSAs are one of the few remaining paths to continued scaling of performance and efficiency.

---

## 2. Special-Purpose Processors — DSPs, ASICs, FPGAs

> **PYQ 2025 Q40**: *"Digital Signal Processors (DSPs), ASICs, FPGAs are all special purpose processors that — (b) optimize power by offloading processing from CPU"* ✅

DSPs, ASICs, and FPGAs are all **special-purpose processors**. Their common property is that they **offload specific workloads from the CPU**, reducing the CPU's power burden and performing the offloaded task far more efficiently.

### Comparison Table

| Platform | Efficiency | Programmability | NRE Cost | Key Property |
|----------|-----------|----------------|----------|--------------|
| **ASIC** | Highest | None (hardwired at design time) | Very high | Logic fixed for one application domain |
| **FPGA** | 10×–100× lower than ASIC | Dynamically reconfigurable | Moderate | Same chip reconfigured for different applications |
| **GPU** | Near-ASIC (for supported domains) | High (CUDA/OpenCL) | Low | Accelerates multiple domains via specialized ops + SIMT |
| **CPU** | Lowest | Highest | Lowest | General purpose; high overhead |

**NRE = Non-Recurring Engineering cost** — the one-time design cost paid regardless of how many units are made.

**CGRA (Coarse-Grain Reconfigurable Architecture):** An intermediate option between FPGA and ASIC. Reconfigurable at word/operator level (not bit level) → lower overhead than FPGAs.

---

## 3. GPU Architecture

> **PYQ 2025 Q42**: *"For NVIDIA GPUs, the cores are called — (b) Streaming Multiprocessors"* ✅

> **PYQ 2025 Q43**: *"GPUs obtain improved performance over superscalar out-of-order CPUs on parallel workloads by dedicating a larger fraction of die area to — (a) arithmetic logic units"* ✅

### 3.1 Structure

A GPU is composed of many cores. In NVIDIA's terminology, these cores are called **Streaming Multiprocessors (SMs)**.

GPUs dedicate a **much larger fraction of die area to ALUs** compared to CPUs. CPUs dedicate a large portion of die area to control logic: branch predictors, out-of-order execution hardware, large caches. GPUs trade away that control complexity for raw arithmetic throughput.

This is why GPUs dramatically outperform CPUs on **highly parallel workloads** — there are simply far more arithmetic units doing work per unit of die area.

### 3.2 SIMT Execution Model

GPUs use a **SIMT (Single Instruction, Multiple Threads)** execution model. Many threads execute the same instruction simultaneously on different data. This offers **order-of-magnitude better efficiency than CPUs** on parallel workloads, at the expense of single-thread performance.

### 3.3 Specialized Instructions in GPUs

GPUs extend their advantage by adding **domain-specific instructions** directly onto the general-purpose GPU:

**HMMA (Half-Precision Matrix Multiply-Accumulate) — NVIDIA Volta V100:**
- Multiplies two $4 \times 4$ half-precision (16-bit) FP matrices, accumulates into a $4 \times 4$ single-precision (32-bit) FP matrix
- One HMMA instruction = **128 floating-point operations** (64 half-prec. multiplies + 64 single-prec. adds)
- Energy breakdown: **77% consumed by arithmetic**, only 23% on instruction overhead
- Accelerates training and inference for convolutional, fully-connected, and recurrent neural network layers

**IMMA (Integer Matrix Multiply-Accumulate) — NVIDIA Turing:**
- Operates on $8 \times 8$ 8-bit integer matrices, accumulates into $8 \times 8$ 32-bit integer result
- One IMMA instruction = **1,024 integer operations**
- Energy breakdown: **87% consumed by arithmetic**, only 13% overhead

A dedicated accelerator (e.g., Google TPU) could be at most **23% (HMMA)** or **13% (IMMA)** more efficient than these GPU instructions on the core matrix multiply — the GPU nearly matches full-custom hardware efficiency via complex instructions.

### 3.4 Efficiency Numbers (Deep Learning vs. CPU)

For deep learning (ResNet-50 inference, Images/s-Watt):

| Platform | Efficiency |
|----------|-----------|
| CPU | 9.9 |
| FPGA | 70.6 |
| GPU | 150 |
| ASIC | 16,279 |

For domains where GPUs have **specialized logic** (e.g., deep learning with HMMA), they provide **near-ASIC efficiency**. For domains without GPU-specific instructions (e.g., genomics), GPUs may be *less* efficient than FPGAs.

---

## 4. CMOS Power: Dynamic vs. Leakage

> **PYQ 2025 Q44**: *"In a CMOS device, the supply voltage V, the switching threshold voltage, temperature, and transistor size determine — (b) leakage power"* ✅

In a CMOS device there are two components of power:

### 4.1 Dynamic Power

$$P_{dynamic} \propto \alpha \cdot C \cdot V^2 \cdot f$$

Where:
- $\alpha$ = activity factor (fraction of transistors switching)
- $C$ = load capacitance
- $V$ = supply voltage
- $f$ = clock frequency

Dynamic power is consumed **only when transistors switch** (i.e., during actual computation). It is determined primarily by supply voltage, capacitance, and switching frequency — **not** by threshold voltage or temperature.

### 4.2 Leakage Power (Static Power)

$$P_{leakage} \propto I_{leak} \cdot V$$

Leakage power is consumed **even when the transistor is not switching** — it flows continuously due to subthreshold conduction and gate oxide tunneling. It is determined by:
- **Supply voltage** $V$
- **Switching threshold voltage** $V_t$ — lower $V_t$ means the transistor is easier to turn on, so more current leaks through even when "off"
- **Temperature** — higher temperature increases subthreshold leakage
- **Transistor size** — more transistors (larger chip / finer geometry) → more aggregate leakage

This is why the PYQ answer is **leakage power**: all four of those parameters ($V$, $V_t$, temperature, transistor size) are the classic determinants of leakage, not dynamic power.

---

## 5. Power Reduction Techniques

> **PYQ 2025 Q41**: *"The technique to reduce Power Consumption by shutting off current to blocks not in use is called — (c) Power Gating"* ✅

> **PYQ 2025 Q45**: *"A scheme that keeps the system in its highest operating state to complete the workload as fast as possible and then goes to sleep — (d) Race-to-idle"* ✅

### 5.1 Dynamic Voltage and Frequency Scaling (DVFS)

Since $P_{dynamic} \propto V^2 \cdot f$, **reducing voltage and frequency** reduces dynamic power cubically (lowering $V$ also allows lowering $f$). Used when full performance is not needed — scales power down with workload demand.

### 5.2 Clock Gating

Disabling the **clock signal** to sections of the circuit that are not currently in use. Since dynamic power requires switching, stopping the clock stops switching and eliminates dynamic power in that block. The circuit retains its state. Does **not** eliminate leakage power.

### 5.3 Power Gating

Physically **cutting off the power supply (current)** to entire blocks of a circuit not in use. This eliminates **both dynamic and leakage power** in the gated block. The state of the block is lost and must be saved before power-off and restored when power returns. More aggressive than clock gating.

### 5.4 Race-to-Idle

A **system-level power and thermal management** strategy: run the processor at its **highest operating state** (maximum performance) to complete the workload as fast as possible, then immediately transition to the **lowest operating state** (deep sleep).

The rationale: a short period at high power followed by a long deep sleep can be more energy-efficient than running at moderate power for a longer total duration, because modern sleep states have extremely low power consumption.

### Summary Table

| Technique | What is disabled | Eliminates |
|-----------|----------------|------------|
| **Clock Gating** | Clock signal to idle blocks | Dynamic power only |
| **Power Gating** | Power supply (current) to idle blocks | Dynamic + Leakage power |
| **DVFS** | Reduces V and f dynamically | Dynamic power (cubically with V) |
| **Race-to-Idle** | System-level: maximize speed, then sleep | Overall energy (minimizes active time) |

---

## PYQ Answer Summary

| Q | Answer | Key Fact |
|---|--------|---------|
| 2025 Q40 | **(b)** optimize power by offloading from CPU | DSPs/ASICs/FPGAs handle specific tasks more efficiently, taking load off CPU |
| 2025 Q41 | **(c)** Power Gating | Shuts off *current* (not just clock) to idle blocks — kills both dynamic and leakage |
| 2025 Q42 | **(b)** Streaming Multiprocessors | NVIDIA GPU cores are called SMs |
| 2025 Q43 | **(a)** arithmetic logic units | GPUs allocate more die to ALUs; CPUs allocate more to control logic |
| 2025 Q44 | **(b)** leakage power | $V$, $V_t$, temperature, transistor size → leakage; dynamic power depends on $V^2$, $C$, $f$ |
| 2025 Q45 | **(d)** Race-to-idle | Run at max, finish fast, sleep long |
