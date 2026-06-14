## Normalisation

![[Pasted image 20260614011115.png]]

![[Pasted image 20260614011220.png]]

PK is a determinant of all other columns

![[Pasted image 20260614011353.png]]
![[Pasted image 20260614011424.png]]

![[Pasted image 20260614011508.png]]

![[Pasted image 20260614011518.png]]

![[Pasted image 20260614011617.png]]


![[Pasted image 20260614011713.png]]

![[Pasted image 20260614011733.png]]

![[Pasted image 20260614012038.png]]


Why use normalisation? -> get rid of redundant data.

Why can't we have redundant data?

![[Pasted image 20260614012410.png]]

Here Branch code automatically determines Branch name and HOD but the cols are still included.

Insertion anomaly

![[Pasted image 20260614012537.png]]

If a new student joins but is not yet assigned to a branch, the entries would either have to be null or the stud has to be assigned a branch first

![[Pasted image 20260614012702.png]]

If we add a new dept IT, its student cols would have to be null too

Deletion anomaly

![[Pasted image 20260614012810.png]]

You delete the student -> the branch is gone too! (as only that student was enrolled in that branch

Updation anomaly

Suppose HOD is changed, then you need to update many rows

**Fix these -> normalisation**

![[Pasted image 20260614013147.png]]

(simple fix)

Decompose tables into multiple tables until SRP (single responsibility principle) is achieved.

## Types 

1NF

-> No multivalued attr.

![[Pasted image 20260614013509.png]]

Create duplicate rows to fix, adds redundancy

2NF

all non prime attr. must depend fully on the PK

![[Pasted image 20260614014031.png]]

AB -> C should be true, not B -> C 

![[Pasted image 20260614014216.png]]

As AB is a PK, both A and B cannot be null but one of them can be null. If B is null then C will be null too as B -> C

![[Pasted image 20260614014351.png]]

![[Pasted image 20260614014409.png]]

Ex- 
![[Pasted image 20260614014502.png]]


![[Pasted image 20260614014512.png]]
(break into tables)

ProjectID is the FK

3NF

![[Pasted image 20260614014751.png]]

Its 2NF as B and C depend on PK = A

![[Pasted image 20260614014849.png]]

B determines C.

The table has redundancy.

![[Pasted image 20260614015029.png]]

BCNF

 ![[Pasted image 20260614015916.png]]
 ![[Pasted image 20260614020126.png]]






