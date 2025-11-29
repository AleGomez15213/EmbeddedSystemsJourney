RTOS stands for *Real Time Operating System*. It is a type of operating system that allows multiple tasks to run at the same time. It contains a specialized scheduler that is in charge of allocating the appropriate time for each task. 

# RTOS vs General Purpose OS
A general purpose OS is designed to simplify the life of the user. This means that it will organize tasks as it sees fit in the name of simplifying the user experience. This however is not always what developers want. There are moments where we need the OS to respond to external events in a timely manner (within milliseconds). RTOS specialize in performing for time-critical operations.

**A regular OS is optimized for average performance. An RTOS is optimized for guaranteed maximum latency.** For this reason, they are usually used in microcontrollers and are smaller in size.