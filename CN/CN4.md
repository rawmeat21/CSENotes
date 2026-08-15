# Layers of OSI Model

The OSI Model is a conceptual framework created by the International Organization for Standardization (ISO) to describe how data is transmitted across a network using a structured seven-layer architecture.

- Divides network communication into seven functional layers.
- Assigns specific responsibilities to each layer.
- Promotes compatibility between different networking systems.
- Simplifies network design, implementation, and troubleshooting.

![[Pasted image 20260615015642.png]]

### Layer 1: The Physical Layer

The [**Physical Layer**](https://www.geeksforgeeks.org/computer-networks/physical-layer-in-osi-model/) is the foundation of the OSI model, acting as the bridge for actual physical connections between devices. Its primary mission is the transmission of raw, unstructured bitstreams over a physical medium from one node to the next.

- **Core Responsibility:** It manages the hardware-level transmission of individual bits. When receiving data, it translates incoming physical signals back into digital 0s and 1s to be processed by the Data Link layer.
- ****Essential Hardware:**** Common devices operating at this level include [Hubs](https://www.geeksforgeeks.org/computer-networks/what-is-network-hub-and-how-it-works/), [Repeaters](https://www.geeksforgeeks.org/computer-networks/repeaters-in-computer-network/), [Modems](https://www.geeksforgeeks.org/computer-networks/what-is-modem/), and [Cables](https://www.geeksforgeeks.org/computer-networks/types-of-ethernet-cable/).
- ****Bit Synchronization:**** It ensures the sender and receiver are "in sync" by providing a clock signal that controls the timing of bit transmission.
- ****Bit Rate Control:**** It dictates the transmission speed, defined as the number of bits sent per second.
- ****Physical**** [****Topologies****](https://www.geeksforgeeks.org/computer-networks/types-of-network-topology/)****:**** It defines the structural layout of the network, such as Bus, Star, or Mesh configurations.
- ****Transmission Mode:**** It determines the direction of data flow, utilizing modes like Simplex (one-way), Half-Duplex (two-way, one at a time), or Full-Duplex (simultaneous two-way).

![[Pasted image 20260615015720.png]]

### Layer 2: The Data Link Layer (DLL)

The [****Data Link Layer****](https://www.geeksforgeeks.org/computer-networks/data-link-layer/) serves as the bridge between the physical hardware and the logical network, ensuring reliable node-to-node delivery of data. Its primary goal is to ensure that data transfer is error-free across the physical medium.

****1. Data Packaging (Framing):**** At this layer, data packets received from the Network layer are divided into manageable units called Frames. This is done by adding specific "start" and "stop" bits so the receiver can recognize where one unit of data ends and the next begins.

****2. Addressing and Hardware:**** The DLL uses [****MAC addresses****](https://www.geeksforgeeks.org/computer-networks/mac-address-in-computer-network/) (Physical Addressing) to identify hosts. It encapsulates the sender’s and receiver’s MAC addresses into the frame header. Common devices operating here are [****Switches and Bridges****.](https://www.geeksforgeeks.org/computer-networks/difference-between-switch-and-bridge/)

****3. Sublayers:****

- [****Logical Link Control (LLC)****](https://www.geeksforgeeks.org/computer-networks/logical-link-control-llc-protocol-data-unit/)****:**** Manages flow and error control while acting as an interface for upper-layer protocols.
- [****Media Access Control (MAC)****](https://www.geeksforgeeks.org/computer-networks/mac-address-in-computer-network/)****:**** Determines how devices physically access the transmission medium and handles hardware addressing.

****4. Error Control:**** It detects and retransmits damaged or lost frames to maintain data integrity.

****5. Flow Control:**** It synchronizes the data rate between a fast sender and a slower receiver to prevent data "bottlenecks" or loss.

****6. Access Control:**** When multiple devices share the same communication channel, the MAC sublayer dictates which device has the right to transmit at any given time to avoid collisions.

![[Pasted image 20260615015915.png]]
### Layer 3: The Network Layer

The [****Network Layer****](https://www.geeksforgeeks.org/computer-networks/network-layer-in-osi-model/) manages data transmission between hosts across different networks by handling logical addressing and path finding.

- ****Data Unit:**** Data segments are encapsulated into Packets.
- ****Logical Addressing:**** Assigns unique [****IP addresses****](https://www.geeksforgeeks.org/computer-science-fundamentals/what-is-an-ip-address/) (sender and receiver) to the packet header to identify devices globally.
- ****Routing:**** Determines the most efficient physical path from the source to the destination across interconnected networks.
- ****Hardware:**** Primarily implemented via [****Routers and Switches****](https://www.geeksforgeeks.org/computer-networks/difference-between-router-and-switch/).
- ****Inter-networking:**** Facilitates communication between disparate networks by directing traffic to the correct destination.
![[Pasted image 20260615015901.png]]
### Layer 4: The Transport Layer

The [Transport Layer](https://www.geeksforgeeks.org/computer-networks/transport-layer-in-osi-model/) ensures the end-to-end delivery of entire messages. It acts as a liaison, providing services to the Application layer while utilizing the infrastructure of the Network layer to ensure data reaches the correct application on the destination host.

- ****Data Unit:**** Data is broken down into Segments.
- ****Service Point Addressing:**** Uses Port Numbers (e.g., Port 80 for web traffic) to ensure data is delivered to the specific process or application intended, not just the device.
- ****Segmentation & Reassembly:**** Splits large messages into smaller segments for transmission and reassembles them in the correct order at the destination.
- ****Protocols:**** Common protocols include [****TCP****](https://www.geeksforgeeks.org/computer-networks/what-is-transmission-control-protocol-tcp/) (reliable), [****UDP****](https://www.geeksforgeeks.org/computer-networks/user-datagram-protocol-udp/) (fast).
- ****Connection-Oriented (TCP):**** Requires a "handshake" to establish a connection; ensures reliability via error checking and acknowledgements.
- ****Connectionless (UDP):**** Sends data immediately without a formal connection; faster but offers no guarantee of delivery.

![[Pasted image 20260615015847.png]]
### Layer 5: The Session Layer

The [****Session Layer****](https://www.geeksforgeeks.org/computer-networks/session-layer-in-osi-model/) acts as the "dialogue manager," governing the opening, closing, and security of communication channels between two devices.

- ****Session Lifecycle:**** Manages the establishment, maintenance, and termination of connections between applications.
- ****Authentication & Security:**** Ensures that the communicating parties are verified and the connection is secure.
- ****Synchronization:**** Inserts checkpoints into the data stream. This allows a transfer to resume from the last saved point if a failure occurs, rather than restarting from the beginning.
- ****Dialog Control:**** Directs whether communication is half-duplex (alternating) or full-duplex (simultaneous).
- ****Practical Example:**** In a web-based messenger, the Session Layer maintains the active link between your browser and the server, ensuring your specific chat remains open and synchronized while handling background encryption and data conversion.

![[Pasted image 20260615015836.png]]
### Layer 6: The Presentation Layer

Often called the ****Translation Layer****, the [Presentation Layer](https://www.geeksforgeeks.org/computer-networks/presentation-layer-in-osi-model/) ensures that data is formatted, secured, and compressed so that the receiving application can correctly interpret it.

- ****Translation:**** It extracts data from the Application Layer and converts it into a standardized format for network transmission, bridging differences in data representation between different systems.
- ****Standards & Formats:**** Handles the encoding of media using standards such as [JPEG, MPEG](https://www.geeksforgeeks.org/computer-graphics/difference-between-jpeg-and-mpeg/), and [GIF](https://www.geeksforgeeks.org/general-knowledge/what-is-a-gif-file/).
- ****Encryption/Decryption:**** Provides security by converting "plain text" into "ciphertext" (encryption) and back again (decryption) using key values. This process is typically handled by protocols like [****TLS/SSL****](https://www.geeksforgeeks.org/computer-networks/difference-between-secure-socket-layer-ssl-and-transport-layer-security-tls/).
- ****Compression:**** Reduces the total number of bits required for transmission, which increases network efficiency and speed.

![[Pasted image 20260615015825.png]]
### Layer 7: The Application Layer

The [****Application Layer****](https://www.geeksforgeeks.org/computer-networks/application-layer-in-osi-model/) sits at the top of the OSI stack and serves as the direct interface between the software user and the network. It produces the data that will be sent and displays the information received from other layers.

- ****User Interface:**** Acts as a "window" for network-based applications (like web browsers or email clients) to access network services.
- ****Core Protocols:**** Utilizes high-level protocols such as [****HTTP/S****](https://www.geeksforgeeks.org/computer-networks/difference-between-http-and-https-2/) (Web), [****SMTP****](https://www.geeksforgeeks.org/computer-networks/simple-mail-transfer-protocol-smtp/) (Email), [****FTP****](https://www.geeksforgeeks.org/computer-networks/file-transfer-protocol-ftp-in-application-layer/) (File Transfer), and [****DNS****](https://www.geeksforgeeks.org/computer-networks/domain-name-system-dns-in-application-layer/) (Domain Name Resolution).
- ****Network Virtual Terminal (NVT):**** Enables users to log into and interact with remote hosts as if they were physically present at the terminal.
- ****File Transfer, Access, and Management (FTAM):**** Provides the framework for users to retrieve, manage, and manipulate files stored on remote computers.
- ****Directory Services:**** Offers distributed database access to manage global information regarding various network objects and services.

![[Pasted image 20260615015813.png]]


## Data Flows in the OSI Model

When we transfer information from one device to another, it travels through 7 layers of OSI model. First data travels down through 7 layers from the sender's end and then climbs back 7 layers on the receiver's end. Data flows through the OSI model in a step-by-step process:

- ****Application Layer:**** Applications create the data.
- ****Presentation Layer:**** Data is formatted and encrypted.
- ****Session Layer:**** Connections are established and managed.
- ****Transport Layer:**** Data is broken into segments for reliable delivery.
- ****Network Layer:**** Segments are packaged into packets and routed.
- ****Data Link Layer:**** Packets are framed and sent to the next device.
- ****Physical Layer:**** Frames are converted into bits and transmitted physically.

Each layer adds specific information to ensure the data reaches its destination correctly, and these steps are reversed upon arrival.

