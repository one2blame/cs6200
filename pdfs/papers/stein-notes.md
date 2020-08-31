# Implementing Lightweight Threads

## Introduction

This paper by SunSoft describes the implementation of a lightweight threads library, the very lightweight process (LWP) concept used to multiplex user-level threads and kernel threads, preventing the user level processes from utilizing too many kernel resources.

## Threads Library Architecture

Threads are the programmer's interface for multi-threading, implemented by a library that utilizes underlying kernel-supported threads of control called lightweight processes (LWP). Threads are scheduled on a pool of LWPs, and the kernel schedules LWPs on a pool of processors. The library can permanently bind a thread to an LWP if that thread needs constant visibility by the CPU (for example a mouse interface thread). LWPs allow programmers using threads to not have to worry about how kernel resources are being utilized.

## LWP interfaces

LWPs are like threads, sharing most of the process's recourses. Each LWP has its own set of private registers and a signal mask. LWPs also have attributes that are unavailable to threads like kernel-supported scheduling, virtual time timers, an alternate signal stack, and a profiling buffer.

## Threads Library Implementation

Pretty standard here. Threads have a structure with a thread ID, an area to save the execution context, the thread signal mask, the thread priority, and a pointer to the thread stack.

## Thread-local Storage

Threads have private storage in addition to their stack called thread-local storage (TLS). TLS is unshared, statically allocated data - used for thread-private data that must be accessed quickly. After program startup, the size of TLS is fixed and can no longer grow, restricting programmatic dynamic linking to libraries that don't utilize TLS.

## Thread Scheduling

The threads library implements a thread scheduler that multiplexes thread execution across a pool of LWPs. The LWPs are nearly identical, allowing any thread to execute on any of the LWPs in the pool. When a thread executes, it attaches to an LWP and has all the attributes of being a kernel-supported thread.

### Thread States and the Two Level Model

An unbound thread can be in one of five different states:
1. RUNNABLE
2. ACTIVE - the LWP serving the thread is running, sleeping in the kernel, stopped, or in the kernel dispatch queue waiting on a processor
3. SLEEPING
4. STOPPED
5. ZOMBIE

### Idling and Parking

When an unbound thread exits and there are no more RUNNABLE threads, the LWP that was RUNNING the ACTIVE thread switches to an *idle* stack associated with each LWP and waits for a global LWP condition variable to signal the availability of new RUNNABLE threads.

When a bound thread blocks on a process-local synchronization variable, the underlying LWP stops running and waits on a semaphore associated with the thread. This state for an LWP is called *parked*. When the bound thread unblocks, the semaphore is signaled and he LWP continues executing the thread. The same goes for an unbound thread attached to a LWP if there are no more RUNNABLE threads available.

### Preemption

Threads compete for LWPs just as kernel threads compete for CPUs. If there is a possibility that a higher priority RUNNABLE thread exists, the ACTIVE queue is searched to find a lower priority thread. The lower priority ACTIVE thread is preempted, and the LWP schedules the higher priority RUNNABLE thread.

There are two cases when the need to preempt arises:
1. When a RUNNABLE thread with a higher priority of the lowest priority ACTIVE thread is created
2. When the priority of an ACTIVE thread is lowered below that of the highest priority RUNNABLE thread

One side-effect of preemption is that if the target thread is blocked in the kernel executing a system call, it will be interrupted by SIGLWP, and then the system call will be re-started when the thread resumes execution after the preemption. This is only a problem if the system call should not be re-started.

### The Size of the LWP Pool

The threads library automatically adjusts the number of LWPs in the pool of LWPs that are used to run unbound threads. There are two requirements in setting the size of the LWP pool:
1. The pool must not allow the program to deadlock due to lack of LWPs
2. The library should make efficient use of LWPs

The current threads library for SunOS starts by guaranteeing that the application does not deadlock. It does so by using a SIGWAITING signal that is sent by the kernel when all the LWPs in the process block in indefinite waits. Another implementation might attempt to compute a weighted time average of the application's concurrency requirements and adjust the pool of LWPs more aggressively. The library also provides an interface for programmers to set concurrency based upon what the application should expect. If the LWP pool grows greater than the threads in the process, LWPs will age off if gone unused.

## Mixed Scope Scheduling

Bound and unbound threads live in harmony as unbound threads continue to be scheduled in a multiplexed fashion while bound threads just maintain their priority and connection to their assigned LWP.

### Reaping Bound/Unbound Threads

When a *detached* thread exits, it is place on a single queue called `deathrow` and the state is set to `ZOMBIE`. The action of free'ing a thread's stack is not done at exit time - this is unnecessary and expensive work. The threads library has a special thread called the reaper who frees the threads' memory periodically. The reaper runs when there are idle LWPs or when the `deathrow` gets full.

When an *undetached* thread exits, it's added to the zombie list of its parent thread. The parent thread will reap the zombie threads executing `thr_join()`.

The reaper runs at high priority and should be run not too frequently, but not too sparsely either as reaping has an impact on how quickly threads are created. The reaper puts the freed stacks on a cache of available stacks, speeding up stack allocation for new threads.

## Thread Synchronization

The thread library implements two types of synchronization variables, process-local and process-shared.

### Process-local Synchronization Variables

The default blocking behavior is to put the thread to sleep, and place the thread on the sleep queue of the synchronization variable. The blocked thread is then surrendered to the scheduler, and the scheduler dispatches another thread to the LWP. Bound threads will stay permanently to the LWP, and the LWP will become *parked*, not *idle*.

Blocked threads wake up when the synchronization variable they were waiting on becomes available. The synchronization primitives check if threads are waiting on the synchronization variable, then a blocked thread is moved from the synchronization variable's sleep queue and is dispatched by the scheduler to an LWP. For bound threads, the scheduler unparks the thread and the LWP is dispatched by the kernel.

### Process-shared Synchronization Variables

Process-shared synchronization objects can also be used to synchronize threads across different processes. These objects have to be initialized when they are created because their blocking behavior is different from the default. The initialization function must be called to mark the object as process-shared - the primitives will then recognize the synchronization variables are shared and provide the correct blocking behavior. The primitives rely on LWP synchronization primitives to put the blocking threads to sleep in the kernel still attached to their LWPs, and to correctly synchronize between processes.

## Signals

A challenge is presented for signals because the kernel can send signals, but thread masks are invisible to the kernel, so signal delivery is dependent upon the thread signal mask making it hard to elicit the correct program behavior.

The thread implementation also has to support asynchronous safe synchronization primitives. For example, if a thread calls `mutex_lock()` and then is interrupted, what if the interrupt handler also tried to call `mutex_lock()` and enters a deadlock? One way to make this safe is to signal mask while in a thread's critical section.

### Signal Model Implementation

A possibility is to have the LWP replicate the thread's signal mask to make it visible to the kernel. The signals that a process can receive changes based upon the threads in the application that cycle through the ACTIVE state.

A problem arises when you have threads that aren't ACTIVE often, but are the only threads in the application that have specific signals enabled. These threads are asleep waiting on signals they'll never receive. In addition, anytime the LWP switches threads with different masks or the thread adjusts its mask, the LWP has to make a system call to notify the kernel.

To solve this problem, we define the set of signals a process can receive equal to the intersection of all the thread signal masks. The LWP signal mask is either less restrictive or equal to the thread's signal mask. Occasionally, signals will be sent by the kernel to ACTIVE threads that have the signal disabled.

When this occurs, the threads library prevents the thread from being interrupted by interposing its own signal handler below the application's signal handler. When a signal is received, the global handler checks the current thread's signal mask to determine if the thread can receive the signal.

If the signal is masked, the global handler sets the LWP's signal mask to the thread's signal mask and resend the signal either to the process (for undirected signals) or to the LWP (for directed signals). If the signal is not masked, the global handler calls the signal's application handler. If the signal is not applicable to any of the ACTIVE threads, the global handler will wakeup one of the inactive threads to run if it has the signal unmasked.

This is all for asynchronous signals. Synchronously generated signals are simply delivered by the kernel to the ACTIVE thread that caused them.

### Sending a Directed Signal

Threads can send other threads signals using `thr_kill()`, however, if the thread is currently not running on the LWP, the signal will remain pending on the thread in a pending signals mask. When the target thread resumes execution, it receives the pending signals. If the thread is ACTIVE, `thr_kill()` is sent to the target thread's LWP. If the LWP is blocking the signal, the signal stays pending on the LWP until the thread unblocks it. While the signals is pending on the LWP, the thread is temporarily bound to the LWP until the signals are delivered to the thread.

### Signal Safe Critical Sections

To prevent deadlock in the presence of signals, critical sections that are reentered in a signal handler in both multi-threaded applications and the threads library should be safe with respect to signals. All async signals should be masked during critical sections.

To make critical sections as safe as efficiently possible, the threads library implements `thr_sigsetmask()`. If signals do not occur, `thr_sigsetmask()` makes no system calls making it just as fast as modifying the user-level thread signal mask.

The threads library sets/clears a special flag in the threads structure whenever it enter/exits an internal critical section. Effectively, this flag serves as a signal mask to mask out all signals.