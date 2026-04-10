# 11.3 Observability

## Logs
- Structured JSON vs plain text
- Log levels: when to use error vs warn
- Correlation / request IDs across services
- PII and secrets in logs

## Metrics
- Counters, gauges, histograms
- RED (rate, errors, duration) for services
- USE for resources (utilization, saturation, errors)

## Traces
- Spans, trace context propagation
- Sampling (why not trace everything forever)

## Operations
- Dashboards for golden signals
- Alerting: symptom-based vs cause-based
- Runbooks and on-call hygiene

## Study Materials
- [ ] Add request ID middleware and grep across two services
- [ ] One dashboard: latency p50/p95/p99

## Practice Problems
- [ ] User says "slow" — which pillar do you check first and why?
- [ ] Metric cardinality explosion: what causes it?
