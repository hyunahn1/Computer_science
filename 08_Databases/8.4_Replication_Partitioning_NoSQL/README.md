# 6.4 Replication, Partitioning & NoSQL

## Replication
- Goals: availability, read scaling, disaster recovery
- Single-leader (primary / replica)
- Replication lag and stale reads
- Failover: split-brain risks (conceptual)

## Partitioning (Sharding)
- Shard key choice and hot partitions
- Resharding and operational complexity
- When sharding is premature vs necessary

## Consistency (Practical)
- Strong vs eventual consistency (user-visible effects)
- Read-your-writes, monotonic reads (distributed client experience)

## NoSQL Families
- Key-value: use cases, limitations
- Document: flexible schema, embedding vs referencing
- Wide-column: row key, column families (Cassandra-style intuition)
- Graph: when relationships are the main query pattern

## Study Materials
- [ ] Compare one SQL vs document model for the same domain
- [ ] Draw single-leader replication with read replica lag

## Practice Problems
- [ ] Where would you accept eventual consistency in a product?
- [ ] Shard key design for a high-write timeline feed (discussion)
