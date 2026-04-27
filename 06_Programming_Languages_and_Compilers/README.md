# Programming Languages & Runtime

## Overview
How programs are executed and organized at runtime: compilation vs interpretation, memory layout, type systems, and concurrency models. Bridges **05_Operating_Systems** (processes/threads) with **day-to-day language behavior** in interviews.

## Course Structure

### [6.1 Execution Models](./6.1_Execution_Models/)
- Compilation, interpretation, JIT (high level)
- Frontend vs backend of a compiler (lexer, parser, IR)
- **Advanced Compiler Theory**: AST (Abstract Syntax Tree), IR (Intermediate Representation like LLVM IR)
- Linking and dynamic libraries (intuition)

### [6.2 Memory & Runtime Layout](./6.2_Memory_and_Runtime_Layout/)
- Stack vs heap
- Call stack, stack frames, stack overflow
- Heap allocation, fragmentation (conceptual)
- Garbage collection: tracing, generations, stop-the-world (language-agnostic)
- **Advanced GC Algorithms**: Tri-color marking, G1GC, ZGC (Concurrent & parallel collection)
- Manual memory: RAII, smart pointers (if applicable to your stack)

### [6.3 Type Systems](./6.3_Type_Systems/)
- Static vs dynamic typing
- Strong vs weak (informal usage — use carefully in interviews)
- Nominal vs structural typing
- Parametric polymorphism (generics)
- Subtyping, variance (awareness for JVM/TS interviews)

### [6.4 Concurrency in Languages](./6.4_Concurrency_in_Languages/)
- Threads vs green threads / M:N models
- Async/await: cooperative scheduling, blocking the event loop
- Coroutines, actors (where they appear)
- Compare to OS threads (**05_Operating_Systems**)

## Study Approach
1. Pick **one** primary language and map these topics to its docs
2. Second language: contrast memory and concurrency model

## Interview Preparation
- Explain stack overflow vs heap exhaustion
- Why GC pauses happen and how to mitigate (measure first)
- How async avoids blocking without creating unbounded threads

## Advanced Topics to Add

- Compiler frontend: lexing, parsing, AST, symbol tables, scope, name resolution, diagnostics.
- Type systems: inference, unification, variance, subtyping, generics, soundness intuition.
- IR and optimization: SSA, dataflow analysis, constant propagation, DCE, inlining, escape analysis.
- Code generation: instruction selection, register allocation, calling convention, ABI, linking.
- Runtime: stack frames, closures, exceptions, GC algorithms, JIT, FFI, memory model.
- Concurrency: async runtimes, structured concurrency, actors, green threads, data-race semantics.

## Expert Depth Checklist
- [ ] Describe the formal or implementation model: syntax, semantics, type rules, runtime representation, or execution strategy.
- [ ] Build a small artifact: parser, interpreter, bytecode VM, type checker, allocator experiment, or concurrency demo.
- [ ] Explain compile-time vs runtime responsibilities and where errors are detected.
- [ ] Inspect generated IR, bytecode, assembly, or runtime traces when possible.
- [ ] Compare two languages or runtimes and identify the trade-off, not just the surface syntax.
- [ ] Document a failure mode: type unsoundness, memory leak, GC pause, data race, deadlock, or event-loop blocking.
