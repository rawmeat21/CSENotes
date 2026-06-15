## Segmentation

![[Pasted image 20260615185629.png]]

The process is not using the free space between the stack and heap, so its being wasted.

**Segment** - segment is just a contiguous portion of the address space of a particular length.

Instead of having just one base and bounds pair in our MMU, we have a base and bounds pair **per logical segment of the address space**

There are 3 logical segments here: code, stack, heap.

Segmentation approach - 

![[Pasted image 20260615190304.png]]

