# 13.1 Git & Collaboration

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
