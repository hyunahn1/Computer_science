# 2.5 Build, Debug & Toolchain

## Cross-Compilation

### Cross Compiler
- **Definition**
  - Compiler running on one platform (host) generating code for another platform (target)
  - Example: x86 PC (host) → ARM MCU (target)
  
- **Naming Convention**
  - `arm-none-eabi-gcc`
  - Architecture-Vendor-OS-ABI-tool

### Toolchain Components
- **Essential Tools**
  - Compiler: `gcc`, `clang`
  - Assembler: `as`
  - Linker: `ld`
  - Archiver: `ar` (static libraries)
  - Debugger: `gdb`
  - Binary utilities: `objcopy`, `objdump`, `nm`, `size`

## Build Process

### Compilation Stages
1. **Preprocessing**
   - `gcc -E`
   - Include header files
   - Expand macros
   - Handle `#ifdef` directives
   - Output: `.i` file
   
2. **Compilation**
   - `gcc -S`
   - Parse C code
   - Generate assembly code
   - Output: `.s` file
   
3. **Assembly**
   - `gcc -c`
   - Convert assembly to machine code
   - Generate object file
   - Output: `.o` file
   
4. **Linking**
   - `ld` or `gcc`
   - Combine object files
   - Resolve symbols
   - Apply linker script
   - Output: `.elf` file

## Linker Script

### Linker Script (.ld) Role
- **Memory Layout Definition**
  - FLASH origin and size
  - RAM origin and size
  - Stack and heap placement
  
- **Section Placement**
  - `.text` → FLASH (code)
  - `.rodata` → FLASH (constants)
  - `.data` → RAM, load from FLASH (initialized globals)
  - `.bss` → RAM (zero-initialized)
  
- **Symbol Definition**
  - `_estack` (stack end)
  - `_sdata`, `_edata` (data section bounds)
  - `_sbss`, `_ebss` (bss section bounds)

### Example Linker Script Structure
```ld
MEMORY
{
  FLASH (rx) : ORIGIN = 0x08000000, LENGTH = 256K
  RAM (rwx)  : ORIGIN = 0x20000000, LENGTH = 64K
}

SECTIONS
{
  .text : { *(.text) } > FLASH
  .data : { *(.data) } > RAM AT> FLASH
  .bss  : { *(.bss)  } > RAM
}
```

## Binary Analysis Tools

### Map File Analysis
- **Contents**
  - Memory usage summary
  - Section addresses and sizes
  - Symbol addresses (functions, variables)
  - Cross-reference
  
- **Use Cases**
  - Check memory consumption
  - Find large functions/data
  - Verify section placement
  - Debug linker issues

### ELF File Format
- **Structure**
  - ELF Header (magic number, entry point)
  - Program Headers (segments for loading)
  - Section Headers (.text, .data, .bss, etc.)
  - Symbol Table (functions, variables)
  - Debug Info (DWARF format)
  
- **Executable and Linkable Format**
  - Standard for Unix/Linux
  - Contains debug information

### objdump
- **Functions**
  - Disassemble code: `objdump -d file.elf`
  - Show headers: `objdump -h file.elf`
  - Show all info: `objdump -x file.elf`
  
- **Use Cases**
  - Verify generated assembly
  - Check optimization
  - Reverse engineering

### readelf
- **Functions**
  - Display ELF structure: `readelf -h file.elf`
  - Show sections: `readelf -S file.elf`
  - Show symbols: `readelf -s file.elf`
  
- **More Detailed Than objdump**
  - ELF-specific information

### nm (Name List)
- **Function**
  - List symbols in object/executable
  - `nm file.o` or `nm file.elf`
  
- **Symbol Types**
  - `T` = text (code)
  - `D` = initialized data
  - `B` = bss (uninitialized)
  - `U` = undefined (external)
  
- **Use Cases**
  - Find function/variable addresses
  - Check if symbol is defined
  - Resolve linking errors

### size
- **Function**
  - Show section sizes
  - `size file.elf`
  
- **Output**
  - text, data, bss sizes
  - Total memory usage
  
- **Quick Memory Check**
  - Flash = text + data
  - RAM = data + bss (+ stack + heap)

## Build Automation

### Makefile
- **Purpose**
  - Automate build process
  - Dependency management
  - Incremental compilation
  
- **Key Concepts**
  - Targets, dependencies, recipes
  - Variables (CC, CFLAGS, LDFLAGS)
  - Automatic variables ($@, $<, $^)
  - Pattern rules (%.o: %.c)

### CMake
- **Modern Build System Generator**
  - Cross-platform
  - Generates Makefiles, Ninja, IDE projects
  
- **Benefits**
  - Better dependency tracking
  - Out-of-source builds
  - Package management (find_package)
  
- **CMakeLists.txt**
  - Project configuration
  - Target definition
  - Compiler flags

## Debugging

### Semihosting
- **Concept**
  - Target uses debugger connection for I/O
  - `printf()` output to debug console
  
- **Advantages**
  - Easy debugging without UART
  - File I/O to host PC
  
- **Disadvantages**
  - Very slow (interrupts on every character)
  - Requires debugger connection
  - Hangs without debugger

### GDB (GNU Debugger)

#### Basic Commands
- **Control**
  - `break` / `b`: set breakpoint
  - `continue` / `c`: resume execution
  - `step` / `s`: step into function
  - `next` / `n`: step over function
  - `finish`: run until function return
  
- **Inspection**
  - `print` / `p`: print variable
  - `info registers`: show register values
  - `backtrace` / `bt`: show call stack
  - `disassemble`: show assembly
  
- **Memory**
  - `x/nfu addr`: examine memory
  - Example: `x/10x 0x20000000` (10 hex words)

### Breakpoints

#### Hardware Breakpoint
- **Characteristics**
  - Uses CPU comparator hardware
  - Limited number (typically 4-6)
  - Can break in FLASH (ROM)
  - No code modification
  
- **Use When**
  - Breaking in read-only memory
  - Critical performance path

#### Software Breakpoint
- **Characteristics**
  - Replaces instruction with breakpoint instruction (BKPT)
  - Unlimited in RAM
  - Cannot use in FLASH (if not erasable)
  
- **Use When**
  - Many breakpoints needed
  - RAM code execution

### Core Dump
- **Post-Mortem Debugging**
  - Memory snapshot at crash
  - Registers, stack, variables
  
- **Analysis**
  - Load dump in GDB
  - Examine state without re-running
  - Find crash root cause

## Testing & Validation

### Assertion (assert())
- **Purpose**
  - Runtime condition checking
  - Catch logic errors early
  
- **Usage**
```c
assert(ptr != NULL);
assert(value < MAX_SIZE);
```

- **Embedded Considerations**
  - Define custom assert handler
  - Log to flash before reset
  - Toggle LED or GPIO
  
- **Release Builds**
  - Disable with `NDEBUG`
  - Or keep critical assertions

### Static Analysis
- **Concept**
  - Analyze code without execution
  - Find bugs, vulnerabilities, violations
  
- **Tools**
  - Cppcheck
  - Clang Static Analyzer
  - PC-Lint / FlexeLint
  - Coverity
  
- **MISRA-C Compliance**
  - Safety-critical coding standard
  - Automotive, medical, aerospace
  - Enforces safe subset of C

## Binary Formats

### Intel Hex File (.hex)
- **Text Format**
  - ASCII representation
  - Contains address and data
  - Checksum for integrity
  
- **Structure**
  - `:LLAAAATT[DD...]CC`
  - LL = length, AAAA = address, TT = type, DD = data, CC = checksum
  
- **Advantages**
  - Portable, readable
  - Multiple memory regions
  - Bootloader friendly

### Binary File (.bin)
- **Raw Binary**
  - Pure data dump
  - No address information
  - No metadata
  
- **Usage**
  - Simplest format
  - Direct flash programming
  - Assumes fixed start address

### Conversion
- **objcopy**
  - `objcopy -O ihex file.elf file.hex`
  - `objcopy -O binary file.elf file.bin`

## Advanced Topics

### XIP (Execute In Place)
- **Concept**
  - Execute code directly from FLASH
  - No copy to RAM
  
- **Benefits**
  - Save RAM space
  - Faster boot time
  
- **Requirements**
  - Fast FLASH access
  - Memory-mapped FLASH
  
- **Limitations**
  - FLASH slower than RAM
  - Wear leveling concerns

## Measurement Tools

### Oscilloscope
- **Trigger Settings**
  - Edge trigger (rising/falling)
  - Level trigger (voltage threshold)
  - Pulse width trigger
  
- **Measurements**
  - Voltage levels
  - Timing (pulse width, frequency)
  - Signal integrity
  
- **Debug Technique**
  - Toggle GPIO in code
  - Measure execution time

### Logic Analyzer
- **Sampling Rate Rule**
  - Minimum 4× signal frequency
  - Recommended 10× for reliable capture
  
- **Use Cases**
  - Multi-channel digital signals
  - Protocol decoding (SPI, I2C, UART)
  - Timing analysis
  
- **Advantages Over Scope**
  - More channels (8, 16, 32+)
  - Protocol-aware

## Interview "Killer Question"

### "Most Difficult Debugging Experience?"
**Prepare a STAR Story:**

- **Situation**
  - Context and complexity
  
- **Task**
  - What needed to be found/fixed
  
- **Action**
  - Hypothesis → Verification approach
  - Tools used (scope, logic analyzer, GDB)
  - Techniques (binary search, printf debugging, assertions)
  
- **Result**
  - Root cause identified (e.g., race condition, timing issue, hardware fault)
  - Solution implemented
  - Lessons learned

**Example Topics:**
- Heisenbug (disappears when debugging)
- Race condition in interrupt + task
- Stack overflow from recursion
- Watchdog reset loop
- Hardware signal integrity issue

## Interview Practice
- [ ] Explain complete build process
- [ ] Read and explain linker script
- [ ] Analyze map file for memory optimization
- [ ] Use GDB to debug crashed program
- [ ] Compare .hex vs .bin formats
- [ ] Describe hardware vs software breakpoints
- [ ] Prepare detailed debugging story
