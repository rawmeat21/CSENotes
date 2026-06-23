Sources: https://medium.com/@shivambhadani_/acid-properties-explained-a-complete-guide-with-postgresql-implementation-071afa6aaf8a
https://medium.com/@ykods/understanding-acid-properties-in-database-management-98243bfe244c

## What is a transaction?

A transaction is a group of SQL statements (such as `SELECT`, `UPDATE`, `INSERT`, etc.) that are treated as a single unit of work within a database.

**E-commerce application**

**Steps:**

1. Check product availability
2. Deduct money from the customer’s account
3. Reduce stock quantity
4. Create order record

If payment fails, the order is not created.

All steps together form **one transaction**.

SQL statements:

```sql

BEGIN;  
  
-- 1️. Check product availability  
SELECT stock   
FROM products   
WHERE product_id = 101   
FOR UPDATE;  
  
-- 2️. Deduct money from customer account  
UPDATE customers  
SET balance = balance - 500  
WHERE customer_id = 1  
AND balance >= 500;  
  
-- 3️. Reduce stock quantity  
UPDATE products  
SET stock = stock - 1  
WHERE product_id = 101  
AND stock > 0;  
  
-- 4️. Create order record  
INSERT INTO orders (customer_id, product_id, amount)  
VALUES (1, 101, 500);  
  
COMMIT;
```

To start a transaction, you need to specify `BEGIN;` and at the end write `COMMIT;` And all the SQL statements written between BEGIN and COMMIT will be a transaction.

**Ticket Booking**

When booking a train or flight ticket:

Steps:

1. Check seat availability
2. Block seat
3. Make payment
4. Confirm booking


```sql
BEGIN;  
  
-- 1. Check seat availability and lock it  
SELECT id   
FROM seats  
WHERE trip_id = 1  
  AND seat_number = 'A1'  
  AND status = 'available'  
FOR UPDATE;  
  
-- 2. Block the seat  
UPDATE seats  
SET status = 'blocked'  
WHERE trip_id = 1  
  AND seat_number = 'A1';  
  
-- 3. Create booking  
INSERT INTO bookings (user_id, trip_id, seat_id, status)  
VALUES (1, 1, 10, 'blocked');  
  
-- 4. Make payment  
INSERT INTO payments (booking_id, amount, status)  
VALUES (1, 500.00, 'success');  
  
-- 5. Confirm booking  
UPDATE bookings  
SET status = 'confirmed'  
WHERE id = 1;  
  
UPDATE seats  
SET status = 'booked'  
WHERE id = 10;  
  
COMMIT;
```


### Atomicity

Transactions are treated as atomic units, meaning they are indivisible and irreducible. A transaction is either completed in its entirety or not at all. If a transaction encounters an error midway, all changes made within that transaction are rolled back, ensuring that the database remains in a consistent state.

```sql
BEGIN;  
  
UPDATE accounts SET balance = balance - 300 WHERE name='Shivam';  
  
-- simulate error  
SELECT 1/0;  
  
UPDATE accounts SET balance = balance + 300 WHERE name='Aman';  
  
COMMIT;
```

`SELECT 1/0;` gives error: `ERROR: division by zero` and the program crashes, so it will also rollback the first successfully executed SQL and the DB behave that this transaction never happened.

![[Pasted image 20260623125407.png]]


### Isolation


The concept of **isolation** comes into play when two or more transactions run at the same time. 

In a real-world scenario, this could happen when multiple users send requests to a server simultaneously, and the application server starts multiple transactions in the database at the same time.

#### Anomalies during multiple transactions

##### **Dirty Read**

A transaction (T1) reads the data that is being modified by an uncommitted concurrent transaction (T2). 

If the concurrent transaction (T2) eventually commits, the data read is valid. 

**However**, if it rolls back, the reading transaction (T1) would have accessed data that never actually existed, and this is called a dirty read.

![[Pasted image 20260623130459.png]]

##### **Nonrepeatable Read  

It happens when a transaction reads the same piece of data twice, but gets different values each time because another transaction changed it in between.

For example:

(i) Transaction A reads a value from the database.  
(ii) Transaction B updates that value and commits.  
(iii) Transaction A reads the same value again and sees the new value.

This can be confusing because it looks like the data “changed by itself” while Transaction A was running.

![[Pasted image 20260623130708.png]]



##### **Phantom Read  

It happens when a transaction runs a query twice and sees different rows each time, even though it didn’t change anything itself. This usually happens because another transaction added or deleted rows in the meantime.

For example:

(i) Transaction A runs a query to get all orders over $100.  
(ii) Transaction B inserts a new order over $100 and commits.  
(iii) Transaction A runs the same query again and now sees the new order like a “phantom” appeared.

Phantom reads are different from nonrepeatable reads because the **number of rows changes**, not just the values in existing rows.

![[Pasted image 20260623130825.png]]



#### Isolation Levels


- READ UNCOMMITTED
- READ COMMITTED
- REPEATABLE READ
- SERIALIZABLE (strongest)

(1st is not present in PostgreSQL as of now.)

Stricter isolation levels provide greater data consistency, but they can negatively impact performance.

![[Pasted image 20260623131014.png]]

(Which isolation protects which anomaly, this is for PSQL)

Default is Read committed, which means Dirty Reads are NOT possible. This is used for most cases.

Next is Repeatable Read, that prevents **dirty reads**, **nonrepeatable reads**, and **phantom reads**.

```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;  
  
SELECT balance FROM accounts WHERE id = 1;  
UPDATE accounts SET balance = balance - 100 WHERE id=1;  
  
COMMIT;
```

**Terminal 1:**

```sql
BEGIN ISOLATION LEVEL REPEATABLE READ;  
SELECT balance FROM accounts WHERE name='Shivam';
```

Output = 5500

**Terminal 2:**

```sql
UPDATE accounts SET balance = balance + 500 WHERE name='Shivam';  
COMMIT;
```

**Terminal 1:**

```sql
SELECT balance FROM accounts WHERE name='Shivam';
```

Output = 5500  

In reality, in the DB, currently, the `balance` of `Shivam` is changed to 6000

#### Why the fuck would you want to do this? Why read the old value?

**Example 1**

Suppose you are running a ticket booking system. A transaction needs to make a decision based on a user's current status:

1. **Step 1:** Look up if Shivam is a 'VIP' member. (Output: `Yes`)
    
2. **Step 2:** Apply a 50% VIP discount to a $1,000 ticket.
    
3. **Step 3:** Charge the card $500.
    

If you don't use `REPEATABLE READ`, another admin could change Shivam's status to 'Standard' between Step 1 and Step 2. Your code would proceed to apply a VIP discount to a non-VIP user. 

`REPEATABLE READ` guarantees that if a condition is true at the start of your transaction, it stays true until you are done processing.

**Example 2**

Imagine you are running a midnight balance sheet report across millions of accounts. The report takes 10 minutes to calculate the total sum of all money in the bank.

If you use the default `READ COMMITTED` level, look at what happens:

- **12:00 AM:** Your report reads **Account A**, which has **$100**.
    
- **12:01 AM:** A customer transfers **$50** from Account B to Account A. The bank updates both accounts and commits. (Now Account A has $150, Account B has $50).
    
- **12:02 AM:** Your report reaches **Account B** and reads its new balance: **$50**.
    

When your report finishes, it adds Account A ($100) and Account B ($50) to get **$150**. But the actual total pool of money was $200! **$50 vanished into thin air** on your report because the data shifted mid-read.

With `REPEATABLE READ`, your report sees the bank exactly as it looked at 12:00 AM. It will read Account B as $100, and your final total ($200) will balance perfectly.


##### Serializable

It is the strongest Isolation Level. It guarantees full serializability. **Serializable** means **the database behaves as if transactions happened one at a time**, preserving correctness even in concurrent execution.

```sql
BEGIN ISOLATION LEVEL SERIALIZABLE;  
  
-- Your operations  
UPDATE accounts SET balance = balance - 100 WHERE id = 1;  
UPDATE accounts SET balance = balance + 100 WHERE id = 2;  
  
COMMIT;
```

If you set the Isolation Level to `Serializable`:

- PostgreSQL might **allow both transactions to start**.
- At **commit**, it checks for **conflicts**.
- If a conflict is detected that would break serializability, **one transaction is aborted with an error**:


```
ERROR:  could not serialize access due to concurrent update
```

#### Choosing the Right Isolation Level

The choice of isolation level depends on your specific requirements:

**Use Read Uncommitted** when you need maximum performance and can tolerate reading potentially invalid data (like for reporting queries where approximate results are acceptable).

**Use Read Committed** for most general-purpose applications where you need a balance of consistency and performance.

**Use Repeatable Read** when you need to ensure that data doesn’t change during your transaction (like financial calculations).

**Use Serializable** when data integrity is absolutely critical and you can accept lower performance (like in banking or medical systems).


![[Pasted image 20260623134737.png]]


### Durability

Once a transaction is successfully committed, its effects are permanent. 

Even in the face of system crashes or power failures, the changes made by committed transactions are retained, and the database is able to recover to a consistent state upon restarting.


**Once COMMIT happens, data will never be lost.**


Postgres uses:

- WAL (Write Ahead Log)
- fsync
- disk flush

Flow:

1. Transaction commits
2. Changes written to WAL
3. WAL flushed to disk
4. Then commit returns

Even if the server crashes, WAL replays.


### Consistency

Consistency ensures that **every transaction takes the database from one valid state to another valid state**, following all rules, constraints, and schemas defined in the database.


**Constraints include:**

- Primary key and unique constraints
- Foreign key constraints
- Check constraints
- Data types
- Triggers and rules


Example 1 (constraints):


```sql
CREATE TABLE accounts (  
    id SERIAL PRIMARY KEY,  
    balance INT CHECK (balance >= 0)  
);  
  
INSERT INTO accounts (balance) VALUES (100); -- (executed successfully)  
INSERT INTO accounts (balance) VALUES (-50); -- (gives error) violates CHECK constraint
```

Example 2 (triggers):

Triggers allow **custom logic to enforce complex business rules** that constraints alone cannot capture.

```sql
CREATE FUNCTION ensure_balance() RETURNS trigger AS $$  
BEGIN  
    IF NEW.balance < 0 THEN  
        RAISE EXCEPTION 'Balance cannot be negative';  
    END IF;  
    RETURN NEW;  
END;  
$$ LANGUAGE plpgsql;  
  
CREATE TRIGGER check_balance  
BEFORE INSERT OR UPDATE ON accounts  
FOR EACH ROW EXECUTE FUNCTION ensure_balance();
```


```sql
INSERT INTO accounts (name, balance) VALUES ('Mohit', 100);  
-- Works fine  
  
INSERT INTO accounts (name, balance) VALUES ('Messi', -50);  
-- ERROR: Balance cannot be negative
```

```sql
UPDATE accounts SET balance = -20 WHERE id = 1;  
-- ERROR: Balance cannot be negative
```

Example 3 (Referential integrity):

```sql
-- Orders table references customers table  
CREATE TABLE customers (  
  customer_id INT PRIMARY KEY,  
  name VARCHAR(100),  
  email VARCHAR(100)  
);  
  
CREATE TABLE orders (  
  order_id INT PRIMARY KEY,  
  customer_id INT,  
  order_total DECIMAL(10,2),  
    
  -- Foreign key constraint ensures referential integrity  
  FOREIGN KEY (customer_id) REFERENCES customers(customer_id)  
);
```

With this setup, you can’t create an order for a customer that doesn’t exist, and you can’t delete a customer who has existing orders (unless you handle it properly). This maintains consistency across related data.

