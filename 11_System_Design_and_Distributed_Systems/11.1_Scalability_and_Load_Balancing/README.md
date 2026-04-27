# 11.1 Scalability & Load Balancing

## Scaling
- Vertical scaling: bigger machine limits
- Horizontal scaling: more instances, coordination cost
- Stateless application tier: why it simplifies scaling

## Load Balancing
- Round robin, least connections, weighted routing
- Layer 4 (TCP/UDP) vs Layer 7 (HTTP) load balancing
- Health checks: passive vs active
- Sticky sessions: when needed, when harmful

## Capacity & Reliability (Intro)
- Redundancy and failover
- Overload: queue vs drop vs degrade
- Connection pooling at service and DB

## Study Materials
- [ ] Sketch traffic path: client → LB → app → DB
- [ ] List single points of failure in a naive diagram

## Practice Problems
- [ ] Design LB strategy for WebSocket-heavy app
- [ ] Explain when to avoid sticky sessions

## Expert Depth Checklist
- [ ] State requirements, nonrequirements, workload, latency budget, consistency needs, and failure assumptions before designing.
- [ ] Draw the data flow and identify every stateful component, queue, cache, and consistency boundary.
- [ ] Quantify capacity with rough calculations and identify the first bottleneck.
- [ ] Analyze failure modes: partial outage, retry storm, split brain, stale cache, duplicate delivery, hot partition, or cascading failure.
- [ ] Compare consistency, availability, cost, latency, and operational complexity trade-offs.
- [ ] Connect the design to known primitives: quorum, consensus, idempotency, backpressure, leases, logical clocks, or outbox pattern.
