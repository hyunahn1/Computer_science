# 7.4 Messaging & Event-Driven Architecture

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
