# 8.1 Relational Model & SQL

## Relational Foundations
- Tables, rows, columns, schemas
- Primary keys, foreign keys, unique constraints
- NULL semantics and three-valued logic (brief)

## Normalization
- Functional dependencies (intuition)
- 1NF, 2NF, 3NF, BCNF — goals and trade-offs (denormalization for read performance)

## SQL
- SELECT, WHERE, GROUP BY, HAVING, ORDER BY
- INNER / LEFT / RIGHT / FULL joins
- Aggregations and window functions (overview)
- Subqueries and correlated subqueries
- UNION vs UNION ALL

## Application Patterns
- ORM benefits and pitfalls
- N+1 queries: detection and fixes (eager loading, batching, JOIN strategy)

## Study Materials
- [ ] Mini schema design exercise (users, orders, line items)
- [ ] Rewrite nested loops as set-based SQL

## Practice Problems
- [ ] Schema design from requirements
- [ ] Complex join and aggregation queries

## Expert Depth Checklist
- [ ] Model data formally: keys, constraints, functional dependencies, normalization, and invariants.
- [ ] Run real queries and inspect `EXPLAIN` or `EXPLAIN ANALYZE` output.
- [ ] Reproduce a concurrency anomaly or locking behavior with two sessions.
- [ ] Explain the storage structure involved: heap file, B+tree, WAL, MVCC tuple/version, LSM tree, or replication log.
- [ ] Compare correctness, latency, throughput, durability, and operational complexity trade-offs.
- [ ] Read vendor documentation and note where PostgreSQL, MySQL, SQLite, or distributed databases differ.
