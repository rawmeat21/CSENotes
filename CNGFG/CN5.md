The TCP/IP model is a layered networking framework that explains how data is communicated between devices over a network using standardized protocols to ensure reliable and efficient transmission.

- Defined as a four-layer architecture consisting of Application, Transport, Internet, and Network Access layers.
- Standardized by RFC 1122, which specifies its structure and behavior.
- Simpler and more practical than the seven-layer OSI model.
- Serves as the core framework of the modern Internet and networking systems.

![[Pasted image 20260615085504.png]]

## Layers of TCP/IP Model

### 1. Application Layer

This is the top layer of the TCP/IP model, where applications like web browsers, email clients, and file-sharing tools interact with the network.

- Acts as a bridge between user applications and lower network layers.
- Supports protocols such as HTTP, FTP, SMTP, and DNS.
- Handles data formatting so information is correctly understood by both sender and receiver.
- Provides encryption for secure communication.
- Manages sessions to track ongoing connections.

### 2. Transport Layer

Ensures reliable and efficient delivery of data between devices, managing segmentation, ordering, and retransmission as needed.

- Breaks large messages into packets and reassembles them at the destination.
- TCP checks for errors, resends lost data, and ensures correct order.
- UDP provides low-latency, connectionless delivery without error checking.
- Prevents the receiver from being overwhelmed by regulating data flow.
- Uses port numbers to allow multiple applications to share the network simultaneously.

![[Pasted image 20260615085556.png]]

**TCP (Transmission Control Protocol):** TCP is used when reliability and accuracy are important. It ensures that data is delivered exactly as sent.

- TCP detects errors in the data using checksums to ensure integrity.
- If any data is lost or corrupted during transmission, TCP automatically resends it.
- Data is broken into packets, and TCP ensures these packets arrive in the correct sequence.
- TCP establishes a connection between sender and receiver before sending data, maintaining a stable session throughout the communication.
- Loading web pages, downloading files, sending emails, or any application where data must be complete and accurate.

**UDP (User Datagram Protocol):** UDP is used when speed is more important than perfect accuracy. It is faster but does not guarantee reliable delivery.

- UDP detects errors via checksums but does not verify or recover from them.
- Lost or damaged data is not resent.
- Packets may arrive out of order, and the protocol does not fix it.
- Because it avoids extra checks and connections, UDP is faster and uses fewer resources.
- Live video streaming, online gaming, VoIP calls, or real-time applications where speed matters more than reliability.

### 3. Internet Layer

Responsible for addressing, packaging, and routing data packets so they can travel across networks and reach the correct destination device. It ensures that data can move between different networks efficiently.

![[Pasted image 20260615085631.png]]

### Network Access (Link Layer)

Responsible for physically transmitting data over network hardware, including cables, switches, and wireless connections. It handles how data is formatted for the network medium and ensures it reaches the next device on the path.

