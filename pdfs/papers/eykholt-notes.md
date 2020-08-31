# Beyond Multiprocessing Notes

## Introduction

Written in 1992, SunOS was looking for a way to have their kernel support multiprocessing utilizing threads. They wanted to achieve a high degree of concurrency and support more than one thread of control for a user process - each thread able to work independently.

They also wanted the kernel to be able to provide real-time response, providing support for preemption of almost any point in the kernel. The kernel SunOS built is a complex multi-threaded program. Threads can be leveraged by user applications to manage asynchronous activities - the kernel benefits from a similar thread facility.

Solaris 2.0 if fully preemptible, has real-time scheduling, symmetrically supports multiprocessors, and supports user-level multithreading.

## Overview of the Kernel Architecture

Kernel threads are units that are scheduled and dispatched onto one of the CPUs of the system. Lightweight, only having a small data structure and stack, kernel threads do not require a change of virtual memory address space information - this makes context switching of kernel threads inexpensive.

Kernel threads are fully preemptible and may be scheduled by an of the scheduling classes in the system, including the real time class. All execution entities are using kernel threads, representing a  fully preemptible, real time nucleus in the kernel. Kernel threads support synchronization primitives that support protocols for preventing priority inversion so that a thread's priority is determined by which activities it is impeding by holding locks as well as by the service it is performing.

User processes in Solaris 2.0 use lightweight processes (LWPs) that share the address space of the process and other resources such as open files. The kernel supports these LWPs by assigned a kernel thread to each LWPs. A user-level thread library uses LWPs to implement user-level threads - the user-level threads are scheduled by the library thus removing the requirement of the library having to enter the kernel to switch currently executing threads.

## Kernel Thread Scheduling

Solaris 2.0 uses priority based scheduling and, if multiple threads are of the same priority, the scheduler will scheduler the threads in a round-robin order. The kernel is preemptive, a runnable thread runs as soon as is practical after its priority becomes high enough.

## System Threads

Can be created for short or long-term activities, these belong to the system scheduling class. These threads have no need for a LWP structure, their thread structure and stack is allocated in a non-swappable area.

## Synchronization Architecture

The kernel implements similar synchronization objects as are provided to the user-level libraries. Mutex locks, condition variables, semaphores, etc. Upon the creation of these objects, it is possible to set options that enable statistics gathering. This can provide the writer a method of grabbing usage data and wait times.

## Mutual Exclusion Lock Implementation

If contention is met for a lock on Solaris 2.0, the blocking action taken depends on the mutex type. The default blocking policy has the thread or process spinning while the owner of the lock remains running on a processor. This is done by polling the owner's status in a wait loop.

## Summary

SunOS 5.0 is a multithreaded and symmetric multiprocessor kernel featuring:
* fully preemptible, real-time kernel
* high degree of concurrency on symmetric multiprocessors
* support for user threads
* interrupts handled as independent threads
* adaptive mutual-exclusion locks