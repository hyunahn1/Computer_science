# Cloud & Operations

## Overview
How software runs in modern environments: cloud abstractions, containers, orchestration, and observability. Ties **11_System_Design_and_Distributed_Systems** decisions to **how you operate** them in production.

## Course Structure

### [12.1 Cloud Service Models](./12.1_Cloud_Service_Models/)
- IaaS, PaaS, SaaS examples and trade-offs
- Regions, zones, edge (high level)
- Managed services vs self-operated

### [12.2 Docker, Containers & Kubernetes](./12.2_Docker_and_Kubernetes/)
- Docker / OCI images, layers, Dockerfile practices
- Containers vs VMs (isolation, kernel sharing; ties to **05_Operating_Systems**)
- Kubernetes: Pod, Deployment, Service, Ingress (conceptual)
- Health checks: liveness vs readiness

### [12.3 Observability](./12.3_Observability/)
- Logs: structured logging, correlation IDs
- Metrics: RED/USE methods (awareness), SLIs
- Traces: distributed tracing basics
- Dashboards and alerts: signal vs noise

## Study Approach
1. Deploy a toy app to a free tier once
2. Add one structured log field and find it in a log UI

## Interview Preparation
- Explain liveness vs readiness probes
- Three pillars of observability and when each helps debug an incident
- One story: how you found root cause using metrics or traces

## Advanced Topics to Add

- Cloud architecture: IAM, networking, regions/zones, managed service failure modes, shared responsibility.
- Containers: image layers, OCI runtime, namespaces/cgroups, seccomp, capabilities, supply-chain scanning.
- Kubernetes: scheduling, probes, requests/limits, controllers, service discovery, ingress, network policy.
- Observability: RED/USE, SLIs/SLOs, alert fatigue, distributed tracing, cardinality, log correlation.
- Operations: runbooks, incident command, rollback, capacity planning, autoscaling, cost controls, disaster recovery.

## Expert Depth Checklist
- [ ] Explain the control plane and data plane involved in the cloud or orchestration feature.
- [ ] Deploy or simulate the component and capture observable evidence: logs, metrics, traces, events, manifests, or alerts.
- [ ] Define SLOs/SLIs and show how the system tells you when they are violated.
- [ ] Analyze operational failures: bad rollout, resource exhaustion, noisy alert, broken probe, autoscaling lag, or network policy issue.
- [ ] Compare managed vs self-operated trade-offs in cost, reliability, security, and lock-in.
- [ ] Record runbook steps for detection, mitigation, rollback, and post-incident prevention.
