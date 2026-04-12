# 4.5 Application Layer

## DNS (Domain Name System)

### Purpose
- **Translate domain names to IP addresses**
  - Humans: remember names (google.com)
  - Computers: need IPs (142.250.185.46)
  - Distributed database

### DNS Hierarchy

#### Structure (Right to Left)
```
www.example.com.
 |     |      |  |
host  domain  TLD root
```

#### Levels
1. **Root (.)**: Top of hierarchy (13 root servers)
2. **TLD (Top-Level Domain)**: .com, .org, .net, .kr
3. **Second-Level Domain**: example.com, google.com
4. **Subdomain**: www.example.com, mail.google.com

### DNS Query Types

#### Recursive Query
- **Client → DNS Server**
  - Server does all work
  - Returns final answer or error
  - Common for clients

#### Iterative Query
- **DNS Server → DNS Server**
  - Returns best answer it knows
  - Or referral to another server
  - Requester follows referrals

### DNS Resolution Process
```
1. User types www.example.com
2. Check local DNS cache
3. If not cached, query DNS resolver (ISP)
4. Resolver queries root server → refers to .com server
5. Resolver queries .com server → refers to example.com server
6. Resolver queries example.com server → returns IP
7. Resolver caches result and returns to client
8. Client connects to IP address
```

### DNS Record Types

#### A Record (Address)
- **Domain → IPv4**
  - `example.com → 93.184.216.34`

#### AAAA Record
- **Domain → IPv6**
  - `example.com → 2606:2800:220:1:248:1893:25c8:1946`

#### CNAME (Canonical Name)
- **Alias → Real name**
  - `www.example.com → example.com`
  - Then resolve example.com

#### MX (Mail Exchange)
- **Domain → Mail server**
  - `example.com → mail.example.com` (priority 10)
  - Multiple MX records possible (priority order)

#### NS (Name Server)
- **Domain → Authoritative DNS server**
  - `example.com → ns1.example.com`

#### PTR (Pointer)
- **Reverse lookup: IP → Domain**
  - Used for verification (email anti-spam)

#### TXT (Text)
- **Arbitrary text data**
  - SPF records (email authentication)
  - Domain verification

### DNS Caching
- **TTL (Time To Live)**
  - How long to cache record (seconds)
  - Balance: freshness vs load
  
- **Levels**
  - Browser cache
  - OS cache
  - Router cache
  - ISP DNS cache

### DNS Port
- **UDP port 53** (queries)
- **TCP port 53** (zone transfers, large responses)

## HTTP (Hypertext Transfer Protocol)

### Characteristics
- **Application layer protocol**
  - Client-server model
  - Request-response
  - Stateless (no memory of previous requests)
  - Text-based

### HTTP Versions
- **HTTP/1.0**: One request per connection
- **HTTP/1.1**: Persistent connections, pipelining
- **HTTP/2**: Multiplexing, binary, server push
- **HTTP/3**: QUIC (UDP-based), faster

### HTTP Request

#### Structure
```
METHOD /path HTTP/1.1
Header-Name: value
Header-Name: value

[Body]
```

#### Example
```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Accept: text/html
Accept-Language: en-US
Connection: keep-alive

```

### HTTP Methods

#### GET
- **Retrieve resource**
  - Read-only operation
  - Parameters in URL (query string)
  - No body
  - Idempotent (repeated = same result)
  - Cacheable

#### POST
- **Submit data**
  - Create new resource
  - Data in body
  - Not idempotent
  - Not cacheable

#### PUT
- **Update/replace resource**
  - Idempotent
  - Full replacement

#### PATCH
- **Partial update**
  - Modify part of resource

#### DELETE
- **Remove resource**
  - Idempotent

#### HEAD
- **Like GET but no body**
  - Only headers
  - Check if resource exists
  - Check last-modified time

#### OPTIONS
- **Query supported methods**
  - CORS preflight requests

### HTTP Response

#### Structure
```
HTTP/1.1 STATUS_CODE Reason
Header-Name: value
Header-Name: value

[Body]
```

#### Example
```
HTTP/1.1 200 OK
Date: Mon, 06 Feb 2026 12:00:00 GMT
Server: Apache/2.4.41
Content-Type: text/html; charset=UTF-8
Content-Length: 1234
Connection: keep-alive

<!DOCTYPE html>
<html>...
```

### HTTP Status Codes

#### 1xx: Informational
- **100 Continue**: Proceed with request

#### 2xx: Success
- **200 OK**: Request succeeded
- **201 Created**: Resource created
- **204 No Content**: Success, no body

#### 3xx: Redirection
- **301 Moved Permanently**: Permanent redirect
- **302 Found**: Temporary redirect
- **304 Not Modified**: Use cached version

#### 4xx: Client Error
- **400 Bad Request**: Malformed request
- **401 Unauthorized**: Authentication required
- **403 Forbidden**: No permission
- **404 Not Found**: Resource doesn't exist
- **405 Method Not Allowed**: Wrong HTTP method
- **429 Too Many Requests**: Rate limit

#### 5xx: Server Error
- **500 Internal Server Error**: Generic server error
- **502 Bad Gateway**: Invalid response from upstream
- **503 Service Unavailable**: Server overloaded
- **504 Gateway Timeout**: Upstream timeout

### Common Headers

#### Request Headers
- **Host**: Domain name (required in HTTP/1.1)
- **User-Agent**: Client software
- **Accept**: Acceptable content types
- **Accept-Language**: Preferred languages
- **Authorization**: Credentials
- **Cookie**: Stored cookies
- **Referer**: Previous page URL
- **If-Modified-Since**: Conditional request

#### Response Headers
- **Content-Type**: MIME type of body
- **Content-Length**: Body size in bytes
- **Set-Cookie**: Store cookie
- **Cache-Control**: Caching directives
- **Location**: Redirect URL
- **Server**: Server software
- **Date**: Response timestamp

### Cookies & Sessions

#### Cookies
- **Small data stored by browser**
  - Sent with every request to domain
  - Used for state management
  
- **Set by server**
  ```
  Set-Cookie: sessionid=abc123; Expires=Wed, 09 Jun 2026 10:18:14 GMT; HttpOnly; Secure
  ```
  
- **Attributes**
  - Expires/Max-Age: Lifetime
  - Domain: Which domain can access
  - Path: Which paths can access
  - Secure: HTTPS only
  - HttpOnly: No JavaScript access (XSS protection)
  - SameSite: CSRF protection

## HTTPS (HTTP Secure)

### TLS/SSL
- **Transport Layer Security**
  - Encryption of HTTP
  - Confidentiality, integrity, authentication
  - Uses TCP port 443

### How HTTPS Works
1. **TCP connection** established
2. **TLS handshake**
   - Client Hello (supported ciphers)
   - Server Hello (chosen cipher)
   - Server sends certificate (public key)
   - Client verifies certificate
   - Key exchange (symmetric key agreed)
3. **Encrypted HTTP** communication
4. **Close connection**

### Certificate
- **Digital document**
  - Domain name
  - Public key
  - Issued by CA (Certificate Authority)
  - Signed by CA's private key
  
- **Certificate Chain**
  - Root CA → Intermediate CA → Server Certificate
  - Browser trusts root CAs

### Why HTTPS?
- **Encryption**: Eavesdropping protection
- **Integrity**: Detect tampering
- **Authentication**: Verify server identity
- **SEO**: Google ranks HTTPS higher
- **Browser trust**: Padlock icon

## Email Protocols

### SMTP (Simple Mail Transfer Protocol)

#### Purpose
- **Send email**
  - Client to server
  - Server to server
  
#### Port
- **25**: Server-to-server
- **587**: Client-to-server (with auth)
- **465**: SMTPS (deprecated)

#### Push Protocol
- Sender initiates connection

### POP3 (Post Office Protocol v3)

#### Purpose
- **Retrieve email from server**
  - Download and delete from server
  - Offline access
  
#### Port
- **110**: POP3
- **995**: POP3S (secure)

#### Characteristics
- Simple
- One-way sync (download only)
- Not good for multiple devices

### IMAP (Internet Message Access Protocol)

#### Purpose
- **Access email on server**
  - Keep mail on server
  - Bidirectional sync
  - Multiple devices
  
#### Port
- **143**: IMAP
- **993**: IMAPS (secure)

#### Advantages over POP3
- Email stays on server
- Folder structure synced
- Search on server
- Partial downloads (headers only)
- Multiple device support

### Email Flow
```
Sender → SMTP → Sender's Mail Server
         ↓ SMTP
    Recipient's Mail Server
         ↓ IMAP/POP3
    Recipient's Client
```

## Remote Access Protocols

### Telnet

#### Purpose
- **Remote command-line access**
  - Text-based
  - Port 23
  
#### Problem
- **No encryption**
  - Passwords sent in plain text
  - Security risk
  - **Obsolete**: replaced by SSH

### SSH (Secure Shell)

#### Purpose
- **Secure remote access**
  - Encrypted communication
  - Port 22
  
#### Features
- **Authentication**
  - Password
  - Public key (more secure)
  
- **Encryption**
  - All traffic encrypted
  - Protect against eavesdropping
  
- **Uses**
  - Remote shell
  - File transfer (SCP, SFTP)
  - Port forwarding (tunneling)
  - Git over SSH

#### SSH vs Telnet
| Feature | Telnet | SSH |
|---------|--------|-----|
| Encryption | No | Yes |
| Authentication | Weak | Strong |
| Security | Insecure | Secure |
| Port | 23 | 22 |
| Use | Obsolete | Standard |

## Other Application Protocols

### FTP (File Transfer Protocol)
- **Port 20, 21**
- File upload/download
- Insecure (use SFTP or FTPS)

### DHCP (Dynamic Host Configuration Protocol)
- **Port 67, 68**
- Automatic IP address assignment
- Also provides subnet mask, gateway, DNS

### SNMP (Simple Network Management Protocol)
- **Port 161, 162**
- Network device monitoring
- Routers, switches, servers

### NTP (Network Time Protocol)
- **Port 123**
- Clock synchronization
- Critical for logs, certificates

## Interview Practice
- [ ] Explain DNS resolution process step-by-step
- [ ] Describe HTTP request/response structure
- [ ] Compare HTTP methods (GET vs POST)
- [ ] What happens when you visit https://example.com?
- [ ] HTTP status codes (common ones)
- [ ] How does HTTPS provide security?
- [ ] IMAP vs POP3 comparison
- [ ] Why is SSH preferred over Telnet?
- [ ] Explain cookies and sessions
- [ ] What is HTTP/2 and its improvements?
