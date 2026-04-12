# 2.1 CPU Architecture & Logic

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

## Cache & Memory

### Cache Coherence (MESI Protocol)
- **Multi-core Cache Synchronization**
  - M (Modified): dirty, exclusive
  - E (Exclusive): clean, only in this cache
  - S (Shared): clean, may exist in other caches
  - I (Invalid): data is stale

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

## Interview Practice
- [ ] Explain pipeline hazards with examples
- [ ] Draw MESI state transition diagram
- [ ] Compare ARM vs x86 architectures
- [ ] Explain context switching in detail
- [ ] Describe interrupt handling flow
