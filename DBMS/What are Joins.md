Sources: https://medium.com/@johnnyJK/understanding-sql-joins-a-comprehensive-guide-88bab3457270


In SQL, joins are operations used to combine rows from two or more tables based on a related column between them. Joins are essential in relational databases because they allow you to retrieve data that is spread across multiple tables and present it in a unified way.

## Types:

### INNER JOIN

An INNER JOIN returns only the rows that have matching values in both tables. If there is no match, the row is not included in the result set.

Combines the data from two or more tables based on a related column, and it filters out rows that do not satisfy the join condition.

![[Pasted image 20260624134428.png]]


Do an inner join:

```sql
SELECT columns  
FROM table1  
INNER JOIN table2  
ON table1.common_column = table2.common_column;
```

employees table:

| id  | name    | department_id |
| --- | ------- | ------------- |
| 1   | Alice   | 1             |
| 2   | Bob     | 2             |
| 3   | Charlie | 3             |

departments table:

| id  | department_name |
| --- | --------------- |
| 1   | HR              |
| 2   | Finance         |
| 3   | IT              |
| 4   | Marketing       |

```sql
SELECT employees.name, departments.department_name  
FROM employees  
INNER JOIN departments  
ON employees.department_id = departments.id;
```

Result:

| name    | department_name |
| ------- | --------------- |
| Alice   | HR              |
| Bob     | Finance         |
| Charlie | IT              |


### LEFT JOIN

A LEFT JOIN, also referred to as a **LEFT OUTER JOIN**, is a type of SQL join where all rows from the left table (table1) are returned regardless of whether there is a match in the right table (table2). It **includes the matching rows from the right table** based on a related column specified in the ON clause of the LEFT JOIN statement.

This join condition ensures that for each row from the left table, the LEFT JOIN includes the corresponding matching rows from the right table in the result set. If there is no match found in the right table, NULL values are returned for the columns from the right table.

![[Pasted image 20260624134906.png]]


Do a left join:

```sql
SELECT columns  
FROM table1  
LEFT JOIN table2  
ON table1.common_column = table2.common_column;
```


customers table:

| id  | name    | email               |
| --- | ------- | ------------------- |
| 1   | Alice   | alice@example.com   |
| 2   | Bob     | bob@example.com     |
| 3   | Charlie | charlie@example.com |
| 4   | David   | david@example.com   |

orders table:

| id  | customer_id | product    | quantity |
| --- | ----------- | ---------- | -------- |
| 1   | 1           | Laptop     | 1        |
| 2   | 2           | Smartphone | 2        |
| 3   | 1           | Headphones | 1        |
| 4   | 3           | Tablet     | 1        |

Query

```sql
SELECT customers.name, orders.id AS order_id, orders.product, orders.quantity  
FROM customers  
LEFT JOIN orders ON customers.id = orders.customer_id;
```

Result:

| name    | order_id | product    | quantity |
| ------- | -------- | ---------- | -------- |
| Alice   | 1        | Laptop     | 1        |
| Alice   | 3        | Headphones | 1        |
| Bob     | 2        | Smartphone | 2        |
| Charlie | 4        | Tablet     | 1        |
| David   | NULL     | NULL       | NULL     |


### RIGHT JOIN

A RIGHT JOIN, also known as **RIGHT OUTER JOIN**, is a type of SQL join that returns all rows from the right table (table2) and includes matching rows from the left table (table1). The join condition is based on a related column between the two tables, specified in the ON clause of the RIGHT JOIN statement.

The RIGHT JOIN ensures that all rows from the right table are retained in the result set, and for each row from the right table, it includes the corresponding matching rows from the left table based on the join condition. If there is no match found in the left table, NULL values are returned for the columns from the left table.

![[Pasted image 20260624135219.png]]

Do a right join
```sql
SELECT columns  
FROM table1  
RIGHT JOIN table2  
ON table1.common_column = table2.common_column;
```

customers table:

| id  | name    | email               |
| --- | ------- | ------------------- |
| 1   | Alice   | alice@example.com   |
| 2   | Bob     | bob@example.com     |
| 3   | Charlie | charlie@example.com |
| 4   | David   | david@example.com   |

orders table:

| id  | customer_id | product    | quantity |
| --- | ----------- | ---------- | -------- |
| 1   | 1           | Laptop     | 1        |
| 2   | 2           | Smartphone | 2        |
| 3   | 1           | Headphones | 1        |
| 4   | 3           | Tablet     | 1        |
| 5   | NULL        | Keyboard   | 1        |

Query

```sql
SELECT customers.name, orders.id AS order_id, orders.product, orders.quantity  
FROM customers  
RIGHT JOIN orders ON orders.customer_id = customers.id;
```

Result:

| name    | order_id | product    | quantity |
| ------- | -------- | ---------- | -------- |
| Alice   | 1        | Laptop     | 1        |
| Bob     | 2        | Smartphone | 2        |
| Alice   | 3        | Headphones | 1        |
| Charlie | 4        | Tablet     | 1        |
| NULL    | 5        | Keyboard   | 1        |


### FULL JOIN

A FULL JOIN, also known as a **FULL OUTER JOIN**, is a type of SQL join that returns all rows from both the left table (table1) and the right table (table2). When there is a match between the tables based on the join condition, the result set includes the matched rows from both tables. When there is no match, the result set includes NULL values for columns from the table without a match.

![[Pasted image 20260624135445.png]]


Do full join:
```sql
SELECT columns  
FROM table1  
FULL JOIN table2  
ON table1.common_column = table2.common_column;
```

customers table:

| id  | name    | email               |
| --- | ------- | ------------------- |
| 1   | Alice   | alice@example.com   |
| 2   | Bob     | bob@example.com     |
| 3   | Charlie | charlie@example.com |
| 4   | David   | david@example.com   |

orders table:

| id  | customer_id | order_date | total_amount |
| --- | ----------- | ---------- | ------------ |
| 1   | 3           | 2024-05-01 | 150          |
| 2   | 2           | 2024-05-02 | 200          |
| 3   | 1           | 2024-05-03 | 100          |
| 4   | 5           | 2024-05-04 | 80           |

Query:

```sql
SELECT customers.name, orders.id AS order_id, orders.order_date, orders.total_amount  
FROM customers  
FULL JOIN orders ON customers.id = orders.customer_id;
```

Result:

| name    | order_id | order_date | total_amount |
| ------- | -------- | ---------- | ------------ |
| Alice   | 3        | 2024-05-03 | 100          |
| Bob     | 2        | 2024-05-02 | 200          |
| Charlie | 1        | 2024-05-01 | 150          |
| David   | NULL     | NULL       | NULL         |
| NULL    | 4        | 2024-05-04 | 80           |


### CROSS JOIN

A CROSS JOIN is a type of SQL join that returns the Cartesian product of the two joined tables. This means that each row from the first table is combined with each row from the second table. The result set includes all possible combinations of rows from both tables.

![[Pasted image 20260624135733.png]]

customers table:

| id  | name    | email               |
| --- | ------- | ------------------- |
| 1   | Alice   | alice@example.com   |
| 2   | Bob     | bob@example.com     |
| 3   | Charlie | charlie@example.com |
| 4   | David   | david@example.com   |

products table:

| id  | name     | amount |
| --- | -------- | ------ |
| 1   | laptop   | 150    |
| 2   | mouse    | 200    |
| 3   | keyboard | 100    |
| 4   | speaker  | 80     |


Query:

```sql
SELECT customers.name AS customer_name, products.name AS product_name, products.amount  
FROM customers  
CROSS JOIN products;
```

Result:

| customer_name | product_name | amount |
| ------------- | ------------ | ------ |
| Alice         | laptop       | 150    |
| Alice         | mouse        | 200    |
| Alice         | keyboard     | 100    |
| Alice         | speaker      | 80     |
| Bob           | laptop       | 150    |
| Bob           | mouse        | 200    |
| Bob           | keyboard     | 100    |
| Bob           | speaker      | 80     |
| Charlie       | laptop       | 150    |
| Charlie       | mouse        | 200    |
| Charlie       | keyboard     | 100    |
| Charlie       | speaker      | 80     |
| David         | laptop       | 150    |
| David         | mouse        | 200    |
| David         | keyboard     | 100    |
| David         | speaker      | 80     |

### SELF JOIN

A SELF JOIN is a type of SQL join where a table is joined with itself. Useful when you want to compare rows within the same table.

```sql
SELECT t1.column1, t2.column2  
FROM table t1  
JOIN table t2 ON t1.common_column = t2.common_column;
```

`t1` and `t2` are aliases for the same table.



employees table:

| id  | name    | manager_id |
| --- | ------- | ---------- |
| 1   | Alice   | 3          |
| 2   | Bob     | 3          |
| 3   | Charlie | NULL       |
| 4   | David   | 2          |

We want to retrieve the names of employees and their respective managers.

Query:

```sql
SELECT e1.name AS employee_name, e2.name AS manager_name  
FROM employees e1  
JOIN employees e2 ON e1.manager_id = e2.id;
```

Result:

The table `employees` is joined with itself using aliases `e1` and `e2`. The join condition is based on the `manager_id` column in `e1` matching the `employee_id` column in `e2`, establishing a relationship between employees and their respective managers.

Here is what the result will look like:

| employee_name | manager_name |
| ------------- | ------------ |
| Alice         | Charlie      |
| Bob           | Charlie      |
| David         | Bob          |



## How to do Joins

### Inner Join (The "Matches Only" Join)

An inner join only keeps rows where the join key exists in **both** tables.

#### Step-by-Step on Paper:

1. Put your pen on the first row of **Table A (Left)** and look at its join key.
    
2. Scan all of **Table B (Right)** looking for that exact key.
    
3. **If you find a match:** Write a new row on your output sheet combining all columns from Table A and Table B. (If it matches 3 rows in Table B, write 3 separate combined rows).
    
4. **If you find NO match:** Do nothing. Move to the next row of Table A.
    
5. Repeat for every row in Table A.
    
6. **Final Cleanup:** Look at Table B. If there are any rows in Table B that you never touched, ignore them completely. They do not go on the paper.
    

### 2. Left Join (The "Keep All Left Rows" Join)

A left join guarantees that **every single row from Table A** makes it onto your output sheet, regardless of whether it finds a match in Table B.

#### Step-by-Step on Paper:

1. Put your pen on the first row of **Table A (Left)**.
    
2. Scan **Table B (Right)** for a matching key.
    
3. **If you find a match:** Write the combined row(s) on your output sheet just like an inner join.
    
4. **If you find NO match:** You must still write this Table A row on your output sheet. Fill in the Table A columns with their actual data, and write **`NULL`** (or leave blanks) for all the columns that belong to Table B.
    
5. Repeat for every row in Table A.
    
6. **Final Cleanup:** Ignore any leftover, unmatched rows in Table B.
    

### 3. Right Join (The "Keep All Right Rows" Join)

A right join is the exact inverse of a left join. It guarantees that **every single row from Table B** makes it onto your output sheet, regardless of whether it matches anything in Table A.

#### Step-by-Step on Paper:

1. Flip your perspective: Put your pen on the first row of **Table B (Right)**.
    
2. Scan **Table A (Left)** for a matching key.
    
3. **If you find a match:** Write the combined row(s) on your output sheet. (Keep your column layout consistent: Table A columns on the left, Table B columns on the right).
    
4. **If you find NO match:** You must still write this Table B row on your output sheet. Fill in the Table B columns with their actual data, and write **`NULL`** for all the columns that belong to Table A.
    
5. Repeat for every row in Table B.
    
6. **Final Cleanup:** Ignore any leftover, unmatched rows in Table A.


### 4. Full Join (The "Leave No Row Behind" Join)

A Full Outer Join combines the behavior of a Left Join and a Right Join. It ensures that **every single row from both tables** makes it onto your output sheet. If a row finds a partner, they combine; if it doesn't, it still gets listed with `NULL` pads.

#### Step-by-Step on Paper:

1. Put your pen on the first row of **Table A (Left)**.
    
2. Scan **Table B (Right)** for a matching key.
    
3. **If you find a match:** Write the combined row(s) on your output sheet. Mark or check off those matching rows in Table B so you know they’ve been used.
    
4. **If you find NO match:** Write the Table A row on your output sheet, and pad all Table B columns with `NULL`.
    
5. Repeat this for every row in Table A.
    
6. **The Second Pass:** Now, look at **Table B**. Find any rows that do _not_ have a checkmark next to them (the ones that failed to match anything in Table A).
    
7. Write those unmatched Table B rows onto your output sheet, padding all the Table A columns with `NULL`.
    

### 5. Cross Join (The "Every Possible Combination" Join)

A Cross Join produces a Cartesian product. It completely ignores keys because **there is no `ON` condition**. Every row from Table A pairs with every row from Table B.

If Table A has 3 rows and Table B has 4 rows, your output sheet _must_ end up with exactly 3×4=12 rows.

#### Step-by-Step on Paper:

1. Put your pen on the **first row of Table A**.
    
2. Go to Table B, and pair this first row of Table A with **every single row in Table B**, one by one. Write each pair on your output sheet.
    
3. Move your pen to the **second row of Table A**.
    
4. Again, pair it with every single row in Table B. Write them down.
    
5. Repeat this process until you have exhausted every row in Table A.
    

### 6. Self Join (The "Photocopy" Join)

A Self Join isn't actually a distinct type of join algorithm; it is just a normal join (Inner, Left, or Full) where **the left table and the right table are the exact same table**.

#### Step-by-Step on Paper:

1. Take your single table and literally **make a physical photocopy of it**.
    
2. Lay the two copies side-by-side on your desk.
    
3. Take a marker and write a distinct role name at the top of each sheet to avoid getting confused.
    
    - _Example:_ Label the left sheet **"Table M (Members)"** and the right sheet **"Table R (Recommenders)"**.
        
4. Now, pick your join type (usually an **Inner Join** or **Left Join**).
    
5. Follow the exact step-by-step paper rules for that join type, treating your two copies as completely separate entities. Use the column you want to map from the left sheet (e.g., `recommendedby`) and match it against the target column on the right sheet (e.g., `memid`).