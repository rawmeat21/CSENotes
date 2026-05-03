## Interconnection Networks

![[Pasted image 20260503120456.png]]


### What is an Interconnection Network?

An interconnection network is a **programmable system** that transports data between terminals. Terminals T1,T2,…,T6T1​,T2​,…,T6​ are connected to the network using channels. When terminal T3T3​ wishes to communicate some data with terminal T5T5​, it sends a message containing the data into the network and the network delivers the message to T5T5​. The network is programmable in the sense that it makes different connections at different points in time — it may deliver a message from T3T3​ to T5T5​ in one cycle and then use the same resources to deliver a message from T3T3​ to T1T1​ in the next cycle.

The network is a **system** because it is composed of many components — buffers, channels, switches, and controls — that work together to deliver data.

> **PYQ (2022 Q27, 2019 Q15):** An interconnection network is a system because **(c) it is composed of many components: buffers, channels, switches, and controls that work together to deliver data.**

---

### The Three Main Issues: Topology, Routing, and Flow Control

**Topology** — the interconnection network is implemented with a collection of shared router nodes connected by shared channels. The connection pattern of these nodes defines the network's topology. A message is then delivered between terminals by making several hops across the shared channels and nodes from its source terminal to its destination terminal.

**Routing** — once a topology has been chosen, there can be many possible paths (sequences of nodes and channels) that a message could take through the network to reach its destination. Routing determines which of these possible paths a message actually takes. The path actually taken by a message to reach its destination is determined by routing.

**Flow control** — flow control dictates which messages get access to particular network resources over time. More precisely, flow control determines how a network's resources — channel bandwidth, buffer capacity, and control state — are allocated to packets traversing the network. A good flow control method allocates these resources in an efficient manner so the network achieves a high fraction of its ideal bandwidth and delivers packets with low, predictable latency.

Flow control can be viewed as either a problem of **resource allocation** or one of **contention resolution**. From the resource allocation perspective, resources in the form of channels, buffers, and state must be allocated to each packet as it advances from source to destination. From the contention resolution perspective, if two packets arriving on different inputs of a router at the same time both desire the same output channel, the flow control mechanism resolves this contention by allocating the channel to one packet and somehow dealing with the other blocked packet.

> **PYQ (2018 Q12):** The path actually taken by a message to reach its destination is determined by **(b) routing.**

---

### Circuit Switching

Circuit switching is a form of **bufferless flow control**. It operates by first allocating channels to form a circuit from source to destination, and then sending one or more packets along this circuit. When no further packets need to be sent, the circuit is deallocated.

#### Non-Blocking Networks

A network is said to be **non-blocking** if it can handle all circuit requests that are a permutation of the inputs and outputs — that is, a dedicated path can be formed from each input to its selected output without any conflicts (shared channels). A network is **blocking** if it cannot handle all such circuit requests without conflicts.

A **strictly non-blocking** network is one where any permutation can be set up incrementally, one circuit at a time, without the need to reroute (rearrange) any of the circuits that are already set up.

A **rearrangeably non-blocking** (or rearrangeable) network is one where the network can route circuits for arbitrary permutations, but incremental construction of a permutation may require rearranging some of the early circuits to permit later circuits to be set up.

> **PYQ (2018 Q13):** A network is said to be non-blocking if **(c) it can handle all circuit requests without any conflicts (shared channels).**

> **PYQ (2019 Q16, 2023 Q7):** Circuit-switching is a form of **(a) bufferless flow control.**

---

### Crossbar Networks

![[Pasted image 20260503120518.png]]


An n×mn×m crossbar (or crosspoint switch) directly connects nn inputs to mm outputs with no intermediate stages. Each output may be connected to at most one input at a time, while each input may be connected to any number of outputs. Such a switch consists of mm nn-to-1 multiplexers, one for each output.

For example, a 4×54×5 crossbar has 4 input lines, 5 output lines, and 20 crosspoints. It consists of **5 four-to-one multiplexers**, one per output. Each multiplexer selects which of the 4 inputs to connect to its corresponding output. If the connection is {1,0,3,1,2}→{0,1,2,3,4}{1,0,3,1,2}→{0,1,2,3,4}, then output 0 is connected to input 1, output 1 to input 0, output 2 to input 3, output 3 to input 1, and output 4 to input 2.

Each of the nn input lines connects to one input of mm nn-to-1 multiplexers. The outputs of the multiplexers drive the mm output ports. The multiplexers may be implemented with tri-state gates driving an output line, with wired-OR gates, or with a tree of logic gates to realize a more conventional multiplexer.

In an n×mn×m crossbar:

- There are mm multiplexers, each of size nn-to-1.
- Each input connects to one input pin on each of the mm multiplexers.
- Each output is driven by exactly one multiplexer.

![[Pasted image 20260503120555.png]]


> **PYQ (2018 Q14):** A 4×54×5 crossbar switch can be implemented with **(a) 5 four-to-one multiplexers** — one per output, each selecting among the 4 inputs.

> **PYQ (2022 Q28):** In an n×mn×m crossbar, **(b) there are mm number of nn-to-1 multiplexers.**

> **PYQ (2023 Q8):** A 4×54×5 crossbar is implemented with 5 four-to-one multiplexers. Each input is connected to **(a) 5 multiplexers** — each input connects to one pin of each of the 5 multiplexers.

---

### Clos Networks

![[Pasted image 20260503120610.png]]


A Clos network is a **three-stage network** in which each stage is composed of a number of crossbar switches. A symmetric Clos network is characterized by a triple (m,n,r)(m,n,r) where:

- rr = number of input switches (= number of output switches)
- nn = number of input (output) ports on each input (output) switch
- mm = number of middle-stage switches

Each input switch is an n×mn×m crossbar, each middle-stage switch is an r×rr×r crossbar, and each output switch is an m×nm×n crossbar. In a Clos network, each middle-stage switch has **one input link from every input switch and one output link to every output switch**.

**Example:** An (m=3,n=3,r=4)(m=3,n=3,r=4) symmetric Clos network has 4 input switches (each 3×33×3), 3 middle-stage switches (each 4×44×4), and 4 output switches (each 3×33×3). Each middle-stage switch has r=4r=4 inputs and r=4r=4 outputs.

> **PYQ (2023 Q9):** Consider a symmetric Clos network with m=3,n=3,r=4m=3,n=3,r=4. Each middle-stage switch has **(d) 4 inputs** — the middle-stage switches are r×r=4×4r×r=4×4 crossbars.

---

### Butterfly Networks

![[Pasted image 20260503120626.png]]


A **k-ary n-fly** butterfly network consists of:

- k^n source terminal nodes
- n stages of switch nodes
- k^(n−1) switch nodes per stage, each of which is a k×kk×k crossbar
- k^n destination terminal nodes

So the total number of k×kk×k switch nodes in the entire network is n⋅kn−1n⋅kn−1.

The number of crossbar switch nodes in **each stage** of a k-ary n-fly butterfly network is k^(n−1)

Switch nodes are labeled s.ps.p where ss is the stage number (00 to n−1n−1) and pp is the position within that stage (00 to kn−1−1kn−1−1).

**Example — 2-ary 3-fly network (k=2,n=3k=2,n=3):**

- kn=23=8kn=23=8 source terminals (labeled 0 through 7)
- n=3n=3 stages of switch nodes
- kn−1=22=4kn−1=22=4 switch nodes per stage (labeled s.0,s.1,s.2,s.3s.0,s.1,s.2,s.3 for each stage ss)
- Each switch node is a 2×22×2 crossbar
- kn=8kn=8 destination terminals

The structure is:

```
Terminals    Stage 0    Stage 1    Stage 2    Terminals
    0  -----> 0.0 -----> 1.0 -----> 2.0 -----> 0
    1  --/                                      1
    2  -----> 0.1 -----> 1.1 -----> 2.1 -----> 2
    3  --/                                      3
    4  -----> 0.2 -----> 1.2 -----> 2.2 -----> 4
    5  --/                                      5
    6  -----> 0.3 -----> 1.3 -----> 2.3 -----> 6
    7  --/                                      7
```

#### Routing in a Butterfly Network

To route from source terminal ss to destination terminal dd in a kk-ary nn-fly network, at each stage the routing decision is based on comparing the digits of ss and dd in base kk. The position of the switch node at stage ii is determined by the leading digits of the source address processed so far, and the output port is selected based on the ii-th digit of the destination address.

**Example — Routing from terminal 6 to terminal 1 in the 2-ary 3-fly:**

Express source 6=(1,1,0)26=(1,1,0)2​ and destination 1=(0,0,1)21=(0,0,1)2​ in binary.

- Source terminal 6 enters switch node 0.30.3 (since 6/2=36/2=3, position 3 in stage 0).
- At stage 0, the destination digit (MSB of destination) is 0 → take lower output → proceed to node 1.11.1.
- At stage 1, the next destination digit is 0 → take lower output → proceed to node 2.02.0.
- At stage 2, the last destination digit is 1 → take upper output → arrive at destination terminal 1.

Path: terminal 6→0.3→1.1→2.0→6→0.3→1.1→2.0→ terminal 11.

> **PYQ (2025 Q13):** For a 2-ary 4-fly network, routing from terminal 12 to terminal 3 — apply the same digit-by-digit method using 2-bit groups. The answer given in the exam options requires tracing through 4 stages using the binary representations.

> **PYQ (2022 Q29):** In a 2-ary 3-fly butterfly network, **(c) each switch node is a 2×22×2 crossbar.** There are 32=932=9... wait — for k=2,n=3k=2,n=3: kn−1=4kn−1=4 nodes per stage, and each is a 2×22×2 crossbar.

> **PYQ (2023 Q10):** The number of crossbar switch nodes in each stage of a kk-ary nn-fly butterfly network is **(b) kn−1kn−1.**

---

### Allocation Units: Messages, Packets, Flits, and Phits

There is a hierarchy of allocation units in an interconnection network. Understanding what each unit is, what it is used for, and its typical size is essential.

#### Message

At the top level, a **message** is a logically contiguous group of bits that are delivered from a source terminal to a destination terminal. Messages may be arbitrarily long. Because messages may be arbitrarily long, resources are not directly allocated to messages. Instead, messages are divided into one or more **packets**.

#### Packet

A **packet** is a segment of a message to which a packet header is prepended. The packet header includes **routing information (RI)** and, if needed, a **sequence number (SN)**. The packet is the **basic unit of routing and sequencing**. Resources at a network node are allocated to packets rather than messages because messages may be arbitrarily long — restricting the length of a packet restricts the size and time duration of a resource allocation, which is often important for the performance and functionality of a flow control mechanism.

A packet consists of a **head flit**, zero or more **body flits**, and a **tail flit**.

#### Flit

A packet may be further divided into **flow control digits or flits**. A flit is the **basic unit of bandwidth and storage allocation** used by most flow control methods. Flits carry no routing and sequence information and thus must follow the same path and remain in order. However, flits may contain a **virtual-channel identifier (VCID)** to identify which packet the flit belongs to in systems where multiple packets may be in transit over a single physical channel at the same time.

#### Phit

A flit is itself subdivided into one or more **physical transfer digits or phits**. A phit is the **unit of information that is transferred across a channel in a single clock cycle**. Although no resources are allocated in units of phits, a link-level protocol must interpret the phits on the channel to find the boundaries between flits.

#### Bit-Length Summary

|Allocation Unit|Min (bits)|Typical (bits)|Max (bits)|
|---|---|---|---|
|Phit|1|8|64|
|Flit|16|64|512|
|Packet|128|1K|512K|

> **PYQ (2018 Q18, 2022 Q30, 2023 Q12, 2024 Q—):** The basic unit of routing and sequencing is a **(c) packet.**

> **PYQ (2018 Q19, 2022 Q31, 2024 Q15):** The basic unit of bandwidth and storage allocation is a **(a) flit.**

> **PYQ (2023 Q12):** The unit of information transferred across a channel in a single clock cycle is called a **(d) phit.**

> **PYQ (2024 Q16):** The typical bit-length of a phit is **(a) 8 bits**, with a maximum of 64 bits.

> **PYQ (2022 Q32):** The typical bit-length of a flit is **(d) 64 bits**, with a maximum of 512 bits.

> **PYQ (2025 Q19):** The maximum length of a flit is **(c) 512 bits.**

---

### Flow Control Methods

#### Bufferless Flow Control

In bufferless flow control there are no buffers at the nodes. When two packets arrive at a router simultaneously and both request the same output channel, there is no buffer to hold the loser. The flow control mechanism must immediately resolve the contention. **Dropping flow control** is one such technique — one packet (A) acquires the channel and the other (B) is dropped. B must be retransmitted from the source. A negative acknowledgment (NACK) triggers this retransmission.

**Example:** A five-flit packet is sent along a four-hop route. The first transmission is unable to allocate channel 3 and is dropped. A NACK triggers a retransmission of the packet which succeeds.

> **PYQ (2023 Q13):** Dropping flow control is a technique of **(b) bufferless flow control.**

#### Buffered Flow Control — Packet-Buffer Methods

Buffered flow control is more efficient than bufferless flow control. A buffer **decouples the allocation of adjacent channels** — it gives a place to store the packet (or flit) while waiting for the second channel to be allocated, allowing the allocation of the second channel to be delayed without complications.

There are two packet-buffer methods:

##### Store-and-Forward Flow Control

With store-and-forward flow control, each node along a route **waits until a packet has been completely received (stored)** before forwarding it to the next node. The packet must be allocated two resources before it can be forwarded: a **packet-sized buffer** on the far side of the channel, and **exclusive use of the channel**. Once the entire packet has arrived at a node and these two resources are acquired, the packet is forwarded to the next node.

While waiting to acquire resources, no channels are being held idle and only a single packet-sized buffer on the current node is occupied. However, store-and-forward has **high latency** because a node must wait for the complete packet before forwarding begins.

> **PYQ (2023 Q14):** In store-and-forward flow control, a node **(a) waits for a complete packet to be received before forwarding it.**

##### Cut-Through Flow Control

Cut-through flow control **forwards a packet as soon as the header is received** and resources (buffer and channel) are acquired, without waiting for the entire packet to be received. This significantly reduces latency compared to store-and-forward.

If the packet encounters contention — it cannot immediately acquire the next channel — it must wait. During this wait period, the already-received flits must be stored in buffers at the current node.

**Comparison:**

- Store-and-forward: **high latency** (waits for full packet at each hop).
- Cut-through: **lower latency** (forwards as soon as header received and resources available).

> **PYQ (2023 Q15):** Cut-through flow control **(b) forwards a packet as soon as the header is received.**

##### Shortcomings of Packet-Buffer Flow Control

1. **Buffers are allocated in units of packets.** This results in very inefficient use of buffer storage. The efficient method is flit buffering.
2. **Contention latency is increased.** A high-priority packet colliding with a low-priority packet must wait for the entire low-priority packet to be transmitted before it can acquire the channel.

> **PYQ (2019 Q18):** Packet-buffer flow control is inefficient because **(c) both (a) and (b)** — buffers are allocated in units of packets (wasteful), AND contention latency is increased.

#### Flit-Buffer (Wormhole) Flow Control

Flit-buffer flow control, also called **wormhole flow control**, operates like cut-through but with channels and buffers allocated to **flits rather than packets**. When the head flit of a packet arrives at a node, it must acquire three resources before it can be forwarded to the next node along the route:

1. A **virtual channel** (channel state) for the packet.
2. One **flit buffer**.
3. One **flit of channel bandwidth**.

Body flits of the packet use the virtual channel acquired by the head flit, and hence need only acquire a flit buffer and a flit of channel bandwidth to advance. The tail flit is handled like a body flit, but also **releases the virtual channel** as it passes.

A **virtual channel** holds the state needed to coordinate the handling of the flits of a packet over a channel — output channel, state, etc.

**Step-by-step traversal of a 4-flit packet [T,B,B,H][T,B,B,H] through a node:**

|Step|What happens|
|---|---|
|(a)|Head flit arrives. Virtual channel is idle (I). Desired upper (U) output channel is busy — allocated to the lower (L) input.|
|(b)|Head is buffered. Virtual channel in waiting (W) state. First body flit arrives.|
|(c)|Head and first body flit are buffered. Virtual channel still in waiting (W state, pending 2 cycles). Second body flit can't acquire a flit buffer.|
|(d)|Output virtual channel becomes available and is allocated to this packet. State moves to active (A). Head is transmitted to the next node.|
|(e)|Body flit moves.|
|(f)|Body flit moves.|
|(g)|Tail flit is transmitted and frees the virtual channel.|

> **PYQ (2022 Q33–Q35, 2024 Q17–Q19, 2025 Q14–Q16):** These questions trace the flit-by-flit traversal of a wormhole packet through a node over specific cycles. Apply the step-by-step sequence above: head arrives and waits for a virtual channel, body flits follow in order, tail flit releases the channel.

#### Virtual-Channel Flow Control

The key problem with wormhole flow control is **blocking**. When a packet B blocks while holding the sole virtual channel associated with physical channel pp, channels pp and qq are idled even though packet A requires the use of these idle channels.

**Virtual-channel flow control** overcomes the blocking problems of wormhole flow control by **associating several virtual channels with a single physical channel**. Each virtual channel has its own channel state and flit buffers. When packet B blocks, packet A is able to proceed over channels pp and qq using a second virtual channel associated with channel pp on node 2, using the channel bandwidth that would otherwise be left idle.

Virtual-channel flow control allows other packets to use the channel bandwidth that would otherwise be left idle when a packet blocks.

> **PYQ (2019 Q19):** Virtual-channel flow control overcomes the blocking problems of wormhole flow control **(c) by associating several virtual channels with a single physical channel.**

---

### Buffer Management and Backpressure

All buffered flow control methods need a means to communicate the availability of buffers at the downstream nodes. When the upstream node forwards a flit, it consumes a downstream buffer. Upstream nodes can determine when a buffer is available to hold the next flit (or packet for store-and-forward or cut-through) to be transmitted. This type of buffer management provides **backpressure** by informing the upstream nodes when they must stop transmitting flits because all of the downstream flit buffers are full.

Three common low-level flow control mechanisms that provide backpressure:

1. **Credit-based** — the upstream router keeps a count of the number of free flit buffers in each virtual channel downstream. Each time the upstream router forwards a flit, it decrements the appropriate count. If the count reaches zero, all downstream buffers are full and no further flits can be forwarded until a buffer becomes available. Once the downstream router forwards a flit and frees the associated buffer, it sends a **credit** to the upstream router, causing the buffer count to be incremented.
2. **On/off** — the downstream node signals the upstream node with an on/off signal indicating whether it can accept more flits.
3. **Ack/nack** — the downstream node sends an acknowledgment (ack) or negative acknowledgment (nack) for each flit, indicating success or failure of reception.