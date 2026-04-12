# 7.1 Scalability & Load Balancing

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
