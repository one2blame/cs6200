# The Performance of Spin Lock Alternatives for Shared-Memory Multiprocessors

## Introduction

This paper examines the question: are there efficient algorithms for software spin-waiting for busy locks given hardware support for atomic instructions, or are more complex kinds of hardware support needed for performance?

Spinning processors slow other processors doing useful work, including the ones holding the lock, because the processors are continuously using up all communication bandwidth in an attempt to check and acquire the lock.

This paper proposes establishing a queue in which processors requesting a spin lock receive a unique ID and when the lock is available, the next processor in the queue with the correct ID receives the lock. This paper also proposes an addition to snoopy cache protocols that exploits the semantics of spin lock requests to obtain better performance. Performance problems of spin-waiting, software alternatives, and hardware solutions are all discussed in this paper.

## Section II

Cache coherence can effect the performance of spin locks and the atomic instructions used to set the locks. Multistage networks connect multiple processors to multiple memory modules, with each memory modules having its own cache. When a value is read during an atomic instruction, all cached copies of that value across all memory modules must be invalidated. Subsequent accesses to that module will be delayed as the new value is calculated.

## Section III: The Performance of Simple Approaches to Spin-Waiting

**Spin on Test-and-Set**

Simplest method of acquiring lock, worst performance as number of processors spinning on the lock increases. In order to release the lock, the lock holder must contend with all the processors spinning on the lock. All the processors attempting to access the same lock can cause slow accesses to other memory locations if using the same bus, hot-spots delaying memory access to memory modules in a networked architecture, and the consuming of bus transactions, saturating the bus.

**Spin on Read (Test-and-Test-and-Set)**

This algorithm is a proposal to read the lock and if it's not BUSY, attempt to set the lock. As soon as the lock is released, all the of the caches are invalidated or written to with a new value. A waiting processor sees the change in state and performs a test-and-set - whoever gets to the lock first wins. When the critical section is small, however, things can get hairy as the remaining processors are still waiting on their caches to update. The lock is acquired by a processor, and all of the other processors still updating their caches will now attempt to test-and-set (because the lock used to be TRUE) and they will fail, thus causing the test-and-set instruction to hit the communication channel and cause congestion again.

**Test Results**

Test results show that, for both spin on test-and-set and spin on read then test-and-set, performance dramatically decreases when more processors are added and contend for a lock. As the number of spinning processors increase, the time for the caches to all quiesce also increases.

## Section IV: New Software Alternatives

They tested adding a delay where the delay starts at 0 and goes to P (number of processors). Like CSMA over ethernet, the processor attempts to acquire the lock. If contention occurs, it conducts a backoff and increases its delay until it reaches P. Eventually, each processor will be delaying at a different time, decreasing lock contention and allowing for the caches to quiesce better as well. Unfortunately, if there aren't enough delay spots, processors within the same delay will contend.

They added a delay after every read as well (for test and test-and-set), however, if a critical section is crazy long, the delayed processor could delay for a ridiculous amount of time because of all the reads. When the lock is eventually released, the processor will still have to complete the delay before acquiring the lock.

