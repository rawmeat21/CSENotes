![[Pasted image 20260531183612.png]]
![[Pasted image 20260531184109.png]]

![[Pasted image 20260531184401.png]]

Note: for the scheduling algos, it is assumed that there will be no IO. So a process can leave running state by either preemption or termination

## CPU Scheduling Algorithms

1. FCFS (First come first serve): processes are executed in order which they *arrive*

![[Pasted image 20260531184638.png]]

![[Pasted image 20260531185949.png]]

Convoy effect: When a slow process which takes a lot of time, is scheduled before a faster process, then the system cannot make use of the fast process and the processing becomes slow