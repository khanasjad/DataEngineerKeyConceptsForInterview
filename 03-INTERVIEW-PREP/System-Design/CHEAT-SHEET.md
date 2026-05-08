# System Design Interview Cheat Sheet

**Use this for quick reference before interviews**

---

## Interview Structure (35-45 minutes)

**1. Clarify Requirements (5-10 min)**
- Functional requirements (what system does)
- Non-functional requirements (scale, performance, availability)
- Constraints (time, budget, tech stack)
- Scale estimates (users, QPS, storage)

**2. High-Level Design (10-15 min)**
- Draw box diagram
- Identify main components
- Show data flow
- Explain briefly

**3. Deep Dive (15-20 min)**
- Drill into 2-3 components
- Discuss trade-offs
- Address bottlenecks
- Propose optimizations

**4. Wrap Up (5 min)**
- Address edge cases
- Monitoring & alerting
- Future improvements
- Questions

---

## Common Data Engineering System Designs

### 1. Real-Time Analytics Pipeline

**Example:** "Design a system to show real-time dashboards for e-commerce"

**Architecture:**
```
Events → Kafka → Stream Processing → Storage → Serving Layer → Dashboard
         ↓           (Flink/Spark)      (Delta)    (Redis)
      (Partitioned)
```

**Key Components:**
- **Ingestion:** Kafka (partitioned by user_id or event_type)
- **Processing:** Spark Streaming or Flink
  - Windowed aggregations (tumbling, sliding)
  - Stateful processing
- **Storage:**
  - Hot: Redis (recent data, fast access)
  - Warm: Delta Lake (queryable, time-series)
  - Cold: S3 (archive)
- **Serving:** REST API or WebSocket for real-time updates

**Scale Considerations:**
- Kafka: 100K events/sec → 20-30 partitions
- Flink: Horizontal scaling, checkpointing
- Redis: Cluster mode for high throughput
- Monitoring: Lag, latency, error rate

**Trade-offs:**
- Latency vs accuracy (micro-batching vs true streaming)
- Cost vs performance (caching strategy)
- Complexity vs features (how real-time is "real-time"?)

---

### 2. Batch ETL Data Warehouse

**Example:** "Design a data warehouse for an e-commerce company"

**Architecture:**
```
Sources → Ingestion → Data Lake → Transform → Data Warehouse → BI Tools
(DBs,APIs)  (Airflow)   (S3/ADLS)   (Spark)     (Snowflake)    (Tableau)
                                      (dbt)
```

**Key Components:**
- **Sources:** Multiple databases, APIs, files
- **Ingestion:**
  - Airflow DAGs for orchestration
  - CDC for databases (Debezium)
  - API connectors
  - File sensors
- **Data Lake:** S3 / ADLS (Bronze/Silver/Gold layers)
- **Transform:**
  - Spark for heavy transformations
  - dbt for SQL-based transformations
- **Data Warehouse:** Snowflake, Databricks SQL, Redshift
- **Serving:** BI tools, APIs, ML models

**Data Modeling:**
- Star schema: Fact tables + Dimension tables
- Slowly Changing Dimensions (SCD Type 2 for history)
- Partitioning by date for performance

**Best Practices:**
- ELT over ETL (transform in warehouse)
- Incremental loading (not full refresh)
- Data quality checks (Great Expectations)
- Idempotent pipelines (rerunnable)

---

### 3. ML Feature Pipeline

**Example:** "Design a feature pipeline for fraud detection"

**Architecture:**
```
Raw Data → Feature Engineering → Feature Store → ML Model
           (Batch: Spark)         (Feast)         (Serving API)
           (Streaming: Flink)
```

**Key Components:**
- **Batch Features:**
  - Spark jobs (daily/hourly)
  - Aggregate historical data
  - Store in offline store (S3/Delta)
- **Real-Time Features:**
  - Kafka → Flink → Redis
  - Recent activity (last 1h, 24h)
- **Feature Store (Feast):**
  - Offline: Training data
  - Online: Low-latency serving (<10ms)
  - Feature registry (definitions, lineage)
- **ML Model:**
  - Retrieves features
  - Makes prediction
  - Logs for monitoring

**Critical Points:**
- **Consistency:** Same logic for batch and streaming
- **Low Latency:** <100ms for fraud detection
- **Monitoring:** Feature drift, data quality
- **Versioning:** Feature definitions versioned

**Example Features:**
- Batch: total_transactions_30d, avg_amount_90d
- Real-time: transactions_last_1h, location_changes

---

### 4. Log Aggregation System

**Example:** "Design a system like Splunk or Datadog for log aggregation"

**Architecture:**
```
Apps → Log Agents → Kafka → Processing → Storage → Query/Viz
       (Filebeat)            (Logstash)  (ES)      (Kibana)
```

**Key Components:**
- **Collection:** Filebeat, Fluentd (on each server)
- **Ingestion:** Kafka (buffer, decoupling)
- **Processing:** Logstash or custom parser
  - Parse logs (regex, grok)
  - Enrich (add metadata)
  - Filter (drop noise)
- **Storage:** Elasticsearch (full-text search, aggregations)
- **Query:** Kibana dashboards, alerts

**Scale:**
- 1M logs/sec → Kafka cluster (30+ partitions)
- ES: Time-based indices, ILM policy
- Retention: Hot (7d) → Warm (30d) → Cold (S3)

**Optimizations:**
- Sampling (not all logs needed)
- Compression
- Smart indexing (not all fields)

---

### 5. Streaming Data Pipeline

**Example:** "Design Uber's surge pricing system"

**Architecture:**
```
Driver/Rider → Kafka → Flink → Redis → Pricing API
 Events              (Process)  (Cache)
```

**Key Components:**
- **Event Stream:** Kafka topics
  - driver_location_updates
  - ride_requests
- **Stream Processing (Flink):**
  - Window by geohash (geographic area)
  - Count drivers and requests per area
  - Compute supply/demand ratio
  - Calculate surge multiplier
- **State Store:** Redis (current surge per area)
- **API:** GET /pricing/{lat}/{lon}

**Algorithms:**
- Geohashing for spatial indexing
- Tumbling window (5-minute windows)
- Surge formula: f(demand, supply, time_of_day)

**Scale:**
- Millions of location updates/sec
- Sub-second latency requirement
- Global deployment (multiple regions)

---

## Common Components & Technologies

### Ingestion
- **Kafka:** Distributed streaming, high throughput
- **Kinesis:** AWS-native streaming
- **Pub/Sub:** GCP streaming
- **Airflow:** Batch orchestration
- **Debezium:** CDC from databases

### Processing
- **Spark:** Batch and streaming, mature
- **Flink:** True streaming, stateful
- **dbt:** SQL transformations
- **Pandas/Python:** Small-scale processing

### Storage
- **S3/ADLS:** Object storage, data lake
- **Delta Lake:** ACID on data lake
- **Snowflake:** Cloud data warehouse
- **Redshift:** AWS data warehouse
- **BigQuery:** GCP data warehouse
- **Elasticsearch:** Full-text search, logs
- **Redis:** Caching, low-latency
- **PostgreSQL:** Transactional DB

### Serving
- **REST API:** General purpose
- **GraphQL:** Flexible queries
- **gRPC:** High performance
- **WebSocket:** Real-time updates

### Monitoring
- **Prometheus:** Metrics collection
- **Grafana:** Visualization
- **ELK:** Logging (Elasticsearch, Logstash, Kibana)
- **Datadog:** All-in-one observability

---

## Capacity Estimation

### Back-of-Envelope Calculations

**QPS (Queries Per Second):**
- 100M daily active users
- Each user 10 requests/day
- 100M * 10 / 86400 sec ≈ 12K QPS average
- Peak (3x average) ≈ 36K QPS

**Storage:**
- 1M transactions/day
- 1KB per transaction
- 1M * 1KB = 1GB/day
- Annual: 365GB ≈ 0.5TB
- 5 years: 2.5TB

**Bandwidth:**
- 1KB per request
- 12K QPS
- 12K * 1KB = 12 MB/sec

**Memory (Caching):**
- 20% of requests are hot data
- Cache 100K items * 1KB = 100MB

**Partitioning:**
- 100K QPS
- Each partition handles 4K QPS
- Need 25 partitions

### Rule of Thumb
- **1K QPS:** Single server
- **10K QPS:** Load balancer + multiple servers
- **100K QPS:** Distributed system, caching
- **1M+ QPS:** CDN, global distribution

---

## Trade-Offs Discussion

**Always discuss trade-offs:**

**CAP Theorem:**
- Consistency, Availability, Partition Tolerance (pick 2)
- Most systems: AP (eventual consistency)
- Financial: CP (strong consistency)

**Latency vs Throughput:**
- Lower latency → smaller batches → lower throughput
- Higher throughput → larger batches → higher latency

**Cost vs Performance:**
- Caching → lower latency, higher cost
- Compression → lower storage, higher CPU

**Complexity vs Features:**
- More features → more complex → harder to maintain
- Simple → limited features → easier to operate

**Consistency vs Availability:**
- Strong consistency → may reject writes during partition
- Eventual consistency → always available, may serve stale data

---

## Interview Do's and Don'ts

### DO:
✅ Ask clarifying questions
✅ State assumptions
✅ Start with high-level design
✅ Draw diagrams
✅ Think out loud
✅ Discuss trade-offs
✅ Consider scale and bottlenecks
✅ Mention monitoring and alerting
✅ Be open to feedback
✅ Show enthusiasm

### DON'T:
❌ Jump to details immediately
❌ Design in silence
❌ Ignore scale requirements
❌ Present only one solution
❌ Forget about failures
❌ Ignore operational concerns
❌ Be defensive about your design
❌ Use buzzwords without understanding

---

## Quick Reference: Your Optum Projects

### RQNS Platform
**What it is:** Real-time and batch data platform for healthcare analytics

**Architecture:**
- Kafka for event streaming (member events)
- Spark for processing
- Databricks + Delta Lake for storage
- Azure infrastructure (VNet, NSGs, AKS)
- Airflow for orchestration

**Scale:** 5M+ events/day, 5TB storage

**Your role:** Architected end-to-end, optimized performance, implemented security

**Use for:** Any real-time pipeline, healthcare, Azure infrastructure questions

### Member Microservices
**What it is:** Event-driven microservices for member data

**Architecture:**
- Kafka for event-driven communication
- Spring Boot microservices
- MongoDB for low-latency access
- Spark for data transformation

**Your role:** Designed event-driven architecture, built Spark jobs

**Use for:** Microservices, event-driven, low-latency questions

---

## Before Interview Checklist

**5 minutes before:**
- [ ] Whiteboard/paper ready
- [ ] Water nearby
- [ ] Quiet environment
- [ ] Good internet connection
- [ ] Reviewed company research
- [ ] Confident mindset

**Remember:**
- Clarify before designing
- Think out loud
- Draw diagrams
- Discuss trade-offs
- You're interviewing them too!

**Good luck! 🚀**
