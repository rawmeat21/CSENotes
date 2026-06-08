![[Pasted image 20260608182745.png]]

@Indexed- set some property, here uniqueness of username

![[Pasted image 20260608183046.png]]

You need to add the 4th line for this to work
@NonNull- yk
@DbRef- journalEntries will store references of JournalEntry objects **in the database**

![[Pasted image 20260608182825.png]]

This works like a foreign key

![[Pasted image 20260608183147.png]]

Separate classes added for handling users

![[Pasted image 20260608183428.png]]

![[Pasted image 20260608183740.png]]
(updated function in JournalEntry controller)

![[Pasted image 20260608184705.png]]
