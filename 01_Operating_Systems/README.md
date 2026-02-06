# Operating Systems

## Overview
Comprehensive study materials covering fundamental operating system concepts, from process management to Linux kernel internals.

## Course Structure

### [1.1 Process & Thread](./1.1_Process_and_Thread/)
- Process concepts (PID, State, Context)
- Thread fundamentals
- Process/Thread creation and termination
- fork(), pthread_create(), exit(), wait()

### [1.2 CPU Scheduling](./1.2_CPU_Scheduling/)
- CPU scheduler concepts
- Scheduling algorithms (FCFS, SJF, SRTF, Round Robin, Priority, MLFQ)
- Scheduling metrics (Turnaround, Waiting, Response Time)

### [1.3 Memory Management](./1.3_Memory_Management/)
- Memory hierarchy (Register, Cache, RAM, Disk)
- Virtual memory (Paging, Segmentation, TLB)
- Page replacement algorithms (FIFO, LRU, LFU, Optimal)
- Memory allocation strategies

### [1.4 Synchronization & Mutual Exclusion](./1.4_Synchronization/)
- Critical section and race conditions
- Mutex (Spinlock vs Sleep lock)
- Semaphore (Binary, Counting)
- Advanced synchronization (Barrier, RW Lock, Atomic ops)

### [1.5 Deadlock](./1.5_Deadlock/)
- Necessary conditions (4 conditions)
- Deadlock prevention, avoidance, detection
- Banker's algorithm
- Recovery strategies

### [1.6 File System](./1.6_File_System/)
- File system structure (Boot, Super Block, Inode)
- File allocation methods (Contiguous, Linked, Indexed)
- File caching (Write-Through vs Write-Back)

### [1.7 I/O & Interrupt](./1.7_IO_and_Interrupt/)
- I/O methods (Polling, Interrupt, DMA)
- Interrupt handling and priorities
- Interrupt vector table
- Device drivers

### [1.8 Linux Kernel Basics](./1.8_Linux_Kernel/)
- User space vs Kernel space
- System call interface
- Major subsystems (Scheduler, Memory Manager, VFS, Network Stack)

## Study Approach
1. Read through each topic's README for an overview
2. Take detailed notes in the topic's directory
3. Complete practice problems
4. Build projects to apply concepts

## Legacy Directories
- `notes/` - General study notes
- `resources/` - Books, papers, and reference materials
- `exercises/` - Cross-topic exercises
- `projects/` - Implementation projects
