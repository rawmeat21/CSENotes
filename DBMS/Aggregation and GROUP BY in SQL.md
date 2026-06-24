
- `COUNT()` – Counts the number of rows.
    
- `SUM()` – Adds up numeric values.
    
- `AVG()` – Calculates the average of numeric values.
    
- `MIN()` – Finds the smallest value.
    
- `MAX()` – Finds the largest value.


Without group by, these functions are applied on all entries.

Example:

```sql
SELECT COUNT(employee) FROM employees;
SELECT COUNT(employee) FROM employees WHERE age >= 21;

```

**COUNT(column) vs COUNT( * )**

-> When you specify the column, it will ignore rows where column values are NULL
-> COUNT( * ) will count ALL rows


### Conditional aggregation

**Always remember: aggregation functions (or other SQL statements  I guess) take in an expression. On each row, an expression is evaluated. A column name evaluates to the value in the column for that row. Even CASE works similarly. Column -> Value in that column for current row

#### Method 1 (Works in some languages like Postgres)

|id|type|amount|
|---|---|---|
|1|income|500|
|2|expense|100|
|3|income|200|
|4|expense|50|

```sql
SELECT 
    SUM(amount) FILTER (WHERE type = 'income') AS total_income,
    SUM(amount) FILTER (WHERE type = 'expense') AS total_expense
FROM transactions;
```

```sql
SUM(amount) FILTER (WHERE type = 'income') AS total_income
```

This means "Find the sum of amount column, but only consider the rows for which type = 'income' "


#### Method 2:

```sql
SELECT 
    SUM(CASE WHEN type = 'income' THEN amount ELSE 0 END) AS total_income,
    SUM(CASE WHEN type = 'expense' THEN amount ELSE 0 END) AS total_expense
FROM transactions;
```


```sql
COUNT(CASE WHEN type = 'income' THEN 1 ELSE 0 END) 
```

The problem with this is if type is NULL, then we get 0. 0 is a NON-NULL value and COUNT(anything that isn't NULL) is 1. So, those rows are counted too.

Fix:

```sql
COUNT(CASE WHEN type = 'income' THEN 1 END) 
```

OR

```sql
COUNT(*) FILTER (WHERE type = 'income')
```



## Aggregation with GROUP BY

A GROUP BY makes things a little different. First, it will create several 'buckets' for each unique value in a specified column.

Then, it does a normal scan. Each time, it evaluates the expression inside the () of the aggregator, gets the value and performs the aggregrator function with the value currently in the bucket for that column's entry and stores the result in the bucket. 

Kind of like going through an array of (key, value) pairs, using a map to create buckets for all keys, then again going through the array and doing mp[key] ~ value for each (key, value) pair. ~ is the aggregator function. Finally, after the scan, it will return the map's (key, value) pairs (not the array's!). I call this the bucket table.

```sql
SELECT employee, SUM(amount * 1.10) AS total_with_tax
FROM sales
GROUP BY employee;
```


```sql
SELECT 
    CASE WHEN amount < 150 THEN 'Small Sale' ELSE 'Large Sale' END AS sale_tier,
    COUNT(*) AS total_count
FROM sales
GROUP BY CASE WHEN amount < 150 THEN 'Small Sale' ELSE 'Large Sale' END;
```

1. First, group by runs, it will create exactly 2 buckets ('Small Scale' and 'Large Scale')
2. Then, it goes through each row again. Each time, it decides by amount which bucket to put in the value
3. After deciding, it will perform aggregation COUNT( * ), which will return 1. Hence the bucket's value increases by 1 

A better analogy is that there are no 2 scans, this is all done in 1 single scan.

#### HAVING

What if you wish to filter on the bucket table?

Extract the employees who had >200 in sales:

```sql
SELECT employee, SUM(amount) 
FROM sales
WHERE SUM(amount) > 200
GROUP BY employee;
```

This is wrong. WHERE runs before any aggregation, it runs before the Group by scan is even done. Simply put, _An aggregate may not appear in the WHERE clause._

```sql
SELECT employee, SUM(amount) 
FROM sales
GROUP BY employee
HAVING SUM(amount) > 200;
```

Why this works: Basically HAVING runs _after_ the bucket table is made.

**Step 1: `WHERE` Filter

The database scans the rows. It looks at Bob's sale of `50`. Because `50 > 60` is False, **that row is instantly thrown in the trash**. It never makes it to the aggregation step.

**Step 2: The `GROUP BY` & Aggregation (Single-Pass)**

The remaining rows go through the conveyor belt to form the buckets:

Bucket table is formed.

**Step 3: The `HAVING` Filter (Bucket-by-Bucket)

Now, the `HAVING SUM(amount) > 200` clause wakes up. It loops through the completed buckets, evaluating the expression against the final aggregate values.





