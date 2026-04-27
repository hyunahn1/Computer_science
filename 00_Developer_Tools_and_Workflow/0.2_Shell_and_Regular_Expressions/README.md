# 0.2 Shell & Regular Expressions

## Shell
- Paths, globbing vs regex (different!)
- Pipelines and stderr vs stdout
- Exit codes and `&&` / `||`
- Scripts: shebang, quoting (single vs double)
- Defensive bash: `set -euo pipefail` when appropriate

## Regular Expressions
- Literals, `.`, character classes `[]`, negation `^`
- Quantifiers `*`, `+`, `?`, `{n,m}`
- Anchors `^`, `$`, word boundaries `\b` (engine-dependent)
- Groups `()` and capturing vs non-capturing
- Greedy vs lazy matching

## Tools
- grep -E, ripgrep for search
- sed for simple transforms (care with portability)

## Study Materials
- [ ] Extract IPs or emails from a log file (toy data)
- [ ] Write a script that fails fast on undefined variables

## Practice Problems
- [ ] Why can catastrophic backtracking happen?
- [ ] Difference between glob `*.txt` and regex `.*\.txt`

## Expert Depth Checklist
- [ ] Explain the data model behind the tool, not only the commands: Git objects/refs, shell process model, regex automata, debugger symbols, profiler sampling.
- [ ] Reproduce a realistic failure: bad rebase, merge conflict, broken pipeline, quoting bug, regex backtracking issue, crash, or performance regression.
- [ ] Use primary documentation (`git help`, shell manual, debugger docs, profiler docs) and record exact command evidence.
- [ ] Build one repeatable workflow script with clear error handling, logging, and rollback or recovery notes.
- [ ] Compare at least two approaches and justify trade-offs in correctness, auditability, speed, and team risk.
- [ ] Produce a short incident-style note: symptom, hypothesis, command evidence, root cause, fix, and prevention.
