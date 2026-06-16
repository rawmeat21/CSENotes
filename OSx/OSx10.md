## Managing free space

It is easy when the space you are managing is divided into ﬁxed-sized units; in such a case, you just keep a list of these ﬁxed-sized units; when a client requests one of them, return the ﬁrst entry.

Where free-space management becomes more difﬁcult is when the free space you are managing consists of variable-sized units

This arises in a user-level memory-allocation library (as in malloc() and free()) and in an OS managing physical memory when using segmentation to implement virtual memory. In either case, the problem that exists is known as external fragmentation.

![[Pasted image 20260616202532.png]]


### Assumptions

Assume a basic interface such as that provided by malloc() and free(). Speciﬁcally, void malloc(size t size) takes a single parameter, size, which is the number of bytes requested by the application; it hands back a pointer (of no particular type, or a void pointer in  
C lingo) to a region of that size (or greater).

The complementary routine void free(void *ptr) takes a pointer and frees the corresponding  
chunk. 

Note the implication of the interface: the user, when freeing the  
space, does not inform the library of its size; thus, the library must be able  
to ﬁgure out how big a chunk of memory is when handed just a pointer  
to it. 


The space that this library manages is known historically as the heap,  
and the generic data structure used to manage free space in the heap is  
some kind of free list.

Assume that primarily we are concerned with external fragmentation, as described above. Allocators could of course also have the problem of internal fragmentation; if an allocator hands out chunks of memory bigger than that requested, any unasked for (and thus unused) space in such a chunk is considered internal fragmentation.


Assume that once memory is handed out to a client, it cannot be relocated to another location in memory. For example, if a program calls malloc() and is given a pointer to some space within the heap, that memory region is essentially “owned” by the program (and cannot be moved by the library) until the program returns it via a correspond-  
ing call to free(). Thus, no compaction of free space is possible, which would be useful to combat fragmentation.

Assume that the allocator manages a contiguous region of bytes. In some cases, an allocator could ask for that region to grow; for example, a user-level memory-allocation library might call into the kernel to grow the heap (via a system call such as sbrk) when it runs out of space. However, for simplicity, we’ll just assume that the region is a single ﬁxed size throughout its life.


### Splitting

![[Pasted image 20260616203817.png]]

Assume we have a request for just 1 byte of memory. In this case, the allocator will perform an action known as **splitting:**

Find a free chunk of memory that can satisfy the request and split it into two. The ﬁrst chunk it will return to the caller; the second chunk will remain on the list.

If a request for 1 byte were made, and the allocator decided to use the second of the two elements on the list to satisfy the request, the call to malloc() would return 20 (the address of the 1-byte allocated region) and the list would end up looking like this:

![[Pasted image 20260616204002.png]]

### Coalescing

What happens when an application calls free(10), thus returning the space in the middle of the heap?

![[Pasted image 20260616204110.png]]

You need to combine these.

![[Pasted image 20260616204137.png]]

(After coalescing)


### When you free(ptr), how does the allocator know how much memory to free?

There is extra information in a header block which is kept in memory, usually just before  
the handed-out chunk of memory.

On malloc(20):

![[Pasted image 20260616204627.png]]

(magic number is used for sanity checking)

When freeing, pointer arithmetic is done to get the size of the block. 

**Note: when a user requests N bytes of memory, the library does not search for a free chunk of size N ; rather, it searches for a free chunk of size N plus the size of the header.**


