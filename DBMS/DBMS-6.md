
# ER -> Relational Model

![[Pasted image 20260602193227.png]]

Strong enitity gets own table
Weak enitity gets a table and its owner's PK. Note that Loan_num is also a FK

By itself payment_num is not a PK, but Loan_num+payment_num is a valid PK


How to handle composite attr. ? 

![[Pasted image 20260602193522.png]]

Its broken into simple attr. 

How to handle multivalued attr?

Create a new relation/ table for the mutlivalued attribues. Use a foreign key to identify them.

Ex- Employee <-> Dependent Name

![[Pasted image 20260602193900.png]]

![[Pasted image 20260602193959.png]]

(Why just emp_id is not enough!)


How to handle generalisation?

Make all tables
![[Pasted image 20260602194154.png]]

 ![[Pasted image 20260602194231.png]]
(Or don't use the Base class at all)

Method-2 may not be desired as it may lead to redundancy; Like say a Bank provides both current and savings account features, So one bank account has 1 entry in CurrentAc and 1 in SavingsAc. But balance is repeated in both, which is data duplication. 


How to handle aggregation:

![[Pasted image 20260602194758.png]]

![[Pasted image 20260602194839.png]]


Unary relation:

![[Pasted image 20260602194924.png]]

Create a new Foreign key which is taken from the SAME table's PK


one to one (1 : 1)
![[Pasted image 20260602195149.png]]

Many to many (M : N)

Create a separate table for the relation
![[Pasted image 20260602195356.png]]

id and prereq_course_id are both PKs in Course, so both are foreign keys in prereq


## Facebook RM


![[Pasted image 20260602195841.png]]

![[Pasted image 20260602195614.png]]

