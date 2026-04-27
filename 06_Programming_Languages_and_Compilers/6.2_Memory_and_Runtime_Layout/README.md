# 6.2 Memory & Runtime Layout

## Stack
- Stack frames: locals, return addresses
- Stack overflow: recursion depth, large stack allocations
- Thread per stack

## Heap
- Dynamic lifetime objects
- Allocator strategies (arena, pool — awareness)
- Fragmentation (internal vs external)

## Garbage Collection (Conceptual)
- Reachability from roots
- Mark-sweep, copying, generational hypothesis
- Write barriers, incremental/concurrent GC (names only)
- Tuning: pause time vs throughput

## Manual Memory (When Relevant)
- malloc/free pitfalls
- RAII, determinism of destruction

## Study Materials
- [ ] Draw stack frames for a small recursive function
- [ ] One profiling session: allocation hot spot

## Practice Problems
- [ ] Heap vs stack for large temporary buffer?
- [ ] Explain generational GC in one paragraph

## Expert Depth Checklist
- [ ] Describe the formal or implementation model: syntax, semantics, type rules, runtime representation, or execution strategy.
- [ ] Build a small artifact: parser, interpreter, bytecode VM, type checker, allocator experiment, or concurrency demo.
- [ ] Explain compile-time vs runtime responsibilities and where errors are detected.
- [ ] Inspect generated IR, bytecode, assembly, or runtime traces when possible.
- [ ] Compare two languages or runtimes and identify the trade-off, not just the surface syntax.
- [ ] Document a failure mode: type unsoundness, memory leak, GC pause, data race, deadlock, or event-loop blocking.
