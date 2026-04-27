# Developer Tools & Workflow

## Overview
Day-to-day engineering tools: version control collaboration, shell and regular expressions, debugging and profiling. Pairs with **10_Software_Engineering_and_Architecture** (process) — this section is **hands-on mechanics**.

## Course Structure

### [0.1 Git & Collaboration](./0.1_Git_and_Collaboration/)
- Repositories, commits, branches, merges
- Merge vs rebase (trade-offs, golden rules)
- Conflict resolution
- Cherry-pick, revert vs reset (safe practices)
- Pull requests: small diffs, meaningful messages

### [0.2 Shell & Regular Expressions](./0.2_Shell_and_Regular_Expressions/)
- Shell basics: pipes, redirection, exit codes
- Scripting: variables, functions, error handling (`set -euo pipefail`)
- Regex: literals, character classes, quantifiers, groups
- grep/sed/awk at a useful subset (not mastery of entire awk)

### [0.3 Debugging & Profiling](./0.3_Debugging_and_Profiling/)
- Breakpoints, watch expressions, conditional breakpoints
- Core dumps / crash logs (platform-dependent)
- CPU profiling vs allocation profiling
- Time complexity vs wall time (measure before optimizing)

## Study Approach
1. Use CLI for one week for Git operations you usually do in GUI
2. Solve one log-parsing task with grep + regex

## Interview Preparation
- Explain merge vs rebase to a teammate
- How you diagnosed a memory leak or hot loop

## Advanced Topics to Add

- Git internals: object database, packfiles, refs, reflog, index, three trees, bisect, submodules, signed commits.
- Shell rigor: quoting, word splitting, globbing, process substitution, signals, traps, exit status propagation.
- Regex depth: automata intuition, backtracking engines, catastrophic backtracking, lookaround, capture cost.
- Debugging: symbols, stack unwinding, core dumps, sanitizers, watchpoints, time-travel/replay debugging awareness.
- Profiling: sampling vs instrumentation, flame graphs, CPU vs wall time, allocation profiling, benchmark noise.

## Expert Depth Checklist
- [ ] Explain the data model behind the tool, not only the commands: Git objects/refs, shell process model, regex automata, debugger symbols, profiler sampling.
- [ ] Reproduce a realistic failure: bad rebase, merge conflict, broken pipeline, quoting bug, regex backtracking issue, crash, or performance regression.
- [ ] Use primary documentation (`git help`, shell manual, debugger docs, profiler docs) and record exact command evidence.
- [ ] Build one repeatable workflow script with clear error handling, logging, and rollback or recovery notes.
- [ ] Compare at least two approaches and justify trade-offs in correctness, auditability, speed, and team risk.
- [ ] Produce a short incident-style note: symptom, hypothesis, command evidence, root cause, fix, and prevention.
