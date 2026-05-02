### Dynamic Scheduling — Tomasulo's Algorithm

#### The Core Problem with Static (In-Order) Execution

In a standard pipeline, instructions execute in program order. If instruction ii is stalled (waiting for a result), every instruction after it also stalls — even if those later instructions have no dependency on ii at all. You're leaving functional units idle unnecessarily.

**Dynamic scheduling** breaks this constraint. The hardware itself decides, at runtime, which instruction to execute next based on operand availability — not program order.

---

#### The Mental Model

Stop thinking of a program as a sequence of steps. Think of it as a **set of instructions, each waiting for its inputs**. The moment an instruction's inputs are ready AND a functional unit is free, it executes — regardless of where it sits in program order.

This is **out-of-order execution**.

---

#### Slide 9.6 — Tomasulo's Algorithm: The Big Picture

Invented by Robert Tomasulo for the IBM 360/91 floating-point unit (1967). It solves three problems simultaneously:

**1. RAW hazards** — handled by tracking when operands become available and only executing when they're ready.

**2. WAR hazards** — eliminated by register renaming. WAR is a _false dependency_. It only exists because two instructions share a register name, not because one truly needs the other's value.

**3. WAW hazards** — also eliminated by register renaming. Same reason.

The mechanism that provides renaming is the **Reservation Station**.

---

#### Slide 9.7 — WAR and WAW: Why They're False

**WAR (Write After Read):**

```
i1: mul r1, r2, r3
i2: add r2, r4, r5   ← writes r2, which i1 reads
```

i2 must not write r2 before i1 reads it. But this is only a problem because both instructions use the name `r2`. If you rename r2 in i2 to a fresh register r6, the dependency vanishes entirely. No actual data flows from i2 to i1.

**WAW (Write After Write):**

```
i1: mul r1, r2, r3
i2: add r1, r4, r5   ← both write r1
```

i2 must not write r1 before i1 writes it. Again, a naming conflict, not a true data dependency. Renaming the destination of one of them removes it.

**Only RAW is a true dependency** — it represents actual data flowing from one instruction to another.

---

#### Slide 9.3 — The Three Steps of Execution

Every instruction in Tomasulo goes through exactly three steps:

**Step 1: Issue**

Take the next instruction from the instruction queue (maintained in FIFO order). If a reservation station for that functional unit is free, issue the instruction to it.

- If both operands are already in registers → copy the values directly into the reservation station fields VjVj​, VkVk​.
- If an operand is not yet available → store in QjQj​ or QkQk​ the name of the reservation station that will produce it.
- If no reservation station is free → stall (structural hazard).

This step also performs **register renaming**: the register file's `Qi` field is updated to point to this reservation station as the producer of the destination register.

**Step 2: Execute**

The reservation station watches the Common Data Bus (CDB). When it sees a result broadcast by the station named in its QjQj​ or QkQk​ field, it captures the value and clears the Q field.

When **both** VjVj​ and VkVk​ are filled (both Q fields are empty), the instruction is ready. It then executes on the functional unit. This avoids RAW hazards — an instruction never executes before its inputs are ready.

For **LOADs and STOREs**, execution is two steps:

1. Compute effective address (when base register is available)
2. Access memory (LOADs execute as soon as memory unit is free; STOREs wait until both address and value are available)

**Step 3: Write Result**

When execution completes, broadcast the result on the **CDB**. Every reservation station and every register that is waiting for this result captures it simultaneously. The reservation station is freed.

STOREs also complete here — when both address and data are available, they're sent to the memory unit.

---

#### Slide 9.4 — Hardware Structure

The hardware has:

- **Instruction Queue** — feeds instructions in program order
- **Reservation Stations** — one set per functional unit (FP adders, FP multipliers)
- **Load Buffers** — track outstanding loads in three states: (i) holding effective address components, (ii) tracking loads waiting on memory, (iii) holding completed load results waiting for CDB
- **Store Buffers** — (i) hold effective address components, (ii) hold address + value waiting to be sent to memory
- **FP Register File** — holds values + a `Qi` field per register
- **Common Data Bus (CDB)** — broadcasts results to all reservation stations and the register file simultaneously. Everything goes here _except_ to the load buffer (load buffer gets data from memory directly)

---

#### Slide 9.5 — The Reservation Station Fields

Each reservation station has **7 fields**:

|Field|Meaning|
|---|---|
|**Name**|Identifier of this station (e.g., Add1, Mult2)|
|**Busy**|Is this station occupied?|
|**Op**|Operation to perform (ADD.D, MUL.D, etc.)|
|**Vj**|Value of first source operand (if available)|
|**Vk**|Value of second source operand (if available)|
|**Qj**|Name of station that will produce first operand (if not yet available)|
|**Qk**|Name of station that will produce second operand (if not yet available)|
|**A**|For loads/stores: initially holds immediate offset, then holds computed effective address|

**Key rule:** Either VjVj​ is valid OR QjQj​ is set — never both simultaneously. If QjQj​ is empty, VjVj​ holds the actual value. If QjQj​ is set, VjVj​ is meaningless.

The **Register File** has an extra field:

**Qi** — the name of the reservation station whose result should be stored into this register. If Qi is empty, the register holds a valid up-to-date value.

---

#### Slide 9.5 — Worked Example

Instructions:

```
1. L.D   F6, 34(R2)    ← already completed; result on CDB
2. L.D   F2, 45(R3)    ← issued (in Load2)
3. MUL.D F0, F2, F4    ← issued (in Mult1)
4. SUB.D F8, F2, F6    ← issued (in Add1)
5. DIV.D F10, F0, F6   ← issued (in Mult2)
6. ADD.D F6, F8, F2    ← issued (in Add2)
```

**Reservation station state (snapshot after all 6 issued):**

- **Load2**: Busy, Op=Load, A = 45+Regs[R3]
- **Add1** (SUB.D F8, F2, F6): Busy, Op=SUB, VkVk​ = Mem[34+Regs[R2]] (F6 value, already arrived), QjQj​ = Load2 (waiting for F2)
- **Add2** (ADD.D F6, F8, F2): Busy, Op=ADD, QjQj​ = Add1, QkQk​ = Load2
- **Mult1** (MUL.D F0, F2, F4): Busy, Op=MUL, VkVk​ = Regs[F4], QjQj​ = Load2
- **Mult2** (DIV.D F10, F0, F6): Busy, Op=DIV, VkVk​ = Mem[34+Regs[R2]], QjQj​ = Mult1

**Register file Qi fields:**

- F0 → Mult1, F2 → Load2, F6 → Add2, F8 → Add1, F10 → Mult2

This shows the renaming: F6 appears as both a source (its old value already captured in VkVk​ of Add1 and Mult2) and a destination (Add2 will write to it). No WAW or WAR conflict — the hardware has already separated the old F6 value from the future one.

---

#### Why WAR/WAW Are Eliminated — The Precise Mechanism

When instruction 6 (ADD.D F6, ...) is issued, the register file's Qi for F6 is updated to Add2. Any later instruction that reads F6 will see Qi=Add2 and know to wait for Add2's result. Meanwhile, instruction 4 (SUB.D) already has the *old* F6 value captured in its VkVk​ at issue time — it doesn't care what happens to F6 afterward. This is register renaming in action.

