Source: https://medium.com/@rajiimalleboyina/operating-systems-day-9-virtual-memory-in-os-685db4c93b23
https://medium.com/@maruti.patil20/virtual-memory-57edbd3a3042

## What is Virtual Memory?

We come across situations where one may play a game of size **8 GB** on a computer with only **4 GB of RAM**. One can run multiple programs whose combined size is more than the RAM size. This is because of **virtual memory**.

**Virtual Memory** is a technique used by the operating system that allows a system to run programs **_larger than physical RAM_** by using **_secondary storage (disk)_** as an extension of main memory.

## How it works

Consider a system where the operating system needs **120 MB of memory** to run all active programs, but only **50 MB of physical RAM** is available.

In such a case, the operating system uses a component called the **Virtual Memory Manager (VMM)**. The VMM creates a space of **70 MB** on the hard disk, known as the **paging file or swap file**, which acts as an extension of main memory.

The programs are divided into smaller units called **pages**. During execution, only the pages that are currently required are kept in RAM, while the remaining pages stay in the paging file on disk. When the operating system needs a page that is not present in RAM, it brings that page from disk.

If RAM is already full, the OS removes a less recently used page from memory to make space for the required one. This process of moving pages between RAM and disk is called **swapping** (or paging). Since memory space is limited, the operating system uses **page replacement algorithms** to decide which page should be removed.

This mechanism allows large programs to run smoothly and enables multiple applications to execute simultaneously, even on systems with limited physical memory.


### Thrashing

**Thrashing** is a condition where the operating system spends **_most of its time swapping pages between RAM and disk instead of executing programs._**

![[Pasted image 20260708161426.png]]

Thrashing occurs when too many processes are running while the available RAM is insufficient to hold their active pages. This leads to a high page fault rate, causing the system to spend most of its time swapping pages between memory and disk instead of executing programs.

### Page Fault

A **page fault** occurs when a running program tries to access a part of memory that is not currently present in RAM. Since virtual memory allows pages to be stored on disk, this situation is normal and expected during program execution.

When a page fault happens, the operating system temporarily pauses the program, brings the required page from disk into RAM, updates its records, and then resumes the program. However, if page faults happen too frequently (thrashing), system performance starts to degrade.

In some situations, no pages are loaded into the main memory initially. Pages are loaded only when demanded by the process by generating page faults. This technique is called **Demand Paging**.


### Page Replacement

RAM is limited, and it cannot hold all pages of all running programs at the same time. When a page fault occurs and the RAM is already full, the operating system must free up space before loading the required page.

To do this, the OS decides which existing page in memory should be removed and replaced with the new one. The techniques used to make this decision are called **page replacement algorithms**.

### Page Replacement Algorithms

#### 1. First In First Out (FIFO) Page Replacement

This is the simplest page replacement algorithm. Oldest page in the main memory will be selected for replacement.

**Belady’s Anomaly:**

Increasing the number of page frames causes an increase in the number of page faults. It occurs with First in First Out page replacement algorithm.

#### 2. Optimal Page Replacement

- Replace page that will not be used for longest period of time.
- It has the lowest page-fault rate of all algorithms and will never suffer from Belady’s anomaly.
- It is difficult to implement, because it requires future knowledge of the reference string.

#### 3. Least Recently Used(LRU) Page Replacement

- Page which has not been used for the longest period of time is selected for replacement.
- This algorithm uses past knowledge rather than future.
- This is better than FIFO , but worse compared to optimal.


## DO WE NEED VIRTUAL MEMORY EVEN IF WE INCREASE THE RAM STORAGE?

Virtual memory allows us to use a portion of our hard drive as though it were RAM and combine this part and the real RAM together. When the RAM runs low, virtual memory will move the data out of the RAM also transfer them into a space called paging train. In this way, calculating performance can be bettered to some extent.

It seems that there is no need for us to use virtual memory presently if we’ve a RAM that’s large enough, not to mention that the reading speed of hard drive is slower than it of RAM. also will the running speed be bettered if we disable the virtual memory? In fact, the answer is NO.

![](https://miro.medium.com/v2/resize:fit:840/1*u3k4mq_obTn6yXdeF0d9kg.png)

In fact, numerous of the core functions of Windows and some third- party software will employ paging lines. In this case, third- party software may witness the lack of virtual memory if we choose to disable the ultimate, especially for software like Photoshop. Thus, no matter how large the capacity of RAM is, it’s still necessary for us to enable virtual memory.

In other words, the paging train extends the RAM’s capacity, as it stores RAM data that has not been used or penetrated recently. Also, if you have the paging train enabled, operations that exceed the limited RAM space are automatically transferred to the paging train to be stored to free up RAM. A paging train can be read as a connected knob of data from RAM, which is must briskly than reading the data from multiple locales

Another thing about virtual memory is that Windows only uses paging lines when it’s necessary. In other words, Windows doesn’t use paging lines all the time. So indeed, though we disable virtual memory, the performance of our computer still will not be better at all.