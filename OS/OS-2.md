
![[Pasted image 20260424011444.png]]

1. Uniprogramming OS (Single CPU)- ONLY 1 process in RAM, CPU utilization is not there. Also processes may starve. Ex- MSDOS
	a process cannot keep both IO and CPU busy at a time
2. Mulitprogramming OS (Single CPU)- >1 process allowed
	the CPU and IO can both be busy (good)
	degree of multiprogramming = number of processes running in MM
	usually, as deg of multiprogramming increases -> CPU is more utlised
	but this only goes on upto a certain limit
	The jobs/processes are in a ready queue
	Context switch happens when a process goes for IO
	Types:
	1. premptive
		processes can be taken out of cpu
	2. non premptive
		process gets cpu time until its done (terminate, io operation)
3. Multitasking/Timesharing OS (single CPU): extension of multipOS, processes execute in round robin fashion
	cpu executes each process for a really short time, and goes through them one by one. This gives the illusion of multitasking
	Its better than multiprocessing OS due to Timesharing. It executes one process for some amount of time then executes some other process
	CPU utilization is High, Starvation is low
4. Multiuser OS
	![[Pasted image 20260424013706.png]]
	Windows- NOT multiuser
	Linux- Multiuser
5. MultiProcessing OS: used in systems with multiple cpus
	Multitasking OS + >=1 CPU
	
	Types:
	2. Tightly coupled (shared memory)
	3. Loosely coupled (distributed memory)
	![[Pasted image 20260424014134.png]]

6. Embedded OS: for embedded computer systems (cars, AC, fridge etc)
7. Real time OS: for large number of events
	Where its required- Low error desired, and works in real time

Batch processing OS
![[Pasted image 20260531192706.png]]
