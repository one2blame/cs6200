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

The entire project is written in C. In this project I learned how to
efficiently use the
[pthreads](https://man7.org/linux/man-pages/man7/pthreads.7.html)
library for multithreaded programming, how to avoid race conditions, effective
signal handling, multithreaded and thread-safe file access conventions,
network programming, process memory management, how to use stack and queue
data structures, and how to use Valgrind and
[ASan](https://github.com/google/sanitizers/wiki/AddressSanitizer) to hunt
down memory leaks and programming errors.
