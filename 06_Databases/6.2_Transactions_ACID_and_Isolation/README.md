# 6.2 Transactions, ACID & Isolation

## Transactions
- BEGIN / COMMIT / ROLLBACK semantics
- Atomicity: all-or-nothing
- Durability: committed data survives crashes (WAL intuition)

## ACID
- **A**tomicity, **C**onsistency, **I**solation, **D**urability
- Consistency: constraints + application invariants

## Isolation Levels
- Read Uncommitted
- Read Committed
- Repeatable Read
- Serializable
- Which anomalies each level prevents or allows

## Anomalies
- Dirty read
- Non-repeatable read
- Phantom read
- Write skew (advanced; good for senior interviews)

## Concurrency Control
- Two-phase locking (intuition)
- Deadlocks: detection, timeout, retry discipline
- MVCC vs locking (high-level comparison)

## Study Materials
- [ ] Trace two concurrent sessions under different isolation levels
- [ ] Vendor-specific defaults (e.g., Postgres vs MySQL) — read docs once

## Practice Problems
- [ ] Given a bug report, infer likely isolation / locking issue
- [ ] Choose isolation level for a money transfer scenario
