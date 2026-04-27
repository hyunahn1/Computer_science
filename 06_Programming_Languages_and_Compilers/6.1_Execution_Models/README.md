# 6.1 Execution Models

## From Source to Running Program
- Lexing, parsing, AST
- IR and optimizations (brief)
- Machine code vs bytecode vs source interpretation

## Compilation vs Interpretation vs JIT
- AOT compilation trade-offs
- Interpreter startup vs peak performance
- JIT: profiling, deoptimization (awareness)

## Linking
- Static vs dynamic linking
- Symbol resolution, name mangling (C++ awareness)

## Study Materials
- [ ] Trace one `hello world` through compile + link in your toolchain
- [ ] Read your language VM spec section (if applicable)

## Practice Problems
- [ ] Why might startup time differ: interpreted vs JIT vs native?
- [ ] What is a shared library load failure symptom?

## Expert Depth Checklist
- [ ] Describe the formal or implementation model: syntax, semantics, type rules, runtime representation, or execution strategy.
- [ ] Build a small artifact: parser, interpreter, bytecode VM, type checker, allocator experiment, or concurrency demo.
- [ ] Explain compile-time vs runtime responsibilities and where errors are detected.
- [ ] Inspect generated IR, bytecode, assembly, or runtime traces when possible.
- [ ] Compare two languages or runtimes and identify the trade-off, not just the surface syntax.
- [ ] Document a failure mode: type unsoundness, memory leak, GC pause, data race, deadlock, or event-loop blocking.
