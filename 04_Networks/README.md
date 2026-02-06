# Computer Networks

## Overview
Comprehensive study materials covering computer networking from physical layer to application layer, including protocols, security, and practical implementations.

## Course Structure

### [4.1 Network Fundamentals](./4.1_Network_Fundamentals/)
- OSI 7-Layer Model (Physical to Application)
- TCP/IP 4-Layer Model
- Data encapsulation process
- Protocol Data Units (PDUs)
- Communication types (Unicast, Broadcast, Multicast, Anycast)
- Network topologies and types (LAN, WAN, MAN)

### [4.2 Physical & Data Link Layer](./4.2_Physical_and_Link_Layer/)
- Physical layer media (Twisted pair, Coaxial, Fiber optic)
- Signal encoding and transmission
- MAC addressing and Ethernet frames
- Error detection (CRC)
- CSMA/CD and collision handling
- Switches vs Hubs
- VLANs (Virtual LANs)
- ARP (Address Resolution Protocol)

### [4.3 Network Layer (IP)](./4.3_Network_Layer_IP/)
- IPv4 addressing and subnetting
- Subnet masks and CIDR notation
- IPv4 header structure and fields
- Routing tables and forwarding decisions
- Default gateway and next hop
- ICMP (Ping, Traceroute)
- NAT (Network Address Translation)

### [4.4 Transport Layer](./4.4_Transport_Layer/)
- Port numbers and sockets
- UDP (Connectionless, unreliable)
- TCP (Connection-oriented, reliable)
- 3-way handshake (connection establishment)
- 4-way handshake (connection termination)
- TCP header (Sequence, ACK, Flags)
- TCP state machine
- Flow control (Sliding window)
- Congestion control (Slow start, AIMD)

### [4.5 Application Layer](./4.5_Application_Layer/)
- DNS (Hierarchy, queries, record types)
- HTTP/HTTPS (Methods, status codes, headers)
- TLS/SSL and certificates
- Email protocols (SMTP, POP3, IMAP)
- Remote access (Telnet, SSH)
- Cookies and sessions

### [4.6 Network Security](./4.6_Network_Security/)
- Symmetric encryption (AES, DES)
- Asymmetric encryption (RSA, ECC)
- Digital signatures and hash functions
- Certificates and PKI
- Common attacks (MITM, DoS, SQL Injection, XSS, CSRF)
- Firewall and VPN

## Study Approach
1. Understand **layered architecture** concept
2. Learn **protocols at each layer**
3. Practice **packet analysis** (Wireshark)
4. Trace **data flow** through layers
5. Study **security** at each layer

## Key Concepts to Master

### Encapsulation
- How data travels down and up the stack
- Headers added/removed at each layer
- PDU names at each layer

### Addressing
- MAC addresses (Layer 2, local)
- IP addresses (Layer 3, global)
- Port numbers (Layer 4, application)

### Protocols
- When to use each protocol
- How protocols work together
- Trade-offs (TCP vs UDP, HTTP vs HTTPS)

### Troubleshooting
- Use appropriate tools (ping, traceroute, netstat)
- Understand what each layer can reveal
- Systematic approach to network issues

## Interview Preparation
- Explain OSI layers with real-world examples
- Trace packet from browser to server (all layers)
- Compare protocols (TCP vs UDP, HTTP vs HTTPS)
- Describe common attacks and defenses
- Calculate subnets and IP ranges
- Explain how DNS, DHCP, ARP work

## Common Interview Question
**"What happens when you type google.com in your browser?"**
- Prepare detailed answer covering all layers:
  1. DNS resolution
  2. TCP connection (3-way handshake)
  3. TLS handshake (HTTPS)
  4. HTTP request/response
  5. HTML parsing and rendering
