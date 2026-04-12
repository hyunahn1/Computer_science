# 7.3 Consistency, Availability & Data Design

## CAP (Practical Framing)
- Partition tolerance in real networks
- CP vs AP products (hand-wavy labels — know the limits of CAP storytelling)
- Linearizability vs sequential consistency (intuition)

## Consistency Models (User-Visible)
- Strong consistency when money or inventory matters
- Eventual consistency when latency or availability wins
- Read-your-writes, monotonic reads

## Reliability Primitives
- Timeouts, retries, exponential backoff + jitter
- Idempotency keys for POST-like operations
- At-least-once delivery + deduplication
- Circuit breaker pattern (stop hammering sick dependencies)

## Data Placement
- Single DB vs read replicas vs sharding (pointer to **06_Databases**)

## Study Materials
- [ ] Write pseudo-headers for idempotency on a payment API
- [ ] Failure story: retry storm without jitter

## Practice Problems
- [ ] Choose consistency level for social "like" count vs bank transfer
- [ ] Design dedup store for webhook deliveries
