Source: https://medium.com/@mkcode0323/indexing-in-dbms-enhancing-performance-and-efficiency-88a9ab0914bb
https://medium.com/@rohmatmret/understanding-hash-indexing-in-databases-11c02b7d4ed1

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


**_Bitmap Index_**: Suited for columns with low cardinality, bitmap indexes represent data as a bitmap, where each bit corresponds to a specific value.

![[Pasted image 20260623142440.png]]


***Clustered Index:*** In this type of index, the physical order of the data on disk matches the index’s order, leading to improved query performance.

![[Pasted image 20260623142542.png]]



**_Non-Clustered Index_**: Non-clustered indexes store a copy of the indexed columns along with a pointer to the actual data location.

![[Pasted image 20260623142619.png]]


