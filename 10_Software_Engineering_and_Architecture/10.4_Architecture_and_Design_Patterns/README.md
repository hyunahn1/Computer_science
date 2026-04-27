# 10.4 Architecture & Design Patterns

## Scope
Architecture is about controlling dependencies, change, and operational risk. Patterns are vocabulary, not goals. Study them by identifying the forces they resolve and the costs they introduce.

## Design Principles
- SOLID as dependency-management heuristics, not rules to memorize
- Coupling vs cohesion
- Information hiding and stable interfaces
- Dependency inversion and boundary ownership
- Composition over inheritance
- Local simplicity vs global consistency

## Architecture Styles
- Layered architecture
- Hexagonal architecture / ports and adapters
- Clean architecture
- Modular monolith
- Microservices and service boundaries
- Event-driven architecture
- CQRS and event sourcing, when justified

## Design Patterns
- Creational: Factory Method, Abstract Factory, Builder, Singleton and why it is often harmful
- Structural: Adapter, Facade, Decorator, Proxy, Composite
- Behavioral: Strategy, Observer, Command, State, Template Method
- Concurrency-aware patterns: producer-consumer, worker pool, reactor, circuit breaker

## Expert Depth Checklist
- [ ] For each pattern, state the problem, forces, trade-offs, and failure modes
- [ ] Refactor one code example from inheritance-heavy design to composition
- [ ] Draw dependency direction for layered, hexagonal, and clean architecture
- [ ] Explain when a modular monolith is better than microservices
- [ ] Identify transactional boundaries and data ownership in a service design
- [ ] Explain how architecture affects deployability, testability, and observability
- [ ] Write an ADR (Architecture Decision Record) for one design choice
- [ ] Compare DDD tactical patterns with simpler module boundaries

## Practice Problems
- [ ] Given a tangled module, propose boundaries and justify what should not be abstracted yet
- [ ] Given a feature request, decide whether to use Strategy, State, Observer, or no pattern
- [ ] Review a microservice proposal and identify coupling, data consistency, and operational risks

## Primary Sources
- [ ] Martin Fowler: Patterns of Enterprise Application Architecture
- [ ] Design Patterns: Elements of Reusable Object-Oriented Software
- [ ] Domain-Driven Design by Eric Evans, selectively
- [ ] Fundamentals of Software Architecture by Richards and Ford
