# 11.3 Consistency, Availability & Data Design

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
- Single DB vs read replicas vs sharding (pointer to **08_Databases**)

## Study Materials
- [ ] Write pseudo-headers for idempotency on a payment API
- [ ] Failure story: retry storm without jitter

## Practice Problems
- [ ] Choose consistency level for social "like" count vs bank transfer
- [ ] Design dedup store for webhook deliveries

## Expert Depth Checklist
- [ ] State requirements, nonrequirements, workload, latency budget, consistency needs, and failure assumptions before designing.
- [ ] Draw the data flow and identify every stateful component, queue, cache, and consistency boundary.
- [ ] Quantify capacity with rough calculations and identify the first bottleneck.
- [ ] Analyze failure modes: partial outage, retry storm, split brain, stale cache, duplicate delivery, hot partition, or cascading failure.
- [ ] Compare consistency, availability, cost, latency, and operational complexity trade-offs.
- [ ] Connect the design to known primitives: quorum, consensus, idempotency, backpressure, leases, logical clocks, or outbox pattern.
