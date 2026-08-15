## Working of App layer

**What actually happens in a standard HTTP request**

Every network endpoint is an `IP:port` pair on both sides. When your phone makes a HTTP request to a server, here's what's really going on:

1. Your OS picks a random **ephemeral port** (say, 54231) for that connection. This is done automatically by the OS, not by your server.
2. Your phone sends a TCP SYN packet to `your-pc-ip:8080`. (say)
3. Your PC's OS completes the TCP handshake (SYN-ACK, ACK). This is the 3-way handshake — it's real, but it happens at the OS networking layer, invisibly.
4. Now there's an established TCP connection: `phone-ip:54231 ↔ your-pc-ip:8080`.
5. Over that connection, your phone sends an HTTP request (GET /users, whatever).
6. Server receives it, a thread picks it up, processes it, sends a response.
7. Connection closes (or stays open for keep-alive).


**HTTP (Hypertext Transfer Protocol)** is the foundational protocol used on the World Wide Web to enable communication between clients (like web browsers) and servers. It is an application-layer protocol that defines the rules and methods for exchange of data over the internet. It facilitates the exchange of information in a structured and standardized manner. HTTPS (Hypertext Transfer Protocol Secure) is a more secure version of HTTP.


### HTTP is not used, why?

Standard HTTP has a major security flaw: **all data transmitted using the HTTP protocol is unencrypted**. This means that sensitive information, such as passwords and credit card details, can be intercepted by hackers while it travels across the public internet. The sensitive information is transferred in plain text which is not secure. As a result, using standard HTTP to exchange sensitive data can put your personal information at risk.


HTTPs is HTTP with the addition of an encryption security feature that makes the data being transmitted unreadable to unauthorized parties.


**_What is Encryption?_** plaintext -> ciphertext

The process of transforming easily readable (plaintext) information into indecipherable (ciphertext) data, making it either impossible to retrieve the original data (one-way encryption) or requiring an inverse decryption process for recovery (two-way encryption).


### How HTTPs encrypts data

Secure HTTP achieves data protection through encryption algorithms that scramble the data during transmission. 

This encryption ensures that even if a hacker intercepts the data, they would only obtain meaningless, encrypted information that is virtually impossible to decipher without the **decryption key**.


HTTPS uses an encryption protocol to encrypt communications. 

The protocol is called **Transport Layer Security (TLS)**, although formerly it was known as **Secure Sockets Layer (SSL)**. 

This protocol secures communications by using what’s known as an [asymmetric public key infrastructure](https://www.cloudflare.com/learning/ssl/how-does-public-key-encryption-work/). This type of security system uses two different keys to encrypt communications between two parties:

1. The private key - this key is controlled by the owner of a website and it’s kept private. This key lives on a web server and is used to decrypt information encrypted by the public key.
2. The public key - this key is available to everyone who wants to interact with the server in a way that’s secure. Information that’s encrypted by the public key can only be decrypted by the private key.

![[Pasted image 20260615133149.png]]


HTTPS uses port 443. This differentiates HTTPS from HTTP, which uses port 80.


**TLS handshake

![[Pasted image 20260615134008.png]]


- The client sends a **ClientHello** which just contains the information of the client’s supported SSL/TLS versions, cryptographic algorithms, etc.
- The server responds with **ServerHello** which contains the information about what algorithm it choose from the list of algorithms that it received from **ClientHello,** the Server’s digital certificate along with the server’s public key, etc.
- ==The client verifies if the received digital certificate is valid by contacting the Certificate Authority that issued the digital certificate.==
- Once the authenticity of the webserver is verified from the previous step, **ClientKeyExchange** takes place. In which a shared secret key for the purposes of Symmetric key encryption is encrypted with the Server’s public key received in Step-2.
- The client sends a Finished message
- The server now sends a Finished message encrypted with the key sent by the Client in Step-4, implying that the communication is encrypted.

**in pt 3, What happens is that each client comes with the trust store with all of the trusted CA's root certificates.


Note: **Digital certificate** is a certificate issued by some **Certificate Authority** (CA) which contains 2 things primarily. One thing is the public key of the Server, and the other is Digital Signature that’s been encryted using private key of the **CA** who issued that certificate.


**Certificate Authority** in simple terms, is an entity that does due diligence of the company before issuing them a signed certificate which helps to prove the Authenticity of the company that uses the certificate. 

The way it works is all major web browsers come with public key’s of major Certificate Authorities throughout the globe. Browser can verify the **Authenticity** of the certificate by decrypting it using the corresponding public key it has. 

It’s works as follows, lets say I have my own company ABC with domain **abc.com** and I want to get a Certificate for encrypting communication between my server and the clients, using https. I go to a popular CA like **Google**, **VeriSign** and ask them to provide me a Digital Certificate(by creating a **Certificate Signing Request(CSR)**). 

The CA does their own checks and finally issue me a **digital certificate** which contains **ABC’s** public key and a digital signature signed using CA’s private key. So anyone who trusts that CA can trust the company **ABC.**



References:

https://medium.com/@mohithmarisetti_58912/https-made-easy-df1b88dc2a0e
https://www.cloudflare.com/learning/ssl/what-is-https/
https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/

