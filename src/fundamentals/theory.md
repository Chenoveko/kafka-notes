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

## Topic Replication

## Producer Acknowledgement and Topic Distribution

## Kafka KRaft



[^1]: A **record batch** is a group of Kafka records grouped together before being sent to a broker. Kafka can compress the entire batch to save space and improve performance.