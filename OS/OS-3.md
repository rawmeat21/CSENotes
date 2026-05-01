
# Process (Run programs)

= Program + runtime activity
program under exectuion

![[Pasted image 20260424020810.png]]
![[Pasted image 20260424020848.png]]
![[Pasted image 20260424020928.png]]
![[Pasted image 20260424020952.png]]

PID- process ID (used to identify process)
PC- program counter
GPR- general purpose register

## PCB (How to save a process!)

Attributes of a process are stored in PCB (process control block) or process descriptor
![[Pasted image 20260424021614.png]]

when a process is stopped and we want to resume it, how does the process have its contents restored?
![[Pasted image 20260424232925.png]]

P1 is running 
when its paused, its entire 'context' is saved in its PCB in OS. PC, register values, etc, everything is copied

Content of PCB= context of process
![[Pasted image 20260424233207.png]]

Context Switch: pause current process and resume/start another process

**Remember that a process can only access what its allowed to. The OS can control its memory usage (addresses its allowed and not allowed), among other things.

**PCB is stored in a protected area in an OS. Processes are NOT allowed to access the PCB

Who does context switch? - Well its the OS, specifically its the **dispatcher** program 


