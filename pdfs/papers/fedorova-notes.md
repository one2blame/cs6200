# Chip Multithreading Systems Need a New Operating System Scheduler

## Summary

At the time of this paper being written, computer scientists were having difficulty ensuring the processor pipeline was utilized as much as possible by modern workloads. This is due to the number of frequent branches of execution and control transfers modern workloads conducted. Chip multithreading (CMT) is a processor architecture combining chip multiprocessing and hardware multithreading to address the issue of low processor pipeline utilization. This paper covers a proposal for a new operating system scheduler designed to make use of the new technology provided by CMT.

## Introduction

Transaction-processing-like workloads, workloads generated for application servers, web services, on-line transaction processing systems, etc., usually consist of multiple threads of control that execute short stretches of operations with frequent dynamic branches. Cache locality and branch prediction are negatively affected by these types of workloads and frequently cause the processor to stall.

Chip multiprocessing (CMP) and hardware multithreading (MT) techniques are designed to improve processor utilization for these types of workloads. CMP and MT offer better support for thread-level parallelism (TLP) rather than the instruction-level parallelism (ILP) offered by superscalar processors that are used for more scientific workloads.

A CMP processor includes multiple processor cores on a single chip, allowing for more than one thread to be active at a time.

A MT processor has multiple sets of registers and other thread state, interleaving execution of instructions from different threads, either by switching between threads or by executing instructions from multiple threads simultaneously.

In this paper, the authors propose that a specialized scheduler for CMT systems can provide a performance gain as large as a factor of two. They state that schedulers designed for single-processor multithreaded systems do not scale for the proposed architecture of CMT processors. CMT systems will require a fundamentally new design for the operating system scheduler.

Ideal schedulers maximize system throughput and minimize resource contention. Resource contention ultimately determines performance, a scheduler designed for CMT systems must understand how its scheduling decisions will affect resource contention.

## Experimental Platform

The authors created a model multithreaded processor core for their experiments. The MT core has multiple hardware contexts. These hardware contexts allow the processor to interleave the execution of multiple threads, switching between their contexts on each clock cycle. When a thread blocks, if another thread is available it switches to that thread's context. For multithreaded workloads, this hides the latency of long operations.

## Studying Resource Contention

On a CMT system, each hardware context appears as a logical processor to the operating system. Software threads are assigned to a hardware context for the duration of their scheduling time slice.

When assigned threads to hardware contexts, the scheduler has to decide which threads should be run on the same processor, and which threads should be run on separate processors. The scheduler should aim to achieve the optimal thread assignment in which the workload produces high utilization of the processor.

Pipeline utilization and contention depends upon the latencies of the instructions that the workload executes. Threads running workloads dominated by instructions with long delay latencies have little resource contention, but a lot of the resources are going unused. Threads running workloads with short delay latencies will be consistently using resources and the pipeline, possibly starving longer delay latency threads.

The experiments in this paper signify that the instruction mix, namely the average instruction delay latency, can be used as a heuristic for approximating the processor pipeline requirements for a workload. A scheduler can use instruction delay latency to determine scheduling decisions for a specific workload. It makes sense that, if a device has a long instruction delay latency, you should schedule instruction mixes that have a mix of both long and short instruction delay latencies. This will allow higher resource usage, as threads waiting on memory bound action to take place can be context switched for threads immediately ready to conduct operations.

This doesn't take into account cache contention. If a thread requires a large portion of the cache, we lose performance for memory-bound instruction mixes because, more often than not, the cache will have to be refreshed upon context switch.

## Summary

After the experiments shown in this paper, we can determine that CPI works well to determine scheduling for simple workloads. A load-balanced CPI across cores allows for multithreading with those cores to utilize all available resources for computation. Unfortunately, CPI does not provide information about a workload's resource requirements for things such as the ALU, cache, etc. - all of these results are based upon the resource usage of the processor pipeline.

## Definitions

* **hardware context** - consists of a set of registers and other thread state
* **instruction delay latency** - when a thread performs a long-latency operation and becomes "blocked", subsequent instructions to be issued by that thread are _delayed_ until the long-latency operation completes.
* **cycles-per-instruction** - captures average instruction delay and serves as an approximation for making scheduling decisions.