# Computer Architecture & Embedded Hardware

## Overview
Comprehensive study materials covering computer architecture, digital logic, microcontroller peripherals, RTOS, and embedded systems development. This section is designed for embedded systems interviews and practical hardware development.

## Course Structure

### [2.1 CPU Architecture & Logic](./2.1_CPU_Architecture/)
- Von Neumann vs Harvard architecture
- RISC vs CISC (ARM vs x86)
- Pipeline techniques and hazards
- Branch prediction
- Cache coherence (MESI protocol)
- Interrupt handling and DMA
- ARM-specific features (modes, register banking, CPSR)
- MMU vs MPU

### [2.2 Digital Logic & Hardware Fundamentals](./2.2_Digital_Logic/)
- Pull-up/Pull-down resistors and floating states
- Open drain vs Push-pull outputs
- Setup time, hold time, and metastability
- Flip-flop vs Latch
- Debouncing techniques
- ADC/DAC and PWM
- GPIO modes and configurations
- Clock generation (Crystal, Oscillator, PLL)
- Signal integrity and impedance matching

### [2.3 Microcontroller & Peripherals](./2.3_MCU_Peripherals/)
- Timer and counter fundamentals
- Watchdog timer operation
- Interrupt controller (NVIC)
- Input capture and output compare
- Encoder interface
- Power modes (Sleep, Stop, Standby)
- RTC (Real-Time Clock)
- JTAG vs SWD debugging
- Bootloader and dual bank firmware
- Startup code and initialization

### [2.4 Real-Time Operating System (RTOS)](./2.4_RTOS/)
- RTOS vs GPOS differences
- Hard vs Soft real-time
- Task states and scheduling
- Priority inversion and solutions
- Synchronization primitives (Mutex, Semaphore, Event Groups)
- Message queues and task notifications
- Stack overflow detection
- Critical sections and reentrancy
- RTOS porting considerations

### [2.5 Build, Debug & Toolchain](./2.5_Build_and_Debug/)
- Cross-compilation and toolchain
- Build process (preprocessing, compilation, assembly, linking)
- Linker scripts and memory layout
- Binary analysis tools (objdump, nm, readelf, size)
- Map file analysis
- GDB debugging commands
- Hardware vs software breakpoints
- Semihosting
- Intel Hex vs Binary formats
- XIP (Execute In Place)
- Static analysis and MISRA-C

## Study Approach
1. Focus on **conceptual understanding** first
2. Practice explaining concepts as if in an interview
3. Prepare real-world debugging stories
4. Understand hardware-software interaction
5. Practice with oscilloscope and logic analyzer (if available)

## Interview Preparation
Each section includes:
- Key concepts and definitions
- Comparison tables (X vs Y)
- Practical examples
- Common interview questions
- Debug scenarios

## Essential Interview Question
**"Describe your most difficult debugging experience"** - Always have a detailed STAR (Situation-Task-Action-Result) story ready with:
- Symptoms observed
- Hypothesis formation
- Verification methods
- Root cause (race condition, timing issue, hardware problem)
- Solution and lessons learned
