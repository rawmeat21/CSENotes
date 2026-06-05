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

![[Pasted image 20260605153601.png]]
![[Pasted image 20260605153642.png]]

% - any number of chars
_ - single char
![[Pasted image 20260605153728.png]]

![[Pasted image 20260605153739.png]]

![[Pasted image 20260605153921.png]]

ILIKE ignores case

Group BY

![[Pasted image 20260605154038.png]]
![[Pasted image 20260605154054.png]]

(doesnt work lol)

![[Pasted image 20260605154151.png]]
![[Pasted image 20260605154159.png]]

![[Pasted image 20260605154220.png]]
![[Pasted image 20260605154231.png]]


![[Pasted image 20260605154500.png]]
Having should come after groupby

![[Pasted image 20260605154516.png]]

![[Pasted image 20260605154552.png]]
You can use <=, >=, < 

Aggregrates

![[Pasted image 20260605154950.png]]

![[Pasted image 20260605155023.png]]


![[Pasted image 20260605155702.png]]
![[Pasted image 20260605155716.png]]

Adding `GROUP BY make, model` completely changes how `MIN(price)` behaves. Instead of finding the minimum price for the **whole table**, SQL finds the minimum price for **each unique combination of make and model**.

SQL syntax forces you to write `SELECT` first, but the database actually executes `GROUP BY` _before_ it calculates `MIN(price)`.

### Step 1: `FROM car`

The database grabs the raw car dataset.

### Step 2: `GROUP BY make, model`

Instead of one big list, SQL sorts the cars into "buckets" based on **both** the manufacturer and the car model.

|Make|Model|Price|
|---|---|---|
|**Toyota**|**Corolla**|$22,000|
|**Toyota**|**Corolla**|$19,500|
|_Toyota_|_Camry_|$26,000|
|_Toyota_|_Camry_|$28,500|
|**Ford**|**Mustang**|$35,000|
|**Ford**|**Mustang**|$42,000|

Behind the scenes, SQL groups them into 3 distinct buckets:

1. **[Toyota, Corolla]** bucket (contains 2 cars)
    
2. **[Toyota, Camry]** bucket (contains 2 cars)
    
3. **[Ford, Mustang]** bucket (contains 2 cars)
    

### Step 3: `SELECT make, model, MIN(price)`

Now, SQL looks inside **each individual bucket** one by one. It looks at the prices inside that specific bucket, picks the lowest one (`MIN`), and outputs exactly **one row per bucket**.

- From the **[Toyota, Corolla]** bucket, the lowest price is **$19,500**.
    
- From the **[Toyota, Camry]** bucket, the lowest price is **$26,000**.
    
- From the **[Ford, Mustang]** bucket, the lowest price is **$35,000**.


![[Pasted image 20260605160134.png]]
![[Pasted image 20260605160157.png]]

## Arithmetic operations

![[Pasted image 20260605160334.png]]

![[Pasted image 20260605160556.png]]
(get 10% discounted price column)
![[Pasted image 20260605160627.png]]

![[Pasted image 20260605160723.png]]
![[Pasted image 20260605160739.png]]

Notice that the col names are 'round'

![[Pasted image 20260605160931.png]]
