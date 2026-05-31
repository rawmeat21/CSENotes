
![[Pasted image 20260424011444.png]]

1. Uniprogramming OS- ONLY 1 process in RAM, CPU utilization is not there. Also processes may starve. Ex- MSDOS
	a process cannot keep both IO and CPU busy at a time
2. Mulitprogramming OS- >1 process allowed
	the CPU and IO can both be busy (good)
	degree of multiprogramming = number of processes running in MM
	usually, as deg of multiprogramming increases -> CPU is more utlised
	but this only goes on upto a certain limit

	Types:
	1. premptive
		processes can be taken out of cpu
	2. non premptive
		process gets cpu time until its done (terminate, io operation)
3. Multitasking/Timesharing OS: extension of multipOS, processes execute in round robin fashion
	cpu executes each process for a really short time, and goes through them one by one. This gives the illusion of multitasking
4. Multiuser OS
	![[Pasted image 20260424013706.png]]
	Windows- NOT multiuser
	Linux- Multiuser
5. MultiPorcessing OS: used in systems with multiple cpus
	Types:
	1. Tightly coupled (shared memory)
	2. Loosely coupled (distributed memory)
	![[Pasted image 20260424014134.png]]

6. Embedded OS: for embedded computer systems (cars, AC, fridge etc)
7. Real time OS: for large number of events

Batch processing OS
![[Pasted image 20260531192706.png]]
