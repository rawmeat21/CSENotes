![](../assets/Pasted%20image%2020261016095905.png)

The network layer requires knowledge of network topology (the graph layout of interconnected routers and physical links) for four primary operational reasons:

- **Path Computation and Routing:** To route packets from a source to a destination across intermediate hops, network layer protocols must model the network as a graph. Routing algorithms (such as Dijkstra's in link-state protocols or Bellman-Ford in distance-vector protocols) need node connections and edge weights to calculate shortest or least-cost paths.
    
- **Load Balancing and Link Optimization:** Without knowing available alternative paths and link capacities, the network layer cannot balance traffic efficiently. Topology awareness allows routers to distribute flows across multiple paths to avoid overloading specific links while leaving available bandwidth unused.
    
- **Fault Tolerance and Dynamic Re-routing:** Physical links and routers continuously experience failures or state changes. Knowing the network structure enables routing protocols to detect topology changes, prevent routing loops, and fast-converge onto alternate viable paths.
    
- **Congestion Avoidance and Inter-networking:** Topology mapping exposes network boundaries, subnet topologies, and bottleneck links. This structural context is essential for applying end-to-end congestion control and forwarding traffic across diverse subnets.

![](../assets/Pasted%20image%2020261016100111.png)

### Why does routing need addressing at all?

When a packet leaves your machine, it needs to travel through multiple routers before reaching its destination. Each router in the middle has no idea what the final destination is, unless the packet _carries that information with it_.

### Now, why not just use MAC addresses?

Instead of thinking about computers, think about a person named John Doe.

John Doe currently lives at:

123 Cisco Way 66778 New York

In this example:

- John Doe is the MAC address.
    
- 123 Cisco Way, 66778 New York is the IP address.
    

If someone wants to send John a letter, they don't just write "John Doe" on the envelope. They need an address so the postal service knows where to deliver it. They cant just write 123 cisco way either because then when it arrives no one will know who its meant for.

The letter is sent to John Doe 123 Cisco Way, 66778 New York and so the post office knows where to send the letter and the people there know who the letter is for.

Now imagine John moves to:

345 Ruckus Ave 78653 Boston, Massachusetts

John is still the same person, but his address has changed.

This is similar to networking. A device's IP address can change when it moves to a different network, but the MAC address remains the same because it identifies the specific network interface.

The IP address tells the network where to send the traffic. The MAC address identifies which device the traffic belongs to once it reaches the local network.

In simple terms:

- IP Address = Where is it?
    
- MAC Address = Who is it?
    

You need both because one identifies the location, and the other identifies the specific device at that location.


A MAC address looks like this: `3C:22:FB:A1:09:DE`

It’s just a flat serial number. There is _no geography_ in it. It tells you nothing about where that device is located in the world. The manufacturer assigned it. It encodes the vendor, not the location.

Now think about what a router needs to do. A router in Mumbai receives a packet. It needs to forward it. To decide _where_ to forward it, the router needs to consult a table. That table maps destinations to next hops.

If you used MAC addresses, that table would have to contain **every single device on the entire internet**. There are roughly 15–20 billion connected devices today. Every router, everywhere, would need a table with 15 billion entries. And every time a new device joins the network, every router on Earth updates its table.

#### IP addresses solve this with hierarchy

An IP address like `192.168.1.45` encodes _location_ in the network, not device identity. The `192.168.1` part tells you the _network_, and `45` tells you the specific host within it.

This means routers can think in _blocks_. A router in Frankfurt doesn’t need to know every device in India. It just needs to know: “anything starting with this prefix, send it toward this direction.” That’s called **prefix-based routing**, and it’s what makes the internet scalable.

The routing table stays manageable because it stores _network prefixes_, not individual devices.

- If the Internet routed purely on MAC addresses, core routers could not perform route aggregation. Because adjacent MAC addresses might be located in different continents, every core Internet router would have to maintain a routing entry for every single network interface on Earth (billions of entries). This would instantly overwhelm router TCAM memory and lookup algorithms (O(N) routing table complexity).
    
- With IP hierarchy, core routers only store thousands of aggregated network prefixes (O(subnets)), forwarding traffic toward the target network chunk by chunk until it reaches the destination's local subnet, where the final router uses ARP/NDP to map the IP address to a local MAC address.


![](../assets/Pasted%20image%2020261016102737.png)

X and Y are on two _physically different_ networks, maybe one is Ethernet, the other Token-Ring, or just two separate network segments connected through different hardware. They can’t talk to each other directly. The packet has to pass through routers A, B, C to get from X to Y.

Layers 1 and 2 (Physical and Data Link) depend on the actual hardware technology, Ethernet uses one framing format, Token-Ring uses a different one. They’re not compatible with each other at that level.

But Layer 3 (Network layer) doesn’t care about the underlying hardware. An IP packet looks the same whether it’s traveling over Ethernet or Token-Ring or anything else.

Look at router A in the diagram. Trace what happens to the packet as it moves through A’s stack:

1. The packet arrives at A’s **Physical layer** (layer 1) — raw bits come in over the wire.
2. It moves up to A’s **Data Link layer** (layer 2) — this layer reads the hardware-specific framing (MAC addresses, Ethernet/Token-Ring headers) and strips that header off.
3. Once stripped, what’s left is passed up to A’s **Network layer** (layer 3) — this is the IP packet, hardware-independent.
4. Router A looks at the IP packet, checks the destination IP address, and decides: “this needs to go out toward B next.”
5. Now the packet needs to go back _down_ — but this time it’s re-wrapped with a _new_ Data Link layer header, appropriate for whatever physical network connects A to B.
6. It goes out through A’s Physical layer again, now addressed for the A→B link.

![](../assets/Pasted%20image%2020261016103457.png)

