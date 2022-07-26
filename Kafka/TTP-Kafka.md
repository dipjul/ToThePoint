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

## Benefits of using clusters
- Kafka cluster is basically a group of multiple brokers
- They are used to maintain load balance
- Kafka brokers are stateless, they rely on Zookeeper to keep track of their cluster state

## Partitioning key
![image](https://user-images.githubusercontent.com/20329508/180903594-dbcaf7eb-b039-42e4-bcb2-b938bcb75053.png)
- messages are referred to as records. Each record has a key and a value, with the key being optional. 
- Partitioning is done using the record's key. By default, Kafka producer uses the record's key to determine which partition the record should be written to. 
- The producer will always choose the same partition for two records with the same key.
- Partitions allow a single topic to be partitioned across numerous servers from the perspective of the Kafka broker
-  A partition is a unit of parallelism from the consumer's perspective.

## Multi-tenancy
![image](https://user-images.githubusercontent.com/20329508/180904143-21e10f1a-8cd8-4442-9997-d4da7f60c7f5.png)
- Multi-tenancy is a software operation mode in which many instances of one or more programs operate in a shared environment independently of one another
- Kafka is multi-tenant because it allows for the configuration of many topics for data consumption and production on the same cluster

## Replication tool
- The Kafka Replication Tool is used to create a high-level design for the replica maintenance process
- Preferred Replica Leader Election Tool:
- Topics tool:
- Tool to reassign partitions:
- StateChangeLogMerger tool:
- Change topic configuration tool:

## Parameters fo optimal performance
![image](https://user-images.githubusercontent.com/20329508/180906503-2900d3dd-287a-4275-968b-84297388feca.png)
- Kafka producer tuning:
  - Data that producers must provide to brokers is kept in a batch
  -  The linger duration is included to create a delay while more records are added to the batch, allowing for larger records to be transmitted.
  -  More messages can be transmitted in one batch with a longer linger period, but latency may suffer as a result. 
  -  A shorter linger time, on the other hand, will result in fewer messages being transmitted faster, resulting in lower latency but also lower throughput.

- Tuning the Kafka broker:
  -  Each partition in a topic has a leader, and each leader has 0 or more followers. It's critical that the leaders are appropriately balanced, and that some nodes aren't overworked in comparison to others
- Tuning Kafka Consumers: 
  -  To ensure that consumers keep up with producers, the number of partitions for a topic should be equal to the number of consumers

## Security
- Encryption:
  - All communications sent between the Kafka broker and its many clients are encrypted.
- Authentication:
  - Before being able to connect to Kafka, apps that use the Kafka broker must be authenticated
- Authorization:
  -  The permission ensures that write access to apps can be restricted to prevent data contamination

## Kafka MirrorMaker
![image](https://user-images.githubusercontent.com/20329508/180907813-02da0fe0-1632-4f3d-9303-b06a22baa05f.png)

- The MirrorMaker is a standalone utility for copying data from one Apache Kafka cluster to another

## Message Compression
![image](https://user-images.githubusercontent.com/20329508/180908122-ba1f1066-09eb-41b6-bdd0-c2302192f59a.png)
- Producers transmit data to brokers in JSON format in Kafka. The JSON format stores data in string form, which can result in several duplicate records being stored in the Kafka topic.
- Compression or delaying of data is performed to save disk space. 
- Because message compression is performed on the producer side, no changes to the consumer or broker setup are required.
 ### Advantages:
  - decreases the latency of messages
  - Producers can send more net messages with less bandwidth
  - When data is saved in Kafka using cloud platforms, saves money
  - reduces the amount of data stored on disk, allowing for faster read and write operations

  ### Disadvantages:
  - Producers must use some CPU cycles to compress
  - CPU cycles for decompression

## Use cases of Kafka not suitable
- Messaging System would be better  if only a small number of messages need to be processed every day
- For ETL (extract, transform, load) jobs, Kafka should be avoided
- when a simple task queue is required(RabbitMQ)
- If long-term storage is necessary

## Log compaction
![image](https://user-images.githubusercontent.com/20329508/180910360-d119377f-98ca-4520-93dd-655922485668.png)
- is a way through which Kafka assures that for each topic partition, at least the last known value for each message key within the log of data is kept

## Quotas
![image](https://user-images.githubusercontent.com/20329508/180910560-a7349a59-0ada-483f-bb62-5f71bae9e53b.png)
- Quotas prevent a single application from monopolizing broker resources and causing network saturation by consuming extremely large amounts of data.

## Unbalanced cluster
- If they have the following peoblems:
  - Leader Skew:
  ![image](https://user-images.githubusercontent.com/20329508/180910872-198563b5-d680-476c-9b5c-ce2dd8ea0f20.png)
  ![image](https://user-images.githubusercontent.com/20329508/180911067-4b493a00-e585-46ab-aa7c-f1d4bc978146.png)
  ![image](https://user-images.githubusercontent.com/20329508/180911084-fb589e99-1ab0-41a6-a4bf-82773810a701.png)
  ![image](https://user-images.githubusercontent.com/20329508/180911109-75375fb0-7f4e-4bfc-9018-0f7c9a334df7.png)
  - Solving the leader skew problem:
    - The auto.leader.rebalance.enable=true broker option allows the controller node to transfer leadership to the preferred replica leaders, restoring the even distribution
    - When  Kafka-preferred-replica-election.sh is run, the preferred replica is selected for all partitions
  - Broker Skew:
    -  if the number of partitions per broker on a given issue is more than the average, the broker is considered to be skewed




## RabbitMQ vs Kafka

## JMS vs Kafka
