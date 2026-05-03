### Loop Unrolling

![[Pasted image 20260503122758.png]]

#### The Problem: Loop Overhead

Consider any loop. Every iteration, you execute the actual computation AND the loop control code — the counter update and the branch check. That branch + counter update is pure overhead; it contributes nothing to the computation itself.

For a loop running millions of iterations, this overhead adds up significantly.

#### The Idea

Instead of executing the loop body once per iteration and then jumping back, you **replicate the loop body** multiple times inside the loop, and adjust the loop termination condition accordingly. Now each "iteration" of the outer loop does the work of multiple original iterations, and you only pay the branch+counter cost once for all of them.

This is called **loop unrolling**.

---

#### Slide 9.1 — The Example

The original loop:

```
Loop:  L.D    F0, 0(R1)
       DADDUI R1, R1, #-8
       ADD.D  F4, F0, F2
       stall
       BNE    R1, R2, Loop
       S.D    F4, 8(R1)
```

This runs **6 cycles per iteration** (including 1 stall between L.D and ADD.D due to load-use RAW hazard).

**After unrolling 4 times**, the loop body is replicated 4 times. The DADDUI and BNE appear only once (at the end of all 4 copies). The four copies operate on different array elements using offsets: `0(R1)`, `-8(R1)`, `-16(R1)`, `-24(R1)`.

The result: **28 cycles per 4 elements = 7 cycles per element**.

That's worse than the original 6! Why? Because unrolling alone doesn't help — the stalls are still there. The unrolled loop has **14 issue cycles + 14 stalls = 28 cycles**.

The note says: "can be improved by scheduling." That's the next step.

---

#### Slide 9.2 — Scheduling the Unrolled Loop

![[Pasted image 20260503122738.png]]


After unrolling, you have 4 independent L.D instructions loading into F0, F6, F10, F14. These are independent of each other. So you can **reorder instructions** to fill the stall slots.

The scheduled version groups all 4 L.Ds together first, then all 4 ADD.Ds, then all 4 S.Ds. Since the L.Ds are all independent, and by the time the first ADD.D executes, the corresponding L.D result has already arrived — the stalls are eliminated.

Result: **14 cycles per 4 elements = 3.5 cycles per element**.

Compare:

- Original loop: 6 cycles/element
- Unscheduled unrolled: 7 cycles/element
- Scheduled unrolled: **3.5 cycles/element**

This is the key insight: **unrolling exposes independent instructions that scheduling can exploit to eliminate stalls.**

**PYQ connection:** CT-II 2021 Q14 asks which instructions are NOT replicated when unrolling — that's DADDUI and BNE (the loop control). Q15 asks how to improve after mechanical unrolling — the answer is scheduling.