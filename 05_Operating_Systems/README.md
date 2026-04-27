# Operating Systems

## Overview
Comprehensive study materials covering fundamental operating system concepts, from process management to Linux kernel internals.

## Course Structure

### [5.1 Process & Thread](./5.1_Process_and_Thread/)
- Process concepts (PID, State, Context)
- Thread fundamentals
- Process/Thread creation and termination
- fork(), pthread_create(), exit(), wait()
- **Inter-process communication (IPC)**: pipes, FIFOs, shared memory, message queues, signals (overview)

### [5.2 CPU Scheduling](./5.2_CPU_Scheduling/)
- CPU scheduler concepts
- Scheduling algorithms (FCFS, SJF, SRTF, Round Robin, Priority, MLFQ)
- Scheduling metrics (Turnaround, Waiting, Response Time)

### [5.3 Memory Management](./5.3_Memory_Management/)
- Memory hierarchy (Register, Cache, RAM, Disk)
- Virtual memory (Paging, Segmentation, TLB)
- Page replacement algorithms (FIFO, LRU, LFU, Optimal)
- Memory allocation strategies
- **Thrashing** and **working set** model (why RAM pressure hurts performance)

### [5.4 Synchronization & Mutual Exclusion](./5.4_Synchronization/)
- Critical section and race conditions
- Mutex (Spinlock vs Sleep lock)
- Semaphore (Binary, Counting)
- Advanced synchronization (Barrier, RW Lock, Atomic ops)
- **Memory visibility and ordering** (happens-before, barriers) — ties to concurrent programming interviews

### [5.5 Deadlock](./5.5_Deadlock/)
- Necessary conditions (4 conditions)
- Deadlock prevention, avoidance, detection
- Banker's algorithm
- Recovery strategies

### [5.6 File System](./5.6_File_System/)
- File system structure (Boot, Super Block, Inode)
- File allocation methods (Contiguous, Linked, Indexed)
- File caching (Write-Through vs Write-Back)

### [5.7 I/O & Interrupt](./5.7_IO_and_Interrupt/)
- I/O methods (Polling, Interrupt, DMA)
- Interrupt handling and priorities
- Interrupt vector table
- Device drivers

### [5.8 Linux Kernel Basics](./5.8_Linux_Kernel/)
- User space vs Kernel space
- System call interface
- Major subsystems (Scheduler, Memory Manager, VFS, Network Stack)
- **cgroups** and **namespaces** (Linux containers); hands-on Docker/Kubernetes → **[12.2 Docker & Kubernetes](../12_Cloud_and_Operations/12.2_Docker_and_Kubernetes/)**

## Study approach
Use the repo-wide [how-to](../README.md#how-to-use-this-repository). For this track, pair each chapter README with small runnable examples (`fork`, pthreads, syscall tracing) when possible.

## Advanced Topics to Add

- Process model: fork/exec, copy-on-write, signals, process groups, job control, namespaces.
- Scheduling: MLFQ, CFS intuition, priority inversion, real-time classes, scheduler latency.
- Memory: page tables, TLB shootdowns, mmap, page cache, NUMA, huge pages, allocator internals.
- Concurrency: futexes, condition variables, lost wakeups, RCU, lock ordering, memory barriers.
- Storage/I/O: journaling, crash consistency, epoll/io_uring awareness, block layer, driver interrupt path.
- Linux kernel: VFS, cgroups, capabilities, seccomp, eBPF observability, kernel tracing.

## Expert Depth Checklist
- [ ] Trace the kernel/user boundary for the topic: syscall, trap, interrupt, scheduler, page fault, VFS, or driver path.
- [ ] Explain the invariant the OS maintains and what breaks if it is violated.
- [ ] Reproduce behavior with a small C program, shell experiment, `strace`, `perf`, `/proc`, or debugger output.
- [ ] Analyze concurrency and failure: race, deadlock, starvation, priority inversion, lost wakeup, or crash consistency bug.
- [ ] Compare policy vs mechanism and explain which part is configurable.
- [ ] Read at least one authoritative source: OSTEP, Linux man pages, kernel docs, or OS textbook chapter.
