Source: https://medium.com/@mkcode0323/indexing-in-dbms-enhancing-performance-and-efficiency-88a9ab0914bb
https://medium.com/@rohmatmret/understanding-hash-indexing-in-databases-11c02b7d4ed1

If the table has no indexes at all, its called a heap. This causes:

- **Slow Reads (Table Scans):** To find specific data, the database is forced to read the entire table from the beginning to the end, resulting in a full table scan. [](https://learn.microsoft.com/en-us/sql/relational-databases/indexes/heaps-tables-without-clustered-indexes?view=sql-server-ver17)
- **Resource Intensive:** Searching and filtering become drastically slower and use a massive amount of memory and disk I/O, particularly as the table grows larger.

## What is indexing?


Indexing in DBMS is the process of creating a data structure, known as an **index**, which allows for quick access to specific data records within a database table.

![[Pasted image 20260623141903.png]]

**Index** holds a sorted copy of selected columns from the table. 

This index structure includes key-value pairs, where:

**key** represents the data column(s) being indexed, 
**value** is the corresponding pointer to the actual record location. 

By using this index structure, the DBMS can directly locate the desired data, eliminating the need for a full table scan.


IMP facts:

- **Data Structure (Logical view):** An index is a data structure (like a B+ Tree) used to search keys efficiently.
    
- **File (Physical view):** To persist this data structure across system reboots, the nodes of this tree must be written into an actual file on the disk.

So, when we say "Index File," it refers to the physical file on disk that stores the nodes of that indexing data structure.


### Types of Indexing:


**_B-Tree Index_**: This is the most commonly used index type, suitable for balanced searching and range queries.


![[Pasted image 20260623142152.png]]


**_Hash Index_**: Ideal for equality-based searches, hash indexes use a hash function to generate a direct link between the search key and the corresponding data.

![[Pasted image 20260623142217.png]]

1. **Create a Hash Index**:

- Suppose you have a `shipments` table with columns such as `shipment_id`, `tracking_number`, `origin`, `destination`, and `status`.
- You frequently search for shipments based on `tracking_number`, so you create a hash index on this column:

```sql
CREATE INDEX idx_tracking_hash ON shipments USING hash (tracking_number);
```

2. **Storing Data in Buckets**:

- When the hash index is created, the hash function maps each unique `tracking_number` to a hash code (e.g., 001, 002, etc.).
- Rows with `tracking_number` values are stored in buckets according to their hash codes, making it possible to access each bucket directly based on the hash.

3. **Searching with Hash Index**:

- When querying for a specific `tracking_number`, the hash index retrieves the hash code for that value.
- The database then goes directly to the bucket associated with the hash code, bypassing irrelevant rows, resulting in very fast lookup times.

Read more: https://medium.com/@rohmatmret/understanding-hash-indexing-in-databases-11c02b7d4ed1


**_Bitmap Index_**: Suited for columns with low cardinality, bitmap (binary arrays) indexes represent data as a bitmap, where each bit corresponds to a specific value.

![[Pasted image 20260623142440.png]]

![[Pasted image 20260624113958.png]]
![[Pasted image 20260624114016.png]]
![[Pasted image 20260624114037.png]]
![[Pasted image 20260624114055.png]]
![[Pasted image 20260624114110.png]]
![[Pasted image 20260624114121.png]]



***Clustered Index:*** In this type of index, the physical order of the data on disk matches the index’s order, leading to improved query performance.

![[Pasted image 20260623142542.png]]

-> The data file (has the records) is ordered (sorted). The data rows are physically stored and ordered on disk to match the logical order of the index key.

-> Usually implemented using a B+ tree, which speeds up insertions. We need to sort the data after each insertion, B+ tree is used to handle this efficiently.

-> When a query searches for a specific key value using a clustering index:

1. The database engine traverses the non-leaf nodes (internal nodes) of the B+ Tree using key comparisons to navigate down to the appropriate leaf node.
    
2. Once the target leaf node is reached, the database engine has directly located the actual data record, eliminating the need for an additional disk I/O operation to fetch the row from a separate data page (heap).

-> Good for queries like: 

```sql
SELECT * FROM transactions WHERE transaction_date BETWEEN '2026-01-01' AND '2026-01-31';
```
(Range queries)

```sql
SELECT * FROM users WHERE user_id = 4194304;
```
(Searching)

```sql
SELECT * FROM logs WHERE log_sequence_id ORDER BY log_sequence_id ASC;
```
(Requiring some ordering)

```sql
SELECT department_id, COUNT(*) FROM employees GROUP BY department_id;
```
(Queries that aggregate data. `GROUP BY`, the engine can execute **stream aggregation** because identical or consecutive keys are physically adjacent.)

For `MIN` and `MAX` functions, the engine does not scan the table; it immediately jumps to the leftmost or rightmost boundary leaf node of the B+ Tree

It is also useful for joins.

#### On Which Key is the Physical Order Maintained?

The physical sorting of data in a clustered index is enforced on the **Clustering Key** (also referred to as the Index Key).

How the database determines this key follows a strict evaluation hierarchy:

1. **Explicit Clustered Index:** If you explicitly define a clustered index on a specific column or set of columns (composite key) using syntax like `CREATE CLUSTERED INDEX`, the database uses those columns.

2. **Primary Key (Default):** If no explicit clustered index is defined, the RDBMS automatically selects the column(s) designated as the `PRIMARY KEY`.
    
3. **Unique Non-Nullable Index:** If no primary key exists, some database engines (such as MySQL InnoDB) will search for the first `UNIQUE` index where all key columns are `NOT NULL` and use it as the clustering key.
    
4. **Internal Hidden Key:** If none of the above are present, the storage engine generates an internal, hidden identifier to physically order the rows.


**The leaf nodes do not point to the rows, the leaf nodes _are_ the rows.** By definition, a clustered index is not a separate structural entity copied from the table; it is the physical organization of the table itself. So, the DB at the time of creation, MUST decide on a clustering index.

Can we change the clustering index? Yes, use `ALTER TABLE`



**_Non-Clustered Index_**: Non-clustered indexes store a copy of the indexed columns along with a pointer to the actual data location.

When you execute an `ORDER BY` query on a column that is _not_ the clustering key, the database engine cannot rely on the physical layout of the table. To tackle this, one of the methods is a non-clustered index.

![[Pasted image 20260623142619.png]]


The database creates a secondary B+ Tree structure for the alternate column. The logical layout of this index matches the sort order of that specific column.

- **Mechanism:** The leaf nodes of a non-clustered index do not contain data rows. Instead, they contain the secondary ==key value and a **pointer**== to the corresponding row in the clustered index (the primary key value).

- **Execution:** The engine scans the secondary index in its sorted order, extracts the primary key pointers, and then performs a **Key Lookup** (or RID lookup) on the clustered index to fetch the remaining data columns.

![[Pasted image 20260624110037.png]]

Source: https://www.youtube.com/watch?v=ITcOiLSfVJQ&t=139s


![[Pasted image 20260624110405.png]]


#### But how will a non-clustered index work with multiple columns?


When you define a composite index:

```sql
CREATE INDEX idx_state_city ON users (state, city);
```

The keys stored within both the internal nodes and the leaf nodes of the B+ Tree are not scalar values; they are composite tuples: `(state, city)`.

The binary evaluation logic used by the storage engine to traverse the tree or insert keys operates on this comparison matrix:

1. Compare `state_1` and `state_2`. If `state_1 != state_2`, the sorting order is determined immediately.
    
2. If `state_1 == state_2`, execute a tie-breaker comparison between `city_1` and `city_2`.

Now if you've ever used pairs in C++, you know that the data is sorted by first key but not the 2nd one neccessarily.

**Case 1: Exact Match on All Columns:**


```sql
SELECT user_id FROM users WHERE state = 'Texas' AND city = 'Dallas';
```

The engine traverses the B+ Tree directly to the single `('Texas', 'Dallas')` composite key node.


**Case 2: Range Search on the Right Column (with Equality on Left):**

```sql
SELECT user_id FROM users WHERE state = 'Texas' AND city LIKE 'H%';
```

The engine drills down to the `'Texas'` section. Because the cities _within_ Texas are sorted, it can execute a localized range scan for cities starting with `'H'`.

**Case 3: Search on Right Column Only:**


```sql
SELECT user_id FROM users WHERE city = 'Dallas';
```

**Why it fails:** The engine cannot traverse the tree. Since the index is ordered by `state` first, rows containing `'Dallas'` are scattered non-contiguously across completely different pages of the index tree (e.g., under `Texas`, under `Georgia`, etc.). The index is structurally useless for an **Index Seek** operation; the optimizer will likely drop back to a full table scan.

Source: Gemini

