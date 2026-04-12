# 11.2 Docker, Containers & Kubernetes

**Docker** builds and runs **container images**; **Kubernetes** (K8s) orchestrates many containers across machines. OS-level isolation (namespaces, cgroups) is introduced in **[01 Operating Systems](../../01_Operating_Systems/README.md)**; this topic focuses on images, runtime practices, and cluster primitives.

## Containers (Docker and OCI images)
- Namespaces and cgroups (see **[01 Operating Systems](../../01_Operating_Systems/README.md)**, esp. Linux kernel / containers-related notes)
- Image immutability, tags vs digests
- Multi-stage builds for smaller images

## Runtime
- Container vs host user mapping (root risks)
- Read-only root filesystem where possible

## Orchestration (Kubernetes Sketch)
- Pod: co-located containers
- Deployment: desired replicas, rollouts
- Service: stable network endpoint
- Ingress: HTTP routing into cluster
- ConfigMaps / Secrets (never commit real secrets)

## Health
- Liveness: restart if broken
- Readiness: remove from load balancer if not ready

## Study Materials
- [ ] `docker run` a small app; inspect layers
- [ ] Read one Deployment + Service YAML

## Practice Problems
- [ ] Why readiness failing might cause cascading errors?
- [ ] VM vs container isolation boundary in one sentence

