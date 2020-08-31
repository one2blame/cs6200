# Chapter 6

## Overview

To avoid race conditions, processes must request permission to enter their critical section to work on some set of the code that accesses shared memory. This request for permission location is known as the entry section. Once the process enters the critical section and then completes all of their instructions, they reach an exit section and a remainder section.

Solutions to the *critical-section problem* must satisfy the following three requirements:
1. Mutual exclusion - if a process is in the critical section, no other processes can be executing in their critical sections.
2. Progress - if no process is executing in its critical section and some process wishes to enter its critical section, then only those processes that are not executing their remainder section can participate in deciding which will enter its critical section next, and this selection cannot be postponed indefinitely.
3. Bounded waiting - there exists a bound, or limit, on the number of times that other processes are allowed to enter their critical sections after a process has made a request to enter its critical section and before that request is granted.

General approaches to handling critical sections in operating systems include *preemptive kernels* and *nonpreemptive kernels*. Nonpreemptive kernels are essentially safe from race conditions, are less responsive as they will run for an arbitrarily long period of time before releasing the CPU. Preempitve kernels are harder to implement because interrupts will disrupt the flow of execution, requiring the programming to solve the critical section problem, however, preemptive kernels are more suitable for real-time programming.

Compare and swap are how mutex locks are designed in software. Lock are either contended or uncontended. A lock is considered contended if a thread blocks while trying to acquire the lock. Vice-versa for uncontended locks.

## Definitions
* **Cooperating process** - a process that can affect or be affected by other processes executing in the system.
* **Race condition** - several processes access and manipulate the same data concurrently and the outcome of the execution depends on the particular order in which the access takes place
* **Critical section** - a segment of code in which the process may be accessing - and updating - data that is shared with other processes
* **Entry section** - a segment of code preceding a critical section in which the process requests permission to enter said critical section
* **Exit section** - a segment of code following a critical section
* **Remainder section** - a segment of code following an exit section
* **Preemptive kernels** - allows a process to be preempted while it is running in kernel mode
* **Nonpreemptive kernels** - does not allow a process running in kernel mode to be preempted
* **Memory barrier** - instructions to ensure the system conducts all loads and stores before and subsequent load or store operation is performed; making all changes to registers visible to other processors.
* **Atomic instructions** - instructions as one uninterruptible unit
* **Mutual exclusion lock** - software based protection tool to solve the critical-section problem
* **Spin lock** - a mutex lock in which the process "spins" on the processor, waiting for the mutex to be available
* **Semaphore** - like a mutex lock but has more sophisticated ways to synchronize activities
* **Counting semaphore** - semaphore value can range over an unrestricted domain
* **Binary semaphore** - can range only between 0 and 1