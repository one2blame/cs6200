# Chapter 3

## Process Concept

Early computers executed what were then known as jobs, then time-shared systems emerged that ran user programs or tasks. The concept is that a computer must be able to run multiple different programs at one time, conduct memory management, etc. All of these different activities are called processes.

The status of a process is represented by the program counter and the contents of the processor's registers. The memory layout of a process is typically divided into multiple sections:

* **Text section** - the executable code
* **Data section** - global variables
* **Heap section** - memory that is dynamically allocated during program run time
* **Stack section** - temporary data storage when invoking functions (such as function parameters, return addresses, and local variables)

The text and data sections of a process are static and will remain the same throughout the execution of the process, however, the stack and heap can shrink and grow dynamically during program execution. Even though the stack and heap grow *toward* one another, the operating systems ensures they do not *overlap*.

As a process executes, it changes states with each state being defined by the current activity of that process:

* **New** - the process is being created
* **Running** - instructions are being executed by the processor
* **Waiting** - the process is waiting for some event to occur (I/O or a signal)
* **Ready** - the process is waiting to be assigned to a processor
* **Terminated** - the process has finished execution

The states above are found on all operating systems, however, some operating systems also more finely delineate process states.

It is important to understand that only **one** process may be in the **running** state on any processor core at any instant. Many processes may be **ready** and **waiting**.

All processes within an operating system are represented by a process control block (PCB). PCBs contain these pieces of information about a process:

* **Process state**
* **Program counter** - indicates the address of the next instruction to be executed for the process
* **CPU registers** - vary in number and type depending upon computer architecture; includes accumulators, index registers, stack pointers, general-purpose pointers, condition-code information; this information must be saved when an interrupt occurs to allow the process to be continued correctly when it is rescheduled to run
* **CPU-scheduling information** - includes the process priority, pointers to scheduling queues, and other scheduling parameters
* **Memory-management information** - includes value of the base and limit registers, page tables, segment tables
* **Accounting information** - includes the amount of CPU and real time used, time limits, account numbers, job or process numbers, etc.
* **I/O status information** - includes the list of I/O devices allocated to the process, a list of open files, etc.

## Process Scheduling

The purpose of multiprogramming is to have processes running at all time so as to maximize CPU utilization. Time-sharing switches a CPU core among processes frequently enough that users can interact with each programming while the program is running. Process schedulers meet these objectives by selecting available processes and scheduling them for execution on a core; each core can run one process at a time.

In order to balance multiprogramming and time-sharing, we have to consider the general behavior of a process. Most processes can be defined as I/O-bound or CPU-bound. I/O bound processes spend more time conducting I/O, and CPU-bound processes spend more time doing computations.

Processes are entered into the ready queue, a linked list that has a header which points to the first PCB in the list, and each PCB points to the next process in the queue. Processes that are waiting on some sort of I/O event or signal are placed in the wait queue, which is also a linked list with a header point to the first PCB and each PCB pointing to the process after it.

A CPU scheduler conducts the selection of which processes get to utilize a CPU core for execution, and if memory is overcommitted some operating systems swapping. Swapping is a method of freeing up memory by saving a running process to disk, and vice-versa in order to place a process back in memory.

Context switching happens when a process encounters an interrupt and changes its process state. The operating system conducts a context switch for this process by saving its context (the PCB) using a state save. To resume operations, an operating system conducts a state restore for the process. So, when suspending a process we use a state save and when the process is ready to execute again we use a state restore.

Context switching is expensive and is completely overhead - no instructions are being executed during a context switch. The speed at which context switches occur are mainly based upon hardware support, special instructions required by the operating system, the number of registers that need to be copied, and the memory's speed.

## Operations on Processes

When a parent process creates a child process, the child process will need resources allocated to it to conduct execution (CPU time, memory, files, I/O devices). A child process may be able to obtain these resources directly from the operating system, or it could be constrained to a set of resources defined by the parent. Restricting a child to a set of resources prevents any process from overloading the system by creating too many child processes.

In addition to providing resources, parent processes can also pass along initialization input to the child process. This can encompass file names, file descriptors, device names, etc.

When a parent process spawns another process, two possibilities for the parent's execution exist:
1. The parent continues to execute concurrently with the children.
2. The parent waits until some or all of the children have terminated execution.

There are also two address-space possibilities for the new process:
1. The child process is a duplicate of the parent process (it has the same program data as the parent).
2. The child process has a new program loaded into it.

In UNIX, we use the `fork()` system call to create a new child process. This child process will have a copy of the address space of the parent process allowing the parent to communicate easily with the child. Both the child and the parent will continue execution after the `fork()` call, however, the return code for `fork()` for the child is `0` whereas the return code for `fork()` for the parent is the `pid` of the child.

Typically, one of the two processes uses the `exec()` system call to replace the process's memory space with a new program. The process using `exec()` will now execute an entirely different set of instructions. Now, the two processes are able to communicate and go their separate ways. The parent can continue execution, create more children, or issue the system call `wait()` to suspend its operation until the child terminates execution.

Processes terminate execution and return all resources back to the operating system using the `exit()` system call. The process will then return some exit value, usually an integer, to the waiting parent process. A process can also cause the termination of other processes by invoking system calls (in Windows, the `TerminateProcess()` system call). The parent needs to know the `pid` of the child process to terminate it, thus the identity of the newly created process is passed to the parent. A parent terminates the execution of child processes for a variety of reasons - here are some examples:

* The child has exceeded its usage of some of the resources it was allocated.
* The task assigned to the child is no longer required.
* The parent is exiting execution, and the operating system does not allow a child to continue if its parent terminates.

When a process terminates, it resources are deallocated by the operating system, however, the process's entry in the process table will remain until the parent calls `wait()`. A process spawned by a parent that hasn't called `wait()` yet is known as a **zombie** process. If a parent process terminates without calling `wait()`, the child process will become and **orphan**. In traditional UNIX operating systems, **orphan** processes will be re-assigned `init` as their parent process, and `init` will call `wait()` to terminate all orphans.

## Definitions

* **Process** - a program in execution. The unit of work in most systems. An active entity.
* **System** - consist of a collection of processes: operating-system processes execute system code, and user processes execute user code.
* **Threads** - different parts of a process that run in parallel.
* **Program counter** - represents the status of the current activity of a process
* **Activation record** - contains function parameters, local variables, and the return address; created when a function is called; placed onto the stack; popped from the stack when the function is complete
* **Program** - a passive entity, a list of instructions stored on disk; an executable file
* **State** - state of a process, defined by the current activity of that process.
* **Process control block** - represents processes in an operating system; also called a task control block; represents the context of a process
* **Process scheduler** - selects and available process for program execution on a core
* **Degree of multiprogramming** - the number of processes currently in memory
* **Ready queue** - linked list containing PCBs of ready processes
* **Wait queue** - linked list containing PCBs of processes waiting on some event
* **Dispatch** - when a process is selected by the scheduler for execution
* **Queueing diagram** - a common representation of process scheduling
* **CPU scheduler** - selects from among the processes that are in the ready queue and allocates a CPU core to the selected process
* **Swapping** - moving a process from disk to memory and vice-versa; only necessary when memory has been overcommitted and must be freed up
* **Process identifier** - an integer number representing a process on the operating system
* **Zombie process** - a process that has completed execution but is waiting on the parent process to terminate it.
* **Orphan process** - a process whose parent has completed execution without calling `wait()` to terminate the child.

## Practical Data

* The GNU `size` command can be used to determine the size (in bytes) of each section of a process. The `size` command will return the size values for `text`, `data`, `bss`, `dec`, `hex`, and `filename` of a program. `data` refers to uninitialized data and `bss` refers to initialized data.
* The process control block in the Linux operating system is represented by the C structure `task_struct`, found in `<include/linux/sched.h>`. Within the Linux kernel, all active processes are represented using a doubly linked list of `task_struct`. The kernel maintains a pointer, `current`, to the process currently executing on the system.
* In the beginning stages of smart phones, particularly with Apple iOS, due to the limitations of the hardware components, Apple separated process execution and multitasking based upon the display. Applications running in the **foreground** were described as applications on the display - these applications were considered running. Applications in the **background** remained in memory but did not occupy the screen - these applications had limited execution options. Eventually technology advanced, and multiple **foreground** applications were supported for Apple iOS using **split-screen**.
* Android has always supported multitasking and does not place constraints on what applications can run in the background. Background applications on Android use a **service** which runs on behalf of the background process. Services do not have a user interface and have a small memory footprint, useful for multitasking on mobile devices.
* On a typical Linux operating system, `systemd` serves as the root parent process for all user processes and is the first user process created when the system boots; assigned a `pid` of 1.
* On traditional UNIX systems, `init` serves as the root parent process for all child processes; assigned a `pid` of 1.
* `pstree` displays a tree of all processes in the system.
* In UNIX, a new process is created by the `fork()` system call.
* In UNIX, the `exec()` system call loads a binary file into memory (destroying the memory image of the program containing the `exec()` system call) and starts its execution.
* In Windows, the `CreateProcess()` system call executes similar to the UNIX `fork()`. In contrast to UNIX, Windows `CreateProcess()` requires loading a specified program into the address space of the child process at creation. `CreateProcess()` also expects no fewer than ten parameters, whereas `fork()` is passed no parameters.