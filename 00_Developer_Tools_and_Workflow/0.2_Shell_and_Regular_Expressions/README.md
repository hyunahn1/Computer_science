# 13.2 Shell & Regular Expressions

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
