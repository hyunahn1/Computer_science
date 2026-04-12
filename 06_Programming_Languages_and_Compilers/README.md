# Programming Languages & Runtime

## Overview
How programs are executed and organized at runtime: compilation vs interpretation, memory layout, type systems, and concurrency models. Bridges **01_Operating_Systems** (processes/threads) with **day-to-day language behavior** in interviews.

## Course Structure

### [9.1 Execution Models](./9.1_Execution_Models/)
- Compilation, interpretation, JIT (high level)
- Frontend vs backend of a compiler (lexer, parser, IR)
- **Advanced Compiler Theory**: AST (Abstract Syntax Tree), IR (Intermediate Representation like LLVM IR)
- Linking and dynamic libraries (intuition)

### [9.2 Memory & Runtime Layout](./9.2_Memory_and_Runtime_Layout/)
- Stack vs heap
- Call stack, stack frames, stack overflow
- Heap allocation, fragmentation (conceptual)
- Garbage collection: tracing, generations, stop-the-world (language-agnostic)
- **Advanced GC Algorithms**: Tri-color marking, G1GC, ZGC (Concurrent & parallel collection)
- Manual memory: RAII, smart pointers (if applicable to your stack)

### [9.3 Type Systems](./9.3_Type_Systems/)
- Static vs dynamic typing
- Strong vs weak (informal usage — use carefully in interviews)
- Nominal vs structural typing
- Parametric polymorphism (generics)
- Subtyping, variance (awareness for JVM/TS interviews)

### [9.4 Concurrency in Languages](./9.4_Concurrency_in_Languages/)
- Threads vs green threads / M:N models
- Async/await: cooperative scheduling, blocking the event loop
- Coroutines, actors (where they appear)
- Compare to OS threads (**01_Operating_Systems**)

## Study Approach
1. Pick **one** primary language and map these topics to its docs
2. Second language: contrast memory and concurrency model

## Interview Preparation
- Explain stack overflow vs heap exhaustion
- Why GC pauses happen and how to mitigate (measure first)
- How async avoids blocking without creating unbounded threads
