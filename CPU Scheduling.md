*from course EECS 111 - Operating Systems*
# Priority Scheduling
A **non-preemptive** scheduler will allow process to finish if another process is suddenly introduced. A **preemptive** scheduler will interrupt a current process of lower priority.

> Note: make sure to know how to calculate avg wait time in preemptive and non schedulers/scenarios.

**Starvation** is when lower priority process never execute
**Aging**

If newer high-priority (greater priority than p5) processes keep being introduced. It will never allow P4 to execute bc it's priority 5.

# Multilevel Queue
A ready queue is split into **foreground (interactive)** and **background (batch)** queues. Each queue has its own scheduling algorithm.

![[Pasted image 20250429205743.png|500]]
Scheduling needs to be done between queues. If processes stay in their respective queues forever, it can cause starvation. Oftentimes, process will be moved from one queue to a higher priority one to make sure it is ran at some point. *Aging helps because process will be moved up the queue the longer it waits. This means it will eventually run.*

# Thread Scheduling
**Asymmetric multiprocessing** means only one processor accesses the system data structures, alleviating the need for data sharing.
- One CPU is chosen as a master core which tells the other CPUs which kernel threads to execute. Pro is its easier to manage, con is one CPU is now in charge of the others and can be bottlenecked. Modern systems don't use this.
**Symmetric multiprocessing (SMP)** occur when each processor is self-scheduling, all processes in common ready queue, or each has its own private queue of ready processes.
- Work is distributed across all cores.
- Efficiency depends on implementation

**NUMA** allows us to use MSB as memory addresses. The rest of the bits will be data.
![[Pasted image 20250429211602.png|500]]

**Soft affinity:** When an operating system has a policy attempting to keep a process running on the same processor but this is not enforced.
**Hard affinity:** When an operating system forces a process to run on a specific processor at all times.

To calculate CPU utilization: `processing time / period`
The period is the allowed time a process has to complete. When multiple processes run at the same time, the processing time of a process may exceed the period. It's important to be able to schedule process to ensure they meet the deadline requirements.