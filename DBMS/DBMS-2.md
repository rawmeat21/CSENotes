# ER Model

## Entity

![[Pasted image 20260531150318.png]]

![[Pasted image 20260531150332.png]]

![[Pasted image 20260531150400.png]]

![[Pasted image 20260531150625.png]]

The actual students are the enitites (objects)

![[Pasted image 20260531150722.png]]

![[Pasted image 20260531150745.png]]

Remember this!
![[Pasted image 20260531151811.png]]

Customer- enitity
Name, Addr- composite attr
CustID, firstname,lastname,etc- simple attr
Age- Derived attr
PhoneNum- multivalued
StudentID- single valued
## Relationships

![[Pasted image 20260531151154.png]]

![[Pasted image 20260531150943.png]]

![[Pasted image 20260531151053.png]]

Strong vs weak relationships

Strong entity- Independent existence of entity, can be recognized with a primary key. Ex-  Student in College

Strong relationship- b/w 2 strong entities. Ex- Student enrolls Course, Customer places Order. All entities exist independently, and can be addressed by a primary key.

Weak enitity- Dependent existence, it exists because of some other strong enitity. No primary key can identify this enitity. Ex- Payment may be a dependent enitity that depends on Loan enitity for a bank system

![[Pasted image 20260531152723.png]]

Weak relationship- b/w weak entity and its owner


Degree of relation- Number of enitites participating in a relationship

![[Pasted image 20260531153207.png]]

Binary- all that was shown

Ternary

![[Pasted image 20260531153247.png]]

### Relationship & participation constraints

![[Pasted image 20260531153404.png]]

![[Pasted image 20260531153540.png]]

![[Pasted image 20260531153634.png]]

![[Pasted image 20260531153715.png]]

![[Pasted image 20260531153759.png]]


Participation models:

![[Pasted image 20260531153948.png]]

-> It is not possible that a Loan enitity exists but its not related to some customer (each loan enitity has a preimage) (Total)

-> It may be possible that a customer exists BUT is not associated to any Loan (Partial)

**Weak enitity has total part. constraint, ex- Payment enitity in the above example is always related to a Loan!

![[Pasted image 20260531154340.png]]


![[Pasted image 20260531154414.png]]
