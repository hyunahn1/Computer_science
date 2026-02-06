# 4.1 Network Fundamentals

## OSI 7-Layer Model

### Complete Layer Stack
```
7. Application    - End-user protocols (HTTP, FTP, SMTP)
6. Presentation   - Data format, encryption, compression
5. Session        - Connection management, sessions
4. Transport      - End-to-end connection (TCP, UDP)
3. Network        - Routing, IP addressing
2. Data Link      - Frame, MAC address, error detection
1. Physical       - Cables, signals, bits
```

### Layer-by-Layer Breakdown

#### Layer 1: Physical Layer
- **Function**
  - Transmission of raw bits over physical medium
  - Electrical/optical signals
  
- **Components**
  - Cables (twisted pair, coaxial, fiber)
  - Connectors (RJ45, BNC)
  - Hubs, repeaters
  
- **Specifications**
  - Voltage levels
  - Timing of signals
  - Physical data rates
  - Pin layouts

#### Layer 2: Data Link Layer
- **Function**
  - Frame creation
  - MAC addressing
  - Error detection
  - Flow control (local)
  
- **Sub-layers**
  - LLC (Logical Link Control)
  - MAC (Media Access Control)
  
- **Protocols**
  - Ethernet
  - Wi-Fi (802.11)
  - PPP (Point-to-Point Protocol)
  
- **Devices**
  - Switches
  - Bridges

#### Layer 3: Network Layer
- **Function**
  - Logical addressing (IP)
  - Routing between networks
  - Packet forwarding
  
- **Protocols**
  - IP (IPv4, IPv6)
  - ICMP (ping, traceroute)
  - ARP (Address Resolution Protocol)
  
- **Devices**
  - Routers
  - Layer 3 switches

#### Layer 4: Transport Layer
- **Function**
  - End-to-end communication
  - Segmentation and reassembly
  - Flow control (end-to-end)
  - Error recovery
  
- **Protocols**
  - TCP (connection-oriented, reliable)
  - UDP (connectionless, unreliable)
  
- **Concepts**
  - Port numbers (identify applications)
  - Multiplexing

#### Layer 5: Session Layer
- **Function**
  - Establish, manage, terminate sessions
  - Dialog control
  - Synchronization
  
- **Examples**
  - NetBIOS
  - RPC (Remote Procedure Call)
  - Session tokens

#### Layer 6: Presentation Layer
- **Function**
  - Data format translation
  - Encryption/decryption
  - Compression
  
- **Examples**
  - SSL/TLS
  - JPEG, GIF (image formats)
  - ASCII, EBCDIC (character encoding)

#### Layer 7: Application Layer
- **Function**
  - Network services to applications
  - User interface
  
- **Protocols**
  - HTTP/HTTPS (web)
  - FTP (file transfer)
  - SMTP (email)
  - DNS (name resolution)
  - SSH (secure shell)

## TCP/IP Model (4 Layers)

### Simplified Stack
```
4. Application    - Combines OSI 5, 6, 7
3. Transport      - TCP, UDP (same as OSI 4)
2. Internet       - IP, ICMP (same as OSI 3)
1. Link           - Combines OSI 1, 2
```

### TCP/IP vs OSI

| TCP/IP | OSI | Description |
|--------|-----|-------------|
| Application | Application, Presentation, Session | End-user services |
| Transport | Transport | TCP, UDP |
| Internet | Network | IP routing |
| Link | Data Link, Physical | Hardware |

### Why Two Models?

#### OSI Model
- **Academic/theoretical**
- Detailed layering
- Reference model for understanding
- Not all layers used in practice

#### TCP/IP Model
- **Practical/real-world**
- Based on actual Internet protocols
- Simpler, fewer layers
- What's actually implemented

## Data Encapsulation

### Top-Down Process
```
Application: [DATA]
Transport:   [TCP/UDP Header | DATA]  → Segment
Network:     [IP Header | TCP Header | DATA]  → Packet
Data Link:   [Frame Header | IP | TCP | DATA | Trailer]  → Frame
Physical:    [Bits on wire]
```

### At Each Layer
1. **Application Layer**
   - Generate data (HTTP request, email)
   
2. **Transport Layer**
   - Add TCP/UDP header
   - Add source/destination port
   - Create segment
   
3. **Network Layer**
   - Add IP header
   - Add source/destination IP
   - Create packet
   
4. **Data Link Layer**
   - Add Ethernet header and trailer
   - Add source/destination MAC
   - Add error checking (CRC)
   - Create frame
   
5. **Physical Layer**
   - Convert to electrical/optical signals
   - Transmit bits

## Protocol Data Units (PDUs)

| Layer | PDU Name | Example |
|-------|----------|---------|
| Application | Data | HTTP message |
| Transport | Segment (TCP) / Datagram (UDP) | TCP segment |
| Network | Packet | IP packet |
| Data Link | Frame | Ethernet frame |
| Physical | Bit | Electrical signal |

## Layering Benefits

### Modularity
- Each layer independent
- Changes in one layer don't affect others
- Easier to develop and maintain

### Standardization
- Common interfaces between layers
- Different implementations possible
- Interoperability

### Abstraction
- Upper layers don't need to know lower layer details
- Application doesn't care about physical medium
- Simplifies development

## Communication Types

### Unicast
- One-to-one communication
- Single destination
- Most common (HTTP, SSH, etc.)

### Broadcast
- One-to-all communication
- All devices on network receive
- Layer 2 (MAC FF:FF:FF:FF:FF:FF)
- Limited to local network

### Multicast
- One-to-many communication
- Selected group of receivers
- Efficient for streaming, video conferencing
- Multicast IP addresses (224.0.0.0/4)

### Anycast
- One-to-nearest communication
- Multiple servers share same IP
- Traffic goes to closest
- Used in CDN, DNS

## Network Topologies

### Physical Topologies

#### Bus
- Single cable backbone
- All devices connected to cable
- Cheap but single point of failure

#### Star
- Central hub/switch
- Each device has dedicated connection
- Most common in modern LANs
- Easy to troubleshoot

#### Ring
- Devices connected in circle
- Data travels in one direction
- Token Ring (obsolete)

#### Mesh
- Every device connected to every other
- Highly redundant
- Expensive
- Used in critical networks

### Logical vs Physical
- **Physical**: How devices are wired
- **Logical**: How data flows

## Network Types by Scale

### PAN (Personal Area Network)
- Very small area (10m)
- Bluetooth devices
- Personal devices

### LAN (Local Area Network)
- Building or campus
- Ethernet, Wi-Fi
- High speed, low latency

### MAN (Metropolitan Area Network)
- City-wide
- Cable TV networks
- Fiber optic rings

### WAN (Wide Area Network)
- Large geographic area
- Internet is largest WAN
- Lower speed, higher latency

## Interview Practice
- [ ] Explain all 7 OSI layers with examples
- [ ] Compare OSI vs TCP/IP models
- [ ] Describe data encapsulation process
- [ ] Explain unicast, broadcast, multicast
- [ ] Trace a packet from application to physical layer
- [ ] Why do we need layered architecture?
- [ ] What happens when you type a URL in browser? (all layers)
