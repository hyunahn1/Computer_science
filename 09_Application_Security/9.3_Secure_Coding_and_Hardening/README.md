# 9.3 Secure Coding & Hardening

## Least Privilege
- DB roles: app user vs migration user
- Cloud IAM: narrow policies

## Secrets
- Environment vs secret manager
- Rotation and audit
- Pre-commit hooks for secret scanning

## Input / Output
- Validate on trust boundaries
- Encode for the correct context on output

## Common Pitfalls (Awareness)
- SSRF: fetching user-supplied URLs server-side
- Path traversal: unsanitized file paths
- Insecure deserialization: trusted type gadgets
- Mass assignment / IDOR: missing object-level checks

## Supply Chain
- Dependency scanning (SCA)
- Pinning versions, reviewing transitive deps

## Study Materials
- [ ] Threat model one feature: assets, threats, mitigations
- [ ] Run `npm audit` / equivalent on a project and triage one finding

## Practice Problems
- [ ] How to prevent IDOR on `/api/orders/{id}`?
- [ ] Why is logging raw tokens dangerous?

## Expert Depth Checklist
- [ ] Start with a threat model: assets, trust boundaries, attacker capability, entry points, and impact.
- [ ] Reproduce the vulnerability safely in a lab or minimal example, then implement and verify the mitigation.
- [ ] Map issues to OWASP, CWE, or a concrete CVE when possible.
- [ ] Explain why a defense works and what it does not protect against.
- [ ] Review authentication, authorization, input validation, output encoding, secrets, logging, and dependency risks.
- [ ] Include abuse cases and failure modes, not only happy-path security controls.
