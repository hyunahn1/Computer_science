# Developer Tools & Workflow

## Overview
Day-to-day engineering tools: version control collaboration, shell and regular expressions, debugging and profiling. Pairs with **08_Software_Engineering** (process) — this section is **hands-on mechanics**.

## Course Structure

### [13.1 Git & Collaboration](./13.1_Git_and_Collaboration/)
- Repositories, commits, branches, merges
- Merge vs rebase (trade-offs, golden rules)
- Conflict resolution
- Cherry-pick, revert vs reset (safe practices)
- Pull requests: small diffs, meaningful messages

### [13.2 Shell & Regular Expressions](./13.2_Shell_and_Regular_Expressions/)
- Shell basics: pipes, redirection, exit codes
- Scripting: variables, functions, error handling (`set -euo pipefail`)
- Regex: literals, character classes, quantifiers, groups
- grep/sed/awk at a useful subset (not mastery of entire awk)

### [13.3 Debugging & Profiling](./13.3_Debugging_and_Profiling/)
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
