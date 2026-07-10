https://medium.com/@hnasr/how-vpns-really-work-a5da843d0eb3

Let's say you want to connect to Google (e.g. IP 1.2.3.4) port 80 let us assume your source ip is 6.6.6.6. This is really your public router IP and not your private laptop IP so I'm going to skip NAT for simplicity.

Normally with no VPN, your client sends a SYN segment to port 80 that goes into an IP packet with a destination IP 1.2.3.4 and source ip 6.6.6.6 and Google replies back directly to you with a SYN/ACK destination IP 6.6.6.6 and source IP 1.2.3.4 and this goes on. Your ISP sees the IP packet you are sending back and forth to 1.2.3.4. They can choose to deep inspect it, and see the content, they (the ISP and pretty much anyone in between) can do that in case of plaintext HTTP (port 80) but not really on HTTPS (port 443).

![[Pasted image 20260710213134.png]]

Now say you deploy a UDP based VPN, and you use a VPN server on IP 3.3.3.3. The client still produces the SYN ip packet with destination 1.2.3.4 and source ip 6.6.6.6 but then the vpn client captures that IP packet, encrypts it and puts it on a new UDP datagram with VPN info and that UDP goes into a new IP packet destination ip is 3.3.3.3 source is 6.6.6.6.


![[Pasted image 20260710213148.png]]

That ip packet is what leaves your NIC, your ISP see you going to 3.3.3.3 and not (1.2.3.4) because that is encrypted and encapsulated in that outer ip packet vpn server receives the ip packet unpack decrypt it (complex logic must find the key etc) then sees that aha this guy want to go to 1.2.3.4 and creates a brand new ip packet (or reuse for zero copy) changing the source ip to its own 3.3.3.3 so the SYN reaches Google.

![[Pasted image 20260710213159.png]]

Google replies back to 3.3.3.3 with SYN/ACK the VPN server which knows that this packet must go to you (6.6.6.6)*, so it creates a new IP packet with its source ip as 3.3.3.3 and destination ip 6.6.6.6 and puts the SYN/ACK in it.

> How does the VPN server know that this packet needs to go to 6.6.6.6? there might be many others who sent packets through the VPN to 1.2.3.4. That is why the VPN keeps a table of who connected to what. The IP address might not be enough to do the lookup so the VPN might use the source port to identify. And in some rare cases if two clients used the source source port, the VPN might have to change the source port too and not just the IP address.

So in summary the VPN does not terminate the TCP it simply passes the SYN all the way through so you get an end to end TCP connection between you and Google but through this encrypted tunnel.

Remember the same thing happen when you use TLS, the TLS client hello is forwarded all the way to Google through the VPN just like any other packet. That is why you also get an end-to-end encryption and the VPN cannot really read HTTPS traffic as well. That means yes you take the hit of double encryption and decryption when using VPN.

Why UDP and not TCP for VPN? You can but you might get TCP meltdown as the two congestion algorithm fight each other (outer and inner TCP connection). With UDP it is easy to retransmit only the lost packets and have a simpler retry logic without all the complexity of TCP.


Also read: https://medium.com/@ali.azaz.alam/behind-the-secrets-of-vpn-communication-for-beginners-%EF%B8%8F-1df2154ec844