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


