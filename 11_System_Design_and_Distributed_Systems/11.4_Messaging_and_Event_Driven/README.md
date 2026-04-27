# 11.4 Messaging & Event-Driven Architecture

## Messaging Basics
- Queues: point-to-point, competing consumers
- Pub/sub: topics, fan-out
- Brokers vs brokerless (overview only)

## Delivery Semantics
- At-most-once, at-least-once, exactly-once (marketing vs engineering reality)
- Acknowledgments: auto vs manual
- Poison messages and dead-letter queues

## Ordering
- Single partition ordering vs global ordering
- Per-entity ordering (e.g., same user id → same partition)

## Patterns
- Event-driven microservices: choreography vs orchestration
- Outbox pattern for reliable event publication
- Saga: compensating transactions (conceptual walkthrough)

## Study Materials
- [ ] Compare queue vs log (Kafka-style) at high level
- [ ] Draw outbox + consumer flow

## Practice Problems
- [ ] Design async order processing with failures and retries
- [ ] Where would you use DLQ in your pipeline?

## Expert Depth Checklist
- [ ] State requirements, nonrequirements, workload, latency budget, consistency needs, and failure assumptions before designing.
- [ ] Draw the data flow and identify every stateful component, queue, cache, and consistency boundary.
- [ ] Quantify capacity with rough calculations and identify the first bottleneck.
- [ ] Analyze failure modes: partial outage, retry storm, split brain, stale cache, duplicate delivery, hot partition, or cascading failure.
- [ ] Compare consistency, availability, cost, latency, and operational complexity trade-offs.
- [ ] Connect the design to known primitives: quorum, consensus, idempotency, backpressure, leases, logical clocks, or outbox pattern.
