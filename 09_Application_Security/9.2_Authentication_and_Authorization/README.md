# 10.2 Authentication & Authorization

## Definitions
- **Authentication**: who you are
- **Authorization**: what you may do

## Sessions
- Server-side session store vs client-held session id
- Cookie flags: HttpOnly, Secure, SameSite

## Tokens
- JWT structure (header, payload, signature) — high level
- Where to store tokens (trade-offs: XSS vs CSRF)
- Refresh tokens and rotation

## OAuth2 / OIDC (High Level)
- Roles: authorization server, resource server, client
- Authorization code flow with PKCE (why PKCE for public clients)

## Passwords
- Hashing algorithms appropriate for passwords (bcrypt, Argon2)
- Salts, pepper (optional), adaptive work factor

## MFA
- Something you know / have / are
- TOTP basics

## Study Materials
- [ ] Diagram authorization code + PKCE
- [ ] List mistakes: JWT in URL, `alg: none`, long-lived tokens without rotation

## Practice Problems
- [ ] Session fixation: what is it and how to prevent?
- [ ] When is OAuth2 not authentication by itself?
