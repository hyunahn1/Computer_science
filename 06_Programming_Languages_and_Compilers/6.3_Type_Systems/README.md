# 6.3 Type Systems

## Classification (Interview-Safe Wording)
- Static checking at compile time vs dynamic checks at runtime
- "Strong/weak" — define what you mean or avoid the buzzwords

## Polymorphism
- Parametric (generics)
- Subtype polymorphism (inheritance / interfaces)
- Ad-hoc (overloading)

## Type Inference
- Hindley-Milner intuition vs local inference (Java `var`, etc.)

## Advanced (Skim)
- Variance: covariance, contravariance (collections, functions)
- Nullable types and option types

## Study Materials
- [ ] Same API modeled with generics vs interface in your language
- [ ] One bug caught by static types that would slip through dynamically

## Practice Problems
- [ ] Variance: why `List<Dog>` is not `List<Animal>` in Java
- [ ] When do you prefer structural typing?

## Expert Depth Checklist
- [ ] Describe the formal or implementation model: syntax, semantics, type rules, runtime representation, or execution strategy.
- [ ] Build a small artifact: parser, interpreter, bytecode VM, type checker, allocator experiment, or concurrency demo.
- [ ] Explain compile-time vs runtime responsibilities and where errors are detected.
- [ ] Inspect generated IR, bytecode, assembly, or runtime traces when possible.
- [ ] Compare two languages or runtimes and identify the trade-off, not just the surface syntax.
- [ ] Document a failure mode: type unsoundness, memory leak, GC pause, data race, deadlock, or event-loop blocking.
