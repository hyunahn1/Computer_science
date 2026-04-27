# 10.1 Testing

## Levels
- Unit: fast, isolated, many
- Integration: modules + real dependencies (DB, HTTP where needed)
- E2E: full system path, fewer, slower
- Contract tests (consumer/provider) — awareness

## Test Doubles
- Stub: canned answers
- Fake: working lightweight implementation (in-memory DB)
- Mock: assert interactions (use sparingly)
- Spy: record calls

## Style
- AAA / GWT structure
- One logical assertion focus per test (guideline)
- Table-driven tests where useful

## Quality
- Determinism: time, randomness, concurrency
- Flakiness: retries are a smell; fix root cause
- Coverage: useful metric, not a goal by itself

## Study Materials
- [ ] Convert one untested module to tested with doubles you can justify
- [ ] List what you would not mock in an integration test

## Practice Problems
- [ ] Choose test level for: password hasher, payment adapter, login page
- [ ] Design tests for retry logic with clock control

## Expert Depth Checklist
- [ ] Tie every practice to risk reduction: correctness, maintainability, operability, delivery speed, or team coordination.
- [ ] Use a real or toy codebase to demonstrate the technique rather than only defining it.
- [ ] Compare alternatives and state the cost of overengineering.
- [ ] Write evidence: tests, CI output, review checklist, ADR, refactoring steps, or defect analysis.
- [ ] Identify failure modes: flaky tests, brittle mocks, unsafe deploys, dependency cycles, unclear ownership, or architectural drift.
- [ ] Explain how the practice changes under small project, large team, regulated system, and high-availability service constraints.
