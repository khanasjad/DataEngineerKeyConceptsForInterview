# Chapter 6: Pipeline Architecture for Data Engineering

**Master data pipeline design, Spark optimization, and scalable architectures**

---

## Table of Contents

1. [Introduction to Data Pipelines](#1-introduction-to-data-pipelines)
2. [Batch vs Streaming Processing](#2-batch-vs-streaming-processing)
3. [Apache Spark Fundamentals](#3-apache-spark-fundamentals)
4. [Spark Optimization Techniques](#4-spark-optimization-techniques)
5. [Data Ingestion Patterns](#5-data-ingestion-patterns)
6. [ETL vs ELT Architecture](#6-etl-vs-elt-architecture)
7. [Scalability and Performance](#7-scalability-and-performance)
8. [Cost Optimization Strategies](#8-cost-optimization-strategies)
9. [Data Pipeline Orchestration](#9-data-pipeline-orchestration)
10. [Common Architecture Patterns](#10-common-architecture-patterns)
11. [Practice Problems](#11-practice-problems)
12. [Summary and Self-Assessment](#12-summary-and-self-assessment)

---

## 1. Introduction to Data Pipelines

### What is a Data Pipeline?

A **data pipeline** is a set of processes that move data from one or more sources to a destination, typically transforming and enriching the data along the way.

**Key Components:**
```
Source → Ingestion → Processing → Storage → Consumption
```

**Real-World Example:**
```
E-commerce Clickstream Pipeline:
1. Source: User clicks on website
2. Ingestion: Events sent to Kafka
3. Processing: Spark job aggregates by session
4. Storage: Results written to data warehouse
5. Consumption: BI dashboards show user behavior
```

### Why Pipeline Architecture Matters

**Interview Relevance:**
- 30-40% of data engineering interviews focus on pipeline design
- Tests understanding of trade-offs (batch vs streaming, cost vs speed)
- Requires knowledge of distributed systems
- Common questions: "How would you design a pipeline to process 1TB of data daily?"

**Industry Applications:**
- **Netflix**: Real-time recommendation pipelines
- **Uber**: Trip processing and surge pricing
- **Airbnb**: Search ranking and pricing optimization
- **Spotify**: Music recommendation and analytics

### Pipeline Characteristics

**1. Volume**
- How much data? (MB, GB, TB, PB)
- Determines infrastructure needs

**2. Velocity**
- How fast does data arrive? (batch, micro-batch, real-time)
- Influences processing model choice

**3. Variety**
- Structured (databases), semi-structured (JSON), unstructured (logs)
- Affects parsing and schema design

**4. Latency Requirements**
- Seconds (real-time fraud detection)
- Minutes (dashboard updates)
- Hours (daily reports)
- This drives batch vs streaming decision

---

## 2. Batch vs Streaming Processing

### Batch Processing

**Definition:** Process large volumes of data in scheduled intervals (hourly, daily, weekly).

**Characteristics:**
```
Schedule: Every 24 hours
Input: All data since last run
Processing: Full dataset at once
Output: Complete results when finished
```

**Advantages:**
- ✅ Simpler to implement and debug
- ✅ Lower cost (can use cheaper resources)
- ✅ Better for complex analytics requiring full dataset
- ✅ Easier to reprocess historical data

**Disadvantages:**
- ❌ High latency (data only available after batch completes)
- ❌ Resource spikes (need capacity for peak load)
- ❌ No real-time insights

**Best Use Cases:**
1. **Daily Reports** - Financial summaries, KPI dashboards
2. **Model Training** - ML models trained on historical data
3. **Data Warehousing** - Loading data into dimensional models
4. **Historical Analytics** - Analyzing trends over months/years

**Example: Daily Sales Summary**
```python
# Batch job runs at midnight
from pyspark.sql import SparkSession
from pyspark.sql.functions import sum, count, date_trunc

spark = SparkSession.builder.appName("DailySales").getOrCreate()

# Read yesterday's data
sales = spark.read.parquet("s3://sales/date=2024-01-15/")

# Aggregate
daily_summary = sales.groupBy(
    date_trunc("day", "transaction_time").alias("date"),
    "product_category"
).agg(
    sum("amount").alias("total_sales"),
    count("transaction_id").alias("num_transactions")
)

# Write results
daily_summary.write.mode("overwrite").parquet("s3://reports/daily_sales/")
```

### Streaming Processing

**Definition:** Process data continuously as it arrives, providing near real-time results.

**Characteristics:**
```
Schedule: Continuous
Input: Events as they arrive
Processing: Micro-batches or event-by-event
Output: Results within seconds/minutes
```

**Advantages:**
- ✅ Low latency (results available immediately)
- ✅ Smooth resource usage (no spikes)
- ✅ Real-time decision making
- ✅ Better for time-sensitive use cases

**Disadvantages:**
- ❌ More complex to implement and debug
- ❌ Higher operational cost
- ❌ Harder to reprocess data
- ❌ State management complexity

**Best Use Cases:**
1. **Fraud Detection** - Flag suspicious transactions immediately
2. **Monitoring & Alerting** - Detect anomalies in real-time
3. **Recommendation Systems** - Update based on recent user behavior
4. **IoT Analytics** - Process sensor data continuously

**Example: Real-Time Fraud Detection**
```python
# Streaming job runs continuously
from pyspark.sql import SparkSession
from pyspark.sql.functions import window, avg, col

spark = SparkSession.builder.appName("FraudDetection").getOrCreate()

# Read from Kafka stream
transactions = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "transactions") \
    .load()

# Calculate average transaction amount per user in 5-minute windows
fraud_detection = transactions \
    .groupBy(
        window("timestamp", "5 minutes"),
        "user_id"
    ) \
    .agg(avg("amount").alias("avg_amount"))

# Flag transactions > 10x average
flagged = fraud_detection.filter(col("amount") > col("avg_amount") * 10)

# Write alerts
flagged.writeStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("topic", "fraud_alerts") \
    .start()
```

### Hybrid Approaches: Lambda & Kappa Architecture

**Lambda Architecture**
```
                    ┌─── Batch Layer (complete, accurate) ───┐
Data Sources ───────┤                                          ├──→ Merge ──→ Serving Layer
                    └─── Speed Layer (fast, approximate) ─────┘
```

**Characteristics:**
- Batch layer: Processes all data, slow but accurate
- Speed layer: Processes recent data, fast but approximate
- Serving layer: Combines both for complete view

**Example Use Case:**
- **E-commerce Product Views**
  - Batch: Daily aggregation of all historical views
  - Speed: Last hour's views updated every minute
  - Serving: Combine for total lifetime + recent views

**Kappa Architecture**
```
Data Sources ──→ Stream Processing ──→ Serving Layer
                       ↓
                  Replayable Log (Kafka)
```

**Characteristics:**
- Everything is a stream
- Reprocess historical data by replaying stream
- Simpler than Lambda (one processing path)

**Example Use Case:**
- **User Activity Analytics**
  - All events stored in Kafka (30-day retention)
  - Streaming job processes events continuously
  - Reprocess by replaying Kafka from earlier offset

### Decision Matrix: Batch vs Streaming

| Factor | Batch | Streaming |
|--------|-------|-----------|
| **Latency** | Hours-Days | Seconds-Minutes |
| **Complexity** | Low | High |
| **Cost** | Lower | Higher |
| **Use Case** | Historical analytics | Real-time decisions |
| **Data Volume** | Large (TB-PB) | Continuous flow |
| **Debugging** | Easier | Harder |
| **Reprocessing** | Simple | Complex |

**Interview Question Pattern:**
"When would you choose batch over streaming?"

**Answer Framework:**
1. **Check latency requirement** - If results needed within minutes → streaming
2. **Consider complexity** - Simple analytics → batch
3. **Evaluate cost** - Limited budget → batch
4. **Assess data characteristics** - Scheduled dumps → batch, continuous events → streaming

---

## 3. Apache Spark Fundamentals

### What is Apache Spark?

**Apache Spark** is a unified analytics engine for large-scale data processing, supporting both batch and streaming workloads.

**Key Features:**
- In-memory processing (10-100x faster than MapReduce)
- Supports SQL, streaming, ML, and graph processing
- Works with HDFS, S3, Cassandra, and more
- Language support: Scala, Python, Java, R

### Spark Architecture

```
Driver Program
├── SparkContext
└── Cluster Manager (YARN, Mesos, Kubernetes)
    ├── Executor 1 (Worker Node 1)
    │   └── Tasks
    ├── Executor 2 (Worker Node 2)
    │   └── Tasks
    └── Executor N (Worker Node N)
        └── Tasks
```

**Components:**

1. **Driver Program**
   - Runs main() function
   - Creates SparkContext
   - Converts code into tasks

2. **SparkContext**
   - Connection to cluster
   - Coordinates execution

3. **Cluster Manager**
   - Allocates resources
   - Manages executors

4. **Executors**
   - Run on worker nodes
   - Execute tasks
   - Store data in memory

### RDD, DataFrame, and Dataset

**1. RDD (Resilient Distributed Dataset)**
- Low-level API
- Immutable, partitioned collection
- Functional transformations

```python
# RDD example (rarely used now)
rdd = spark.sparkContext.parallelize([1, 2, 3, 4, 5])
result = rdd.map(lambda x: x * 2).filter(lambda x: x > 5).collect()
# [6, 8, 10]
```

**2. DataFrame (Recommended for most use cases)**
- High-level API with schema
- Optimized with Catalyst optimizer
- SQL-like operations

```python
# DataFrame example (most common)
from pyspark.sql import Row

data = [
    Row(name="Alice", age=25, salary=50000),
    Row(name="Bob", age=30, salary=60000),
    Row(name="Charlie", age=35, salary=70000)
]

df = spark.createDataFrame(data)

# SQL-style
df.filter(df.age > 25).select("name", "salary").show()

# Output:
# +-------+------+
# |   name|salary|
# +-------+------+
# |    Bob| 60000|
# |Charlie| 70000|
# +-------+------+
```

**3. Dataset (Scala/Java only)**
- Type-safe version of DataFrame
- Not available in PySpark

### Transformations vs Actions

**Transformations (Lazy)**
- Return a new RDD/DataFrame
- Not executed immediately
- Examples: `map()`, `filter()`, `groupBy()`, `join()`

**Actions (Eager)**
- Trigger execution
- Return results to driver
- Examples: `collect()`, `count()`, `show()`, `write()`

```python
# Transformations (lazy - nothing happens yet)
df1 = df.filter(df.age > 25)  # No computation
df2 = df1.select("name")      # Still no computation

# Action (eager - triggers execution)
df2.show()  # Now Spark executes all transformations
```

**Why Lazy Evaluation?**
- Spark can optimize entire plan
- Avoid unnecessary computations
- Pipeline multiple operations efficiently

### Common DataFrame Operations

**1. Reading Data**
```python
# CSV
df = spark.read.csv("s3://bucket/data.csv", header=True, inferSchema=True)

# Parquet (preferred for performance)
df = spark.read.parquet("s3://bucket/data.parquet")

# JSON
df = spark.read.json("s3://bucket/data.json")

# JDBC (database)
df = spark.read.jdbc(
    url="jdbc:postgresql://host:5432/db",
    table="users",
    properties={"user": "admin", "password": "pass"}
)
```

**2. Filtering**
```python
# Filter by condition
df.filter(df.age > 30)
df.filter((df.age > 30) & (df.salary > 50000))  # Multiple conditions

# Filter with SQL
df.filter("age > 30 AND salary > 50000")
```

**3. Selecting and Renaming**
```python
# Select columns
df.select("name", "age")

# Rename
df.withColumnRenamed("name", "full_name")

# Add new column
from pyspark.sql.functions import col
df.withColumn("age_plus_10", col("age") + 10)
```

**4. Aggregations**
```python
from pyspark.sql.functions import sum, avg, count, max, min

# Group by
df.groupBy("department").agg(
    avg("salary").alias("avg_salary"),
    count("employee_id").alias("num_employees")
)

# Multiple aggregations
df.groupBy("department", "location").agg(
    sum("salary").alias("total_salary"),
    max("age").alias("max_age")
)
```

**5. Joins**
```python
# Inner join
employees.join(departments, "department_id", "inner")

# Left join
employees.join(departments, "department_id", "left")

# Multiple column join
employees.join(
    departments,
    (employees.dept_id == departments.id) &
    (employees.location == departments.location),
    "inner"
)
```

**6. Window Functions**
```python
from pyspark.sql.window import Window
from pyspark.sql.functions import row_number, rank, dense_rank

# Partition by department, order by salary
window_spec = Window.partitionBy("department").orderBy(col("salary").desc())

df.withColumn("rank", rank().over(window_spec)) \
  .withColumn("row_num", row_number().over(window_spec))
```

**7. Writing Data**
```python
# Parquet (best for analytics)
df.write.mode("overwrite").parquet("s3://bucket/output/")

# Partitioned write (improves query performance)
df.write.partitionBy("year", "month").parquet("s3://bucket/output/")

# CSV
df.write.csv("s3://bucket/output.csv", header=True)

# Database
df.write.jdbc(
    url="jdbc:postgresql://host:5432/db",
    table="results",
    mode="append",
    properties={"user": "admin", "password": "pass"}
)
```

---

## 4. Spark Optimization Techniques

Spark optimization is a critical interview topic. Understanding how to make Spark jobs run faster and cheaper separates junior from senior engineers.

### 1. Partitioning

**What is Partitioning?**
A partition is a logical chunk of data. Spark processes each partition in parallel on different executors.

```
DataFrame (1000 rows)
├── Partition 1 (rows 1-250)   → Executor 1
├── Partition 2 (rows 251-500) → Executor 2
├── Partition 3 (rows 501-750) → Executor 3
└── Partition 4 (rows 751-1000)→ Executor 4
```

**Check Number of Partitions**
```python
df.rdd.getNumPartitions()  # Returns number of partitions
```

**Repartitioning**
```python
# Increase partitions (adds shuffle)
df_more_partitions = df.repartition(100)

# Decrease partitions (no shuffle, combines existing)
df_fewer_partitions = df.coalesce(10)
```

**When to Repartition:**

1. **Too Few Partitions** (each partition > 128MB)
   - Symptom: Tasks take too long
   - Solution: Increase partitions with `repartition()`

2. **Too Many Partitions** (each partition < 10MB)
   - Symptom: Task overhead dominates
   - Solution: Decrease with `coalesce()`

**Optimal Partition Size:** 128MB - 256MB per partition

**Example:**
```python
# 100GB dataset, want 128MB partitions
# 100,000 MB / 128 MB = ~781 partitions
df = spark.read.parquet("s3://bucket/large_data/").repartition(800)
```

**Partition By Column** (for downstream queries)
```python
# Partition by date for date-range queries
df.write.partitionBy("year", "month", "day").parquet("s3://output/")

# Directory structure:
# s3://output/year=2024/month=01/day=15/part-00000.parquet
# s3://output/year=2024/month=01/day=16/part-00001.parquet

# Later: Read only specific date (fast!)
df_jan15 = spark.read.parquet("s3://output/year=2024/month=01/day=15/")
```

### 2. Caching and Persistence

**When to Cache:**
- Reusing the same DataFrame multiple times
- Iterative algorithms (ML training)
- Interactive analysis

**Caching Example:**
```python
# Without caching (reads from disk 3 times!)
df = spark.read.parquet("s3://bucket/data/")
count1 = df.filter(df.age > 30).count()  # Read 1
count2 = df.filter(df.age < 20).count()  # Read 2
avg_sal = df.agg(avg("salary"))          # Read 3

# With caching (reads from disk once, then from memory)
df = spark.read.parquet("s3://bucket/data/")
df.cache()  # or df.persist()
count1 = df.filter(df.age > 30).count()  # Read from disk, cache result
count2 = df.filter(df.age < 20).count()  # Read from cache
avg_sal = df.agg(avg("salary"))          # Read from cache

# Clean up when done
df.unpersist()
```

**Storage Levels:**
```python
from pyspark import StorageLevel

# Memory only (default, fastest but can fail if OOM)
df.persist(StorageLevel.MEMORY_ONLY)

# Memory and disk (safer, spills to disk if memory full)
df.persist(StorageLevel.MEMORY_AND_DISK)

# Disk only (slowest but most reliable)
df.persist(StorageLevel.DISK_ONLY)

# Serialized (uses less memory, more CPU)
df.persist(StorageLevel.MEMORY_ONLY_SER)
```

**Rule of Thumb:**
- Use `cache()` for interactive analysis
- Use `MEMORY_AND_DISK` for production jobs
- Always `unpersist()` when done to free memory

### 3. Broadcast Joins

**Problem: Shuffle Joins are Expensive**
```python
# Regular join (shuffle join)
large_df = spark.read.parquet("s3://bucket/large/")  # 100GB
small_df = spark.read.parquet("s3://bucket/small/")  # 10MB

result = large_df.join(small_df, "user_id")
# Spark shuffles both DataFrames across network - SLOW!
```

**Solution: Broadcast Join**
```python
from pyspark.sql.functions import broadcast

# Broadcast small table to all executors
result = large_df.join(broadcast(small_df), "user_id")
# Small table copied to each executor (no shuffle) - FAST!
```

**How Broadcast Works:**
```
Executor 1: large_df partition 1 + [small_df copy] → join locally
Executor 2: large_df partition 2 + [small_df copy] → join locally
Executor 3: large_df partition 3 + [small_df copy] → join locally
```

**When to Broadcast:**
- Small table < 100MB (configurable with `spark.sql.autoBroadcastJoinThreshold`)
- One table much smaller than the other
- Avoid for tables > 1GB (will cause OOM errors)

**Example Performance Gain:**
- Shuffle join: 10 minutes
- Broadcast join: 30 seconds
- **20x faster!**

### 4. Avoiding Shuffles

**What is a Shuffle?**
Moving data across network between executors. Very expensive!

**Operations That Cause Shuffles:**
- `groupBy()` / `agg()`
- `join()` (non-broadcast)
- `distinct()`
- `repartition()`
- `sortBy()`

**Minimize Shuffles:**

**Bad (multiple shuffles):**
```python
df.groupBy("user_id").count() \
  .groupBy("count").count() \
  .join(other_df, "count")
# 3 shuffles!
```

**Good (fewer shuffles):**
```python
# Combine operations
df.groupBy("user_id").count() \
  .groupBy("count").count()  # Only 2 shuffles

# Use broadcast for join
.join(broadcast(other_df), "count")  # No shuffle for join
```

### 5. Filter Early (Predicate Pushdown)

**Bad (filters after reading everything):**
```python
df = spark.read.parquet("s3://bucket/all_data/")  # Reads 1TB
result = df.filter(df.date == "2024-01-15")       # Filters to 10GB
```

**Good (filters during read):**
```python
# If data partitioned by date
df = spark.read.parquet("s3://bucket/all_data/date=2024-01-15/")  # Reads only 10GB

# Or use partition filters
df = spark.read.parquet("s3://bucket/all_data/").filter("date = '2024-01-15'")
# Spark's Catalyst optimizer pushes filter to scan
```

**Column Pruning:**
```python
# Bad (reads all columns)
df = spark.read.parquet("s3://bucket/data/")  # 50 columns
result = df.select("user_id", "amount")       # Uses 2 columns

# Good (reads only needed columns)
df = spark.read.parquet("s3://bucket/data/").select("user_id", "amount")
# Parquet columnar format only reads 2 columns
```

### 6. Spark Configuration Tuning

**Key Configurations:**

```python
spark = SparkSession.builder \
    .appName("OptimizedJob") \
    .config("spark.executor.memory", "8g") \      # Memory per executor
    .config("spark.executor.cores", "4") \        # Cores per executor
    .config("spark.sql.shuffle.partitions", "200") \  # Partitions for shuffles
    .config("spark.default.parallelism", "200") \ # Default parallelism
    .config("spark.sql.autoBroadcastJoinThreshold", "100MB") \  # Broadcast threshold
    .getOrCreate()
```

**Common Tuning Scenarios:**

**Large Dataset (1TB+):**
```python
.config("spark.executor.memory", "16g")
.config("spark.executor.cores", "5")
.config("spark.sql.shuffle.partitions", "2000")  # More partitions
```

**Small Dataset (<10GB):**
```python
.config("spark.executor.memory", "4g")
.config("spark.sql.shuffle.partitions", "50")  # Fewer partitions
```

**Memory-Intensive (large joins):**
```python
.config("spark.executor.memory", "20g")
.config("spark.memory.fraction", "0.8")  # 80% for execution
.config("spark.memory.storageFraction", "0.3")  # 30% for caching
```

### 7. Using Explain Plans

**Understand Execution Plan:**
```python
df.explain()  # Physical plan
df.explain(True)  # Logical + physical plans

# Example output:
# == Physical Plan ==
# *(2) HashAggregate(keys=[user_id#10], functions=[count(1)])
# +- Exchange hashpartitioning(user_id#10, 200)  ← SHUFFLE!
#    +- *(1) HashAggregate(keys=[user_id#10], functions=[partial_count(1)])
#       +- *(1) FileScan parquet [user_id#10]
```

**Look for:**
- `Exchange` → Shuffle (expensive!)
- `BroadcastHashJoin` → Broadcast join (good!)
- `FileScan` → Reading from disk
- Number of stages (fewer is better)

---

## 5. Data Ingestion Patterns

### Full Load vs Incremental Load

**Full Load (Replace Everything)**
```python
# Load entire dataset each time
df = spark.read.jdbc(url="jdbc:mysql://db/source", table="users")
df.write.mode("overwrite").parquet("s3://warehouse/users/")
```

**Pros:**
- Simple logic
- No state tracking
- Data consistency guaranteed

**Cons:**
- Slow for large tables
- Wastes resources
- High database load

**Use When:**
- Small tables (< 1GB)
- Full refresh needed (SCD Type 1)
- Source has no timestamp column

**Incremental Load (Only New/Changed Data)**
```python
# Track last processed timestamp
last_run = get_last_run_time()  # From metadata table

df = spark.read.jdbc(
    url="jdbc:mysql://db/source",
    table="users",
    predicates=[f"updated_at > '{last_run}'"]
)

# Merge with existing data (upsert)
df.write.mode("append").parquet("s3://warehouse/users/")

# Update last run time
save_last_run_time(current_time)
```

**Pros:**
- Fast (only process changes)
- Lower resource usage
- Less database impact

**Cons:**
- Complex logic
- Requires timestamp column
- Risk of missing data

**Use When:**
- Large tables (> 10GB)
- Source has `updated_at` column
- Frequent updates

### Change Data Capture (CDC)

**What is CDC?**
Capture insert/update/delete operations from source database and apply to target.

**CDC Pattern:**
```
Source DB → CDC Tool (Debezium, DMS) → Message Queue (Kafka) → Spark Streaming → Target
```

**Example with Kafka:**
```python
# Read CDC events from Kafka
cdc_stream = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "db.users.cdc") \
    .load()

# Parse CDC events
from pyspark.sql.functions import from_json, col

schema = "user_id INT, name STRING, email STRING, operation STRING"

parsed = cdc_stream.select(
    from_json(col("value").cast("string"), schema).alias("data")
).select("data.*")

# Apply changes
def upsert_batch(batch_df, batch_id):
    # Merge logic here
    inserts = batch_df.filter(col("operation") == "INSERT")
    updates = batch_df.filter(col("operation") == "UPDATE")
    deletes = batch_df.filter(col("operation") == "DELETE")

    # Apply to target
    inserts.write.mode("append").saveAsTable("users")
    # ... handle updates and deletes

parsed.writeStream \
    .foreachBatch(upsert_batch) \
    .start()
```

**Benefits:**
- Near real-time sync
- Captures all changes
- Low source database impact

**Challenges:**
- Complex setup
- Requires CDC tool
- State management

### API Ingestion

**REST API Pattern:**
```python
import requests
from pyspark.sql import Row

def fetch_api_data(page):
    response = requests.get(f"https://api.example.com/users?page={page}")
    return response.json()

# Fetch all pages
all_data = []
for page in range(1, 100):
    all_data.extend(fetch_api_data(page))

# Convert to DataFrame
df = spark.createDataFrame([Row(**item) for item in all_data])
df.write.parquet("s3://raw/api_data/")
```

**Challenges:**
- Rate limiting
- API pagination
- Error handling
- Authentication

**Best Practices:**
- Implement retry logic
- Respect rate limits
- Use connection pooling
- Cache credentials securely

### File-Based Ingestion

**S3/Cloud Storage Pattern:**
```python
# Read new files only
processed_files = get_processed_files()  # From tracking table

all_files = list_s3_files("s3://incoming/data/")
new_files = [f for f in all_files if f not in processed_files]

# Process new files
for file_path in new_files:
    df = spark.read.json(file_path)
    df.write.mode("append").parquet("s3://processed/")
    mark_file_processed(file_path)
```

**Auto Loader (Databricks Specific):**
```python
# Automatically detects and processes new files
df = spark.readStream \
    .format("cloudFiles") \
    .option("cloudFiles.format", "json") \
    .load("s3://incoming/data/")

df.writeStream \
    .format("delta") \
    .option("checkpointLocation", "s3://checkpoints/") \
    .start("s3://processed/")
```

---

## 6. ETL vs ELT Architecture

### ETL (Extract, Transform, Load)

**Definition:** Transform data **before** loading into warehouse.

```
Source → Extract → Transform (Spark/Airflow) → Load → Warehouse
```

**Characteristics:**
- Transformation happens in separate compute (Spark cluster)
- Clean, ready-to-use data loaded to warehouse
- Warehouse stores only final results

**Example:**
```python
# Extract
raw_data = spark.read.jdbc(url="jdbc:mysql://source", table="orders")

# Transform (complex business logic)
transformed = raw_data \
    .filter(col("status") == "completed") \
    .withColumn("revenue", col("quantity") * col("price")) \
    .groupBy("customer_id", "month").agg(sum("revenue"))

# Load
transformed.write.jdbc(url="jdbc:redshift://warehouse", table="monthly_revenue")
```

**Pros:**
- Less warehouse compute needed
- Sensitive data filtered before storage
- Optimized data shape for queries

**Cons:**
- No raw data in warehouse (can't re-transform)
- Slower (separate transformation step)
- More complex pipeline

**Use When:**
- Limited warehouse compute
- Privacy/compliance requires filtering
- Transformation logic is complex
- Traditional data warehouses (Redshift, Oracle)

### ELT (Extract, Load, Transform)

**Definition:** Load raw data first, **then** transform in warehouse.

```
Source → Extract → Load → Warehouse → Transform (SQL/dbt)
```

**Characteristics:**
- Raw data stored in warehouse
- Transformation using warehouse SQL
- Multiple views from same raw data

**Example:**
```python
# Extract
raw_data = spark.read.jdbc(url="jdbc:mysql://source", table="orders")

# Load (no transformation!)
raw_data.write.jdbc(url="jdbc:snowflake://warehouse", table="raw_orders")

# Transform (happens in Snowflake via SQL/dbt)
-- CREATE OR REPLACE VIEW monthly_revenue AS
-- SELECT
--     customer_id,
--     DATE_TRUNC('month', order_date) AS month,
--     SUM(quantity * price) AS revenue
-- FROM raw_orders
-- WHERE status = 'completed'
-- GROUP BY 1, 2;
```

**Pros:**
- Raw data always available (easy to re-transform)
- Faster pipeline (no separate transform step)
- Leverage warehouse optimizations
- Simpler architecture

**Cons:**
- Higher warehouse costs (compute + storage)
- Must load all data (even sensitive)
- Warehouse must be powerful

**Use When:**
- Modern cloud warehouses (Snowflake, BigQuery)
- Need flexibility to re-transform
- Want to leverage warehouse features
- Multiple downstream use cases

### Decision Matrix: ETL vs ELT

| Factor | ETL | ELT |
|--------|-----|-----|
| **Warehouse Type** | Traditional (Redshift, Oracle) | Modern (Snowflake, BigQuery) |
| **Data Volume** | Reduce before loading | Load everything |
| **Transform Complexity** | Heavy (Spark, Python) | SQL-based (dbt) |
| **Storage Cost** | Lower (only final data) | Higher (raw + transformed) |
| **Compute Cost** | Separate cluster | Warehouse compute |
| **Flexibility** | Fixed transformations | Easy to re-transform |
| **Speed** | Slower (extra step) | Faster (direct load) |

**Hybrid Approach:**
Many organizations use both:
- ETL for sensitive data filtering
- ELT for rest of data

---

## 7. Scalability and Performance

### Horizontal vs Vertical Scaling

**Vertical Scaling (Scale Up)**
```
Before: 4 cores, 16GB RAM per node
After:  16 cores, 64GB RAM per node
```

**Pros:**
- Simple (just upgrade machines)
- No code changes

**Cons:**
- Limited by hardware
- Expensive
- Single point of failure

**Horizontal Scaling (Scale Out)**
```
Before: 4 nodes
After:  16 nodes
```

**Pros:**
- Nearly unlimited scaling
- Fault tolerant
- Cost-effective (commodity hardware)

**Cons:**
- Network overhead
- More complex

**Spark's Approach:**
Primarily **horizontal scaling** - add more executors/nodes.

### Partitioning Strategies

**Hash Partitioning (Default)**
```python
df.repartition(100, "user_id")
# user_id hashed to determine partition
# Even distribution
```

**Range Partitioning**
```python
df.repartitionByRange(100, "date")
# Partitions by ranges: [Jan 1-5], [Jan 6-10], etc.
# Useful for sorted data
```

**Custom Partitioning**
```python
# Partition by year for yearly queries
df.write.partitionBy("year").parquet("s3://output/")
```

### Data Skew Handling

**What is Data Skew?**
Uneven distribution of data across partitions.

```
Partition 1: 1000 records  → Fast
Partition 2: 1000 records  → Fast
Partition 3: 1000 records  → Fast
Partition 4: 1,000,000 records  → SLOW (bottleneck!)
```

**Symptoms:**
- One task takes 100x longer than others
- Memory errors on single executor
- Underutilized cluster

**Detection:**
```python
# Check partition sizes
df.groupBy(spark_partition_id()).count().show()
```

**Solutions:**

**1. Salt Keys**
```python
# Skewed join on user_id (user 123 has 1M records)
from pyspark.sql.functions import rand, floor

# Add random salt
df = df.withColumn("salt", floor(rand() * 10))
other_df = other_df.withColumn("salt", floor(rand() * 10))

# Join on salted key
result = df.join(other_df, ["user_id", "salt"])
```

**2. Broadcast Skewed Keys**
```python
# If skewed side is small, broadcast it
result = large_df.join(broadcast(small_skewed_df), "key")
```

**3. Adaptive Query Execution (Spark 3.0+)**
```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
# Spark automatically handles skew!
```

### Compression

**Why Compress?**
- Reduce storage costs (5-10x smaller)
- Faster I/O (less data to read)
- Tradeoff: More CPU for decompression

**Compression Formats:**

| Format | Compression Ratio | Speed | Splittable | Use Case |
|--------|-------------------|-------|------------|----------|
| **Snappy** | ~2x | Very Fast | Yes (with container) | Default, balanced |
| **Gzip** | ~4x | Slow | No | Archival, cold storage |
| **LZO** | ~2x | Fast | Yes | Real-time processing |
| **Zstd** | ~3x | Fast | Yes | Modern default |

**Example:**
```python
# Write with Snappy compression (default for Parquet)
df.write.parquet("s3://output/", compression="snappy")

# Gzip for cold storage
df.write.parquet("s3://archive/", compression="gzip")
```

**File Formats:**

| Format | Storage | Speed | Schema Evolution | Use Case |
|--------|---------|-------|------------------|----------|
| **Parquet** | Columnar | Fast reads | Yes | Analytics (recommended) |
| **ORC** | Columnar | Fast reads | Yes | Hive, ACID workloads |
| **Avro** | Row-based | Fast writes | Yes | Streaming, CDC |
| **CSV/JSON** | Text | Slow | No | Human-readable, legacy |

**Recommendation:**
- **Analytics:** Parquet with Snappy
- **Streaming:** Avro
- **Archival:** Parquet with Gzip

---

## 8. Cost Optimization Strategies

### Compute Cost Optimization

**1. Right-Size Clusters**
```python
# Over-provisioned (wasteful)
spark = SparkSession.builder \
    .config("spark.executor.instances", "100") \  # Too many!
    .config("spark.executor.memory", "32g") \     # Too much!
    .getOrCreate()

# Right-sized
spark = SparkSession.builder \
    .config("spark.executor.instances", "20") \
    .config("spark.executor.memory", "8g") \
    .getOrCreate()
```

**Formula:**
```
Total Cluster Memory = Executor Instances × Executor Memory
Example: 20 executors × 8GB = 160GB total
```

**2. Auto-Scaling (Cloud)**
```python
# AWS EMR auto-scaling
"AutoScalingRole": "EMR_AutoScaling_DefaultRole",
"ScalingRule": {
    "MinCapacity": 2,
    "MaxCapacity": 20,
    "TargetOnDemandCapacity": 0,
    "TargetSpotCapacity": 18  # Use Spot instances (60-80% cheaper!)
}
```

**3. Spot Instances**
- AWS Spot: 70-90% cheaper than on-demand
- Risk: Can be terminated (use for fault-tolerant workloads)
- Strategy: Mix spot + on-demand

**Example Cost:**
```
10 r5.4xlarge on-demand: $10/hour
10 r5.4xlarge spot: $3/hour
Savings: $7/hour = $5,000/month
```

**4. Schedule Batch Jobs Off-Peak**
```python
# Run at 2 AM when compute is cheaper
# Airflow DAG
schedule_interval="0 2 * * *"  # Daily at 2 AM
```

### Storage Cost Optimization

**1. Partition Pruning**
```python
# Without partitioning: Scan 1TB
df = spark.read.parquet("s3://data/")

# With partitioning: Scan only 10GB
df = spark.read.parquet("s3://data/year=2024/month=01/")
# Savings: 99% less I/O
```

**2. Columnar Storage**
```python
# CSV: Read all columns (100 columns, 100GB)
df = spark.read.csv("s3://data.csv").select("user_id", "amount")

# Parquet: Read only 2 columns (10GB)
df = spark.read.parquet("s3://data.parquet").select("user_id", "amount")
# Savings: 90% less data read
```

**3. Lifecycle Policies**
```
S3 Lifecycle:
- Hot (S3 Standard): 0-30 days
- Warm (S3 IA): 30-90 days (50% cheaper)
- Cold (S3 Glacier): 90+ days (80% cheaper)
```

**4. Compression**
```
Uncompressed: 1TB = $23/month (S3 Standard)
Snappy: 500GB = $11.50/month
Gzip: 250GB = $5.75/month
Savings: $17.25/month per TB
```

### Query Optimization

**1. Cache Intermediate Results**
```python
# Recompute every time (expensive)
df = spark.read.parquet("s3://large_data/")
result1 = df.filter(...).groupBy(...).agg(...)
result2 = df.filter(...).groupBy(...).agg(...)

# Cache and reuse
df.cache()  # Compute once, use multiple times
```

**2. Predicate Pushdown**
```python
# Reads 100GB, filters to 1GB
df = spark.read.parquet("s3://data/").filter("date = '2024-01-15'")

# Reads only 1GB (filter pushed to storage)
# Automatic with partitioned data
```

**3. Limit Early**
```python
# Bad: Processes everything
df.orderBy("salary", ascending=False).show(10)

# Good: Uses topK optimization
df.orderBy("salary", ascending=False).limit(10).show()
```

---

## 9. Data Pipeline Orchestration

### What is Orchestration?

**Orchestration** = Scheduling and managing dependencies between pipeline tasks.

```
Task 1: Ingest from API
     ↓
Task 2: Transform data (depends on Task 1)
     ↓
Task 3: Load to warehouse (depends on Task 2)
     ↓
Task 4: Generate report (depends on Task 3)
```

### Apache Airflow

**Most Popular Orchestration Tool**

**Key Concepts:**
- **DAG (Directed Acyclic Graph):** Workflow definition
- **Task:** Single unit of work
- **Operator:** Template for tasks (PythonOperator, SparkSubmitOperator)
- **Sensor:** Waits for condition (file arrival, time)

**Example DAG:**
```python
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'data_team',
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'email_on_failure': True,
    'email': ['alerts@company.com']
}

dag = DAG(
    'daily_sales_pipeline',
    default_args=default_args,
    description='Process daily sales data',
    schedule_interval='0 2 * * *',  # Daily at 2 AM
    start_date=datetime(2024, 1, 1),
    catchup=False
)

# Task 1: Ingest data
ingest_task = SparkSubmitOperator(
    task_id='ingest_sales',
    application='/path/to/ingest.py',
    conn_id='spark_default',
    dag=dag
)

# Task 2: Transform
transform_task = SparkSubmitOperator(
    task_id='transform_sales',
    application='/path/to/transform.py',
    dag=dag
)

# Task 3: Load to warehouse
def load_to_warehouse():
    # Python logic
    pass

load_task = PythonOperator(
    task_id='load_warehouse',
    python_callable=load_to_warehouse,
    dag=dag
)

# Task 4: Data quality checks
def run_quality_checks():
    # Validate data
    pass

quality_task = PythonOperator(
    task_id='quality_checks',
    python_callable=run_quality_checks,
    dag=dag
)

# Define dependencies
ingest_task >> transform_task >> load_task >> quality_task
```

**Airflow Features:**

**1. Retry Logic**
```python
retries=3
retry_delay=timedelta(minutes=5)
retry_exponential_backoff=True  # Wait longer each retry
```

**2. SLAs**
```python
sla=timedelta(hours=2)  # Alert if task takes > 2 hours
```

**3. Sensors (Wait for Events)**
```python
from airflow.sensors.s3_key_sensor import S3KeySensor

wait_for_file = S3KeySensor(
    task_id='wait_for_file',
    bucket_name='my-bucket',
    bucket_key='data/{{ ds }}/file.csv',  # ds = execution date
    poke_interval=60,  # Check every 60 seconds
    dag=dag
)
```

**4. Dynamic DAGs**
```python
# Generate tasks dynamically
for region in ['US', 'EU', 'ASIA']:
    task = SparkSubmitOperator(
        task_id=f'process_{region}',
        application=f'/path/to/process_{region}.py',
        dag=dag
    )
```

### Other Orchestration Tools

**1. Prefect**
- Modern, Pythonic alternative to Airflow
- Better error handling
- Easier local development

**2. Dagster**
- Asset-based (data-centric vs task-centric)
- Strong typing
- Built-in data lineage

**3. AWS Step Functions**
- Serverless orchestration
- Good for AWS-native pipelines
- Pay per execution

**4. Databricks Workflows**
- Integrated with Databricks
- Notebook orchestration
- Good for Spark-heavy workloads

---

## 10. Common Architecture Patterns

### Pattern 1: Medallion Architecture (Bronze/Silver/Gold)

**Concept:** Progressive data refinement through layers.

```
Bronze (Raw) → Silver (Cleaned) → Gold (Business-Ready)
```

**Bronze Layer (Raw)**
```python
# Ingest exactly as received
raw_df = spark.read.json("s3://landing/raw_events.json")

# No transformation, just save
raw_df.write.format("delta") \
    .mode("append") \
    .save("s3://bronze/events/")
```

**Characteristics:**
- Exact copy of source
- All data (including errors)
- Append-only
- Retention: 30-90 days

**Silver Layer (Cleaned)**
```python
# Read from bronze
bronze_df = spark.read.format("delta").load("s3://bronze/events/")

# Clean and standardize
silver_df = bronze_df \
    .filter(col("user_id").isNotNull()) \  # Remove nulls
    .withColumn("timestamp", to_timestamp("event_time")) \  # Standardize
    .dropDuplicates(["event_id"]) \  # Deduplicate
    .select("event_id", "user_id", "event_type", "timestamp")  # Select relevant

silver_df.write.format("delta") \
    .mode("append") \
    .save("s3://silver/events/")
```

**Characteristics:**
- Cleaned and validated
- Standardized schema
- Deduplicated
- Retention: 1-2 years

**Gold Layer (Business-Ready)**
```python
# Read from silver
silver_df = spark.read.format("delta").load("s3://silver/events/")

# Aggregate for business use case
gold_df = silver_df \
    .filter(col("event_type") == "purchase") \
    .groupBy("user_id", window("timestamp", "1 day").alias("date")) \
    .agg(
        sum("amount").alias("daily_revenue"),
        count("*").alias("num_purchases")
    )

gold_df.write.format("delta") \
    .mode("overwrite") \
    .save("s3://gold/daily_user_revenue/")
```

**Characteristics:**
- Aggregated for specific use case
- Business-friendly naming
- Optimized for queries
- Retention: Indefinite

**Benefits:**
- Clear data lineage
- Easy debugging (can trace back to raw)
- Different consumers use different layers
- Separation of concerns

### Pattern 2: Lambda Architecture

**Already covered in Section 2, but here's implementation:**

```python
# BATCH LAYER (Complete, Accurate)
def batch_processing():
    # Run daily
    df = spark.read.parquet("s3://all_historical_data/")
    aggregated = df.groupBy("user_id").agg(sum("revenue"))
    aggregated.write.mode("overwrite").save("s3://batch_views/user_revenue/")

# SPEED LAYER (Fast, Approximate)
def stream_processing():
    # Run continuously
    stream = spark.readStream.format("kafka").load()
    recent = stream.groupBy("user_id").agg(sum("revenue"))
    recent.writeStream.save("s3://speed_views/user_revenue/")

# SERVING LAYER (Merge Both)
def query_user_revenue(user_id):
    batch_revenue = read_from_batch_view(user_id)  # All-time revenue
    recent_revenue = read_from_speed_view(user_id)  # Last hour
    return batch_revenue + recent_revenue
```

### Pattern 3: Kappa Architecture

**All Stream, Replayable:**

```python
# Single stream processing pipeline
stream = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "events") \
    .option("startingOffsets", "earliest")  # Can replay from beginning!

processed = stream \
    .groupBy(window("timestamp", "1 hour"), "user_id") \
    .agg(sum("amount"))

processed.writeStream \
    .format("delta") \
    .option("checkpointLocation", "s3://checkpoints/") \
    .start("s3://output/")

# To reprocess: Just change startingOffsets and re-run
```

### Pattern 4: Event Sourcing

**Store all events, derive state:**

```python
# Event log (immutable)
events = [
    {"user_id": 1, "event": "account_created", "timestamp": "2024-01-01"},
    {"user_id": 1, "event": "deposit", "amount": 100, "timestamp": "2024-01-02"},
    {"user_id": 1, "event": "withdrawal", "amount": 30, "timestamp": "2024-01-03"}
]

# Derive current state from events
def get_account_balance(user_id):
    user_events = filter_events_by_user(user_id)
    balance = 0
    for event in user_events:
        if event["event"] == "deposit":
            balance += event["amount"]
        elif event["event"] == "withdrawal":
            balance -= event["amount"]
    return balance

# Can replay to any point in time!
def get_balance_at(user_id, timestamp):
    user_events = filter_events_by_user(user_id, before=timestamp)
    # ... compute balance up to that timestamp
```

---

## 11. Practice Problems

### Problem 1: Optimize Slow Spark Job

**Scenario:**
You have a Spark job that joins two tables and takes 2 hours. One table is 500GB (users), the other is 50MB (countries).

```python
users = spark.read.parquet("s3://data/users/")  # 500GB, 100M rows
countries = spark.read.parquet("s3://data/countries/")  # 50MB, 200 rows

result = users.join(countries, "country_code")
result.write.parquet("s3://output/")
```

**Question:** How would you optimize this?

<details>
<summary>Click for Solution</summary>

**Problem:** Shuffle join is expensive for large table.

**Solution:** Use broadcast join for small table.

```python
from pyspark.sql.functions import broadcast

users = spark.read.parquet("s3://data/users/")
countries = spark.read.parquet("s3://data/countries/")

# Broadcast small table
result = users.join(broadcast(countries), "country_code")
result.write.parquet("s3://output/")
```

**Why This Works:**
- Small table (50MB) copied to all executors
- No shuffle needed for large table
- Expected improvement: 2 hours → 10 minutes (12x faster)

**Additional Optimizations:**
```python
# 1. Partition output
result.write.partitionBy("country_code").parquet("s3://output/")

# 2. Coalesce before write (reduce small files)
result.coalesce(100).write.parquet("s3://output/")

# 3. Use compression
result.write.option("compression", "snappy").parquet("s3://output/")
```

</details>

---

### Problem 2: Design Daily User Activity Pipeline

**Scenario:**
Design a pipeline to process daily user activity logs (10GB/day) and generate:
1. Daily active users per country
2. Top 10 most viewed products
3. Average session duration

**Requirements:**
- Data arrives in S3 as JSON files
- Results needed by 8 AM daily
- Must handle late-arriving data (up to 24 hours late)

<details>
<summary>Click for Solution</summary>

**Architecture:**

```
1. Ingestion (S3) → 2. Processing (Spark) → 3. Storage (Parquet) → 4. Orchestration (Airflow)
```

**Airflow DAG:**
```python
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator
from datetime import datetime, timedelta

dag = DAG(
    'daily_user_activity',
    schedule_interval='0 6 * * *',  # 6 AM (allows for late data)
    start_date=datetime(2024, 1, 1),
    catchup=False
)

process_activity = SparkSubmitOperator(
    task_id='process_activity',
    application='/path/to/process_activity.py',
    application_args=['{{ yesterday_ds }}'],  # Process previous day
    dag=dag
)
```

**Spark Job (`process_activity.py`):**
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
import sys

date_to_process = sys.argv[1]  # e.g., "2024-01-15"

spark = SparkSession.builder.appName("UserActivity").getOrCreate()

# 1. Read data (includes current day + previous day for late arrivals)
df = spark.read.json(f"s3://logs/activity/date={date_to_process}/") \
    .union(
        spark.read.json(f"s3://logs/activity/date={date_to_process - 1}/")
            .filter(col("event_time") >= date_to_process)  # Only late arrivals
    )

# 2. Daily Active Users per Country
dau_by_country = df \
    .select("user_id", "country") \
    .distinct() \
    .groupBy("country") \
    .agg(count("user_id").alias("daily_active_users"))

dau_by_country.write \
    .mode("overwrite") \
    .parquet(f"s3://results/dau_by_country/date={date_to_process}/")

# 3. Top 10 Products
top_products = df \
    .filter(col("event_type") == "product_view") \
    .groupBy("product_id") \
    .agg(count("*").alias("views")) \
    .orderBy(col("views").desc()) \
    .limit(10)

top_products.write \
    .mode("overwrite") \
    .parquet(f"s3://results/top_products/date={date_to_process}/")

# 4. Average Session Duration
session_duration = df \
    .groupBy("session_id") \
    .agg(
        (max("event_time").cast("long") - min("event_time").cast("long")).alias("duration_sec")
    ) \
    .agg(avg("duration_sec").alias("avg_session_duration"))

session_duration.write \
    .mode("overwrite") \
    .parquet(f"s3://results/avg_session_duration/date={date_to_process}/")
```

**Key Design Decisions:**
1. **Schedule at 6 AM** - Allows late data to arrive
2. **Process previous day** - Complete dataset available
3. **Handle late arrivals** - Read current day + previous day, filter by event_time
4. **Partition output by date** - Easy to query specific days
5. **Overwrite mode** - Reprocess if needed (idempotent)

</details>

---

### Problem 3: Handle Data Skew

**Scenario:**
You're joining user transactions with user profiles. User ID 999 has 10 million transactions (celebrity account), while others have < 100. The join is very slow.

```python
transactions = spark.read.parquet("s3://transactions/")  # 1TB
profiles = spark.read.parquet("s3://profiles/")  # 100MB

result = transactions.join(profiles, "user_id")  # SLOW!
```

<details>
<summary>Click for Solution</summary>

**Problem:** User 999 causes extreme skew - one partition has 10M records, others have ~100.

**Solution 1: Broadcast Join**
```python
# If profiles is small enough
result = transactions.join(broadcast(profiles), "user_id")
```

**Solution 2: Salting (if broadcast not possible)**
```python
from pyspark.sql.functions import rand, floor, col, when

# Identify skewed keys
skewed_users = [999]  # In practice, detect automatically

# Salt skewed keys
transactions_salted = transactions.withColumn(
    "salt",
    when(col("user_id").isin(skewed_users), floor(rand() * 100))
    .otherwise(lit(0))
)

profiles_salted = profiles.withColumn(
    "salt",
    when(col("user_id").isin(skewed_users), explode(array([lit(i) for i in range(100)])))
    .otherwise(lit(0))
)

# Join on user_id + salt
result = transactions_salted.join(profiles_salted, ["user_id", "salt"])
```

**How Salting Works:**
- User 999's 10M transactions split into 100 partitions (100K each)
- User 999's profile duplicated 100 times (small overhead)
- Even distribution across partitions

**Solution 3: Adaptive Query Execution (Spark 3.0+)**
```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.skewedPartitionFactor", "5")
spark.conf.set("spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes", "256MB")

# Spark automatically detects and handles skew!
result = transactions.join(profiles, "user_id")
```

</details>

---

### Problem 4: Design a CDC Pipeline

**Scenario:**
Design a Change Data Capture pipeline to sync a MySQL database (1 million users, 10K updates/day) to a data warehouse in near real-time.

**Requirements:**
- Capture inserts, updates, deletes
- Latency < 5 minutes
- Handle schema changes

<details>
<summary>Click for Solution</summary>

**Architecture:**

```
MySQL → Debezium CDC → Kafka → Spark Streaming → Delta Lake
```

**Step 1: Setup Debezium CDC**
```json
{
  "name": "mysql-cdc-connector",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "database.hostname": "mysql-host",
    "database.port": "3306",
    "database.user": "debezium",
    "database.password": "password",
    "database.server.name": "production",
    "table.include.list": "mydb.users",
    "database.history.kafka.topic": "schema-changes",
    "include.schema.changes": "true"
  }
}
```

**Step 2: Spark Streaming Job**
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from delta.tables import DeltaTable

spark = SparkSession.builder \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()

# Read CDC stream from Kafka
cdc_stream = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "production.mydb.users") \
    .option("startingOffsets", "latest") \
    .load()

# Parse CDC events
schema = """
    user_id INT,
    name STRING,
    email STRING,
    updated_at TIMESTAMP,
    op STRING
"""

parsed_stream = cdc_stream.select(
    from_json(col("value").cast("string"), schema).alias("data")
).select("data.*")

# Upsert to Delta Lake
def upsert_to_delta(batch_df, batch_id):
    delta_table = DeltaTable.forPath(spark, "s3://warehouse/users")

    # Separate operations
    inserts = batch_df.filter(col("op").isin(["c", "r"]))  # create, read (initial)
    updates = batch_df.filter(col("op") == "u")  # update
    deletes = batch_df.filter(col("op") == "d")  # delete

    # Merge (upsert)
    if not updates.isEmpty():
        delta_table.alias("target").merge(
            updates.alias("source"),
            "target.user_id = source.user_id"
        ).whenMatchedUpdateAll() \
         .whenNotMatchedInsertAll() \
         .execute()

    # Inserts
    if not inserts.isEmpty():
        inserts.write.format("delta").mode("append").save("s3://warehouse/users")

    # Deletes
    if not deletes.isEmpty():
        for row in deletes.collect():
            delta_table.delete(f"user_id = {row.user_id}")

# Write stream
parsed_stream.writeStream \
    .foreachBatch(upsert_to_delta) \
    .option("checkpointLocation", "s3://checkpoints/users_cdc/") \
    .trigger(processingTime="1 minute") \
    .start()
```

**Benefits:**
- Near real-time (< 5 min latency)
- Handles all change types (insert/update/delete)
- Schema evolution via Debezium
- Exactly-once semantics with Delta Lake

</details>

---

### Problem 5: Cost Optimization

**Scenario:**
Your daily Spark job processes 100GB of data and costs $500/month. Management wants to reduce costs by 50%. The job currently:
- Runs on 20 r5.4xlarge instances (on-demand)
- Reads CSV files
- Processes all columns
- Runs at 3 PM daily

**Question:** How would you reduce costs?

<details>
<summary>Click for Solution</summary>

**Solution (Multiple Optimizations):**

**1. Use Spot Instances (70% savings)**
```python
# Switch to spot instances
"TargetSpotCapacity": 18,  # 18 spot, 2 on-demand for stability
"TargetOnDemandCapacity": 2
# Cost: $500 → $200/month (60% reduction)
```

**2. Convert to Parquet (90% smaller)**
```python
# One-time conversion
csv_df = spark.read.csv("s3://data/*.csv")
csv_df.write.parquet("s3://data_parquet/")

# Future jobs read Parquet
df = spark.read.parquet("s3://data_parquet/")
# Faster + smaller = less compute time
# Cost: $200 → $150/month (25% reduction)
```

**3. Column Pruning**
```python
# Before: Read all 50 columns
df = spark.read.parquet("s3://data/")

# After: Read only 5 needed columns
df = spark.read.parquet("s3://data/").select("col1", "col2", "col3", "col4", "col5")
# Parquet columnar format reads 90% less data
# Cost: $150 → $120/month (20% reduction)
```

**4. Schedule Off-Peak**
```python
# Run at 2 AM instead of 3 PM (off-peak pricing)
schedule_interval="0 2 * * *"
# Some cloud providers offer cheaper off-peak rates
# Cost: $120 → $110/month (8% reduction)
```

**5. Right-Size Cluster**
```python
# Before: 20 × r5.4xlarge (16 vCPU, 128GB each)
# After job optimization, need less compute

# After: 10 × r5.2xlarge (8 vCPU, 64GB each)
# Half the instances, half the cost
# Cost: $110 → $55/month (50% reduction)
```

**Total Savings:**
- Original: $500/month
- After optimizations: $55/month
- **89% cost reduction!**

**Summary of Techniques:**
1. Spot instances: 60% savings
2. Parquet format: 25% savings
3. Column pruning: 20% savings
4. Off-peak scheduling: 8% savings
5. Right-sizing: 50% savings (after other optimizations)

</details>

---

## 12. Summary and Self-Assessment

### Key Takeaways

**1. Pipeline Fundamentals**
- Pipelines move data from source → destination with transformations
- Key decision: Batch (latency ok) vs Streaming (need real-time)
- Lambda architecture = Batch + Speed layers
- Kappa architecture = Stream everything

**2. Spark Mastery**
- Use DataFrames (not RDDs)
- Transformations are lazy, actions trigger execution
- Partitioning is critical for performance
- Broadcast small tables in joins

**3. Optimization Techniques**
- Partition correctly (128MB-256MB per partition)
- Cache when reusing DataFrames
- Avoid shuffles (use broadcast, filter early)
- Use Parquet + Snappy compression
- Watch for data skew

**4. Architectural Patterns**
- Medallion (Bronze/Silver/Gold) for data refinement
- ETL vs ELT depends on warehouse capabilities
- CDC for real-time sync
- Event sourcing for audit/replay

**5. Cost Optimization**
- Use spot instances (70% cheaper)
- Parquet + compression (90% smaller)
- Column pruning (read less data)
- Right-size clusters
- Schedule off-peak

### Self-Assessment Checklist

**After completing this chapter, you should be able to:**

- [ ] Explain when to use batch vs streaming processing
- [ ] Design a Spark job with proper partitioning
- [ ] Optimize a slow Spark job
- [ ] Implement broadcast joins for small tables
- [ ] Handle data skew with salting
- [ ] Choose between ETL and ELT architectures
- [ ] Design a CDC pipeline
- [ ] Write an Airflow DAG for orchestration
- [ ] Implement Medallion architecture (Bronze/Silver/Gold)
- [ ] Reduce pipeline costs by 50%+
- [ ] Explain Parquet vs CSV trade-offs
- [ ] Use caching effectively
- [ ] Understand shuffle operations
- [ ] Design for horizontal scalability

### Common Interview Questions

**1. "How would you process 1TB of data daily?"**

<details>
<summary>Answer Framework</summary>

1. **Understand requirements:**
   - Latency? (Batch if hours ok, streaming if minutes)
   - Complexity? (Simple aggregations vs complex transformations)

2. **Architecture:**
   ```
   S3 → Spark (EMR/Databricks) → Parquet (partitioned) → Warehouse
   ```

3. **Key decisions:**
   - Batch processing (daily)
   - Partition data by date (easy to query ranges)
   - Use Parquet for compression
   - Right-size cluster: 1TB / 256MB = ~4000 partitions, 100 executors
   - Orchestrate with Airflow

4. **Cost optimization:**
   - Spot instances
   - Compress with Snappy
   - Schedule off-peak

</details>

**2. "When would you use streaming over batch?"**

<details>
<summary>Answer Framework</summary>

Use streaming when:
- Low latency required (< 5 minutes)
- Real-time decisions (fraud detection)
- Continuous data (IoT sensors, clickstreams)
- Time-sensitive alerts

Use batch when:
- Latency ok (hours/days)
- Complete dataset needed (historical analysis)
- Complex transformations
- Lower budget

Example: Fraud detection = streaming, daily reports = batch

</details>

**3. "How do you optimize a slow Spark join?"**

<details>
<summary>Answer Framework</summary>

1. **Check table sizes:**
   - Small + large → Broadcast join
   - Large + large → Check for skew

2. **Broadcast if possible:**
   ```python
   large.join(broadcast(small), "key")
   ```

3. **If skewed:**
   - Enable AQE (Spark 3.0+)
   - Or use salting

4. **Partition output:**
   ```python
   .write.partitionBy("date").parquet()
   ```

5. **Filter early:**
   ```python
   large.filter("date = '2024-01-15'").join(small, "key")
   ```

</details>

### Next Steps

**Practice:**
1. Implement a simple Spark job (read CSV, transform, write Parquet)
2. Create an Airflow DAG with dependencies
3. Design a pipeline for a real-world use case (e-commerce, IoT, etc.)
4. Optimize a slow job (try broadcast, caching, partitioning)

**DataDriven.io Problems:**
- Filter by "Pipeline Architecture" tag
- Start with "Easy" to learn concepts
- Progress to "Medium" for interview readiness
- Focus on:
  - Spark optimization questions
  - ETL design scenarios
  - Batch vs streaming decisions

**Further Reading:**
- "Spark: The Definitive Guide" by Bill Chambers
- "Designing Data-Intensive Applications" by Martin Kleppmann
- Apache Spark documentation (especially optimization guide)
- Databricks blog (practical tips)

---

**Congratulations!** You now understand data pipeline architecture at a level sufficient for mid-level data engineering interviews. Practice the concepts with DataDriven.io problems and you'll be ready to design production pipelines.

**Next Chapter:** Data Modeling (star schemas, normalization, SCD patterns)

---

**Chapter 6 Complete** | [Back to Main Guide](../README.md) | [Next: Chapter 7 - Data Modeling →](../07-Data-Modeling/Chapter-07-Data-Modeling.md)
