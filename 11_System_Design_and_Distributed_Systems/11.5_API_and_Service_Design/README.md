# 11.5 API & Service Design

## REST (Pragmatic)
- Resources, nouns, HTTP verbs
- Status codes: common choices and misuse
- Pagination: offset vs cursor
- Versioning: URL vs header trade-offs

## API Quality
- Idempotency for unsafe methods where needed
- Rate limiting and quotas (user vs client)
- Error model: stable codes, actionable messages

## Service Boundaries
- Monolith vs services: drivers and pain points
- Data ownership per service
- Synchronous chains vs async handoffs

## Alternatives (Awareness)
- GraphQL: benefits and N+1 / complexity risks
- gRPC: when binary contracts and streaming help

## Study Materials
- [ ] Design REST resources for one domain (library, rideshare)
- [ ] List failure responses a client should handle

## Practice Problems
- [ ] Debounce vs throttle for a public API (conceptual)
- [ ] When would you split a service vs keep a module?

## Expert Depth Checklist
- [ ] State requirements, nonrequirements, workload, latency budget, consistency needs, and failure assumptions before designing.
- [ ] Draw the data flow and identify every stateful component, queue, cache, and consistency boundary.
- [ ] Quantify capacity with rough calculations and identify the first bottleneck.
- [ ] Analyze failure modes: partial outage, retry storm, split brain, stale cache, duplicate delivery, hot partition, or cascading failure.
- [ ] Compare consistency, availability, cost, latency, and operational complexity trade-offs.
- [ ] Connect the design to known primitives: quorum, consensus, idempotency, backpressure, leases, logical clocks, or outbox pattern.
