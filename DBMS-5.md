# Relational Model

We use tables!

Each row/ tuple represents 1 enitity

![[Pasted image 20260602183905.png]]

![[Pasted image 20260602184151.png]]

![[Pasted image 20260602184250.png]]

How to establish relationships? - Foreign key

## Keys

Primary Key- Uniquely identify a single enitity / tuple

Super Key- a group of attr. which can uniquely idenitfy an enitity

![[Pasted image 20260602184928.png]]

Candidate Key- Super Key of min cardinality

![[Pasted image 20260602185102.png]]

Name here is redundant, it doesn't uniquely identify any enitity

![[Pasted image 20260602185204.png]]

Only CustID is CK (min size)
**(CK can contain null values)


Primary Key- CK of min size

Alternate Key= CK - PK 

Foreign Key- A key which is a primary key of a different entity set

![[Pasted image 20260602185408.png]]

CustID from Customer added to Order

![[Pasted image 20260602185559.png]]

Now we see that an Order is related to a unique enitity in Customer set

This is how relations are made b/w tables

Customer- Referenced/ Parent relation
Order- Referencing/ Child relation


Composite Key- PK using >=2 attr

Compound Key- PK using 2 FK

Surrogate Key

![[Pasted image 20260602190036.png]]

If the school databases are merged, we cannot use reg_no as a primary Key as 2 different students can get the same key

We assign a generated number to each tuple. This is the surrogate key. This can be used as a PK

## Integrity constraints

![[Pasted image 20260602190502.png]]

What prevents us from putting someone with RollNo = Z ?
We assign constraints to attributes 

## Referential constraints

![[Pasted image 20260602190717.png]]

Insert constraint

![[Pasted image 20260602190821.png]]

We cannot add a tuple to child which doesn't have the given Foreign key value in parent

![[Pasted image 20260602191006.png]]

Delete constraint

You cannot delete a value in Parent table if it is present in Child table (its being referenced)

IMP:

ON delete cascade
![[Pasted image 20260602191201.png]]

ON delete NULL
![[Pasted image 20260602191413.png]]

Can FK be null? Yes if we use ON delete Null when removing a tuple from Parent. We make the FK in corresponding tuple NULL too

![[Pasted image 20260602191553.png]]

## Key constraints

1. Not Null (default is NULL)- Any CRUD operation violating this is invalid
2. Unique
![[Pasted image 20260602191843.png]]

PK is unique by default

3. Default- default value of constraint
![[Pasted image 20260602192030.png]]

Consider this Amazon system, by default Prime_status (does user has Amazon prime) is 0 

4. Check- limit value range
![[Pasted image 20260602192144.png]]

5. PK constraint
![[Pasted image 20260602192222.png]]

 There can only be 1 PK constraint per table

6. FK constraint
![[Pasted image 20260602192322.png]]

