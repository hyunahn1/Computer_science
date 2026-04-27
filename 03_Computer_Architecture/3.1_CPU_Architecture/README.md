# 3.1 CPU Architecture & Logic

## Computer Architecture Models

### Von Neumann vs Harvard Architecture
- **Von Neumann Architecture**
  - Unified memory for data and instructions
  - Bottleneck: cannot fetch instruction and data simultaneously
  - Simpler design
  
- **Harvard Architecture**
  - Separate memory buses for data and instructions
  - Can access both simultaneously (faster)
  - More complex, used in DSP and modern CPUs

### RISC vs CISC Architecture
- **RISC (Reduced Instruction Set Computer)**
  - Simple, fixed-length instructions
  - ARM architecture
  - Pipeline-friendly
  - Load-store architecture
  
- **CISC (Complex Instruction Set Computer)**
  - Complex, variable-length instructions
  - x86 architecture
  - More instructions, fewer lines of code
  - Direct memory operations

## Pipeline & Performance

### Pipeline Technique
- **Concept**
  - Overlapping Fetch-Decode-Execute stages
  - Increases throughput
  - 5-stage pipeline: IF, ID, EX, MEM, WB
  
- **Benefits**
  - Multiple instructions in flight
  - Better CPU utilization

### Pipeline Hazards (3 Types)
1. **Structural Hazard**
   - Resource conflict (hardware limitation)
   - Solution: duplicate resources
   
2. **Data Hazard**
   - Data dependency between instructions
   - RAW (Read After Write), WAR, WAW
   - Solution: forwarding, stalling
   
3. **Control Hazard**
   - Branch/jump instructions
   - Don't know next instruction until execution
   - Solution: branch prediction

### Branch Prediction
- **Purpose**
  - Guess next instruction path
  - Keep pipeline filled
  
- **Misprediction Cost**
  - Pipeline flush (discard wrong instructions)
  - Refetch correct instructions
  - Performance penalty (3-20 cycles)

### Superscalar and Out-of-Order Execution
- **Superscalar Issue**
  - Multiple instructions can be issued in one cycle when dependencies and execution units allow it
  - Throughput is limited by instruction-level parallelism, dependency chains, and resource conflicts

- **Out-of-Order Execution**
  - CPU may execute later independent instructions before earlier stalled instructions
  - Preserves architectural state as if instructions retired in program order

- **Core Structures**
  - Register renaming removes false dependencies (WAR/WAW)
  - Reservation stations or scheduler queues hold ready operations
  - Reorder buffer (ROB) commits results in order
  - Load/store queue tracks memory dependencies

- **Study Target**
  - Be able to explain why two programs with the same Big-O can differ due to dependency chains, branch predictability, and memory stalls

## Cache & Memory

### Cache Organization
- **Cache Line**
  - Minimum unit moved between memory and cache
  - Spatial locality helps when nearby data is accessed together

- **Associativity**
  - Direct-mapped: one possible location, simple but conflict-prone
  - Set-associative: several possible ways per set
  - Fully associative: any line can go anywhere, expensive at scale

- **Replacement Policy**
  - LRU is idealized; hardware often uses approximations
  - Replacement behavior can change benchmark results even when asymptotic complexity is unchanged

### Cache Coherence (MESI Protocol)
- **Multi-core Cache Synchronization**
  - M (Modified): dirty, exclusive
  - E (Exclusive): clean, only in this cache
  - S (Shared): clean, may exist in other caches
  - I (Invalid): data is stale

### False Sharing
- **Definition**
  - Independent variables on the same cache line cause coherence traffic when different cores write them

- **Why It Matters**
  - The program has no logical sharing, but the hardware sees cache-line sharing
  - Can destroy multicore scalability in counters, queues, and per-thread state

- **Required Experiment**
  - Write a benchmark with adjacent per-thread counters, then add cache-line padding and compare throughput

### Memory Consistency and Ordering
- **Problem**
  - Coherence says caches agree on a location; consistency says when writes become visible across locations

- **Key Concepts**
  - Store buffer, load speculation, compiler reordering, hardware reordering
  - Acquire/release ordering for synchronization
  - Memory fences/barriers when ordering must be forced

- **Architecture Contrast**
  - x86 is relatively strong (TSO-style model)
  - ARM is weaker and requires more explicit ordering in low-level concurrent code

### TLB and Address Translation
- **TLB**
  - Cache for virtual-to-physical translations
  - TLB misses can dominate performance for pointer-heavy or large-working-set programs

- **Page Size Trade-off**
  - Larger pages reduce TLB pressure
  - Smaller pages reduce internal fragmentation and can improve protection granularity

### Cache Miss Types (3C)
1. **Compulsory Miss**
   - First access to data (cold start)
   - Unavoidable
   
2. **Capacity Miss**
   - Cache too small to hold working set
   - Solution: larger cache
   
3. **Conflict Miss**
   - Multiple addresses map to same cache line
   - Solution: higher associativity

## Interrupt Handling

### Interrupt Vector Table
- **Definition**
  - Table of ISR (Interrupt Service Routine) addresses
  - Indexed by interrupt number
  
- **Operation**
  - Interrupt occurs → lookup vector → jump to ISR

### Polling vs Interrupt (Hardware Perspective)
- **Polling**
  - CPU continuously checks device status
  - Wastes CPU cycles and power
  - Simple implementation
  
- **Interrupt**
  - Device signals CPU when ready
  - CPU sleeps until interrupted
  - Power efficient, responsive

### DMA (Direct Memory Access)
- **Role**
  - Transfers data between memory and peripherals
  - Without CPU involvement
  
- **Benefits**
  - Reduces CPU load
  - Higher throughput
  - CPU can do other work

## Specialized Units

### FPU (Floating Point Unit)
- **With FPU**
  - Hardware acceleration for float/double operations
  - Fast and efficient
  
- **Without FPU**
  - Software emulation via library
  - Slow (10-100x slower)
  - Common in low-cost MCUs

### SIMD (Single Instruction Multiple Data)
- **Concept**
  - One instruction operates on multiple data elements
  - Data-level parallelism
  
- **Use Cases**
  - Multimedia processing
  - Vector/matrix operations
  - DSP applications
  - ARM NEON, x86 SSE/AVX

### Performance Counters
- **Purpose**
  - Measure hardware events instead of guessing

- **Examples**
  - Cycles, instructions retired, branch misses, cache misses, LLC misses, TLB misses

- **Study Target**
  - For any performance claim, identify which counter would support or falsify it

## ARM-Specific Architecture

### ARM Operating Modes
- **User Mode**
  - Normal application code
  - Restricted access
  
- **FIQ (Fast Interrupt)**
  - Highest priority interrupt
  - More banked registers
  
- **IRQ (Interrupt)**
  - Normal interrupt mode
  
- **Supervisor (SVC)**
  - OS kernel mode
  - System calls
  
- **Abort/Undefined**
  - Exception handling

### Register Banking
- **Concept**
  - Different register sets for different modes
  - Mode switch doesn't require stack saving
  - Fast context switch
  
- **Banked Registers**
  - Each mode has its own R13 (SP), R14 (LR)
  - FIQ has R8-R12 banked

### Special Registers

#### Program Counter (PC)
- **R15 in ARM**
  - Address of next instruction to execute
  - Branch = modify PC

#### Stack Pointer (SP)
- **R13 in ARM**
  - Top of current stack
  - Each mode has its own SP

#### Link Register (LR)
- **R14 in ARM**
  - Stores return address on function call
  - BL (Branch with Link) saves PC to LR
  - Fast return without stack access

#### CPSR (Current Program Status Register)
- **Contents**
  - Condition flags: N (Negative), Z (Zero), C (Carry), V (Overflow)
  - Interrupt mask bits: I (IRQ), F (FIQ)
  - Mode bits: user, IRQ, FIQ, SVC, etc.
  - Thumb state bit

### Context Switching
- **Hardware Information to Save**
  - General purpose registers (R0-R15)
  - CPSR (status register)
  - FPU registers (if used)
  - Coprocessor state
  
- **Process**
  - Save current context
  - Load new context
  - Resume execution

## Memory Management Units

### Systems Without MMU
- **MCU (Microcontroller)**
  - No virtual address translation
  - Direct physical address access
  - Simpler but less protection
  
- **MPU Available**
  - Memory Protection Unit as alternative

### MPU (Memory Protection Unit)
- **Role**
  - Controls access permissions (Read/Write/Execute)
  - Memory region protection
  - RTOS task isolation
  
- **No Address Translation**
  - Only protection, not virtualization
  - Lighter than MMU

## Expert Depth Checklist

### Mechanism
- [ ] Draw a 5-stage pipeline and annotate structural, data, and control hazards
- [ ] Explain how register renaming removes WAR/WAW dependencies but not true RAW dependencies
- [ ] Describe how a ROB lets a CPU execute speculatively while retiring in order
- [ ] Explain the difference between cache coherence and memory consistency
- [ ] Trace a virtual memory access through TLB lookup, page table walk, cache lookup, and memory access

### Analysis
- [ ] Calculate CPI from base CPI, cache miss rate, miss penalty, branch frequency, and misprediction penalty
- [ ] Compare direct-mapped vs set-associative cache behavior on a concrete address trace
- [ ] Explain why pointer chasing hurts prefetching and memory-level parallelism
- [ ] Analyze why false sharing can make a multicore program slower than a single-threaded one
- [ ] Distinguish algorithmic complexity from microarchitectural performance

### Implementation and Measurement
- [ ] Write a microbenchmark that demonstrates branch prediction effects
- [ ] Write a cache locality benchmark comparing array traversal and pointer chasing
- [ ] Reproduce false sharing and fix it with cache-line padding
- [ ] Use `perf stat` or equivalent PMU tooling to compare branch misses, cache misses, and instructions retired
- [ ] Inspect generated assembly for a small C/C++ function and explain calling convention effects

### Architecture Contrast
- [ ] Compare x86 TSO and ARM weak memory ordering at the level needed for lock-free code
- [ ] Compare Cortex-A with MMU and Cortex-M with MPU/no MMU
- [ ] Explain when DMA requires cache maintenance on embedded systems
- [ ] Explain why `volatile` is not a synchronization primitive for multithreaded code
- [ ] Explain when memory-mapped I/O needs compiler barriers, CPU barriers, or both

### Failure Modes
- [ ] Diagnose a branch-heavy benchmark with high branch-miss rate
- [ ] Diagnose a large-working-set workload with high LLC or TLB misses
- [ ] Explain a stale device-buffer bug caused by DMA/cache incoherence
- [ ] Explain a data race that appears only on weakly ordered hardware
- [ ] Explain a hard fault or protection fault caused by MPU/MMU permissions

### Primary Sources
- [ ] Read the relevant chapters of Hennessy & Patterson or Bryant/O'Hallaron for pipeline, cache, and virtual memory
- [ ] Read the ARM Architecture Reference Manual sections on exceptions, barriers, and memory ordering
- [ ] Read an x86 or Intel/AMD memory ordering reference at least once
- [ ] Read your target MCU reference manual for interrupt controller, MPU, DMA, and cache behavior
- [ ] Keep notes that separate textbook models from vendor-specific behavior

## Interview Practice
- [ ] Explain pipeline hazards with examples
- [ ] Draw MESI state transition diagram
- [ ] Compare ARM vs x86 architectures
- [ ] Explain context switching in detail
- [ ] Describe interrupt handling flow
