# Network Devices

![[Pasted image 20260615015036.png]]

## Layer 1 Devices: Physical Layer

These devices deal with raw electrical or optical signals (bits). These devices do not understand IP addresses or MAC addresses.

### 1. Hub

It is a central connection point for devices in a LAN.

- It takes an incoming signal on one port and blindly broadcasts it to all other ports without knowing the recipient.
- High network traffic collisions and security risks (everyone sees everyone's data).
- It is a obsolete device, replaced by Switches.

### 2. Repeater

It regenerates signals to extend the range of a network.

- It receives a weak signal (due to attenuation over long cables), amplifies it, and retransmits it.
- Extending WiFi range or Ethernet cables beyond 100 meters.

### 3. Modem (Modulator-Demodulator)

This bridges the gap between digital computer data and analog signals, allowing devices to connect to the internet via telephone, cable, or fiber lines.

- Connects your home network to the ISP (Internet Service Provider).
- Converts digital signals from your computer into analog signals for telephone/cable lines (Modulation) and vice-versa (Demodulation).

## Layer 2 Devices: Data Link Layer

These devices work with MAC Addresses (Physical Addresses). They are smarter than hubs.

### 4. Switch

A switch is a high-speed networking device that connects devices (computers, printers, servers) within a Local Area Network (LAN), 

- Unlike hub, switch learns the MAC address of every connected device.
- It sends data only to the specific port where the destination device is connected.
- Reduces collisions and improves security and speed.

### 5. Bridge

This connects two separate LAN segments to make them appear as one.

- Filters traffic based on MAC addresses to keep local traffic local, reducing congestion.
- Largely replaced by Switches (which are essentially multi-port bridges).

### 6. Access Point (AP)

It is a networking hardware device that allows Wi-Fi-enabled devices, such as laptops and smartphones, to connect to a wired network.

- Acts as a bridge between wired Ethernet and wireless Wi-Fi devices.
- An AP only provides Wi-Fi signal and it does not route traffic or assign IP addresses.

## Layer 3 Devices: Network Layer

These devices work with IP Addresses (Logical Addresses) and connect different networks together.

### 7. Router

It is a networking device that connects multiple networks and directs data packets between them using IP addresses.

- It uses IP Addresses to determine the best path for data packets to travel.
- It maintains a Routing Table to make these decisions.
- Routers stop broadcast traffic, isolating networks from each other.

### 8. Brouter (Bridging Router)

It is a networking device combining bridge and router functions, operating at both the Data Link and Network layers of the OSI model.

- It can route known protocols (like TCP/IP).
- It bridge unknown protocols (like non-routable legacy traffic) at the same time and rarely used today.

## Layer 4-7 Devices: Advanced Processing

### 9. Gateway

It is a hardware device or software node that acts as an entry/exit point between two distinct networks using different protocols, translating data to ensure compatibility. 

- It can convert protocols, data formats, or architectures (e.g., connecting a TCP/IP network to a legacy Mainframe network).
- An Internet Gateway translates your private LAN requests into public internet requests.

### 10. Firewall

It is a network security system, comprising hardware, software, or both that monitors and controls incoming and outgoing network traffic based on predetermined security rules

- It is a physical appliance at the network edge.
- This is installed on a specific server or PC.
- It tracks active connections.


![[Pasted image 20260615015135.png]]
