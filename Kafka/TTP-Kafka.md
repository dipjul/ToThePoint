# Kafka
## Features
- high throughput and fault tolerance
- built-in patriation system known as a Topic
- replication feature
- a queue that can handle large amounts of data and move messages from one sender to another
- save the messages to storage and replicate them across the cluster
- uses Zookeeper to coordination and synchronization with other services

## Message Queuing
- point-to-point technique, message queuing pattern
- A message in the queue will be destroyed once it has been consumed
- Asynchronous messaging is possible
- if a consumer is unavailable, the message will be held in the queue until it can be sent
- messages aren't always sent in the same order, they are given on a first-come, first-served basis

## Publisher - Subscriber Model
- publishers producing ("publishing") messages in multiple categories and subscribers consuming published messages from the various categories to which they are subscribed
- Scalable : A cluster of devices is used to partition and streamline the data
- Faster : Thousands of clients can be served by a single Kafka broker
- Durability and Fault-Tolerant : By copying the data in the clusters

## Components
![image](https://user-images.githubusercontent.com/20329508/180700354-1fe20dc1-fb7a-43d6-9792-007ae8a0c063.png)

### Topic
- a category or feed in which records are saved and published, consumer reads and producer writes data to them
- records remain the cluster for configurable retention period
- Kafka keeps records in logs, cosumers have to keep track of the offset(where in the log)
- Consumer can consume messages in linear or any other order

### Producer
-  a data source for one or more Kafka topics that optimizes, writes, and publishes messages
-  Partitioning allows Kafka producers to serialize, compress, and load balance data among brokers

### Consumer
- Data is read by reading messages from topics to which they have subscribed. 
- Consumers will be divided into groups. 
- Each consumer in a consumer group will be responsible for reading a subset of the partitions of each subject to which they have subscribed

### Brokers
- a server that works as part of a Kafka cluster
- Multiple brokers typically work together to build a Kafka cluster, which provides load balancing, reliable redundancy, and failover
- The cluster is managed and coordinated by brokers using Apache ZooKeeper
- Zookeeper used for leader elections, a broker is chosen to lead the handling of client requests for a certain partition of a topic

## Four core APIs
### Producer API
- allows an application to publish a stream of records to one or more Kafka topics
### Consumer API
- allows an application ton subscribe to one or more Kafka topics
### Streams API
- take input streams from one or more topics, process them using streams operations, and generate output streams to transmit to one or more topics
- allows you to convert input streams into output streams
### Connect API
- connects Kafka topics to applications
- opens up possibilities for constructing and managing the operations of producers and consumers

## Partition
- topics are separated into partitions, each of which contains records in a fixed order
- a unique offset is assigned and attributed to each record in a partition
- multiple partition logs can be found in a single topic, allowing several users to read from the same topic at the same time
- replication in Kafka is done at the partition level
- One server serves as the leader of each partition (replica), while the others function as followers. 
- The leader replica is in charge of all read-write requests for the partition, while the followers replicate the leader. 
- If the lead server goes down, one of the followers takes over as the leader.

## Zookeeper
- a naming registry for distributed applications 
- used by Kafka brokers to maintain and coordinate the Kafka cluster
- when brokers and topics are added or removed, ZooKeeper notifies all nodes
- allows brokers and topic partition pairs to elect leaders, allowing them to select which broker will be the leader for a given partition
- Note: Kafka 2.8.0 onwards can be used without Zookeeper

## Leader and Follower in Kafka
-  each partition has one server that acts as a Leader and one or more servers that operate as Followers.
-  The Leader is in charge of all read and writes requests for the partition, while the Followers are responsible for passively replicating the leader
![image](https://user-images.githubusercontent.com/20329508/180707534-5c6b8656-a958-4ed3-9bcc-c21b2570aa33.png)

## Topic Replication
- When one broker fails, topic replicas on other brokers remain available to ensure that data is not lost and that the Kafka deployment is not disrupted. 
- The replication factor specifies the number of copies of a topic that are kept across the Kafka cluster
- An In-Sync Replica (ISR) is a replica that is up to date with the partition's leader

## Consumer group
![image](https://user-images.githubusercontent.com/20329508/180708678-aecae202-f095-4344-9631-82f06873d9f2.png)

-  a collection of consumers who work together to ingest data from the same topic or range of topics
-  The ‘-group' command must be used to consume messages from a consumer group

## Max size of message
- By default, the maximum size of a Kafka message is 1MB (megabyte). 
- The broker settings allow you to modify the size. 
- Kafka, on the other hand, is designed to handle 1KB messages as well.

## Not an In-Sync Replica
- A replica that has been out of ISR for a long period of time indicates that the follower is unable to fetch data at the same rate as the leader

## Start a Kafka server
- Start the ZooKeeper service by doing the following:  
`$bin/zookeeper-server-start.sh config/zookeeper.properties`
- To start the Kafka broker service, open a new terminal and type the following commands:
`$bin/kafka-server-start.sh config/server.properties`

## Geo-replication
- a feature that allows messages in one cluster to be copied across many data centers or cloud regions
- accomplished with Kafka's MirrorMaker Tool

## Disadvantages of Kafka
- a performance degrades if there is message tweaking
- Wildcard topic selection is not supported
- Brokers and consumers reduces performance when dealing with huge messages by compressing and decompressing the messages
- Certain message paradigms, including point-to-point queues and request/reply, are not supported
-  does not have a complete set of monitoring tools

## Real world usages
- Messaging
- To Monitor operational data
- Website activity tracking
- Data logging
- Stream Processing

## Use cases of Kafka monitoring
- Track System Resource Consumption
- Monitor threads and JVM usage

## Schema registry
![image](https://user-images.githubusercontent.com/20329508/180715660-b46046a6-8a33-4ca9-b9ad-3ac09fe3005c.png)

- A Schema Registry is present for both producers and consumers, and it holds Avro schemas.
- is used to ensure that the schema used by the consumer and the schema used by the producer are identical
- The producers just need to submit the schema ID and consumer looks up the matching schema in the Schema Registry using the schema ID

