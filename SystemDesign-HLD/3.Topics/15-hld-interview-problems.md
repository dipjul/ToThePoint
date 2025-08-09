# HLD Interview Problems

## Overview

High Level Design interviews test your ability to design large-scale distributed systems. This section covers common interview problems across three difficulty levels, providing structured approaches and detailed solutions that integrate all concepts from previous sections.

## Level-1 Problems (7 topics)

### Design Rate Limiter

**Problem**: Design a rate limiter for API requests.

**Requirements**:
- Handle 10M requests/second
- Support different limiting rules (per user, IP, API key)
- Low latency (< 1ms overhead)
- Distributed environment

**Solution Architecture**:
```
Client → Load Balancer → Rate Limiter → API Servers
                           ↓
                    Redis Cluster
```

**Key Components**:
- **Algorithm**: Token bucket with Redis
- **Storage**: Redis with Lua scripts for atomicity
- **Rules Engine**: Configurable rate limiting policies
- **Monitoring**: Metrics for blocked/allowed requests

### Design Consistent Hashing

**Problem**: Distribute data across multiple servers with minimal redistribution.

**Requirements**:
- Minimize data movement when servers added/removed
- Handle server failures gracefully
- Support 1000+ servers
- Load balancing with virtual nodes

**Implementation Approach**:
```python
class ConsistentHash:
    def __init__(self, replicas=150):
        self.replicas = replicas
        self.ring = {}
        self.sorted_keys = []
        
    def add_node(self, node):
        for i in range(self.replicas):
            key = self.hash(f"{node}:{i}")
            self.ring[key] = node
            self.sorted_keys.append(key)
        self.sorted_keys.sort()
```

### Design Key Value Store

**Problem**: Build a distributed key-value store like DynamoDB.

**Architecture**:
```
Clients → Load Balancer → API Gateway → Storage Nodes
                                ↓
                    Consistent Hashing Ring
                                ↓
                [Node1] [Node2] [Node3] ... [NodeN]
```

**Key Features**:
- Consistent hashing for partitioning
- Quorum-based replication (N=3, W=2, R=2)
- Vector clocks for conflict resolution
- Gossip protocol for failure detection

### Design Unique ID Generator

**Problem**: Generate unique IDs in distributed environment.

**Solution - Snowflake Algorithm**:
```
64-bit ID: [1 unused][41 timestamp][10 machine][12 sequence]
```

**Benefits**:
- Time-ordered IDs
- 10,000 IDs/second per machine
- No coordination between machines
- Compact 64-bit format

### Design URL Shortener

**Problem**: Build a URL shortening service like bit.ly.

**System Flow**:
```
Write: Long URL → Base62 Encoding → Store → Short URL
Read: Short URL → DB Lookup → Redirect
```

**Key Decisions**:
- Base62 encoding for short URLs
- Redis caching for popular URLs
- Database sharding by short_url
- Analytics tracking for clicks

### Design Web Crawler

**Problem**: Build a web crawler for search engines.

**Architecture**:
```
Seed URLs → URL Frontier → Fetcher → Content Processor → Storage
              ↓              ↓           ↓
        Priority Queue   Rate Limiter  Duplicate Filter
```

**Components**:
- URL frontier with priority queues
- Politeness policy and rate limiting
- Content parsing and extraction
- Duplicate detection with bloom filters

### Design Notification System

**Problem**: Send notifications across multiple channels.

**System Design**:
```
Trigger → Notification Service → Channel Services → Users
             ↓                      ↓
        Template Engine         [Email][SMS][Push]
```

**Features**:
- Multi-channel delivery (email, SMS, push)
- Template management
- Delivery tracking and analytics
- Rate limiting and retry logic

## Level-2 Problems (7 topics)

### Design News Feed System

**Problem**: Build a social media feed like Facebook.

**Requirements**:
- 300M daily active users
- Sub-200ms feed generation
- Support multimedia content
- Real-time updates

**Architecture**:
```
Clients → CDN → API Gateway → Feed Service
                                ↓
                    [User Service][Post Service][Media Service]
                                ↓
                        [Feed Cache][Message Queue]
```

**Feed Generation Strategies**:
- **Push Model**: Pre-compute feeds (fast read, high storage)
- **Pull Model**: Generate on-demand (slow read, low storage)  
- **Hybrid**: Push for normal users, pull for celebrities

### Design Chat System

**Problem**: Build real-time messaging like WhatsApp.

**Architecture**:
```
Clients → WebSocket Gateway → Message Service → Storage
             ↓                      ↓
    Persistent Connections    [Message DB][Media Storage]
```

**Key Features**:
- WebSocket for real-time communication
- Message ordering and delivery guarantees
- Online presence tracking
- Media file handling

### Design Search Autocomplete

**Problem**: Provide search suggestions as users type.

**System Components**:
```
User Input → Autocomplete Service → Trie Data Structure
                ↓                        ↓
        Analytics Service           Suggestion Cache
```

**Implementation**:
- Trie data structure for prefix matching
- Popularity-based ranking
- Real-time updates from search analytics
- Caching for performance

### Design YouTube

**Problem**: Build a video sharing platform.

**Architecture**:
```
Upload: Client → Upload Service → Video Processing → CDN
View: Client → Metadata Service → Streaming Service → CDN
```

**Key Services**:
- Video upload and processing pipeline
- Multiple quality transcoding
- Global content delivery network
- Recommendation system

### Design Google Drive

**Problem**: Build a cloud file storage system.

**System Design**:
```
Clients → API Gateway → File Service → Storage Service
             ↓              ↓            ↓
        Sync Service   Metadata DB   Block Storage
```

**Features**:
- File synchronization across devices
- Version control and conflict resolution
- Sharing and collaboration
- Efficient storage with deduplication

### Design Book My Show

**Problem**: Build a movie ticket booking system.

**Architecture**:
```
Users → Load Balancer → Booking Service → Payment Service
           ↓                ↓                ↓
    Inventory Service   Seat Locking    Transaction DB
```

**Key Challenges**:
- Concurrent seat booking
- Inventory management
- Payment processing
- Seat locking mechanism

### Design Spotify

**Problem**: Build a music streaming service.

**System Components**:
```
Clients → CDN → Streaming Service → Content Service
             ↓         ↓               ↓
        Audio Files  Recommendation  Metadata DB
```

**Features**:
- Audio streaming with adaptive bitrate
- Music recommendation engine
- Playlist management
- Offline download capability

## Level-3 Problems (5 topics)

### Design Uber

**Problem**: Build a ride-hailing service.

**Core Services**:
```
Mobile Apps → API Gateway → [Location][Matching][Trip][Payment]
                               ↓         ↓        ↓       ↓
                          Geo Database  ML Engine  Trip DB  Payment DB
```

**Key Algorithms**:
- Real-time location tracking with geospatial indexing
- Driver-rider matching optimization
- Dynamic pricing based on supply/demand
- Route optimization and ETA calculation

### Design Hotstar (OTT Platform)

**Problem**: Build a video streaming platform for live and on-demand content.

**Architecture**:
```
CDN → Load Balancer → Streaming Service → Content Management
  ↓         ↓              ↓                    ↓
Video Cache  User Service  Video Processing   Content DB
```

**Challenges**:
- Handle traffic spikes during live events
- Adaptive bitrate streaming
- Content delivery optimization
- Digital rights management

### Design Top K YouTube Videos

**Problem**: Find top K most viewed videos in real-time.

**Solution Approach**:
```
Video Views → Stream Processing → Top-K Algorithm → Results
                ↓                      ↓
        Message Queue            Heavy Hitters
```

**Implementation**:
- Stream processing with Apache Kafka
- Count-Min Sketch for frequency estimation
- Heavy Hitters algorithm for top-K
- Real-time updates with minimal latency

### Design Ad Click Aggregator

**Problem**: Aggregate ad click data for real-time analytics.

**System Design**:
```
Ad Clicks → Message Queue → Stream Processor → Aggregation DB
              ↓                 ↓                 ↓
        Partitioning      Time Windows      Analytics API
```

**Features**:
- High-throughput click ingestion
- Real-time aggregation by dimensions
- Time-windowed analytics
- Fraud detection and filtering

### Design Redis (Distributed Cache)

**Problem**: Build an in-memory distributed cache.

**Architecture**:
```
Clients → Load Balancer → Redis Cluster
                            ↓
                [Master1][Master2][Master3]
                    ↓        ↓        ↓
                [Slave1] [Slave2] [Slave3]
```

**Key Features**:
- In-memory data structures
- Master-slave replication
- Consistent hashing for sharding
- Persistence with RDB and AOF

## Interview Strategy

### Structured Approach (45 minutes)

1. **Requirements Clarification** (5-10 min)
   - Functional and non-functional requirements
   - Scale estimation (users, data, QPS)
   - Success metrics and constraints

2. **High-Level Design** (15-20 min)
   - Start simple, add complexity gradually
   - Major components and data flow
   - API design

3. **Detailed Design** (15-20 min)
   - Deep dive into core components
   - Database schema and data models
   - Algorithms and data structures

4. **Scale and Optimize** (10-15 min)
   - Identify bottlenecks
   - Scaling strategies
   - Monitoring and reliability

### Best Practices

**Do's**:
✅ Ask clarifying questions
✅ Start with simple design
✅ Explain your reasoning
✅ Consider trade-offs
✅ Think about failure scenarios

**Don'ts**:
❌ Jump to implementation details
❌ Ignore scalability requirements  
❌ Over-engineer from the start
❌ Forget about monitoring
❌ Ignore the interviewer's feedback

## Conclusion

Success in HLD interviews requires:

1. **Systematic Approach**: Follow structured methodology
2. **Fundamental Knowledge**: Understand core distributed systems concepts
3. **Trade-off Analysis**: Evaluate different design options
4. **Communication**: Clearly explain your thought process
5. **Practice**: Regular practice with different problem types

Remember: Focus on demonstrating your problem-solving approach rather than memorizing solutions. Each problem can have multiple valid solutions depending on the specific requirements and constraints.

The key is to show you can design systems that are scalable, reliable, and meet the business requirements while making informed trade-offs.