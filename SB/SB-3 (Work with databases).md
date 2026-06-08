How to let Sb connect to db

![[Pasted image 20260608172207.png]]

replace localhost by remote db if there is a remote db you're working with

![[Pasted image 20260608172335.png]]
Create a separate package for handling endpoint services. 

![[Pasted image 20260608172740.png]]
And a repository package which will talk to the actual database

controller -> service -> repository -> database

![[Pasted image 20260608172902.png]]

To make CRUD operations through the repo class, extend MongoRepository

![[Pasted image 20260608172952.png]]
(A view of MongoRepository class, helps doing CRUD operations)

![[Pasted image 20260608173041.png]]

T = object that maps to database
ID = datatype of id

![[Pasted image 20260608173206.png]]

Update JournalEntry class by adding: 

@Document: Tells SB that is class is mapped to a mongodb collection

![[Pasted image 20260608173716.png]]

If you don't provide collection, SB will look for a collection named JournalEntry

@Id- for the id (unique key)



![[Pasted image 20260608173545.png]]

Note the types, for ID we use String as your id has type string

![[Pasted image 20260608173434.png]]


Create a bean repo class

(save() is provided by MongoRepository, if id is not present, it creates one, if it is, it updates)

![[Pasted image 20260608173927.png]]

Adding a service class

![[Pasted image 20260608174000.png]]
(updated post function)

![[Pasted image 20260608174305.png]]
![[Pasted image 20260608174334.png]]
(ID not provided => its created)

_class -> fully qualified name of the class

![[Pasted image 20260608174511.png]]

If you provide an ID, it gets used

![[Pasted image 20260608174559.png]]
(we sent a post request but changed the message, a new entry was NOT created. Instead the record was updated)

![[Pasted image 20260608174647.png]]

![[Pasted image 20260608174720.png]]
 (adding a getAll() service)


![[Pasted image 20260608174935.png]]

![[Pasted image 20260608175114.png]]

MongoRepository also comes with a findById() method

![[Pasted image 20260608175131.png]]

![[Pasted image 20260608175409.png]]
