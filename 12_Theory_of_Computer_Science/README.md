# Theory of Computer Science

## Overview
Formal models of computation and complexity: automata, grammars, decidability, and the P vs NP landscape. Depth here is **interview-practical** — enough to discuss reductions and why brute force explodes, not a full theory course.

## Course Structure

### [12.1 Automata & Formal Languages](./12.1_Automata_and_Formal_Languages/)
- Finite automata (DFA, NFA) and regular languages
- Pushdown automata and context-free languages (CFGs)
- Turing machines (informal)
- Chomsky hierarchy (placement awareness)

### [12.2 Computability & Complexity](./12.2_Computability_and_Complexity/)
- Decidable vs undecidable (Halting problem intuition)
- P, NP, NP-hard, NP-complete definitions
- Polynomial-time reductions (conceptual)
- Classic NP-complete problems: SAT, 3-SAT, Vertex Cover (names)
- Approximation and heuristics when exact is intractable

## Study Approach
1. Connect each class to something you implement (regex engine → NFA)
2. For interviews: practice stating P vs NP without hand-waving

## Interview Preparation
- "Why is brute force O(2^n) unacceptable at scale?"
- Give an example of a problem in NP you can verify in poly time
- When would you use an approximation algorithm?
