# 11.2 Caching

## Patterns
- Cache-aside (lazy loading)
- Read-through / write-through / write-behind
- When each increases complexity vs benefit

## Operations
- TTL selection and clock skew awareness
- Eviction: LRU, LFU, random (conceptual)
- Cache stampede / thundering herd: mitigation (singleflight, probabilistic early expiration)

## Layers
- In-process cache vs distributed cache (Redis, Memcached)
- HTTP caching: Cache-Control, ETag, CDN behavior (tie to **07_Networks**)

## Consistency
- Stale reads after writes
- Invalidation strategies: time-based vs event-based

## Study Materials
- [ ] Diagram cache-aside sequence for read and update
- [ ] One example of bad TTL causing user-visible bug

## Practice Problems
- [ ] Design caching for a read-heavy product detail page
- [ ] How to invalidate after a price change?

## Expert Depth Checklist
- [ ] State requirements, nonrequirements, workload, latency budget, consistency needs, and failure assumptions before designing.
- [ ] Draw the data flow and identify every stateful component, queue, cache, and consistency boundary.
- [ ] Quantify capacity with rough calculations and identify the first bottleneck.
- [ ] Analyze failure modes: partial outage, retry storm, split brain, stale cache, duplicate delivery, hot partition, or cascading failure.
- [ ] Compare consistency, availability, cost, latency, and operational complexity trade-offs.
- [ ] Connect the design to known primitives: quorum, consensus, idempotency, backpressure, leases, logical clocks, or outbox pattern.
