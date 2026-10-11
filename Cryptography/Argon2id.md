https://veracrypt.io/en/Argon2id.html
https://sealtask.com/learn/argon2id-password-hardening/
https://medium.com/@manojsrinivasa98/how-i-encrypt-user-data-using-argon2id-aes-gcm-and-aws-kms-without-storing-keys-a020b4d4e776
https://skycloak.io/blog/why-password-hashing-is-vital-for-security/

Argon2id is a memory-hard key derivation function designed to resist both time-memory trade-off attacks and side-channel attacks.

It is a password hashing function. Password hashing converts a password into a fixed-length, one-way fingerprint that cannot be turned back into the original, so a stolen database doesn’t hand an attacker every user’s credentials.

### Key Features

- **Memory-hard**: Requires a configurable amount of memory, making it resistant to specialized hardware attacks
- **Side-channel resistant**: Combines data-dependent and data-independent memory access patterns
- **Internal hash function**: Uses BLAKE2b-512 internally, eliminating the need for separate hash algorithm selection
- **Tunable parameters**: Allows adjustment of memory cost, time cost, and parallelism
### Argon2 Variants

Argon2 has three variants:

- **Argon2d**: Uses data-dependent memory access, resistant to time-memory trade-off attacks but vulnerable to side-channel attacks
- **Argon2i**: Uses data-independent memory access, resistant to side-channel attacks but more vulnerable to time-memory trade-off attacks
- **Argon2id**: Hybrid approach that combines both variants, providing resistance to both attack types


Argon2id is the hybrid Argon2 variant defined in RFC 9106 for password hashing and password-based key derivation. It is deliberately memory-hard and configurable so each password guess consumes meaningful memory and computation.

**Is Argon2id encryption?** - No. It is a password hashing and password-based key derivation function.

**Does a salt need to be secret?** - No. A salt should be unique and stored with the password record. Its main purpose is to prevent precomputation and identical outputs for identical passwords.

![](../assets/Pasted%20image%2020261011103842.png)


