# Connect to database

\c dbname

psql -h hostname(localhost) -p 5432 -U username 

![[Pasted image 20260604213004.png]]

![[Pasted image 20260604213018.png]]

![[Pasted image 20260604213558.png]]

![[Pasted image 20260604213632.png]]

BIGSERIAL increments itself

![[Pasted image 20260604213933.png]]

![[Pasted image 20260604214001.png]]

whats person_id_seq? - Its due to bigserial, its not a table, its a sequence

## How to insert

![[Pasted image 20260604214131.png]]

![[Pasted image 20260604214222.png]]

![[Pasted image 20260604215021.png]]

![[Pasted image 20260604224405.png]]
run from file

![[Pasted image 20260604224456.png]]
Select tuples

![[Pasted image 20260604224544.png]]

![[Pasted image 20260604224738.png]]

 ![[Pasted image 20260604224756.png]]
 (id decreasing)

![[Pasted image 20260604225001.png]]

![[Pasted image 20260604225013.png]]


![[Pasted image 20260604230250.png]]

![[Pasted image 20260604230310.png]]

![[Pasted image 20260604230347.png]]

![[Pasted image 20260604230356.png]]

![[Pasted image 20260604230415.png]]

![[Pasted image 20260604230538.png]]

t means true

![[Pasted image 20260604230651.png]]

<> means !=

![[Pasted image 20260604230824.png]]
limit 10 means first 10 rows

![[Pasted image 20260604230922.png]]
Offset 5 starts taking from 6

![[Pasted image 20260604231059.png]]

Fetch works same as limit, infact u should use fetch

![[Pasted image 20260604231208.png]]
This is too much work

![[Pasted image 20260604231300.png]]
Use IN

![[Pasted image 20260604231527.png]]
Between
