## Multi level feedback queue

The fundamental problem MLFQ tries to address is two-fold. First, it would like to optimiz turnaround time, which, as we saw in the previous note, is done by running shorter jobs ﬁrst; unfortunately, the OS doesn’t generally know how long a job will run for, exactly the knowledge that algorithms like SJF (or STCF) require. Second, MLFQ would like to make system feel responsive to interactive users (i.e., users sitting and staring at the screen waiting for a process to ﬁnish), and thus minimize response time; unfortunately, algorithms like Round Robin reduce response time but are terrible for turnaround time. Thus, our problem: given that we in general do not know anything about a process, how can we build a scheduler to achieve these goals?

![[Pasted image 20260607105335.png]]
![[Pasted image 20260607105356.png]]


The key to MLFQ scheduling therefore lies in how the scheduler sets priorities. Rather than giving a ﬁxed priority to each job, MLFQ varies the priority of a job based on its observed behavior. If, for example, a job repeatedly relinquishes the CPU while waiting for input from the keyboard, MLFQ will keep its priority high, as this is how an interactive process might behave. If, instead, a job uses the CPU intensively for long periods of time, MLFQ will reduce its priority. In this way, MLFQ will try to learn about processes as they run, and thus use the history of the job to predict its future behavior.

![[Pasted image 20260607105631.png]]


### How to change priority of jobs

**Allotment**- Amount of time a job can spend at a given priority level before the scheduler reduces its priority

(assume the allotment is equal to a single time slice)

• **Rule 3:** When a job enters the system, it is placed at the highest priority (the topmost queue).  
• **Rule 4a:** If a job uses up its allotment while running, its priority is reduced (i.e., it moves down one queue).  
• **Rule 4b:** If a job gives up the CPU (for example, by performing an I/O operation) before the allotment is up, it stays at the same priority level (i.e., its allotment is reset).

![[Pasted image 20260607110227.png]]

![[Pasted image 20260607110506.png]]

![[Pasted image 20260607110655.png]]
![[Pasted image 20260607110703.png]]

### Problems

![[Pasted image 20260607111114.png]]
![[Pasted image 20260607111121.png]]


## Priority boost

**Rule 5:** After some time period S, move all the jobs in the system  
to the topmost queue. (boost priority)

![[Pasted image 20260607111658.png]]

There is a priority boost every 100 ms and thus we at least guarantee that the long-running job will make some progress, getting boosted to the highest priority every 100 ms and thus  
getting to run periodically.

![[Pasted image 20260607111941.png]]


### Better accounting

How to prevent gaming of CPU? 

![[Pasted image 20260607112324.png]]




![[Pasted image 20260607112827.png]]
