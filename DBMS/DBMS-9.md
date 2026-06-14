![[Pasted image 20260614090311.png]]

To user, the money transfer is a single operation
But to computer, it involves 6 steps. 

These steps should be atomic (considered single task) and should either all happen or none should happen


![[Pasted image 20260614090811.png]]

## ACID properties

1. Atomicity
![[Pasted image 20260614091117.png]]

2. Consistency
see notes

3. Isolation

![[Pasted image 20260614091408.png]]

A sends 50 to B through 2 different sources: Gpay and netbanking

Both read 1000 from A
Both subtract 50 from 1000 and store 950 in A's bank ac
B gets +100 (ok)
But A had only -50!

The fix: do one transaction after another finishes

4. Durability

![[Pasted image 20260614091901.png]]

When a transaction is done, the changes made to DB must be permanent, even if system fails.

This is done by the Recovery component of DB. We can keep logs of the transactions


### How to implement atomicity?

Shadow copy scheme

![[Pasted image 20260614092828.png]]

Log based recovery

![[Pasted image 20260614094121.png]]

 ![[Pasted image 20260614094648.png]]


CheckPoints : - The checkpoint is a type of mechanism where all the previous logs are removed from the system and permanently stored in the storage disk. - The checkpoint is like a bookmark. While the execution of the transaction, such checkpoints are marked, and the transaction is executed then using the steps of the transaction, the log files will be created. - When it reaches to the checkpoint, then the transaction will be updated into the database, and till that point, the entire log file will be removed from the file. Then the log file is updated with the new step of transaction till next checkpoint and so on. - The checkpoint is used to declare a point before which the DBMS was in the consistent state, and all transactions were committed.