![[Pasted image 20260531104349.png]]

Data has no meaning by itself. It has to be processed first.

Data- Collection of raw unorganised bytes

Information- Processed, organised and structured data

Database- System where data is stored in a way it can be easily accessed, managed and updated

DBMS (Database management system)- Collection of interrelated data (Database) and a set of programs to manage that data

![[Pasted image 20260531105715.png]]


## Why not just use File systems? (or Why use DBMS?)

1. Data redundancy- Multiple files may store the same information of users
2. Data inconsistency- Change to some data in one file creates inconsistent copies in other files
3. Difficulty in accessing data
4. Data isolation
5. Integrity problems- Suppose we have a constraint that savings ac balance cannot go less than 10k. With file systems, we have to change the source code to implement this functionality. Ok, but if these changes are needed frequently or if we want to add more features, then we have to do more work
6. Atomicity problems- Some transactions should be atomic. This is easy to implement with DBMS but not file systems, we need to add extra checks
7. Concurrent access anomalies- We need functionality to handle concurrent transactions. These functionalites need to be manually added in File systems
8. Security problems- We may want to restrict access to specific data. This functionality comes with DBMS, but has to be added with FS.

TLDR: DBMS comes with extra functionality to manage data. FS needs such things to be manually implemented




