Segmentation is a memory management technique used in operating systems to divide a computer’s memory into multiple segments, each with its specific size and purpose.


- **Segment:** A segment is a contiguous block of memory with a specific size and purpose, such as code, data, or stack. Each segment is identified by a unique segment number.

- **Segment Table:** A segment table is a data structure that stores information about each segment, including its base address, size, and access rights (read-only, read-write, execute-only, etc.).

- **Segment Descriptor:** A segment descriptor is an entry in the segment table that defines the characteristics of a segment.

### Segment table:

**Base register** - base address of the segment 
**Limit register** - length of the segment.

![[Pasted image 20260708160853.png]]

#### Translation of Logical address into a physical address by segment table

CPU generates a logical address which contains two parts:

1. Segment Number
2. Offset


The **Segment number** is mapped to the segment table. The limit of the respective segment is compared with the **offset**. If the offset is less than the limit then the address is valid otherwise it throws an error as the address is invalid.

![[Pasted image 20260708160941.png]]


