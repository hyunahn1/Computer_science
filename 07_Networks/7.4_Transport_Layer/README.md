# 4.4 Transport Layer (TCP & UDP)

## Transport Layer Overview

### Purpose
- **End-to-end communication**
  - Process-to-process delivery
  - Port numbers identify applications
  - Multiplexing/demultiplexing

### Port Numbers
- **16-bit value (0-65535)**
  - Identifies specific application/service
  
- **Categories**
  - Well-known ports: 0-1023 (HTTP=80, SSH=22)
  - Registered ports: 1024-49151 (custom services)
  - Dynamic/private ports: 49152-65535 (client-side)

### Socket
- **IP address + Port number**
  - Uniquely identifies endpoint
  - Example: `192.168.1.100:80`

## UDP (User Datagram Protocol)

### Characteristics
- **Connectionless**
  - No connection establishment
  - No handshake
  - Just send data
  
- **Unreliable**
  - No delivery guarantee
  - No acknowledgment
  - Packets may be lost, duplicated, or reordered
  
- **Unordered**
  - No sequence numbers
  - Receive order may differ from send order
  
- **No flow/congestion control**
  - Sender doesn't adapt to network conditions

### UDP Header (8 bytes - simple!)
```
0      7 8     15 16    23 24    31
+--------+--------+--------+--------+
|   Source Port   |  Dest Port      |
+--------+--------+--------+--------+
|     Length      |    Checksum     |
+--------+--------+--------+--------+
|          Data (payload)           |
```

#### Fields
- **Source Port (2 bytes)**: Sender's port
- **Destination Port (2 bytes)**: Receiver's port
- **Length (2 bytes)**: Header + data length
- **Checksum (2 bytes)**: Error detection (optional in IPv4)

### When to Use UDP
- **Speed over reliability**
  - Low overhead (only 8-byte header)
  - No connection setup delay
  
- **Use Cases**
  - **DNS**: Quick query/response
  - **DHCP**: IP address assignment
  - **Streaming**: Video/audio (minor loss acceptable)
  - **Online gaming**: Real-time, low latency
  - **VoIP**: Voice calls (slight loss ok)
  - **SNMP**: Network monitoring
  - **TFTP**: Trivial File Transfer

### Advantages
- Fast (no overhead)
- Simple
- Small header
- Broadcast/multicast support

### Disadvantages
- No reliability
- No ordering
- No flow control
- Application must handle errors

## TCP (Transmission Control Protocol)

### Characteristics
- **Connection-oriented**
  - Must establish connection first
  - 3-way handshake
  
- **Reliable**
  - Guaranteed delivery
  - Acknowledgments
  - Retransmission on loss
  
- **Ordered**
  - Sequence numbers
  - Data delivered in order sent
  
- **Flow control**
  - Sliding window
  - Prevent overwhelming receiver
  
- **Congestion control**
  - Slow start, congestion avoidance
  - Adapt to network conditions

### TCP Header (20-60 bytes)
```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |       |C|E|U|A|P|R|S|F|                               |
| Offset| Resvd |W|C|R|C|S|S|Y|I|            Window             |
|       |       |R|E|G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (if any)                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### Key Header Fields

#### Sequence Number (32 bits)
- **Byte number of first byte in segment**
  - ISN (Initial Sequence Number) chosen randomly
  - Increments by data bytes sent
  
- **Example**
  - Segment 1: SEQ=1000, 100 bytes → next SEQ=1100
  - Segment 2: SEQ=1100, 200 bytes → next SEQ=1300

#### Acknowledgment Number (32 bits)
- **Next expected byte**
  - ACK=1100 means "received up to 1099, send 1100 next"
  - Cumulative acknowledgment

#### Flags (Control Bits)
- **URG**: Urgent pointer valid
- **ACK**: Acknowledgment number valid
- **PSH**: Push data to application immediately
- **RST**: Reset connection (error/abort)
- **SYN**: Synchronize sequence numbers (connection setup)
- **FIN**: Finish, no more data (connection teardown)

#### Window Size (16 bits)
- **Receive window**
  - How many bytes receiver can accept
  - Flow control mechanism
  - Dynamically adjusted

#### Checksum (16 bits)
- Error detection (mandatory)

#### Options (variable)
- **MSS (Maximum Segment Size)**: Largest data chunk
- **Window Scale**: Scale window beyond 65535
- **Timestamps**: RTT measurement
- **SACK (Selective ACK)**: Acknowledge non-contiguous data

## 3-Way Handshake (Connection Establishment)

### Process
```
Client                           Server
   |                                |
   |  SYN (SEQ=x)                   |
   |------------------------------->|
   |                                | LISTEN → SYN-RECEIVED
   |      SYN-ACK (SEQ=y, ACK=x+1)  |
   |<-------------------------------|
ESTABLISHED                         |
   |  ACK (SEQ=x+1, ACK=y+1)        |
   |------------------------------->|
   |                                | ESTABLISHED
```

### Step-by-Step
1. **Client → Server: SYN**
   - SYN flag set
   - SEQ = x (random ISN)
   - "I want to connect"
   
2. **Server → Client: SYN-ACK**
   - SYN and ACK flags set
   - SEQ = y (server's ISN)
   - ACK = x + 1
   - "I accept, here's my ISN"
   
3. **Client → Server: ACK**
   - ACK flag set
   - SEQ = x + 1
   - ACK = y + 1
   - "Connection established"

### Why 3-Way?
- **Synchronize sequence numbers**
  - Both sides agree on starting SEQ
  - Prevents old duplicate packets
  
- **Full-duplex**
  - Both directions initialized

## 4-Way Handshake (Connection Termination)

### Process
```
Client                           Server
   |                                |
   |  FIN (SEQ=x)                   |
   |------------------------------->|
   |                                | CLOSE-WAIT
   |      ACK (ACK=x+1)             |
   |<-------------------------------|
   |                                |
   |      FIN (SEQ=y)               |
   |<-------------------------------|
   |                                | LAST-ACK
   |  ACK (ACK=y+1)                 |
   |------------------------------->|
TIME-WAIT                           | CLOSED
   |                                |
```

### Steps
1. **Client → Server: FIN**
   - "I'm done sending"
   
2. **Server → Client: ACK**
   - "I received your FIN"
   - Server can still send data (half-close)
   
3. **Server → Client: FIN**
   - "I'm done sending too"
   
4. **Client → Server: ACK**
   - "I received your FIN"
   - Client enters TIME-WAIT

### TIME-WAIT State
- **Duration**: 2 × MSL (Maximum Segment Lifetime)
  - Typically 2-4 minutes
  
- **Purpose**
  - Allow late packets to expire
  - Ensure last ACK received
  
- **Problem**: Port not reusable immediately

## TCP State Machine

### States
- **CLOSED**: No connection
- **LISTEN**: Server waiting for connection
- **SYN-SENT**: Client sent SYN, waiting for SYN-ACK
- **SYN-RECEIVED**: Server received SYN, sent SYN-ACK
- **ESTABLISHED**: Connection active, data transfer
- **FIN-WAIT-1**: Sent FIN, waiting for ACK
- **FIN-WAIT-2**: Received ACK of FIN, waiting for peer's FIN
- **CLOSE-WAIT**: Received FIN, waiting for application to close
- **CLOSING**: Both sides closing simultaneously
- **LAST-ACK**: Sent FIN after receiving FIN, waiting for ACK
- **TIME-WAIT**: Waiting for late packets to expire

### State Transitions
```
CLOSED → LISTEN (server)
CLOSED → SYN-SENT → ESTABLISHED (client)
ESTABLISHED → FIN-WAIT-1 → FIN-WAIT-2 → TIME-WAIT → CLOSED
ESTABLISHED → CLOSE-WAIT → LAST-ACK → CLOSED
```

## Flow Control (Sliding Window)

### Purpose
- **Prevent receiver overflow**
  - Receiver has limited buffer
  - Sender must not send too fast

### Mechanism
- **Receiver advertises window size**
  - Window = available buffer space
  - Sender can send up to window bytes without ACK
  
- **Dynamic adjustment**
  - Window shrinks as buffer fills
  - Window grows as application reads data
  - Zero window = stop sending

### Example
```
Sender                          Receiver (Window=1000)
   |  SEQ=1, 500 bytes              |
   |---------------------------------|
   |         ACK=501, WIN=500        | (buffer 500/1000 used)
   |  SEQ=501, 500 bytes             |
   |---------------------------------|
   |         ACK=1001, WIN=0         | (buffer full!)
   |  (must wait...)                 |
   |         ACK=1001, WIN=800       | (app read data)
   |  SEQ=1001, 800 bytes            |
```

## Congestion Control

### Problem
- **Network congestion**
  - Too much traffic
  - Routers drop packets
  - Retransmissions worsen congestion

### Congestion Window (cwnd)
- **Sender's estimate of network capacity**
  - Separate from receiver's window
  - Actual window = min(cwnd, receiver window)

### Algorithms

#### Slow Start
- **Initial phase**
  - Start with cwnd = 1 MSS
  - Double cwnd each RTT (exponential growth)
  - Continue until threshold or loss
  
- **Exponential growth**
  - RTT 1: cwnd = 1
  - RTT 2: cwnd = 2
  - RTT 3: cwnd = 4
  - RTT 4: cwnd = 8

#### Congestion Avoidance
- **After threshold**
  - Linear growth (additive increase)
  - Increment cwnd by 1 MSS per RTT
  - Probe for available bandwidth

#### Fast Retransmit
- **Detect loss quickly**
  - 3 duplicate ACKs = packet lost
  - Retransmit immediately (don't wait for timeout)

#### Fast Recovery
- **After fast retransmit**
  - Cut cwnd in half (multiplicative decrease)
  - Resume congestion avoidance
  - Don't drop to 1 (slow start)

### AIMD (Additive Increase, Multiplicative Decrease)
- Increase slowly on success
- Decrease quickly on congestion
- Fair bandwidth sharing

## TCP vs UDP Comparison

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented | Connectionless |
| Reliability | Reliable | Unreliable |
| Ordering | Ordered | Unordered |
| Speed | Slower (overhead) | Faster |
| Header size | 20-60 bytes | 8 bytes |
| Flow control | Yes | No |
| Congestion control | Yes | No |
| Use case | File transfer, web, email | Streaming, gaming, DNS |
| Error checking | Yes (retransmit) | Yes (but no retransmit) |
| Broadcast | No | Yes |

## Interview Practice
- [ ] Explain 3-way handshake in detail
- [ ] Why is 3-way handshake needed? (why not 2-way?)
- [ ] Describe TCP state machine
- [ ] What is TIME-WAIT and why is it needed?
- [ ] Explain flow control vs congestion control
- [ ] Describe slow start and congestion avoidance
- [ ] TCP vs UDP: when to use each?
- [ ] How does TCP detect and handle packet loss?
- [ ] What is silly window syndrome?
- [ ] Explain TCP sequence and acknowledgment numbers
