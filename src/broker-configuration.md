# Broker Configuration

```properties
#========== Broker basics ==========#
# Process role
process.roles=broker

# Unique ID for each node
node.id=2

# Enable topic deletion 
delete.topic.enable=true

# Enable topic autocreation
auto.create.topics.enable=false 

#========== Network Configuration ==========#

# Address where the broker listens
listeners=BROKER://0.0.0.0:9092

# Address advertised to clients and other brokers
# Change localhost if clients connect from another machine
advertised.listeners=BROKER://localhost:9092

# Listener used for communication between brokers
inter.broker.listener.name=BROKER

# Controller listener
controller.listener.names=CONTROLLER

# Listener protocol mapping
listener.security.protocol.map=BROKER:PLAINTEXT,CONTROLLER:PLAINTEXT

# KRaft controller quorum
controller.quorum.voters=1@localhost:9093

#========== Logs basics ==========#
# Comma separated list of directories to store log files
log.dirs=/data/kafka

# Default number of partitions for new topics
num.partitions=0

# Default replication factor 
default.replication.factor=3

# Minimum number of in-sync replicas
min.insync.replicas=2

#========== Logs Retention Policy ==========#
# Delete records older than 7 days
log.retention.hours=168

# Maximum size of a log segment file: 1 GiB
# When reached, Kafka creates a new segment
log.segment.bytes=1073741824

# Interval for checking whether log segments
# can be deleted according to retention policies
# 300000 ms = 5 minutes
log.retention.check.interval.ms=300000


#========== Performance Configuration ==========#
background.threads=10
num.io.threads=8
num.network.threads=3
```