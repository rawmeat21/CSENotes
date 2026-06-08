## Specialization

![[Pasted image 20260602010506.png]]

You have a Person enitity which may also act as a Employee or a Customer

Putting all attributes into a single enitity is messy. Then lets create separate Employee and Customer classes give them their attributes. This works, but common attr. get copied to both, which is redundant.

![[Pasted image 20260602010736.png]]

Customer and Employee enitities are child/subentities of Person. They inherit all properties of their parent enitity. Specialization is a 'is-a' relationship. 

![[Pasted image 20260602010953.png]]

![[Pasted image 20260602011411.png]]

![[Pasted image 20260602011421.png]]

## Generalization (Reverse of Specialization)

![[Pasted image 20260602011544.png]]

Bottom Up:

We make Car, SUV and Bus. 
We see that they have common attr. 
So we put them in a new parent enitity (Vehicle)

## Aggregation (Relationships among relationships)
	

![[Pasted image 20260602012239.png]]

We need a Manager enitity to manage the employee, branch and job as a whole. This diagram is redundant and doesn't illustrate the true nature of the relationship

![[Pasted image 20260602012526.png]]

Modified ER diagram, we put Job, Branch and Employee as a single entity

![[Pasted image 20260602012839.png]]

A student may not have a subject (may not be enrolled)
A semester can have a subject, but its studied by students afterall. 