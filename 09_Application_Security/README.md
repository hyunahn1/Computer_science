# Application Security

## Overview
Security from the application and API perspective: common vulnerability classes, authentication vs authorization, and secure coding. Complements **07_Networks/7.6** (crypto, TLS, PKI) with **what can go wrong in your code and design**.

## Course Structure

### [9.1 OWASP & Web Security Fundamentals](./9.1_OWASP_and_Web_Security/)
- OWASP Top 10 (keep current edition in mind)
- Injection (SQL, command, LDAP)
- XSS: reflected, stored, DOM-based
- CSRF: same-site cookies, tokens
- Security headers (CSP, HSTS, X-Frame-Options — awareness)

### [9.2 Authentication & Authorization](./9.2_Authentication_and_Authorization/)
- AuthN vs AuthZ
- Sessions vs tokens (JWT benefits and misuse)
- OAuth2 / OIDC roles at high level (resource owner, client, IdP)
- Password storage: hashing, salt, pepper (where applicable), work factors
- MFA basics

### [9.3 Secure Coding & Hardening](./9.3_Secure_Coding_and_Hardening/)
- Least privilege (DB users, service accounts)
- Secrets management: never in repo, rotation
- Input validation and output encoding
- SSRF, path traversal, deserialization risks (awareness)
- Dependency vulnerabilities (SCA)

## Study Approach
1. Read one CVE summary and map it to a CWE category
2. Try OWASP WebGoat or similar lab once

## Interview Preparation
- Walk through CSRF and XSS defenses for a cookie-based app
- Why JWT in localStorage is controversial
- How you would store passwords today

## Advanced Topics to Add

- Threat modeling: STRIDE, attack trees, trust boundaries, abuse cases, security requirements.
- Web security: XSS variants, CSRF mechanics, SSRF, deserialization, path traversal, request smuggling awareness.
- Auth: OAuth2/OIDC flows, token lifetimes, session fixation, password reset security, MFA failure modes.
- Secure coding: memory safety, input canonicalization, output encoding, secrets management, dependency risk.
- Verification: SAST/DAST/SCA, fuzzing, security logging, incident response, CVE/CWE mapping.

## Expert Depth Checklist
- [ ] Start with a threat model: assets, trust boundaries, attacker capability, entry points, and impact.
- [ ] Reproduce the vulnerability safely in a lab or minimal example, then implement and verify the mitigation.
- [ ] Map issues to OWASP, CWE, or a concrete CVE when possible.
- [ ] Explain why a defense works and what it does not protect against.
- [ ] Review authentication, authorization, input validation, output encoding, secrets, logging, and dependency risks.
- [ ] Include abuse cases and failure modes, not only happy-path security controls.
