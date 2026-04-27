# 0.3 Debugging & Profiling

## Debugging
- Reproduce first; minimize the repro case
- Binary search the fault (which commit introduced it?)
- Debugger: step in/over/out, watch, evaluate expressions
- Logging vs debugger trade-offs

## Performance
- Measure before optimizing (profile, don't guess)
- CPU flame graphs (conceptual)
- Wall time vs CPU time

## Memory
- Allocation profiling: hot allocation sites
- Heap dumps for leak suspicion (high level)

## Production-Like Debugging
- Correlation IDs across logs
- Sampling and rate-limited error logging

## Study Materials
- [ ] Debug one null pointer / segfault with a debugger
- [ ] Profile one slow function; report top cost

## Practice Problems
- [ ] Heisenbug from uninitialized memory vs race — how to tell?
- [ ] When is printf-debugging better than breakpoints?

## Expert Depth Checklist
- [ ] Explain the data model behind the tool, not only the commands: Git objects/refs, shell process model, regex automata, debugger symbols, profiler sampling.
- [ ] Reproduce a realistic failure: bad rebase, merge conflict, broken pipeline, quoting bug, regex backtracking issue, crash, or performance regression.
- [ ] Use primary documentation (`git help`, shell manual, debugger docs, profiler docs) and record exact command evidence.
- [ ] Build one repeatable workflow script with clear error handling, logging, and rollback or recovery notes.
- [ ] Compare at least two approaches and justify trade-offs in correctness, auditability, speed, and team risk.
- [ ] Produce a short incident-style note: symptom, hypothesis, command evidence, root cause, fix, and prevention.
