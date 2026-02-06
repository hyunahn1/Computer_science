# 5.2 Number Theory & Cryptography

## Prime Numbers

### Definition
- **Prime Number (소수)**
  - Natural number > 1
  - Only divisible by 1 and itself
  - No other factors
  
- **Examples**: 2, 3, 5, 7, 11, 13, 17, 19, 23, 29...

### Prime Testing

#### Sieve of Eratosthenes
- **Algorithm** to find all primes up to N
  1. List numbers from 2 to N
  2. Mark 2 as prime, cross out multiples of 2
  3. Find next unmarked number, mark as prime
  4. Cross out its multiples
  5. Repeat

- **Complexity**: O(N log log N)

### Prime Factorization (소인수분해)
- **Fundamental Theorem of Arithmetic**
  - Every integer > 1 has unique prime factorization
  - 60 = 2² × 3 × 5
  
- **Importance**
  - RSA security relies on difficulty of factoring large numbers
  - Hard to factor N = p × q when p, q are huge primes

## GCD & LCM

### Greatest Common Divisor (최대공약수)

#### Euclidean Algorithm
```python
def gcd(a, b):
    while b != 0:
        a, b = b, a % b
    return a
```

- **Principle**: gcd(a, b) = gcd(b, a % b)
- **Complexity**: O(log min(a, b))

### Least Common Multiple (최소공배수)
```
lcm(a, b) = (a × b) / gcd(a, b)
```

- **Use Cases**
  - Synchronization problems
  - Periodic events

## Modular Arithmetic

### Properties
```
(a + b) mod n = ((a mod n) + (b mod n)) mod n
(a × b) mod n = ((a mod n) × (b mod n)) mod n
(a - b) mod n = ((a mod n) - (b mod n) + n) mod n
```

### Congruence (합동식)
- **Notation**: a ≡ b (mod n)
- **Meaning**: a and b have same remainder when divided by n
- **Equivalently**: n | (a - b)

### Modular Inverse
- **Definition**: a × a⁻¹ ≡ 1 (mod n)
- **Exists** if and only if gcd(a, n) = 1
- **Extended Euclidean Algorithm** to find

## Advanced Number Theory

### Fermat's Little Theorem
```
If p is prime and gcd(a, p) = 1:
a^(p-1) ≡ 1 (mod p)
```

- **Application**
  - Fast modular exponentiation
  - Finding modular inverse: a⁻¹ ≡ a^(p-2) (mod p)
  - Primality testing (Fermat test)

### Euler's Totient Function (φ)
- **φ(n)**: Count of positive integers ≤ n that are coprime to n
  
- **Examples**
  - φ(10) = 4 (1, 3, 7, 9)
  - φ(p) = p - 1 (if p is prime)
  - φ(p × q) = (p-1)(q-1) (if p, q are distinct primes)
  
- **RSA Core**: φ(N) where N = p × q

### Chinese Remainder Theorem (CRT)
- **Problem**: Solve system of congruences
  ```
  x ≡ a₁ (mod n₁)
  x ≡ a₂ (mod n₂)
  ...
  ```
  
- **Condition**: n₁, n₂, ... must be pairwise coprime
- **Result**: Unique solution modulo N = n₁ × n₂ × ...

## RSA Cryptography

### Mathematical Principle
1. **Key Generation**
   - Choose two large primes p, q
   - N = p × q (public)
   - φ(N) = (p-1)(q-1) (secret)
   - Choose e with gcd(e, φ(N)) = 1 (public exponent)
   - Compute d ≡ e⁻¹ (mod φ(N)) (private exponent)
   
2. **Encryption**: C = M^e (mod N)
3. **Decryption**: M = C^d (mod N)

### Security
- **Hard to factor N** into p and q
- If factoring is easy, can compute φ(N) and find d
- Modern RSA uses 2048+ bit keys

## Hash Functions

### Mathematical Requirements
1. **One-way (일방향)**
   - Easy to compute H(m)
   - Hard to find m from H(m)
   
2. **Collision Resistance**
   - Hard to find m₁ ≠ m₂ with H(m₁) = H(m₂)
   
3. **Fixed Output**
   - Arbitrary input size → fixed hash size
   
- **Examples**: SHA-256, SHA-3

## Random Number Generation

### Pseudo-Random Number Generator (PRNG)
- **Deterministic**
  - Seed determines entire sequence
  - Same seed → same sequence
  
- **Not truly random**
  - But appears random
  - Fast, reproducible

### Linear Congruential Generator (LCG)
```
X_{n+1} = (a × X_n + c) mod m
```

- **Parameters**
  - X₀: Seed
  - a: Multiplier
  - c: Increment
  - m: Modulus
  
- **Used in**: `rand()` function (C standard library)

- **Weakness**: Predictable, not cryptographically secure

## Bit Operations & 2's Complement

### Two's Complement
```
-x = ~x + 1
```

- **Why Important**
  - Unified addition/subtraction circuit
  - No separate logic for negative numbers
  
- **Range** (n-bit):
  - -2^(n-1) to 2^(n-1) - 1
  - 8-bit: -128 to 127

## Error Detection Codes

### Parity Bit
- **Even Parity**: XOR of all bits = 0
- **Odd Parity**: XOR of all bits = 1
- **Detection**: Single-bit error
- **Cannot correct**

### CRC (Cyclic Redundancy Check)
- **Polynomial Division**
  - Treat data as polynomial
  - Divide by generator polynomial
  - Remainder = CRC checksum
  
- **Properties**
  - Detects burst errors
  - Used in Ethernet, ZIP, PNG
  
- **Common**: CRC-32 (Ethernet)

## Finite Fields (Galois Fields)

### Definition
- **Finite set** with addition, subtraction, multiplication, division (except ÷0)
- **Notation**: GF(p^n)
  - GF(2): {0, 1} with XOR and AND
  - GF(256): Used in AES encryption
  
### Applications
- AES encryption
- Reed-Solomon error correction (QR codes, CDs)
- Cryptographic protocols

## Fast Exponentiation

### Binary Exponentiation
```python
def power(x, n, mod):
    result = 1
    x = x % mod
    while n > 0:
        if n % 2 == 1:
            result = (result * x) % mod
        x = (x * x) % mod
        n = n // 2
    return result
```

- **Complexity**: O(log n)
- **Used in**: RSA, Diffie-Hellman

## Diophantine Equations

### Definition
- **Integer solutions only**
- Linear: ax + by = c
  
### Extended Euclidean Algorithm
- Finds x, y such that ax + by = gcd(a, b)
- Solution exists if gcd(a, b) | c

## Gray Code

### Mathematical Generation
```python
def binary_to_gray(n):
    return n ^ (n >> 1)
```

### Properties
- Adjacent codes differ by 1 bit
- Used in rotary encoders (minimize errors)
- Karnaugh maps

## Interview Practice
- [ ] Implement Euclidean algorithm for GCD
- [ ] Apply Fermat's Little Theorem for modular inverse
- [ ] Explain RSA key generation and encryption
- [ ] Implement fast exponentiation
- [ ] Solve system with Chinese Remainder Theorem
- [ ] Calculate Euler's totient function
- [ ] Implement LCG random number generator
- [ ] Understand CRC calculation
- [ ] Convert binary to Gray code
- [ ] Explain two's complement representation
