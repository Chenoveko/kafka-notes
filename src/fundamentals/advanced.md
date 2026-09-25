# Advanced Topic Configuration
[Topic Config](https://kafka.apache.org/41/configuration/topic-configs/)
[Conduktor Advanced Kafka topics](https://www.conduktor.io/kafka/kafka-topics-advanced)

## Topic Partitions and Segments
The basic storage unit of Kafka is a partition replica. When you create a topic, Kafka first decides how to allocate the partitions between brokers. It spreads replicas evenly among brokers.

Kafka brokers split each partition into segments. Each segment is stored in a single data file on the disk attached to the broker. By default, each segment contains either 1 GB of data or a week of data, whichever limit is attained first.

When the Kafka broker receives data for a partition, as the segment limit is reached, it will close the file and start a new one:

![Segments](https://www.conduktor.io/assets/kafka/Adv-Kafka-Topic-Internals-1.png)

Only one segment is ACTIVE at any point in time - the one data is being written to. A segment can only be deleted if it has been closed beforehand.

**Segment Configuration**

| Configuration       | Default | Description                          |
|---------------------|---------|--------------------------------------|
| `log.segment.bytes` | 1 GB    | Maximum size of a single segment     |
| `log.segment.ms`    | 7 days  | Time before closing segment if not full |

These broker-level configurations can be overridden at the topic level using `segment.bytes` and `segment.ms`. See [log retention](https://www.conduktor.io/kafka/kafka-topic-configuration-log-retention) for more details.

A Kafka broker keeps an open file handle to every segment in every partition - even inactive segments. This leads to a usually high number of open file handles, and the OS has to be tuned accordingly.


## Topic Segments and Indexes
Kafka allows consumers to start fetching messages from any available offset. To help brokers quickly locate the message for a given offset, Kafka maintains two indexes for each segment:

| Index type            | Purpose                              | Use case                       |
|-----------------------|--------------------------------------|--------------------------------|
| Offset to position    | Maps offset to byte position in segment | Fast message lookup by offset |
| Timestamp to offset  | Maps timestamp to nearest offset     | Time-based message seeking    |

![f](https://www.conduktor.io/assets/kafka/Adv-Kafka-Topic-Internals-2.png)

## kafka-configs.sh
**Add Topic Config**
```bash
kafka-configs.sh --bootstrap-server localhost:9092 \
    --entity-type topics \
    --entity-name my-topic \
    --alter --add-config max.message.bytes=128000
```
**Check Topic Config**
```bash
kafka-configs.sh --bootstrap-server localhost:9092 \
    --entity-type topics \
    --entity-name my-topic \
    --describe
```
**Delete Topic Config**
```bash
kafka-configs.sh --bootstrap-server localhost:9092 \
    --entity-type topics \
    --entity-name my-topic \
    --alter --delete-config max.message.bytes
```
## Log Cleanup Policies
Kafka stores messages for a set amount of time and purges messages older than the retention period. This expiration happens due to a policy called log.cleanup.policy. There are two cleanup policies:
| Policy   | Default for          | Behavior                                  |
|----------|----------------------|-------------------------------------------|
| `delete` | User topics          | Deletes events older than retention time  |
| `compact`| `__consumer_offsets` | Keeps only the most recent value per key   |

```bash
kafka-topics.sh --bootstrap-server localhost:9092 \
    --describe \
    --topic __consumer_offsets
```

### Log Cleanup Policies: Delete
Kafka log retention controls how long messages are stored before being deleted. Understanding retention configuration is essential for managing storage costs, compliance requirements, and consumer catch-up scenarios.

**Retention by Time**

| Setting                 | Default      | Description                    |
|-------------------------|--------------|--------------------------------|
| `log.retention.hours`   | 168 (7 days) | Retention time in hours        |
| `log.retention.minutes` | -            | Retention time in minutes      |
| `log.retention.ms`      | -            | Retention time in milliseconds |

**Retention by Size**

| Setting               | Default        | Description                |
|-----------------------|----------------|----------------------------|
| `log.retention.bytes` | -1 (unlimited) | Maximum size per partition |

![dd](https://www.conduktor.io/assets/kafka/Adv-Kafka-Topic-Log-Comp-1.png)

### Log Cleanup Policies: Compact
Log compaction is Kafka's alternative cleanup policy that keeps only the most recent value for each key, making it ideal for changelog-style topics. This guide covers both the theory and hands-on practice of log compaction.

Kafka supports use cases by allowing the retention policy on a topic to be set to compact, with the property to only retain at least the most recent value for each key in the partition. It is very useful if we just require a SNAPSHOT instead of full history.

**Example**

We want to keep the most recent salary for our employees. We create a topic named `employee-salary` for the purpose. We don't want to know about the old salaries of the employees.

![f](https://www.conduktor.io/assets/kafka/Adv-Kafka-Topic-Log-Comp-2.png)

Applications producing these events should contain both a key and a value. The key, in this case, will be employee id, and the value will be their salary. As the data comes in, it will be appended into segments of a partition. After compaction, a new segment is created with only the latest events for a key being retained. The older events for that key are deleted, the offset of the messages are kept intact.

**Guarantees**

There are some important guarantees that Kafka provides for messages produced on the log-compacted topics:

- Tail consumers see all messages: Any consumer that is reading from the tail of a log, i.e., the most current data, will still see all the messages sent to the topic
- Ordering preserved: Ordering of messages at the key level and partition level is kept, log compaction only removes some messages, but does not re-order them
- Offsets are immutable: The offset of a message never changes. Offsets are just skipped if a message is missing
- Deleted records visible briefly: Deleted records can still be seen by consumers for a period of `log.cleaner.delete.retention.ms` (default is 24 hours)

**Myth Busting**

Let us look at some of the misconceptions around log compaction and clear them.

What log compaction does NOT do:

- It doesn't prevent duplicate data: De-duplication is done after a segment is committed. Your consumers will still read from the tail as soon as data arrives
- It doesn't prevent reading duplicates: If a consumer re-starts, it may see duplicate data based on at-least-once semantics

You also can't trigger log compaction using an API call—it happens in the background automatically if enabled.


**How log compaction works**

If compaction is enabled when Kafka starts, each broker will start a compaction manager thread and a number of compaction threads. These are responsible for performing the compaction tasks.

![f](https://www.conduktor.io/assets/kafka/Adv-Kafka-Topic-Log-Comp-3.png)

1. Cleaner threads start with the oldest segment and check their contents. The active segments are left untouched
2. If the message it has just read is still the latest for a key, it copies over the message to a replacement segment. Otherwise it omits the message
3. Once the cleaner thread has copied over all the messages that still contain the latest value for their key, we swap the replacement segment for the original
4. At the end of the process, we are left with one message per key - the one with the latest value


**Log compaction configurations**

| Configuration                     | Default    | Description                          |
|-----------------------------------|------------|--------------------------------------|
| `log.cleaner.enable`              | true       | Enable/disable log compaction        |
| `log.cleaner.threads`             | 1          | Background threads for log cleaning  |
| `log.segment.ms`                  | 7 days     | Max time before closing active segment |
| `log.segment.bytes`              | 1 GB       | Max size of a segment                |
| `log.cleaner.delete.retention.ms` | 24 hours   | How long tombstones are visible      |
| `log.cleaner.backoff.ms`          | 15 seconds | Sleep time when no logs to clean     |
| `min.cleanable.dirty.ratio`       | 0.5        | Minimum dirty ratio to trigger cleaning |

**Log compaction practice**

Create a log-compacted topic
```bash
kafka-topics --bootstrap-server localhost:9092 --create --topic employee-salary \
    --partitions 1 --replication-factor 1 \
    --config cleanup.policy=compact \ # Enables log compaction
    --config min.cleanable.dirty.ratio=0.001 \ # Ensures log cleanup is triggered frequently (for testing)
    --config segment.ms=5000 # New segment every 5 seconds (compaction only happens on closed segments)
```
Describe the topic to verify
```bash
kafka-topics --bootstrap-server localhost:9092 --describe --topic employee-salary
```
Produce messages with duplicated keys
```bash
kafka-console-producer --bootstrap-server localhost:9092 \
    --topic employee-salary \
    --property parse.key=true \
    --property key.separator=,
```
```text
Patrick,salary: 10000
Lucy,salary: 20000
Bob,salary: 20000
Patrick,salary: 25000
Lucy,salary: 30000
Patrick,salary: 30000
```
Wait a minute, and produce a few more messages:
```text
John,salary: 0
```
Consume and verify compaction
```bash
kafka-console-consumer --bootstrap-server localhost:9092 \
    --topic employee-salary \
    --from-beginning \
    --property print.key=true \
    --property key.separator=,
```
After compaction completes, you'll see only the unique keys with their latest values:
```text
Bob,salary: 20000
Lucy,salary: 30000
Patrick,salary: 30000
John,salary: 0
```
Log compaction will take place in the background automatically. We cannot trigger it explicitly. However, we can control how often it is triggered with the log compaction properties.

## Topic Naming Conventions
[Topic Naming Conventions](https://www.conduktor.io/kafka/kafka-topics-naming-convention)

## Unclean Leader Election
When no in-sync replicas (ISRs) are available, Kafka has to decide between staying unavailable or promoting an out-of-sync replica to leader. The `unclean.leader.election.enable` configuration controls this critical trade-off.

## Large Messages 
Kafka's default 1 MB message size limit can be raised — set Kafka max message size with `message.max.bytes` and `max.request.size`, plus compression alternatives.