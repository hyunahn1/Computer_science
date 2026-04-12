# 1.5 Deadlock

## Deadlock Necessary Conditions
All four conditions must hold for deadlock to occur:

1. **Mutual Exclusion**
   - Resources cannot be shared
   - Only one process at a time
   
2. **Hold and Wait**
   - Process holds resources while waiting
   - Partial allocation
   
3. **No Preemption**
   - Resources cannot be forcibly taken
   - Voluntary release only
   
4. **Circular Wait**
   - Circular chain of waiting processes
   - P1 → P2 → P3 → P1

## Deadlock Handling Strategies

### Prevention
- **Break Mutual Exclusion**
  - Make resources shareable (if possible)
  
- **Eliminate Hold and Wait**
  - Request all resources at once
  - Release before requesting new ones
  
- **Allow Preemption**
  - Force resource release
  - Save and restore state
  
- **Prevent Circular Wait**
  - Resource ordering
  - Numbered resources

### Avoidance
- **Safe State**
  - System can allocate resources safely
  - Guaranteed completion sequence
  
- **Banker's Algorithm**
  - Resource allocation algorithm
  - Maximum need declaration
  - Safe state checking
  
- **Resource Allocation Graph**
  - Visual representation
  - Cycle detection

### Detection & Recovery
- **Detection Algorithm**
  - Periodic checking
  - Wait-for graph
  - Cycle detection
  
- **Recovery Methods**
  - Process termination (abort one or all)
  - Resource preemption (rollback)
  - Checkpoint and restart

### Ignore (Ostrich Algorithm)
- **Trade-off**
  - Rare occurrence
  - Prevention cost vs occurrence cost
  - Reboot if needed

## Study Materials
- [ ] Deadlock example scenarios
- [ ] Banker's algorithm walkthrough
- [ ] Resource allocation graphs
- [ ] Detection algorithm implementation

## Practice Problems
- [ ] Identify deadlock conditions
- [ ] Apply Banker's algorithm
- [ ] Design deadlock-free systems
- [ ] Analyze real-world deadlocks
