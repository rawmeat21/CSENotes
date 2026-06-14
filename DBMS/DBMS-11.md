![[Pasted image 20260614145705.png]]

NoSQL models are required when we want data *fast*, ex- modern apps.

![[Pasted image 20260614150612.png]]

We store it like this. 

Disadvantage- When we want to get some data (last_name say), we have to get the **whole entry**


#### Scaling

Say our app has a lot of users and we're now running out of storage.

Then we upgrade our storage method, this is scaling.

There are 2 types of scaling:

Horizontal (scale out)- add multiple nodes.

![[Pasted image 20260614151158.png]]

say each is 1TB. When 1st runs out you store in 2nd, when 2nd runs out u store in 3rd, ...

Vertical (scale up)- Just upgrade the hardware (buy more RAM, CPU). SQL only supports vertical scaling practically.

#### Why no horizontal scaling in SQL?

We can do horizontal scaling, but its not efficient.

![[Pasted image 20260614151639.png]]

SQL is a collection of tables.

Say you have 3 nodes (servers) and they store tables T1, T2, T3.

Now to get meaningful data out of a relational DB, we need to do a join.

This will require the tables in one place, say another node S4.

Fetching a table from servers is very slow! Not only that, after you do a join on 2 huge tables the fetch will be slow too. 


Types of NoSQL

![[Pasted image 20260614201225.png]]

Why column wise store? - fast aggregration (notice the entries for age are all consecutive)

![[Pasted image 20260614201622.png]]


