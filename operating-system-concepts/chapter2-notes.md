# Chapter 2

## Types of System Calls

* **Process Control**

  * create process, terminate process
  * load, execute
  * get process attributes, set process attributes
  * wait event, signal event
  * allocate and free memory

* **File Management**

  * create file, delete file
  * open, close
  * read, write, reposition
  * get file attributes, set file attributes

* **Device Management**

  * request device, release device
  * read, write, reposition
  * get device attributes, set device attributes
  * locgically attach or detach devices

* **Information Maintenance**

  * get time or date, set time or date
  * get system data, set system data
  * get process, file, or device attributes
  * set process, file, or device attributes

* **Communications**

  * create, delete communication connection
  * send, receive messages
  * transfer status information
  * attach or detach remote devices

* **Protection**

  * get file permissions
  * set file permissions

## Definitions

* **Run-time environment** - the full suite of software needed to execute applications written in a given programming language, including its compilers or interpreters as well as other software, such as libraries and loaders.

* **System-call interface** - serves as the link to system calls made available by the operating system. This interface intercepts function calls in the API and invokes the necessary system calls within the operating system.

* **Relocatable object file** - source files are compiled into object files that are designed to be loaded into any physical memory location.

* **Linker** - combines relocatable object files into a single binary executable

## Programming Projects

**Lessons learned from the File Copy exercise.**

* For portability, always use the C99 standard. To compile to the C99 standard, use this gcc option: `--std=c99`.
* For full error checking, use this gcc option: `-Wall`
* To compare a variable to a string literal, use this function: `strncmp()`
* To find a library to use in your project that conforms to C99 standards, view the library's `man` page and **goto** the 'CONFORMING TO' section.

I wrote the File Copy executable with `open()`, `read()`, and `write()` which aren't C99 standard functions - these functions are contained in the `<fcntl.h>` library.

The C99 standard library functions for opening files, reading, and closing can be found in `<stdio.h>`.

**Lessons learned from the Kernel Modules exercise.**

* `lsmod` lists all of your currently loaded kernel modules.
* `sudo insmod kernel_mod.ko` and `sudo rmmod kernel_mod.ko` insert kernel modules into the Linux kernel, and remove kernel modules from the Linux kernel, respectively.
* `dmesg` is `stdout` for the kernel, essentially. All kernel messages have a priority level when you use `printk` to print to dmesg. These priority levels are similar to SNMP traps.
* `HZ`, located in the `<asm/param.h>` library, contains the **tick rate** that establishes the frequency of the timer interrupt of the operating system.
* `jiffies`, located in the `<linux/jiffies.h>` library, keeps track of the number of timer interrupts that have occured since the system was booted.

This exercise was pretty cool. Got to mess around with kernel modules for the first time, use the kernel to build custom kernel modules and created processes in the `/proc/` folder.
