These notes include practice questions that would be given for a firmware communications engineer role. This is based on the content I reviewed to practice for my CubeSat Software Comms interview. I have 3 sections each with their own questions.
# Task Scheduling Systems
**Q1** What's the difference between polling, interrupts, and scheduled tasks?
	Polling is a method where a device is periodically checking if a condition is true. Interrupts are different in that, they only trigger when an action is performed instead of constantly checking. Scheduled tasks are automated processes that occur at specific times or when conditions are met.

**Q2** Explain the difference between cooperative and preemptive scheduling.
	Preemptive scheduling occurs when process can be interrupted by the OS before completion. Cooperative scheduling is similar in that the process voluntarily yields control to another process. In a non-preemptive scheduling system, processes need to finish before another process can occur. Cooperative scheduling usually occurs on smaller systems where software is designed to work together, making it easier to share time and resources amongst processes.

**Q3** What is a context switch, and how is it triggered?
	When a process is executed by the CPU, registers are filled in with whatever data is required by the process. Context switching is the process of saving a process's state and loading in a new process. The previous state is saved to memory and can be retrieved at a later time.

**Q4** What are the roles of tasks, queues, and semaphores in RTOS?
	- Applications that work in an RTOS can be structured into a sequence of tasks. Each task is works within its own context (no dependencies). Only 1 task is executed at a time. Since the OS is in charge of context switching tasks, each task will have its own stack
	- Queues are used to keep track of different aspects of an OS. For example, you can have a queue of "waiting" processes that keeps track of which processes are waiting to be executed.
	- Semaphores are a synchronization tool used in systems to manage shared resources and coordinate execution. They prevent race conditions.

**Q5** What is a telemetry task?
	A task that who's purpose is to collect and transmit data from a system to central location, usually wirelessly.

**Q6** What happens if a high-priority task runs longer than expected?
	This ultimately depends on what kind of system/scheduler we have but in a preemptive scheduling system, this can cause lower priority tasks to never be run since the OS is still busy running the high priority task. This can cause starvation for some tasks. It can also result in missed deadlines.
# Communication Protocols
**Q1** What is a start bit and stop bit in UART communication?
	UART defines a message to be in the format:
	`| Start bit | Data | End bit |`
	The start and end bit serve as markers to let a receiving device know when a message starts and ends. The start bit always transitions from high to low voltage

**Q2** What is the purpose of a checksum or CRC in a packet?
	A checksum is a piece of data that is calculated from the contents of a data packet and is used to verify integrity of the packet. When data is sent from a CubeSat to ground, it can get corrupted due to noise, signal degrading, or bit flips.
	These tools work by running an algorithm on the sending device. On the receiver end, the same algorithm is ran. If the calculated values do not match, something went wrong and an error is detected.

**Q3** How would you handle data loss or corruption over an unreliable serial link?
	- If data is simple like an integer/decimal number, send the average of multiple readings. This ensures no outliers are recorded.
	- We can use checksum or CRC to determine if our data is corrupted. If it is, throw it away and use the next packet that passes.
	- Add redundant fields like double headers to ensure reliable data is passed even if part of it was corrupted.
	- Use error-tolerant encoding (like FEC) for very noisy links

**Q4** What is AX.25, and how is it used in amateur satellite communications?
	It is a radio-friendly protocol used by CubeSats to communicate with ground stations. Messages are structured as packets that include things like a flag, PID, payload (commands or data), and CRC for error detection.

**Q5** You are given a packet format:
	| Start (1 byte) | Length (1 byte) | Payload (N bytes) | Checksum (1 byte) |
- **a** How would you parse this in C?
	1. Verify start byte
	2. Read the length byte
	3. Extract payload
	4. Verify checksum
- **b** How would you verify the checksum?

# C Programming (Embedded-Focused)
**Q1** What is the difference between `const` and `volatile` in C?
	`const` tells the compiler that a variable will never change values. It always stays the same and any attempted changes would result in a compilation error. `volatile` tells the compiler to not cache the value of a variable or optimize it in any way. Oftentimes, we don't want the compiler to optimize variables out in order to continue using them.
**Q2** Explain how a pointer to a pointer works, and give an example.
**Q3** What are the dangers of using dynamic memory allocation (`malloc/free`) in embedded systems?
	With embedded systems you tend to have a very limited RAM. `malloc` tends to allocate whatever is available and if this happens too many times, it results in fragments of unused memory stuck in-between blocks of used memory. Even if there is enough total memory available for the next allocation, because they're fragmented across the heap, the program will say there is not enough to allocate and run out of memory. This is called **memory fragmentation.**
**Q4** When would you use a `union` instead of a `struct`?
**Q5** Given the example below, what is wrong with it?
```c
char* get_message() {
    char msg[] = "Hello";
    return msg;
}
```
**Q6** How do you ensure atomic access to a shared variable between an ISR and the main loop?
**Q7** Write a function in C that reverses the bits in a byte. For example: Input: `0b00000011` → Output: `0b11000000`

See more info on [[C Programming]] here