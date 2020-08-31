# Chapter 4

## Overview

A traditional process has a single thread of control, however, if it has multiple threads it can perform more than one task at a time. An application is typically implemented as a process with several threads of control. A few examples of multithreaded applications are:

* An application that creates photo thumbnails from a collection of images may use a separate thread to generate a thumbnail from each separate image.
* A web browser might have one thread display images or text while another thread retrieves data from the network.
* A word processor may have a thread for displaying graphics, another thread for responding to keystrokes from the user, and a third thread for performing spelling and grammar checking in the background.

Before multithreading was invented, applications would create separate processes to conduct the same steps of execution in order to conduct more computation in parallel. At scale, however, create an entirely new process is more time consuming and resource intensive than utilizing threads.

The benefits of multithreading can be broken down into four categories:

1. Responsiveness - allows a program counter to continue running even if part of the program is blocked or performing a lengthy operation. This allows the user to perceive more responsiveness from the application, rather than the application freezing as it conducts all operations sequentially.
2. Resource sharing - processes are only able to share resources through shared memory and message passing, requiring those mechanisms to be explicitly defined by the programmer. Threads share the memory and resources of the process they belong to by default.
3. Economy - because threads share the resources of the process to which they belong, it is more economical to create and context-switch threads.
4. Scalability - threads can run in parallel on different processing cores, whereas a single-threaded process can run on only one processor, regardless of how many are available.

## Multicore Programming

Multithreaded programming provides a mechanism for more efficient use of multiple computing cores and improved concurrency. On a system with a single computing core, an application with four threads will only be able to execute one at a time, however, each thread will progress in execution. In contrast, on a multicore processor, these four threads will be able to run in parallel, as multiple threads will be able to execute simultaneously. This is the distinction between concurrency and parallelism.

As multicore processing became more popular, designers of operating systems had to write scheduling algorithms that supported parallel execution. Five areas present challenges in programming for multicore systems:

1. Identifying tasks - examining applications and finding areas of execution that can be divided into separate, concurrent tasks; ideally independent of one another and can run parallel on individual cores.
2. Balance - ensuring that tasks perform equal work of equal value.
3. Data splitting - ensuring the data accessed and manipulated by the tasks is divided to run on seperate cores.
4. Data dependency - ensuring dependencies for data between tasks is resolved and operations on data are synchronized.
5. Testing and debugging - ensuring concurrent programs and their paths of execution are bug free.

Data parallelism and task parallelism contrast because data parallelism is concerned with distributing data across multiple cores, conducting the same operation on each datum, while task parallelism separates tasks across multiple cores in threads - data agnostic.

## Multithreading Models

Support for threads can be provided at the user level, i.e. user threads, or by the kernel, i.e. kernel threads. There are three common ways of establishing relationships between threads: many-to-one model, one-to-one model, and the many-to-many model.

In the many-to-one model, many user threads are mapped to one kernel thread. Thread management is conducted by a thread library in user space, increasing efficiency, however, the entire process will block if a thread makes a blocking system call for some I/O or signal. Because only one thread can access the kernel at a time, multiple threads of a process are unable to run in parallel on multicore systems.

In the one-to-one model, each user thread is mapped to a kernel thread providing more concurrency than the many-to-one by allowing other threads to run if the thread makes a blocking system call. This also allows other threads to run in parallel on multiprocessors. The biggest down-side is that for every user thread created, a kernel thread has to be created as well - this large number of kernel threads may burden the performance of a system.

The many-to-many model multiplexes many user level threads to a smaller or equal number of kernel threads. The many-to-many model suffers none of the shortcomings of the previous two threading models: the programmer can create as many threads as necessary and the kernel threads can run in parallel on a multiprocessor. When the user threads execute a system call that will block, the kernel can schedule another thread for execution.

One variation of the many-to-many model still multiplexes many user level threads to a smaller or equal number of kernel threads but also allows a user level thread to be bound to a kernel thread. This variation is referred to as the **two-level model**.

## Thread Libraries

Thread libraries can be implemented in two ways:

1. The first approach is to provide a library entirely in user space with no kernel support. All code and data structures for the library exist in user space - invoking a function in the library does not require system calls.
2. The second approach is to implement the library in kernel space - the library is directly supported by the operating system. Code and datastructures for the library exist in kernel space - invoking a function in the API for the library typically results in a system call to the kernel.

Two strategies exist for creating multiple threads:

1. Asynchronous threading - once the parent thread creates a child thread, the parent resumes execution.
2. Synchronous threading - the parent thread creates one or more children threads and then waits for all children to terminate before resuming execution.

## Implicit Threading

Soon enough, most applications are going to be using hundreds to thousands of threads, and application developers will have to learn how to handle these new requirements. One way to address this issue is by using implicit threading, moving the design implementation of multithreading away from the developers and having the compiler and run-time library figure out how to implement the application in a multithreaded fashion. How this would be achieved is by the developer identifying *tasks*, not threads, that can run in parallel. Then the run-time library would map each task to a separate thread in the many-to-many model.

Creating threads as requirements increase is bad practice. It takes work to create the thread, and if we're just going to destroy the thread after we're done, as well, this could become computationally expensive. We also aren't placing bounds on the number of threads that a process can create, making this possibly resource intensive. To solve this issue, we utilize **thread pools**.

Thread pools offer these benefits:
1. Servicing a request with an existing thread is often faster than waiting to create a thread.
2. Thread pools limit the number of threads in existence.
3. Separating the the task to be performed from the mechanics of creating the task allows us to use different strategies for running the task.

More sophisticated methods of thread pooling exist for adjusting the number of threads in a pool. Dynamically increasing or decreasing the number of threads based upon usage patterns and requests, etc.

## Threading Issues

So what happens if we call `fork()` or `exec()` inside of a thread? Some UNIX systems have chosen to support two versions of `fork()`: one that duplicates all threads and one that duplicates only the thread invoking the `fork()` system call. `exec()` works the same way as usual for threads - if a thread invokes `exec()` the program specified in the parameter passed to `exec()` will replace the entire process and all its threads.

Best practice is, if `exec()` is called immediately after `fork()`, it's best to just duplicate that one process. However, if the separate process never calls `exec()`, it's best to duplicate the entire process, including its threads.

Signals, whether received synchronously or asynchronously, follow the same pattern:
1. A signal is generated by the occurrence of a particular event.
2. The signal is delivered to a process.
3. Once delivered, the signal must be handled.

Synchronous signals are delivered to the same process the performed the operation causing the signal. Example: illegal memory access and division by 0. Asynchronous signals are signals generated by an event external to a running process. Examples would be terminating a process with CTRL + C or having a timer expire.

A signal may be *handled* by one of two possible handlers:
1. A default signal handler
2. A user-defined signal handler

Handling signals in sequential, single-thread programs is pretty simple. How do we handle signals in multithreaded processes though? where should the signals be delivered? A couple of options are available:
1. Deliver the signal to the thread to which the signal applies.
2. Deliver the signal to every thread in the process.
3. Deliver the signal to certain threads in the process.
4. Assign a specific thread to receive all signals for the process.

The standard method in UNIX for delivering a signal is:

`kill(pid_t pid, int signal)`

Most multithreaded versions of UNIX allow threads to determine which threads they will handled and which they will block. Because signals need to be handled only once, usually a signal is handled by the first thread found that isn't blocking it. POSIX Pthreads implementation provides an API call that allows a signal to be delivered to a specified thread:

`pthread_kill(pthread_t tid, int signal)`

If multiple threads are searching through a database and one thread returns the result before the rest of the threads, the remaining threads need to be cancelled (*thread cancellation*). Cancellation of target threads may occur as such:
1. Asynchronous cancellation - one thread immediately terminates the target thread
2. Deferred cancellation - the target thread periodically checks whether it should terminate, allowing it an opportunity to terminate itself in an orderly fashion.

Asynchronous cancellation is dangerous because the thread could be killed before freeing resources. Pthreads thread cancellation is initiated using the `pthread_cancel()` function. A target thread will then cancel in a deferred state once it reaches a blocking call like `recv()`. `pthread_testcancel()` is also available for threads to use to see if they have been signalled to cancel. Pthreads also provides a `cleanup handler` function that allows a thread to cleanup resources it has allocated before cancelling.

Thread-local storage allows threads the ability to save data across function invocations. It's different from local storage as it will remain static as the thread continues to execute during it's lifetime.

To the user-thread, the lightweight process is a virtual processor on which the application can schedule a user thread to be run. Each LWP is attached to a kernel thread, and the kernel threads are what get scheduled by the operating system to run on the processor. If a kernel thread blocks on some I/O or event, the LWP blocks and then the user-level thread attached to the LWP blocks.

One scheme for communication between user and kernel threads is *scheduler activation*. The kernel provides an application with a set of virtual processors (LWPs) and the application can schedule user threads onto an available virtual processor. The kernel will notify the application of certain events by using upcalls. Upcalls are handled by the thread library and must run on a LWP.

## Definitions
* **Thread** - a basic unit of CPU utilization; includes a thread ID, program counter, a register set, and a stack. It shares with other threads of the same process its code section, data section, and other resources such as open files and signals.
* **Multicore** - systems in which multiple computing cores are placed on a single processing chip and each one appears as a separate CPU to the operating system.
* * **Concurrency** - the ability to support more than one task by allowing all the tasks to make progress.
* **Parallelism** - the ability to perform more than one task simultaneously.
* **Data parallelism** - focuses on distributing subsets of data across multiple cores and performing the same operation on each core.
* **Task parallelism** - focuses on distributing tasks across multiple computing cores, with each thread performing a unique operation either on the same data or different data
* **User threads** - threads supported above the kernel and are managed with kernel support
* **Kernel threads** - threads supported and managed directly by the operating system
* **Two-level model** - a variation of the many-to-many model that also allows user level threads to be bound to a specific kernel thread.
* **Thread library** - provides the programmer with an API for creating and managing threads.
* **Implicit threading** - the concept of transferring the creation and management of threading from application developers to compilers and run-time libraries.
* **Thread pool** - a pool of *N* number of threads created at the beginning of a process.
* **Signal** - used in UNIX systems to notify a process that a particular event has occurred.
* **Default signal handler** - kernel defined signal handler
* **User-defined signal handler** - signal handler defined by the user that overrides kernel defined handlers
* **Asynchronous procedure calls** - Windows method for emulating UNIX signals
* **Thread cancellation** - involves terminating a thread before it has completed
* **Target thread** - a thread that is to be canceled
* **Thread-local storage** - how threads keep their own copy of certain data
* **Lightweight process** - intermediate data structure between the user and the kernel threads usually implemented for the many-to-many or the two-level model
* **Upcall** - the method in which the kernel can notify a process of an event with using LWPs

## Practical Data
* Most operating system kernels are also typically multithreaded. During system boot time on Linux, several kernel threads are created with each performing a specific task such as: managing devices, memory management, or interrupt handling. `kthreadd` (with `pid == 2`) serves as the parent of all the kernel threads.
* * **Amdahl's Law** - a formula that identifies potential performance gains from adding additional computing cores to an application that has both serial (nonparallel) and parallel components.
* **Green threads** - a thread library available for Solaris systems and adopted in early versions of Java - used the many-to-one model, however, very few systems continue to use the model because of its inability to take advantage of multicore processing.
* Linux and Windows implement the one-to-one threading model.
* The three main thread libraries in use today are `POSIX` Pthreads, Windows, and Java. Pthreads, the threads extension of the `POSIX` standard, may be provided as either a user level or kernel level library. The Windows thread library is a kernel level library provided on Windows systems. The Java thread API allows threads to be created and managed directly in Java programs. Because Java runs in the Java Runtime Environment `JVM`, Java threads utilize the thread library available on the host operating system. This means that if Java is running on Windows it's using kernel level Windows threads and if it's running on anything UNIX, Linux, macOS, etc., it's using Pthreads. For POSIX and Windows threading, anything declared globally is shared across all threads. Java has no equivalent notion of global data, access to shared data must be explicitly arranged between threads.
* Windows and Java thread programming APIs both have builtin thread pool functions, allowing a programmer to create a pool of threads easily and allowing a programmer to queue work of a pool of threads.
* **OpenMP** is a set of compiler directives as well as an API for programs written in C, C++, or FORTRAN that provide support for parallel programming in shared-memory environments. **OpenMP** identifies **parallel regions** as blocks of code that may run in parallel.
* **Grand Central Dispatch** - a technology developed by Apple for its macOS operating system, it is a combination of run-time library, an API, and language extensions that allow developers to identify sections of code to run in parallel.
* **Intel Thread Building Blocks** - Intel provides a template library that supports designing parallel applications in C++. Developers specify tasks that can run in parallel, and the TBB scheduler maps these tasks onto underlying threads.