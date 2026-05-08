# Apache Kafka - 100 Interview Questions & Answers

**Your Strength:** "Owned and scaled real-time data pipelines (Kafka + Spark) processing millions of records daily"

---

## FUNDAMENTALS (Questions 1-25)

### Q1: What is Apache Kafka? Why use it?

**Answer:**

Apache Kafka is a distributed streaming platform for building real-time data pipelines and streaming applications.

**Core Capabilities:**
- **Publish/Subscribe:** Producers send messages, consumers receive them
- **Storage:** Durable message storage (configurable retention)
- **Stream Processing:** Process streams in real-time

**Key Features:**
- High throughput (millions of messages/second)
- Low latency (milliseconds)
- Fault-tolerant (replication)
- Scalable (horizontal scaling)
- Durable (persistent storage)

**Use Cases:**
- Real-time analytics
- Event sourcing
- Log aggregation
- Metrics collection
- CDC (Change Data Capture)
- Microservices messaging

**vs Traditional Message Queues:**
| Feature | Kafka | RabbitMQ/ActiveMQ |
|---------|-------|-------------------|
| Throughput | Very High | Moderate |
| Ordering | Per partition | Per queue |
| Persistence | Always | Optional |
| Retention | Time-based | Until consumed |
| Replayability | Yes | No |

---

### Q2: Explain Kafka architecture components

**Answer:**

**Components:**

**1. Broker:**
- Kafka server that stores and serves data
- Each broker handles reads/writes
- Identified by unique ID

**2. Topic:**
- Category/feed name for messages
- Logical channel for data
- Split into partitions

**3. Partition:**
- Ordered, immutable sequence of messages
- Topic divided into partitions for parallelism
- Each message has an offset (position)

**4. Producer:**
- Publishes messages to topics
- Chooses which partition to send to

**5. Consumer:**
- Reads messages from topics
- Tracks position (offset)
- Part of consumer group

**6. ZooKeeper (legacy) / KRaft (new):**
- Cluster coordination
- Metadata management
- Leader election

**Architecture Diagram:**
```
Producers → Topic (Partition 0, 1, 2) → Consumers
              ↓
          Brokers (replicated across 3+ servers)
              ↓
          ZooKeeper/KRaft (coordination)
```

---

### Q3: What is a Kafka topic?

**Answer:**

**Topic:** Named category/stream of messages

**Characteristics:**
- Logical entity (physical data in partitions)
- Can have multiple partitions
- Can have multiple producers and consumers
- Retention configurable (time or size-based)

**Example:**
```bash
# Create topic
kafka-topics --create \
  --bootstrap-server localhost:9092 \
  --topic user-events \
  --partitions 6 \
  --replication-factor 3

# Describe topic
kafka-topics --describe \
  --bootstrap-server localhost:9092 \
  --topic user-events

# Output:
# Topic: user-events  PartitionCount: 6  ReplicationFactor: 3
# Partition: 0  Leader: 1  Replicas: 1,2,3  Isr: 1,2,3
# Partition: 1  Leader: 2  Replicas: 2,3,1  Isr: 2,3,1
```

**Topic Configuration:**
```properties
# Retention time (7 days)
retention.ms=604800000

# Retention size (1GB per partition)
retention.bytes=1073741824

# Cleanup policy
cleanup.policy=delete  # or 'compact'

# Compression
compression.type=lz4  # or snappy, gzip, zstd
```

---

### Q4: What is a Kafka partition? Why important?

**Answer:**

**Partition:** Ordered, immutable sequence of messages within a topic

**Why Important:**

**1. Scalability:**
- More partitions = more parallelism
- Each partition can be on different broker
- Scale horizontally by adding partitions

**2. Ordering Guarantee:**
- Messages within a partition are ordered
- No ordering across partitions

**3. Consumer Parallelism:**
- One partition → one consumer (in a group)
- 6 partitions → up to 6 parallel consumers

**Example:**
```
Topic: user-events (6 partitions)

Partition 0: [msg1, msg2, msg3, msg4, ...]  ← User A events
Partition 1: [msg1, msg2, msg3, ...]        ← User B events
Partition 2: [msg1, msg2, ...]              ← User C events
...

# Same user's events always go to same partition (if keyed properly)
# Ensures ordering per user
```

**Partitioning Strategy:**
```java
// By key (default)
producer.send(new ProducerRecord<>("topic", userId, event));
// Messages with same key → same partition

// Round-robin (if no key)
producer.send(new ProducerRecord<>("topic", event));
// Distributed evenly across partitions

// Custom partitioner
class CustomPartitioner implements Partitioner {
    public int partition(String topic, Object key, ...) {
        return hash(key) % numPartitions;
    }
}
```

**Choosing Partition Count:**
- Target throughput / throughput per partition
- Number of consumers needed
- Consider broker count
- Can increase, but can't easily decrease

**Your Optum Example:**
"For RQNS member events, we used 24 partitions partitioned by member_id to ensure all events for the same member maintained order while achieving 200K messages/sec throughput."

---

### Q5: What are offsets in Kafka?

**Answer:**

**Offset:** Unique sequential ID (integer) for each message within a partition

**Characteristics:**
- Starts at 0, increments by 1
- Never changes for a message
- Unique only within a partition

**Types of Offsets:**

**1. Current Offset:**
- Next message to be read by consumer

**2. Committed Offset:**
- Last offset successfully processed
- Stored in `__consumer_offsets` topic
- Used for recovery after crash

**3. Log End Offset (LEO):**
- Offset of the last message in partition

**4. High Water Mark:**
- Offset of last replicated message

**Offset Management:**
```java
// Auto-commit (default)
properties.put("enable.auto.commit", "true");
properties.put("auto.commit.interval.ms", "5000");
// Commits offset every 5 seconds

// Manual commit (better control)
properties.put("enable.auto.commit", "false");

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        process(record);  // Process message
    }
    consumer.commitSync();  // Commit after processing
}

// Async commit
consumer.commitAsync();
```

**Offset Reset:**
```bash
# Reset to earliest
kafka-consumer-groups --bootstrap-server localhost:9092 \
  --group my-group \
  --topic my-topic \
  --reset-offsets --to-earliest \
  --execute

# Reset to specific offset
--reset-offsets --to-offset 1000

# Reset by time
--reset-offsets --to-datetime 2024-05-03T00:00:00.000
```

**Consumer Lag:**
```
Lag = (Log End Offset) - (Current Offset)

Example:
Log End Offset: 10000
Current Offset: 9500
Lag: 500 messages behind
```

---

### Q6: Explain Kafka producers

**Answer:**

**Producer:** Client that publishes messages to Kafka topics

**Key Concepts:**

**1. Send Modes:**
```java
// Fire and forget (fastest, can lose data)
producer.send(new ProducerRecord<>("topic", "key", "value"));

// Synchronous (wait for ack)
Future<RecordMetadata> future = producer.send(record);
RecordMetadata metadata = future.get();  // Blocks

// Asynchronous with callback
producer.send(record, new Callback() {
    public void onCompletion(RecordMetadata metadata, Exception e) {
        if (e != null) {
            log.error("Send failed", e);
        } else {
            log.info("Sent to partition " + metadata.partition());
        }
    }
});
```

**2. Producer Configuration:**
```java
Properties props = new Properties();

// Required
props.put("bootstrap.servers", "localhost:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

// Performance
props.put("batch.size", 16384);           // 16KB batches
props.put("linger.ms", 10);               // Wait 10ms for batching
props.put("buffer.memory", 33554432);     // 32MB buffer
props.put("compression.type", "lz4");     // Compression

// Reliability
props.put("acks", "all");                 // Wait for all replicas
props.put("retries", 3);                  // Retry 3 times
props.put("enable.idempotence", "true");  // Exactly-once
```

**3. Partitioning:**
```java
// Partition by key
ProducerRecord<String, String> record =
    new ProducerRecord<>("topic", userId, event);
// Same userId → same partition

// Specify partition
ProducerRecord<String, String> record =
    new ProducerRecord<>("topic", 2, userId, event);
// Goes to partition 2
```

**4. Serialization:**
```java
// String serializer (built-in)
props.put("value.serializer", "...StringSerializer");

// Avro serializer
props.put("value.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
props.put("schema.registry.url", "http://localhost:8081");

// Custom serializer
class UserSerializer implements Serializer<User> {
    public byte[] serialize(String topic, User data) {
        return data.toJson().getBytes();
    }
}
```

---

### Q7: Explain Kafka consumers and consumer groups

**Answer:**

**Consumer:** Client that reads messages from Kafka topics

**Consumer Group:** Set of consumers working together to consume a topic

**Key Rules:**
1. Each partition assigned to exactly ONE consumer in a group
2. A consumer can read from multiple partitions
3. Multiple groups can consume same topic independently

**Example:**
```
Topic: events (6 partitions)
Consumer Group: analytics (3 consumers)

Consumer 1 → Partitions 0, 1
Consumer 2 → Partitions 2, 3
Consumer 3 → Partitions 4, 5

# Each partition consumed by exactly one consumer in the group
```

**Consumer Configuration:**
```java
Properties props = new Properties();

// Required
props.put("bootstrap.servers", "localhost:9092");
props.put("group.id", "my-consumer-group");
props.put("key.deserializer", "...StringDeserializer");
props.put("value.deserializer", "...StringDeserializer");

// Offset management
props.put("enable.auto.commit", "false");  // Manual commit
props.put("auto.offset.reset", "earliest");  // or "latest"

// Performance
props.put("fetch.min.bytes", 1024);         // Min 1KB
props.put("fetch.max.wait.ms", 500);        // Max wait 500ms
props.put("max.poll.records", 500);         // Max 500 records/poll
```

**Consuming Messages:**
```java
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

// Subscribe to topics
consumer.subscribe(Arrays.asList("topic1", "topic2"));

// Or assign specific partitions
TopicPartition partition0 = new TopicPartition("topic", 0);
consumer.assign(Arrays.asList(partition0));

// Poll loop
while (true) {
    ConsumerRecords<String, String> records =
        consumer.poll(Duration.ofMillis(100));

    for (ConsumerRecord<String, String> record : records) {
        System.out.printf("offset=%d, key=%s, value=%s%n",
            record.offset(), record.key(), record.value());

        // Process message
        process(record);
    }

    // Manual commit
    consumer.commitSync();
}
```

**Rebalancing:**
When consumer joins/leaves group, partitions are reassigned.

```
Before (2 consumers, 4 partitions):
Consumer 1 → P0, P1
Consumer 2 → P2, P3

New consumer joins:
Consumer 1 → P0
Consumer 2 → P1, P2
Consumer 3 → P3

# Rebalancing triggered!
```

---

### Q8: What is consumer rebalancing?

**Answer:**

**Rebalancing:** Process of reassigning partitions to consumers when group membership changes

**Triggers:**
1. Consumer joins group
2. Consumer leaves (crash or shutdown)
3. Consumer deemed dead (missed heartbeat)
4. Partitions added to topic

**Types:**

**1. Eager Rebalancing (default, older):**
- All consumers stop consuming
- Release all partitions
- Rejoin and get new assignments
- **Downside:** Pause in processing

**2. Cooperative Rebalancing (newer, incremental):**
- Only affected partitions reassigned
- Other consumers continue consuming
- **Benefit:** Minimal disruption

**Configuration:**
```java
// Heartbeat interval
props.put("heartbeat.interval.ms", 3000);  // 3 seconds

// Session timeout
props.put("session.timeout.ms", 10000);  // 10 seconds
// If no heartbeat for 10s, consumer considered dead

// Max poll interval
props.put("max.poll.interval.ms", 300000);  // 5 minutes
// Max time between polls before considered dead

// Cooperative rebalancing
props.put("partition.assignment.strategy",
    "org.apache.kafka.clients.consumer.CooperativeStickyAssignor");
```

**Rebalance Listeners:**
```java
consumer.subscribe(topics, new ConsumerRebalanceListener() {
    public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
        // Called before rebalancing
        // Commit offsets for partitions being revoked
        consumer.commitSync();
    }

    public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
        // Called after rebalancing
        // Partitions assigned, can start processing
    }
});
```

**Impact on Performance:**
- Processing paused during rebalance
- Can take seconds to minutes
- Minimize by: stable consumers, tuned timeouts

**Your Optum Example:**
"When scaling RQNS consumers from 6 to 12 instances, we tuned session timeout to 30s and used sticky assignor to minimize rebalance disruption, maintaining <1min rebalance time."

---

### Q9-Q25: Fundamentals (continued)

**Q9: What is a broker?**
- Kafka server that stores and serves data
- Handles reads/writes for partitions
- Multiple brokers form a cluster

**Q10: What is ZooKeeper's role?** (legacy)
- Cluster coordination
- Metadata storage
- Leader election
- **Note:** Being replaced by KRaft (Kafka Raft)

**Q11: What is KRaft mode?**
- Kafka without ZooKeeper
- Built-in consensus (Raft protocol)
- Simpler architecture
- Faster recovery

**Q12: What are replicas?**
- Copies of partition on different brokers
- Replication factor: number of copies
- One leader, rest are followers

**Q13: What is ISR (In-Sync Replicas)?**
- Replicas fully caught up with leader
- Critical for acks=all
- If replica falls behind, removed from ISR

**Q14: Explain leader and follower**
- **Leader:** Handles all reads/writes for partition
- **Follower:** Replicates leader, can become leader if leader fails

**Q15: What is retention?**
```properties
# Time-based
retention.ms=604800000  # 7 days

# Size-based
retention.bytes=1073741824  # 1GB per partition

# Whichever limit hit first
```

**Q16: What is log compaction?**
- Cleanup policy to keep latest value per key
- Useful for changelog topics
- Retains latest state

**Q17: What is a message/record?**
```
Message = Key + Value + Timestamp + Headers + Offset
```

**Q18: What are headers?**
- Metadata key-value pairs
- Don't affect partitioning
- Useful for routing, tracing

**Q19: Kafka message size limits?**
- Default max: 1MB
- Configurable: `message.max.bytes`
- **Best practice:** Keep messages small, store large data elsewhere

**Q20: What is `acks` parameter?**
```
acks=0: Fire and forget
acks=1: Wait for leader
acks=all (or -1): Wait for all ISR
```

**Q21: What is `min.insync.replicas`?**
- Minimum ISR required for `acks=all`
- Typical: 2 (with replication factor 3)
- Ensures durability

**Q22: What is idempotent producer?**
```java
props.put("enable.idempotence", "true");
// Prevents duplicate writes
// Automatically sets: acks=all, retries=MAX, max.in.flight=5
```

**Q23: What are transactions in Kafka?**
- Atomic writes across partitions
- Exactly-once semantics
- All-or-nothing delivery

**Q24: What is `auto.offset.reset`?**
```
earliest: Start from beginning
latest: Start from end (default)
none: Throw exception if no offset
```

**Q25: What is consumer lag?**
- Difference between latest offset and consumer's offset
- **Lag = Log End Offset - Current Offset**
- Monitor to ensure consumers keep up

---

## INTERMEDIATE (Questions 26-60)

### Q26: Explain exactly-once semantics (EOS)

**Answer:**

**Exactly-Once:** Guarantee that each message is processed exactly once, no duplicates or loss

**Three Pieces:**

**1. Idempotent Producer:**
```java
props.put("enable.idempotence", "true");
// Producer assigns sequence numbers to messages
// Broker detects duplicates and ignores them
```

**2. Transactional Producer:**
```java
props.put("transactional.id", "my-transaction-id");

producer.initTransactions();

try {
    producer.beginTransaction();
    producer.send(record1);
    producer.send(record2);
    producer.commitTransaction();
} catch (Exception e) {
    producer.abortTransaction();
}
```

**3. Transactional Consumer:**
```java
props.put("isolation.level", "read_committed");
// Only reads committed messages
// Ignores aborted transactions
```

**End-to-End EOS:**
```java
// Producer
props.put("enable.idempotence", "true");
props.put("transactional.id", "txn-1");

// Consumer
props.put("isolation.level", "read_committed");
props.put("enable.auto.commit", "false");

// Processing
producer.beginTransaction();
for (ConsumerRecord record : records) {
    // Process
    ProducerRecord output = process(record);
    producer.send(output);

    // Send offset to transaction
    Map<TopicPartition, OffsetAndMetadata> offsets = ...;
    producer.sendOffsetsToTransaction(offsets, "group-id");
}
producer.commitTransaction();
```

**Performance Impact:**
- Slight overhead (10-20%)
- Worth it for critical data (financial, billing)

---

### Q27: How does Kafka achieve high throughput?

**Answer:**

**1. Sequential I/O:**
- Writes are sequential (append-only log)
- Much faster than random I/O
- Leverages OS page cache

**2. Zero-Copy:**
- Data transferred from disk to network without copying to application memory
- Uses `sendfile()` system call

**3. Batching:**
```java
// Producer batching
props.put("batch.size", 16384);    // 16KB batches
props.put("linger.ms", 10);        // Wait 10ms to fill batch
// Trades latency for throughput

// Consumer batching
props.put("fetch.min.bytes", 1024);  // Wait for 1KB before returning
props.put("max.poll.records", 500);   // Fetch 500 records at once
```

**4. Compression:**
```java
props.put("compression.type", "lz4");  // or snappy, gzip, zstd
// Reduces network and disk usage
```

**5. Partitioning:**
- Parallelism across partitions
- Distributed across brokers
- Scale horizontally

**6. No Complex Routing:**
- Simple pub-sub model
- No message selectors or complex routing

**Your Optum Example:**
"Optimized RQNS Kafka pipeline from 50K to 200K msgs/sec by:
- Enabling lz4 compression (60% network reduction)
- Increasing batch size to 64KB
- Tuning fetch sizes on consumers
- Optimizing partition count to 24"

---

### Q28: What causes consumer lag? How to fix?

**Answer:**

**Causes:**

**1. Slow Processing:**
- Consumer processing takes too long
- Complex transformations
- Slow database queries
- Network calls in processing

**2. Under-provisioned:**
- Not enough consumer instances
- Fewer consumers than partitions

**3. Rebalancing:**
- Frequent rebalances pause processing
- Unstable consumers

**4. Network Issues:**
- Slow network between consumer and broker
- Network congestion

**Solutions:**

**1. Scale Consumers:**
```bash
# Add more consumer instances (up to # of partitions)
# 6 partitions → can have up to 6 consumers in group
```

**2. Optimize Processing:**
```java
// Bad: Synchronous DB call per message
for (ConsumerRecord record : records) {
    database.update(record);  // Slow!
}

// Good: Batch database operations
List<Record> batch = new ArrayList<>();
for (ConsumerRecord record : records) {
    batch.add(record);
    if (batch.size() >= 100) {
        database.batchUpdate(batch);
        batch.clear();
    }
}
```

**3. Increase Fetch Size:**
```java
props.put("max.poll.records", 500);      // Fetch more records
props.put("fetch.min.bytes", 1048576);   // 1MB minimum
```

**4. Parallelize Within Consumer:**
```java
ExecutorService executor = Executors.newFixedThreadPool(10);

for (ConsumerRecord record : records) {
    executor.submit(() -> process(record));
}
```

**5. Separate Heavy Processing:**
```java
// Consumer 1: Read from topic A, write to topic B (fast)
// Consumer 2: Read from topic B, do heavy processing
// Decouple fast ingestion from slow processing
```

**6. Increase Partitions:**
```bash
# More partitions = more parallelism
kafka-topics --alter --topic my-topic --partitions 12
```

**Monitoring:**
```bash
kafka-consumer-groups --bootstrap-server localhost:9092 \
  --group my-group --describe

# Shows:
# TOPIC  PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
# events 0          1000            1500            500   ← 500 behind
```

---

### Q29-Q60: Intermediate Topics (condensed for space)

**Q29: How to monitor Kafka?**
- JMX metrics (under-replicated partitions, lag, throughput)
- Tools: Prometheus, Grafana, Confluent Control Center
- Key metrics: Consumer lag, broker CPU/disk, ISR shrinks

**Q30: Kafka performance tuning - producer**
```java
batch.size=65536          // Larger batches
linger.ms=10              // Small wait for batching
compression.type=lz4      // Compression
buffer.memory=67108864    // 64MB buffer
```

**Q31: Kafka performance tuning - consumer**
```java
fetch.min.bytes=1048576       // 1MB minimum
fetch.max.wait.ms=500         // Max 500ms wait
max.poll.records=500          // Batch processing
```

**Q32: Kafka performance tuning - broker**
```properties
num.network.threads=8
num.io.threads=16
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
```

**Q33: What is Kafka Streams?**
- Java library for stream processing
- Processes data in Kafka topics
- Stateful operations (aggregations, joins, windows)
- No separate cluster needed

**Q34: Kafka Streams vs Spark Streaming?**
| Feature | Kafka Streams | Spark Streaming |
|---------|--------------|-----------------|
| Latency | Lower (ms) | Higher (seconds) |
| Language | Java only | Java, Scala, Python |
| Deployment | Library | Cluster |
| State | Built-in | Checkpointing |

**Q35: What is Kafka Connect?**
- Framework for connectors (sources and sinks)
- Source: DB → Kafka
- Sink: Kafka → DB/S3/etc
- No code needed for common integrations

**Q36: Source connector vs Sink connector?**
- **Source:** External system → Kafka (JDBC, File, etc.)
- **Sink:** Kafka → External system (S3, Elasticsearch, etc.)

**Q37: What is Schema Registry?**
- Stores Avro/Protobuf schemas
- Ensures compatibility
- Versions schemas
- Confluent component (not part of Apache Kafka)

**Q38: Avro vs JSON for Kafka?**
| Aspect | Avro | JSON |
|--------|------|------|
| Size | Smaller (binary) | Larger (text) |
| Schema | Required, versioned | Optional |
| Speed | Faster | Slower |
| Human-readable | No | Yes |

**Q39: How to handle schema evolution?**
- Use Schema Registry
- Backward compatibility (new consumers, old data)
- Forward compatibility (old consumers, new data)
- Full compatibility (both)

**Q40: What is log segment?**
- Partition divided into segments (files)
- Default: 1GB or 7 days
- Old segments deleted based on retention
- Active segment: currently being written

**Q41: What is log compaction?**
```properties
cleanup.policy=compact
# Keeps only latest value for each key
# Useful for changelog topics (snapshots)
```

**Q42: Kafka security - authentication?**
- SASL/PLAIN
- SASL/SCRAM
- SSL/TLS
- Kerberos

**Q43: Kafka security - authorization?**
- ACLs (Access Control Lists)
- Per-topic, per-group permissions
- Read, Write, Create, Delete operations

**Q44: Kafka security - encryption?**
- In-transit: SSL/TLS
- At-rest: Encrypt disk volumes
- End-to-end: Application-level encryption

**Q45: Kafka quotas?**
- Limit producer/consumer throughput
- Per-client or per-user
- Protects cluster from abuse

**Q46: What is `max.in.flight.requests.per.connection`?**
- Max unacknowledged requests
- Default: 5
- Set to 1 for strict ordering (if retries enabled)

**Q47: Delivery semantics?**
- **At-most-once:** Can lose messages
- **At-least-once:** Can duplicate messages (default)
- **Exactly-once:** Each message once (EOS)

**Q48: What is consumer session timeout?**
```java
session.timeout.ms=10000  // 10 seconds
// If no heartbeat for 10s, consumer kicked from group
```

**Q49: What is consumer heartbeat?**
- Periodic signal to coordinator
- "I'm alive and processing"
- Separate thread from polling

**Q50: Partition reassignment?**
```bash
# Generate reassignment plan
kafka-reassign-partitions --generate ...

# Execute reassignment
kafka-reassign-partitions --execute ...

# Verify
kafka-reassign-partitions --verify ...
```

**Q51: Leader election?**
- Controller picks new leader from ISR
- Failover automatic
- Usually milliseconds

**Q52: Unclean leader election?**
```properties
unclean.leader.election.enable=false  # Default, safer
# If true, can elect out-of-sync replica (data loss risk)
```

**Q53: Kafka Connect modes?**
- **Standalone:** Single process
- **Distributed:** Multiple workers, fault-tolerant

**Q54: Kafka MirrorMaker?**
- Replicates data between clusters
- Disaster recovery
- Geo-replication

**Q55: Topic deletion?**
```bash
kafka-topics --delete --topic my-topic
# Requires: delete.topic.enable=true
```

**Q56: Dynamic configuration?**
```bash
# Change config without restart
kafka-configs --alter --entity-type topics \
  --entity-name my-topic \
  --add-config retention.ms=86400000
```

**Q57: What is __consumer_offsets topic?**
- Internal topic
- Stores consumer group offsets
- Compacted topic

**Q58: What is __transaction_state topic?**
- Internal topic
- Stores transaction state
- For exactly-once semantics

**Q59: How to increase partitions?**
```bash
# Can only increase, not decrease
kafka-topics --alter --topic my-topic --partitions 12
```

**Q60: Kafka metrics to monitor?**
- UnderReplicatedPartitions (critical!)
- ActiveControllerCount (should be 1)
- OfflinePartitionsCount (should be 0)
- Consumer lag
- Request latency

---

## ADVANCED (Questions 61-100)

### Q61: Design a high-throughput Kafka pipeline

**Answer:**

**Scenario:** Process 1M messages/second with low latency

**Architecture:**
```
Producers (100 instances)
    ↓
Topic: events (50 partitions, RF=3)
    ↓
Consumer Group (50 instances)
```

**Producer Configuration:**
```java
// Throughput optimized
batch.size=65536              // 64KB batches
linger.ms=10                  // Small batency for batching
compression.type=lz4          // Fast compression
buffer.memory=67108864        // 64MB buffer
acks=1                        // Balance durability/speed
max.in.flight.requests=5      // Pipelining
```

**Topic Configuration:**
```properties
# Partitions: Target 20K msgs/sec per partition
# 1M msgs/sec / 20K = 50 partitions

num.partitions=50
replication.factor=3
min.insync.replicas=2
compression.type=lz4
```

**Consumer Configuration:**
```java
fetch.min.bytes=1048576       // 1MB batches
max.poll.records=1000         // Large batches
max.poll.interval.ms=300000   // 5 min for processing
```

**Broker Configuration:**
```properties
# More threads
num.network.threads=8
num.io.threads=16

# Larger buffers
socket.send.buffer.bytes=1048576
socket.receive.buffer.bytes=1048576

# Replication
num.replica.fetchers=4
```

**Monitoring:**
- JMX metrics
- Consumer lag < 1000
- Broker CPU < 70%
- Disk I/O < 80%

---

### Q62: Explain Kafka's storage internals

**Answer:**

**Storage Hierarchy:**
```
Broker
└── Topics
    └── Partitions
        └── Segments (files)
            ├── .log (messages)
            ├── .index (offset → position)
            ├── .timeindex (timestamp → offset)
```

**Segment Files:**
```bash
/data/kafka/events-0/
├── 00000000000000000000.log       # Messages 0-999
├── 00000000000000000000.index
├── 00000000000000001000.log       # Messages 1000-1999
├── 00000000000000001000.index
└── 00000000000000002000.log       # Active segment
```

**Log File (.log):**
```
[Offset][Size][CRC][Magic][Attributes][Timestamp][Key][Value]
```

**Index File (.index):**
```
Offset → Physical Position
1000   → 0
1500   → 52428
2000   → 104856
```

**How Read Works:**
1. Consumer requests offset 1500
2. Broker finds correct segment (based on filename)
3. Looks up offset 1500 in index → position 52428
4. Seeks to position 52428 in .log file
5. Reads messages

**Segment Rolling:**
```properties
# Roll segment when:
log.segment.bytes=1073741824      # 1GB
log.segment.ms=604800000          # 7 days

# Whichever comes first
```

**Compaction:**
```
Before:
Key A: value1, value2, value3
Key B: value1, value2

After compaction:
Key A: value3  ← Only latest
Key B: value2  ← Only latest
```

---

### Q63-Q100: Advanced Topics (condensed)

**Q63: Kafka in Kubernetes (Strimzi)?**
- Kubernetes operator for Kafka
- Manages brokers, topics, users
- Auto-scaling, rolling updates

**Q64: Multi-datacenter Kafka?**
- MirrorMaker 2.0
- Active-active or active-passive
- Geo-replication

**Q65: Kafka vs Pulsar?**
| Feature | Kafka | Pulsar |
|---------|-------|--------|
| Architecture | Monolithic | Layered (BookKeeper) |
| Multi-tenancy | Basic | Native |
| Geo-replication | MirrorMaker | Built-in |
| Maturity | More mature | Newer |

**Q66: Tiered storage?**
- Archive old segments to S3/HDFS
- Reduce broker storage costs
- Confluent feature (or custom)

**Q67: How to reduce consumer lag?**
1. Add consumers (up to # partitions)
2. Optimize processing code
3. Increase partitions
4. Batch database operations
5. Parallelize within consumer

**Q68: Kafka data loss scenarios?**
- acks=0/1 with broker failure
- min.insync.replicas=1
- Unclean leader election
- All ISR down before replication

**Q69: Kafka data duplication scenarios?**
- Retries after timeout (but actually sent)
- Consumer reprocesses after crash (before commit)
- Use idempotent producer + transactions

**Q70: Kafka Connect at scale?**
- Distributed mode
- Multiple workers
- Task parallelism
- Offset management automatic

**Q71: Kafka command-line tools?**
```bash
kafka-topics
kafka-console-producer
kafka-console-consumer
kafka-consumer-groups
kafka-configs
kafka-reassign-partitions
kafka-log-dirs
```

**Q72: Debug slow consumer?**
1. Check lag (kafka-consumer-groups)
2. Profile processing code
3. Check network latency
4. Review rebalancing frequency
5. Check fetch sizes

**Q73: Debug slow producer?**
1. Check broker metrics
2. Review batching settings
3. Check compression overhead
4. Network latency
5. Broker under-resourced?

**Q74: Kafka upgrades best practices?**
1. Rolling upgrade (one broker at a time)
2. Test in staging
3. Backup configs
4. Review release notes
5. Monitor during upgrade

**Q75: Kafka disaster recovery?**
- Multi-datacenter replication
- Regular backups
- Document runbooks
- Test failover procedures

**Q76: Kafka cost optimization?**
1. Tiered storage
2. Compression
3. Retention tuning
4. Right-size brokers
5. Remove unused topics

**Q77: Kafka with Spark Streaming?**
```scala
val df = spark
  .readStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "localhost:9092")
  .option("subscribe", "topic")
  .load()
```

**Q78: Kafka with Flink?**
```java
FlinkKafkaConsumer<String> consumer = new FlinkKafkaConsumer<>(
    "topic",
    new SimpleStringSchema(),
    properties);

stream.addSource(consumer);
```

**Q79: Consumer offset management?**
- Automatic (enable.auto.commit=true)
- Manual sync (commitSync)
- Manual async (commitAsync)
- Store offsets externally

**Q80: Kafka throttling?**
- Quotas per client
- Throttle producer/consumer bandwidth
- Protect cluster from runaway clients

**Q81-Q100: Your Optum Use Cases**

**Q81: How did you optimize Kafka at Optum?**
"Reduced consumer lag from 2M to <50K messages:
- Refactored synchronous DB calls to async
- Scaled consumers from 6 to 12
- Increased batch size 16KB → 64KB
- Enabled lz4 compression
- Result: 40% throughput increase, 60% network reduction"

**Q82: Kafka monitoring at Optum?**
"Set up Grafana dashboards with:
- Consumer lag per partition
- Broker resource usage
- ISR shrinks
- Alerts on lag >100K"

**Q83: Kafka security at Optum?**
"Implemented:
- SSL/TLS encryption
- SASL authentication
- ACLs for topic-level permissions
- Audit logging"

**Q84: Exactly-once at Optum?**
"Used for financial transactions:
- Idempotent producers
- Transactional writes
- read_committed consumers
- Prevented duplicate billing"

**Q85-Q100: Scenario questions**
- Design real-time fraud detection
- Handle schema evolution
- Multi-region setup
- Kafka for microservices
- Event sourcing with Kafka
- CDC pipelines
- Log aggregation
- Metrics collection
- IoT data ingestion
- Kafka + data lake integration
- Kafka + warehouse (Snowflake)
- Backpressure handling
- Dead letter queues
- Message ordering across partitions
- Large message handling
- Kafka testing strategies

---

## Summary

**Core Topics Covered:**
- ✅ Architecture (Brokers, Topics, Partitions)
- ✅ Producers (Batching, Compression, Acks)
- ✅ Consumers (Groups, Offsets, Lag)
- ✅ Performance (Throughput, Latency)
- ✅ Reliability (Replication, EOS)
- ✅ Operations (Monitoring, Tuning)
- ✅ Your Optum Experience (40% optimization)

**Study Plan:**
- Week 1: Q1-40 (Fundamentals + Intermediate)
- Week 2: Q41-70 (Advanced concepts)
- Week 3: Q71-100 (Scenarios + Your examples)

Good luck! 🚀

---

**Q74: How to tune Kafka producer performance?**

Adjust batch.size, linger.ms, compression.type, acks, buffer.memory.

```python
producer = KafkaProducer(
    batch_size=32768,  # Larger batches
    linger_ms=20,  # Wait for batch to fill
    compression_type='lz4',  # Compress
    acks='1'  # Leader ack only (faster)
)
```

---

**Q75: Explain Kafka MirrorMaker for replication.**

Replicates topics between clusters (DR, multi-region).

```bash
# MirrorMaker 2.0
kafka-mirror-maker2.sh --config mm2.properties
```

Use for disaster recovery, geo-replication.

---

**Q76: How to handle Kafka consumer lag?**

Monitor lag with `kafka-consumer-groups`, scale consumers, optimize processing.

```bash
# Check lag
kafka-consumer-groups --bootstrap-server localhost:9092 \
  --group my-group --describe

# Solutions:
# 1. Add more consumers (up to partition count)
# 2. Increase fetch.min.bytes
# 3. Optimize consumer logic
# 4. Use batch processing
```

---

**Q77: What are Kafka Connect transformations?**

Modify data in-flight (Single Message Transforms - SMTs).

```json
{
  "transforms": "maskPII",
  "transforms.maskPII.type": "org.apache.kafka.connect.transforms.MaskField$Value",
  "transforms.maskPII.fields": "ssn,dob"
}
```

Common SMTs: Cast, Filter, Flatten, InsertField, ReplaceField.

---

**Q78: Explain Kafka producer idempotence.**

Exactly-once semantics by preventing duplicate writes.

```python
producer = KafkaProducer(
    enable_idempotence=True,  # Prevents duplicates
    acks='all',
    max_in_flight_requests_per_connection=5
)
```

Kafka assigns sequence numbers to detect duplicates.

---

**Q79: How to implement Kafka message ordering?**

Within partition only. Use same partition key for related messages.

```python
# All messages for same member_id go to same partition
producer.send('claims', key=member_id.encode(), value=claim_data)
```

Cannot guarantee order across partitions.

---

**Q80: What is Kafka log compaction?**

Retains only latest value per key (useful for changelog topics).

```bash
# Configure topic
kafka-configs --alter --entity-type topics --entity-name user-profiles \
  --add-config cleanup.policy=compact
```

---

**Q81: Explain Kafka consumer group rebalancing.**

Partitions redistributed when consumers join/leave.

**Strategies:**
- **Range:** Divide partitions by range (default)
- **Round Robin:** Distribute evenly
- **Sticky:** Minimize partition movement

```python
consumer = KafkaConsumer(
    partition_assignment_strategy=[StickyPartitionAssignor]
)
```

---

**Q82: How to monitor Kafka health?**

Use JMX metrics, Prometheus, Grafana.

**Key metrics:**
- Under-replicated partitions
- Consumer lag
- Request rate/latency
- Disk usage
- Network throughput

---

**Q83: What are Kafka quotas?**

Rate limits for producers/consumers.

```bash
# Limit producer to 1MB/sec
kafka-configs --alter --entity-type clients --entity-name producer-1 \
  --add-config 'producer_byte_rate=1048576'
```

Prevent noisy neighbors.

---

**Q84: Explain Kafka exactly-once semantics (EOS).**

Combination of idempotent producer + transactional writes.

```python
producer = KafkaProducer(
    enable_idempotence=True,
    transactional_id='my-transactional-id'
)

producer.init_transactions()
try:
    producer.begin_transaction()
    producer.send('topic', value)
    producer.commit_transaction()
except:
    producer.abort_transaction()
```

---

**Q85: How to handle large Kafka messages?**

1. Increase `max.request.size` (producer) and `message.max.bytes` (broker)
2. Store in external storage (S3), send reference in Kafka
3. Compress messages
4. Split into multiple messages

```python
# Reference pattern
reference = upload_to_s3(large_data)
producer.send('topic', {'ref': reference})
```

---

**Q86: What is Kafka controller?**

One broker elected as controller, manages partition leaders, cluster metadata.

```bash
# Check controller
zookeeper-shell localhost:2181 get /controller
```

---

**Q87: Explain Kafka broker replication.**

Data replicated across brokers for fault tolerance.

```bash
# Create topic with 3 replicas
kafka-topics --create --topic claims \
  --partitions 10 --replication-factor 3
```

Leader handles reads/writes, followers replicate.

---

**Q88: How to implement Kafka dead letter queue?**

Send failed messages to separate topic.

```python
try:
    process(message)
    consumer.commit()
except Exception as e:
    producer.send('dlq-topic', value=message.value, headers=[('error', str(e))])
    consumer.commit()  # Don't retry indefinitely
```

---

**Q89: What are Kafka client configurations for reliability?**

```python
# Producer
producer = KafkaProducer(
    acks='all',  # All replicas ack
    retries=10,
    max_in_flight_requests_per_connection=1,  # Maintain order
    enable_idempotence=True
)

# Consumer
consumer = KafkaConsumer(
    enable_auto_commit=False,  # Manual commit
    isolation_level='read_committed'  # Only read committed transactions
)
```

---

**Q90: Explain Kafka partition reassignment.**

Move partitions between brokers (rebalancing load).

```bash
# Generate reassignment plan
kafka-reassign-partitions --bootstrap-server localhost:9092 \
  --topics-to-move-json-file topics.json \
  --broker-list "0,1,2" \
  --generate

# Execute
kafka-reassign-partitions --execute --reassignment-json-file plan.json
```

---

**Q91: How to secure Kafka?**

SSL/TLS encryption, SASL authentication, ACLs for authorization.

```properties
# server.properties
listeners=SSL://localhost:9093
security.inter.broker.protocol=SSL
ssl.keystore.location=/var/ssl/kafka.keystore.jks
ssl.keystore.password=secret
ssl.key.password=secret

# ACLs
kafka-acls --add --allow-principal User:alice \
  --operation Read --topic claims
```

---

**Q92: What is Kafka Streams exactly-once processing?**

Use `processing.guarantee=exactly_once_v2`.

```java
Properties props = new Properties();
props.put(StreamsConfig.PROCESSING_GUARANTEE_CONFIG, StreamsConfig.EXACTLY_ONCE_V2);

StreamsBuilder builder = new StreamsBuilder();
// Build topology
KafkaStreams streams = new KafkaStreams(builder.build(), props);
```

---

**Q93: Explain Kafka retention policies.**

Time-based or size-based retention.

```bash
# Retain 7 days or 1GB per partition
kafka-configs --alter --entity-type topics --entity-name claims \
  --add-config retention.ms=604800000,retention.bytes=1073741824
```

Or use log compaction for keeping latest per key.

---

**Q94: How to optimize Kafka Streams performance?**

Tune `num.stream.threads`, `cache.max.bytes.buffering`, compression, state store configs.

```java
props.put(StreamsConfig.NUM_STREAM_THREADS_CONFIG, 4);
props.put(StreamsConfig.CACHE_MAX_BYTES_BUFFERING_CONFIG, 10 * 1024 * 1024);
props.put(StreamsConfig.COMMIT_INTERVAL_MS_CONFIG, 1000);
```

---

**Q95: What are Kafka producer callbacks?**

Async notifications on send completion.

```python
def on_success(metadata):
    print(f"Sent to {metadata.topic} partition {metadata.partition}")

def on_error(error):
    print(f"Error: {error}")

producer.send('topic', value).add_callback(on_success).add_errback(on_error)
```

---

**Q96: Explain Kafka broker configurations for performance.**

Adjust `num.network.threads`, `num.io.threads`, `socket.send.buffer.bytes`.

```properties
# server.properties
num.network.threads=8
num.io.threads=16
socket.send.buffer.bytes=1048576
socket.receive.buffer.bytes=1048576
```

---

**Q97: How to implement Kafka message deduplication?**

1. Enable idempotent producer (Kafka-level)
2. Consumer-side deduplication using message ID

```python
processed_ids = set()

for message in consumer:
    msg_id = message.key.decode()
    if msg_id not in processed_ids:
        process(message)
        processed_ids.add(msg_id)
```

---

**Q98: What is Kafka Zookeeper (and KRaft)?**

**Zookeeper:** Manages cluster metadata (being replaced).
**KRaft:** Kafka Raft, removes Zookeeper dependency (Kafka 3.0+).

```properties
# KRaft mode
process.roles=broker,controller
node.id=1
controller.quorum.voters=1@localhost:9093
```

---

**Q99: Explain Kafka multi-datacenter replication strategies.**

1. **Active-Active:** MirrorMaker 2.0 bidirectional replication
2. **Active-Passive:** One-way replication for DR
3. **Aggregation:** Multiple DCs → central cluster

Use MirrorMaker 2.0, Confluent Replicator, or custom consumers/producers.

---

**Q100: Tell me about your Kafka implementation at Optum.**

**Real-time claims event streaming:**
- 10M+ events/day across 50+ topics
- 3-node Kafka cluster (AWS MSK)
- Partitions: 20-50 per topic for parallelism
- Replication factor: 3 (high availability)
- Consumers: Spark Streaming, Kafka Streams, Python microservices
- Use cases: Fraud detection, real-time dashboards, audit logging

**Key challenges solved:**
- Consumer lag: Scaled from 5 to 20 consumers per group
- Message ordering: Used member_id as partition key
- Exactly-once: Enabled idempotent producers + transactional writes
- Monitoring: Kafka Exporter → Prometheus → Grafana (track lag, throughput)

**Performance:**
- 99.9% uptime
- <100ms p95 latency
- Auto-scaling consumers based on lag

---


**Q99: Explain Kafka monitoring key metrics.**

Track: consumer lag, under-replicated partitions, request rate/latency, broker disk/CPU/network usage, producer/consumer throughput. Use Kafka Exporter → Prometheus → Grafana. Alert on lag >1M messages, under-replicated >0, high latency >500ms.

**Q100: Kafka performance tuning checklist.**

Producer: batch.size=32KB, linger.ms=20, compression=lz4, acks=1. Consumer: fetch.min.bytes=1MB, max.poll.records=500. Broker: num.network.threads=8, num.io.threads=16, log.segment.bytes=1GB. Monitor and adjust based on load.

