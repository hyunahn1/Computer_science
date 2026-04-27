# 9.1 OWASP & Web Security Fundamentals

## OWASP Top 10
- Read the current list; be able to give one example each

## Injection
- SQL injection: parameterized queries
- Command injection: shelling out unsafely

## XSS
- Context: HTML, attribute, JavaScript, URL
- Encoding and CSP as defense in depth

## CSRF
- SameSite cookies
- Anti-CSRF tokens for state-changing requests

## Headers & Browser Controls
- Content-Security-Policy basics
- HSTS for HTTPS enforcement

## Study Materials
- [ ] Fix a deliberate XSS in a toy app
- [ ] Explain why escaping alone is not enough for all contexts

## Practice Problems
- [ ] Given HTML form, where can CSRF strike?
- [ ] Differentiate XSS from SQLi in one sentence each

## Expert Depth Checklist
- [ ] Start with a threat model: assets, trust boundaries, attacker capability, entry points, and impact.
- [ ] Reproduce the vulnerability safely in a lab or minimal example, then implement and verify the mitigation.
- [ ] Map issues to OWASP, CWE, or a concrete CVE when possible.
- [ ] Explain why a defense works and what it does not protect against.
- [ ] Review authentication, authorization, input validation, output encoding, secrets, logging, and dependency risks.
- [ ] Include abuse cases and failure modes, not only happy-path security controls.
