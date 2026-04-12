# 6.3 Indexing & Query Execution

## Indexes
- Purpose: reduce rows examined, support ORDER BY / lookups
- B+Tree (default in many RDBMS): range queries, ordered scans
- Hash indexes: equality-only (where supported)
- Unique vs non-unique indexes

## Physical Design
- Clustered vs non-clustered (terminology varies by engine)
- Composite indexes: column order and selectivity
- Covering indexes (include columns to avoid table lookups)

## Query Execution (Intuition)
- Full table scan vs index seek / range scan
- Cost-based optimizer: statistics role
- Join algorithms at high level: nested loop, hash join, merge join

## Tools
- EXPLAIN / EXPLAIN ANALYZE
- Identifying missing indexes vs over-indexing (write amplification)

## Study Materials
- [ ] Run EXPLAIN on before/after adding an index
- [ ] Document one slow query you fixed with index + query change

## Practice Problems
- [ ] Propose indexes for a given workload (read-heavy vs write-heavy)
- [ ] Explain why a wrong composite index order fails to help
