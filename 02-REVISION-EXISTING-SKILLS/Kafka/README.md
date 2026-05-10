# Apache Kafka - Distributed Event Streaming Platform

**Master Kafka for real-time data engineering interviews**

---

## 📚 What's in This Folder

### **1. 100-QUESTIONS.md**
Comprehensive Q&A covering:
- Kafka fundamentals (topics, partitions, brokers)
- Producers and consumers
- Consumer groups and rebalancing
- Kafka Connect and Streams
- Exactly-once semantics
- Performance tuning
- Monitoring and operations
- Real-world architectures
- Optum use cases

### **2. CHEATSHEET.md**
Quick reference with:
- One-line concept definitions
- Common commands
- Configuration parameters
- Architecture diagrams
- Performance tuning tips
- Best practices

---

## 🎯 What You'll Learn

### **Core Concepts**
- **Topics:** Logical streams of events
- **Partitions:** Parallelism and ordering units
- **Brokers:** Kafka servers that store data
- **Producers:** Publish messages to topics
- **Consumers:** Subscribe and read messages
- **Consumer Groups:** Scalable message processing
- **Offsets:** Position in partition
- **Replication:** Fault tolerance via replicas

### **Advanced Features**
- **Kafka Connect:** Data integration framework
- **Kafka Streams:** Stream processing library
- **Schema Registry:** Schema management for Avro/JSON
- **Exactly-Once Semantics:** Guaranteed message delivery
- **Transactions:** Atomic writes across partitions
- **Compaction:** Log cleanup strategy
- **Quotas:** Rate limiting

### **Production Topics**
- Cluster sizing and capacity planning
- Monitoring (metrics, lag, throughput)
- Security (SSL, SASL, ACLs)
- Multi-datacenter replication
- Disaster recovery
- Performance optimization
- Common failure scenarios

---

## 💼 Why Kafka Matters

### **Industry Adoption**
- Used by: LinkedIn, Netflix, Uber, Airbnb, Spotify, Twitter
- 80%+ of Fortune 100 companies use Kafka
- De facto standard for event streaming
- Core component of modern data architectures

### **Key Use Cases**
- **Real-time ETL:** Stream data between systems
- **Event-driven architecture:** Microservices communication
- **Log aggregation:** Centralize application logs
- **Stream processing:** Real-time analytics and transformations
- **Messaging:** Decouple systems with pub/sub
- **CDC (Change Data Capture):** Database replication
- **Activity tracking:** User behavior analytics

### **Interview Focus**
1. Architecture understanding (brokers, partitions, replication)
2. Consumer group mechanics and rebalancing
3. Delivery semantics (at-most-once, at-least-once, exactly-once)
4. Performance tuning and scaling
5. Production operations (monitoring, troubleshooting)

---

## 🚀 Quick Start Guide

### **1. Read Cheatsheet (15 minutes)**
Get familiar with core terminology and concepts from `CHEATSHEET.md`.

### **2. Study 100 Questions (4-5 hours)**
- **Q1-Q25:** Fundamentals (topics, partitions, producers, consumers)
- **Q26-Q50:** Advanced (consumer groups, offsets, replication)
- **Q51-Q75:** Kafka Streams, Connect, Schema Registry
- **Q76-Q100:** Production operations and real-world scenarios

### **3. Hands-On Practice**
```bash
# Download Kafka
wget https://downloads.apache.org/kafka/3.6.0/kafka_2.13-3.6.0.tgz
tar -xzf kafka_2.13-3.6.0.tgz
cd kafka_2.13-3.6.0

# Start Zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

# Start Kafka broker
bin/kafka-server-start.sh config/server.properties

# Create topic
bin/kafka-topics.sh --create --topic test --bootstrap-server localhost:9092 \
  --partitions 3 --replication-factor 1

# Produce messages
bin/kafka-console-producer.sh --topic test --bootstrap-server localhost:9092

# Consume messages
bin/kafka-console-consumer.sh --topic test --bootstrap-server localhost:9092 \
  --from-beginning
```

---

## 📖 Topic Coverage

### **Fundamentals (Q1-Q25)**

**Kafka Architecture:**
```
┌─────────────────────────────────────────┐
│          Kafka Cluster                  │
│  ┌────────┐  ┌────────┐  ┌────────┐   │
│  │Broker 1│  │Broker 2│  │Broker 3│   │
│  │ Topic A│  │ Topic A│  │ Topic A│   │
│  │  P0(L) │  │  P1(L) │  │  P2(L) │   │
│  │  P1(F) │  │  P2(F) │  │  P0(F) │   │
│  └────────┘  └────────┘  └────────┘   │
└─────────────────────────────────────────┘
     ↑                            ↓
 Producers                    Consumers
```
- L = Leader, F = Follower

**Key Concepts:**
- Topic: Category/feed name
- Partition: Ordered, immutable sequence of messages
- Offset: Unique ID for each message within partition
- Broker: Single Kafka server
- Cluster: Group of brokers

### **Producer/Consumer (Q26-Q50)**

**Producer:**
```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("acks", "all");  // Wait for all replicas
props.put("retries", 3);

Producer<String, String> producer = new KafkaProducer<>(props);
producer.send(new ProducerRecord<>("topic", "key", "value"));
```

**Consumer:**
```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("group.id", "my-group");
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("enable.auto.commit", "false");  // Manual commit

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("topic"));

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        processRecord(record);
    }
    consumer.commitSync();  // Commit offsets
}
```

### **Consumer Groups (Q51-Q75)**

**Partitioning Strategy:**
- Each partition assigned to exactly one consumer in group
- If consumers < partitions: some consumers get multiple partitions
- If consumers > partitions: some consumers are idle
- Rebalancing occurs when consumers join/leave

**Example:**
```
Topic: orders (4 partitions)
Consumer Group: order-processors (2 consumers)

Consumer 1: Partitions 0, 1
Consumer 2: Partitions 2, 3
```

### **Production Patterns (Q76-Q100)**

**Delivery Semantics:**
- **At-most-once:** May lose messages (acks=0)
- **At-least-once:** May duplicate messages (acks=1, retries)
- **Exactly-once:** No loss, no duplicates (transactions, idempotence)

**Performance Tuning:**
- Producer: batch.size, linger.ms, compression.type
- Consumer: fetch.min.bytes, max.poll.records
- Broker: num.network.threads, num.io.threads

---

## 🎓 Interview Preparation

### **Week 1: Fundamentals**
- Study Q1-Q30
- Understand broker, topic, partition, offset
- Practice producer/consumer basics
- Set up local Kafka cluster

### **Week 2: Advanced**
- Study Q31-Q60
- Deep dive on consumer groups
- Learn rebalancing mechanics
- Understand offset management

### **Week 3: Production**
- Study Q61-Q100
- Performance tuning
- Monitoring and alerting
- Disaster recovery scenarios

### **Interview Day**
- Review CHEATSHEET.md
- Be ready to whiteboard architecture
- Know exact-once semantics cold
- Prepare real-world stories

---

## 💡 Common Interview Questions

### **Conceptual**

**Q: "What is Kafka and why use it?"**
- Distributed event streaming platform
- High throughput (millions msgs/sec)
- Fault-tolerant (replication)
- Scalable (horizontal scaling)
- Durable (persistent storage)
- Real-time processing

**Q: "Explain Kafka architecture"**
- **Brokers:** Store data
- **Topics:** Logical categories
- **Partitions:** Parallelism units
- **Producers:** Write data
- **Consumers:** Read data
- **ZooKeeper:** Cluster coordination (being phased out)

**Q: "How does Kafka guarantee ordering?"**
- Ordering within partition (not across partitions)
- Messages with same key go to same partition
- Consumer reads partitions in order

**Q: "Explain consumer groups"**
- Group of consumers sharing topic consumption
- Each partition assigned to one consumer in group
- Enables parallel processing
- Rebalancing on consumer changes

### **Scenario Questions**

**Q: "Design real-time order processing system"**
```
Orders Service → Kafka Topic "orders" →
  - Consumer Group 1: Inventory Update
  - Consumer Group 2: Email Notification
  - Consumer Group 3: Analytics Aggregation
```

**Q: "Handle consumer lag?"**
- Monitor lag metrics (records-lag-max)
- Scale consumers (add to group)
- Optimize consumer processing (reduce per-message time)
- Increase partitions (if needed)
- Use Kafka Streams for stateful processing

**Q: "Ensure exactly-once processing?"**
- Enable producer idempotence
- Use transactions for atomic writes
- Use transactional consumer (isolation.level=read_committed)
- Implement idempotent consumer logic

---

## 🔗 Resources

### **Official Docs**
- [Kafka Documentation](https://kafka.apache.org/documentation/)
- [Kafka Quickstart](https://kafka.apache.org/quickstart)
- [Kafka Streams](https://kafka.apache.org/documentation/streams/)

### **Learning**
- [Confluent Tutorials](https://developer.confluent.io/)
- [Kafka: The Definitive Guide](https://www.oreilly.com/library/view/kafka-the-definitive/9781492043072/) (book)
- [Stephane Maarek's Kafka Course](https://www.udemy.com/course/apache-kafka/)

### **Tools**
- [Kafka UI](https://github.com/provectus/kafka-ui) - Web UI for Kafka
- [kafka-python](https://kafka-python.readthedocs.io/) - Python client
- [Conduktor](https://www.conduktor.io/) - Kafka desktop client

---

## 🎯 Key Takeaways

### **Must Know**
✅ Topics, partitions, offsets, brokers
✅ Producer acknowledgments (acks)
✅ Consumer groups and rebalancing
✅ Delivery semantics (at-most, at-least, exactly-once)
✅ Replication and ISR (in-sync replicas)

### **Production Experience**
✅ Monitoring lag and throughput
✅ Performance tuning (batching, compression)
✅ Handling rebalances gracefully
✅ Schema evolution (Avro, Schema Registry)
✅ Disaster recovery strategies

### **Red Flags**
❌ Not understanding partition key importance
❌ Confusing topic-level vs partition-level ordering
❌ Not knowing how consumer groups work
❌ Ignoring monitoring and alerting
❌ Not planning for failure scenarios

---

## 📝 Quick Commands Reference

```bash
# Topic management
kafka-topics.sh --create --topic my-topic --bootstrap-server localhost:9092 \
  --partitions 3 --replication-factor 2

kafka-topics.sh --list --bootstrap-server localhost:9092

kafka-topics.sh --describe --topic my-topic --bootstrap-server localhost:9092

# Consumer groups
kafka-consumer-groups.sh --list --bootstrap-server localhost:9092

kafka-consumer-groups.sh --describe --group my-group --bootstrap-server localhost:9092

# Check lag
kafka-consumer-groups.sh --describe --group my-group --bootstrap-server localhost:9092 \
  | grep LAG

# Produce/Consume
kafka-console-producer.sh --topic my-topic --bootstrap-server localhost:9092

kafka-console-consumer.sh --topic my-topic --bootstrap-server localhost:9092 \
  --from-beginning --group my-group
```

---

**Master Kafka and unlock real-time data engineering! 🚀**

*For detailed answers, see 100-QUESTIONS.md*
*For quick review, see CHEATSHEET.md*
