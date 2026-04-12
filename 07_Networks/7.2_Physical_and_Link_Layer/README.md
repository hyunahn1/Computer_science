# 4.2 Physical & Data Link Layer

## Physical Layer (Layer 1)

### Function
- **Bit Transmission**
  - Convert data to electrical/optical/radio signals
  - Define physical specifications
  - No understanding of data meaning

### Transmission Media

#### Guided Media (Wired)

##### Twisted Pair Cable
- **UTP (Unshielded Twisted Pair)**
  - Cat5e: 1 Gbps, 100m
  - Cat6: 10 Gbps, 55m
  - Cat6a: 10 Gbps, 100m
  - Cheap, easy to install
  - Susceptible to EMI
  
- **STP (Shielded Twisted Pair)**
  - Foil/braided shield
  - Better EMI protection
  - More expensive

##### Coaxial Cable
- Central conductor + shield
- Better shielding than twisted pair
- Used in cable TV, old Ethernet (10BASE2)
- Legacy in most LANs

##### Fiber Optic Cable
- **Single-Mode Fiber (SMF)**
  - Single light path
  - Long distance (100+ km)
  - Expensive
  - Laser light source
  
- **Multi-Mode Fiber (MMF)**
  - Multiple light paths
  - Shorter distance (2 km)
  - Cheaper than SMF
  - LED light source
  
- **Advantages**
  - No EMI
  - Very high bandwidth
  - Secure (hard to tap)
  - Long distance
  
- **Disadvantages**
  - Expensive
  - Fragile
  - Hard to install/terminate

#### Unguided Media (Wireless)
- Radio waves
- Microwaves
- Infrared
- Wi-Fi, Bluetooth, cellular

### Signal Encoding

#### Digital Signaling
- **NRZ (Non-Return to Zero)**
  - High voltage = 1, Low voltage = 0
  - Simple but clock synchronization issue
  
- **Manchester Encoding**
  - Transition in middle of bit
  - High→Low = 1, Low→High = 0
  - Used in Ethernet (10BASE-T)
  - Self-clocking

#### Analog Signaling
- Amplitude modulation (AM)
- Frequency modulation (FM)
- Phase modulation (PM)

### Physical Layer Devices

#### Hub
- Multi-port repeater
- Broadcast all incoming traffic to all ports
- Layer 1 device (no intelligence)
- Half-duplex (collision domain)
- **Obsolete**: replaced by switches

#### Repeater
- Amplifies/regenerates signal
- Extends cable distance
- No filtering or addressing

## Data Link Layer (Layer 2)

### Functions
1. **Framing**
   - Encapsulate network layer packets
   - Add header and trailer
   
2. **Physical Addressing**
   - MAC address
   
3. **Error Detection**
   - CRC (Cyclic Redundancy Check)
   
4. **Flow Control**
   - Prevent overwhelming receiver
   
5. **Access Control**
   - When to transmit on shared medium

### MAC Address (Media Access Control)

#### Format
- **48 bits (6 bytes)**
  - Usually written as 6 hex pairs
  - Example: `00:1A:2B:3C:4D:5E`
  
- **Structure**
  - First 3 bytes: OUI (Organizationally Unique Identifier) - Vendor
  - Last 3 bytes: NIC-specific (unique per device)

#### Special Addresses
- **Broadcast**: `FF:FF:FF:FF:FF:FF`
  - All devices on local network
  
- **Multicast**: First byte LSB = 1
  - Example: `01:00:5E:xx:xx:xx` (IPv4 multicast)

#### MAC vs IP
| MAC Address | IP Address |
|-------------|------------|
| Layer 2 | Layer 3 |
| Physical address | Logical address |
| Burned into NIC | Configured/assigned |
| Local scope | Global scope |
| Doesn't change | Can change |

### Ethernet Frame Structure

#### Ethernet II Frame (Most Common)
```
[Preamble (7)] [SFD (1)] [Dest MAC (6)] [Src MAC (6)] 
[Type (2)] [Payload (46-1500)] [FCS (4)]
```

#### Field Details
- **Preamble (7 bytes)**
  - `10101010` × 7
  - Synchronization
  
- **SFD (Start Frame Delimiter, 1 byte)**
  - `10101011`
  - Indicates start of frame
  
- **Destination MAC (6 bytes)**
  - Recipient's MAC address
  
- **Source MAC (6 bytes)**
  - Sender's MAC address
  
- **Type/Length (2 bytes)**
  - Ethernet II: Type (0x0800 = IPv4, 0x0806 = ARP, 0x86DD = IPv6)
  - 802.3: Length of payload
  
- **Payload (46-1500 bytes)**
  - Data from upper layer
  - Minimum 46 bytes (padded if needed)
  - Maximum 1500 bytes (MTU)
  
- **FCS (Frame Check Sequence, 4 bytes)**
  - CRC-32 checksum
  - Error detection

#### MTU (Maximum Transmission Unit)
- **Standard Ethernet MTU**: 1500 bytes
- Payload size limit (excluding headers)
- **Jumbo Frames**: up to 9000 bytes (not standard)

### Error Detection: CRC

#### Cyclic Redundancy Check
- **Polynomial division**
  - Sender calculates checksum
  - Receiver recalculates and compares
  
- **Properties**
  - Detects all single-bit errors
  - Detects all double-bit errors
  - Detects burst errors up to certain length
  
- **Does NOT correct errors**
  - Only detects
  - Frame is dropped if error found

### CSMA/CD (Carrier Sense Multiple Access with Collision Detection)

#### Traditional Ethernet (Half-Duplex)
1. **Carrier Sense**: Listen before transmitting
2. **Multiple Access**: Shared medium
3. **Collision Detection**: Detect if collision occurs

#### How It Works
1. Device wants to send
2. Listen to medium (carrier sense)
3. If idle, start transmitting
4. While transmitting, monitor for collision
5. If collision detected:
   - Send jam signal
   - Wait random backoff time (exponential)
   - Retry

#### Modern Ethernet (Full-Duplex)
- **No CSMA/CD needed**
- Separate transmit and receive channels
- Switches create point-to-point links
- No collisions possible

### Switches

#### Layer 2 Switch
- **Function**
  - Forward frames based on MAC address
  - Learn MAC addresses dynamically
  - Each port is separate collision domain
  
- **MAC Address Table**
  - Maps MAC addresses to ports
  - Learned by examining source MAC
  - Aged out if not seen (typically 300s)

#### Switch Operation
1. **Learning**
   - Receive frame
   - Record source MAC and ingress port
   
2. **Forwarding**
   - Look up destination MAC in table
   - If found: forward to that port only (unicast)
   - If not found: flood to all ports except ingress (broadcast)
   
3. **Filtering**
   - Drop frame if destination on same port as source

#### Switch vs Hub
| Feature | Hub | Switch |
|---------|-----|--------|
| Layer | 1 (Physical) | 2 (Data Link) |
| Intelligence | None | MAC learning |
| Traffic | Broadcast all | Selective forwarding |
| Collision domains | One (all ports) | One per port |
| Bandwidth | Shared | Dedicated per port |
| Security | Low | Better |

### VLANs (Virtual LANs)

#### Purpose
- **Logical segmentation**
  - Divide switch into separate broadcast domains
  - Without physical separation
  
- **Benefits**
  - Security (isolate traffic)
  - Performance (reduce broadcast)
  - Flexibility (logical grouping)

#### VLAN Tagging (802.1Q)
- **4-byte tag inserted in Ethernet frame**
  - 12 bits for VLAN ID (4096 VLANs)
  - 3 bits for priority
  
- **Trunk ports**
  - Carry multiple VLANs
  - Tagged frames
  
- **Access ports**
  - Single VLAN
  - Untagged frames

### ARP (Address Resolution Protocol)

#### Purpose
- **Map IP address to MAC address**
  - Layer 3 to Layer 2 translation
  - Local network only

#### Operation
1. Host A wants to send to IP 192.168.1.2
2. Checks ARP cache (no entry)
3. Broadcasts ARP request: "Who has 192.168.1.2?"
4. Host B (192.168.1.2) replies: "I am at MAC aa:bb:cc:dd:ee:ff"
5. Host A caches the mapping
6. Host A sends frame to that MAC

#### ARP Cache
- Temporary storage of IP-MAC mappings
- Reduces ARP traffic
- Entries time out (typically 2-20 minutes)
- View with: `arp -a` (Windows/Linux)

#### RARP (Reverse ARP)
- Obsolete
- Get IP address from MAC
- Replaced by DHCP

## Interview Practice
- [ ] Explain Ethernet frame structure
- [ ] How does a switch learn MAC addresses?
- [ ] Compare hub vs switch
- [ ] Describe CSMA/CD process
- [ ] Why is CRC used instead of simple checksum?
- [ ] Explain ARP protocol step by step
- [ ] What is the difference between collision domain and broadcast domain?
- [ ] How do VLANs work?
- [ ] Why is minimum Ethernet payload 46 bytes?
- [ ] Full-duplex vs half-duplex Ethernet
