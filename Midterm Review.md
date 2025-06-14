Q1: Explain the difference between preemptive and non-preemptive scheduling.

**Preemptive scheduling is a form a scheduling where processes can be interrupted by other higher-priority processes. Non-preemptive requires processes to finish before beginning other ones.**

Q2: Suppose that the following processes arrive for execution at the times indicated. Answer the questions below

| Process | Arrival Time | Burst Time |
| ------- | ------------ | ---------- |
| P1      | 0.0          | 8          |
| P2      | 0.4          | 4          |
| P3      | 1.0          | 1          |
1. What is the average turnaround time for these processes with the FCFS scheduling algorithm?
	P1 completes after 8ms
	P2 completes after 12ms
	P1 completes after 13ms
	Turnaround = (8+11.6+12) / 3 = **10.53ms**
2. What is average turnaround time if it was SJF scheduling algorithm?
	P1 completes after 8ms
	P3 completes after 9ms
	P2 completes after 13ms
	Turnaround = (8+8+12.6) / 3 = **9.53ms**

Q3: Using Amdahl's Law, calculate speedup gain with 60% parallel component for...
$$speedup\ge \frac{1}{S+\frac{1-S}{N}}$$
> where N is number of cores and S is parallel component in decimal form.

(a) 2 processing cores
$$\frac{1}{0.6+\frac{0.4}{2}} = \frac{1}{0.6+0.8}=\frac{1}{1.4}$$
(b) 4 processing cores
$$\frac{1}{0.6+\frac{0.4}{4}} = \frac{1}{0.6+1.6}=\frac{1}{2.2}$$

Q4: True or False questions
(a) User level threads are unknown by the kernel, whereas the kernel is aware of kernel threads
**True**

(b) On systems using either many-to-one or many-to-many model mapping, user threads are scheduled by the thread library, and the kernel schedules kernel threads
**True**

(c) Kernel threads need not be associated with a process, whereas every user thread belongs to a process
**True**

(d) Kernel threads are generally more expensive to maintain than user threads, as they must be represented with a kernel data structure.
**True**

*Study Chapter 4*

Q5: How many processes are created by the program below:

```c
[...]
int main() {
	/* fork a child process */
	fork();
	/* fork another child process */
	fork();
	/* fork amother child process */
	fork();
	
	return 0;
}
```

**8 processes are created**

Q6: When a process creates a new process using the `fork()` operation, which of the following states is shared between the parent and child process? (a) Stack, (b) Heap, (c) Shared memory segments

**Only the shared memory segments are shared. New copies of stack and heap are made**

Q7: A microkernel is a kernel that _ _ _ _
**is stripped of all nonessential components**

Q8: Which of the following is not a technique that allows a program to be run by different operating systems?
**The program has been compiled into a binary executable file**

Q9: Mix and Match
Initialize a pthread attribute -> pthread_attr_init()
Create a thread that runs the function named work() -> pthread_create()
Wait for the thread to complete -> pthread_join()

Q10: Consider the code below

```c
pid_t pid;

pid = fork();
if (pid == 0) {
	fork();
	thread_create(. . .);
}
fork();
```

(a) How many unique processes are created?
**6 processes**
(b) How many unique threads are created?
**2 threads**

Q11: What is the output of line C and P?
![[Pasted image 20250506193857.png]]

**CHILD: value = 5**
**PARENT: value = 0**


## Review
**Unbounded buffer:** places no practical limit on the size of a buffer.