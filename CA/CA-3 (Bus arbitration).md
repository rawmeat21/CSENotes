## Bus Arbitration

### The Single Shared Bus

![[Pasted image 20260503115913.png]]


In a single-bus multiprocessor, all processors P1,P2,…,PkP1​,P2​,…,Pk​ and all memory modules M1,M2,…,MnM1​,M2​,…,Mn​ are connected to a single shared bus. The bus carries five categories of lines: **address lines** (carry the memory address), **data lines** (carry the actual data), **control lines** (carry read/write and other control signals), **interrupt lines** (carry interrupt signals from devices to processors), and **bus exchange lines** (carry the arbitration signals — bus request and bus grant — by means of which processors or other temporary bus masters can request bus allocation).

Since the bus is a shared resource, only one master may use it at any instant. If multiple masters attempt simultaneous access, data is corrupted. The **bus arbiter logic** solves this — it allocates the bus in the case of several simultaneous bus requests, and applies a **bus allocation policy** to decide which requester wins. The arbiter communicates its decision to the winner via **grant lines**. Available allocation policies include: fixed priority, rotating priority, round robin, least recently used, and first come first served.

---

### Centralized Arbitration

![[Pasted image 20260503115933.png]]

In centralized arbitration there is a **single central bus arbiter**. Each master ii has its own **dedicated request line** RiRi​ going to the arbiter, and its own **dedicated grant line** GiGi​ coming back from the arbiter. There is one **single shared bus busy line** that all masters and the arbiter observe.

The protocol works as follows:

**(i)** Master ii requests the bus by **activating its dedicated request line** RiRi​.

**(ii)** If the **bus busy line is passive** (no other master is currently using the bus), the arbiter immediately allocates the bus to the requestor by **activating grant line** GiGi​. The requestor then: deactivates its request line RiRi​, activates the bus busy line (blocking all other requests), performs its bus transaction, and finally deactivates the bus busy line when done.

**(iii)** If the **bus busy line is active**, the arbiter does not accept any bus requests.

**(iv)** When several request lines are simultaneously active at the moment the bus busy line becomes passive, the arbiter selects one winner using its allocation policy.

Key structural fact: in centralized arbitration, the bus request lines are **individual** (one per master), the bus grant lines are **individual** (one per master), and the **bus busy line is the single shared line** common to all.

> **PYQ:** In centralized arbitration, a requesting master that receives a grant **(d) both deactivates its request line AND activates the bus busy line** — both steps happen in sequence.

> **PYQ:** In the centralized scheme, **(c) the bus busy line is common** (shared). The request and grant lines are NOT common — each master has its own.

> **PYQ:** A requesting master gains control of the bus **(c) if the grant line is active** (and the bus busy line is passive — the arbiter will not issue a grant until the bus is free, so these two conditions go together).

---

### Daisy-Chained Bus Arbitration

![[Pasted image 20260503115950.png]]


Daisy-chaining is one of the most popular arbiter logic organizations. Its defining structural feature is that there is only **one single shared bus request line** — all masters use this one line to signal their need for the bus. The **bus grant line**, however, is not shared — it is passed serially from master to master in a chain.

The arbiter passes the grant to Master 1 first, and then it propagates from Master 1 to Master 2 to Master 3, all the way to Master N, creating a chain. Each master's output grant line is the next master's input grant line.

The protocol is:

1. Any master needing the bus activates the **single shared bus request line**.
2. When the arbiter sees the request line active and the bus busy line passive, it activates the grant and passes it to Master 1.
3. Master 1 checks if it needs the bus. If **yes**, and its input grant is active, and the bus busy line is passive — it claims the bus, activates the bus busy line, and does NOT pass the grant further. If **no** (it does not need the bus), it **activates its output grant line**, propagating the grant to Master 2. Master 2 applies the same logic, and so on.
4. The winning master deactivates the bus busy line after its transaction.

**A master can access the bus if and only if: the bus busy line is passive AND its input grant line is active.**

The **priority** of a master is determined entirely by its **position in the grant chain** — the closer it is to the arbiter, the higher its priority. Master 1 always has the highest priority; Master N always has the lowest.

#### Weakness

The daisy-chained scheme **lacks fairness**. Master 1 can continuously re-request the bus and always win before the grant ever propagates to Master N. Masters far from the arbiter can starve indefinitely under heavy load. This motivates the rotating arbiter.

> **PYQ:** In a daisy-chained scheme, the priority of a master **(a) is determined by its position in the grant chain**.

> **PYQ:** When a master does not require the bus and receives an active grant line, it **(c) activates its output grant line, enabling the next master to use the bus**.

> **PYQ:** In a daisy-chained scheme, **(c) there is only one shared bus request line**. The grant is chained, not the request.

> **PYQ:** The number of bus request lines in daisy-chaining is **(a) 1**.

---

### Decentralized Rotating Arbiter

![[Pasted image 20260503120008.png]]


The rotating arbiter is a decentralized design that eliminates the unfairness of daisy-chaining. There is **no single central arbiter** — instead, each master ii has its own **local arbiter** (Arbiter ii). Each arbiter-master pair has its own request line RiRi​ and grant line GiGi​. In addition, arbiters are connected by a chain of **priority lines**: the output priority line PiPi​ of Arbiter ii feeds into Arbiter i+1i+1 as its priority input. All arbiters observe the shared **bus busy line**.

#### Grant Condition

Arbiter ii is allowed to grant the bus to its coupled Master ii **if and only if all three of the following conditions hold simultaneously:**

1. Master ii has **activated its bus request line** RiRi​ (it actually wants the bus).
2. The **bus busy line is passive** (the bus is free).
3. The **priority (i−1)(i−1) input line is active** (the priority token has reached Arbiter ii).

If Master ii has not activated its request line, Arbiter ii simply **activates its output priority line** PiPi​, forwarding the token to Arbiter i+1i+1.

#### The Rotation Mechanism

This is the critical distinction from daisy-chaining. In the two schemes, the lowest-priority unit is selected differently:

- **Daisy-chained arbiter** → lowest priority always goes to the master **farthest from the arbiter** (Master N), regardless of history.
- **Rotating arbiter** → lowest priority always goes to the master that **most recently released the bus**.

After Master ii finishes using the bus, it becomes the lowest priority — the priority token rotates so that the just-served master must wait the longest before being served again. This guarantees no starvation.

> **PYQ:** In the decentralized rotating arbiter, Arbiter ii grants its master when the master has activated its request, the bus busy line is passive, and the priority (i−1)(i−1) input line satisfies the grant condition.

> **PYQ:** In a decentralized rotating arbitration method, an arbiter grants its coupled master if **(d) all of the above** — all three conditions (request active, bus busy passive, priority condition met) must hold.

---

### Summary

| Feature            | Centralized       | Daisy-Chained               | Rotating                     |
| ------------------ | ----------------- | --------------------------- | ---------------------------- |
| Request lines      | One per master    | **One shared**              | One per master               |
| Grant lines        | One per master    | **Chained** through masters | One per master               |
| Number of arbiters | 1 (central)       | 1 (central)                 | N (one per master)           |
| Priority           | Policy-based      | Fixed by chain position     | Rotates — last user = lowest |
| Fairness           | Depends on policy | Poor — can starve           | Good — no starvation         |
| Bus busy line      | Single shared     | Single shared               | Single shared                |