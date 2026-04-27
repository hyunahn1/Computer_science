# System Design & Distributed Systems

## Overview
How to reason about scalable, reliable services: load distribution, caching, data consistency, asynchronous processing, and API design. Complements **07_Networks** (protocols) and **08_Databases** (storage) with **whole-system** thinking typical in software interviews.

## Course Structure

### [11.1 Scalability & Load Balancing](./11.1_Scalability_and_Load_Balancing/)
- Vertical vs horizontal scaling
- Stateless vs stateful services
- Load balancers: L4 vs L7, health checks
- Session stickiness trade-offs
- Rate limiting and backpressure (introduction)

### [11.2 Caching](./11.2_Caching/)
- Cache-aside, read-through, write-through, write-behind
- TTL, eviction policies, stampede mitigation
- CDN role for static and edge caching
- Cache invalidation: "hardest problem" scenarios

### [11.3 Consistency, Availability & Data Design](./11.3_Consistency_and_Data_Design/)
- CAP: what it does and does not guarantee in practice
- Strong vs eventual consistency; quorum intuition
- Idempotency keys for safe retries
- Exactly-once illusion: at-least-once + deduplication

### [11.4 Messaging & Event-Driven Architecture](./11.4_Messaging_and_Event_Driven/)
- Message queues vs pub/sub
- Producers, consumers, acknowledgments
- Ordering guarantees and partitions
- Saga pattern (distributed transactions alternative, high level)
- Outbox pattern (reliable publishing)

### [11.5 API & Service Boundaries](./11.5_API_and_Service_Design/)
- REST constraints (pragmatic, not academic)
- Resource modeling, versioning, pagination
- Synchronous vs asynchronous boundaries
- Service decomposition: coupling vs autonomy

## Study Approach
1. Pick classic prompts (URL shortener, feed, chat) and draw diagrams
2. Always state assumptions (traffic, consistency needs, latency budget)
3. Link each decision to a trade-off

## Interview Preparation
- Start with requirements, then capacity sketch (Fermi optional)
- Name failure modes: dependency down, partial outage, retry storms
- End with "what I would measure in production"

## Advanced Topics to Add

- Distributed theory: partial failure, FLP intuition, logical clocks, vector clocks, leases, quorum systems.
- Consensus/replication: Raft, Paxos awareness, leader election, log replication, split brain, reconfiguration.
- Reliability: retries, timeouts, backpressure, circuit breakers, bulkheads, graceful degradation.
- Data correctness: idempotency, deduplication, outbox, sagas, exactly-once illusion, CRDT awareness.
- Scale: hot partitions, load shedding, rate limiting, cache stampede, multi-region trade-offs.

## Expert Depth Checklist
- [ ] State requirements, nonrequirements, workload, latency budget, consistency needs, and failure assumptions before designing.
- [ ] Draw the data flow and identify every stateful component, queue, cache, and consistency boundary.
- [ ] Quantify capacity with rough calculations and identify the first bottleneck.
- [ ] Analyze failure modes: partial outage, retry storm, split brain, stale cache, duplicate delivery, hot partition, or cascading failure.
- [ ] Compare consistency, availability, cost, latency, and operational complexity trade-offs.
- [ ] Connect the design to known primitives: quorum, consensus, idempotency, backpressure, leases, logical clocks, or outbox pattern.
