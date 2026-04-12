# 5.1 Discrete Mathematics & Logic

## Propositions & Logic

### Proposition (명제)
- **Definition**
  - Statement that is clearly either True or False
  - No ambiguity
  
- **Examples**
  - Valid: "2 + 2 = 4" (True), "5 > 10" (False)
  - Invalid: "This sentence is beautiful" (subjective)

### Logical Operators

#### AND (∧, Conjunction)
- True only if both operands are true
- `A ∧ B`

#### OR (∨, Disjunction)
- True if at least one operand is true
- `A ∨ B`

#### NOT (¬, Negation)
- Flips truth value
- `¬A`

#### XOR (⊕, Exclusive OR)
- True if exactly one operand is true
- `A ⊕ B = (A ∨ B) ∧ ¬(A ∧ B)`

### De Morgan's Laws
```
¬(A ∧ B) ≡ ¬A ∨ ¬B
¬(A ∨ B) ≡ ¬A ∧ ¬B
```

- **Programming Equivalent**
  ```c
  !(A && B) == !A || !B
  !(A || B) == !A && !B
  ```

- **Use Case**
  - Simplifying boolean expressions
  - Circuit optimization

### Truth Tables
- **Purpose**
  - List all possible input combinations
  - Determine output for each
  
- **Example: XOR**
  | A | B | A ⊕ B |
  |---|---|-------|
  | 0 | 0 |   0   |
  | 0 | 1 |   1   |
  | 1 | 0 |   1   |
  | 1 | 1 |   0   |

### Special Propositions

#### Tautology (항진명제)
- Always true
- Example: `A ∨ ¬A`

#### Contradiction (모순)
- Always false
- Example: `A ∧ ¬A`

### Conditional Statements

#### Implication (p → q)
- "If p, then q"
- False only when p is true and q is false

#### Contrapositive (대우)
- `¬q → ¬p`
- **Same truth value as original**
- Used in proofs

#### Converse
- `q → p`
- Not equivalent to original

#### Inverse
- `¬p → ¬q`
- Not equivalent to original

## Proof Techniques

### Mathematical Induction
1. **Base Case**: Prove for n = 1
2. **Inductive Hypothesis**: Assume true for n = k
3. **Inductive Step**: Prove true for n = k + 1

- **Example**: Prove 1 + 2 + ... + n = n(n+1)/2

### Pigeonhole Principle (비둘기집 원리)
- **Statement**
  - If N+1 items are placed into N boxes
  - At least one box contains 2+ items
  
- **Application**
  - Hash table collisions are inevitable
  - Birthday paradox
  - Proving existence without construction

## Sets

### Set Theory Basics

#### Set Definition
- Unordered collection of distinct elements
- No duplicates
- No order

#### Set vs List
| Set | List |
|-----|------|
| No order | Ordered |
| No duplicates | Duplicates allowed |
| `{1, 2, 3}` | `[1, 2, 1, 3]` |

### Set Operations

#### Subset (부분집합)
- A ⊆ B: Every element of A is in B
- **Number of subsets**: 2^n
  - Can represent with bitmask (0 to 2^n - 1)

#### Cartesian Product (카테시안 곱)
- A × B = {(a, b) | a ∈ A, b ∈ B}
- All ordered pairs
- **Database**: Cross Join principle

### Relations

#### Relation Properties
1. **Reflexive (반사)**: ∀a, (a, a) ∈ R
2. **Symmetric (대칭)**: (a, b) ∈ R → (b, a) ∈ R
3. **Transitive (추이)**: (a, b), (b, c) ∈ R → (a, c) ∈ R

- **Equivalence Relation**: All three properties

### Functions

#### Function Types
1. **Injective (단사, One-to-One)**
   - Different inputs → Different outputs
   - No two elements map to same value
   
2. **Surjective (전사, Onto)**
   - Every element in codomain is mapped
   - Range = Codomain
   
3. **Bijective (전단사, One-to-One Correspondence)**
   - Both injective and surjective
   - Perfect pairing
   - Invertible function

## Recursive Definitions

### Recurrence Relations (점화식)
- Define sequence in terms of previous terms

- **Fibonacci**
  ```
  f(0) = 0
  f(1) = 1
  f(n) = f(n-1) + f(n-2)
  ```

- **Factorial**
  ```
  n! = n × (n-1)!
  0! = 1
  ```

## Boolean Algebra

### Circuit Simplification
- **Karnaugh Map (K-Map)**
  - Visual method to minimize boolean expressions
  - Reduce gate count in digital circuits
  
- **Laws**
  - Identity: A ∧ 1 = A, A ∨ 0 = A
  - Null: A ∧ 0 = 0, A ∨ 1 = 1
  - Idempotent: A ∧ A = A, A ∨ A = A
  - Complement: A ∧ ¬A = 0, A ∨ ¬A = 1
  - Absorption: A ∨ (A ∧ B) = A

## Graph Theory

### Euler Path vs Hamilton Path

#### Euler Path (오일러 경로)
- **Traverse all edges exactly once**
- Continuous line drawing (한붓그리기)
- **Condition**: At most 2 vertices with odd degree

#### Hamilton Path (해밀턴 경로)
- **Visit all vertices exactly once**
- Traveling Salesman Problem (TSP)
- NP-complete problem

### Planar Graph (평면 그래프)
- **Definition**
  - Can be drawn without edge crossings
  
- **Applications**
  - Circuit design
  - PCB routing
  
- **Euler's Formula**
  - V - E + F = 2 (vertices, edges, faces)

### Tree (수학적 정의)
- **Properties**
  - Connected graph with no cycles
  - V = E + 1 (vertices = edges + 1)
  - Exactly one path between any two vertices

## Combinatorics

### Permutation vs Combination

#### Permutation (순열)
- **Order matters**
- nPr = n! / (n-r)!
- Example: Password, ranking

#### Combination (조합)
- **Order doesn't matter**
- nCr = n! / (r!(n-r)!)
- Example: Lottery, team selection

### Binomial Theorem
```
(a + b)^n = Σ (nCr) × a^(n-r) × b^r
```

- **Pascal's Triangle**
  - Row n contains coefficients for (a+b)^n
  - Each entry = sum of two above

### XOR Properties (Revisited)
- **Mathematical Properties**
  - A ⊕ A = 0
  - A ⊕ 0 = A
  - Commutative: A ⊕ B = B ⊕ A
  - Associative: (A ⊕ B) ⊕ C = A ⊕ (B ⊕ C)
  
- **Applications**
  - Swap without temp variable
  - Find single non-duplicate
  - Parity checking
  - Simple encryption

## Interview Practice
- [ ] Prove by mathematical induction
- [ ] Apply De Morgan's Laws to simplify expressions
- [ ] Count subsets and combinations
- [ ] Identify function type (injective/surjective/bijective)
- [ ] Solve problems using pigeonhole principle
- [ ] Determine if graph has Euler/Hamilton path
- [ ] Use XOR properties to solve problems
- [ ] Build truth table for complex expression
- [ ] Apply Cartesian product to database joins
- [ ] Prove relation is equivalence relation
