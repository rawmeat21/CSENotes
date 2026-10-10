https://www.okta.com/en-in/identity-101/nonce/
https://www.geeksforgeeks.org/computer-networks/what-is-nonce-in-cryptography/

In cryptograIn cryptography, a nonce is a **random number that is used only once during a secure communication**.

It is an arbitrary and randomly generated number that can only be used once and often also includes a timestamp.

A nonce often includes a timestamp, which means it is only valid during a specific amount of time, to help ensure that it is only used once. If it does not have a time-variant, the nonce will need to be generated with enough random bits to make sure that the probability of it repeating a value that has been generated previously is nearly insignificant.

A nonce can be used in an authentication protocol as a method of preventing replay attacks by ensuring that old communications are not being reused. 

The nonce helps to prove that the message received was sent by the intended sender and was not intercepted and resent by a bad actor. To optimize a nonce, it should have both sequential parts and random parts.


**ChaCha20-Poly1305** is an algorithm that implements AEAD.

