# 1.7 I/O & Interrupt

## I/O Methods

### Polling (Programmed I/O)
- **Busy Waiting**
  - CPU repeatedly checks device status
  - Simple implementation
  
- **Disadvantages**
  - CPU time waste
  - Inefficient for slow devices

### Interrupt-Driven I/O
- **Signal-Based**
  - Device sends interrupt signal
  - CPU notified when ready
  
- **Advantages**
  - CPU free for other tasks
  - Efficient resource usage
  
- **Disadvantages**
  - Interrupt overhead
  - Context switching cost

### DMA (Direct Memory Access)
- **Hardware Controller**
  - Independent data transfer
  - No CPU involvement
  
- **Operation**
  - CPU sets up transfer
  - DMA controller handles data
  - Interrupt upon completion
  
- **Advantages**
  - Minimal CPU overhead
  - High throughput
  - Efficient for large transfers

## Interrupt Handling

### Interrupt Handler
- **Interrupt Service Routine (ISR)**
  - Respond to interrupt
  - Handle device request
  - Minimal processing
  
- **Handler Structure**
  - Save context
  - Execute handler code
  - Restore context
  - Return from interrupt

### Interrupt Vector Table
- **Interrupt Vector**
  - Array of handler addresses
  - Indexed by interrupt number
  
- **Vectored Interrupts**
  - Direct jump to handler
  - Fast response time

### Interrupt Characteristics

#### Interrupt Latency
- **Definition**
  - Time from interrupt to handler start
  
- **Components**
  - Interrupt detection
  - Context saving
  - Handler dispatch

#### Interrupt Priority
- **Priority Levels**
  - Multiple interrupt sources
  - Priority assignment
  
- **Handling**
  - Higher priority preempts lower
  - Nested interrupts
  - Priority inversion issues

## I/O Devices

### Device Types
- **Block Devices**
  - Fixed-size blocks
  - Random access (e.g., disk)
  
- **Character Devices**
  - Byte stream
  - Sequential access (e.g., keyboard)

### Device Drivers
- **Interface**
  - Standardized operations
  - Device abstraction
  
- **Responsibilities**
  - Device initialization
  - Request handling
  - Error management

## Study Materials
- [ ] I/O method comparisons
- [ ] Interrupt handling flow
- [ ] DMA operation diagram
- [ ] Device driver architecture

## Practice Problems
- [ ] Calculate I/O overhead
- [ ] Implement interrupt handler
- [ ] Design DMA controller
- [ ] Analyze interrupt latency
