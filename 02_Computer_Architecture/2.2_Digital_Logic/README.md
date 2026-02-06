# 2.2 Digital Logic & Hardware Fundamentals

## Basic Circuit Concepts

### Pull-up / Pull-down Resistors
- **Purpose**
  - Prevent floating state
  - Ensure definite HIGH or LOW level
  
- **Pull-up Resistor**
  - Connects to VCC
  - Default HIGH state
  
- **Pull-down Resistor**
  - Connects to GND
  - Default LOW state
  
- **Typical Values**
  - 1kΩ - 10kΩ (balance: power vs noise immunity)

### Floating State
- **Definition**
  - Pin not connected to HIGH or LOW
  - Neither 0 nor 1
  
- **Problems**
  - Susceptible to noise
  - Unpredictable behavior
  - Can cause random switching

## Output Driver Types

### Open Drain / Open Collector
- **Characteristics**
  - Output can only pull LOW or Hi-Z (high impedance)
  - Cannot actively drive HIGH
  - Requires external pull-up resistor
  
- **Advantages**
  - Multiple outputs can be wire-AND'ed
  - Different voltage levels possible
  
- **Use Cases**
  - I2C bus (multi-master)
  - Shared interrupt lines

### Push-Pull Output
- **Characteristics**
  - Two transistors (high-side and low-side)
  - Can actively drive both HIGH and LOW
  - Strong output drive
  
- **Advantages**
  - Fast switching
  - No external resistor needed
  
- **Disadvantages**
  - Cannot wire multiple outputs together
  - Can cause conflict

## Timing & Synchronization

### Setup Time and Hold Time
- **Setup Time**
  - Minimum time data must be stable BEFORE clock edge
  - Data must arrive early enough
  
- **Hold Time**
  - Minimum time data must remain stable AFTER clock edge
  - Data must not change too soon
  
- **Violation Consequences**
  - Metastability
  - Data corruption

### Metastability
- **Phenomenon**
  - Flip-flop output stuck at intermediate voltage
  - Caused by setup/hold time violation
  - Unpredictable duration
  
- **Solution**
  - Double flip-flop synchronizer
  - Multi-stage synchronization
  - Increases MTBF (Mean Time Between Failures)

### Flip-flop vs Latch
- **Flip-flop**
  - Edge-triggered (responds at clock edge)
  - Sampling at specific moment
  - More robust, preferred in design
  
- **Latch**
  - Level-triggered (transparent when clock HIGH/LOW)
  - Can cause timing issues
  - Used in specific applications

## Signal Conditioning

### Debouncing
- **Problem**
  - Mechanical switches bounce (multiple transitions)
  - Can register as multiple presses
  
- **Hardware Solutions**
  - RC filter (resistor + capacitor)
  - Schmitt trigger
  - SR latch

### Schmitt Trigger (Hysteresis)
- **Characteristics**
  - Two threshold voltages (HIGH and LOW)
  - Different turn-on and turn-off points
  
- **Benefits**
  - Prevents noise-induced oscillation
  - Clean transitions on slow/noisy signals
  - Built-in noise immunity

## Analog-Digital Conversion

### ADC (Analog-to-Digital Converter)

#### Resolution
- **Bit Depth**
  - 8-bit: 256 levels
  - 10-bit: 1024 levels
  - 12-bit: 4096 levels
  
- **Voltage Step**
  - Resolution = Vref / 2^n
  - Higher bits = better precision

#### Sampling Theorem (Nyquist)
- **Rule**
  - Sample rate ≥ 2 × maximum signal frequency
  - Prevents aliasing
  
- **Example**
  - Audio (20kHz max): need 40kHz+ sampling
  - Typically use 44.1kHz or 48kHz

### DAC (Digital-to-Analog Converter)
- **Purpose**
  - Convert digital value to analog voltage
  
- **Use Cases**
  - Audio output
  - Waveform generation
  - Control voltage generation

### PWM (Pulse Width Modulation)
- **Concept**
  - Digital pulses with varying width
  - Average voltage = VCC × duty cycle
  
- **Duty Cycle**
  - Percentage of time signal is HIGH
  - 0% = always LOW, 100% = always HIGH
  
- **Applications**
  - Motor speed control
  - LED brightness
  - Power regulation
  - Servo control

## GPIO & Pin Configuration

### GPIO Modes (4 Types)
1. **Input Mode**
   - Read external signals
   - Pull-up/pull-down configurable
   
2. **Output Mode**
   - Drive HIGH/LOW
   - Push-pull or open-drain
   
3. **Alternate Function**
   - UART TX/RX
   - SPI MISO/MOSI
   - Timer PWM output
   
4. **Analog Mode**
   - ADC input
   - DAC output
   - Comparator input

### GPIO Slew Rate
- **Definition**
  - Maximum rate of voltage change (V/ns)
  
- **Control Reasons**
  - Slower slew rate → less EMI/noise
  - Faster slew rate → cleaner edges for high-speed signals
  
- **Trade-off**
  - Speed vs electromagnetic compatibility

## Interrupt Configuration

### Edge Trigger vs Level Trigger
- **Edge Trigger**
  - Rising edge (LOW → HIGH)
  - Falling edge (HIGH → LOW)
  - Detects transition moment
  - Good for momentary events
  
- **Level Trigger**
  - HIGH level
  - LOW level
  - Detects sustained state
  - Good for status signals

## Clock Generation

### Crystal vs Oscillator
- **Crystal**
  - Passive component
  - Requires external oscillator circuit
  - Higher accuracy
  - Lower cost
  
- **Oscillator**
  - Active component (complete circuit)
  - Just add power → clock output
  - More expensive
  - Easier to use

### PLL (Phase Locked Loop)
- **Purpose**
  - Multiply low external clock to high internal clock
  - Frequency synthesis
  
- **Example**
  - 8MHz crystal → PLL → 72MHz system clock
  
- **Benefits**
  - Lower frequency crystal (cheaper, less EMI)
  - Flexible clock configuration

## Power & Signal Integrity

### Bypass/Decoupling Capacitor
- **Placement**
  - Very close to IC power pins
  - Between VCC and GND
  
- **Purpose**
  - Filter high-frequency noise
  - Provide instantaneous current
  - Stabilize power supply
  
- **Typical Values**
  - 100nF ceramic (high freq)
  - 10μF electrolytic (low freq)

### Impedance Matching
- **Need**
  - High-speed signals (>10MHz)
  - Long traces
  
- **Purpose**
  - Prevent signal reflection
  - Maintain signal integrity
  - Maximum power transfer
  
- **Methods**
  - Series termination
  - Parallel termination
  - Controlled impedance PCB (50Ω, 100Ω)

### Ground Plane
- **PCB Layer**
  - Continuous copper pour connected to GND
  
- **Benefits**
  - Low-impedance return path
  - EMI shielding
  - Heat dissipation
  - Signal integrity

## Interview Practice
- [ ] Design debounce circuit with RC values
- [ ] Calculate ADC resolution for given Vref
- [ ] Explain metastability and solution
- [ ] Draw push-pull vs open-drain circuits
- [ ] Describe setup/hold time violations
