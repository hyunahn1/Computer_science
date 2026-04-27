# 10.3 Code Quality & Refactoring

## Readability
- Names reveal intent; comments explain "why," not "what"
- Function size and single responsibility (heuristic)
- Error handling: fail fast vs recover; never swallow silently

## Design Principles (Pragmatic)
- SOLID headlines: dependency inversion for testability, etc.
- DRY vs premature abstraction
- YAGNI when it saves complexity

## Refactoring
- Boy Scout rule; small safe steps
- Characterization tests before risky changes
- Extract method, introduce parameter object (catalog awareness)

## Code Review
- Kind, specific, actionable feedback
- Security and correctness first; style nits last
- Author checklist before requesting review

## Technical Debt
- Deliberate vs accidental debt
- Paydown strategies: boy scout, dedicated time, strangler fig

## Study Materials
- [ ] Refactor one messy function with tests green throughout
- [ ] Write a review comment you'd want to receive

## Practice Problems
- [ ] When would you violate DRY?
- [ ] Identify code smell in a snippet and propose refactor

## Expert Depth Checklist
- [ ] Tie every practice to risk reduction: correctness, maintainability, operability, delivery speed, or team coordination.
- [ ] Use a real or toy codebase to demonstrate the technique rather than only defining it.
- [ ] Compare alternatives and state the cost of overengineering.
- [ ] Write evidence: tests, CI output, review checklist, ADR, refactoring steps, or defect analysis.
- [ ] Identify failure modes: flaky tests, brittle mocks, unsafe deploys, dependency cycles, unclear ownership, or architectural drift.
- [ ] Explain how the practice changes under small project, large team, regulated system, and high-availability service constraints.
