
The Internet Protocol is a protocol for transmitting blocks of data from source to destination among computer networks, where sources and destinations are hosts identified by an address called “_IP address”_. 

These blocks are called datagrams. The internet protocol provides for the fragmentation and reassembly of long datagrams.

### IP address

The IP address is a property of the Network Layer in the OSI Model. 

![[Pasted image 20260615142337.png]]

An IP address is divided into 2 parts: a network portion and a host portion. 

The network address is used to identify the network and is common to all the devices attached to the network. 

The host address is used to identify a particular device attached to the network.

**How to know the network address and host address? Subnet masks are used.**

example:

For example, we have an IP address is 192.168.2.4 and a subnet mask is 255.255.255.240

Let's convert them to binary!

_IP address = 192.168.2.4 = 11000000.10101000.00100000.01000000_

_Subnet mask = 255.255.255.240 = 11111111.11111111.11111111.11110000_

Bits in the subnet mask which are 1 denote the network address. The bits that are 0 denote the host address.

![[Pasted image 20260615142644.png]]

Perform a bitwise AND operation to find the network address. Thus, the network address for this example is _11000000.10101000.00000010.00000000_ to decimal is _192.168.2.0_

We can see that the last 4 bits of the subnet mask are 0. This means we can have 2⁴ = 16 unique host addresses. However, **2 addresses are reserved for the network address and broadcast address. So we can have effectively 14 host addresses.**

### Default gateway

Most networks consist of hosts and default gateway because those hosts want to talk somewhere outside their subnet (the network).

![[Pasted image 20260615142940.png]]

E.g: 192.168.1.3 want to talk to 192.168.2.2

![[Pasted image 20260615143001.png]]


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