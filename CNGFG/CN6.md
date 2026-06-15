The Physical Layer is the lowest OSI layer responsible for transmitting raw data bits through physical network hardware.

- Handles physical and electrical data transmission
- Includes cables, connectors, plugs, and receivers
- Transfers raw bits between devices
- Defines hardware standards and signal types

## Functions of Physical Layer

- The Physical Layer is responsible for sending raw data as bits over a physical medium.
- It converts data into signals that can travel through wires, fiber optics, or wireless channels (encoding) and turns these signals back into data at the receiver (decoding).
- It ensures signals are transmitted correctly and uses techniques like modulation to prepare the data for transmission and demodulation to retrieve it at the other end.
- This layer also decides how data flows (one-way, two-way alternately, or simultaneously) through transmission modes and controls the speed and timing of data transmission to keep everything running smoothly.

![[Pasted image 20260615091324.png]]
![[Pasted image 20260615091333.png]]
![[Pasted image 20260615091348.png]]
![[Pasted image 20260615091356.png]]


## Physical Topologies

Physical topologies describe the physical arrangement of devices and cables in a network. Below are the different topologies used in Computer Networks

### Line Configuration 

- **Point-to-Point configuration:** In Point-to-Point configuration, there is a line (link) that is fully dedicated to carrying the data between two devices.
- **Multi-Point configuration:** In a Multi-Point configuration, there is a line (link) through which multiple devices are connected.


## Protocols in Physical Layer

Typically, a combination of hardware and software programming makes up the physical layer. It consists of several protocols that control data transmissions on a network. The following are some examples of Layer 1 protocols:

- **Ethernet (IEEE 802.3)** : Widely used for wired networks.
- **Wi-Fi (IEEE 802.11)** : For wireless communication.
- **Bluetooth (IEEE 802.15.1)** : Short-range wireless communication.
- **USB (Universal Serial Bus)** : For connecting devices over short distances.

## Need of Physical Layer in Security

In security, Physical Layer is essential because many attacks can occur before any software is involved. Threats at this level target the actual hardware or transmission medium.

- **Cable Tapping:** Hackers can intercept data by directly connecting to network cables.
- **Physical Access**: If someone gets into server rooms or hardware areas without permission, they can steal or damage equipment.
- **Wireless Signal Interception**: Attackers can capture Wi-Fi signals from outside using special tools.
- **Hardware Manipulation**: Someone might tamper with physical devices like routers or USB ports to secretly install harmful software.