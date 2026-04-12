# 12.2 Computability & Complexity

## Computability
- Decidable languages vs recognizable
- Halting problem: diagonalization intuition (no proof required unless you want)

## Time Complexity Classes
- **P**: decision problems solvable in polynomial time
- **NP**: verifiable in polynomial time (certificate + verifier)
- **NP-hard**: every NP problem reduces to it
- **NP-complete**: in NP and NP-hard

## Reductions
- Polynomial-time reduction: A ≤p B
- Use: if B in P then A in P; if A is NP-hard then B is NP-hard

## Examples (Recognition)
- SAT, 3-SAT, CLIQUE, Vertex Cover, TSP (decision versions)

## Coping in Practice
- Heuristics, approximation algorithms, SAT solvers for small instances
- Parameterized complexity (awareness only)

## Study Materials
- [ ] State definitions of P and NP without reading notes
- [ ] Watch one short reduction sketch (3-SAT → CLIQUE style)

## Practice Problems
- [ ] Why is "find any solution" often same hardness as "decide existence"?
- [ ] Is sorting in NP? (trick: be precise about decision vs function problems)
