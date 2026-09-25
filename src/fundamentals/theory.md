# Kafka Theory

## Topics, Partitions and Offsets
### Topics
A **topic** is a particular stream of data
- You can create as many topics as you need.
- A topic is identified by its **name**.
- Topics support any kind of message format, such as JSON, text, Avro or binary.
- The sequence of messages stored in a topic is called a **data stream**.
- You cannot query a topic directly like a database table.
- **Kafka Producers** send messages to topics.
- **Kafka Consumers** read messages from topics.
Examples of topics:

```text
orders
payments
users
notifications
inventory
```
Example message in the `orders` topic:
```json
{
  "order_id": 1001,
  "customer_id": 25,
  "amount": 59.99,
  "status": "created"
}
```
![Topic](https://www.conduktor.io/assets/kafka/Apache-Kafka-Cluster-with-4-topics.png)

### Partitions and Offsets
Topics are split in **partitions**
- Messages within each partition are ordered
- Each message within a partition gets an incremental id, called **offset**
- topic are **immutable**: once data is written to a partition, it cannot be changed
- data is kept only for a limited time
- offset only have a meaning for a specific partition. E.g: offset 3 in partition0 does not represent the same data as offset 3 in partition 1
- offsets are not re-used even if a previous message have been deleted
- Order is guaranteed only within a partition, not across partitions
- Data is assigned randomly to a partition unless a key is provided
- You can have as many partitions per topic as you want

![Partitions](https://www.conduktor.io/assets/kafka/Kafka-Topics-1.png)

## Producers, Message and Serialization
### Producers
- Producers write data to topics (which are made of partitions)
- Producers determine which partition should receive each record.
- Producers know which Kafka broker is the leader for the target partition

![Producers](https://www.conduktor.io/assets/kafka/Kafka-Producers-1.png)

### Message Keys
- Producers can choose to send a **key** wit hthe message (string, number, binary, etc ...)
- A key are typically snet if you need message ordering for a specific field. E.g:
    - if `key=null`, data is send round robin (partition 0, then 1, then 2)
    - if `key!=null` then all messages for that key will always go to th same partition (hashing)

![Message Key](https://www.conduktor.io/assets/kafka/Kafka-Producers-2.png)

### Message Anatomy
A Kafka message, also called a **record**, usually contains:
- **Topic** -> the destination where the message is sent.
- **Partition** -> the partition where Kafka stores the message.
- **Offset** -> the position of the message inside the partition. Kafka assigns it after receiving the message.
- **Key** -> an optional identifier used to decide the partition. The key is serialized.
- **Value** -> the actual message data. The value is serialized.
- **Compression Type** → the algorithm used to compress the record batch[^1].
- **Headers** -> optional metadata attached to the message.
- **Timestamp** -> the time associated with the message.

The topic, partition and offset are Kafka metadata. The producer mainly provides the key, value, headers and timestamp.

![Message Anatomy](https://www.conduktor.io/assets/kafka/Kafka-Producers-3.png)

### Message Serializer
Before a producer sends a message, the key and value must be converted into bytes. This process is called serialization.

Kafka cannot directly understand programming-language objects, such as Java, Python or JavaScript objects.

Serialization converts the key and value into a byte format that can be:
- sent over the network
- stored by Kafka brokers
- understood by different applications and programming languages
- efficiently processed and transferred
 
The producer serializes the key and value before sending the record to Kafka:
```text
Object → Serializer → Bytes → Kafka
```
The consumer deserializes the key and value after receiving the record:
```text
Kafka → Bytes → Deserializer → Object
```
Common serializers include:
```text
StringSerializer
IntegerSerializer
LongSerializer
ByteArraySerializer
JSON Serializer
Avro Serializer
Protobuf Serializer
```

![Message Serializer](https://www.conduktor.io/assets/kafka/Kafka-Producers-4.png)

## Consumers and Deserialization

### Consumers
Consumers read data from a topic identified by its name.

- Consumers use a **pull model**. This means that consumers request or fetch records from Kafka instead of Kafka pushing records to them.
- Consumers automatically know which broker to read from by requesting metadata from the Kafka cluster.
- Consumers read records from one or more partitions.
- Records are read in order, from the lowest to the highest offset, within each partition.
- Kafka does not guarantee ordering across different partitions.
- Consumers keep track of their offsets to know which records have already been processed.
- Consumers can commit their offsets automatically or manually.
- If a broker fails, the consumer can reconnect and continue reading from another available replica.
- Consumers do not delete messages after reading them. Messages remain available according to the topic's retention policy.
- Consumers can read old messages again by resetting their offsets.

![Consumer](https://www.conduktor.io/assets/kafka/Kafka-Consumers-1.png)

### Deserialization
Kafka stores and transfers the key and value of a record as bytes. 

Deserialization is the process of converting these bytes back into objects or usable data. 

The consumer must use a deserializer compatible with the serializer used by the producer. For example:
```text
Producer:
Order object → JSON Serializer → Bytes → Kafka

Consumer:
Kafka → Bytes → JSON Deserializer → Order object
```
The key and value can use different deserializers:
```text
Key:   IntegerDeserializer
Value: JSON Deserializer
```
If the wrong deserializer is configured, the consumer may fail to read the record or interpret the data incorrectly.

![Message Deserializer](https://www.conduktor.io/assets/kafka/Kafka-Consumers-2.png)

## Consumer Groups and Consumer Offsets

### Consumer Groups
All the consumers in an application read data as consumer groups. Each consumer within a group reads from exclusive partitions

![Consumer group 1](https://www.conduktor.io/assets/kafka/Consumer-Group-reading-from-topic-with-5-partitions.png)

In the case of too many consumers, if there are more consumers than partitions, some consumers will be inactive

![Consumer group 2](https://www.conduktor.io/assets/kafka/Kafka-Consumer-Groups-2.png)

In Apache Kafka it is acceptable to have multiple consumer groups on the same topic

![Consumer group 3](https://www.conduktor.io/assets/kafka/Kafka-Consumer-Groups-1.png)

To create distinct consumer groups, use the consumer property `group.id`

### Consumer Offsets
An **offset** is the position of a record inside a Kafka partition.

It works like a **bookmark**. It tells a consumer which record it should read next.
```text
Offset 0 → Record 1
Offset 1 → Record 2
Offset 2 → Record 3
Offset 3 → Record 4
```
- Kafka stores the offset commited at which a consumer group group has been reading in the internal topic `__consumer_offsets`
- When a consumer in a group has processed data received from Kafka, it should be periodically commiting the offset (the kafka broker will write to `__consumer__offsets`, not the group itself)
- If a consumer dies, it will be able to read back from where it left off thanks to the committed consumer offsets
- If a record was processed but its offset was not committed, the record may be processed again.

Example:
```text
Record 0 → Processed
Record 1 → Processed
Record 2 → Processed
Committed offset: 3
```
The committed offset is `3` because record `3` is the next record that the consumer should read.
![Consumer Offset](https://www.conduktor.io/assets/kafka/Kafka-Consumer-Groups-3-2x.png)

## Brokers and Topics

### Kafka Brokers
A Kafka cluster is composed of one or more **brokers**.

- A broker is a Kafka server responsible for storing and serving records.
- Each broker is identified by a unique integer **broker ID**.
- Brokers store one or more topic partitions.
- A broker can be the leader for some partitions and a follower replica for others.
- A broker does not necessarily contain all the data in the cluster.
- Kafka distributes partitions across brokers to improve scalability and availability.
- A production cluster commonly uses at least three brokers, although the required number depends on the workload and fault-tolerance requirements.
- In this example, broker IDs start at `100`, but this is arbitrary.

![Kafka Cluster](https://www.conduktor.io/assets/kafka/Kafka-Brokers-1.png)

### Kafka Brokers and topics
In this example:

- **Topic A** has three partitions.
- **Topic B** has two partitions.
- Kafka distributes these partitions across the available brokers.
- A broker may not contain any partition from a particular topic.
- The broker dont have all the data, they only have the data they should have
- For example, broker `103` does not contain any partition from **Topic B**. This is normal because Topic B has only two partitions, and they have been assigned to other brokers.
- Brokers store only the partitions assigned to them.
- The more partitions and brokers a cluster has, the more data and traffic can be distributed across the cluster.
- This distribution is called **horizontal scaling**.
- Partitions allow Kafka to process records in parallel.

Example:
```text
Topic A:
Partition 0 → Broker 100
Partition 1 → Broker 101
Partition 2 → Broker 102

Topic B:
Partition 0 → Broker 100
Partition 1 → Broker 101
```
Broker `103` does not contain data from Topic B because no Topic B partition was assigned to it, because the two partitions have already been placed on our kafka cluster

![Kafka Broker and Topics](https://www.conduktor.io/assets/kafka/Kafka-Brokers-2.png)

### Kafka Brokers Discovery
Kafka clients need to connect to the cluster before they can produce or consume records.

- The bootstrap.servers configuration contains one or more initial broker addresses.
- These brokers are called **bootstrap servers.**
- A bootstrap server is not a special type of broker. It is simply a broker used for the initial connection.
- Each broker knows about all brokers, topcis and partitions (metadata)
- A bootstrap server is not a special type of broker. It is simply a broker used for the initial connection.
- The client only needs to connect to one available bootstrap server to obtain cluster metadata.
- After connecting, the client learns about all brokers, topics and partitions.
- The client also learns which broker is the leader for each partition.
- Kafka clients are therefore considered smart clients because they use this metadata to communicate with the correct broker.
- It is recommended to configure multiple bootstrap servers so the client can connect even if one broker is unavailable.

Example of bootstrap server configuration
```text
bootstrap.servers=broker1.example.com:9092,broker2.example.com:9092
```

![Kafka Broker Discovery](https://www.conduktor.io/assets/kafka/Kafka-Brokers-3.png)

## Topic Replication

### Topic Replication Factor
The **replication factor** defines how many copies of each partition Kafka stores.

- A replication factor greater than `1` is recommended for production topics. Common values are `2` or `3`.
- A replication factor of `3` means that each partition has three replicas.
- Replicas are stored on different brokers.
- If a broker fails, another in-sync replica can become the leader.
- The replication factor cannot be greater than the number of brokers in the cluster.
- A higher replication factor improves fault tolerance but requires more storage.

![Kafka Topic Replication](https://www.conduktor.io/assets/kafka/Kafka-Topic-Replication-1.png)

### Broker Failure
In this example, broker `102` fails.

- The replicas stored on brokers `101` and `103` are still available.
- Producers and consumers receive updated metadata and connect to the new leader.
- The cluster may temporarily have fewer replicas than expected.
- This situation is called **under-replication**
- When broker `102` comes back, it can replicate the missing data and become synchronized again.

![Kafka Topic Replication](https://www.conduktor.io/assets/kafka/Kafka-Topic-Replication-2.png)

### Leader for a Partition
At any time, only one broker can be the leader for a given partition.

- Producers send records to the partition leader.
- Consumers normally read records from the partition leader.
- The other replicas are called followers.
- Followers copy records from the leader.
- If the leader fails, Kafka can elect another in-sync replica as the new leader.
- A partition has one leader and one or more follower replicas.
- The leader is also considered part of the replica set and the ISR when it is in sync.
- The ISR, or In-Sync Replica Set, contains the replicas that are sufficiently synchronized with the leader.
- The ISR is a set of replicas, not a separate type of broker.

Example:
```text
Broker 101 → Leader
Broker 102 → Follower and ISR
Broker 103 → Follower and ISR
```

![Kafka Topic Replication](https://www.conduktor.io/assets/kafka/Kafka-Topic-Replication-3.png)

### In-Sync Replicas (ISR) [^2]
**In-Sync Replicas**, or **ISR**, are the replicas of a partition that are sufficiently synchronized with the leader.

- The ISR includes the partition leader and the followers that are up to date.
- A follower can be removed from the ISR if it fails or falls too far behind the leader.
- Only replicas in the ISR are normally eligible to become the new leader.
- When a broker recovers and catches up with the leader, it can be added to the ISR again.
- The ISR can become smaller when a broker fails.

Example:
```text
Partition 0:

Broker 101 → Leader
Broker 102 → Follower
Broker 103 → Follower

ISR = Broker 101, Broker 102, Broker 103
```
If broker `103` fails or falls behind:
```text
Partition 0:

Broker 101 → Leader
Broker 102 → Follower
Broker 103 → Out of sync

ISR = Broker 101, Broker 102
```
If broker `101` fails, broker `102` can become the new leader because it belongs to the ISR.

### Default Producer and Consumer Behavior with Leaders
By default:

- Kafka producers write records to the leader broker of a partition.
- Kafka consumers read records from the leader broker of a partition.
- Followers replicate the records written to the leader.
- Kafka clients use metadata to discover the current leader.
- If the leader changes, clients refresh their metadata and connect to the new leader.

![Kafka Topic Replication](https://www.conduktor.io/assets/kafka/Kafka-Topic-Partition-Leader--1-.png)

### Kafka Consumer Replica Fetching (Kafka v2.4+)
Since Kafka `v2.4`, consumers can be configured to read from the closest replica instead of always reading from the leader.

- This feature is called follower fetching or rack-aware replica fetching.
- It can reduce network latency.
- It can reduce cross-zone or cross-region network costs in cloud environments.
- The selected replica must be sufficiently synchronized.
- If no suitable replica is available, the consumer can read from the leader.

![Kafka Topic Replication](https://www.conduktor.io/assets/kafka/Kafka-Consumers-Replica-Fetching.png)

## Producer Acknowledgement and Topic Durability

### Producer Acknowledgement (acks)

Producers can choose to recevie acknowledgement of data writes
- `acks=0` -> The producer does not wait for an acknowledgement from the broker.

![Kafka Topic Replication](https://www.conduktor.io/assets/kafka/Adv-Producer-Acks-DD-1.png)

- `acks=1` -> The producer waits for an acknowledgement from the partition leader.

![Kafka Topic Replication](https://www.conduktor.io/assets/kafka/Adv-Producer-Acks-DD-2.png)

- `acks=all` -> The producer waits until all currently in-sync replicas acknowledge the record. The producer may receive an error if the number of in-sync replicas is lower than `min.insync.replicas`. `acks=all` is equivalent to `acks=-1`.

![Kafka Topic Replication](https://www.conduktor.io/assets/kafka/Adv-Producer-Acks-DD-3.png)

The `min.insync.replicas` setting defines the minimum number of in-sync replicas required to accept a write when the producer uses `acks=all`

Example: 
```text
replication.factor=3
min.insync.replicas=2
acks=all
```

![Kafka Topic Replication](https://www.conduktor.io/assets/kafka/Adv-Producer-Acks-DD-4.png)

| Setting    | Durability | Throughput | Latency | Use case       |
|------------|------------|------------|---------|----------------|
| `acks=0`   | None       | Highest    | Lowest  | Metrics, logs  |
| `acks=1`   | Leader only| High       | Low     | Most applications |
| `acks=all` | Full       | Lower      | Higher  | Critical data  |

### Topic Durability and Availability
For a topic replication factor of `3`, topic data durability can withstand the loss of `2` brokers. As a general rule, for a replication factor of `N`, you can permanently lose up to `N-1` brokers and still recover your data.

## Kafka KRaft

**KRaft** stands for **Kafka Raft**. It is Kafka's built-in consensus protocol that replaces Apache ZooKeeper.

- KRaft manages Kafka cluster metadata using the Raft consensus protocol.
- It removes the need to run a separate ZooKeeper cluster.
- Kafka controllers form a quorum to manage metadata.
- Metadata includes brokers, topics, partitions, configurations and leader elections.
- Brokers are responsible for storing and serving the actual actual records.
- Controllers are responsible for managing the Kafka cluster.
- KRaft simplifies Kafka deployment because Kafka no longer depends on an external coordination system.
- KRaft improves scalability, recovery time and operational simplicity.
- KRaft is recommended for new Kafka clusters.

![Kafka Topic Replication](https://images.ctfassets.net/gt6dp23g0g38/7gQZn9CnRAT60NeyYBYflL/b144fee6dad28ce97c3e91e6d09d1167/20230616-Diagram-KRaft.jpg)

![Kafka Topic Replication](https://images.ctfassets.net/gt6dp23g0g38/1NwuwtNGfOmmT4fCUimzhR/eb88cb4ea6903b9a451a5a43fc6870bc/timed-shutdown-operations-in-apache-kafka-with-or-without-zookeeper.png)


## Kafka Theory Roundup
![Kafka Topic Replication](https://substackcdn.com/image/fetch/$s_!KB6A!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fff34925c-b3f9-467b-b512-5f2c89c923c0_1728x2464.jpeg)

[^1]: A **record batch** is a group of Kafka records grouped together before being sent to a broker. Kafka can compress the entire batch to save space and improve performance.
[^2]: [Understanding In-Sync Replicas (ISR) in Apache Kafka](https://www.geeksforgeeks.org/apache-kafka/understanding-in-sync-replicas-isr-in-apache-kafka/)