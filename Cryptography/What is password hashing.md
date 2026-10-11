
A hash function takes an input of any length and produces a fixed-length output, the digest. The operation only runs one way: you can compute the digest from the password, but you can’t compute the password from the digest. So you never store the password at all. 

At login, you hash whatever the user typed and compare digests. Match means correct password, and the original never needs to exist in your database.

Encryption is a different tool for a different job. **It’s designed to be reversed by whoever holds the key**, which is exactly what you want for data you need to read back later, and exactly what you don’t want for passwords.


### What about salts and peppers?

A salt is a unique random value generated per password and stored in the clear next to the hash. It’s not a secret. Its job is to make every hash unique even when passwords aren’t, which kills precomputed lookup tables and the crack-once-unlock-thousands economics that gutted LinkedIn. 

Modern algorithms handle this for you: bcrypt and Argon2 generate and embed the salt automatically, so if you’re manually concatenating salts in 2026, something upstream went wrong.

A pepper is the opposite: a single shared secret applied to every password and stored outside the database, in a secrets manager or HSM (OWASP, 2026). If an attacker gets the database through SQL injection but not the application host, the pepper keeps the hashes uncrackable offline. OWASP positions it as an extra layer on top of proper hashing and salting, never a substitute for either.


