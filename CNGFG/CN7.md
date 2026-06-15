# Network Topology

Network topology is the arrangement of devices (nodes) and connections (links) in a computer network.

There are two main types of topology:

- **Physical Topology:** The actual physical layout of cables and devices.
- **Logical Topology:** How data moves across the network, regardless of physical layout



## Point To Point Topology

Point-to-point topology is a type of topology that works on the functionality of the sender and receiver. It is the simplest communication between two nodes, in which one is the sender and the other one is the receiver. Point-to-Point provides high bandwidth.

![[Pasted image 20260615091718.png]]


Read this from Claude

# Transmission Modes

Transmission Modes describe how data is exchanged between two devices over a communication channel.

## Simplex Mode

Simplex Mode is a transmission mode in which communication occurs in only one direction, from a sender to a receiver, without any possibility of reverse data flow.

- One device acts only as transmitter
- The other device acts only as receiver
- No feedback or acknowledgment is supported

![[Pasted image 20260615091838.png]]

### **Advantages**

- Simple and easy to implement
- Lower installation and maintenance cost
- Utilizes the entire bandwidth for sending
- Suitable for broadcast-type applications

### **Disadvantages**

- No error reporting from receiver
- Not suitable for interactive systems
- Cannot confirm successful delivery
- Limited use in modern two-way communication systems

## Half-Duplex Mode

Half-Duplex Mode is a transmission mode in which communication can occur in both directions, but only one device can transmit at a time. The devices take turns sending and receiving data over the same channel.

- Supports bidirectional communication
- Uses a single shared communication channel
- Transmission occurs alternately between devices

![[Pasted image 20260615091931.png]]

### **Advantages**

- More flexible than one-way communication
- Cost-effective compared to full-duplex systems
- Efficient use of a single channel
- Suitable for controlled communication environments

### **Disadvantages**

- Cannot send and receive simultaneously
- Possible delay due to turn-based transmission
- Performance decreases with heavy traffic
- Risk of collision if control mechanisms fail

## Full-Duplex Mode

Full-Duplex Mode is a transmission mode in which communication takes place in both directions at the same time. Both connected devices can transmit and receive data simultaneously without waiting for each other.

- Enables simultaneous two-way data exchange
- Uses separate channels or divided bandwidth
- Common in real-time communication systems

![[Pasted image 20260615092014.png]]

### **Advantages**

- Faster data transfer rate
- No waiting time between transmissions
- Improved communication efficiency
- Suitable for interactive applications

### **Disadvantages**

- Higher installation cost
- Requires more complex hardware
- Greater bandwidth requirement
- Increased system configuration complexity