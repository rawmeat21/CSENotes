# Process scheduling

Why tho? why schedule processes? - good scheduling ensures resources are used effectively, cpu utilization is also better, without good scheduling, you have an ass system which is slow and shit

## Scheduling queues

The OS keeps the process in 3 queues: Job queue, Ready queue, Device queue

Job queue: processes in new state
Ready queue: processes in ready state
Device queue: processes which are waiting for a device. Each device (printer, monitor, hardisk) has its own device queue.

![[Pasted image 20260425001330.png]]

**Remember that processes are not stored in queues, their contexts are stored.

## Types of schedulers

This are basically 'queue selectors'. They are named according to how frequently they are used. 

1. Long term scheduler- schedules processes from Job queue (new state) -> Ready queue (Ready state)
	
	It can admit a process in 2 ways: either user input (you open some app) or the OS may run its own background processes when it wants

2. Short term scheduler- when there are many processes in ready state, OS has to decide which one to execute (dispatch). this is the short term schduler's job. It puts a process from Ready->Running

3. Mid term scheduler- It is used to replace a process (inactive process) in hard disk for some time so that other processes can run
	![[Pasted image 20260425002801.png]]
	You have a game, mail, music player running, and you want to open Wp. But theres no RAM left. But, the game has been inactive for a while. Mid term scheduler will take (swap out) the Game process and save it in hard disk for sometime, and then wp can take over and run.
	![[Pasted image 20260425003402.png]]
	**Swapping based on priority: Rolling**
	
	Where is the game saved? It is saved in the **swap space**!! This is a special partition in the disk, not usable by user.

	After wp is done, game can swap-in again, its context is loaded again and you can play.

	When a process is swapped out, it is said to be **Suspended**.
	![[Pasted image 20260425004059.png]]

	There are 2 types of suspended: suspended blocked and suspended ready.
	There also may be a scenario where the IO operation is done when process is suspended, in that case it goes from suspended blocked -> suspended ready

	Question- Do new processes have a pcb? if yes, where are they stored? 


![[Pasted image 20260425004709.png]]

IMP: 
 Long term scheduler controls max degree of multiprogramming
 Mid term scheduler reduces degree of mp
 
