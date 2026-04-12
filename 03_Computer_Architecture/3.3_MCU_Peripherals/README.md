# 2.3 Microcontroller & Peripherals

## Timer & Counter

### Timer vs Counter
- **Timer**
  - Counts internal clock pulses
  - Time measurement
  - Periodic interrupts
  
- **Counter**
  - Counts external events
  - Event counting (pulses, edges)
  - Frequency measurement

### Prescaler
- **Definition**
  - Clock divider for timer
  - Reduces timer counting speed
  
- **Purpose**
  - Slow down high-speed system clock
  - Achieve longer time intervals
  
- **Example**
  - 72MHz / 72 = 1MHz timer clock
  - Each tick = 1μs

### Watchdog Timer
- **Operation**
  - Countdown timer
  - Must be periodically "kicked" (reset)
  - If not kicked → timer expires → system reset
  
- **Purpose**
  - Recover from software hang
  - Detect infinite loops
  - Ensure system liveness
  
- **Configuration**
  - Independent watchdog (IWDG): always running
  - Window watchdog (WWDG): kick within specific window

## Interrupt Controller

### NVIC (Nested Vectored Interrupt Controller)
- **ARM Cortex-M Feature**
  - Hardware interrupt priority management
  - Supports nested interrupts
  - Fast interrupt entry/exit
  
- **Priority Levels**
  - Configurable priority for each interrupt
  - Higher priority can preempt lower priority
  
- **Grouping**
  - Preemption priority
  - Sub-priority (when preemption equal)

### Interrupt Priority & Nesting
- **Preemption**
  - Higher priority interrupt can interrupt current ISR
  - Nested execution
  
- **Priority Levels**
  - 0 = highest priority
  - Lower number = higher priority
  
- **Considerations**
  - Deeper nesting = more stack usage
  - Critical sections to prevent race conditions

## Timer Modes

### Input Capture
- **Purpose**
  - Capture timer value when external event occurs
  - Timestamp external signals
  
- **Use Cases**
  - Frequency measurement
  - Pulse width measurement
  - Event timing
  
- **Operation**
  - Edge detected → timer value stored in register

### Output Compare
- **Purpose**
  - Compare timer value with preset value
  - Toggle/set/reset pin when match occurs
  
- **Use Cases**
  - PWM generation
  - Precise timing control
  - Waveform generation
  
- **PWM Mode**
  - Automatically toggle output based on compare values

### Encoder Interface
- **Purpose**
  - Read quadrature encoder signals
  - Detect rotation direction and speed
  
- **Signals**
  - Channel A and B (90° phase shift)
  - Direction determined by lead/lag relationship
  
- **Applications**
  - Motor position control
  - Rotary knob input

## System Timers

### SysTick Timer
- **ARM Cortex-M Core Timer**
  - 24-bit countdown timer
  - Part of CPU core
  
- **Primary Use**
  - RTOS tick generation
  - OS scheduling heartbeat
  - Time base for delays
  
- **Configuration**
  - Typically 1ms or 10ms tick
  - Interrupt-based

## Power Management

### Power Modes
1. **Sleep Mode**
   - CPU clock stopped
   - Peripherals keep running
   - RAM retained
   - Fast wake-up
   
2. **Stop Mode**
   - All clocks stopped (except LSI/LSE)
   - RAM retained
   - Ultra-low power
   - Slower wake-up
   
3. **Standby Mode**
   - Most circuitry powered down
   - Only RTC and backup registers retained
   - Lowest power consumption
   - Requires system reset on wake-up

### Wake-up Sources
- **Common Sources**
  - External interrupt (EXTI)
  - RTC alarm
  - USB resume
  - UART receive
  - Watchdog
  
- **Configuration**
  - Enable specific wake-up sources before sleep

## Real-Time Clock (RTC)

### RTC Features
- **Battery-Backed**
  - Continues running when main power off
  - Uses VBAT pin
  
- **Functions**
  - Time keeping (hours, minutes, seconds)
  - Date keeping (day, month, year)
  - Alarm functionality
  - Wake-up timer
  
- **Clock Source**
  - 32.768kHz crystal (standard)
  - Internal RC oscillator (less accurate)

## Debug & Programming

### JTAG vs SWD
- **JTAG (Joint Test Action Group)**
  - 4-5 pins: TMS, TCK, TDI, TDO, (TRST)
  - Industry standard
  - Also used for boundary scan testing
  
- **SWD (Serial Wire Debug)**
  - 2 pins: SWDIO, SWCLK
  - ARM Cortex-M specific
  - Saves pins on small packages
  - Faster than JTAG
  
- **Recommendation**
  - Use SWD for ARM development

## Bootloader & Firmware

### Bootloader Entry Conditions
- **Common Methods**
  - BOOT pins state at reset
  - Button press during power-on
  - Magic value in RAM/EEPROM
  - USB enumeration failure
  
- **Purpose**
  - Enter firmware update mode
  - Recovery from bad firmware

### Dual Bank Flash
- **Concept**
  - Flash divided into Bank A and Bank B
  - Run from one bank, update the other
  
- **Benefits**
  - Fail-safe updates (power loss recovery)
  - No bricking
  - Can rollback to old firmware
  
- **Process**
  - Update Bank B while running Bank A
  - Switch banks after successful update

### eFuse
- **One-Time Programmable Memory**
  - Can only write once (bits blown)
  - Cannot be erased
  
- **Use Cases**
  - Security keys
  - Configuration locks
  - Device ID
  - Bootloader protection

## Advanced Features

### Shadow Register
- **Purpose**
  - Buffer multi-register updates
  - Ensure atomic consistency
  
- **Operation**
  - Write to shadow register
  - Transfer to actual register atomically
  
- **Example**
  - Timer auto-reload register (ARR)

### Bit-Banding (ARM Cortex-M)
- **Concept**
  - Each bit mapped to separate word address
  - Atomic bit manipulation
  
- **Benefits**
  - No read-modify-write needed
  - Thread-safe without disabling interrupts
  
- **Address Mapping**
  - Bit-band region → bit-band alias
  - 1-bit access as 32-bit operation

## Startup & Initialization

### Startup Code
- **Responsibilities**
  - Set stack pointer (SP)
  - Copy `.data` section from Flash to RAM
  - Zero-initialize `.bss` section
  - Initialize system clocks
  - Call `main()` function
  
- **Files**
  - `startup_*.s` (assembly)
  - Vector table definition
  - Reset handler

## Interview Practice
- [ ] Explain watchdog timer operation
- [ ] Compare JTAG and SWD
- [ ] Describe dual bank firmware update
- [ ] Calculate timer period with prescaler
- [ ] Design power management strategy
- [ ] Explain interrupt priority handling
