## Chapter 1: Instruction Pipelining

### What is Pipelining?

The traditional execution model is: instruction i+1i+1 begins only after instruction ii has **completely finished**. This is sequential execution.

Pipelining breaks this. It allows **overlapped execution** — instruction i+1i+1 is allowed to begin before instruction ii has finished. This is possible because an instruction is broken into **independent stages**, and while instruction ii is in stage s2s2, instruction i+1i+1 can already be in stage s1s1.

Consider a 2-stage instruction (stages s1s1, s2s2):

||Cycle 1|Cycle 2|Cycle 3|
|---|---|---|---|
|i1|s1|s2||
|i2||s1|s2|

Two instructions complete in 3 cycles. Without pipelining: 2+2=42+2=4 cycles. Average execution time = 1.51.5 cycles per instruction.

With 3 instructions:

||C1|C2|C3|C4|
|---|---|---|---|---|
|i1|s1|s2|||
|i2||s1|s2||
|i3|||s1|s2|

3 instructions in 4 cycles → average = 1.331.33 cycles. Once the pipeline is **full** (every stage occupied every cycle), one instruction completes **per cycle**. Pipelining does not reduce the time to execute a single instruction — it increases **throughput**.

---

### The 5-Stage Pipeline (IF, ID, EX, MEM, WB)

For MIPS (a RISC architecture), every instruction executes in at most 5 clock cycles. The stages are:

**Stage 1 — IF (Instruction Fetch):**The PC is sent to instruction memory. The current instruction is fetched. The PC is updated to PC+4PC+4 (since each instruction is 4 bytes).

**Stage 2 — ID (Instruction Decode / Register Fetch):** The instruction is decoded. Source register values are read from the register file. In a RISC architecture, register specifiers are always at a **fixed location** in the instruction — so decoding and register reading happen simultaneously (called **fixed-field decoding**). The branch condition is also tested here, and the branch target address is computed.

**Stage 3 — EX (Execute / Effective Address):** The ALU operates on the values prepared by ID. Three cases:

- **Memory reference:** ALU computes base+offsetbase+offset (effective address).
- **Register-Register ALU instruction:** ALU performs the specified operation on the two register values.
- **Register-Immediate ALU instruction:** ALU performs the operation on one register value and the sign-extended immediate.

**Stage 4 — MEM (Memory Access):**

- For a **load**: the data memory is read at the effective address computed in EX.
- For a **store**: data is written to the effective address.
- ALU instructions do nothing in this stage.

**Stage 5 — WB (Write Back):** The result is written back into the register file — either the value from memory (load) or from the ALU (ALU instruction).

The resulting pipeline timing diagram looks like this:

|Instruction|C1|C2|C3|C4|C5|C6|C7|C8|C9|
|---|---|---|---|---|---|---|---|---|---|
|i|IF|ID|EX|MEM|WB|||||
|i+1||IF|ID|EX|MEM|WB||||
|i+2|||IF|ID|EX|MEM|WB|||
|i+3||||IF|ID|EX|MEM|WB||
|i+4|||||IF|ID|EX|MEM|WB|

From cycle 5 onwards the pipeline is full — one instruction completes every cycle.

---

### Two Important Implementation Details

**1. Separate Instruction and Data Memories:**

Look at the pipeline diagram at cycle CC4: instruction ii is in MEM (accessing **data memory**), while instruction i+3i+3 is in IF (accessing **instruction memory**). If there is only a single memory port, these two accesses **conflict**. The solution is to use **separate instruction memory (IM) and data memory (DM)** — in practice, separate instruction and data caches.

**2. Register Write-Before-Read:**

At cycle CC5: instruction ii is in WB (writing to register file) while instruction i+3i+3 is in ID (reading from register file). Both access the register file in the same cycle. The solution: **register writes are done in the FIRST HALF of the cycle**, and **register reads are done in the SECOND HALF**. So instruction ii's write completes before instruction i+3i+3 reads.

**Pipeline Registers:**

Between every two adjacent stages, there are **pipeline registers**. At the end of each cycle, a stage's results are stored in the pipeline register feeding the next stage. This is what keeps each instruction's data from interfering with another's. The pipeline registers also carry intermediate results across non-adjacent stages (e.g., the destination register number needs to travel from ID all the way to WB).

---

### Pipeline Hazards

A hazard is a situation where the pipeline **cannot proceed as normal** — it prevents achieving the ideal throughput of one instruction per cycle. There are three classes:

**1. Structural Hazards** — arise from **resource conflicts**: two instructions need the same hardware resource in the same cycle.

Example: a single memory port (no separate IM/DM). At CC4, a load instruction accesses data memory while a subsequent instruction tries to fetch from instruction memory. These collide on the same memory unit.

Solution: insert a **stall** (also called a **pipeline bubble**). A stall is a cycle where no new instruction is initiated — the bubble propagates through the pipeline stages doing no useful work, but it separates the conflicting accesses.

In the notes' Figure C.5, instruction i+3i+3 is held back and enters the pipeline in cycle 5 instead of cycle 4. This prevents the memory conflict.

**2. Data Hazards** — arise because one instruction depends on the result of a previous instruction that hasn't finished yet.

**3. Control Hazards** — arise because of branch instructions that change the PC.

Data and control hazards are covered in detail next.

---

### Data Hazards

Consider:

```
DADD  R1, R2, R3    ; writes R1
DSUB  R4, R1, R5    ; reads R1
AND   R6, R1, R7    ; reads R1
OR    R8, R1, R9    ; reads R1
XOR   R10, R1, R11  ; reads R1
```

DADD writes R1 in its WB stage — that's **CC5**. But DSUB reads R1 in its ID stage — that's **CC3**. DSUB reads R1 **before DADD has written it**. This is a **RAW (Read After Write) hazard** — the most common hazard in pipelined processors.

Similarly, AND reads R1 in CC4 — still before DADD writes in CC5. Hazard again.

OR reads R1 in CC5. DADD **writes** R1 in the **first half** of CC5, and OR **reads** in the **second half** — no hazard.

XOR reads R1 in CC6 — well after DADD wrote in CC5. No hazard.

---

### Resolving Data Hazards: Forwarding (Bypassing)

For the DADD → DSUB hazard: DADD's result is produced at the **end of CC3** (end of EX stage). DSUB needs it at the **beginning of CC4** (start of EX stage).

If you add a **direct hardware path** (a wire + multiplexer) from the ALU output of one instruction to the ALU input of the next, the result can be forwarded **without waiting** for it to be written to the register file. This is **forwarding** (also called **bypassing** or **short-circuiting**).

So: DADD → DSUB: forwarding resolves it (ALU output of CC3 → ALU input of CC4). DADD → AND: also resolved by forwarding (ALU output of CC3 → ALU input of CC5). DADD → OR: resolved naturally via the register file (first-half write / second-half read).

---

### Resolving Data Hazards: Stalling (When Forwarding Isn't Enough)

Consider:

```
LD    R1, 0(R2)    ; load from memory
DSUB  R4, R1, R5   ; needs R1
AND   R6, R1, R7
OR    R8, R1, R9
```

LD gets the memory data at the **end of CC4** (end of MEM stage). But DSUB needs R1 at the **beginning of CC4** (start of EX stage). Forwarding would require sending data **backward in time** — impossible.

The only solution is to **stall** DSUB for 1 cycle. DSUB's EX stage is pushed to CC5. By that time, the memory data from LD is available at the end of CC4 (in the MEM/WB pipeline register) and can be forwarded to DSUB's EX stage at CC5.

The stall pushes AND and OR back by 1 cycle as well — they all slide forward together. After the stall:

- AND reads R1 in CC5 (ID stage) — forwarded from LD's pipeline register.
- OR reads R1 in CC6 — LD has already written to the register file in CC5. No forwarding needed.

This specific type — a load followed immediately by an instruction that uses the loaded value — is called a **load-use hazard**, and it always requires exactly 1 stall even with forwarding hardware present.

---

### Control Hazards (Branch Hazards)

Consider:

```
i1:  BEQ R1, R2, label10   ; branch if R1 == R2
i2:  ...
i3:  ...
label10:
i10: ...
```

In a pipeline, when i1 is in the IF stage in cycle 1, i2 is fetched in cycle 2 — before the pipeline has determined whether the branch is taken. The branch condition is evaluated in the **ID stage** (cycle 2 for i1). If the branch is taken, i2 should not execute — but it has already entered the pipeline.

The simplest solution: when a branch is detected in ID, **stall the pipeline for 1 cycle** — re-fetch the correct instruction once the branch target is known.

From the notes' Figure C.11, the branch successor gets an extra IF — it is fetched but ignored, then re-fetched from the correct location. This wastes 1 cycle per branch. With a branch frequency of 10–30% of instructions, this is a significant loss.

This is introduced here; the next topic (Notes-04) covers **how to handle this efficiently**.

---

### Pipeline Scheduling and Stall Reduction (Notes-04)

#### Data Dependences — Formal Classification

A **dependence** is a property of the **program**, not of how it executes. When executed in a pipeline, a dependence can cause a **hazard** — a potential violation of sequential semantics. There are three types:

- **RAW (Read After Write):** instruction jj reads a register before instruction ii has written it. This is a **true dependence** — jj actually needs ii's result. This is the only one that causes real correctness problems in forward-only pipelines.
- **WAW (Write After Write):** instruction jj writes a register before instruction ii writes it. The writes happen in the wrong order. This is a **false dependence** (also called an **output dependence**) — jj doesn't actually need ii's value, but if both write the same register out of order, the wrong value is left behind.
- **WAR (Write After Read):** instruction jj writes a register before instruction ii reads it, so ii reads the wrong (new) value. Also a **false dependence** (also called an **anti-dependence**). Note: in a simple in-order pipeline, WAR rarely causes a hazard because reads happen before writes in pipeline order. WAR becomes important in out-of-order or dynamic scheduling.

False dependences (WAW, WAR) can be **eliminated by register renaming** — giving the destination a fresh register so the name conflict disappears.

---

#### Pipeline Scheduling — Filling Stalls with Useful Work

Instead of inserting a stall (wasted cycle), you can **reorder instructions** to place an independent instruction in the slot, as long as sequential semantics is preserved.

**Example 4.1** from the notes:

The latency table (Figure 3.2):

|Instruction producing result|Instruction using result|Latency (intervening cycles)|
|---|---|---|
|FP ALU op|Another FP ALU op|3|
|FP ALU op|Store double|2|
|Load double|FP ALU op|1|
|Load double|Store double|0|

**Latency** = number of **intervening** cycles required between the two instructions to avoid a stall. So if latency = 1, there must be 1 instruction (or stall) between them.

The unscheduled loop:

```
Loop: L.D    F0, 0(R1)      ; cycle 1
      stall                  ; cycle 2  (latency 1: Load→FP ALU)
      ADD.D  F4, F0, F2     ; cycle 3
      stall                  ; cycle 4  (latency 2: FP ALU→Store)
      stall                  ; cycle 5
      S.D    F4, 0(R1)      ; cycle 6
      DADDUI R1, R1, #-8    ; cycle 7
      stall                  ; cycle 8  (latency 1: DADDUI→BNE)
      BNE    R1, R2, Loop   ; cycle 9
```

Total: **9 cycles** per iteration.

After scheduling — move DADDUI up between L.D and ADD.D:

```
Loop: L.D    F0, 0(R1)      ; cycle 1
      DADDUI R1, R1, #-8    ; cycle 2  (fills the L.D → ADD.D stall slot)
      ADD.D  F4, F0, F2     ; cycle 3
      stall                  ; cycle 4
      stall                  ; cycle 5
      S.D    F4, 8(R1)      ; cycle 6  (address changed: see below)
      BNE    R1, R2, Loop   ; cycle 7
```

Total: **7 cycles** per iteration.

**Why is the address of S.D changed to 8(R1)?**

In the original code, at the start of each iteration, R1 holds value xx. S.D wrote to address 0+x=x0+x=x. But in the scheduled code, DADDUI runs before S.D and decrements R1 to x−8x−8. So to still store at address xx, S.D must use offset 88: 8+(x−8)=x8+(x−8)=x. Sequential semantics is preserved.

What we saved: the stall between L.D and ADD.D is filled by DADDUI (saves 1 cycle), and the latency between DADDUI and BNE is now 4 cycles (well above the required 1), so that stall also disappears (saves 1 more cycle). **2 stalls eliminated.**

---

### PYQ Questions for This Chapter

Now let's go through the relevant exam questions.

---

**From 2021 paper (CT-I context):**

> **Q: DADD R1,R2,R3 / DSUB R4,R1,R5 / AND R6,R1,R7 / OR R8,R1,R9 / XOR R10,R1,R11. DADD enters IF in cycle-1.**
> 
> - Does DSUB get the correct value of R1? → **No** (reads in CC3, DADD writes in CC5)
> - Does AND get the correct value of R1? → **No** (reads in CC4, DADD writes in CC5)
> - Does OR get the correct value of R1? → **Yes** (reads in second half of CC5, DADD writes in first half of CC5)

---

**From 2021 paper:**

> **Q: LD R1,0(R2) / DSUB R4,R1,R5 / AND R6,R1,R7 / OR R8,R1,R9. Can the LD→DSUB hazard be resolved by forwarding?** → **No.** LD gets its result at the end of CC4 (MEM stage). DSUB needs it at the start of CC4 (EX stage). That would require forwarding backward in time.

---

**From 2019 paper Q20:**

> **mul r1,r2,r3 / add r2,r4,r5. What is the dependency?** → **WAR.** The first instruction reads r2 (source), the second writes r2 (destination). If the second executes first, the first reads the wrong value.

---

**From CT-1 2024 / 2023 / 2025 — latency/stall counting with the standard FP latency table:**

The pattern is always: identify which pair of instructions has a dependency, look up the latency in the table, and count how many intervening instructions already exist between them. If fewer than the required latency, `stalls = required latency − existing intervening instructions`.

For example from the 2021 exam (Q8–Q9):

```
Loop: L.D F0, 0(R1)
      ADD.D F4, F0, F2
      S.D F4, 0(R1)
      DADDUI R1, R1, #-8
      BNE R1, R2, Loop
```

- L.D → ADD.D: latency = 1. No instruction between them. **Stalls = 1.**
- ADD.D → S.D: latency = 2. No instruction between them. **Stalls = 2.**
- DADDUI → BNE: latency = 1 (integer). No instruction between them. **Stalls = 1.**

**The scheduled version** (from 2024 paper Q1–Q3): DADDUI moves up between L.D and ADD.D, so DADDUI is issued in cycle 2, ADD.D in cycle 3, S.D in cycle 6. You should be able to trace each issue cycle using the latency rules — we'll do this as practice when we hit the loop unrolling topic.

---

**From 2023 paper Q19–Q20 (structural hazard, single memory port):**

> i1..i5 enter in consecutive cycles CC1..CC5. If there's only one memory port, **two simultaneous memory accesses** occur in CC5 (i1 in MEM, i5 in IF). Delaying i4 to enter in CC5 instead of CC4 avoids the i1 MEM vs i4 IF conflict.

---

**From 2021 Q2 (structural hazard with no separate IM/DM):**

> i1 is a load entering IF in cycle-1. i4 enters IF in cycle-4 (MEM of i1 is cycle-4). **Both access memory in cycle 4** — structural hazard.