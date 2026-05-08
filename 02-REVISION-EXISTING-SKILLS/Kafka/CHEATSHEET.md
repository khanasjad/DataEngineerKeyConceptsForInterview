# Kafka Cheatsheet - Quick Reference

## Core Concepts
- **Kafka**: Distributed event streaming platform for real-time data pipelines
- **Topic**: Category/feed name where records are published
- **Partition**: Ordered, immutable sequence of records within topic
- **Broker**: Kafka server that stores data
- **Producer**: Publishes messages to topics
- **Consumer**: Subscribes to topics and processes messages
- **Consumer Group**: Group of consumers sharing workload (each partition read by one consumer)

## Key Properties
- **Offset**: Unique sequential ID for each message in partition
- **Replication factor**: Number of copies of each partition across brokers
- **Leader**: Broker handling reads/writes for partition
- **Follower**: Replica broker syncing from leader (ISR: In-Sync Replicas)
- **Retention**: How long messages are kept (time or size-based)

## Producer
- **acks=0**: No acknowledgment (fire-and-forget, fast, data loss risk)
- **acks=1**: Leader acknowledgment (balanced)
- **acks=all**: All ISRs acknowledge (safest, slowest)
- **Partition key**: Determines which partition message goes to (same key → same partition)
- **Idempotent producer**: Prevents duplicates (`enable.idempotence=true`)
- **Compression**: Reduce size (gzip, snappy, lz4, zstd)

## Consumer
- **Consumer group**: Parallel processing, each partition assigned to one consumer
- **Offset commit**: Save progress (auto or manual)
- **Rebalancing**: Redistribute partitions when consumer joins/leaves
- **Partition assignment strategies**: Range, Round-robin, Sticky, Cooperative Sticky
- **Poll interval**: How often consumer fetches messages
- **Max poll records**: Max messages per poll

## Kafka Streams
- **Stream processing**: Real-time transformation of Kafka data
- **KStream**: Unbounded stream of records
- **KTable**: Changelog stream (latest value per key)
- **GlobalKTable**: Replicated table on all instances
- **Windowing**: Time-based grouping (tumbling, hopping, sliding, session)
- **Join**: Combine streams/tables (inner, left, outer)

## Kafka Connect
- **Source connector**: Import data into Kafka (JDBC, S3, MongoDB, etc.)
- **Sink connector**: Export data from Kafka (HDFS, Elasticsearch, Snowflake, etc.)
- **Connector config**: JSON configuration for data flow
- **Transforms (SMT)**: Modify data in-flight (mask, filter, flatten)
- **Converters**: Serialize/deserialize (JSON, Avro, Protobuf)

## Schema Registry
- **Schema management**: Store and retrieve Avro/Protobuf/JSON schemas
- **Schema versioning**: Track schema evolution
- **Compatibility modes**: Backward, Forward, Full, None
- **Schema ID**: Embedded in message for efficient serialization

## Performance Tuning
- **Producer**: Increase batch.size, linger.ms, compression, acks=1 (not all)
- **Consumer**: Increase fetch.min.bytes, max.poll.records, parallel processing
- **Broker**: Tune num.network.threads, num.io.threads, log.segment.bytes
- **Partitioning**: More partitions = more parallelism (but more overhead)

## Reliability & Durability
- **Replication**: min.insync.replicas=2 (at least 2 replicas)
- **Unclean leader election**: Disabled (don't elect out-of-sync replica)
- **Producer retries**: Enable with idempotence
- **Consumer offset management**: Commit after processing
- **Exactly-once semantics**: Idempotent producer + transactions

## Monitoring
- **Under-replicated partitions**: Partitions missing ISRs (critical alert)
- **Consumer lag**: Messages behind (offset difference)
- **Request latency**: Time to process requests
- **Throughput**: Messages/bytes per second
- **Disk usage**: Space used by log segments

## Kafka CLI
```bash
# Topics
kafka-topics --create --topic my-topic --partitions 3 --replication-factor 2
kafka-topics --list --bootstrap-server localhost:9092
kafka-topics --describe --topic my-topic

# Producer/Consumer
kafka-console-producer --topic my-topic --bootstrap-server localhost:9092
kafka-console-consumer --topic my-topic --from-beginning

# Consumer groups
kafka-consumer-groups --list
kafka-consumer-groups --describe --group my-group
kafka-consumer-groups --reset-offsets --to-earliest --execute
```

## Best Practices
✅ Use partition keys for ordering | ✅ Set appropriate replication factor (3 for prod) | ✅ Enable compression | ✅ Monitor consumer lag | ✅ Use Schema Registry | ✅ Manual offset commit for critical data | ✅ Set retention policies | ✅ Partition by access patterns | ✅ Dead letter queue for failures
