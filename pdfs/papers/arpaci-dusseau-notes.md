# I/O Devices

## Canonical Device

A canonical device is made of these components:

* Interface
    * Registers
      * Status
      * Command
      * Data

* Internals
  * Micro-controller (CPU)
  * Memory (DRAM or SRAM or both)
  * Other Hardware-specific chips

## Canonical Protocol

Programmed I/O (PIO) is wasteful of CPU cycles because the CPU spends so much time polling and waiting on the device instead of switching to another ready process.

Using interrupts can be costly, however, due to the number of context switches that could take place. If a device is fast, it's better just to poll on its status rather than interrupt because context switching so often due to interrupts will cause significant overhead.

## More Efficient Data Movement with DMA

Direct Memory Access (DMA) engines allow a CPU to write instructions for the data transfer to the device. The operating system provides a DMA engine the location of the data, how much data to copy, and the device the data is destined for.

The DMA engine will complete the operation and then raise an interrupt to notify the operating system that the data transfer was completed.

The DMA provides the CPU the ability to continue doing other meaningful work rather than conducting the entire data transfer for a device. The setup is slower than PIO, however, so is best used for large data transfers. PIO should still be used for small data transfers.

## Definitions

* **hardware interface** - interface for hardware that allows the system to control its operation. All devices have a hardware interface that has some specific protocol for typical interaction.
* **internal structure** - responsible for implementing the abstraction the device presents to the system.
* **firmware** - software within a hardware device
* **status register** - a register that a controller can read to see the status of the device
* **command register** - a register that can be written to to instruct the device to perform a certain task
* **data register** - a register to pass or retrieve data from the device
* **polling** - the operating system continually checks the status register of a device until it is ready to receive a command
* **programmed I/O (PIO)** - when the main CPU is involved with the interaction and data movement of a device
* **interrupt** - signals that can be sent to the CPU when an I/O operation is complete
* **interrupt service routine** - also called an interrupt handler, this is a piece of operating system code that will handle an interrupt received from an I/O device
* **livelock** - when an operating system finds itself only processing interrupts
* **coalescing** - a device waits a bit before delivering an interrupt to the CPU - this causes latency
* **direct memory access (DMA)** - a device within a system that can orchestrate transfers between devices and main memory without CPU intervention
* **I/O instructions** - used by IBM mainframes, these instructions specify a way for the operating system to send data to specific device registers
* **memory mapped I/O** - the hardware makes device registers available as if they were memory locations - the operating system reads or writes to the address in main memory and the hardware routes those operations to the device
* **device driver** - a piece of software in the operating system that knows, in detail, how a device works.
* **raw interface** - allows devices to directly read and write blocks to a device without file abstraction