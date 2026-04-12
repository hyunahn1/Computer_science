# 3.2 Bit Manipulation & Optimization

## XOR Operations

### XOR Characteristics
- **Truth Table**
  - Same bits: 0
  - Different bits: 1
  - `A ^ A = 0`
  - `A ^ 0 = A`
  
- **Commutative & Associative**
  - `A ^ B = B ^ A`
  - `(A ^ B) ^ C = A ^ (B ^ C)`

### XOR Applications

#### Swap Without Temporary Variable
```c
a = a ^ b;
b = a ^ b;  // b = (a ^ b) ^ b = a
a = a ^ b;  // a = (a ^ b) ^ a = b
```

#### Find Single Number (Others Appear Twice)
```c
int findSingle(int arr[], int n) {
    int result = 0;
    for (int i = 0; i < n; i++)
        result ^= arr[i];
    return result;  // All pairs cancel out
}
```

#### Encryption (Simple XOR Cipher)
```c
encrypted = data ^ key;
decrypted = encrypted ^ key;  // data
```

## Power-of-Two Operations

### `x & (x - 1)` Trick
- **Effect**
  - Clears rightmost 1-bit
  - `x = 12 (1100)` → `x - 1 = 11 (1011)` → `x & (x-1) = 8 (1000)`

### Check if Power of Two
```c
bool isPowerOfTwo(int x) {
    return (x > 0) && ((x & (x - 1)) == 0);
}
```
- **Logic**
  - Power of 2 has exactly one 1-bit
  - Clearing it results in 0

### Count 1-Bits (Brian Kernighan's Algorithm)
```c
int countSetBits(int x) {
    int count = 0;
    while (x) {
        x &= (x - 1);  // Clear rightmost 1-bit
        count++;
    }
    return count;
}
```
- **Or use**: `__builtin_popcount(x)` (GCC)

## Odd/Even & Arithmetic

### Odd/Even Check
```c
if (x & 1)  // Odd
else        // Even
```
- **Faster than**: `x % 2`

### Multiply/Divide by 2
```c
x << 1  // Multiply by 2
x >> 1  // Divide by 2 (for positive)
```
- **Shift is faster** than `*` or `/`

### Modulo Power of 2
```c
i % 8   // Slow
i & 7   // Fast (for i >= 0)
```
- **General rule**: `i % (2^n) = i & (2^n - 1)`

## Bit Masking

### Concept
- **Bit Mask**
  - Pattern to select/modify specific bits
  
- **Operations**
  - Set bit: `x | (1 << n)`
  - Clear bit: `x & ~(1 << n)`
  - Toggle bit: `x ^ (1 << n)`
  - Check bit: `(x >> n) & 1`

### Extract Lower Bits
```c
uint8_t lower_byte = value & 0xFF;  // Extract lower 8 bits
```

### Bitmap Data Structure
- **Purpose**
  - Store boolean flags efficiently
  - 1 bit per flag (vs 8 bits for bool)
  
- **Memory Savings**
  - 1/8 memory usage
  - Cache friendly
  
- **Use Cases**
  - Large flag arrays
  - Set membership (bloom filter)

## Two's Complement

### Negative Number Representation
- **Two's Complement**
  - Invert all bits (one's complement)
  - Add 1
  
- **Example**: -5 (8-bit)
  ```
  5  = 00000101
  ~5 = 11111010  (one's complement)
  +1 = 11111011  (two's complement = -5)
  ```

- **Advantages**
  - Only one representation of zero
  - Addition/subtraction work the same

## Endianness

### Byte Order
- **Big-Endian**
  - Most significant byte first
  - Network byte order
  
- **Little-Endian**
  - Least significant byte first
  - x86, ARM (usually)

### Endian Conversion
```c
uint32_t swap_endian(uint32_t x) {
    return ((x >> 24) & 0xFF)       |
           ((x >> 8)  & 0xFF00)     |
           ((x << 8)  & 0xFF0000)   |
           ((x << 24) & 0xFF000000);
}
```

### Or use builtin
```c
__builtin_bswap32(x);  // GCC/Clang
```

## Shift Operations

### Unsigned vs Signed Right Shift
- **Logical Shift (unsigned)**
  - `>>` fills with 0
  - `unsigned int x = 0x80 >> 1` → `0x40`
  
- **Arithmetic Shift (signed)**
  - `>>` fills with sign bit
  - `int x = -8 >> 1` → `-4` (sign extended)
  
- **Portability Warning**
  - Right shift of signed is implementation-defined

## Advanced Bit Tricks

### Addition Without `+` Operator
```c
int add(int a, int b) {
    while (b != 0) {
        int carry = (a & b) << 1;
        a = a ^ b;  // Sum without carry
        b = carry;
    }
    return a;
}
```
- **XOR**: sum without carry
- **AND + shift**: carry calculation

### `~0` Value
- **All bits set to 1**
  - `signed`: -1
  - `unsigned`: UINT_MAX (0xFFFFFFFF for 32-bit)

### Gray Code
- **Definition**
  - Consecutive values differ by only 1 bit
  
- **Binary to Gray**
  ```c
  gray = binary ^ (binary >> 1);
  ```
  
- **Use Cases**
  - Rotary encoders (minimize error)
  - Karnaugh maps
  - Error correction

### Find MSB (Most Significant Bit)
```c
int msb_position(unsigned int x) {
    return 31 - __builtin_clz(x);  // Count leading zeros
}
```

### Absolute Value Without Branch
```c
int abs_no_branch(int x) {
    int mask = x >> 31;  // All 1s if negative, all 0s if positive
    return (x + mask) ^ mask;
}
```

### Extract Specific Bit Range
```c
// Extract bits [start, end] from value
unsigned extract_bits(unsigned value, int start, int end) {
    unsigned mask = ((1 << (end - start + 1)) - 1) << start;
    return (value & mask) >> start;
}
```

## Bit Fields (Struct)

### Usage
```c
struct Register {
    unsigned int enable : 1;
    unsigned int mode   : 2;
    unsigned int value  : 5;
};
```

### Portability Concerns
- **Bit order**
  - Compiler-dependent (MSB-first or LSB-first)
  - Not portable across platforms
  
- **Packing**
  - Alignment rules vary
  
- **Recommendation**
  - Use for space-critical embedded code
  - Document compiler assumptions
  - Consider manual masking for portability

## 64-bit in 32-bit Environment

### Double-Width Operations
- **Problem**
  - 64-bit value needs two 32-bit registers
  - Operations are slower
  
- **Implications**
  - Addition requires carry handling
  - Multiplication much slower
  - Avoid 64-bit when possible on 32-bit MCU

## Interview Practice
- [ ] Implement bit manipulation functions (set/clear/toggle)
- [ ] Swap variables without temp using XOR
- [ ] Check if power of two
- [ ] Count set bits (multiple methods)
- [ ] Implement bitmap for 1000 flags
- [ ] Explain two's complement
- [ ] Convert endianness
- [ ] Find missing number in array [1..N]
- [ ] Reverse bits in integer
- [ ] Find two non-repeating elements
