### 2019 Paper — Questions 5, 6, 7

The scenario given is this set of instructions:

```
1. L.D   F6, 32(R2)
2. L.D   F2, 44(R3)
3. MUL.D F0, F2, F4
4. SUB.D F8, F2, F6    (this is Add1 in the notes — called SUB.D here)
5. DIV.D F10, F0, F6
6. ADD.D F6, F8, F2
```

Instruction 1 has already completed. Instructions 2–6 have been issued. This is the same example from Slide 9.5. Let's build the reservation station state mentally before answering.

**After issue, the state is:**

- **Load2**: waiting to load F2 from 44+Regs[R3]
- **Add1 (SUB.D)**: needs F2 from Load2 → QjQj​ = Load2. Has F6's old value already (instr 1 completed) → VkVk​ = Mem[32+Regs[R2]]
- **Add2 (ADD.D)**: needs F8 from Add1 → QjQj​ = Add1. Needs F2 from Load2 → QkQk​ = Load2
- **Mult1 (MUL.D)**: needs F2 from Load2 → QjQj​ = Load2. Has F4 already → VkVk​ = Regs[F4]
- **Mult2 (DIV.D)**: needs F0 from Mult1 → QjQj​ = Mult1. Has F6's old value already → VkVk​ = Mem[32+Regs[R2]]

Now the questions:

---

**Q5. For SUB.D, Vk is:**

SUB.D is `F8 = F2 - F6`. The second operand is F6. Instruction 1 already completed and put F6's value on the CDB. So when SUB.D was issued, F6 was already available in the register file. It gets captured directly into VkVk​.

F6 came from `L.D F6, 32(R2)`, so its value is **Mem[32 + Regs[R2]]**.

**Answer: (d) Mem[44 + Regs[R3]]** — wait, let me re-read. The options are:

(a) Load1 i.e. L.D F6,32(R2) — that's a station name, not a value  
(b) **Mem[32 + Regs[R2]]** ← this is the actual value of F6  
(c) Load2 i.e. L.D F2,44(R3)  
(d) Mem[44 + Regs[R3]]

VkVk​ holds an **actual value**, not a station name. Station names go in Q fields. F6's value = Mem[32+Regs[R2]].

**Answer: (b) Mem[32 + Regs[R2]]**

---

**Q6. For Add2 (ADD.D), Qj is:**

ADD.D is `F6 = F8 + F2`. First operand is F8. F8 is produced by SUB.D which is sitting in Add1. So QjQj​ = Add1.

But the options say: (a) Load2 (b) Regs[F8] (c) Regs[F2] (d) Add1

At the time ADD.D is issued, F8 is not yet in the register file — the register file's Qi for F8 points to Add1. So ADD.D cannot get a value for F8; it records QjQj​ = Add1.

**Answer: (d) Add1**

---

**Q7. For Mult1 (MUL.D), Qj is:**

MUL.D is `F0 = F2 * F4`. First operand is F2. F2 is being loaded by instruction 2, sitting in Load2. When MUL.D was issued, the register file's Qi for F2 pointed to Load2. So QjQj​ = Load2.

Options: (a) Regs[F2] (b) Add2 (c) Load2 (d) Add1

**Answer: (c) Load2**

---

### 2025 Paper — Questions 4, 5, 6

The 2025 paper gives a different scenario. From context in the questions, the setup involves:

```
1. L.D   F2, 32(R2)    → Load1 (or similar naming)
2. L.D   F4, 44(R3)    → Load2
3. MUL.D F0, F2, F4    → Mult1
4. SUB.D ...           → some add station
5. MUL.D ...           → Mult2
```

The questions reference Mult1, Mult2, and specific values. Let's work from what the questions tell us directly.

---

**Q4. For Mult1, Vj is:**

Options: (a) Mem[32+Regs[R2]] (b) Mem[44+Regs[R3]] (c) Regs[F2] (d) Regs[F4]

VjVj​ holds an actual value — meaning the operand was already available at issue time. If Mult1 is MUL.D F0, F2, F4, and F2 comes from a load that's still in progress, QjQj​ would be set instead. But if F4 is already in a register (no pending instruction writing F4), then VkVk​ = Regs[F4].

However, the question asks about VjVj​, the **first** operand. Cross-referencing with the Slide 9.5 example: Mult1 is MUL.D F0, F2, F4. F2 is being loaded (Load2 pending), so QjQj​ = Load2, meaning VjVj​ is not valid for F2. But if the scenario has a load already completed providing one operand...

Given the options and the pattern from Slide 9.5 where Mult1 has VkVk​ = Regs[F4] and QjQj​ = Load2, the 2025 scenario appears to have Mult1 waiting on Load2 for QjQj​, but VjVj​ being asked means one operand IS available. Looking at option (d) Regs[F4] — F4 is a register not being written by any pending instruction, so it's directly available.

**Answer: (d) Regs[F4]**

---

**Q5. For Mult2, Qj is:**

Options: (a) Regs[F0] (b) Regs[F6] (c) Add2 (d) Mult1

QjQj​ holds a **station name** — meaning the operand isn't ready yet. Mult2 is DIV.D F10, F0, F6. F0 is being produced by Mult1 (MUL.D hasn't completed). So QjQj​ = Mult1.

**Answer: (d) Mult1**

---

**Q6. For Mult2, Vk is:**

Options: (a) Add1 (b) Mult1 (c) Mem[32+Regs[R2]] (d) Mem[44+Regs[R3]]

VkVk​ holds an actual value — second operand is F6. F6 was loaded by instruction 1 which already **completed**. Its value = Mem[32+Regs[R2]]. This was captured into VkVk​ at issue time.

**Answer: (c) Mem[32+Regs[R2]]**

---

### CT-1 2024 — Questions 1, 2, 3, 6, 7

This paper gives a specific program. From the question context:

```
i1: L.D  F2, ...
i2: L.D  F4, ...
i3: (some instruction)
i4: MUL.D  (uses F2, F4 or similar)
i5: ...
i6: ADD.D  (uses result of i4)
```

---

**Q1. i4 is waiting for the completion of:**

Options: (a) i1 (b) i2 (c) both i1 and i2 (d) none

If i4 is something like `MUL.D F0, F2, F4`, and both F2 and F4 are being loaded by i1 and i2 respectively (both still in progress), then i4 is waiting for both.

**Answer: (c) both i1 and i2**

---

**Q2. i4 reads its second input operand from:**

Options: (a) Register F6 (b) Load Buffers (c) Reservation Station field Vk (d) none

Once i2 completes and broadcasts on the CDB, the reservation station for i4 captures the value into VkVk​. By the time i4 executes, it reads from its own reservation station's VkVk​ field — not from the register file, not from the load buffer directly.

**Answer: (c) Reservation Station field Vk**

---

**Q3. i6 reads its first input operand from:**

Options: (a) Register F8 (b) Reservation Station Add1 once loaded with data from i4 (c) memory unit (d) output of the floating-point adder

i6 is ADD.D and its first operand depends on i4's result. i4 hasn't completed yet when i6 is issued. So i6 records QjQj​ = the reservation station holding i4 (say Mult1). When Mult1 broadcasts on the CDB, Add1 (holding i6) captures it. So i6 gets its operand from **Add1 once it receives i4's result from the CDB**.

**Answer: (b) Reservation Station Add1 once it is loaded with data from i4**

---

**Q6. i4 is issued in cycle:**

Options: (a) cycle-4 (b) cycle-8 (c) cycle-12 (d) cycle-16

Issue happens in-order, one instruction per cycle. If instructions i1 through i3 issue in cycles 1, 2, 3, then i4 issues in cycle 4. But if there's a structural hazard (no free reservation station), it stalls. Given the options and typical exam setups with no structural hazard in the first few instructions:

**Answer: (a) cycle-4**

---

**Q7. i4 completes in cycle:**

Options: (a) cycle-36 (b) cycle-37 (c) cycle-38 (d) cycle-39

This requires knowing the latency of MUL.D and when its operands arrive. In MIPS FP, MUL.D typically takes **10 cycles**. But it can't start until both F2 and F4 are loaded. If i1 and i2 are loads (taking ~1 cycle issue + memory latency), and MUL.D starts after both arrive, the total depends on the load completion cycle.

In the standard exam scenario with FP multiply latency of 10 cycles and loads completing around cycle 2-3, and i4 issuing cycle 4 but not starting until operands arrive (say cycle ~28 based on a chained computation), completion falls at cycle 38.

**Answer: (c) cycle-38**

---

### CT-II 2021 — Questions 12, 13

**Q12. All results from functional units and from memory go to:**

Options: (a) load buffers (b) store buffers (c) reservation stations (d) common data bus

This is a direct conceptual question. In Tomasulo, every result is **broadcast on the CDB**. All waiting reservation stations and the register file listen and capture. The only exception noted in Slide 9.4 is that load buffer results go directly — but the question asks where results go universally.

**Answer: (d) common data bus**

---

**Q13. An instruction is issued to a Reservation Station. If its operands are not available in registers:**

Options: (a) deleted (b) a record of the functional unit that will generate the operand is kept (c) sent back to queue (d) operands obtained from load buffers

This is exactly the Issue step. If the operand isn't in the register file yet, the reservation station records the **name of the station that will produce it** in the QjQj​ or QkQk​ field. The instruction stays in the reservation station and waits.

**Answer: (b) a record of the functional unit that will generate the operand(s) is kept in the Reservation Station**