# 2.4 Real-Time Operating System (RTOS)

## RTOS Fundamentals

### RTOS vs GPOS
- **RTOS (Real-Time OS)**
  - Deterministic response time
  - Guaranteed timing
  - Predictable behavior
  - Smaller footprint
  
- **GPOS (General Purpose OS: Linux, Windows)**
  - Best-effort scheduling
  - Optimized for throughput
  - Feature-rich
  - Non-deterministic

### Real-Time Classification

#### Hard Real-Time
- **Characteristics**
  - Missing deadline = system failure
  - Catastrophic consequences
  
- **Examples**
  - Airbag deployment
  - Anti-lock braking (ABS)
  - Pacemaker
  - Flight control systems

#### Soft Real-Time
- **Characteristics**
  - Missing deadline = degraded performance
  - System continues to function
  
- **Examples**
  - Video streaming
  - Audio playback
  - Network packet handling

### Preemptive Kernel
- **Definition**
  - Higher priority task can interrupt running task immediately
  - Context switch happens instantly
  
- **Benefits**
  - Better responsiveness
  - Priority-based scheduling
  
- **Cost**
  - Context switch overhead
  - More complex synchronization

## Task Management

### Task States (4 States)
1. **Running**
   - Currently executing on CPU
   - Only one task per core
   
2. **Ready**
   - Ready to run
   - Waiting for CPU time
   
3. **Blocked (Waiting)**
   - Waiting for event (semaphore, queue, delay)
   - Not consuming CPU time
   
4. **Suspended**
   - Manually suspended
   - Not scheduled until resumed

### Task State Transitions
```
Ready ←→ Running → Blocked
  ↕                  ↓
Suspended ←──────────┘
```

### Idle Task
- **Lowest Priority Task**
  - Runs when no other task is ready
  - Always in ready state
  
- **Responsibilities**
  - Enter low-power mode
  - Background cleanup
  - Watchdog reset
  - CPU usage monitoring

## Stack Management

### Stack Overflow Detection
- **Methods**
  1. **Magic Number / Canary**
     - Write pattern at stack bottom
     - Check periodically
     
  2. **Stack High Water Mark**
     - Track maximum stack usage
     - Fill stack with pattern
     
  3. **Hardware MPU**
     - Guard pages
     - Generate exception on overflow

- **Consequences of Overflow**
  - Corrupted task data
  - System crash
  - Hard to debug

### Stack Size Calculation
- **Factors**
  - Function call depth
  - Local variables
  - Interrupt nesting
  - Context save area
  
- **Safety Margin**
  - Add 20-50% buffer
  - Measure actual usage in testing

## Scheduling

### Priority-Based Scheduling
- **Highest Priority Runs First**
  - Lower number = higher priority (FreeRTOS)
  - Deterministic selection
  
- **Same Priority**
  - Round-robin time slicing
  - Equal time quantum

### Round Robin (Within Same Priority)
- **Time Slice**
  - Fixed time quantum (e.g., 1ms)
  - Each task gets equal CPU time
  
- **Fairness**
  - Prevents starvation
  - All tasks make progress

### Tickless Idle Mode
- **Concept**
  - Suppress tick interrupts when idle
  - Sleep until next scheduled event
  
- **Benefits**
  - Lower power consumption
  - Reduced unnecessary wake-ups
  
- **Implementation**
  - Calculate time to next event
  - Configure timer for that duration

## Synchronization Problems

### Priority Inversion
- **Problem**
  - High-priority task blocked by low-priority task
  - Medium-priority task runs instead
  - Violates priority assumption
  
- **Example**
  - Task H (high) needs mutex held by Task L (low)
  - Task M (medium) preempts Task L
  - Task H waits for Task L, but Task M runs

### Solutions to Priority Inversion

#### Priority Inheritance
- **Mechanism**
  - Low-priority task inherits high-priority temporarily
  - While holding mutex needed by high-priority
  
- **Benefits**
  - Dynamic solution
  - No configuration needed
  
- **Limitations**
  - Deadlock not prevented
  - Chain effects possible

#### Priority Ceiling
- **Mechanism**
  - Mutex has ceiling priority
  - Task holding mutex raised to ceiling
  
- **Benefits**
  - Prevents deadlock
  - Bounded blocking time
  
- **Limitations**
  - Requires careful configuration

## Synchronization Primitives

### Mutex (RTOS Context)
- **Ownership**
  - Only owner can unlock
  - Mutual exclusion lock
  
- **Priority Inheritance**
  - Solves priority inversion
  - Temporary priority boost
  
- **Restrictions**
  - Cannot use in ISR
  - May cause blocking

### Semaphore (RTOS Context)
- **No Ownership**
  - Any task can signal
  - Signaling mechanism
  
- **Types**
  - Binary (0 or 1)
  - Counting (0 to N)
  
- **No Priority Inheritance**
  - Lighter weight
  - More flexible

### Event Groups / Flags
- **Multiple Events**
  - Each bit represents one event
  - 24 or 32 bits available
  
- **Wait Conditions**
  - Wait for ANY bit (OR)
  - Wait for ALL bits (AND)
  - Auto-clear or manual-clear
  
- **Use Cases**
  - Complex synchronization
  - Multiple conditions

### Message Queue
- **Purpose**
  - Task-to-task communication
  - Data passing with buffering
  
- **Operation**
  - Send: copy data to queue
  - Receive: copy data from queue
  
- **Blocking**
  - Block when full (send)
  - Block when empty (receive)
  
- **Overhead**
  - Data copy overhead
  - Memory for queue buffer

### Task Notification (FreeRTOS)
- **Lightweight**
  - Faster than queue/semaphore
  - Less memory overhead
  
- **Features**
  - 32-bit value
  - Increment, set bits, overwrite
  
- **Limitation**
  - Only one notification per task
  - One-to-one communication

## Real-Time Metrics

### Deadline
- **Definition**
  - Maximum time to complete task
  - Hard constraint
  
- **Analysis**
  - Worst-case execution time (WCET)
  - Schedulability analysis

### Jitter
- **Definition**
  - Variation in task start time
  - Deviation from periodic schedule
  
- **Causes**
  - Interrupt latency
  - Higher-priority task interference
  
- **Impact**
  - Control loop stability
  - Sensor sampling accuracy

## Code Considerations

### Reentrancy
- **Reentrant Function**
  - Can be called by multiple tasks safely
  - No global/static state
  
- **Requirements**
  - No global variables (or protected)
  - No static local variables
  - No non-reentrant library calls
  
- **Example: Non-Reentrant**
```c
char* get_string() {
    static char buffer[100];  // WRONG: shared state
    return buffer;
}
```

- **Example: Reentrant**
```c
char* get_string(char* buffer, size_t size) {
    // User provides buffer
    return buffer;
}
```

## RTOS Porting

### Files to Modify
- **`port.c` / `port.h`**
  - Context switch (save/restore registers)
  - Interrupt configuration
  - Critical section implementation
  
- **`portasm.s`**
  - Assembly for context switching
  - PendSV handler (ARM)
  - Stack initialization

### Context Switch Optimization
- **Minimize Overhead**
  - Save only necessary registers
  - Lazy FPU context save
  - Fast interrupt entry/exit
  
- **Techniques**
  - Use hardware stack (ARM)
  - Tail-chaining (ARM Cortex-M)
  - Efficient assembly

## Critical Sections

### Enter/Exit Critical
- **Implementation**
  - Disable interrupts
  - Ensures atomicity
  
- **Usage**
```c
taskENTER_CRITICAL();
// Access shared resource
taskEXIT_CRITICAL();
```

- **Caution**
  - Keep short (<10μs recommended)
  - Affects interrupt latency
  - No blocking operations inside

### Spinlock (Rarely Used in RTOS)
- **Problem on Single-Core MCU**
  - Busy-waiting wastes CPU
  - Can cause deadlock
  - Task holds lock, lower priority spins forever
  
- **Valid on Multi-Core**
  - Short critical sections
  - Different cores can progress

## Scheduler Control

### Starting the Scheduler
- **`vTaskStartScheduler()` (FreeRTOS)**
  - Initialize tick timer
  - Restore first task context
  - Jump to first task
  - Enable interrupts
  - Never returns

## Interview Practice
- [ ] Explain priority inversion with example
- [ ] Compare mutex vs semaphore in RTOS
- [ ] Design task structure for real application
- [ ] Calculate worst-case response time
- [ ] Describe context switch process
- [ ] Solve synchronization problems
- [ ] Explain stack overflow detection methods
