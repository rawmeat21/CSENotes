Source: https://varad-kajarekar19.medium.com/paging-in-os-5a4d96a6ddca
https://unstop.medium.com/what-is-paging-in-operating-system-9689fdf2e19a
https://medium.com/@humbertofilho_30158/operating-systems-an-introduction-to-paging-eec1cb625b50

## What the fuck is paging?

[Paging](https://dare2compete.com/blog/difference-between-paging-and-segmentation) is ==a static memory== allocation method that allows a process’s physical address space to be of non-contiguous type. 

It lets the operating system fetch processes from secondary storage in the form of 'pages' and place them in memory. 

The paging hardware and operating system are integrated to implement the paging process.

First, look into virtual vs physical memory:

- **Logical Memory (Virtual Memory):** This is the memory space that the CPU generates and a running program sees. To the program, its memory looks like a single, continuous, contiguous block of addresses.
    
- **Physical Memory (RAM):** This is the actual hardware memory chip in the system. In reality, it is highly fragmented and shared among dozens of different processes simultaneously.

1. The Physical Address Space is conceptually divided into a number of fixed-size blocks, called **frames**.

2. The Logical address Space is also split into fixed-size blocks, called **pages**.

3. Page Size = Frame Size

![[Pasted image 20260708155201.png]]

At allocation time, the OS allocates the necessary number of page frames to the process. The translations from virtual pages into physical page frames are kept in a per-process data structure called the page table.


### Example of Paging

For example, Let’s say the main memory size is 64B and the frame size is 4B then, No of frames = 64/4 = 16.

There are 4 processes. The size of each process is 16B and page size is also 4B then, No of pages in each process = 16/4 = 4 These pages may be stored in the main memory frames in a non-contiguous form, depending on their availability

![[Pasted image 20260708155337.png]]


### Address translation

Suppose a page size of k bits, an address space m bits long, and a physical address space of size n bits. Given a virtual address, we follow the algorithm below to obtain the corresponding physical address:

- The virtual address is split into an offset (the k least significant bits) and a Virtual Page Number (VPN, the remaining bits):

![[Pasted image 20260708155559.png]]

The VPN is the page where this virtual address is placed in the process address space. The offset is the “delta” of the memory address inside the page (how far it is from the first address in the page):

![[Pasted image 20260708155641.png]]


- We need to find the corresponding Page Frame Number (PFN) in the physical memory. The page table is leveraged for this purpose.

![[Pasted image 20260708155710.png]]


### Page table

The page table can be seen as an array-like data structure where the index corresponds to the virtual page being mapped. The value, often referred to as the Page Table Entry (PTE), does not consist solely of the PFN. Besides this information, the PTE has a couple of additional bits:

- P (Present bit): indicates whether this page is in physical memory or on disk.
- R/W (Read/Write bit): determines whether writes are allowed to this page.
- U/S (User/Supervisor bit): determines if user-mode processes can access the page.
- PWT, PCD, PAT, and G: determine how caching works for these pages.
- A (Access bit): tracks whether a page has been accessed. It is useful in determining which pages are popular and, therefore, should be kept in memory.
- D (Dirty bit): indicates whether the page has been modified since it was brought into memory.

### Advantages:

- Flexibility: supports the effective abstraction of the address space, regardless of how a process utilizes it (i.e., how the heap and stack grow or are used).

- Simple free-space management: the OS keeps a free list mapping the available pages and, at allocation time, returns the first ones.

- With the help of **Paging, the problem** of external fragmentation is solved.
### Disadvantages:

- An extra memory access is needed whenever a virtual address is accessed within a process (one access to fetch the translation from the page table and another one to access the content of the corresponding physical address). This makes the translation process slower.

- The page table is a per-program data structure. A single-page table can be large (i.e., contain several mappings). Because Operating Systems can have hundreds of running processes, this can consume a lot of memory resources.

- In Paging, sometimes the page table consumes more memory. Internal fragmentation is caused by this technique.