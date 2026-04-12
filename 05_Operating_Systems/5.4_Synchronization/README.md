# 1.4 Synchronization & Mutual Exclusion

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
