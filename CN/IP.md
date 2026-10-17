
The Internet Protocol is a protocol for transmitting blocks of data from source to destination among computer networks, where sources and destinations are hosts identified by an address called “_IP address”_. 

These blocks are called datagrams. The internet protocol provides for the fragmentation and reassembly of long datagrams.

## IP address

The IP address is a property of the Network Layer in the OSI Model. 

![Pasted image 20260615142337](../assets/Pasted%20image%2020260615142337.png)

An IP address is divided into 2 parts: a network portion and a host portion. 

The network address is used to identify the network and is common to all the devices attached to the network. 

The host address is used to identify a particular device attached to the network.

This is actually the IPv4 address.

**How do you split those 32 bits between “network” and “host”?**

If you give too many bits to the network portion, you get lots of possible networks but each network can only hold a few hosts. If you give too many bits to the host portion, each network can hold millions of hosts but you can only define a tiny number of networks total.

### Classful addressing

This is an outdated system, so you can skip it.

![](../assets/Pasted%20image%2020261016104117.png)

![](../assets/Pasted%20image%2020261016104132.png)

The 32-bit address space is divided into 5 parts: A, B, C, D, E

![](../assets/Pasted%20image%2020261016104532.png)

But classful addressing wastes IP addresses. When an organization is assigned a Class B network, they get the _entire block_ of 65,534 host addresses — all at once, as a unit. They can’t say “give me just 5,000 of them.” It’s all-or-nothing within that class.

So picture this real scenario: a university needs 4,000 host addresses. There is no Class in between B (65,534 hosts) and C (254 hosts) that fits. They’re forced to take a Class B block, because Class C is too small.

Now they’re using 4,000 addresses out of their 65,534. The remaining ~61,000 addresses in that block are now **unusable by anyone else in the world** — because the whole block belongs exclusively to this university, whether they use it or not. No other organization can take “the unused half” of someone else’s Class B block.

That’s the waste. Not “the bits are invalid” — but “these addresses are permanently assigned to one entity who isn’t using them, and nobody else can have them either.”

### Classless Internet addressing

In 1990s, the number of bits used for the network portion of an IP address became variable instead of fixed.

### Special addresses

Normally, when you send a packet, you address it to _one specific_ destination — one IP, one device. But sometimes a device needs to reach **every device on the network at once**, without knowing their individual addresses in advance.

An example is the DHCP protocol. 

You connect your laptop to a new Wi-Fi network (say, a coffee shop). Your laptop has no IP address yet. It needs to ask: “can someone give me an IP address?”

Who do you send that request _to_? You don’t have an IP address yet, and you don’t know the IP address of the DHCP server (the machine that hands out addresses) either. You have no specific destination to address the packet to.

So your laptop sends the request to the **broadcast address** on the local network, essentially saying “to everyone listening on this network: I need an IP address, can the DHCP server respond?” Every device on the LAN receives this packet, most ignore it, but the DHCP server recognizes the request and replies.

This is exactly why broadcast exists, for “I don’t know who to talk to, but someone out there can help” situations. DHCP is the most common real-world example, but things like ARP requests (“who owns this IP address?”) work the same way.

![](../assets/Pasted%20image%2020261016105219.png)

#### Limited Broadcast

This is “everyone on _my own_ local network, right here, right now.” It never leaves your LAN — routers are specifically designed to **not forward** this address onward to other networks. It stays confined to the segment you’re physically/directly connected to.

This is what your laptop uses for that DHCP request above — it doesn’t know what network it’s even on yet, so it just says “all 1s” meaning “everyone nearby, no network info needed.”

#### Directed Broadcast

This is different — it lets you broadcast to _every host on a specific remote network_, from anywhere. Example: if a network’s address is `192.168.5.0/24`, the directed broadcast address would be `192.168.5.255`. You could send this packet from completely outside that network, it would travel through the internet, arrive at the router owning that network, and _that router_ would then flood it to every host inside `192.168.5.0/24`.

**Why directed broadcast became a security problem: The Smurf attack**

This is the actual historical reason it got killed off. Here’s the attack:

An attacker sends a ping (ICMP echo request) to a directed broadcast address — but _spoofs the source IP_ to be the victim’s IP address instead of their own. The router receiving this floods the ping to every single host on that network. Every one of those hosts then sends a reply — but since the source was spoofed, **all those replies go to the victim**, not the attacker.

So one single packet from the attacker gets multiplied into hundreds or thousands of reply packets, all slamming into the victim simultaneously. This is a **traffic amplification DDoS attack** — the attacker uses an entire network as an unwitting weapon against one target, using almost no bandwidth themselves.

The goal is **denial of service** — meaning your device (or a server) becomes _unable to do its actual job_ because it’s drowning in junk traffic.

This is exactly why the slide says routers now simply **drop** any packet where the host bits are all 1s instead of forwarding it — it closes off this entire attack vector. You’re left with only Limited Broadcast (`255.255.255.255`), which is inherently safe because it physically cannot leave your own local network in the first place.

![](../assets/Pasted%20image%2020261016110234.png)



**How to know the network address and host address? Subnet masks are used.**

example:

For example, we have an IP address is 192.168.2.4 and a subnet mask is 255.255.255.240

Let's convert them to binary!

_IP address = 192.168.2.4 = 11000000.10101000.00100000.01000000_

_Subnet mask = 255.255.255.240 = 11111111.11111111.11111111.11110000_

Bits in the subnet mask which are 1 denote the network address. The bits that are 0 denote the host address.

![Pasted image 20260615142644](../assets/Pasted%20image%2020260615142644.png)

Perform a bitwise AND operation to find the network address. Thus, the network address for this example is _11000000.10101000.00000010.00000000_ to decimal is _192.168.2.0_

We can see that the last 4 bits of the subnet mask are 0. This means we can have 2⁴ = 16 unique host addresses. However, **2 addresses are reserved for the network address and broadcast address. So we can have effectively 14 host addresses.**


![](../assets/Pasted%20image%2020261016113212.png)

### Default gateway

Most networks consist of hosts and default gateway because those hosts want to talk somewhere outside their subnet (the network).

![Pasted image 20260615142940](../assets/Pasted%20image%2020260615142940.png)

E.g: 192.168.1.3 want to talk to 192.168.2.2

![Pasted image 20260615143001](../assets/Pasted%20image%2020260615143001.png)


## What is an IP packet?

IP packets are created by adding an IP header to each packet of data before it is sent on its way. An IP header is just a series of bits (ones and zeros), and it records several pieces of information about the packet, including the sending and receiving IP address. IP headers also report:

- Header length
    
- Packet length
    
- [Time to live (TTL)](https://www.cloudflare.com/learning/cdn/glossary/time-to-live-ttl/), or the number of network hops a packet can make before it is discarded
    
- Which transport protocol is being used (TCP, UDP, etc.)
    
In total there are 14 fields for information in IPv4 headers, although one of them is optional.

## What is TCP/IP?

The Transmission Control Protocol (TCP) is a transport protocol (meaning it dictates the way data is sent and received). 

A TCP header is included in the data portion of each packet that uses [TCP/IP](https://www.cloudflare.com/learning/ddos/glossary/tcp-ip/). 

Before transmitting data, TCP opens a connection with the recipient. TCP ensures that all packets arrive in order once transmission begins. 

The recipient will acknowledge receiving each packet that arrives. Missing packets will be sent again if receipt is not acknowledged.

TCP is designed for reliability, not speed. Because TCP has to make sure all packets arrive in order, loading data via TCP/IP can take longer if some packets are missing.

**TCP and IP were originally designed to be used together, and these are often referred to as the TCP/IP suite. However, other transport protocols can be used with IP.**

## What is UDP/IP?

The User Datagram Protocol, or [UDP](https://www.cloudflare.com/learning/ddos/glossary/user-datagram-protocol-udp/), is another widely used transport protocol. It is faster than TCP, but it is also less reliable. UDP does not make sure all packets are delivered and in order, and it does not establish a connection before beginning or receiving transmissions.

## IPv4 vs. IPv6

The key differences between IPv4 and IPv6 are primarily related to the address format:

1. **Address Length**:

- IPv4 uses 32-bit addresses, allowing for approximately 4.3 billion unique addresses.
- IPv6 uses 128-bit addresses, offering an astronomical number of unique addresses (340 undecillion, to be precise).

2. **Address Notation**:

- IPv4 addresses are expressed in a dotted-decimal format, like 192.168.1.1.
- IPv6 addresses use hexadecimal notation separated by colons, such as 2001:0db8:85a3:0000:0000:8a2e:0370:7334.

3. **Network Configuration**:

- IPv4 relies heavily on Network Address Translation (NAT) to handle address shortages.
- IPv6 eliminates the need for NAT by providing ample address space, simplifying network configuration.



## Types of IP Addresses

There are several types of IP addresses, each serving a specific purpose:

1. **Public IP Addresses**: These are unique addresses assigned to devices directly connected to the internet. Public IP addresses are necessary for devices like web servers, allowing them to be accessible from anywhere on the internet.

2. **Private IP Addresses**: Private IP addresses are used within local networks, such as your home or office. They are not directly accessible from the internet and are often assigned by routers or DHCP (Dynamic Host Configuration Protocol) servers.

3. **Static IP Addresses**: Static IP addresses remain constant and do not change. They are typically used for critical devices like servers, making it easier to locate them on the network.

4. **Dynamic IP Addresses**: Dynamic IP addresses are assigned to devices on a temporary basis. Internet Service Providers (ISPs) commonly use dynamic IP addresses for home users, which can change periodically.


### Sending data using IP addresses

- **Data Packetization**: Your data, whether it’s a text message, image, or video, is divided into small packets.
- **Source and Destination Addressing**: Each packet is tagged with the source IP address (the address of your device) and the destination IP address (the address of the recipient’s device).
- **Routing**: Routers and network devices in between determine the best path for the packets to reach their destination.
- **Packet Transmission**: Packets are transmitted across the network, and they may take different routes to reach the same destination.
- **Reassembly**: The recipient’s device receives the packets and reassembles them in the correct order to reconstruct the original data.
- **Data Delivery**: The reconstructed data is delivered to the recipient, completing the communication process.

References:

https://www.cloudflare.com/learning/network-layer/internet-protocol/
https://medium.com/@nghiadt.dev/what-is-the-internet-protocol-ip-d6e0bd89db63
https://tsharma2907.medium.com/understanding-ip-internet-protocol-a-comprehensive-guide-0b7301d6421a