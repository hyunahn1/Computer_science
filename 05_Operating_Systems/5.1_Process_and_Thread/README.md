# 5.1 Process & Thread

## Process Concepts
- **Process ID (PID)**
  - Process identification
  - Process management
  
- **Process State**
  - Running
  - Ready
  - Blocked
  - Zombie
  
- **Process Context**
  - Registers
  - Memory
  - Files

## Thread Concepts
- **Thread Fundamentals**
  - Thread definition
  - Thread vs Process differences
  - Advantages of threads
  
- **Thread Types**
  - User-level Thread
  - Kernel-level Thread
  - Hybrid models

## Process/Thread Creation
- **Linux**
  - `fork()` system call
  - Process cloning
  - Copy-on-Write (COW)
  
- **Windows**
  - `CreateProcess()`
  - Process creation API
  
- **POSIX Threads**
  - `pthread_create()`
  - Thread attributes

## Process/Thread Termination
- **Normal Termination**
  - `exit()` system call
  - `return` from main
  
- **Zombie Process**
  - Definition and causes
  - Prevention methods
  
- **Process Cleanup**
  - `wait()` system call
  - `waitpid()` system call
  - Reaping child processes

## Study Materials
- [ ] Process lifecycle diagram
- [ ] Thread implementation examples
- [ ] fork() practice code
- [ ] pthread examples

## Practice Problems
- [ ] Implement multi-threaded application
- [ ] Process state transitions
- [ ] Zombie process handling

## Expert Depth Checklist
- [ ] Trace the kernel/user boundary for the topic: syscall, trap, interrupt, scheduler, page fault, VFS, or driver path.
- [ ] Explain the invariant the OS maintains and what breaks if it is violated.
- [ ] Reproduce behavior with a small C program, shell experiment, `strace`, `perf`, `/proc`, or debugger output.
- [ ] Analyze concurrency and failure: race, deadlock, starvation, priority inversion, lost wakeup, or crash consistency bug.
- [ ] Compare policy vs mechanism and explain which part is configurable.
- [ ] Read at least one authoritative source: OSTEP, Linux man pages, kernel docs, or OS textbook chapter.
