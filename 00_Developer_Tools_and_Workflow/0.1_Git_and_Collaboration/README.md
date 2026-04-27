# 0.1 Git & Collaboration

## Core Objects
- Blob, tree, commit, ref
- HEAD, branch, tag

## Daily Commands
- status, diff, add, commit, push, pull
- stash with message discipline

## Branching
- merge: merge commit vs fast-forward
- rebase: linear history; never rebase shared published branches (rule of thumb)
- interactive rebase: cleanup before review

## History Repair (Careful)
- revert (safe, shared-history friendly)
- reset --hard (dangerous on shared branches)

## Collaboration
- Pull request size and description
- Conventional commits (optional team standard)

## Study Materials
- [ ] Resolve a deliberate merge conflict
- [ ] Draw DAG for feature branch merged with merge vs rebase

## Practice Problems
- [ ] When is `git revert` preferred over `git reset`?
- [ ] Recover "lost" commit with reflog (practice in a toy repo)

## Expert Depth Checklist
- [ ] Explain the data model behind the tool, not only the commands: Git objects/refs, shell process model, regex automata, debugger symbols, profiler sampling.
- [ ] Reproduce a realistic failure: bad rebase, merge conflict, broken pipeline, quoting bug, regex backtracking issue, crash, or performance regression.
- [ ] Use primary documentation (`git help`, shell manual, debugger docs, profiler docs) and record exact command evidence.
- [ ] Build one repeatable workflow script with clear error handling, logging, and rollback or recovery notes.
- [ ] Compare at least two approaches and justify trade-offs in correctness, auditability, speed, and team risk.
- [ ] Produce a short incident-style note: symptom, hypothesis, command evidence, root cause, fix, and prevention.
