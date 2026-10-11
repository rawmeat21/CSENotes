
A hash function takes an input of any length and produces a fixed-length output, the digest. The operation only runs one way: you can compute the digest from the password, but you can’t compute the password from the digest. So you never store the password at all. 

At login, you hash whatever the user typed and compare digests. Match means correct password, and the original never needs to exist in your database.

Encryption is a different tool for a different job. **It’s designed to be reversed by whoever holds the key**, which is exactly what you want for data you need to read back later, and exactly what you don’t want for passwords.



