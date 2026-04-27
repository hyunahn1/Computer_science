# 6.4 Concurrency in Languages

## OS vs Language Threads
- 1:1 kernel threads vs M:N green threads
- Thread pool sizing heuristics (I/O vs CPU bound)

## Async / Await
- Event loop model (typical in JS, Python asyncio)
- Non-blocking I/O vs thread-per-request
- Blocking the loop: CPU work, sync I/O mistakes

## Structured Concurrency (Awareness)
- Tasks scoped to parent, cancellation propagation

## Other Models
- CSP / channels (Go-style intuition)
- Actors (Erlang/Akka intuition)
- Software transactional memory (rare; awareness)

## Study Materials
- [ ] Implement same I/O-bound workload with threads vs async
- [ ] Read stack trace from a deadlock in your language

## Practice Problems
- [ ] When would async not reduce thread count meaningfully?
- [ ] Compare mutex vs message passing for shared state

## Expert Depth Checklist
- [ ] Describe the formal or implementation model: syntax, semantics, type rules, runtime representation, or execution strategy.
- [ ] Build a small artifact: parser, interpreter, bytecode VM, type checker, allocator experiment, or concurrency demo.
- [ ] Explain compile-time vs runtime responsibilities and where errors are detected.
- [ ] Inspect generated IR, bytecode, assembly, or runtime traces when possible.
- [ ] Compare two languages or runtimes and identify the trade-off, not just the surface syntax.
- [ ] Document a failure mode: type unsoundness, memory leak, GC pause, data race, deadlock, or event-loop blocking.
