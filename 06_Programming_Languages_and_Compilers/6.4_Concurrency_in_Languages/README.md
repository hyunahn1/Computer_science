# 9.4 Concurrency in Languages

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
