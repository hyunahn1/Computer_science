# 4.1 Automata & Formal Languages

## Finite Automata
- DFA: deterministic transitions
- NFA: epsilon transitions; subset construction (awareness)
- Regular expressions and regular languages (connection)

## Pushdown Automata & CFG
- Context-free grammars: parse trees, ambiguity
- Why some languages need a stack (balanced parentheses intuition)

## Turing Machines
- Tape, states, read/write/move
- Church-Turing thesis (informal statement)

## Hierarchy (Awareness)
- Regular ⊂ Context-free ⊂ Decidable ⊂ Recognizable (vary by textbook; know your chart)

## Study Materials
- [ ] Build a DFA for a small language (e.g., even number of a's)
- [ ] Show a CFG for simple nested structure

## Practice Problems
- [ ] Why can't a DFA count unbounded nesting?
- [ ] Regex in your language: what can it not express?

## Expert Depth Checklist
- [ ] Write formal definitions before giving intuition.
- [ ] Construct or transform a machine, grammar, automaton, or reduction by hand.
- [ ] Prove language membership, non-membership, closure, decidability, or hardness when applicable.
- [ ] Identify what resource is being bounded: time, space, states, stack, nondeterminism, or oracle access.
- [ ] Give a counterexample to an overly broad claim.
- [ ] Connect theory to implementation: regex engines, parsers, type checkers, SAT solvers, or exponential search.
