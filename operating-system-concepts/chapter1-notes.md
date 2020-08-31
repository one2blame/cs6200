# Chapter 1

## Introduction

An introduction to what operating systems are, what they do, and the various parts that go into creating a computing system and operating system.

## Definitions

* **Operating system** - is software that manages a computer's hardware, provides a basis for application programs and acts as an intermediary between the computer use and the computer hardware.

* **Resource allocator** - operating systems can be viewed as these, managing resources such as CPU time, memory space, storage space, I/O devices, etc.

* **Control program** - manages the execution of user programs to prevent errors and improper use of the computer. It is especially concerned with the operation and control of I/O devices.

* **Kernel** - on program running at all times on the computer, the operating system itself.

* **System programs** - programs associated with the operating system, but not part of the kernel.

* **Device driver** - provides the operating system with a uniform interface to a device controller.

* **Interrupt** - a way for a device driver to asynchronously notify the CPU of a change in hardware status.

* **Interrupt vector** - a table of pointers stored low in memory that holds the addresses of the interrupt service routines for various devices (array).

* **Trap (or exception)** - software generated interrupt caused by either an error (e.g. division by zero or invalid memory access) or by a specific request from a user program that an operating-system service be performed.

* **System call** - calls User-Mode programs can request when a Kernel-Mode operation needs to be executed.

* **Mode bit** - bit added to the hardware for the computer to indiciate the current privilege mode: kernel (0) or user (0).
