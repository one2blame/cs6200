# Flash: An Efficient and Portable Web Server

## Introduction

This paper is about Flash, an asymmetric multi-process event-driven (AMPED) Web server architecture. Web servers take different approaches to achieve concurrency such as:
* single-process event driven (SPED) architecture
* multi-process (MP) or multi-threaded (MT) architecture

SPED is good for caching requests and quickly serving those cached requests, whereas MP or MT is better for retrieving content from disk. AMPED attempts to match the performance of SPED on cached workloads, while also matching or exceeding the performance of MP or MT when conducting disk-intensive work.

## Background

Basic processing steps performed by a Web server:
1. Accept connection
2. Read request
3. Find file
4. Send header
5. Read file
6. Send data
7. End connection

**Static content** is stored on webservers in the form of disk files. **Dynamic content** is generated upon request by auxiliary application programs running on the server.

A high-performance Web server must interleave the sequential steps associated with the serving of multiple requests in order to overlap CPU processing with disk accesses and network communication. The servers *architecture* determines what strategy is used to achieve this interleaving.

## Server Architectures

### Multi-process

In this architecture, a process is assigned to execute the steps of serving content in a sequential manner. It performs all steps before accepting a new requests. Because typically 20-200 processes are running, many HTTP requests can be served concurrently. Each process has its own private address space, no synchronization is necessary to handle the processing of different HTTP requests. This makes it more difficult to perform optimizations in this architecture that relies on global information like a shared cache or a list of valid URLs.

### Multi-threaded

This architecture implements multiple threads of control operating within a single shared address space, with each thread performing all the steps associated with one HTTP request before accepting a new request. The difference from MP is that all threads share global variables. This allows more room for optimization, however, the threads have to synchronize and control their access to the shared data.

The MT model requires operating system support for kernel threads - when one thread blocks for an I/O operation, other runnable threads should be scheduled for execution.

### Single-process event driven

This architecture uses a single event-driven server process to perform concurrent processing of multiple HTTP requests. The server uses non-blocking system calls to perform asynchronous I/O operations.

The SPED architecture can be though of as a state machine that performs one basic step associated with the steps of serving an HTTP request at time, thus interleaving the processing steps associated with many HTTP requests. In each iteration, the server performs a `select` or `poll` to see if there is something to do for that step, and if so it completes the operation. Otherwise, the architecture continues to iterate.

This architecture avoids all of the context switching and thread synchronization of the MP and MT architectures.

### Asymmetric Multi-Process Event-Driven

This architecture combines the event-driven approach of the SPED architecture with *helper* processes (threads) that handle the block I/O operations. The main event-driven process handles all processing steps involved with HTTP requests and when a disk operation is required, the main thread instructs a helper process via inter-process communication (IPC) to perform the blocking operation. The helper notifies the main thread that the operation is complete via IPC; the main thread learns of this by using `select`.

## Design comparison

### Performance characteristics

In MP or MT, entire processes or threads are blocked for I/O, which stops the process of completing the HTTP current request. In SPED, the entire server is stopped because there is one thread of execution blocking on I/O. The main cost for AMPED is just the IPC cost, but the main thread will not block for I/O.

### Memory effects

Memory usage of the architectures, ranked from lowest to highest:
1. SPED
2. AMPED
3. MT
4. MP

### Disk utilization

The MP/MT models can cause one disk request per process/thread, while the AMPED model can generate one request per helper. The SPED model can only generate one disk request at a time, so it can not benefit from multiple disks or disk head scheduling.

## Cost/Benefits of optimizations & features

### Information gathering

Web servers gather information like usage statistics, requests, etc. For the MP model, a lot of synchronization and IPC is required to gather this information across multiple processes. MT requires control of global variables or the storage of information across each thread. SPED and AMPED models simplify information gathering since all requests are processed in a centralized fashion.

### Application-level Caching

In MP, each process may have its own cache to avoid IPC and sync, however, these multiple caches increase the number of misses and they lead to less efficient use of memory. MT uses a single cache, but all of this must be coordinated. AMPED and SPED can use a single cache without synchronization.

### Long-lived connections

If someone's taking a long time for a connection, due to the limitations of their physical link, AMPED and SPED are fine because their cost is a file descriptor. For MT and MP, the overhead of an extra thread or process is the cost imposed.
