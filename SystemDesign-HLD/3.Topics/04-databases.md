# Databases

## Overview

Databases are the foundation of most software systems, responsible for storing, organizing, and retrieving data efficiently and reliably. Understanding database concepts, types, optimization techniques, and scaling strategies is crucial for designing robust systems that can handle growing data volumes and user loads while maintaining performance and consistency.

## What are Databases?

A database is an organized collection of structured information, or data, typically stored electronically in a computer system. Databases are managed by Database Management Systems (DBMS), which provide interfaces for users and applications to interact with the data.

### Core Functions of Databases:

#### 1. Data Storage
- **Persistent Storage**: Data survives system restarts and failures
- **Structured Organization**: Data is organized in tables, documents, or other formats
- **Efficient Storage**: Optimized for space utilization and access patterns
- **Data Integrity**: Ensures data accuracy and consistency

#### 2. Data Retrieval
- **Query Processing**: Execute complex queries to find specific data
- **Indexing**: Fast data lookup using various indexing strategies
- **Filtering and Sorting**: Organize results based on criteria
- **Aggregation**: Perform calculations across data sets

#### 3. Data Management
- **CRUD Operations**: Create, Read, Update, Delete data
- **Transaction Management**: Ensure data consistency during operations
- **Concurrency Control**: Handle multiple simultaneous users
- **Security**: Control access to sensitive data

#### 4. Data Reliability
- **ACID Properties**: Atomicity, Consistency, Isolation, Durability
- **Backup and Recovery**: Protect against data loss
- **Replication**: Maintain multiple copies for availability
- **Fault Tolerance**: Continue operating despite failures

## Storage and Retrieval

Understanding how databases store and retrieve data is fundamental to making informed design decisions.

### Storage Mechanisms

#### 1. Disk-Based Storage
```
Application → Database Engine → File System → Disk
```

**Characteristics:**
- **Persistent**: Data survives power failures
- **Large Capacity**: Can store terabytes of data
- **Slower Access**: Mechanical limitations of spinning disks
- **Cost Effective**: Lower cost per GB

**Optimization Techniques:**
- **Sequential Access**: Minimize disk seeks
- **Block-Based I/O**: Read/write in larger chunks
- **Write-Ahead Logging**: Ensure durability
- **Buffer Pools**: Cache frequently accessed pages

#### 2. Memory-Based Storage
```
Application → Database Engine → RAM
```

**Characteristics:**
- **Fast Access**: Microsecond response times
- **Volatile**: Data lost on power failure (unless persisted)
- **Limited Capacity**: Constrained by available RAM
- **Higher Cost**: More expensive per GB

**Use Cases:**
- **Caching**: Frequently accessed data
- **Session Storage**: Temporary user data
- **Real-time Analytics**: Fast data processing
- **In-memory Databases**: Redis, Memcached

#### 3. Hybrid Storage
```
Hot Data (RAM) ← → Warm Data (SSD) ← → Cold Data (HDD)
```

**Tiered Storage Strategy:**
- **Hot Data**: Frequently accessed, stored in memory
- **Warm Data**: Occasionally accessed, stored on SSD
- **Cold Data**: Rarely accessed, stored on HDD
- **Automatic Tiering**: Move data between tiers based on access patterns

### Retrieval Mechanisms

#### 1. Full Table Scans
```
Query → Scan entire table → Filter results → Return matches
```

**When Used:**
- Small tables
- Queries without indexes
- Analytics queries processing large datasets

**Performance:**
- O(n) time complexity
- High I/O cost for large tables
- Can utilize parallel processing

#### 2. Index-Based Retrieval
```
Query → Check index → Locate data pages → Return results
```

**Types of Indexes:**
- **B-Tree**: Balanced tree structure for range queries
- **Hash**: Fast equality lookups
- **Bitmap**: Efficient for low-cardinality data
- **Full-text**: Search within text content

**Performance:**
- O(log n) for B-tree indexes
- O(1) for hash indexes
- Faster queries but slower writes

#### 3. Query Optimization
```
SQL Query → Query Parser → Query Optimizer → Execution Plan → Results
```

**Optimization Techniques:**
- **Cost-Based Optimization**: Choose lowest-cost execution plan
- **Statistics**: Maintain data distribution statistics
- **Join Optimization**: Efficient join algorithms
- **Predicate Pushdown**: Filter data early in the process

## Types of Databases

Different database types are optimized for different use cases and data models.

### SQL Databases (Relational)

SQL databases organize data in tables with predefined schemas and relationships.

#### Structure:
```
Database
├── Table 1 (Users)
│   ├── Column: id (Primary Key)
│   ├── Column: name
│   └── Column: email
├── Table 2 (Orders)
│   ├── Column: id (Primary Key)
│   ├── Column: user_id (Foreign Key)
│   └── Column: amount
└── Relationships (Foreign Keys)
```

#### Key Features:
- **ACID Properties**: Strong consistency guarantees
- **SQL Language**: Standardized query language
- **Schema Enforcement**: Data structure validation
- **Relationships**: Foreign keys and joins
- **Transactions**: Multi-operation consistency

#### Popular SQL Databases:
- **PostgreSQL**: Advanced open-source database
- **MySQL**: Widely-used open-source database
- **Oracle**: Enterprise-grade commercial database
- **SQL Server**: Microsoft's database solution
- **SQLite**: Lightweight embedded database

#### Use Cases:
- **Financial Systems**: Banking, accounting, payments
- **E-commerce**: Order management, inventory
- **CRM Systems**: Customer relationship management
- **ERP Systems**: Enterprise resource planning

### NoSQL Databases

NoSQL databases are designed for specific data models and use cases that don't fit well in relational models.

#### 1. Document Databases
```
Collection: Users
├── Document 1: {
│   "id": "user1",
│   "name": "John Doe",
│   "addresses": [
│     {"type": "home", "city": "NYC"},
│     {"type": "work", "city": "SF"}
│   ]
│ }
└── Document 2: {...}
```

**Characteristics:**
- **Flexible Schema**: Documents can have different structures
- **Nested Data**: Support for complex data types
- **JSON/BSON**: Human-readable format
- **Horizontal Scaling**: Built for distributed systems

**Examples**: MongoDB, CouchDB, Amazon DocumentDB

**Use Cases:**
- Content management systems
- User profiles and preferences
- Product catalogs
- Real-time analytics

#### 2. Key-Value Stores
```
Key: "user:1001" → Value: {"name": "John", "email": "john@example.com"}
Key: "session:abc123" → Value: {"user_id": 1001, "expires": "2024-01-01"}
```

**Characteristics:**
- **Simple Model**: Just key-value pairs
- **High Performance**: Optimized for fast lookups
- **Horizontal Scaling**: Easy to distribute
- **Eventual Consistency**: Often eventually consistent

**Examples**: Redis, Amazon DynamoDB, Riak

**Use Cases:**
- Caching layers
- Session storage
- Shopping carts
- Real-time recommendations

#### 3. Column-Family Databases
```
Row Key: "user1"
├── Column Family: "profile"
│   ├── name: "John Doe"
│   └── email: "john@example.com"
└── Column Family: "activity"
    ├── last_login: "2024-01-01"
    └── login_count: 42
```

**Characteristics:**
- **Column-Oriented**: Data stored by columns
- **Sparse Data**: Efficient for sparse datasets
- **Time-Series Friendly**: Good for time-based data
- **Horizontal Scaling**: Built for big data

**Examples**: Cassandra, HBase, Amazon SimpleDB

**Use Cases:**
- Time-series data
- IoT sensor data
- Log aggregation
- Analytics platforms

#### 4. Graph Databases
```
Nodes: [User1] [User2] [Product1]
Edges: User1 -[FRIENDS]-> User2
       User1 -[BOUGHT]-> Product1
       User2 -[LIKES]-> Product1
```

**Characteristics:**
- **Graph Model**: Nodes and relationships
- **Traversal Queries**: Follow relationships efficiently
- **ACID Properties**: Often support transactions
- **Complex Relationships**: Natural for connected data

**Examples**: Neo4j, Amazon Neptune, ArangoDB

**Use Cases:**
- Social networks
- Recommendation engines
- Fraud detection
- Knowledge graphs

### SQL vs NoSQL Comparison

| Aspect | SQL | NoSQL |
|--------|-----|-------|
| **Schema** | Fixed, predefined | Flexible, dynamic |
| **Scalability** | Vertical (scale up) | Horizontal (scale out) |
| **Consistency** | Strong (ACID) | Eventual (BASE) |
| **Query Language** | SQL (standardized) | Varies by database |
| **Transactions** | Full ACID support | Limited or eventual |
| **Relationships** | Foreign keys, joins | Denormalized, embedded |
| **Use Cases** | Complex queries, reporting | Simple queries, scalability |
| **Learning Curve** | Well-known | Varies by type |

### Other Types of Databases

#### 1. Time-Series Databases
**Optimized for**: Time-stamped data
**Examples**: InfluxDB, TimescaleDB, OpenTSDB
**Use Cases**: Monitoring, IoT, financial data

#### 2. Search Databases
**Optimized for**: Full-text search and analytics
**Examples**: Elasticsearch, Solr, Amazon CloudSearch
**Use Cases**: Search engines, log analysis, business intelligence

#### 3. Multi-Model Databases
**Supports**: Multiple data models in one system
**Examples**: ArangoDB, CosmosDB, OrientDB
**Use Cases**: Applications with diverse data requirements

### Database Selection Based on Use Case

#### Decision Framework:

```
Requirements Analysis
├── Data Structure
│   ├── Structured → SQL
│   ├── Semi-structured → Document
│   └── Unstructured → Key-Value
├── Scalability Needs
│   ├── Vertical → SQL
│   └── Horizontal → NoSQL
├── Consistency Requirements
│   ├── Strong → SQL
│   └── Eventual → NoSQL
├── Query Complexity
│   ├── Complex → SQL
│   └── Simple → NoSQL
└── Team Expertise
    ├── SQL Knowledge → SQL
    └── NoSQL Experience → NoSQL
```

#### Use Case Examples:

**E-commerce Platform:**
- **User Profiles**: Document database (MongoDB)
- **Product Catalog**: Search database (Elasticsearch)
- **Order Management**: SQL database (PostgreSQL)
- **Shopping Cart**: Key-value store (Redis)
- **Recommendations**: Graph database (Neo4j)

**Social Media Platform:**
- **User Posts**: Document database (MongoDB)
- **Social Graph**: Graph database (Neo4j)
- **Real-time Feed**: Key-value store (Redis)
- **Analytics**: Column-family (Cassandra)
- **Search**: Search database (Elasticsearch)

## Database Optimizations

Optimizing database performance involves multiple strategies and techniques.

### Indexing

Indexes are data structures that improve query performance by providing fast access paths to data.

#### Types of Indexes:

#### 1. B-Tree Indexes
```
        [50]
       /    \
   [25]      [75]
   /  \      /  \
[10][40] [60][90]
```

**Characteristics:**
- **Balanced Tree**: All leaf nodes at same level
- **Range Queries**: Efficient for range scans
- **Ordered Data**: Maintains sort order
- **Most Common**: Default index type in most databases

**Use Cases:**
- Primary keys
- Range queries (>, <, BETWEEN)
- ORDER BY clauses
- Equality searches

#### 2. Hash Indexes
```
Hash Function: hash(key) → bucket_location
Key: "john@example.com" → Hash: 12345 → Bucket: 5
```

**Characteristics:**
- **Fast Equality**: O(1) lookup time
- **No Range Queries**: Cannot handle range scans
- **Memory Efficient**: Compact storage
- **Hash Collisions**: Must handle collisions

**Use Cases:**
- Equality searches (=)
- Primary key lookups
- Cache implementations
- Unique constraints

#### 3. Bitmap Indexes
```
Gender Column:
Male:   [1,0,1,0,1,0,1,0]
Female: [0,1,0,1,0,1,0,1]
```

**Characteristics:**
- **Low Cardinality**: Efficient for few distinct values
- **Bitwise Operations**: Fast AND, OR, NOT operations
- **Space Efficient**: Compact for sparse data
- **Read Optimized**: Better for read-heavy workloads

**Use Cases:**
- Data warehousing
- Analytics queries
- Boolean columns
- Categorical data

#### Index Design Best Practices:

**Do's:**
✅ **Index Frequently Queried Columns**: WHERE, JOIN, ORDER BY
✅ **Composite Indexes**: Multiple columns in single index
✅ **Covering Indexes**: Include all needed columns
✅ **Monitor Index Usage**: Remove unused indexes

**Don'ts:**
❌ **Over-Indexing**: Too many indexes slow down writes
❌ **Duplicate Indexes**: Redundant indexes waste space
❌ **Wide Indexes**: Too many columns in composite index
❌ **Ignore Maintenance**: Rebuild fragmented indexes

### Partitioning

Partitioning divides large tables into smaller, more manageable pieces.

#### Types of Partitioning:

#### 1. Horizontal Partitioning (Sharding)
```
Original Table: Users (1M rows)
├── Partition 1: Users 1-250K
├── Partition 2: Users 250K-500K
├── Partition 3: Users 500K-750K
└── Partition 4: Users 750K-1M
```

**Strategies:**
- **Range Partitioning**: Based on value ranges
- **Hash Partitioning**: Based on hash function
- **List Partitioning**: Based on predefined lists
- **Composite Partitioning**: Combination of strategies

**Benefits:**
- **Improved Performance**: Smaller tables to scan
- **Parallel Processing**: Query multiple partitions simultaneously
- **Maintenance**: Easier backup and maintenance
- **Scalability**: Distribute across multiple servers

#### 2. Vertical Partitioning
```
Original Table: Users (id, name, email, bio, preferences, settings)
├── Core Table: Users (id, name, email)
└── Extended Table: UserDetails (id, bio, preferences, settings)
```

**Benefits:**
- **Reduced I/O**: Fetch only needed columns
- **Cache Efficiency**: Better memory utilization
- **Security**: Separate sensitive data
- **Performance**: Faster queries on smaller tables

### Sharding and Its Types

Sharding distributes data across multiple database instances.

#### Sharding Strategies:

#### 1. Range-Based Sharding
```
Shard 1: Users A-F
Shard 2: Users G-M
Shard 3: Users N-S
Shard 4: Users T-Z
```

**Pros:**
✅ **Simple Logic**: Easy to understand and implement
✅ **Range Queries**: Efficient for range scans
✅ **Ordered Data**: Maintains sort order

**Cons:**
❌ **Hot Spots**: Uneven data distribution
❌ **Rebalancing**: Difficult to redistribute
❌ **Predictable**: Attackers can predict shard location

#### 2. Hash-Based Sharding
```
Shard = hash(user_id) % number_of_shards
User 1001 → hash(1001) % 4 = Shard 1
User 1002 → hash(1002) % 4 = Shard 2
```

**Pros:**
✅ **Even Distribution**: Better load balancing
✅ **Unpredictable**: Harder to predict shard location
✅ **Simple**: Easy to implement

**Cons:**
❌ **No Range Queries**: Cannot efficiently query ranges
❌ **Resharding**: Adding shards requires redistribution
❌ **Cross-Shard Queries**: Complex multi-shard operations

#### 3. Directory-Based Sharding
```
Lookup Service:
User 1001 → Shard A
User 1002 → Shard B
User 1003 → Shard A
```

**Pros:**
✅ **Flexibility**: Can use any sharding logic
✅ **Dynamic**: Easy to move data between shards
✅ **Complex Logic**: Support for sophisticated routing

**Cons:**
❌ **Single Point of Failure**: Lookup service dependency
❌ **Latency**: Additional lookup overhead
❌ **Complexity**: More complex to implement

#### Sharding Challenges:

**1. Cross-Shard Queries:**
```
Problem: SELECT * FROM users WHERE age > 25
Solution: Query all shards and merge results
```

**2. Distributed Transactions:**
```
Problem: Transfer money between users on different shards
Solution: Two-phase commit or saga pattern
```

**3. Rebalancing:**
```
Problem: Adding new shards requires data redistribution
Solution: Consistent hashing or virtual shards
```

## Database Replication and Migration

Replication and migration strategies ensure high availability and enable system evolution.

### Database Replica Architectures

#### 1. Master-Slave Replication
```
Master DB (Read/Write)
    ↓ (Async Replication)
Slave DB 1 (Read Only)
Slave DB 2 (Read Only)
```

**Characteristics:**
- **Single Write Node**: All writes go to master
- **Multiple Read Nodes**: Distribute read load
- **Asynchronous**: Slaves may lag behind master
- **Failover**: Manual or automatic promotion of slave

**Use Cases:**
- Read-heavy workloads
- Reporting and analytics
- Geographic distribution
- Backup and disaster recovery

#### 2. Master-Master Replication
```
Master DB 1 (Read/Write) ←→ Master DB 2 (Read/Write)
```

**Characteristics:**
- **Multiple Write Nodes**: Both nodes accept writes
- **Bidirectional**: Changes replicate in both directions
- **Conflict Resolution**: Handle conflicting updates
- **Active-Active**: Both nodes serve traffic

**Challenges:**
- **Write Conflicts**: Same data modified simultaneously
- **Consistency**: Maintaining data consistency
- **Complexity**: More complex to manage

#### 3. Multi-Master Replication
```
Master 1 ←→ Master 2
    ↕         ↕
Master 3 ←→ Master 4
```

**Use Cases:**
- Global applications
- High availability requirements
- Distributed teams
- Regional data centers

### WAL and Change Data Capture

#### Write-Ahead Logging (WAL)
```
Transaction → Write to WAL → Write to Data Files → Commit
```

**Purpose:**
- **Durability**: Ensure changes survive crashes
- **Recovery**: Replay log to restore state
- **Replication**: Stream changes to replicas
- **Point-in-Time Recovery**: Restore to specific time

**WAL Process:**
1. **Log Changes**: Write changes to log before data files
2. **Flush to Disk**: Ensure log is persisted
3. **Apply Changes**: Update actual data files
4. **Checkpoint**: Mark point where all changes are applied

#### Change Data Capture (CDC)
```
Database → CDC Tool → Event Stream → Downstream Systems
```

**Benefits:**
- **Real-time**: Near real-time change propagation
- **Non-intrusive**: Minimal impact on source database
- **Reliable**: Guaranteed delivery of changes
- **Scalable**: Handle high-volume change streams

**Use Cases:**
- **Data Synchronization**: Keep systems in sync
- **Event Sourcing**: Build event-driven architectures
- **Analytics**: Real-time data warehousing
- **Search Indexing**: Update search indexes

### Write Amplification and Split Brain

#### Write Amplification
```
Single Logical Write → Multiple Physical Writes
User Update → WAL Write → Data File Write → Index Update → Replica Sync
```

**Causes:**
- **Logging**: Write-ahead logs
- **Indexing**: Multiple index updates
- **Replication**: Sync to replicas
- **Compaction**: Background maintenance

**Mitigation:**
- **Batch Writes**: Group multiple operations
- **Efficient Indexes**: Minimize index overhead
- **Async Replication**: Reduce synchronous writes
- **SSD Optimization**: Use SSD-friendly patterns

#### Split Brain
```
Network Partition:
Master 1 (thinks it's primary) ←X→ Master 2 (thinks it's primary)
```

**Problem:**
- **Dual Masters**: Both nodes accept writes
- **Data Divergence**: Inconsistent data states
- **Conflict Resolution**: Difficult to merge changes

**Solutions:**
- **Quorum**: Require majority consensus
- **Fencing**: Disable minority partition
- **Witness Node**: Third node for tie-breaking
- **Application Logic**: Handle conflicts gracefully

### Database Replication Types

#### 1. Synchronous Replication
```
Write Request → Master → Replica → Acknowledge → Client
```

**Characteristics:**
- **Strong Consistency**: All replicas have same data
- **Higher Latency**: Wait for replica acknowledgment
- **Durability**: Data guaranteed on multiple nodes
- **Availability Impact**: Failure affects writes

#### 2. Asynchronous Replication
```
Write Request → Master → Acknowledge → Client
                  ↓ (Background)
               Replica
```

**Characteristics:**
- **Lower Latency**: Don't wait for replicas
- **Eventual Consistency**: Replicas may lag
- **Higher Availability**: Replica failures don't affect writes
- **Data Loss Risk**: Potential data loss on master failure

#### 3. Semi-Synchronous Replication
```
Write Request → Master → At least 1 Replica → Acknowledge → Client
```

**Characteristics:**
- **Balanced**: Between sync and async
- **Configurable**: Choose number of replicas to wait for
- **Reduced Risk**: Lower data loss risk than async
- **Better Performance**: Better than full sync

### Database Migrations

#### Types of Migrations:

#### 1. Schema Migrations
```
Version 1: users (id, name, email)
Version 2: users (id, first_name, last_name, email)
```

**Strategies:**
- **Blue-Green**: Switch between environments
- **Rolling**: Gradual migration
- **Backward Compatible**: Support both schemas temporarily

#### 2. Data Migrations
```
Old Format: {"name": "John Doe"}
New Format: {"first_name": "John", "last_name": "Doe"}
```

**Approaches:**
- **Batch Processing**: Migrate data in batches
- **Streaming**: Real-time migration
- **Dual Write**: Write to both old and new formats

#### 3. Migration Across Regions
```
Region A (Primary) → Region B (New Primary)
```

**Challenges:**
- **Network Latency**: Cross-region delays
- **Data Consistency**: Maintain consistency during migration
- **Downtime**: Minimize service interruption
- **Rollback**: Plan for migration failures

**Best Practices:**
- **Test Thoroughly**: Test migration process
- **Monitor Closely**: Watch for issues during migration
- **Plan Rollback**: Have rollback strategy ready
- **Gradual Migration**: Migrate in phases

## Conclusion

Databases are complex systems that require careful consideration of many factors:

1. **Choose the Right Type**: SQL vs NoSQL based on requirements
2. **Optimize Performance**: Use indexing, partitioning, and sharding appropriately
3. **Plan for Scale**: Design replication and sharding strategies
4. **Ensure Reliability**: Implement proper backup and recovery procedures
5. **Monitor and Maintain**: Continuously optimize and maintain database health

Understanding these concepts enables you to design data storage solutions that meet your application's requirements for performance, scalability, consistency, and availability.

## Next Steps

In the following sections, we'll explore:
- **API Design**: Building interfaces for data access
- **Networking**: Communication patterns in distributed systems
- **Caching**: Strategies for improving database performance
- **Security**: Protecting data and access controls
- **Monitoring**: Observability for database systems