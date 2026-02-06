# 1.3 Memory Management

## Memory Hierarchy
- **Register**
  - ~1 cycle access time
  - Closest to CPU
  
- **Cache**
  - L1 Cache (~4 cycles)
  - L2 Cache (~12 cycles)
  - L3 Cache (~40-75 cycles)
  
- **RAM (Main Memory)**
  - 100-300 cycles
  - Volatile storage
  
- **Disk (Secondary Storage)**
  - Millions of cycles
  - Persistent storage

## Virtual Memory
- **Address Translation**
  - Virtual Address Space
  - Physical Address Space
  - Address mapping
  
- **Paging**
  - Fixed-size pages
  - Page frames
  - Page table structure
  
- **Segmentation**
  - Variable-size segments
  - Logical division
  - Segment table
  
- **Page Table**
  - Single-level page table
  - Multi-level page table
  - Inverted page table
  
- **TLB (Translation Lookaside Buffer)**
  - Fast address translation cache
  - TLB hit/miss
  - TLB reach

## Page Replacement Algorithms
- **FIFO (First In First Out)**
  - Simple implementation
  - Belady's anomaly
  
- **LRU (Least Recently Used)**
  - Optimal approximation
  - Implementation methods
  
- **LFU (Least Frequently Used)**
  - Frequency-based
  - Counter maintenance
  
- **Optimal Algorithm**
  - Theoretical best
  - Future knowledge required

## Memory Allocation
- **Allocation Strategies**
  - First Fit
  - Best Fit
  - Worst Fit
  
- **Fragmentation**
  - External fragmentation
  - Internal fragmentation
  - Compaction

## Study Materials
- [ ] Memory hierarchy diagram
- [ ] Address translation examples
- [ ] Page table structure
- [ ] Page replacement simulations

## Practice Problems
- [ ] Calculate page faults
- [ ] Implement page replacement algorithms
- [ ] Virtual to physical address translation
