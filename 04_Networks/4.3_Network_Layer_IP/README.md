# 4.3 Network Layer (IP)

## IPv4 Fundamentals

### IP Address Structure
- **32-bit address**
  - Written as 4 decimal octets
  - Example: `192.168.1.100`
  - Range per octet: 0-255
  
- **Binary Representation**
  - `192.168.1.100` = `11000000.10101000.00000001.01100100`

### IP Address Classes (Historical)

#### Classful Addressing (Obsolete)
| Class | First Octet | Network Bits | Host Bits | Range |
|-------|-------------|--------------|-----------|-------|
| A | 0-127 | 8 | 24 | 0.0.0.0 - 127.255.255.255 |
| B | 128-191 | 16 | 16 | 128.0.0.0 - 191.255.255.255 |
| C | 192-223 | 24 | 8 | 192.0.0.0 - 223.255.255.255 |
| D | 224-239 | - | - | Multicast |
| E | 240-255 | - | - | Reserved |

#### Special Addresses
- **Loopback**: `127.0.0.0/8` (typically `127.0.0.1`)
  - Traffic to self
  - Testing
  
- **Private Addresses** (RFC 1918)
  - `10.0.0.0/8` (Class A)
  - `172.16.0.0/12` (Class B)
  - `192.168.0.0/16` (Class C)
  - Not routable on Internet
  - NAT required for Internet access
  
- **APIPA** (Automatic Private IP Addressing)
  - `169.254.0.0/16`
  - Self-assigned when DHCP fails
  
- **Broadcast**
  - All host bits = 1
  - Example: `192.168.1.255` (for network 192.168.1.0/24)

## Subnet Mask

### Purpose
- **Divide IP into network and host portions**
  - Network portion: identifies subnet
  - Host portion: identifies device within subnet

### Format
- **Dotted decimal**: `255.255.255.0`
- **CIDR notation**: `/24`

### How It Works
- **Bitwise AND** with IP address
  - IP: `192.168.1.100`
  - Mask: `255.255.255.0`
  - Network: `192.168.1.0`

### Common Subnet Masks
| CIDR | Dotted Decimal | Hosts | Use Case |
|------|----------------|-------|----------|
| /8 | 255.0.0.0 | 16,777,214 | Class A |
| /16 | 255.255.0.0 | 65,534 | Class B |
| /24 | 255.255.255.0 | 254 | Class C, common LAN |
| /30 | 255.255.255.252 | 2 | Point-to-point links |
| /32 | 255.255.255.255 | 1 | Single host |

### Calculating Hosts
- **Formula**: 2^(host bits) - 2
  - -2 for network address and broadcast address
  
- **/24 Example**
  - 32 - 24 = 8 host bits
  - 2^8 - 2 = 254 usable hosts

## CIDR (Classless Inter-Domain Routing)

### Notation
- **IP/prefix-length**
  - `192.168.1.0/24`
  - More flexible than classful
  
### Variable Length Subnet Masking (VLSM)
- Different subnet sizes in same network
- Efficient IP address utilization

### Supernetting
- Combine multiple networks into larger block
- Reduce routing table size
- Example: `192.168.0.0/22` includes:
  - `192.168.0.0/24`
  - `192.168.1.0/24`
  - `192.168.2.0/24`
  - `192.168.3.0/24`

## IPv4 Header Structure

### Header Format (20-60 bytes)
```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |Type of Service|          Total Length         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |Flags|      Fragment Offset    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Time to Live |    Protocol   |         Header Checksum       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Source Address                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Destination Address                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (if any)                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### Key Fields

#### Version (4 bits)
- IPv4 = 4, IPv6 = 6

#### IHL (Internet Header Length, 4 bits)
- Header length in 32-bit words
- Minimum: 5 (20 bytes)
- Maximum: 15 (60 bytes)

#### Type of Service / DSCP (8 bits)
- QoS (Quality of Service)
- Priority/delay/throughput preferences

#### Total Length (16 bits)
- Entire packet size (header + data)
- Maximum: 65,535 bytes

#### Identification (16 bits)
- Unique ID for fragmented packets
- All fragments have same ID

#### Flags (3 bits)
- **DF (Don't Fragment)**: Do not fragment packet
- **MF (More Fragments)**: More fragments follow

#### Fragment Offset (13 bits)
- Position of fragment in original packet
- In 8-byte units

#### Time to Live (TTL, 8 bits)
- **Hop limit**
  - Decremented by each router
  - Prevents infinite loops
  - Packet dropped when TTL = 0
  - Typical initial: 64 or 128

#### Protocol (8 bits)
- Upper layer protocol
  - 1 = ICMP
  - 6 = TCP
  - 17 = UDP

#### Header Checksum (16 bits)
- Error detection for header only
- Recalculated at each hop (TTL changes)

#### Source/Destination Address (32 bits each)
- IP addresses

## Routing

### Routing Table
- **List of network destinations**
  - Stored in router or host
  - Determines packet forwarding

### Routing Table Entry
| Destination | Netmask | Gateway | Interface | Metric |
|-------------|---------|---------|-----------|--------|
| 192.168.1.0 | /24 | 0.0.0.0 | eth0 | 0 |
| 0.0.0.0 | /0 | 192.168.1.1 | eth0 | 1 |

#### Fields
- **Destination**: Target network
- **Netmask**: Subnet mask
- **Gateway**: Next hop router (0.0.0.0 = directly connected)
- **Interface**: Outgoing interface
- **Metric**: Cost/distance (lower is better)

### Next Hop
- **Next router in path**
  - Not final destination
  - Each router forwards to next hop

### Default Gateway
- **0.0.0.0/0 route**
  - Matches all destinations
  - Used when no specific route found
  - "Gateway of last resort"
  
- **Configuration**
  - Hosts: typically router IP (e.g., 192.168.1.1)
  - Routers: ISP's router

### Routing Decision Process
1. Receive packet
2. Extract destination IP
3. Search routing table (longest prefix match)
4. Forward to next hop via interface
5. Decrement TTL
6. Recalculate checksum
7. Forward packet

### Longest Prefix Match
- **Most specific route wins**
  - Multiple routes may match
  - Choose route with longest netmask
  
- **Example**: Destination `192.168.1.50`
  - Route 1: `192.168.0.0/16` (16-bit match)
  - Route 2: `192.168.1.0/24` (24-bit match) ✓ **Winner**
  - Route 3: `0.0.0.0/0` (0-bit match, default)

### Static vs Dynamic Routing

#### Static Routing
- **Manually configured**
  - Administrator adds routes
  - Doesn't adapt to changes
  
- **Pros**: Simple, predictable, secure
- **Cons**: Not scalable, manual updates
- **Use**: Small networks, specific routes

#### Dynamic Routing
- **Protocols automatically exchange routes**
  - RIP (Routing Information Protocol)
  - OSPF (Open Shortest Path First)
  - BGP (Border Gateway Protocol)
  
- **Pros**: Adapts to changes, scalable
- **Cons**: More complex, overhead
- **Use**: Large networks, Internet

## ICMP (Internet Control Message Protocol)

### Purpose
- **Control and error messages**
  - Not for data transfer
  - Operates at network layer (uses IP)
  - Protocol number: 1

### Common ICMP Messages

#### Echo Request/Reply (Type 8/0)
- **Ping utility**
  - Test reachability
  - Measure round-trip time
  
- **Usage**
  ```bash
  ping 8.8.8.8
  ```

#### Destination Unreachable (Type 3)
- **Cannot deliver packet**
  - Code 0: Network unreachable
  - Code 1: Host unreachable
  - Code 3: Port unreachable
  - Code 4: Fragmentation needed but DF set

#### Time Exceeded (Type 11)
- **TTL expired**
  - Code 0: TTL expired in transit
  - Used by traceroute

#### Redirect (Type 5)
- Router tells host to use different gateway

### Ping

#### How It Works
1. Send ICMP Echo Request
2. Destination responds with Echo Reply
3. Measure round-trip time

#### Output Information
```
64 bytes from 8.8.8.8: icmp_seq=1 ttl=56 time=10.2 ms
```
- **Bytes**: Packet size
- **icmp_seq**: Sequence number
- **TTL**: Remaining TTL (not original)
- **Time**: Round-trip time

### Traceroute (tracert on Windows)

#### How It Works
1. Send packet with TTL=1
2. First router decrements TTL to 0, sends Time Exceeded
3. Send packet with TTL=2
4. Second router decrements TTL to 0, sends Time Exceeded
5. Repeat until destination reached

#### Output
```
1  192.168.1.1 (192.168.1.1)  1.234 ms
2  10.0.0.1 (10.0.0.1)  5.678 ms
3  8.8.8.8 (8.8.8.8)  10.234 ms
```
- Shows each hop in path
- Identifies routing issues

## NAT (Network Address Translation)

### Purpose
- **Share single public IP**
  - Multiple private IPs → one public IP
  - Address conservation
  - Security (hide internal network)

### Types

#### Static NAT
- One-to-one mapping
- Private IP ↔ Public IP

#### Dynamic NAT
- Pool of public IPs
- Assign dynamically

#### PAT (Port Address Translation / NAT Overload)
- Many-to-one with port numbers
- Most common (home routers)
- Private IP:Port ↔ Public IP:Different Port

### How PAT Works
1. Internal host (192.168.1.100:5000) sends to Internet
2. Router replaces source with public IP and new port (1.2.3.4:60000)
3. Router stores mapping in NAT table
4. Response arrives at 1.2.3.4:60000
5. Router forwards to 192.168.1.100:5000

## Interview Practice
- [ ] Calculate network address and broadcast for given IP/mask
- [ ] Explain difference between classful and CIDR
- [ ] Describe all major IPv4 header fields
- [ ] How does TTL prevent routing loops?
- [ ] Explain longest prefix match with example
- [ ] Describe ping and traceroute operation
- [ ] What is NAT and why is it needed?
- [ ] Calculate number of usable hosts for /26 network
- [ ] How does a router make forwarding decision?
- [ ] Explain fragmentation process
