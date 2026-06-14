## How does a DB get the data to you?

![[Pasted image 20260614141017.png]]

Data file (table) is stored in disk in blocks. Say there are 10k records and a block holds 10 records. Then there are 1k blocks.

Now if the data is sorted by PK, then we could binary search for the record by the PK.

**We cannot apply binary search directly in actual disk records because records are not necessarily stored in a contiguous manner in the disk, we use B tree (threaded M way tree) for this. It makes insertion and deletion faster as well.**

![[Pasted image 20260614141317.png]]

We can store an INDEX data structure. It stores a search key and base pointer pair. this BP maps to a block in memory. 

The Index file is always sorted by the key (so we can apply BS)

	Search key may not be primary key always


Dense indexing- when block size is 1 (every row is indexed)
Sparse indexing- when we index some of the rows only

The above example is sparse indexing

![[Pasted image 20260614142641.png]]

![[Pasted image 20260614143843.png]]

For non key attr. indexing, values may repeat. In that case, the index file has k entries = no. of distinct entries in the table. To look up a value with key = 3, we need to find the first occurence of 3 first (use BS considering the table to be sorted by the value)