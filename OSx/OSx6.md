
## Proportional share scheduler

![[Pasted image 20260607113058.png]]


![[Pasted image 20260607113205.png]]
![[Pasted image 20260607113243.png]]

![[Pasted image 20260607113648.png]]
![[Pasted image 20260607114230.png]]


### Implementation

![[Pasted image 20260607114557.png]]
![[Pasted image 20260607114641.png]]


![[Pasted image 20260607114937.png]]
![[Pasted image 20260607114945.png]]



## Stride scheduling (deterministic fair share scheduler)

![[Pasted image 20260607115301.png]]


### A sexy example
![[Pasted image 20260607115650.png]]
![[Pasted image 20260607115705.png]]

Why even use lottery scheduling then? - no global state

![[Pasted image 20260607115728.png]]



## CFS (Linux Completely fair scheduler)

![[Pasted image 20260607115951.png]]

**Note: The Linux kernel uses the ==EEVDF (Earliest Eligible Virtual Deadline First) scheduler== as its default CPU scheduler for normal processes, having replaced the long-standing CFS (Completely Fair Scheduler).**



