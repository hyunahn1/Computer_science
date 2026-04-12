# System Design & Distributed Systems

## Overview
How to reason about scalable, reliable services: load distribution, caching, data consistency, asynchronous processing, and API design. Complements **04_Networks** (protocols) and **06_Databases** (storage) with **whole-system** thinking typical in software interviews.

## Course Structure

### [7.1 Scalability & Load Balancing](./7.1_Scalability_and_Load_Balancing/)
- Vertical vs horizontal scaling
- Stateless vs stateful services
- Load balancers: L4 vs L7, health checks
- Session stickiness trade-offs
- Rate limiting and backpressure (introduction)

### [7.2 Caching](./7.2_Caching/)
- Cache-aside, read-through, write-through, write-behind
- TTL, eviction policies, stampede mitigation
- CDN role for static and edge caching
- Cache invalidation: "hardest problem" scenarios

### [7.3 Consistency, Availability & Data Design](./7.3_Consistency_and_Data_Design/)
- CAP: what it does and does not guarantee in practice
- Strong vs eventual consistency; quorum intuition
- Idempotency keys for safe retries
- Exactly-once illusion: at-least-once + deduplication

### [7.4 Messaging & Event-Driven Architecture](./7.4_Messaging_and_Event_Driven/)
- Message queues vs pub/sub
- Producers, consumers, acknowledgments
- Ordering guarantees and partitions
- Saga pattern (distributed transactions alternative, high level)
- Outbox pattern (reliable publishing)

### [7.5 API & Service Boundaries](./7.5_API_and_Service_Design/)
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
