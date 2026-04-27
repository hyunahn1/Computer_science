# 10.2 CI/CD & Automation

## Continuous Integration
- Merge frequently, main branch always releasable (goal)
- Fast pipeline stages: fail early (lint, unit tests first)
- Branching strategies: trunk-based, GitHub Flow, GitFlow (pros/cons)

## Pipeline Stages (Typical)
- Dependency install, compile/build
- Static analysis, format check
- Unit + integration tests
- Security scanning (dependencies, secrets)
- Build artifact (container image, binary)

## Delivery vs Deployment
- Continuous Delivery: always releasable, human approves deploy
- Continuous Deployment: automated deploy after gates

## Safe Releases
- Feature flags, gradual rollout, canary (conceptual)
- Rollback strategy: backward-compatible migrations

## Study Materials
- [ ] Read one project's `.github/workflows` or equivalent
- [ ] Document your team's release path end-to-end

## Practice Problems
- [ ] Order pipeline stages for minimal wasted compute
- [ ] How to deploy DB migration without downtime (high-level)

## Expert Depth Checklist
- [ ] Tie every practice to risk reduction: correctness, maintainability, operability, delivery speed, or team coordination.
- [ ] Use a real or toy codebase to demonstrate the technique rather than only defining it.
- [ ] Compare alternatives and state the cost of overengineering.
- [ ] Write evidence: tests, CI output, review checklist, ADR, refactoring steps, or defect analysis.
- [ ] Identify failure modes: flaky tests, brittle mocks, unsafe deploys, dependency cycles, unclear ownership, or architectural drift.
- [ ] Explain how the practice changes under small project, large team, regulated system, and high-availability service constraints.
