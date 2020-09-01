# cs6200

This class is a graduate-level introductory course in operating systems. I
already took an operating systems course during my undergraduate studies,
however, I felt like this would be a good refresher and first class in GATech's
OMSCS program.

[Here](https://omscs.gatech.edu/cs-6200-introduction-operating-systems) is the
official course webpage.

## Books

In the folder `./operating-system-concepts`, you'll find some notes that I took
while reading the textbook, "Operating System Concepts 10th edition", by
Abraham Silberschatz et. al.

This book is a great read and the course material closely follows some of the
chapters in the book. I highly recommend reading this book if you want to gain
a thorough understanding of how operating systems are designed and implemented.

## Pdfs

In the folder `./pdfs/papers`, you'll find a majority of the papers covered in
the course - I don't know if I kept all of them, though. This folder also
contains some of my notes for each paper, essentially summarizing the most
important points.

`./pdfs/cheatsheets` contains a couple of useful cheatsheets for Docker
commands, Linux system calls, and programming in C.

`./pdfs/cs6200.pdf` contains my entire notebook for the course.

## Projects

I worked on each project for this course in a separate repository. Each project
is added to this repository as a submodule. The submodules in this repository
are private to uphold the Georgia Institute of Technology
[Academic Honor Code](https://osi.gatech.edu/content/honor-code).

### Project 1

This project is an exercise in designing and implementing multithreaded
applications. The student is tasked with creating a multithreaded web server
that will serve static files based on a custom protocol. The student must also
create a multithread client application that will use the same custom protocol
to generate requests for files from the web server.

The entire project is written in C. In this project I learned:

* How to efficiently use the [pthreads](https://man7.org/linux/man-pages/man7/pthreads.7.html)
library for multithreaded programming
* How to avoid race conditions
* Effective signal handling
* Multithreaded and thread-safe file access conventions
* Network programming
* Process memory management
* How to use stack and queue data structures
* How to use Valgrind and [ASan](https://github.com/google/sanitizers/wiki/AddressSanitizer) to hunt
down memory leaks and programming errors

### Project 2

Project 2 was an extra credit project that was only graded if a student needed
a couple more points to increase their letter grade. This project required
students to evaluate the performance of their implemetation of Project 1, and
how the project could be designed differently in order to increase
performance.

### Project 3

This project is an exercise in designing and implementing applications that
leverage inter-process communication (IPC). This project builds upon Project 1.
The multithreaded web server previously designed will now work in tandem with a
web proxy. When the multithreaded client requests a file, the web server will
check with the web proxy to see if the requested file exists locally within the
cache. If not, the web proxy will use the [curl](https://curl.haxx.se/) library
to download the file from the internet and then store the contents of the file
locally. The web proxy will then transfer the file to the web server using the
[POSIX shared memory API](https://www.man7.org/linux/man-pages/man7/shm_overview.7.html).
If the file exists locally, the web proxy will not make an HTTP request,
transferring the file to the web server via shared memory. The client and the
web server still use a custom protocol to conduct communication and file
exchanges.

The entire project is written in C. In this project I learned:

* How to use libcurl
* How to create, manage, and use [POSIX message queues](https://www.man7.org/linux/man-pages/man7/mq_overview.7.html)
for IPC
* How to use the POSIX shared memory API for IPC.

### Project 4

This project is an exercise in designing and implementing applications that
leverage remote procedure calls and protocol buffers. In this project, students
design a distributed file system using [gRPC](https://grpc.io/) and Google's
[Protocol Buffers](https://developers.google.com/protocol-buffers). These tools
will allow students to implement several file transfer protocols and a
synchronization system so that multiple clients and one server have consistent
files across different file caches.

The entire project is written in C++. In this project I learned how to:

* Use the protobuf library,
* How to handle asynchronous gRPC calls
* How to use [inotify](https://man7.org/linux/man-pages/man7/inotify.7.html) to monitor
filesystem events
* How to design and implement object-oriented applications written in C++
