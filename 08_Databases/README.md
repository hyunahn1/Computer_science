# Databases & Persistence

## Overview
Relational databases, SQL, transactional guarantees, physical design (indexes), and distributed data patterns (replication, sharding, NoSQL). Essential for backend roles and common in general software interviews.

## Course Structure

### [8.1 Relational Model & SQL](./8.1_RDBMS_and_SQL/)
- Relational algebra intuition (select, project, join)
- SQL DDL vs DML
- Keys, constraints, normalization (1NF–3NF, BCNF at high level)
- Joins (inner, outer), subqueries, EXISTS
- N+1 query problem and mitigation

### [8.2 Transactions, ACID & Isolation](./8.2_Transactions_ACID_and_Isolation/)
- ACID properties
- Isolation levels (Read Uncommitted → Serializable)
- Phenomena: dirty read, non-repeatable read, phantom read
- Locks: shared vs exclusive, deadlock at DB layer
- MVCC intuition (snapshot isolation)

### [8.3 Indexing & Query Execution](./8.3_Indexing_and_Query_Execution/)
- B-Tree / B+Tree indexes (why databases use them)
- Clustered vs non-clustered indexes
- Covering indexes, composite indexes, selectivity
- Query planner basics: scans vs seeks, cost model intuition
- Explain / EXPLAIN ANALYZE workflow

### [8.4 Replication, Partitioning & NoSQL](./8.4_Replication_Partitioning_NoSQL/)
- Single-leader vs multi-leader replication
- Read replicas, replication lag
- Horizontal partitioning (sharding) vs vertical partitioning
- CAP trade-off at a practical level
- NoSQL families: document, key-value, wide-column, graph (when each fits)

## Study Approach
1. Run queries locally (SQLite/Postgres) while reading theory
2. Practice explaining isolation levels without memorizing vendor charts only
3. Tie topics to interview prompts: "How would you design indexes for this query?"

## Interview Preparation
- Compare isolation levels and which anomalies they allow
- Walk through adding an index and what changes in the execution plan
- When to shard vs when to scale vertically first

## Advanced Topics to Add

- Relational theory: functional dependencies, normalization beyond 3NF, relational algebra, query equivalence.
- Storage engines: heap files, B+trees, LSM trees, buffer pool, WAL/ARIES, checkpoints, compaction.
- Transactions: serializability, 2PL, MVCC, snapshot isolation, write skew, predicate locking.
- Query processing: optimizer cost model, join algorithms, statistics, indexes, materialized views.
- Distributed data: replication logs, consensus awareness, sharding, rebalancing, quorum reads/writes, CDC.

## Expert Depth Checklist
- [ ] Model data formally: keys, constraints, functional dependencies, normalization, and invariants.
- [ ] Run real queries and inspect `EXPLAIN` or `EXPLAIN ANALYZE` output.
- [ ] Reproduce a concurrency anomaly or locking behavior with two sessions.
- [ ] Explain the storage structure involved: heap file, B+tree, WAL, MVCC tuple/version, LSM tree, or replication log.
- [ ] Compare correctness, latency, throughput, durability, and operational complexity trade-offs.
- [ ] Read vendor documentation and note where PostgreSQL, MySQL, SQLite, or distributed databases differ.
