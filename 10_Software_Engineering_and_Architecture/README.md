# Software Engineering

## Overview
How professional software is verified, delivered, and maintained: testing, automation, collaboration practices, and safe refactoring. Complements **13_Developer_Tools_and_Workflow** (Git, shell, debuggers).

## Course Structure

### [10.1 Testing](./10.1_Testing/)
- Test pyramid: unit, integration, end-to-end
- Test doubles: mock, stub, fake, spy (know the differences)
- Arrange-Act-Assert / Given-When-Then
- Property-based testing (awareness)
- Flaky tests: causes and fixes

### [10.2 CI/CD & Automation](./10.2_CI_CD_and_Automation/)
- Continuous Integration: fast feedback, trunk-based vs long-lived branches (trade-offs)
- Pipelines: build, test, lint, security scan, artifact
- Continuous Delivery vs Continuous Deployment
- Feature flags and safe rollout

### [10.3 Code Quality & Refactoring](./10.3_Code_Quality_and_Refactoring/)
- Readable code: naming, boundaries, error handling
- Technical debt: intentional vs accidental
- Refactoring workflows: small steps, tests as safety net
- Code review: constructive feedback, checklist mindset

### [10.4 Architecture & Design Patterns](./10.4_Architecture_and_Design_Patterns/)
- SOLID principles (pragmatic, not dogma)
- **GoF Design Patterns**: Creational (Factory, Singleton), Structural (Adapter, Decorator), Behavioral (Observer, Strategy)
- **Hexagonal Architecture** (Ports and Adapters)
- **Clean Architecture** & Domain-Driven Design (DDD) concepts

## Study Approach
1. Write tests before or immediately after features you build
2. Observe one real CI pipeline (open source project)

## Interview Preparation
- Explain how you caught a bug with the right test level
- Trade-offs of mocking vs integration tests
- How you would roll out a risky change safely
