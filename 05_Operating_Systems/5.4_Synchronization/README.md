# 5.4 Synchronization & Mutual Exclusion

## Critical Section Problem
- **Race Condition**
  - Concurrent access issues
  - Non-deterministic results
  
- **Mutual Exclusion**
  - Only one process in critical section
  - Necessity and requirements
  
- **Atomicity**
  - Indivisible operations
  - Atomic instructions

## Synchronization Primitives

### Mutex (Mutual Exclusion Lock)
- **Basic Operations**
  - Lock operation
  - Unlock operation
  
- **Implementation Types**
  - Spinlock (busy waiting)
  - Sleep lock (blocking)
  
- **Properties**
  - Ownership
  - Non-recursive vs Recursive

### Semaphore
- **Binary Semaphore**
  - 0 or 1 value
  - Similar to mutex
  
- **Counting Semaphore**
  - Integer value
  - Resource counting
  
- **Operations**
  - Wait (P operation, down)
  - Signal (V operation, up)
  
- **Use Cases**
  - Resource pooling
  - Signaling between processes

### Condition Variable
- **Operations**
  - wait()
  - signal()
  - broadcast()
  
- **Usage Pattern**
  - Used with mutex
  - Event notification

## Advanced Synchronization

### Barrier
- **Synchronization Point**
  - All threads meet
  - Coordinated execution

### Read-Write Lock
- **Multiple Readers**
  - Concurrent read access
  
- **Single Writer**
  - Exclusive write access
  
- **Priority**
  - Reader priority
  - Writer priority

### Low-Level Primitives
- **Atomic Operations**
  - Compare-and-Swap (CAS)
  - Test-and-Set
  - Fetch-and-Add
  
- **Memory Fence**
  - Memory ordering
  - Visibility guarantees

## Classic Problems
- [ ] Producer-Consumer
- [ ] Readers-Writers
- [ ] Dining Philosophers

## Study Materials
- [ ] Synchronization examples
- [ ] Lock implementation code
- [ ] Semaphore usage patterns

## Practice Problems
- [ ] Implement synchronization solutions
- [ ] Identify race conditions
- [ ] Fix concurrent bugs

## Expert Depth Checklist
- [ ] Trace the kernel/user boundary for the topic: syscall, trap, interrupt, scheduler, page fault, VFS, or driver path.
- [ ] Explain the invariant the OS maintains and what breaks if it is violated.
- [ ] Reproduce behavior with a small C program, shell experiment, `strace`, `perf`, `/proc`, or debugger output.
- [ ] Analyze concurrency and failure: race, deadlock, starvation, priority inversion, lost wakeup, or crash consistency bug.
- [ ] Compare policy vs mechanism and explain which part is configurable.
- [ ] Read at least one authoritative source: OSTEP, Linux man pages, kernel docs, or OS textbook chapter.
