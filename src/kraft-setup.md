# KRaft Cluster Setup

## KRaft Protocol
[A deep dive into Apache Kafka's KRaft protocol - Red Hat Developer](https://developers.redhat.com/articles/2025/09/17/deep-dive-apache-kafkas-kraft-protocol#)

KRaft replaces ZooKeeper with a self-managed metadata quorum based on the Raft protocol.

Kafka nodes can have two roles configured with `process.roles`:

- **Controller**: manages cluster metadata.
- **Broker**: stores and serves messages.

Controllers replicate metadata and elect a leader to keep the cluster consistent.

## KRaft deployment modes

KRaft supports two deployment modes:

- **Combined Mode**: Controllers and brokers run in the same process. Best for:
    - Development environments
    - Small clusters (3-5 nodes)
    - Simplified operations
```properties
process.roles=broker,controller
```
- **Isolated mode**: Controllers and brokers run as separate processes. Best for:
    - Production environments
    - Large clusters
    - Maximum stability

```properties
# On controller nodes
process.roles=controller

# On broker nodes
process.roles=broker
```

## Controller deployment 
```bash
kafka-server-start.sh config/kraft/controller.properties
```

## Brokers deployment 
```bash
# On broker 1 node
kafka-server-start.sh config/kraft/broker1.properties

# On broker 2 node
kafka-server-start.sh config/kraft/broker2.properties

# etc
```