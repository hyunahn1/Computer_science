# 5.2 CPU Scheduling

## CPU Scheduler Concepts
- **Ready Queue**
  - Queue management
  - Process selection
  
- **Context Switching**
  - Context switching overhead
  - State saving/restoring
  
- **Scheduling Types**
  - Preemptive scheduling
  - Non-preemptive scheduling

## Scheduling Algorithms
- **FCFS (First Come First Served)**
  - Non-preemptive
  - Simple implementation
  - Convoy effect
  
- **SJF (Shortest Job First)**
  - Non-preemptive
  - Optimal average waiting time
  - Prediction challenges
  
- **SRTF (Shortest Remaining Time First)**
  - Preemptive version of SJF
  - Context switching overhead
  
- **Round Robin**
  - Preemptive
  - Time quantum
  - Fair CPU distribution
  
- **Priority Scheduling**
  - Static vs Dynamic priority
  - Priority inversion
  - Starvation problem
  
- **Multilevel Feedback Queue**
  - Multiple queues
  - Aging mechanism
  - Flexible scheduling

## Scheduling Metrics
- **Turnaround Time**
  - Completion time - Arrival time
  - Overall performance measure
  
- **Waiting Time**
  - Time spent in ready queue
  - Scheduler efficiency
  
- **Response Time**
  - First response - Arrival time
  - Interactive system metric

## Study Materials
- [ ] Gantt charts for each algorithm
- [ ] Scheduling calculation examples
- [ ] Algorithm comparison table

## Practice Problems
- [ ] Calculate metrics for given scenarios
- [ ] Implement scheduling simulator
- [ ] Compare algorithm performance

## Expert Depth Checklist
- [ ] Trace the kernel/user boundary for the topic: syscall, trap, interrupt, scheduler, page fault, VFS, or driver path.
- [ ] Explain the invariant the OS maintains and what breaks if it is violated.
- [ ] Reproduce behavior with a small C program, shell experiment, `strace`, `perf`, `/proc`, or debugger output.
- [ ] Analyze concurrency and failure: race, deadlock, starvation, priority inversion, lost wakeup, or crash consistency bug.
- [ ] Compare policy vs mechanism and explain which part is configurable.
- [ ] Read at least one authoritative source: OSTEP, Linux man pages, kernel docs, or OS textbook chapter.
