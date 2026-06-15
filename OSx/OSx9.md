## Segmentation

![[Pasted image 20260615185629.png]]

The process is not using the free space between the stack and heap, so its being wasted.

**Segment** - segment is just a contiguous portion of the address space of a particular length.

Instead of having just one base and bounds pair in our MMU, we have a base and bounds pair **per logical segment of the address space**

There are 3 logical segments here: code, stack, heap.

Segmentation approach - 

![[Pasted image 20260615190304.png]]

3 base and bounds register pairs are used to track the address space for each segment.

![[Pasted image 20260615190347.png]]

(Look at the first picture to understand the sizes)

You can see from the ﬁgure that the code segment is placed at physical address 32KB and has a size of 2KB and the heap segment is placed at 34KB and has a size of 3KB.

 Physicial address = Base for segment + segment offset in process' address space

segment offset in process' addres space =  The address you are trying to use - Where your segment is in process' address space

Example: Suppose the process tries to access address 4200 (on process' address space). Its on the heap (see 1st figure!). BUT, heap is itself at address 4k. 

So offset from heap = 4200 - 4096 = 104

Then physical address = Base + 104 = 34k + 104, which is valid as its <= 34 + 3 = 35k


#### Illegal access? 

You get **segmentation fault.**


### But how does the hardware know which segment it is?

#### Explicit approach

In the example we have 14 bit address (16 kb).

There are 3 segments.

=> Use top 2 bits from (13, 12) to find which segment

00 - code
01 - heap
11 - stack

Problem with this approach - IT LIMITS  THE SIZE OF A SEGMENT TO 4KB (HERE). 

How many address have top 2 bits = 00? -> 2^12 = 4KB

#### Implicit approach

The hardware determines the segment by noticing how the address was formed. 

-> If, for example, the address was generated from the program counter (i.e., it was an instruction fetch), then the address is within the code segment; 

-> If the address is based off of the stack or base pointer, it must be in the stack segment; 

-> Any other address must be in the heap.



#### Also, if you notice, we NEED to know whether a segment grows up or down. 

![[Pasted image 20260615193233.png]]


### Coarse Grained vs Fine grained

Coarse- chops up the address space into relatively large, coarse chunks
Fine- address spaces can consist of a large number of smaller segments. Supporting many segments requires even further hardware support, with a **segment table** of some kind stored in memory.


### OK, but how to do context switch?

Just store the segment registers.

#### OK, what if size of segment needs to grow?

A program may call malloc() to allocate an object. 

In some cases, the existing heap will be able to service the request, and thus malloc() will ﬁnd free space for the object and return a pointer to it to the caller. 

In others, however, the heap segment itself may need to grow.  

In this case, the memory-allocation library will perform a system call to grow the heap (e.g., the traditional UNIX sbrk() system call). The OS will then (usually) provide more space, updating the segment size register to the new (bigger) size, and informing the library of success; the library can then allocate space for the new object and return successfully  
to the calling program.

**Do note that the OS could reject the request, if no more physical memory is available, or if it decides that the calling process already has too much.**


#### OK, how does the OS manage free space in memory? When OS allocates a new process, how does it find the free space? The address space can have variable sizes so we can't use a free list...

**External fragmentation**- Physical memory quickly becomes full of little holes of free space, making it difﬁcult to allocate new segments, or to grow existing ones. We call this problem external fragmentation

![[Pasted image 20260615195125.png]]


In this example, a process comes along and wishes to allocate a 20KB segment. There is 24KB free, but not in one **contiguous** segment (rather, in three non-contiguous chunks). 

Thus, the OS cannot  satisfy the 20KB request. (Similar problems could occur when a request to grow a segment arrives; if the next so many bytes of physical space are not available, the OS will have to reject the request, even though there may be free bytes available elsewhere in physical memory)

##### Solution: Compact the memory

But, compaction takes time

Compaction also makes requests to **grow existing segments** hard to serve, and may thus cause further rearrangement to accommodate such requests.

##### Another solution: Use a free list management algorithm

But, external frag can't really be avoided, you can only minimise it.


