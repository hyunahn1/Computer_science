# Cloud & Operations

## Overview
How software runs in modern environments: cloud abstractions, containers, orchestration, and observability. Ties **07_System_Design** decisions to **how you operate** them in production.

## Course Structure

### [11.1 Cloud Service Models](./11.1_Cloud_Service_Models/)
- IaaS, PaaS, SaaS examples and trade-offs
- Regions, zones, edge (high level)
- Managed services vs self-operated

### [11.2 Docker, Containers & Kubernetes](./11.2_Docker_and_Kubernetes/)
- Docker / OCI images, layers, Dockerfile practices
- Containers vs VMs (isolation, kernel sharing; ties to **01_Operating_Systems**)
- Kubernetes: Pod, Deployment, Service, Ingress (conceptual)
- Health checks: liveness vs readiness

### [11.3 Observability](./11.3_Observability/)
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
