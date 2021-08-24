---
title: 'Distributed Messaging System: Kafka'
date: 2021-08-24 10:07:33
categories:
tags:
top:
---
# 1. Overview of Messaging Systems

## 1.1 Why we need a messaging system

- Aim:
    - Reliably transfer a high throughput of messages between different entities
- Challenges
    - how we handle a spike of messages
    - how we divide the work among a set of instances
    - how could we receive messages from different types of sources
    - what will happen if the service is down?

- We need messaging systems in distributed architecture due to challenges above

## 1.2 What is a messaging system?

- responsible for transferring data among services /applications/ processes/ servers
- help decouple different parts of a distributed system by providing an asynchronous way of transferring messaging between the sender and the receiver

- Two common ways to handle messages
    - Queuing
        - msgs are stored sequentially in a queue
        - producers push msg to the rear of the queue
        - consumers extract the msgs from the front of the queue
        - a particular msg can be consumed by a **max of one consumer** only
    - Publish - Subscribe
        - messages are divided into topics
        - a publisher sends a message to a topic
        - subscribers subscribe to a topic to receive every message published to that topic
        - msg system that stores and maintains the msg named as **message broker**

# 2. Kafka

## 2.1 General

- **publish subscribe based** messaging system
- takes streams of messages from applications known as producers, stores them reliably on a central cluster, and allows those messages to be received by applications that process the messages
- kafka is mainly used for
    - reliably storing a huge amount of data
    - enabling high throughput of message transfer between different entities
    - streaming real time data
- kafka is a distributed commit log — write ahead log
    - append-only data structure that can **persistently store a sequence of records**
    - all messages are stored **on disk**
    - since all reads and writes happen **in sequence**, Kafka takes advantage of **sequential disk reads**

## 2.2 Use Cases

- Metrics
    - collect and aggregate monitoring data
- Log Aggregation
    - collect logs from multiple sources and make them available in a standard format to multiple consumers
- Stream Processing
    - the raw data consumed from a topic is transformed, enriched, or aggregated and pushed to a **new topic** for further consumption. This way of data processing is known as stream processing.
- Commit Log
    - can be used as an external commit log for any distributed system
    - Distributed services can log their transactions to Kafka to keep track of what is happening. This transaction data can be used for replication between nodes and also becomes very useful for disaster recovery, for example, to help failed nodes to recover their states.
- Website activity tracking
    - Build a user activity tracking pipeline
    - User activities like page clicks, searches, etc., are published to Kafka into separate topics. These topics are available for subscription for a range of use cases, including real-time processing, real-time monitoring, or loading into Hadoop or data warehousing systems for offline processing and reporting
- Product Suggestion

# 3. High Level Architecture

## 3.1 Common Terms

- Brokers
    - A Kafka server
    - responsible for reliably storing data provided by the producers and making it available to the consumers
- Records
    - A message or an event that get stored in Kafka
    - A record contains
        - key
        - value
        - timestamp
        - optional metadata headers
- Topics
    - Messages are divided into categories called topics
    - Each msg that Kafka receives from a producer is associated with a topic
    - consumers can subscribe to a topic to get notified when new messages are added to the topic
    - a topic can have multiple subscribers that read messages from it
    - a topic is identified by its name and must be unique
    - mes in a topic can be read as often as needed — message are not deleted after consumption, instead, Kafka **retains messages for a configurable amount of time or until a storage size is exceeded**
- Producers
    - Applications that publish or write records to Kafka
- Consumers
    - Applications that subscribe to read and process data from Kafka topics
    - Consumers subscribe to one or more topics and consume published messages by pulling data from the brokers
    - In Kafka, producers and consumers are fully decoupled and agnostic of each other, which is a key design element to achieve the high scalability that Kafka is known for

## 3.2 Architecture

![Overall Architecture](https://i.loli.net/2021/08/25/Chuwtvg4mNfFU58.png)

- Kafka cluster
    - Kafka is run as a cluster of one or more servers, where each server is responsible for running one Kafka broker
- ZooKeeper
    - Distributed key value store
    - Used for coordination and storing configurations
    - Kafka uses ZooKeeper to coordinate between Kafka brokers; ZooKeeper maintains metadata information about the Kafka cluster

## 3.3 Performance concern

### 3.3.1 Storing messages to disks

- there is a huge difference in disk performance between **random block access and sequential access**. Random block access is slower because of **numerous disk seeks**, whereas the sequential nature of writing or reading, enables disk operations to be **thousands of times faster** than random access.
- OS level optimization
    - Read Ahead — prefetch large block multiples
    - Write Behind — group small logical writes into big physical writes
    - PageCache — cache the disk in free RAM
- Zero Copy optimization
    - OS copy data from the pageCache directly to a socket, effectively bypassing the kafka broker application entirely
- Kafka protocol to group msg together
    - reduce network overhead

# 4. Dive Deep in Kafka Cluster

## 4.1 Topic Partitions

- Topics are partitioned, spread over a number of fragments
- Each partition can be placed on a separate Kafka broker
- A new message get appended to one of the topic's partition
    - producer controls which partition it publishes to based on the data
- One partition is an **ordered sequence** of messages
    - producers continually append new messages to partition
    - ordering of messages is **maintained at the partition level, not across the topic**

- Unique sequence ID — offset
    - It will get assigned to every message that enters a partition
    - used to identify every message's sequential position within a topic's partition
    - offset sequences are unique only to each partition
    - to locate a specific message
        - topic
        - partition
        - offset number
    - producers can choose to publish a message to any partition
        - if ordering within a partition is not needed, a round robin partition strategy can be used
        - Placing each partition on separate Kafka brokers enables multiple consumers to read from a topic in parallel. That means, different consumers can concurrently read different partitions present on separate brokers
- Messages once written to partitions are immutable and cannot be updated.
- Kafka guarantees that messages with the same key are written to the same partition.

## 4.2 Dumb Broker and Smart Consumer

- Kafka does not keep track of what records are read by the consumer
- Consumers themselves poll kafka for new messages and say what records they want to read
    - this allow them to increment/ decrement the offset they are as they wish

## 4.3 Leader and Follower

Every topic can be replicated to multiple Kafka brokers to make the data fault-tolerant and highly available. Each topic partition has one leader broker and multiple replica (follower) brokers. 

- Structure
    - the broker cluster could have multiple brokers, each broker could have multiple partitions which belong to different topics
    - Each topic partition would have one lead broker and multiple replica brokers

### 4.3.1 Leader

- A leader is the node responsible for all reads and writes for the given partition
- Each partition has one kafka broker acting as a leader

### 4.3.2 Follower

- To handle single point of failure, Kafka replicate partitions and distribute them across multiple broker servers called followers.
- Each follower's responsibility is to replicate the leader's data to serve as a backup partition
    - any follower can take over the leadership if the leader goes down
        - from the image below, you could see only the leader take read and write requests, follower acts as replica but not take any read and write reqeusts

        ![Leader and Follower](https://i.loli.net/2021/08/25/4t7kVJfv3jliZyK.png)

### 4.3.3 In Sync Replicas

- In Sync Replicas means the broker has the latest data for a given partition
- A leader is always an in sync replica
- A follower is an in sync replica only if it has fully caught up to the partition it is following
- Only ISRs are eligible to become partition leaders.
- Kafka can choose the minimum number of ISRs required before the data becomes available for consumers to read

### 4.3.4 High Water mark

- To ensure data consistency, the leader broker never returns (or exposes) messages which have not been replicated to a minimum set of ISRs
- For this, brokers keep track of the high-water mark, which is the highest offset that all ISRs of a particular partition share
- The leader exposes data only up to the high-water mark offset and propagates the high-water mark offset to all followers

    ![High Water Mark](https://i.loli.net/2021/08/25/pyDKzGCRow6O2Wu.png)

# 5. Consumer Group

- A set of one or more consumers working together in parallel to consume messages from topic partitions, messages are equally divided among all the consumers of a group. with no two consumers receiving the same message

## 5.1 How to distribute a specific message to only a single consumer

- only a single consumer reads messages from any partition within a consumer group
    - means only one consumer can work on a partition in a consumer group at a time
    - every time a consumer is added to or removed from a group, the consumption is rebalanced within the group
- with consumer groups, consumers can be parallelized so that multiple consumers can read from multiple partitions on a topic, allowing a very high message processing throughput
- number of partitions impacts consumers' maximum parallelism. as there cannot be more consumers than partitions
- Kafka stores the **current offset per consumer group per topic per partition**, as it would for a single consumer. This means that unique messages are only sent to a single consumer in a consumer group, and the load is balanced across consumers as equally as possible

- **Number of consumers in a group = number of partitions:** each consumer consumes one partition.
- **Number of consumers in a group > number of partitions:** some consumers will be idle.
- **Number of consumers in a group < number of partitions:** some consumers will consume more partitions than others.

# 6. Kafka Workflow

## 6.1 Pub sub messaging

- Producer publish messages on a topic
- Kafka broker stores messages in the partitions configured for that particular topic.
    - If the producer did not specify the partition in which the msg should be stored, the broker ensures that the msg are equally shared between partitions
    - If the producer sends two msgs and there are two partitions, Kafka will store those two in two partitions separately.
- consumer subscribe to a specific topic
- Kafka will provide the current offset of the topic to the consumer and also saves that offset in the zookeeper
- consumer request kafka at regular intervals for new msgs
- once kafka receives msg from producers, it forward these messages to the consumer
- consumer will receive msg and process it
- once processed, consumer will send an acknowledgement to the kafka broker
- upon receiving the acknowledgement, kafka **increments the offset and updates it in the zooKeeper**
    - this info is stored in zooKeeper, thus consumer could read the next msg correctly even during broker outages
- consumers can rewind/ skip to the desired offset of a topic at any time and read all the subsequent messages

## 6.2 Kafka workflow for consumer group

- Producers publish messages on a topic.
- Kafka stores all messages in the partitions configured for that particular topic, similar to the earlier scenario.
- A single consumer subscribes to a specific topic, assume `Topic-01` with Group ID as `Group-1`.
- Kafka interacts with the consumer in the same way as pub-sub messaging until a new consumer subscribes to the same topic, `Topic-01`, with the same Group ID as `Group-1`.
- Once the new consumer arrives, Kafka switches its operation to share mode, such that each message is passed to only one of the subscribers
of the consumer group `Group-1`. This message transfer is
similar to queue-based messaging, as only one consumer of the group
consumes a message. Contrary to queue-based messaging, messages are not
removed after consumption.
- This message transfer can go on until the number of consumers
reaches the number of partitions configured for that particular topic.
- Once the number of consumers exceeds the number of partitions, the
new consumer will not receive any message until an existing consumer
unsubscribes. This scenario arises because each consumer in Kafka will
be assigned a minimum of one partition. Once all the partitions are
assigned to the existing consumers, the new consumers will have to wait.

# 7. ZooKeeper

## 7.1 What is ZooKeeper

- A distributed configuration and synchronization service
- In Kafka case, help to store basic metadata
    - information about brokers
    - topics
    - partitions
    - partition leader/ followers
    - consumer offsets

## 7.2 Act as central coordinator

ZooKeeper is used for storing all sorts of metadata about the Kafka cluster:

- It maintains the **last offset position** of each consumer group per partition, so that consumers can quickly recover from the last position in case of a failure (although modern clients store offsets in a
separate Kafka topic).
- It tracks the topics, number of partitions assigned to those topics, and leaders’/followers’ location in each partition.
- It also manages the access control lists (ACLs) to different topics in the cluster. ACLs are used to enforce access or authorization.

## 7.3 How to find leaders

- The producer connects to any broker and asks for the leader of Partition 1
    - each broker contains metadata
    - each brokers will talk to zooKeeper to get the latest metadata
- The broker responds with the identification of the leader broker responsible for partition 1
- The producer connects to the leader broker to publish the message

# 8. Controller Broker

- Within the Kafka cluster, one broker will be elected as the Controller
- Responsibility
    - admin operations
        - creating/ deleting a topic
        - adding partitions
        - assigning leaders to partitions
        - monitoring broker failures
    - check the health of other brokers in the system periodically
    - communicates the result of the partition leader election to other brokers in the system

## 8.1 Split brain issue

- some controller has temporary issue, during the period, we assign a new controller, but the previous one auto recover, so we have two controllers and it could bring inconsistency easily.
- Solution:
    - Generation Clock
        - simply a monotonically increasing number to indicate a server’s generation
        - If the old leader had an epoch number of ‘1’, the new one would have ‘2’.
        - This epoch is included in every request that is sent from the Controller to other brokers.
        - This way, brokers can now easily differentiate the real Controller by simply trusting the Controller with the highest number.
        - The Controller with the highest number is undoubtedly the latest one, since the epoch number is always increasing.
        - This epoch number is stored in ZooKeeper.

# 9. Delivery Semantics

## 9.1 Producer Delivery Semantics

- How can a producer know that the data is successfully stored at the leader or that the followers are keeping up with the leader
- Kafka offers three options to denote the **number of brokers** that **must receive the record** before the **producer considers the write as successful**
    - Async
        - Producer sends a msg to kafka and does not wait for acknowledgement from the server
        - fire-and-forget approach gives the best performance as we can write data to Kafka at network speed, but **no guarantee can be made** that the server has received the record in this case.
    - Committed to Leader
        - Producer waits for an acknowledgment from the leader.
        - This ensures that the data is committed at the leader; it will be slower than the ‘Async’ option, as the data has to be written on disk on the leader.
        - Under this scenario, the leader will respond without waiting for acknowledgments from the followers.
        - In this case, the record **will be lost if the leader crashes immediately after acknowledging the producer but before the followers have replicated it**.
    - Committed to Leader and Quorum
        - Producer waits for an acknowledgment from the leader and the quorum. This means the leader will wait for the full set of in-sync replicas to acknowledge the record. This will be the slowest write but guarantees that the record will not be lost as long as at least one in-sync replica remains alive. This is the strongest available guarantee.

## 9.2 Consumer Delivery Semantics

- Ways to provide consistency to the consumer
    - At most once
        - Message may be lost but are never redelivered
        - Under this option, the consumer upon receiving a message, commit (or increment) the offset to the broker. Now, if the consumer crashes before fully consuming the message, that message will be lost, as when the consumer restarts, it will receive the next message from the last committed offset.
    - At least once
        - Messages are never lost but maybe redelivered
        - This scenario occurs when the consumer receives a message from Kafka, and it does not immediately commit the offset.
        - Instead, it waits till it completes the processing.
        - So, if the consumer crashes after processing the message but before committing the offset, it has to reread the message upon restart.
        - Since, in this case, the consumer never committed the offset to the broker, the broker will redeliver the same message. Thus, duplicate message delivery could happen in such a scenario.
    - Exactly once
        - It is very hard to achieve this unless the consumer is working with a transactional system.
        - Under this option, the consumer puts the message processing and the offset increment in one transaction.
        - This will ensure that the offset increment will happen only if the whole transaction is complete.
        - If the consumer crashes while processing, the transaction will be rolled back, and the offset will not be incremented. When the consumer restarts, it can reread the message as it failed to process it last time. This option leads to no data duplication and no data loss but can lead to decreased throughput.