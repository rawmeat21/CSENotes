### Vector Processors

#### The Core Idea

A pipelined functional unit (say, a 3-stage floating-point adder) runs most efficiently when you feed it a long stream of independent, identical operations — one new input per cycle, one result per cycle after the pipeline fills. The problem with scalar code is that dependencies between consecutive instructions constantly break this rhythm.

A **vector** is a one-dimensional array of data. A **vector instruction** is a single instruction that implicitly applies the same operation to every element of the vector. So:

V4←V2+V3V4←V2+V3

is equivalent to:

V4[1]←V2[1]+V3[1]V4[1]←V2[1]+V3[1] V4[2]←V2[2]+V3[2]V4[2]←V2[2]+V3[2] ⋮⋮ V4[n]←V2[n]+V3[n]V4[n]←V2[n]+V3[n]

Each consecutive element pair is independent of the previous. This is exactly the stream a pipelined adder needs — no hazards, no stalls, maximum throughput.

---

### The CRAY X-MP / Model 24 Architecture

#### Overall Structure

The CRAY X-MP Model 24 is a **MIMD** (Multiple Instruction Multiple Data) machine. "Model 24" means 2 CPUs, 4 million words of memory.

Each CPU is split into three physical sections:

**Computation Section** — contains all the registers and functional units. This is where actual work happens.

**Communication Section** — contains 3 clusters of shared registers and semaphores. These are used to coordinate between the two CPUs and manage access to shared memory. Semaphores here are hardware synchronization primitives, not OS-level constructs.

**Control Section** — manages instruction buffers, issues instructions, and controls information flow between other sections.

The two CPUs share a single **main memory** of 4 million words, organized into 32 banks. Each word is 64 data bits + 8 check bits (for error correction). The 32-bank organization allows multiple simultaneous accesses as long as they hit different banks.

There are also **13 pipelined functional units** per CPU, each dedicated to specific integer or floating-point operations.

There are between 2 and 4 **I/O processors** in the system:

- **MIOP** — mandatory, manages front-end interfaces and the console
- **BIOP** — handles transfers between main memory and secondary storage
- **DIOP** — optional, manages additional disk controllers
- **XIOP** — optional, controls block multiplexor channels

---

### The Control Section

The control section manages how instructions move from memory into execution. Think of it as the instruction fetch-and-issue front end.

The key components are:

**Program Address Register (P)** — the program counter. Increments by 1 each cycle, or gets the branch target on a taken branch.

**Instruction Buffers (Buffer0–Buffer3)** — four 64-parcel buffers. Instructions are fetched from memory in blocks into these buffers. This hides memory latency for sequential code.

**NIP (Next Instruction Parcel register)** — holds the parcel that will be issued next cycle.

**CIP (Current Instruction Parcel register)** — holds the parcel currently being decoded and issued. This is where an instruction sits until it can be dispatched for execution.

**LIP (Lower Instruction Parcel register)** — holds the second parcel of a 2-parcel instruction.

**Programmable Clock** — driven by register SjSj​, used for timing.

The flow is: instruction buffer → NIP → CIP → issue to execution units.

---

### Instruction Issue Phase

#### Instruction Lengths

CRAY X-MP instructions are either **16 bits (1 parcel)** or **32 bits (2 parcels)** long. A parcel is always 16 bits.

#### 1-Parcel Instructions

During instruction fetch, the parcel is transferred from the instruction buffer into the **NIP**. This takes one clock cycle.

On the next clock cycle, the parcel moves from the NIP to the **CIP**, where it is decoded. The instruction stays in the CIP until it can be issued — i.e., until the processor is ready to execute it.

Under optimal conditions (no conflicts, no branches, next instruction already in the buffer), the NIP→CIP transfer of one instruction overlaps with the CIP→issue of the previous instruction. This means **one instruction issues per clock cycle** under optimal conditions.

If a previously issued instruction is using a resource that the current instruction also needs, the current instruction cannot issue. It stays in the CIP. This is called a **hold condition**.

#### Branches

When a branch instruction reaches the CIP, it stays there until the branch is resolved — no subsequent instruction can issue. The branch spends its **entire cycle in the issue phase and zero time in the execution phase**. The branch outcome determines what gets loaded into the NIP next.

#### 2-Parcel Instructions

When the first parcel moves from the NIP to the CIP, the second parcel is simultaneously transferred from the instruction buffer to the **LIP**. The NIP is then loaded with a zero (no-operation), so the processor does not attempt to issue an instruction on the next cycle. A 2-parcel instruction therefore occupies the CIP for a **minimum of 2 clock cycles**.

---

### The Scalar Section

#### Registers

The scalar section contains registers **S0–S7**: eight 64-bit scalar registers. These are the general-purpose registers for scalar (non-vector) computation.

There are also intermediate registers **T0–T7** and **T8–T77** (part of a larger set), and status registers.

#### Scalar Functional Units

|Unit|Operation|Stages|
|---|---|---|
|Scalar ADD|64-bit integer add|3|
|Scalar SHIFT|64-bit logical shift|2|
|Scalar SHIFT|128-bit logical shift|3|
|Scalar LOGICAL|64-bit logical|1|
|Scalar POP/PARITY|population or parity count|4|
|Scalar POP/PARITY|leading zero count|3|

Note: scalar floating-point operations are performed in the **vector functional unit**, not here.

#### Output Operand Reservation

When a scalar instruction issues, its **output operand register is reserved**. No other instruction can write to that register until the first instruction completes. This prevents WAW hazards.

Input operands are **not** reserved — other instructions can still read them freely.

This has a direct timing consequence. Consider:

```
Line 1:  S1 = S2 + S3
Line 2:  S1 = S2 + S3   ← same destination
Line 3:  S2 = S1 + S7
Line 4:  S1 = S4 + S5
```

Line 1 issues in cycle 1, reserving S1. Line 2 cannot issue until line 1 completes (S1 reserved). Line 3 cannot issue until line 2 completes (S1 reserved as source, effectively). Line 4 can issue the cycle after line 3 issues because line 3 reserves S2, not S1 — line 4 writes S1 which line 3 doesn't reserve.

#### Pipeline Delay Between Functional Units

If two instructions use the same functional unit and the second issues immediately after the first, their results could emerge in the same cycle — requiring extra hardware to handle multiple results at once. To avoid this, a **1-cycle hold** is inserted when a single-word shift immediately follows a double-word shift. Similarly for leading-zero count following pop/parity count in the scalar pop/parity unit.

---

### The Address Section

#### Registers

**A0–A7** — 8 address registers, each **24 bits** wide. Used for memory addresses in load/store operations, and also as index registers and for short integer computations.

**B0–B77** — 64 registers, each **24 bits** wide. Not directly connected to functional units. Used for saving registers during subroutine calls, or for temporary address storage.

#### Address Functional Units

|Unit|Operation|Stages|
|---|---|---|
|Address ADD|24-bit integer add|2|
|Address MULTIPLY|24-bit integer multiply|4|

#### Memory Addressing

All memory data is stored in 64-bit words. Word addresses are 22 bits wide. Parcel addresses are 24 bits wide (since each word contains 4 parcels of 16 bits each, you need 2 extra bits). A and B register values are interpreted as either 22-bit word addresses or 24-bit parcel addresses depending on the instruction context.

---

### The Vector Section

#### Registers

Each CPU has **8 vector registers, V0–V7**. Each vector register holds **64 elements**, each element being 64 bits wide. So each vector register is 64×64=409664×64=4096 bits total.

**Vector Length Register (VL)** — a scalar register that specifies how many elements of the vector registers to actually use in a vector operation. When a vector instruction issues, VL is read and its value at that moment defines the number of elements processed. This allows vectors shorter than 64 elements.

**Vector Mask** — a 64-bit register where each bit controls whether the corresponding element participates in a masked vector operation.

Since there is no direct path from main memory to the vector functional units, all vector arithmetic goes through vector registers. You first load data from memory into a vector register, operate on it, then store the result back.

#### Vector Functional Units

|Unit|Stages|
|---|---|
|Vector ADD (64-bit integer)|3|
|Vector SHIFT (64-bit logical)|3|
|Vector SHIFT (128-bit logical)|4|
|Full Vector LOGICAL (64-bit)|2|
|Second Vector LOGICAL (64-bit)|4|
|Vector POP/PARITY|5|
|Floating-Point ADD|6|
|Floating-Point MULTIPLY|7|
|Reciprocal Approximation|14|

All vector functional units are pipelined. The source and destination vector registers **and** the functional unit are all reserved for the duration of the operation.

---

### Three Phases of a Vector Operation

Every vector operation goes through exactly three phases:

#### Phase 1: Setup (always 3 cycles)

The functional unit is configured for the operation, and the source/destination routing to vector registers is established. This always takes **3 clock cycles** regardless of which unit or vector length.

#### Phase 2: Execution

One pair of source elements enters the first pipeline stage per cycle. After the pipeline is filled, one result emerges per cycle. This phase lasts exactly **VL cycles** (one per element).

The source registers are reserved for the duration of setup + execution = VL+3VL+3 cycles. After the last pair of source elements enters the pipeline (at the end of setup + execution), the source registers are freed.

The functional unit itself becomes available **one cycle after** the last pair of source elements enters — so the unit is free after 3+VL+1=VL+43+VL+1=VL+4 cycles.

#### Phase 3: Shutdown (always 3 cycles)

After the last element exits the pipeline, there is a 3-cycle shutdown phase before the destination register becomes available for another operation.

So the **destination register** becomes available after:

3+VL+(VL−1)+3=n+VL+5 cycles3+VL+(VL−1)+3=n+VL+5 cycles

Wait — let me be precise using the notes. The shutdown time is 3 cycles. The last result emerges n+(VL−1)n+(VL−1) cycles after issue (setup=3, then VL elements, first result at cycle 3+n3+n where nn is pipeline stages, last result at 3+n+VL−13+n+VL−1). Then 3 more cycles of shutdown. So destination register available at:

3+n+(VL−1)+3=n+VL+5 cycles after issue3+n+(VL−1)+3=n+VL+5 cycles after issue

where nn = number of pipeline stages of that functional unit.

---

### Example 1: Simple Vector Operation (VL = 53)

```
Line 1:  A1  53        Set A1 = 53
Line 2:  VL  A1        Set vector length = 53
Line 3:  V4  V2+V3     Integer vector add
```

Vector integer add has **3 pipeline stages**.

- Setup: 3 cycles
- First result emerges: 3+3=63+3=6 cycles after issue of line 3
- Remaining 52 results: 52 more cycles
- Shutdown: 3 cycles
- **Destination V4 available after:** 3+53+5=613+53+5=61 cycles

**Functional unit reusable after:** VL+4=53+4=57VL+4=53+4=57 cycles

**Source registers V2 and V3 reserved for:** VL+3=53+3=56VL+3=53+3=56 cycles

---

### Example 2(a): No Source Reservation Conflict (VL = 10)

```
Line 3:  V4 = V3 + FV2    (floating-point add, 6 stages)
Line 4:  V6 = V5 * FV7    (floating-point multiply, 7 stages)
```

Lines 3 and 4 use completely different source and destination registers. Line 4 can issue immediately after line 3 — no conflict. Both execute in parallel.

---

### Example 2(b): Source Reservation Conflict (VL = 10)

```
Line 3:  V4 = V3 + FV2    (floating-point add, 6 stages)
Line 4:  V6 = V3 * FV7    (floating-point multiply, 7 stages)
```

Line 4 wants to read **V3**, which is reserved by line 3 for VL+3=13VL+3=13 cycles (cycles 4 through 16 in the timing diagram). Line 4 cannot issue until line 3 releases V3. Line 3 issues in cycle 3, so V3 is reserved through cycle 16. Line 4 can therefore only issue in cycle **17**.

---

### Vector Chaining

Chaining allows the results of one vector operation to feed into a subsequent vector operation **as they are produced**, without waiting for the entire first operation to complete and the destination register to be released.

Specifically: a value that emerges from the pipeline can be used by a waiting operation **2 cycles after it is produced**.

This is fundamentally different from unchained execution where the destination register cannot be accessed until 3 cycles after the last computation (shutdown phase).

#### Chaining Example (VL = 10)

```
Line 3:  V4 = V3 + FV2    (floating-point add, 6 stages)
Line 4:  V5 = V4 * FV7    (floating-point multiply, 7 stages)
```

Line 4 depends on V4, the output of line 3. Without chaining, line 4 would have to wait until V4 is fully available — 6+10+5=216+10+5=21 cycles after line 3 issues. With chaining, line 4 can begin 2 cycles after the **first result** of line 3 emerges.

In the timing diagram: line 3 issues at cycle 3, setup takes 3 cycles, first result of line 3 emerges at the end of cycle 3+3+6−1=113+3+6−1=11 (setup 3 + first element through 6 stages). Line 4 begins chained execution 2 cycles later — it starts issuing its elements from cycle 13 or so, producing its first result shortly after.

The key rule: **chaining is an exception to the n+VL+5n+VL+5 destination availability rule**. The destination can be read element-by-element as results emerge.

---

Now, ready for the PYQs on this topic? There are questions from 2025, CT-1 2024, CT-II 2021, and 2022 all hitting these timing calculations directly.

## You said: Yes yes

Yes yes

9:44 PM

## Claude responded: Let me go through every CRAY X-MP PYQ systematically.

Let me go through every CRAY X-MP PYQ systematically.

---

### 2022 Paper — Questions 47–51

The 2022 paper has a descriptive question about a CRAY X-MP program. From the context of questions 47–51, the program is:

```
Line 1:  A1  53        Set A1 = 53
Line 2:  VL  A1        Set vector length = 53
Line 3:  V4  V2+V3     Vector integer add (3 stages)
```

---

**Q47. The first result (corresponding to the first element of the 53-element vector) emerges after:**

Options: (a) 3 cycles (b) 6 cycles (c) 9 cycles (d) 61 cycles

The first result emerges after setup + pipeline stages = 3+3=63+3=6 cycles after line 3 issues.

Setup is always 3 cycles. Vector integer add has 3 pipeline stages. After setup, the first element pair enters the pipeline. It takes 3 more cycles to traverse all stages. So first result = cycle 6 after issue of the vector instruction.

**Answer: (b) 6 cycles**

---

**Q48. The functional unit (vector adder) can be reused after:**

Options: (a) 52 cycles (b) 56 cycles (c) 57 cycles (d) 61 cycles

The functional unit is free VL+4VL+4 cycles after issue.

53+4=57 cycles53+4=57 cycles

Why VL+4VL+4? The unit is busy during setup (3 cycles) + execution (VL cycles). One cycle after the last element pair enters the pipeline, the unit is free. That's 3+VL+1=VL+43+VL+1=VL+4.

**Answer: (c) 57 cycles**

---

**Q49. The B registers (B0..B77) of the CRAY X-MP have a bit-length of:**

Options: (a) 8 (b) 16 (c) 24 (d) 32

Directly from the notes. B registers are **24-bit** registers.

**Answer: (c) 24**

---

**Q50. During instruction fetch of a 1-parcel instruction, the parcel is first transferred to:**

Options: (a) CIP (b) LIP (c) NIP (d) none

The fetch sequence is: instruction buffer → **NIP** first, then on the next cycle NIP → CIP.

**Answer: (c) NIP**

---

**Q51. How long would the program take to execute (issue-time + execution-time)?**

This is a descriptive/numerical question. The program is lines 1–3 above.

Lines 1 and 2 are scalar instructions (1-parcel each), each taking 1 cycle to issue under optimal conditions. Line 3 is the vector instruction.

- Line 1 issues: cycle 1
- Line 2 issues: cycle 2
- Line 3 issues: cycle 3

Destination register V4 becomes available after n+VL+5=3+53+5=61n+VL+5=3+53+5=61 cycles from when line 3 issues.

Total program execution time = **61 cycles** (measuring from start to when the final result is stored in V4).

---

### CT-II 2021 — Questions 9, 10, 11, 21

The program given in CT-II 2021:

```
Line 1:  A1  10        Set A1 = 10
Line 2:  VL  A1        Set vector length = 10
Line 3:  V4  V3+FV2    Floating-point add (6 stages)
Line 4:  V5  V4*FV7    Floating-point multiply (7 stages), CHAINED from line 3
```

Lines 1 and 2 are scalar, issuing in cycles 1 and 2. Line 3 issues in cycle 3.

---

**Q9. The first result of line 3 emerges at the end of:**

Options: (a) cycle-7 (b) cycle-12 (c) cycle-13 (d) cycle-21

Line 3 issues at cycle 3. Setup = 3 cycles (cycles 3, 4, 5 — the instruction occupies the issue phase during cycle 3, and setup runs). First element enters pipeline at start of cycle 6 (after 3 setup cycles). Floating-point add = 6 stages. First result emerges after 3+6=93+6=9 cycles from issue...

Let me be careful. Line 3 issues at cycle 3. Setup takes 3 cycles: cycles 3, 4, 5. First element pair enters the first pipeline stage at cycle 6. After 6 stages, it exits at the end of cycle 6+6−1=116+6−1=11...

Actually using the formula from the notes directly: first result = setup + pipeline stages = 3+6=93+6=9 cycles **after issue**. Line 3 issues at cycle 3, so first result at end of cycle 3+9=123+9=12.

Wait — the notes say for VL=53 with 3-stage add: "first result emerges after 3+3=63+3=6 cycles." That means it's 6 cycles after the instruction issues. Issue is at cycle 3 (for line 3 in example 1), so first result is at end of cycle 3+6=93+6=9... but the notes say "first result emerges after 3+3=63+3=6 cycles" and "Destination V4 available after 61 cycles." Let me recount:

The notes state the formula directly: first result = 3+n3+n cycles after issue, where nn = number of pipeline stages. For floating-point add, n=6n=6. Line 3 issues at cycle 3.

First result at end of cycle 3+(3+6)=3+9=12First result at end of cycle 3+(3+6)=3+9=12

Hmm, but that gives cycle 12. Let's verify with example 1: n=3n=3, issues at cycle 3. First result at 3+(3+3)=93+(3+3)=9? But notes say 6 cycles = cycle 9 from start. Actually the notes say "first result emerges after 3+3=63+3=6 cycles" meaning 6 cycles after the vector instruction issues — not 6 cycles from program start. If line 3 issues at cycle 3, first result is at cycle 3+6=93+6=9 for example 1. That's consistent.

For CT-II 2021, line 3 issues at cycle 3, n=6n=6: first result at cycle 3+(3+6)=123+(3+6)=12.

**Answer: (b) cycle-12**

---

**Q10. Line 4 begins chained execution in:**

Options: (a) cycle-12 (b) cycle-13 (c) cycle-14 (d) cycle-15

With chaining, line 4 can begin 2 cycles after the first result of line 3 emerges. First result of line 3 is at end of cycle 12. So line 4 can begin chained execution at cycle 12+2=1412+2=14.

But there's also the issue of line 4 needing to go through its own setup (3 cycles) before execution. The chaining rule says: a result emerging from the pipeline can be used 2 cycles later. So line 4's setup must complete such that its first execution cycle aligns with cycle 14.

Line 4 begins chained execution (i.e., its first element enters its pipeline) at cycle **14**.

**Answer: (c) cycle-14**

---

**Q11. The first result of line 4 emerges at the end of:**

Options: (a) cycle-15 (b) cycle-20 (c) cycle-21 (d) cycle-24

Line 4 is floating-point multiply with 7 stages. Its first element enters at cycle 14. After 7 stages, first result emerges at end of cycle 14+7−1=2014+7−1=20.

**Answer: (b) cycle-20**

---

**Q21. Draw a timing diagram for the CRAY X-MP program (descriptive)**

This is the full diagram for the chaining example above. The key points to state:

Line 3 (FP add, 6 stages): issues cycle 3, setup cycles 3–5, execution cycles 6–15 (10 elements), first result end of cycle 12, last result end of cycle 17, shutdown cycles 18–20, V4 available cycle 21.

Line 4 (FP multiply, 7 stages): chains from line 3. First element enters cycle 14 (2 cycles after first result of line 3). Execution cycles 14–23, first result end of cycle 20, last result end of cycle 27.

In at most 5 sentences: Line 3 issues at cycle 3 with a 3-cycle setup, so the first floating-point add result emerges at the end of cycle 12. Line 4 chains from line 3, meaning it begins processing elements 2 cycles after the first result of line 3, i.e., from cycle 14. Line 4 is a 7-stage floating-point multiply, so its first result emerges at the end of cycle 20. The chaining mechanism allows line 4 to begin before line 3 completes, overlapping the multiply pipeline with the tail of the add pipeline. Without chaining, line 4 would have to wait until V4 was fully released, which would be at cycle 3+6+10+5=243+6+10+5=24, significantly later.

---

### 2025 Paper — Questions 9–12

The 2025 paper scenario involves a 2-parcel instruction and then a vector chaining program. From the questions:

**Q9. For a two-parcel instruction, when the first parcel moves from the NIP to the CIP, the second parcel is transferred to the:**

Options: (a) Program Address Register (b) NIP (c) LIP (d) Vector Length Register

This is directly from the notes on 2-parcel instruction issue. When the first parcel moves NIP→CIP, the second parcel comes from the instruction buffer into the **LIP**.

**Answer: (c) LIP**

---

**Q10. For a scalar operation:**

Options: (a) input operands are reserved (b) output operands are reserved (c) both input and output operands are reserved (d) neither input nor output operands are reserved

Directly from the notes: "Scalar output operands are reserved." Input operands are NOT reserved.

**Answer: (b) output operands are reserved**

---

**Q11. Line-4 starts execution in:**

The 2025 paper has a chaining program. From the context of questions 11 and 12, the program appears to be:

```
Line 1:  A1  10
Line 2:  VL  A1        VL = 10
Line 3:  V4  V3+FV2    FP add (6 stages), issues cycle 3
Line 4:  V5  V4*FV7    FP multiply (7 stages), chained
```

This is the same structure as CT-II 2021. Line 4 begins chained execution at cycle 14.

Options: (a) cycle-5 (b) cycle-25 (c) cycle-14 (d) cycle-17

**Answer: (c) cycle-14**

---

**Q12. The first element of V4 (result of line 3) is available at the end of:**

Wait — Q12 says "first element of V4" which is the output of line 3 (FP add). That's the first result of line 3 = end of cycle 12.

Options: (a) cycle-12 (b) cycle-21 (c) cycle-26 (d) cycle-35

**Answer: (a) cycle-12**

---

### CT-1 2024 — Questions 4, 5, 8

The CT-1 2024 paper also has vector questions. From the context:

```
Line i:   (some scalar setup)
Line i+1: (vector instruction, possibly with branch scenario)
Line j:   (branch)
Line j+1: (instruction after branch)
Line i4:  V6 = V5 + FV7    (floating-point add, 6 stages, VL=?)
Line i5:  (next instruction)
...
```

From questions 4 and 5 about branch timing, this seems to be a mix of branch + vector questions.

---

**Q4. If the branch IS TAKEN, instruction i+1 enters EX stage in:**

Options: (a) cycle-4 (b) cycle-5 (c) cycle-6 (d) none

A branch in the CRAY X-MP stays in the CIP until resolved — no succeeding instruction issues during that time. If the branch is at instruction ii, it occupies the issue phase for its full cycle. Once resolved as taken, the next instruction fetched is the branch target. It takes time to fetch from the new address, go through NIP, then CIP, then issue. That's at minimum 2 cycles after the branch resolves before the taken target instruction begins execution.

If the branch issues (and resolves) at cycle 3, the target instruction enters NIP at cycle 4, CIP at cycle 5, issues at cycle 5, enters EX at cycle **6**.

**Answer: (c) cycle-6**

---

**Q5. If the branch IS TAKEN, instruction j+1 enters EX stage in:**

Options: (a) cycle-4 (b) cycle-5 (c) cycle-6 (d) cycle-7

Same reasoning but for a branch at a later point in the sequence. If branch jj is at cycle 4, then j+1 enters EX at cycle 4+3=74+3=7.

**Answer: (d) cycle-7**

---

**Q8. If i4 were V6 = V5 * FV7, it would complete in:**

Options: (a) cycle-24 (b) cycle-26 (c) cycle-28 (d) none

This replaces a floating-point add (6 stages) with floating-point multiply (7 stages). The question is asking when V6 becomes available — i.e., the destination register is fully written.

Using n+VL+5n+VL+5 where n=7n=7, and assuming VL=10 (from setup earlier), and i4 issues at cycle 4:

4+(7+10+5)=4+22=26 cycles4+(7+10+5)=4+22=26 cycles

**Answer: (b) cycle-26**
