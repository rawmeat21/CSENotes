![[Pasted image 20260604153655.png]]


![[Pasted image 20260604152709.png]]

![[Pasted image 20260604152841.png]]

## 1. First In, First Out (FIFO) or First come, first serve (FCFS) - Non preemptive

![[Pasted image 20260604153058.png]]

![[Pasted image 20260604153259.png]]


This problem is generally referred to as the **convoy effect**, where  
a number of relatively-short potential consumers of a resource get queued behind a heavyweight resource consumer.

## 2. Shortest Job First (SJF) - Non preemptive

![[Pasted image 20260604153557.png]]


![[Pasted image 20260604153836.png]]

## 3. Shortest Time-to-Completion First (STCF) / Preemptive Shortest Job First (PSJF)

![[Pasted image 20260604154210.png]]

It can preempt job A when B and C arrive and decide to run another job, perhaps continuing A later.

Any time a new job enters the system, the STCF scheduler determines which of the remaining jobs (including the new job) has the least time left, and schedules  
that one.


### Response Time

![[Pasted image 20260604154455.png]]


STCF and related disciplines are not particularly good for response time. If three jobs arrive at the same time, for example, the third job has to wait for the previous two jobs to run in their entirety before being scheduled just once. While great for turnaround time, this approach is quite bad for response time and interactivity. 

## 4. Round Robin 

Instead of running jobs to completion, RR runs a job for a time slice (sometimes called a scheduling quantum) and then switches to the next job in the run queue. It repeatedly does so until the jobs are finished. For this reason, RR is sometimes called time-slicing. 

![[Pasted image 20260604154950.png]]

![[Pasted image 20260604155025.png]]


Note : The length of a time slice must be a multiple of the timer-interrupt period; thus if the timer interrupts every 10 milliseconds, the time slice could be 10, 20, or any other multiple of 10 ms.


The length of the time slice is critical for RR. The shorter it is, the better the performance of RR under the response-time metric. However, making the time slice too short is problematic: suddenly the cost of context switching will dominate overall performance. Thus, deciding on the length of the time slice presents a trade-off to a system designer, making it long enough to amortize the cost of switching without making it so long that the system is no longer responsive.

![[Pasted image 20260604155217.png]]

**Cost of Context switch**: It does not arise solely from the OS actions of saving and restoring a few registers. When programs run, they build up a great deal of state in CPU caches, TLBs, branch predictors, and other on-chip hardware. Switching to another job causes this state to be ﬂushed and new state relevant to the currently-running job to be brought in, which may exact a noticeable performance cost. 

![[Pasted image 20260604155557.png]]

More generally, any policy (such as RR) that is fair, i.e., that evenly divides the CPU among active processes on a small time scale, will perform poorly on metrics such as turnaround time. Indeed, this is an inherent trade-off: if you are willing to be unfair, you can run shorter jobs to completion, but at the cost of response time; if you instead value fairness, response time is lowered, but at the cost of turnaround time. 


## IO?

![[Pasted image 20260604155914.png]]

![[Pasted image 20260604155937.png]]

![[Pasted image 20260604160130.png]]
![[Pasted image 20260604160144.png]]

![[Pasted image 20260604160158.png]]

