# Networking

## Overview

Networking is the foundation of distributed systems, enabling communication between different components, services, and users across the globe. Understanding networking concepts, protocols, and architectures is essential for designing systems that are performant, reliable, and scalable. This section covers the OSI model, network architectures, protocols, and key networking components.

## OSI Model and Its Layers

The OSI (Open Systems Interconnection) model is a conceptual framework that standardizes the functions of a telecommunication or computing system into seven abstraction layers.

### The Seven Layers:

```
7. Application Layer    ← User Applications (HTTP, FTP, SMTP)
6. Presentation Layer   ← Data Translation (SSL/TLS, Compression)
5. Session Layer        ← Session Management (NetBIOS, RPC)
4. Transport Layer      ← End-to-End Communication (TCP, UDP)
3. Network Layer        ← Routing (IP, ICMP, OSPF)
2. Data Link Layer      ← Node-to-Node Communication (Ethernet, WiFi)
1. Physical Layer       ← Physical Transmission (Cables, Radio Waves)
```

### Layer 1: Physical Layer

**Purpose**: Transmit raw bits over a physical medium

**Responsibilities:**
- **Electrical Signals**: Convert bits to electrical, optical, or radio signals
- **Physical Topology**: Define how devices are physically connected
- **Transmission Media**: Cables, fiber optics, wireless frequencies
- **Hardware Specifications**: Voltage levels, timing, physical connectors

**Examples:**
- **Ethernet Cables**: Cat5e, Cat6, fiber optic cables
- **Wireless**: WiFi radio frequencies, Bluetooth
- **Connectors**: RJ45, USB, HDMI
- **Hubs**: Simple signal repeaters (largely obsolete)

**Key Concepts:**
- **Bandwidth**: Maximum data transmission rate
- **Attenuation**: Signal degradation over distance
- **Interference**: External factors affecting signal quality
- **Duplex**: Full-duplex (bidirectional) vs half-duplex (one direction at a time)

### Layer 2: Data Link Layer

**Purpose**: Provide node-to-node communication and error detection/correction

**Responsibilities:**
- **Framing**: Package bits into frames with headers and trailers
- **Error Detection**: Detect transmission errors using checksums
- **Flow Control**: Manage data transmission rate between nodes
- **MAC Addressing**: Use hardware addresses for local network communication

**Sublayers:**
- **LLC (Logical Link Control)**: Interface with network layer
- **MAC (Media Access Control)**: Control access to transmission medium

**Examples:**
- **Ethernet**: Most common LAN technology
- **WiFi (802.11)**: Wireless LAN technology
- **PPP**: Point-to-Point Protocol for direct connections
- **Switches**: Forward frames based on MAC addresses

**Frame Structure:**
```
[Preamble][Destination MAC][Source MAC][Type/Length][Data][FCS]
```

### Layer 3: Network Layer

**Purpose**: Route packets between different networks

**Responsibilities:**
- **Logical Addressing**: Use IP addresses for global identification
- **Routing**: Determine best path for packet delivery
- **Packet Forwarding**: Forward packets toward destination
- **Fragmentation**: Break large packets into smaller ones if needed

**Key Protocols:**
- **IP (Internet Protocol)**: Primary network layer protocol
- **ICMP**: Internet Control Message Protocol for error reporting
- **OSPF**: Open Shortest Path First routing protocol
- **BGP**: Border Gateway Protocol for inter-domain routing

**IP Address Types:**
- **IPv4**: 32-bit addresses (192.168.1.1)
- **IPv6**: 128-bit addresses (2001:db8::1)
- **Public**: Globally routable addresses
- **Private**: Local network addresses (RFC 1918)

**Routing Concepts:**
```
Source → [Router 1] → [Router 2] → [Router 3] → Destination
         ↓ Routing    ↓ Routing    ↓ Routing
         Table        Table        Table
```

### Layer 4: Transport Layer

**Purpose**: Provide end-to-end communication and reliability

**Responsibilities:**
- **Segmentation**: Break application data into segments
- **Reassembly**: Reconstruct original data from segments
- **Error Recovery**: Detect and recover from transmission errors
- **Flow Control**: Manage data transmission rate end-to-end
- **Multiplexing**: Allow multiple applications to use network simultaneously

**Key Protocols:**

#### TCP (Transmission Control Protocol)
```
Features:
✅ Connection-oriented
✅ Reliable delivery
✅ Ordered delivery
✅ Error detection and correction
✅ Flow control
✅ Congestion control
```

#### UDP (User Datagram Protocol)
```
Features:
✅ Connectionless
✅ Low overhead
✅ Fast transmission
❌ No reliability guarantees
❌ No ordering guarantees
❌ No flow control
```

**Port Numbers:**
- **Well-known Ports**: 0-1023 (HTTP: 80, HTTPS: 443, SSH: 22)
- **Registered Ports**: 1024-49151 (Application-specific)
- **Dynamic Ports**: 49152-65535 (Temporary client ports)

### Layer 5: Session Layer

**Purpose**: Manage sessions between applications

**Responsibilities:**
- **Session Establishment**: Set up communication sessions
- **Session Management**: Maintain session state
- **Session Termination**: Properly close sessions
- **Synchronization**: Provide checkpoints for recovery

**Examples:**
- **NetBIOS**: Network Basic Input/Output System
- **RPC**: Remote Procedure Call
- **SQL Sessions**: Database connection sessions
- **SMB**: Server Message Block for file sharing

**Session Management:**
```
Client → [Session Request] → Server
Client ← [Session Accept] ← Server
Client ↔ [Data Exchange] ↔ Server
Client → [Session Close] → Server
```

### Layer 6: Presentation Layer

**Purpose**: Handle data translation, encryption, and compression

**Responsibilities:**
- **Data Translation**: Convert between different data formats
- **Encryption/Decryption**: Secure data transmission
- **Compression/Decompression**: Reduce data size for transmission
- **Character Encoding**: Handle different character sets

**Examples:**
- **SSL/TLS**: Secure communication protocols
- **JPEG/PNG**: Image compression formats
- **MPEG**: Video compression formats
- **ASCII/Unicode**: Character encoding standards

**Data Transformation:**
```
Application Data → [Encrypt] → [Compress] → [Encode] → Network
Application Data ← [Decrypt] ← [Decompress] ← [Decode] ← Network
```

### Layer 7: Application Layer

**Purpose**: Provide network services directly to applications

**Responsibilities:**
- **Network Process to Application**: Interface between applications and network
- **Service Advertisement**: Announce available network services
- **User Authentication**: Verify user identity
- **Data Formatting**: Present data in application-specific formats

**Examples:**
- **HTTP/HTTPS**: Web browsing and APIs
- **FTP/SFTP**: File transfer
- **SMTP/IMAP**: Email services
- **DNS**: Domain name resolution
- **DHCP**: Dynamic IP address assignment

## Network Architecture

Network architecture defines how network components are organized and how they communicate.

### Client-Server Architecture

In client-server architecture, clients request services from centralized servers.

#### Structure:
```
[Client 1] ←→ [Server]
[Client 2] ←→ [Server]
[Client 3] ←→ [Server]
```

#### Characteristics:
- **Centralized**: Server provides services to multiple clients
- **Asymmetric**: Clients request, servers respond
- **Scalable**: Can add more clients easily
- **Manageable**: Centralized administration and control

#### Types:

##### 1. Two-Tier Architecture
```
[Client (Presentation + Logic)] ←→ [Database Server]
```

**Advantages:**
✅ Simple architecture
✅ Direct database access
✅ Good performance for small applications

**Disadvantages:**
❌ Limited scalability
❌ Business logic mixed with presentation
❌ Database changes affect all clients

##### 2. Three-Tier Architecture
```
[Client (Presentation)] ←→ [Application Server (Logic)] ←→ [Database Server]
```

**Advantages:**
✅ Separation of concerns
✅ Better scalability
✅ Centralized business logic
✅ Easier maintenance

**Disadvantages:**
❌ More complex
❌ Additional network hops
❌ Single point of failure (app server)

##### 3. N-Tier Architecture
```
[Client] ←→ [Web Server] ←→ [App Server] ←→ [Database Server]
                ↓
         [Cache Server] [Message Queue] [File Server]
```

**Advantages:**
✅ Highly scalable
✅ Flexible architecture
✅ Specialized servers for different functions
✅ Better fault tolerance

**Use Cases:**
- **Web Applications**: E-commerce, social media
- **Enterprise Applications**: ERP, CRM systems
- **Database Applications**: Data warehousing, analytics
- **File Sharing**: Network file systems

### Peer-to-Peer (P2P) Architecture

In P2P architecture, all nodes can act as both clients and servers.

#### Structure:
```
[Peer 1] ←→ [Peer 2]
    ↕         ↕
[Peer 4] ←→ [Peer 3]
```

#### Characteristics:
- **Decentralized**: No central authority
- **Symmetric**: All peers have equal roles
- **Self-organizing**: Network organizes itself
- **Fault Tolerant**: No single point of failure

#### Types:

##### 1. Pure P2P
```
All peers are equal, no central coordination
Examples: Early file-sharing networks
```

##### 2. Hybrid P2P
```
Central server for coordination, P2P for data transfer
Examples: BitTorrent (tracker servers)
```

##### 3. Structured P2P
```
Organized overlay network with specific topology
Examples: Distributed Hash Tables (DHT)
```

#### P2P Advantages:
✅ **Scalability**: Performance improves with more peers
✅ **Fault Tolerance**: No single point of failure
✅ **Cost Effective**: No need for expensive servers
✅ **Resource Sharing**: Utilize distributed resources

#### P2P Disadvantages:
❌ **Security**: Harder to secure and control
❌ **Quality of Service**: Inconsistent performance
❌ **Discovery**: Finding resources can be complex
❌ **Management**: Difficult to manage and monitor

#### Use Cases:
- **File Sharing**: BitTorrent, IPFS
- **Cryptocurrency**: Bitcoin, Ethereum
- **Communication**: Skype (original architecture)
- **Content Delivery**: Distributed CDNs

## Internet Connection

Understanding how devices connect to the internet involves several key components.

### ISP (Internet Service Provider)

ISPs provide internet access to consumers and businesses.

#### ISP Hierarchy:
```
Tier 1 ISPs (Global Backbone)
    ↓
Tier 2 ISPs (Regional)
    ↓
Tier 3 ISPs (Local)
    ↓
End Users
```

#### Tier 1 ISPs:
- **Global Reach**: Worldwide network infrastructure
- **Peering**: Exchange traffic with other Tier 1 ISPs
- **No Transit Fees**: Don't pay for internet access
- **Examples**: AT&T, Verizon, NTT, Deutsche Telekom

#### Tier 2 ISPs:
- **Regional Coverage**: Serve specific geographic regions
- **Mixed Model**: Peer with some ISPs, pay others
- **Transit Customers**: Provide internet to smaller ISPs
- **Examples**: Regional telecom companies

#### Tier 3 ISPs:
- **Local Service**: Serve end users and small businesses
- **Purchase Transit**: Pay upstream ISPs for internet access
- **Last Mile**: Provide final connection to customers
- **Examples**: Local cable companies, DSL providers

#### Connection Types:
- **Dial-up**: 56 Kbps over phone lines (obsolete)
- **DSL**: Digital Subscriber Line over phone lines
- **Cable**: High-speed over cable TV infrastructure
- **Fiber**: Fiber optic cables for highest speeds
- **Satellite**: Internet via satellite (rural areas)
- **Mobile**: 4G/5G cellular networks

### DNS (Domain Name System)

DNS translates human-readable domain names into IP addresses.

#### DNS Hierarchy:
```
Root Servers (.)
    ↓
Top-Level Domain Servers (.com, .org, .net)
    ↓
Authoritative Name Servers (example.com)
    ↓
Local DNS Resolvers
```

#### DNS Resolution Process:
```
1. User types "www.example.com" in browser
2. Browser checks local cache
3. If not cached, query local DNS resolver
4. Resolver queries root servers for .com servers
5. Resolver queries .com servers for example.com servers
6. Resolver queries example.com servers for www.example.com
7. IP address returned to browser
8. Browser connects to IP address
```

#### DNS Record Types:
- **A Record**: Maps domain to IPv4 address
- **AAAA Record**: Maps domain to IPv6 address
- **CNAME**: Canonical name (alias) record
- **MX**: Mail exchange server record
- **NS**: Name server record
- **TXT**: Text record for various purposes
- **SOA**: Start of authority record

#### DNS Caching:
```
Browser Cache (minutes)
    ↓
OS Cache (minutes to hours)
    ↓
Router Cache (hours)
    ↓
ISP Cache (hours to days)
```

#### DNS Security:
- **DNSSEC**: DNS Security Extensions for authentication
- **DNS over HTTPS (DoH)**: Encrypted DNS queries
- **DNS over TLS (DoT)**: TLS-encrypted DNS queries

### CDN (Content Delivery Network)

CDNs distribute content across multiple geographic locations to improve performance.

#### CDN Architecture:
```
Origin Server (Primary content)
    ↓
CDN Edge Servers (Cached content)
    ↓
Users (Served from nearest edge)
```

#### How CDNs Work:
```
1. User requests content (e.g., image, video)
2. DNS resolves to nearest CDN edge server
3. Edge server checks if content is cached
4. If cached, serve from edge (cache hit)
5. If not cached, fetch from origin (cache miss)
6. Cache content at edge for future requests
7. Serve content to user
```

#### CDN Benefits:
✅ **Reduced Latency**: Content served from nearby locations
✅ **Improved Performance**: Faster load times
✅ **Reduced Bandwidth**: Less traffic to origin server
✅ **High Availability**: Multiple servers provide redundancy
✅ **DDoS Protection**: Distributed infrastructure absorbs attacks

#### CDN Use Cases:
- **Static Content**: Images, CSS, JavaScript files
- **Video Streaming**: On-demand and live video
- **Software Distribution**: Downloads and updates
- **API Acceleration**: Cache API responses
- **Security**: Web application firewall (WAF)

#### Popular CDN Providers:
- **Cloudflare**: Global CDN with security features
- **Amazon CloudFront**: AWS's CDN service
- **Akamai**: One of the largest CDN networks
- **Google Cloud CDN**: Google's CDN offering
- **Microsoft Azure CDN**: Microsoft's CDN service

## Application Layer Protocols

Application layer protocols define how applications communicate over networks.

### AMQP (Advanced Message Queuing Protocol)

AMQP is an open standard for message-oriented middleware.

#### AMQP Components:
```
Producer → [Exchange] → [Queue] → Consumer
```

#### Key Concepts:
- **Producer**: Sends messages
- **Exchange**: Routes messages to queues
- **Queue**: Stores messages
- **Consumer**: Receives messages
- **Binding**: Rules for routing messages

#### Exchange Types:
- **Direct**: Route based on routing key
- **Topic**: Route based on pattern matching
- **Fanout**: Broadcast to all bound queues
- **Headers**: Route based on message headers

#### AMQP Features:
✅ **Reliability**: Message acknowledgments and persistence
✅ **Routing**: Flexible message routing
✅ **Security**: Authentication and authorization
✅ **Interoperability**: Cross-platform compatibility

### HTTP/HTTPS

HTTP is the foundation of web communication.

#### HTTP Versions:

##### HTTP/1.1
```
Features:
- Persistent connections
- Chunked transfer encoding
- Host header (virtual hosting)
- Cache control
```

##### HTTP/2
```
Features:
- Binary protocol (not text-based)
- Multiplexing (multiple requests per connection)
- Server push
- Header compression (HPACK)
```

##### HTTP/3
```
Features:
- Built on QUIC (UDP-based)
- Improved performance over unreliable networks
- Reduced connection establishment time
- Better handling of packet loss
```

#### HTTPS (HTTP Secure)
```
HTTP + TLS/SSL = HTTPS
- Encryption of data in transit
- Server authentication
- Data integrity verification
```

### WebSockets

WebSockets provide full-duplex communication over a single TCP connection.

#### WebSocket Handshake:
```
Client → HTTP Upgrade Request → Server
Client ← HTTP 101 Switching Protocols ← Server
Client ↔ WebSocket Messages ↔ Server
```

#### WebSocket Features:
- **Full Duplex**: Both client and server can send messages
- **Low Latency**: No HTTP overhead after connection
- **Persistent**: Connection stays open
- **Binary Support**: Can send binary data

#### Use Cases:
- **Real-time Chat**: Instant messaging
- **Live Updates**: Stock prices, sports scores
- **Gaming**: Real-time multiplayer games
- **Collaboration**: Shared documents

### gRPC / WebRTC

#### gRPC (Google Remote Procedure Call)
```
Features:
- HTTP/2 based
- Protocol Buffers for serialization
- Bidirectional streaming
- Built-in authentication
```

#### WebRTC (Web Real-Time Communication)
```
Features:
- Peer-to-peer communication
- Real-time audio/video
- Data channels
- NAT traversal
```

### File Transfer Protocols

#### FTP (File Transfer Protocol)
```
Control Connection (Port 21):
Client ←→ Server (Commands and responses)

Data Connection (Port 20 or dynamic):
Client ←→ Server (File transfers)
```

**FTP Modes:**
- **Active Mode**: Server initiates data connection
- **Passive Mode**: Client initiates data connection

#### SFTP (SSH File Transfer Protocol)
```
Features:
- Encrypted file transfers
- Built on SSH protocol
- Authentication and authorization
- File system operations
```

### Other Protocols

#### DASH (Dynamic Adaptive Streaming over HTTP)
```
Purpose: Adaptive video streaming
Features:
- Multiple quality levels
- Adaptive bitrate based on network conditions
- HTTP-based delivery
```

#### SMTP (Simple Mail Transfer Protocol)
```
Purpose: Email delivery
Process:
Client → [SMTP Server] → [Recipient SMTP Server] → Recipient
```

## Transport Layer Protocols

### TCP (Transmission Control Protocol)

TCP provides reliable, ordered delivery of data.

#### TCP Features:
- **Connection-oriented**: Establish connection before data transfer
- **Reliable**: Guaranteed delivery with acknowledgments
- **Ordered**: Data delivered in correct sequence
- **Flow Control**: Prevent overwhelming receiver
- **Congestion Control**: Adapt to network conditions

#### TCP Connection Process:
```
Three-Way Handshake:
Client → [SYN] → Server
Client ← [SYN-ACK] ← Server
Client → [ACK] → Server
Connection Established
```

#### TCP Termination:
```
Four-Way Handshake:
Client → [FIN] → Server
Client ← [ACK] ← Server
Client ← [FIN] ← Server
Client → [ACK] → Server
Connection Closed
```

### UDP (User Datagram Protocol)

UDP provides fast, connectionless communication.

#### UDP Characteristics:
- **Connectionless**: No connection establishment
- **Unreliable**: No delivery guarantees
- **Unordered**: No sequence guarantees
- **Low Overhead**: Minimal protocol overhead
- **Fast**: No connection setup or reliability mechanisms

#### When to Use UDP:
- **Real-time Applications**: Gaming, video streaming
- **DNS Queries**: Fast lookups
- **DHCP**: Dynamic IP assignment
- **Broadcast/Multicast**: One-to-many communication

### QUIC (Quick UDP Internet Connections)

QUIC is a transport protocol built on UDP, designed to improve web performance.

#### QUIC Features:
- **Built on UDP**: Avoids TCP limitations
- **Multiplexing**: Multiple streams without head-of-line blocking
- **Encryption**: Built-in TLS 1.3 encryption
- **Fast Connection**: 0-RTT connection establishment
- **Mobility**: Connection survives IP address changes

#### QUIC vs TCP:
```
TCP:
Connection Setup: 3 RTTs (TCP + TLS)
Head-of-line blocking: Yes
Multiplexing: No (HTTP/2 workaround)

QUIC:
Connection Setup: 0-1 RTT
Head-of-line blocking: No
Multiplexing: Native support
```

## Network Layer Protocols

### IP (Internet Protocol)

IP is the primary protocol for routing packets across networks.

#### IPv4
```
Address Format: 32-bit (192.168.1.1)
Address Space: ~4.3 billion addresses
Header Size: 20-60 bytes
Fragmentation: Routers can fragment packets
```

#### IPv6
```
Address Format: 128-bit (2001:db8::1)
Address Space: ~340 undecillion addresses
Header Size: 40 bytes (fixed)
Fragmentation: Only source can fragment
```

#### IP Addressing:
- **Public Addresses**: Globally routable
- **Private Addresses**: Local network use (RFC 1918)
- **Loopback**: 127.0.0.1 (localhost)
- **Multicast**: One-to-many communication
- **Broadcast**: One-to-all communication (IPv4 only)

## Proxy

Proxies act as intermediaries between clients and servers.

### What is a Proxy and Why Use It?

A proxy server is an intermediary server that forwards requests from clients to other servers and returns the responses back to clients.

#### Benefits of Using Proxies:
✅ **Security**: Hide client identity and filter malicious content
✅ **Performance**: Cache frequently requested content
✅ **Access Control**: Restrict access to certain resources
✅ **Load Balancing**: Distribute requests across multiple servers
✅ **Monitoring**: Log and analyze network traffic
✅ **Bandwidth Optimization**: Compress and optimize content

### Forward Proxy

A forward proxy sits between clients and the internet, forwarding client requests to servers.

#### Architecture:
```
[Client] → [Forward Proxy] → [Internet] → [Server]
```

#### Use Cases:
- **Corporate Networks**: Control employee internet access
- **Content Filtering**: Block inappropriate websites
- **Caching**: Cache frequently accessed content
- **Anonymity**: Hide client IP addresses
- **Bandwidth Control**: Limit bandwidth usage

#### Example Configuration:
```
Client Configuration:
Proxy Server: proxy.company.com
Port: 8080
Authentication: username/password
```

### Reverse Proxy

A reverse proxy sits between the internet and servers, forwarding client requests to backend servers.

#### Architecture:
```
[Client] → [Internet] → [Reverse Proxy] → [Backend Servers]
```

#### Use Cases:
- **Load Balancing**: Distribute requests across multiple servers
- **SSL Termination**: Handle SSL encryption/decryption
- **Caching**: Cache responses from backend servers
- **Security**: Hide backend server details
- **Compression**: Compress responses to reduce bandwidth

#### Popular Reverse Proxies:
- **Nginx**: High-performance web server and reverse proxy
- **HAProxy**: Load balancer and reverse proxy
- **Apache HTTP Server**: Web server with proxy modules
- **Cloudflare**: Cloud-based reverse proxy service

### Differences Between Forward and Reverse Proxy

| Aspect | Forward Proxy | Reverse Proxy |
|--------|---------------|---------------|
| **Position** | Between client and internet | Between internet and servers |
| **Purpose** | Hide client from server | Hide server from client |
| **Configuration** | Client-side configuration | Server-side configuration |
| **Use Case** | Corporate internet access | Load balancing, caching |
| **Visibility** | Server doesn't see client IP | Client doesn't see server details |
| **Control** | Client/network admin control | Server/service admin control |

#### Traffic Flow Comparison:
```
Forward Proxy:
Client → Proxy → Server
(Server sees proxy IP, not client IP)

Reverse Proxy:
Client → Proxy → Backend Server
(Client sees proxy, not backend server)
```

## Conclusion

Networking is fundamental to distributed systems, providing the infrastructure for communication between components. Key takeaways:

1. **Understand the Stack**: OSI model provides framework for understanding network communication
2. **Choose Right Architecture**: Client-server vs P2P based on requirements
3. **Select Appropriate Protocols**: Different protocols serve different purposes
4. **Plan for Performance**: Use CDNs, caching, and optimization techniques
5. **Consider Security**: Implement proper authentication, encryption, and access controls
6. **Design for Scale**: Use load balancing, proxies, and distributed architectures

Understanding these networking concepts enables you to design systems that communicate efficiently, securely, and reliably across networks of any scale.

## Next Steps

In the following sections, we'll explore:
- **Caching**: Performance optimization through strategic data storage
- **Security**: Advanced security patterns and practices
- **Monitoring**: Network observability and performance monitoring