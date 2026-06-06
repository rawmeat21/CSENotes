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

![[Pasted image 20260605161056.png]]

![[Pasted image 20260605161124.png]]


How to handle nulls
![[Pasted image 20260605161236.png]]

tries the first one, skip if null
tries 2nd, skip if null,
if not null then it takes that value

![[Pasted image 20260605161341.png]]

Notice that some emails are empty

![[Pasted image 20260605161439.png]]

![[Pasted image 20260605161427.png]]


NULLIF

![[Pasted image 20260605161705.png]]

if 1st arg == 2nd arg -> null
else 1st arg

How to handle div by 0:
![[Pasted image 20260605161835.png]]


![[Pasted image 20260605162003.png]]

(provide alt values)

## Dates

![[Pasted image 20260605162045.png]]

![[Pasted image 20260605162108.png]]
convert to date by casting (::)

![[Pasted image 20260605162129.png]]

### Adding and subtracting dates

![[Pasted image 20260605162325.png]]
![[Pasted image 20260605162542.png]]

All these return Timestamp

![[Pasted image 20260605162621.png]]

Cast is to Date again

### Extract things

![[Pasted image 20260605162713.png]]



## Age()

![[Pasted image 20260605162839.png]]

![[Pasted image 20260605162849.png]]




## Primary keys

![[Pasted image 20260605163135.png]]

![[Pasted image 20260605163304.png]]

Entering a tuple with same primary key is not allowed 

You can drop the pkey constraint

![[Pasted image 20260605163437.png]]

![[Pasted image 20260605163505.png]]
Now it works (bad)

### How to add primary Key?

![[Pasted image 20260605163843.png]]

You see that (id) is passed (single element array). We may also require multiple cols

### How to add unique constraint?

Unique constraint means all entries are distinct

![[Pasted image 20260605164219.png]]

unique_email_address is the name of the constraint 

![[Pasted image 20260605164255.png]]

![[Pasted image 20260605164450.png]]
(Drop the constraint because why not)

#### Method2 to add a unique constraint
![[Pasted image 20260605164602.png]]
Now psql will decide the name

![[Pasted image 20260605164624.png]]

See?


## Check constraint

Maybe you're a fucking transphobe and you don't want to allow genders other than Male and Female. How would u do that?

![[Pasted image 20260605165040.png]]

Check() takes in a condition to check

![[Pasted image 20260605165143.png]]

No faggots allowed in our database. 


## How to delete tuples?

![[Pasted image 20260605165717.png]]
 You should basically want to delete using the PK

DON'T FUCKING DO THIS:
![[Pasted image 20260605165845.png]]
All your data gone


## Update Columns

![[Pasted image 20260607022110.png]]

What it do- makes the whole column have the same email. Don't do this dumbo.

![[Pasted image 20260607022159.png]]
![[Pasted image 20260607022209.png]]

![[Pasted image 20260607022312.png]]


## On conflict Do nothing

![[Pasted image 20260607022536.png]]

There should be a unique/exclusion constraint on the column you give as input

## Do something (upsert- update + insert)

![[Pasted image 20260607022935.png]]

email- refers to email in table
EXCLUDED.email- refers to email in INSERT statement

![[Pasted image 20260607023041.png]]

![[Pasted image 20260607023155.png]]
(Update multiple)


## Foreign keys

![[Pasted image 20260607023358.png]]

FK- column that refers the PK of another column

## Adding relationships

![[Pasted image 20260607024212.png]]

How to add FK ^^^

UNIQUE(car_id) adds unique constraint on car_id

## How to update FK columns?

![[Pasted image 20260607030420.png]]


Can I insert car_ids which don't exist? - Fuck No
![[Pasted image 20260607030534.png]]



## Inner Joins

![[Pasted image 20260607030707.png]]

we want to combine tables A and B

![[Pasted image 20260607030735.png]]

![[Pasted image 20260607030857.png]]

ON takes in a column which will be used for the join. 
Here we join using the FK, person.car_id = car.id

![[Pasted image 20260607031047.png]]

You can see id, make, model, price from Car has been added

![[Pasted image 20260607031159.png]]

Adriana doesn't have a car (broke bitch), so she's not added

![[Pasted image 20260607031311.png]]
(select only cols u want and join)

![[Pasted image 20260607031344.png]]
(same shit)

## Left joins

![[Pasted image 20260607031436.png]]

Full A but common from B

![[Pasted image 20260607031649.png]]

Adriana is here even tho she doesn't have a car. So all tuples from A are taken, but only tuples from B which are referred by some tuple in A are taken.

Get all people who don't have a Car:

![[Pasted image 20260607031924.png]]


## Deleting records with FKs

![[Pasted image 20260607032155.png]]

You just can't delete a referenced tuple.

Solution: Delete referencing (dependent) tuple first, then do the indepenedent one
OR make car_id of the entry NULL

![[Pasted image 20260607032449.png]]

You can also use cascade (prolly don't tho)

## Export to csv

![[Pasted image 20260607032717.png]]


## Serial and sequences

![[Pasted image 20260607032821.png]]

the id cols are managed by a sequence, while having types bigint

![[Pasted image 20260607033037.png]]

