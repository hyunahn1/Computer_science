# Application Security

## Overview
Security from the application and API perspective: common vulnerability classes, authentication vs authorization, and secure coding. Complements **04_Networks/4.6** (crypto, TLS, PKI) with **what can go wrong in your code and design**.

## Course Structure

### [10.1 OWASP & Web Security Fundamentals](./10.1_OWASP_and_Web_Security/)
- OWASP Top 10 (keep current edition in mind)
- Injection (SQL, command, LDAP)
- XSS: reflected, stored, DOM-based
- CSRF: same-site cookies, tokens
- Security headers (CSP, HSTS, X-Frame-Options — awareness)

### [10.2 Authentication & Authorization](./10.2_Authentication_and_Authorization/)
- AuthN vs AuthZ
- Sessions vs tokens (JWT benefits and misuse)
- OAuth2 / OIDC roles at high level (resource owner, client, IdP)
- Password storage: hashing, salt, pepper (where applicable), work factors
- MFA basics

### [10.3 Secure Coding & Hardening](./10.3_Secure_Coding_and_Hardening/)
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
