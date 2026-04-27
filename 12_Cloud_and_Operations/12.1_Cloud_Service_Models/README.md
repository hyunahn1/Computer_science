# 12.1 Cloud Service Models

## Service Models
- **IaaS**: VMs, networks, you manage OS and up
- **PaaS**: runtime + scaling abstracted; less control
- **SaaS**: complete application

## Geography & Availability
- Region vs availability zone
- Latency vs data residency trade-offs

## Managed Services
- Managed DB, queue, cache: operational burden vs vendor lock-in (pragmatic view)

## Cost (Awareness)
- Pay for provisioned vs consumed
- Egress fees surprise pattern

## Study Materials
- [ ] Map one app you know to IaaS/PaaS pieces
- [ ] Read pricing page for one service you use (15 minutes)

## Practice Problems
- [ ] When choose PaaS over k8s for a small team?
- [ ] What fails if a single AZ dies?

## Expert Depth Checklist
- [ ] Explain the control plane and data plane involved in the cloud or orchestration feature.
- [ ] Deploy or simulate the component and capture observable evidence: logs, metrics, traces, events, manifests, or alerts.
- [ ] Define SLOs/SLIs and show how the system tells you when they are violated.
- [ ] Analyze operational failures: bad rollout, resource exhaustion, noisy alert, broken probe, autoscaling lag, or network policy issue.
- [ ] Compare managed vs self-operated trade-offs in cost, reliability, security, and lock-in.
- [ ] Record runbook steps for detection, mitigation, rollback, and post-incident prevention.
