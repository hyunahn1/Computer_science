# 13.3 Debugging & Profiling

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
