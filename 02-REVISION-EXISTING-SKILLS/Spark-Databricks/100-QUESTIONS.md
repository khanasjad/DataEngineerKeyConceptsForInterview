# Apache Spark & Databricks - 100 Interview Questions

**Your Context:** 8+ years at Optum/UnitedHealth Group | Architected Databricks + Delta Lake platform for RQNS

---

## FUNDAMENTALS (Q1-25) - Core Spark Concepts

### Q1: What is Apache Spark and why is it faster than MapReduce?

**Answer:**
Apache Spark is a unified analytics engine for large-scale data processing with built-in modules for SQL, streaming, machine learning, and graph processing.

**Why faster than MapReduce:**
- **In-memory processing**: Spark caches data in RAM between operations vs MapReduce writing to disk after each operation
- **DAG execution**: Optimizes entire pipeline vs MapReduce's rigid map-reduce stages
- **Lazy evaluation**: Optimizes before execution
- **10-100x faster** for iterative algorithms

**Your Optum Experience:**
"Migrated batch processing from MapReduce to Spark, reducing healthcare claims processing time from 4 hours to 25 minutes"

---

### Q2: Explain RDD, DataFrame, and Dataset. When would you use each?

**Answer:**

**RDD (Resilient Distributed Dataset):**
- Low-level API, immutable distributed collection
- Type-safe but no optimization
- Use for: Unstructured data, fine-grained control

```scala
val rdd = sc.textFile("hdfs://path/to/file")
  .map(line => line.split(","))
  .filter(arr => arr(0) == "ERROR")
```

**DataFrame:**
- Distributed collection with named columns (like SQL table)
- Catalyst optimizer, Tungsten execution engine
- Use for: Structured data, SQL operations, Python/R

```python
df = spark.read.parquet("s3a://bucket/claims/")
df.filter(df.status == "APPROVED").groupBy("provider_id").count()
```

**Dataset:**
- Type-safe DataFrame (Scala/Java only)
- Compile-time type checking + Catalyst optimization
- Use for: Type safety with optimization

```scala
case class Claim(id: String, amount: Double, status: String)
val ds = spark.read.parquet("path").as[Claim]
ds.filter(_.amount > 10000)
```

**Recommendation:** Use DataFrame for 90% of use cases. Dataset for type safety. RDD only when absolutely necessary.

---

### Q3: What is lazy evaluation in Spark? Why is it important?

**Answer:**

**Lazy Evaluation:**
Spark doesn't execute transformations immediately. It builds a DAG (execution plan) and only executes when an action is called.

**Transformations (lazy):** map, filter, join, groupBy
**Actions (eager):** count, collect, save, show

```python
# Nothing executes here - building DAG
df1 = spark.read.parquet("input1")
df2 = df1.filter(col("amount") > 1000)  # Lazy
df3 = df2.select("id", "amount")        # Lazy

# Execution happens here
df3.count()  # Action - entire DAG executes
```

**Why Important:**
1. **Optimization**: Spark optimizes entire pipeline (predicate pushdown, column pruning)
2. **Efficiency**: Combines operations to minimize data scanning
3. **Fault tolerance**: Can recompute from lineage

**Your Optum Example:**
```python
# Spark optimizes this entire pipeline before execution
claims_df = spark.read.parquet("s3a://optum/claims/")
filtered = claims_df.filter(col("claim_date") >= "2024-01-01")  # Lazy
aggregated = filtered.groupBy("provider_id").agg(sum("amount"))  # Lazy
result = aggregated.count()  # Action - Spark optimizes all above steps
```

---

### Q4: Explain Spark's execution model: Driver, Executors, and Cluster Manager.

**Answer:**

**Architecture:**

```
Driver Program (your code)
  ├─ SparkContext
  └─ DAG Scheduler
      ↓
Cluster Manager (YARN/Kubernetes/Databricks)
      ↓
Executors (worker nodes)
  ├─ Cache
  └─ Tasks (threads)
```

**Driver:**
- Runs main() function
- Creates SparkContext
- Converts code to DAG
- Schedules tasks to executors
- **Memory:** Typically 4-16GB

**Executors:**
- Run tasks assigned by driver
- Store data in cache/memory
- Report results back to driver
- **Memory:** 8-64GB per executor

**Cluster Manager:**
- YARN (Hadoop)
- Kubernetes
- Databricks (managed)
- Allocates resources

**Your Optum Setup:**
```python
spark = SparkSession.builder \
    .appName("RQNS-Claims-Processing") \
    .config("spark.driver.memory", "8g") \
    .config("spark.executor.memory", "32g") \
    .config("spark.executor.cores", "4") \
    .config("spark.executor.instances", "20") \
    .getOrCreate()

# 20 executors × 32GB = 640GB total executor memory
# Processed 500GB daily claims data
```

---

### Q5: What is a Spark DAG (Directed Acyclic Graph)?

**Answer:**

**DAG** = Execution plan showing sequence of transformations.

**Example:**
```python
# Code
df = spark.read.csv("input.csv")           # Stage 0
filtered = df.filter(col("age") > 21)       # Stage 0 (narrow)
counts = filtered.groupBy("city").count()   # Stage 1 (wide - shuffle)
counts.write.parquet("output")              # Stage 1

# DAG created:
# Stage 0: read → filter (narrow transformation, no shuffle)
#   ↓ (shuffle boundary)
# Stage 1: groupBy → count → write (wide transformation)
```

**Viewing DAG:**
```python
# In Spark UI: http://driver-node:4040
# Jobs → Stages → DAG Visualization
```

**Why Important:**
- Spark optimizes entire DAG before execution
- Identifies shuffle boundaries
- Plans task parallelism
- Helps debug performance issues

**Your Optum Experience:**
"Analyzed DAG visualization to identify expensive shuffle in claims processing pipeline. Reduced from 8 stages to 4 by reordering operations, cutting runtime by 40%"

---

### Q6: What are transformations vs actions in Spark?

**Answer:**

**Transformations (Lazy):**
- Return new RDD/DataFrame
- Build execution plan
- Examples: map, filter, join, groupBy

**Actions (Eager):**
- Trigger execution
- Return values or save data
- Examples: count, collect, save, show

```python
# Transformations
df1 = spark.read.parquet("input")                    # Not a transformation (data source)
df2 = df1.filter(col("status") == "APPROVED")        # Transformation (lazy)
df3 = df2.select("id", "amount")                     # Transformation (lazy)
df4 = df3.join(providers, "provider_id")             # Transformation (lazy)

# Nothing executed yet!

# Actions
count = df4.count()                                  # Action - triggers execution
df4.show(10)                                         # Action - triggers execution
df4.write.parquet("output")                          # Action - triggers execution
```

**Common Transformations:**
- Narrow: map, filter, select, union (no shuffle)
- Wide: groupBy, join, distinct, repartition (shuffle)

**Common Actions:**
- count(), collect(), take(n), show()
- save(), write.parquet(), write.csv()
- reduce(), foreach()

---

### Q7: What is partitioning in Spark? Why does it matter?

**Answer:**

**Partitioning** = Splitting data across executors for parallel processing.

**Default partitioning:**
- HDFS/S3 files: 1 partition per block (128MB)
- Post-shuffle: `spark.sql.shuffle.partitions` (default 200)

```python
# Check partitions
df = spark.read.parquet("s3a://bucket/claims/")
print(df.rdd.getNumPartitions())  # e.g., 450 partitions

# Repartition (full shuffle)
df_repartitioned = df.repartition(100)

# Coalesce (no shuffle, only reduce)
df_coalesced = df.coalesce(50)
```

**Why It Matters:**

**Too few partitions:**
- Underutilizes cluster (only few executors working)
- Large tasks → memory issues

**Too many partitions:**
- Task overhead
- Small files problem

**Optimal partitioning:**
- **Partition size:** 100MB - 200MB per partition
- **Partitions:** 2-4x number of executor cores

**Your Optum Configuration:**
```python
# 20 executors × 4 cores = 80 cores
# Target: 200 partitions (2.5x cores)
spark.conf.set("spark.sql.shuffle.partitions", "200")

# For 500GB data: 500GB / 200 partitions = 2.5GB per partition ❌ TOO LARGE

# Better:
spark.conf.set("spark.sql.shuffle.partitions", "2000")
# 500GB / 2000 = 250MB per partition ✅ OPTIMAL
```

---

### Q8: What is a shuffle in Spark? Why is it expensive?

**Answer:**

**Shuffle** = Redistributing data across partitions (network transfer).

**Operations causing shuffle:**
- groupBy, join, distinct
- repartition, sortBy
- aggregations (count, sum, avg)

**Why Expensive:**

1. **Disk I/O**: Write shuffle files to disk
2. **Network**: Transfer data between executors
3. **Serialization**: Serialize/deserialize data
4. **Creates stage boundary**: Breaks pipeline

```python
# No shuffle (narrow transformation)
df1 = spark.read.parquet("input")
df2 = df1.filter(col("age") > 21)        # Each partition processes independently
df3 = df2.select("id", "name")           # No data movement

# Shuffle (wide transformation)
df4 = df3.groupBy("city").count()        # Must gather all records per city
# Executor 1: [NYC records from all partitions]
# Executor 2: [LA records from all partitions]
```

**Minimizing Shuffle:**

```python
# ❌ Bad: Multiple shuffles
df.groupBy("provider_id").count() \
  .groupBy("state").sum("count")   # 2 shuffles

# ✅ Good: Single shuffle
df.groupBy("provider_id", "state").count()  # 1 shuffle
```

**Your Optum Optimization:**
"Reduced shuffle data from 200GB to 50GB by:
- Filter before join (remove 60% of records)
- Broadcast small dimension tables (<2GB)
- Partition by provider_id (pre-shuffle on common join key)
Result: 40% faster pipeline"

---

### Q9: What is broadcast join? When should you use it?

**Answer:**

**Broadcast Join** = Send small table to all executors to avoid shuffle.

**Standard Join (shuffle both sides):**
```python
# Both tables shuffled across network
large_claims.join(providers, "provider_id")  # Expensive if providers is small
```

**Broadcast Join (no shuffle):**
```python
from pyspark.sql.functions import broadcast

# Send providers table to all executors (must be <2GB)
large_claims.join(broadcast(providers), "provider_id")
```

**How It Works:**
```
Driver: Collect small table → Broadcast to all executors
Executor 1: Has full providers table in memory → Join locally
Executor 2: Has full providers table in memory → Join locally
# No shuffle!
```

**When to Use:**

✅ **Use broadcast when:**
- Small table < 2GB (configurable via `spark.sql.autoBroadcastJoinThreshold`)
- Joining large table (TB) with small dimension table (MB-GB)

❌ **Don't use when:**
- Both tables large
- Small table > executor memory

**Configuration:**
```python
# Auto-broadcast tables < 100MB
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 100 * 1024 * 1024)

# Disable auto-broadcast
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1)
```

**Your Optum Example:**
```python
# Claims: 500GB (100M records)
# Providers: 500MB (50K providers)

# ❌ Before: Sort-merge join, 200GB shuffle
claims.join(providers, "provider_id")

# ✅ After: Broadcast join, 0 shuffle
claims.join(broadcast(providers), "provider_id")

# Result: 5 min → 2 min (60% faster)
```

---

### Q10: Explain Spark's caching/persistence. When should you cache?

**Answer:**

**Caching** = Storing DataFrame/RDD in memory for reuse.

**Storage Levels:**

```python
from pyspark.storageLevels import StorageLevel

# Most common
df.cache()                              # MEMORY_AND_DISK (default)
df.persist(StorageLevel.MEMORY_ONLY)    # OOM if doesn't fit

# All options
MEMORY_ONLY          # Fast, OOM if too large
MEMORY_AND_DISK      # Spill to disk (recommended)
MEMORY_ONLY_SER      # Serialized (save memory, slower)
DISK_ONLY            # Slow
OFF_HEAP             # Tachyon/Alluxio
```

**When to Cache:**

✅ **Cache when:**
- DataFrame used multiple times
- Iterative algorithms (ML)
- After expensive transformations

❌ **Don't cache when:**
- Used only once
- Data too large (no memory left for processing)
- Already cached upstream

**Example:**
```python
# ❌ Bad: No caching, read from S3 3 times
df = spark.read.parquet("s3a://bucket/claims/")
count1 = df.filter(col("status") == "APPROVED").count()
count2 = df.filter(col("status") == "DENIED").count()
count3 = df.filter(col("amount") > 10000).count()
# Reads from S3: 3 times

# ✅ Good: Cache after read
df = spark.read.parquet("s3a://bucket/claims/").cache()
count1 = df.filter(col("status") == "APPROVED").count()  # Triggers cache
count2 = df.filter(col("status") == "DENIED").count()    # Uses cache
count3 = df.filter(col("amount") > 10000).count()        # Uses cache
# Reads from S3: 1 time, rest from memory
```

**Uncache when done:**
```python
df.unpersist()  # Free memory
```

**Your Optum Use Case:**
```python
# Provider lookup table used in 5 different joins
providers = spark.read.parquet("s3a://optum/providers/") \
    .select("provider_id", "name", "specialty", "network") \
    .cache()

# Used in 5 pipelines without re-reading from S3
claims_enriched = claims.join(providers, "provider_id")
denials_enriched = denials.join(providers, "provider_id")
# ... 3 more joins
```

---

### Q11: What is the Catalyst Optimizer?

**Answer:**

**Catalyst** = Spark SQL's query optimizer using rule-based and cost-based optimization.

**Optimization Phases:**

1. **Analysis**: Resolve column names, validate schema
2. **Logical Optimization**: Apply rules to improve plan
3. **Physical Planning**: Generate physical execution plans
4. **Code Generation**: Generate Java bytecode

**Common Optimizations:**

**Predicate Pushdown:**
```python
# Original query
df = spark.read.parquet("s3a://bucket/claims/")  # 500GB
filtered = df.filter(col("year") == 2024)

# Catalyst optimization:
# Push filter to Parquet reader → Only read 2024 partition (50GB)
# Reads: 50GB instead of 500GB
```

**Column Pruning:**
```python
# Only read needed columns from Parquet
df = spark.read.parquet("input")  # 100 columns
result = df.select("id", "amount")  # Only 2 columns

# Catalyst: Only reads id and amount columns from Parquet
```

**Constant Folding:**
```python
df.filter(col("amount") > 100 * 10)
# Optimized to:
df.filter(col("amount") > 1000)
```

**View Execution Plan:**
```python
df.explain(True)  # Logical and physical plans

# Or use extended mode
df.explain("extended")
df.explain("cost")      # Cost-based optimization
df.explain("formatted") # Pretty print
```

**Your Optum Example:**
```python
# Query
claims = spark.read.parquet("s3a://optum/claims/year=2024/month=01/")
result = claims.filter(col("status") == "APPROVED") \
    .select("claim_id", "amount") \
    .groupBy("provider_id").sum("amount")

result.explain(True)

# Catalyst optimizations applied:
# 1. Partition pruning: Only read year=2024/month=01
# 2. Column pruning: Only read claim_id, amount, provider_id, status
# 3. Predicate pushdown: Filter at Parquet reader level
# 4. Projection pushdown: Select columns early
# Result: Read 5GB instead of 500GB
```

---

### Q12: What is Tungsten execution engine?

**Answer:**

**Tungsten** = Spark's physical execution engine for CPU/memory efficiency.

**Key Features:**

**1. Memory Management:**
- Off-heap memory (bypass JVM GC)
- Binary format (no Java object overhead)
- Explicit memory management

**2. Code Generation:**
- Whole-stage code generation
- Generates custom Java bytecode for each query
- Eliminates virtual function calls

**3. Cache-aware computation:**
- Optimizes CPU cache usage
- Reduces memory bandwidth

**Example - Code Generation:**
```python
# Your query
df.filter(col("amount") > 1000).select("id", "amount")

# Tungsten generates:
# Instead of: filter() → select() (2 function calls)
# Generates single optimized function:
# for (row in data):
#     if (row.amount > 1000):
#         emit(row.id, row.amount)
```

**Performance Impact:**
- 2-5x faster than non-Tungsten
- 50% less memory

**Your Optum Experience:**
"Tungsten's whole-stage code generation reduced claim processing from 15min to 6min by eliminating intermediate materialization in 10-stage pipeline"

---

### Q13: What are narrow vs wide transformations?

**Answer:**

**Narrow Transformations:**
- Each input partition → Single output partition
- No shuffle, no network transfer
- Pipelined in single stage

Examples: map, filter, select, union, mapPartitions

```python
df.filter(col("age") > 21)     # Narrow
df.select("id", "name")         # Narrow
df.map(lambda x: x * 2)         # Narrow
```

**Wide Transformations:**
- Input partition → Multiple output partitions
- Requires shuffle
- Creates stage boundary

Examples: groupBy, join, distinct, repartition, sortBy

```python
df.groupBy("city").count()      # Wide
df1.join(df2, "id")             # Wide
df.distinct()                   # Wide
```

**Visualization:**
```
Narrow (no shuffle):
Partition 1 → Process → Partition 1'
Partition 2 → Process → Partition 2'
Partition 3 → Process → Partition 3'

Wide (shuffle):
Partition 1 ↘
Partition 2 → Shuffle → Partition A (all NYC records)
Partition 3 ↗           Partition B (all LA records)
```

**Your Code:**
```python
# Narrow chain (single stage, no shuffle)
df = spark.read.parquet("claims")
    .filter(col("year") == 2024)        # Narrow
    .select("id", "amount")              # Narrow
    .withColumn("tax", col("amount") * 0.1)  # Narrow

# Wide (creates new stage)
df.groupBy("provider_id").sum("amount")  # Wide - shuffle here
```

---

### Q14: How does Spark achieve fault tolerance?

**Answer:**

**Lineage-based Fault Tolerance:**

Spark tracks transformation lineage (DAG). If partition lost, recompute from source.

**RDD Lineage:**
```python
rdd1 = sc.textFile("input")
rdd2 = rdd1.map(lambda x: x.upper())
rdd3 = rdd2.filter(lambda x: "ERROR" in x)

# Lineage: input → map → filter

# If Executor 2 fails while processing rdd3:
# Spark recomputes only partition 2 of rdd3:
# read partition 2 of input → map → filter
```

**DataFrame Lineage:**
```python
df1 = spark.read.parquet("claims")
df2 = df1.filter(col("status") == "APPROVED")
df3 = df2.groupBy("provider_id").count()

# If task fails during groupBy:
# Recompute: read → filter → groupBy (only failed partition)
```

**Checkpointing (for long lineages):**
```python
# Save intermediate results to avoid re-computation from beginning
df2.checkpoint()  # Persist to HDFS/S3

# If failure after checkpoint:
# Read from checkpoint (don't recompute df1 → df2)
```

**Your Optum Example:**
"During 8-hour batch job, executor failure at hour 6. Spark recomputed only 15min of lost work instead of restarting entire job"

---

### Q15: What is speculative execution in Spark?

**Answer:**

**Speculative Execution** = Launching duplicate tasks for slow-running (straggler) tasks.

**Problem:**
One slow task delays entire stage (e.g., slow disk, bad executor).

**Solution:**
If task takes 1.5x median time, launch duplicate task on different executor. Use result from whichever finishes first.

**Configuration:**
```python
spark.conf.set("spark.speculation", "true")
spark.conf.set("spark.speculation.multiplier", "1.5")  # 1.5x median
spark.conf.set("spark.speculation.quantile", "0.75")   # After 75% tasks complete
```

**Example:**
```
Stage: 100 tasks
- 99 tasks complete in 1 minute
- 1 task (slow disk) taking 10 minutes

With speculation:
- After 99 tasks done, detect straggler
- Launch duplicate on healthy executor
- Finishes in 1 minute instead of 10
```

**When to Enable:**
- ✅ Heterogeneous clusters
- ✅ Cloud (variable instance performance)
- ❌ Small clusters (wastes resources)

**Your Optum Case:**
```python
# Enabled on AWS EMR cluster with spot instances
spark.conf.set("spark.speculation", "true")

# Result: 99th percentile runtime reduced from 45min → 25min
# Occasional slow spot instances no longer bottleneck
```

---

### Q16: What is dynamic resource allocation in Spark?

**Answer:**

**Dynamic Allocation** = Automatically scale executors based on workload.

**Static Allocation (default):**
```python
spark.conf.set("spark.executor.instances", "20")  # Fixed 20 executors
```

**Dynamic Allocation:**
```python
spark.conf.set("spark.dynamicAllocation.enabled", "true")
spark.conf.set("spark.dynamicAllocation.minExecutors", "5")
spark.conf.set("spark.dynamicAllocation.maxExecutors", "100")
spark.conf.set("spark.dynamicAllocation.initialExecutors", "10")

# Spark adds executors when tasks queue
# Removes idle executors after timeout
```

**How It Works:**
```
Load High → Add executors
Load Low  → Remove executors after idle timeout
```

**Benefits:**
- Better cluster utilization
- Cost savings (pay for what you use)
- Auto-scale for variable workloads

**Your Databricks Configuration:**
```python
# Databricks auto-scaling cluster
{
  "autoscale": {
    "min_workers": 2,
    "max_workers": 50
  },
  "cluster_name": "RQNS-Claims-Processing"
}

# During peak (8am-5pm): Scales to 40 workers
# During off-peak: Scales down to 2 workers
# Cost savings: 60% compared to fixed 40-worker cluster
```

---

### Q17: What is AQE (Adaptive Query Execution)?

**Answer:**

**Adaptive Query Execution** = Re-optimize execution plan during runtime based on statistics.

**Enabled in Spark 3.0+:**
```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
```

**Key Features:**

**1. Dynamically Coalescing Shuffle Partitions:**
```python
# Static partitions
spark.conf.set("spark.sql.shuffle.partitions", "200")  # Always 200

# AQE: Adjusts based on data size
# If shuffle output is 50MB → Coalesce to 10 partitions
# If shuffle output is 5GB → Keep 200 partitions
```

**2. Dynamically Switching Join Strategies:**
```python
# Plan: Sort-merge join (both tables looked large)
# Runtime: One table only 50MB after filter
# AQE: Switch to broadcast join (no shuffle!)
```

**3. Dynamically Optimizing Skew Joins:**
```python
# Skewed partition (provider_id=12345 has 80% of data)
# AQE: Split skewed partition into sub-partitions
# Process in parallel instead of single task
```

**Configuration:**
```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
```

**Your Optum Example:**
```python
# Query with filter selectivity unknown at planning time
claims = spark.read.parquet("claims")
providers = spark.read.parquet("providers")

result = claims.filter(col("diagnosis_code") == "Z23") \  # Unknown selectivity
    .join(providers, "provider_id")

# Without AQE: Plan for sort-merge join
# With AQE: Filter reduces claims to 500MB → Switch to broadcast join
# Result: 10min → 3min
```

---

### Q18: What are accumulators in Spark?

**Answer:**

**Accumulators** = Shared variables for aggregating values across executors (write-only from executors, read from driver).

**Use Cases:**
- Counters (errors, records processed)
- Debugging
- Monitoring

**Example:**
```python
# Create accumulator
error_count = sc.accumulator(0)

def process_record(record):
    try:
        # Process
        return transform(record)
    except Exception as e:
        error_count.add(1)  # Increment from executor
        return None

rdd = sc.textFile("input").map(process_record)
rdd.count()  # Trigger action

print(f"Errors: {error_count.value}")  # Read from driver
```

**DataFrame Example:**
```python
from pyspark.accumulators import AccumulatorParam

# Track record counts
approved_count = sc.accumulator(0)
denied_count = sc.accumulator(0)

def classify_claim(row):
    if row.status == "APPROVED":
        approved_count.add(1)
    elif row.status == "DENIED":
        denied_count.add(1)
    return row

claims.foreach(classify_claim)

print(f"Approved: {approved_count.value}, Denied: {denied_count.value}")
```

**Important:**
- Only use in actions (not transformations) for guaranteed counts
- Executors can only add (write-only)
- Driver can read value

**Your Optum Monitoring:**
```python
# Track data quality issues
null_provider = sc.accumulator(0)
invalid_amount = sc.accumulator(0)
duplicate_claims = sc.accumulator(0)

# Process 100M claims
claims.foreach(validate_and_count)

# Log metrics
print(f"Null providers: {null_provider.value}")
print(f"Invalid amounts: {invalid_amount.value}")
print(f"Duplicates: {duplicate_claims.value}")
```

---

### Q19: What are broadcast variables?

**Answer:**

**Broadcast Variables** = Read-only variables cached on each executor (avoid sending with every task).

**Without Broadcast (inefficient):**
```python
# Lookup table sent with EVERY task (1000 tasks × 100MB = 100GB network transfer)
lookup = {"A": 1, "B": 2, ...}  # 100MB dictionary

rdd.map(lambda x: lookup.get(x))  # Lookup sent 1000 times
```

**With Broadcast (efficient):**
```python
# Lookup sent ONCE per executor (10 executors × 100MB = 1GB network transfer)
lookup = {"A": 1, "B": 2, ...}
broadcast_lookup = sc.broadcast(lookup)

rdd.map(lambda x: broadcast_lookup.value.get(x))  # Lookup sent once per executor
```

**DataFrame Example:**
```python
# Mapping table
specialty_codes = {
    "01": "Primary Care",
    "02": "Cardiology",
    # ... 1000 codes
}

broadcast_codes = sc.broadcast(specialty_codes)

def enrich_provider(provider_id, specialty_code):
    specialty_name = broadcast_codes.value.get(specialty_code)
    return (provider_id, specialty_name)

df = claims.rdd.map(lambda x: enrich_provider(x.provider_id, x.specialty_code)) \
    .toDF(["provider_id", "specialty_name"])
```

**Cleanup:**
```python
broadcast_lookup.unpersist()  # Free memory
```

**Your Optum Use Case:**
```python
# ICD-10 diagnosis code lookup (200MB, 70K codes)
icd10_codes = spark.read.parquet("s3a://optum/reference/icd10/").collect()
icd10_dict = {row.code: row.description for row in icd10_codes}

broadcast_icd10 = sc.broadcast(icd10_dict)

# Enrich 100M claims
enriched_claims = claims.rdd.map(lambda x: (
    x.claim_id,
    x.diagnosis_code,
    broadcast_icd10.value.get(x.diagnosis_code)
))

# Without broadcast: 200MB × 1000 tasks = 200GB network
# With broadcast: 200MB × 20 executors = 4GB network
# 50x reduction in network transfer
```

---

### Q20: How do you handle skewed data in Spark?

**Answer:**

**Skew** = Uneven data distribution causing few partitions with most data.

**Problem:**
```python
# 1000 partitions, partition 1 has 80% of data
# 999 tasks finish in 1 min, task 1 takes 50 min
# Total time: 50 min (limited by slowest task)
```

**Solutions:**

**1. Salting (for skewed joins):**
```python
# Problem: provider_id=12345 has 80% of claims
claims_skewed = claims.withColumn("salt", (rand() * 10).cast("int"))
claims_salted = claims_skewed.withColumn(
    "provider_id_salted",
    concat(col("provider_id"), lit("_"), col("salt"))
)

# Replicate small table
providers_replicated = providers.withColumn("salt", explode(array([lit(i) for i in range(10)])))
providers_salted = providers_replicated.withColumn(
    "provider_id_salted",
    concat(col("provider_id"), lit("_"), col("salt"))
)

# Join on salted key (distributes skewed provider across 10 partitions)
result = claims_salted.join(providers_salted, "provider_id_salted")
```

**2. Adaptive Query Execution (AQE) - Spark 3.0+:**
```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes", "256MB")

# AQE automatically detects and splits skewed partitions
```

**3. Repartition by multiple columns:**
```python
# Instead of:
df.repartition("provider_id")  # Skewed

# Use:
df.repartition("provider_id", "claim_date")  # More even distribution
```

**4. Filter skewed keys separately:**
```python
# Split processing
normal_claims = claims.filter(col("provider_id") != "12345")
skewed_claims = claims.filter(col("provider_id") == "12345").repartition(100)

result = normal_claims.union(skewed_claims)
```

**Your Optum Fix:**
```python
# Problem: Top 10 providers (out of 50K) had 60% of claims
# Single partition took 45min while others took 2min

# Solution: AQE + Salting
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")

# For top 10 providers, added salt
top_providers = ["12345", "67890", ...]
claims_processed = claims.withColumn(
    "provider_id_final",
    when(col("provider_id").isin(top_providers),
         concat(col("provider_id"), lit("_"), (rand() * 20).cast("int")))
    .otherwise(col("provider_id"))
)

# Result: 45min → 8min (80% faster)
```

---

### Q21: What is the difference between cache() and persist()?

**Answer:**

**cache():**
- Shortcut for persist(StorageLevel.MEMORY_AND_DISK)
- Caches in memory, spills to disk if needed

**persist():**
- Allows specifying storage level

```python
from pyspark.storageLevels import StorageLevel

df.cache()  # Equivalent to:
df.persist(StorageLevel.MEMORY_AND_DISK)

# Other options:
df.persist(StorageLevel.MEMORY_ONLY)        # OOM if too large
df.persist(StorageLevel.DISK_ONLY)          # Slow
df.persist(StorageLevel.MEMORY_ONLY_SER)    # Serialized (less memory, slower)
df.persist(StorageLevel.MEMORY_AND_DISK_SER)
```

**When to use each:**

```python
# Small critical dataset used many times
df.persist(StorageLevel.MEMORY_ONLY)

# Large dataset, some reuse
df.cache()  # MEMORY_AND_DISK

# Very large, low reuse
df.persist(StorageLevel.DISK_ONLY)
```

**Checking cached data:**
```python
# Spark UI → Storage tab
# Shows: Size in memory, size on disk, number of partitions
```

---

### Q22: What is the difference between repartition() and coalesce()?

**Answer:**

**repartition(n):**
- Full shuffle
- Can increase or decrease partitions
- Even distribution

**coalesce(n):**
- No shuffle (just combines partitions)
- Only decrease partitions
- May be uneven

```python
df = spark.read.parquet("input")  # 1000 partitions

# Increase partitions (must use repartition)
df.repartition(2000)  # Shuffle

# Decrease partitions
df.repartition(100)   # Shuffle, even distribution
df.coalesce(100)      # No shuffle, may be uneven

# Example:
# repartition(2): Shuffle all data, create 2 even partitions
# coalesce(2): Combine partitions 1-500 → partition A, 501-1000 → partition B (no shuffle)
```

**When to use:**

```python
# ✅ Use coalesce() when:
# - Reducing partitions (e.g., before writing small files)
# - Want to avoid shuffle
df.coalesce(10).write.parquet("output")  # Write 10 files

# ✅ Use repartition() when:
# - Increasing partitions
# - Need even distribution
# - Partitioning by column
df.repartition(100, "provider_id")  # Even distribution by provider_id
```

**Your Optum Example:**
```python
# Problem: Writing 2000 small files (10MB each) to S3
# S3 list operations slow with many small files

# Solution:
df.coalesce(50).write.parquet("s3a://optum/claims/")
# Output: 50 files of 400MB each
# No shuffle overhead
```

---

### Q23: What are the different join types in Spark?

**Answer:**

**Join Types:**

```python
# Inner join (default)
df1.join(df2, "id")
df1.join(df2, "id", "inner")

# Left outer join
df1.join(df2, "id", "left")
df1.join(df2, "id", "left_outer")

# Right outer join
df1.join(df2, "id", "right")

# Full outer join
df1.join(df2, "id", "outer")
df1.join(df2, "id", "full")

# Left semi join (like SQL EXISTS)
df1.join(df2, "id", "left_semi")  # Only columns from df1

# Left anti join (like SQL NOT EXISTS)
df1.join(df2, "id", "left_anti")   # Rows in df1 not in df2

# Cross join (Cartesian product)
df1.crossJoin(df2)  # df1.count() × df2.count() rows
```

**Examples:**
```python
claims = spark.createDataFrame([
    (1, "C001", 100),
    (2, "C002", 200),
    (3, "C003", 300)
], ["id", "claim_id", "amount"])

providers = spark.createDataFrame([
    (1, "Dr. Smith"),
    (2, "Dr. Jones"),
    (4, "Dr. Brown")
], ["id", "name"])

# Inner: Only matching IDs (1, 2)
claims.join(providers, "id", "inner").show()
# id=1, id=2 (2 rows)

# Left: All claims + matched providers (1, 2, 3)
claims.join(providers, "id", "left").show()
# id=1 (Dr. Smith), id=2 (Dr. Jones), id=3 (null)

# Right: All providers + matched claims (1, 2, 4)
claims.join(providers, "id", "right").show()
# id=1, id=2, id=4 (amount=null)

# Left semi: Claims with matching provider (1, 2)
claims.join(providers, "id", "left_semi").show()
# Only claim columns, id=1, id=2

# Left anti: Claims without matching provider (3)
claims.join(providers, "id", "left_anti").show()
# Only id=3
```

**Your Optum Use Cases:**
```python
# Find claims without providers (data quality check)
orphan_claims = claims.join(providers, "provider_id", "left_anti")

# Enrich claims with provider info (keep all claims)
enriched = claims.join(providers, "provider_id", "left")

# Find only claims with valid providers
valid_claims = claims.join(providers, "provider_id", "left_semi")
```

---

### Q24: What is the difference between map() and flatMap()?

**Answer:**

**map():**
- 1 input → 1 output
- Transforms each element

**flatMap():**
- 1 input → 0 or more outputs
- Flattens nested structure

```python
data = ["hello world", "foo bar"]

# map: 1 → 1
rdd.map(lambda x: x.split())
# Output: [["hello", "world"], ["foo", "bar"]]

# flatMap: 1 → many
rdd.flatMap(lambda x: x.split())
# Output: ["hello", "world", "foo", "bar"]
```

**DataFrame Example:**
```python
# map
df = spark.createDataFrame([(1,), (2,), (3,)], ["num"])
df.rdd.map(lambda x: x.num * 2).collect()
# [2, 4, 6]

# flatMap
df.rdd.flatMap(lambda x: range(x.num)).collect()
# [0, 0, 1, 0, 1, 2]
```

**Your Optum Use Case:**
```python
# Parse multi-diagnosis claims (each claim can have multiple diagnosis codes)
claims = spark.createDataFrame([
    ("C001", "E11.9,I10,Z23"),  # 3 diagnosis codes
    ("C002", "J44.0"),           # 1 diagnosis code
], ["claim_id", "diagnosis_codes"])

# flatMap to create one row per diagnosis
exploded = claims.rdd.flatMap(lambda row: [
    (row.claim_id, code.strip())
    for code in row.diagnosis_codes.split(",")
]).toDF(["claim_id", "diagnosis_code"])

exploded.show()
# C001, E11.9
# C001, I10
# C001, Z23
# C002, J44.0
```

---

### Q25: What are the different cluster managers supported by Spark?

**Answer:**

**Cluster Managers:**

**1. Standalone (built-in):**
- Simple cluster manager bundled with Spark
- Good for development/small clusters
- Easy setup

**2. YARN (Hadoop):**
- Most common in enterprise
- Integrates with Hadoop ecosystem
- Resource sharing with other YARN apps

**3. Kubernetes:**
- Container orchestration
- Cloud-native
- Dynamic resource allocation

**4. Mesos (deprecated):**
- Apache Mesos
- Being phased out

**5. Local:**
- Single machine
- Development/testing

**Configuration:**

```python
# Local
spark = SparkSession.builder.master("local[4]").getOrCreate()

# Standalone
spark = SparkSession.builder.master("spark://master:7077").getOrCreate()

# YARN
spark = SparkSession.builder.master("yarn") \
    .config("spark.submit.deployMode", "cluster")  # or "client"
    .getOrCreate()

# Kubernetes
spark = SparkSession.builder.master("k8s://https://kubernetes:443") \
    .config("spark.kubernetes.container.image", "spark:3.5.0") \
    .getOrCreate()
```

**Your Optum Environment:**
```python
# On-prem Hadoop cluster: YARN
spark-submit \
    --master yarn \
    --deploy-mode cluster \
    --num-executors 20 \
    --executor-memory 32g \
    --executor-cores 4 \
    rqns_pipeline.py

# AWS EMR: YARN
# Azure Databricks: Managed (abstracted)
# Azure AKS: Kubernetes for containerized workloads
```

---

## INTERMEDIATE (Q26-60) - Performance & Optimization

### Q26: How do you optimize Spark join performance?

**Answer:**

**Join Optimization Strategies:**

**1. Broadcast small tables (<2GB):**
```python
large_claims.join(broadcast(small_providers), "provider_id")
```

**2. Partition by join key:**
```python
# Pre-partition both tables by join key
claims.write.partitionBy("provider_id").parquet("claims_partitioned")
providers.write.partitionBy("provider_id").parquet("providers_partitioned")

# Join reads only matching partitions
claims_part = spark.read.parquet("claims_partitioned")
providers_part = spark.read.parquet("providers_partitioned")
claims_part.join(providers_part, "provider_id")  # Faster
```

**3. Bucketing:**
```python
# Pre-shuffle and bucket data
claims.write.bucketBy(100, "provider_id").saveAsTable("claims_bucketed")
providers.write.bucketBy(100, "provider_id").saveAsTable("providers_bucketed")

# Join without shuffle
spark.table("claims_bucketed").join(spark.table("providers_bucketed"), "provider_id")
```

**4. Filter before join:**
```python
# ❌ Bad
claims.join(providers, "provider_id").filter(col("claim_date") > "2024-01-01")

# ✅ Good
claims.filter(col("claim_date") > "2024-01-01").join(providers, "provider_id")
```

**5. Use appropriate join type:**
```python
# If you only need columns from left table
claims.join(providers, "provider_id", "left_semi")  # Faster than inner join + select
```

**6. Salting for skewed joins:**
```python
# Covered in Q20
```

**Your Optum Optimization:**
```python
# Before: 500GB claims ⋈ 5GB providers = 45min
claims.join(providers, "provider_id")

# After: 15min (3x faster)
# 1. Filter claims first (500GB → 200GB)
filtered_claims = claims.filter(col("claim_date") >= "2024-01-01")

# 2. Broadcast providers (5GB → 500MB after selecting columns)
small_providers = providers.select("provider_id", "name", "network")
optimized = filtered_claims.join(broadcast(small_providers), "provider_id")

# 3. Enable AQE for runtime optimization
spark.conf.set("spark.sql.adaptive.enabled", "true")
```

---

### Q27: What metrics do you monitor in Spark UI?

**Answer:**

**Spark UI Tabs:**

**1. Jobs Tab:**
- Number of stages
- Duration per stage
- Failed jobs

**2. Stages Tab:**
- Task count per stage
- Duration
- Shuffle read/write
- Skewed tasks (outliers)

**3. Storage Tab:**
- Cached RDDs/DataFrames
- Memory used
- Disk spill

**4. Executors Tab:**
- Active executors
- Memory/disk usage per executor
- GC time
- Failed tasks

**5. SQL Tab:**
- Query execution time
- DAG visualization
- Physical plan

**Key Metrics:**

```
Stage duration: Should be balanced (no single stage taking 80% of time)
Shuffle read/write: Minimize shuffle (expensive)
GC time: Should be <10% of task time
Spill to disk: Indicates memory pressure
Task duration: Check for stragglers (skew)
```

**Your Optum Monitoring:**
```python
# Key issues identified in Spark UI:

# 1. Shuffle size
# Problem: Stage 3 shuffle write 200GB
# Fix: Filter earlier, reduce shuffle to 50GB

# 2. GC time
# Problem: Executor GC time 30% of runtime
# Fix: Increase executor memory 16GB → 32GB, reduce GC to 5%

# 3. Spill to disk
# Problem: 100GB memory spill to disk
# Fix: Increase spark.memory.fraction from 0.6 to 0.8

# 4. Skewed tasks
# Problem: 1 task takes 40min, others 2min
# Fix: Enable AQE skew join optimization

# Result: 60min → 25min (58% improvement)
```

**Accessing Spark UI:**
```bash
# Local: http://localhost:4040
# YARN: http://resource-manager:8088
# Databricks: Cluster → Spark UI
# History Server: http://history-server:18080
```

---

### Q28: How do you tune Spark memory configuration?

**Answer:**

**Spark Memory Model:**

```
Executor Memory (e.g., 32GB)
├─ Reserved (300MB, hardcoded)
└─ Usable Memory (31.7GB)
    ├─ Spark Memory (spark.memory.fraction = 0.6) → 19GB
    │   ├─ Storage (cache/persist) (0.5) → 9.5GB
    │   └─ Execution (shuffles, joins) (0.5) → 9.5GB
    └─ User Memory (0.4) → 12.7GB
        └─ User data structures, UDFs
```

**Key Configurations:**

```python
# Executor memory
spark.conf.set("spark.executor.memory", "32g")

# Memory fraction for Spark operations
spark.conf.set("spark.memory.fraction", "0.6")  # Default 0.6

# Storage vs execution split (dynamic, but initial)
spark.conf.set("spark.memory.storageFraction", "0.5")

# Off-heap memory (avoid GC)
spark.conf.set("spark.memory.offHeap.enabled", "true")
spark.conf.set("spark.memory.offHeap.size", "10g")

# Overhead memory for containers (YARN/K8s)
spark.conf.set("spark.executor.memoryOverhead", "4g")
```

**Memory Pressure Symptoms:**
- Spill to disk (check Spark UI)
- High GC time (>10%)
- OOM errors

**Solutions:**

```python
# 1. Increase executor memory
spark.conf.set("spark.executor.memory", "64g")  # Was 32g

# 2. Increase memory fraction (if lots of shuffles/joins)
spark.conf.set("spark.memory.fraction", "0.8")  # Was 0.6

# 3. Reduce cached data
df.unpersist()  # Free storage memory

# 4. Partition data better (smaller partitions)
spark.conf.set("spark.sql.shuffle.partitions", "1000")  # Was 200

# 5. Use off-heap memory
spark.conf.set("spark.memory.offHeap.enabled", "true")
spark.conf.set("spark.memory.offHeap.size", "20g")
```

**Your Optum Tuning:**
```python
# Problem: Processing 500GB claims with 100GB spill to disk

# Before:
spark.conf.set("spark.executor.memory", "16g")
spark.conf.set("spark.executor.instances", "20")
spark.conf.set("spark.memory.fraction", "0.6")
# Total executor memory: 20 × 16GB = 320GB
# Spark memory: 320GB × 0.6 = 192GB
# Not enough for 500GB workload → Spill

# After:
spark.conf.set("spark.executor.memory", "32g")
spark.conf.set("spark.executor.instances", "20")
spark.conf.set("spark.memory.fraction", "0.75")
spark.conf.set("spark.memory.offHeap.enabled", "true")
spark.conf.set("spark.memory.offHeap.size", "10g")
# Total executor memory: 20 × (32GB + 10GB) = 840GB
# Spark memory: 20 × (32GB × 0.75 + 10GB) = 680GB
# Sufficient for 500GB → No spill

# Result: 60min → 25min, 0GB spill
```

---

### Q29: What is the difference between client vs cluster deploy mode?

**Answer:**

**Client Mode:**
- Driver runs on client machine (where spark-submit runs)
- Executors run on cluster

**Cluster Mode:**
- Driver runs on cluster (on a worker node)
- Executors run on cluster

```bash
# Client mode
spark-submit --deploy-mode client app.py
# Driver: Local machine
# Executors: Cluster nodes

# Cluster mode
spark-submit --deploy-mode cluster app.py
# Driver: Cluster node
# Executors: Cluster nodes
```

**When to Use:**

**Client Mode:**
- ✅ Interactive work (pyspark shell, notebooks)
- ✅ Local development
- ✅ Need fast feedback
- ❌ Network latency if client far from cluster
- ❌ Client must stay up for duration

**Cluster Mode:**
- ✅ Production batch jobs
- ✅ Long-running jobs
- ✅ Client can disconnect after submission
- ✅ Driver closer to executors (low latency)
- ❌ Harder to debug (logs on cluster)

**Your Optum Setup:**

```bash
# Development (Databricks notebook): Client mode
# Driver runs in notebook, close to data

# Production (Airflow scheduled jobs): Cluster mode
spark-submit \
    --master yarn \
    --deploy-mode cluster \
    --conf spark.driver.memory=8g \
    --conf spark.executor.memory=32g \
    --conf spark.executor.instances=20 \
    rqns_daily_pipeline.py

# Airflow can submit and disconnect
# Job runs independently on cluster
```

---

### Q30: How do you handle small files problem in Spark?

**Answer:**

**Problem:**
Many small files (<128MB) cause:
- Too many tasks (overhead)
- Slow metadata operations (listing files)
- Inefficient storage

**Solutions:**

**1. Coalesce before writing:**
```python
# ❌ Bad: 10,000 small files (10MB each)
df.write.parquet("output")

# ✅ Good: 50 larger files (2GB each)
df.coalesce(50).write.parquet("output")
```

**2. Repartition by date/partition key:**
```python
df.repartition("year", "month") \
    .write.partitionBy("year", "month") \
    .parquet("output")
# Fewer files per partition
```

**3. Use maxRecordsPerFile:**
```python
df.write.option("maxRecordsPerFile", 100000) \
    .parquet("output")
# Limits records per file (prevents huge files)
```

**4. Periodic compaction:**
```python
# Merge small files weekly
small_files = spark.read.parquet("input_with_small_files")
small_files.coalesce(10).write.mode("overwrite").parquet("compacted")
```

**5. Use Delta Lake auto-optimize:**
```python
# Delta Lake auto-compaction
df.write.format("delta") \
    .option("delta.autoOptimize.optimizeWrite", "true") \
    .option("delta.autoOptimize.autoCompact", "true") \
    .save("delta_table")
```

**Your Optum Fix:**
```python
# Problem: Daily incremental writes created 1000 small files/day
# After 1 year: 365K small files
# Listing files took 10 minutes!

# Solution 1: Coalesce daily writes
daily_claims.coalesce(10) \
    .write.mode("append") \
    .partitionBy("claim_date") \
    .parquet("s3a://optum/claims/")

# Solution 2: Weekly compaction job
old_data = spark.read.parquet("s3a://optum/claims/year=2024/month=01/")
old_data.coalesce(100) \
    .write.mode("overwrite") \
    .parquet("s3a://optum/claims_compacted/year=2024/month=01/")

# Solution 3: Migrate to Delta Lake
df.write.format("delta") \
    .option("delta.autoOptimize.optimizeWrite", "true") \
    .partitionBy("year", "month") \
    .save("s3a://optum/claims_delta/")

# Run OPTIMIZE monthly
spark.sql("OPTIMIZE delta.`s3a://optum/claims_delta/`")

# Result: 365K files → 3.6K files (100x reduction)
# Listing time: 10min → 5sec
```

---

### Q31: Explain Spark's shuffle mechanism in detail.

**Answer:**

**Shuffle** = Redistributing data across partitions.

**Shuffle Phases:**

**1. Shuffle Write (Map Side):**
- Each task writes shuffle files to local disk
- Creates `shuffle_X_Y_Z` files (X=shuffle_id, Y=map_id, Z=reduce_id)
- Hash-based or sort-based partitioning

**2. Shuffle Read (Reduce Side):**
- Tasks fetch shuffle files from other executors
- Reads data over network
- Combines data for reduce operation

**Internals:**
```
Executor 1:                  Executor 2:
Task 1 (partition 1)         Task 3 (partition 3)
  ├─ Write shuffle_0_1_0      ├─ Read from all executors
  ├─ Write shuffle_0_1_1      │   ├─ shuffle_0_1_0
  └─ Write shuffle_0_1_2      │   ├─ shuffle_0_2_0
                              │   └─ shuffle_0_3_0
Task 2 (partition 2)         Task 4 (partition 4)
  ├─ Write shuffle_0_2_0      ├─ Read from all executors
  ├─ Write shuffle_0_2_1      ...
  └─ Write shuffle_0_2_2
```

**Configuration:**

```python
# Shuffle partitions (post-shuffle)
spark.conf.set("spark.sql.shuffle.partitions", "200")

# Shuffle compression
spark.conf.set("spark.shuffle.compress", "true")
spark.conf.set("spark.shuffle.spill.compress", "true")

# Shuffle manager
spark.conf.set("spark.shuffle.manager", "sort")  # Default since 1.2

# Network timeout
spark.conf.set("spark.network.timeout", "600s")

# Shuffle file buffer
spark.conf.set("spark.shuffle.file.buffer", "64k")
```

**Monitoring Shuffle:**
```python
# Spark UI → Stages → Shuffle Read/Write
# Look for:
# - Shuffle Write: How much data written to disk
# - Shuffle Read: How much data read over network
# - Spill: Memory overflow to disk
```

**Your Optum Optimization:**
```python
# Query: Group 500GB claims by provider_id
df.groupBy("provider_id").agg(sum("amount"))

# Shuffle metrics (before):
# Shuffle Write: 500GB
# Shuffle Read: 500GB
# Duration: 45 min

# Optimization 1: Filter first
df.filter(col("claim_date") >= "2024-01-01")  # 500GB → 200GB
  .groupBy("provider_id").agg(sum("amount"))

# Shuffle Write: 200GB (60% reduction)

# Optimization 2: Reduce shuffle partitions
spark.conf.set("spark.sql.shuffle.partitions", "500")  # Was 200
# Partition size: 200GB / 500 = 400MB per partition (optimal)

# Optimization 3: Enable compression
spark.conf.set("spark.shuffle.compress", "true")
spark.conf.set("spark.io.compression.codec", "lz4")  # Fast

# Result:
# Shuffle Write: 80GB (compressed from 200GB)
# Shuffle Read: 80GB
# Duration: 15 min (3x faster)
```

---

### Q32: What is predicate pushdown? How does it work?

**Answer:**

**Predicate Pushdown** = Pushing filters down to data source to read less data.

**How It Works:**

**Without Pushdown:**
```python
# Read entire 500GB Parquet file
df = spark.read.parquet("claims")  # 500GB

# Filter in Spark
filtered = df.filter(col("year") == 2024)  # 50GB after filter
# Read: 500GB ❌
```

**With Pushdown:**
```python
# Filter pushed to Parquet reader
df = spark.read.parquet("claims")
filtered = df.filter(col("year") == 2024)

# Parquet reader only reads year=2024 row groups
# Read: 50GB ✅
```

**Data Sources Supporting Pushdown:**
- Parquet ✅ (column pruning + predicate pushdown)
- ORC ✅
- JDBC ✅ (pushes SQL WHERE clause to database)
- Delta Lake ✅
- CSV ❌ (must read entire file)

**Example:**

```python
# JDBC pushdown
jdbc_df = spark.read.jdbc(
    url="jdbc:postgresql://db:5432/claims",
    table="claims",
    properties={"user": "admin", "password": "pass"}
)

# Filter pushed to PostgreSQL
filtered = jdbc_df.filter(col("claim_date") > "2024-01-01")

# Spark sends to PostgreSQL:
# SELECT * FROM claims WHERE claim_date > '2024-01-01'
# Only filtered data transferred over network
```

**Parquet Pushdown:**
```python
# Parquet file structure:
# Row Group 1: year=2023
# Row Group 2: year=2024
# Row Group 3: year=2024

df = spark.read.parquet("claims.parquet")
filtered = df.filter(col("year") == 2024)

# Spark reads only Row Groups 2 and 3 (skips Row Group 1)
```

**Verification:**
```python
df.explain(True)

# Look for "PushedFilters" in output:
# PushedFilters: [IsNotNull(year), EqualTo(year,2024)]
# Means: Filter pushed to data source
```

**Your Optum Example:**
```python
# Partitioned by year/month
# s3://optum/claims/year=2024/month=01/
# s3://optum/claims/year=2024/month=02/
# ...

df = spark.read.parquet("s3a://optum/claims/")

# Query recent claims
recent = df.filter(
    (col("year") == 2024) &
    (col("month") >= 6) &
    (col("status") == "APPROVED")
)

# Spark optimizations:
# 1. Partition pruning: Only read year=2024/month=06,07,08,... directories
#    (500GB → 100GB)
# 2. Predicate pushdown: Push status filter to Parquet reader
#    (100GB → 40GB row groups with APPROVED claims)
# 3. Column pruning: Only read needed columns

recent.explain("extended")
# PartitionFilters: [year=2024, month>=6]
# PushedFilters: [IsNotNull(status), EqualTo(status, APPROVED)]

# Result: Read 40GB instead of 500GB (92% reduction)
```

---

### Q33: What is column pruning in Spark?

**Answer:**

**Column Pruning** = Reading only required columns from columnar storage (Parquet/ORC).

**How It Works:**

```python
# Parquet stores columns separately:
# claim_id.parquet (1GB)
# amount.parquet (500MB)
# status.parquet (200MB)
# provider_id.parquet (300MB)
# diagnosis.parquet (5GB)
# ... 50 more columns (total 100GB)

# Without column pruning (read all columns)
df = spark.read.parquet("claims")
result = df.select("claim_id", "amount")  # Only need 2 columns
# Reads: 100GB ❌

# With column pruning (Catalyst optimizer)
df = spark.read.parquet("claims")
result = df.select("claim_id", "amount")
# Spark pushes projection to Parquet reader
# Reads: 1.5GB (only claim_id + amount) ✅
```

**Combined with Predicate Pushdown:**
```python
df = spark.read.parquet("claims")  # 100 columns, 500GB

result = df.filter(col("year") == 2024) \  # Predicate pushdown
    .select("claim_id", "amount")           # Column pruning

# Spark optimizes:
# 1. Only read year=2024 partitions (500GB → 50GB)
# 2. Only read claim_id and amount columns (50GB → 750MB)
# Final read: 750MB ✅
```

**Verification:**
```python
df.explain("extended")

# Output includes:
# ReadSchema: struct<claim_id:string, amount:double>
# This confirms only 2 columns read from Parquet
```

**When It Doesn't Work:**
```python
# ❌ Select all columns
df.select("*")  # Reads all columns

# ❌ CSV (must read entire file)
spark.read.csv("claims.csv").select("claim_id", "amount")  # Still reads all columns

# ❌ UDF on entire row
df.rdd.map(lambda row: my_udf(row))  # Must read all columns
```

**Your Optum Optimization:**
```python
# Table: 150 columns, 1TB total
# Needed: 5 columns for report

# ❌ Before: Read all columns
df = spark.table("claims")
report = df.groupBy("provider_id").agg(
    sum("amount"),
    count("claim_id")
)
# Read: 1TB

# ✅ After: Column pruning
df = spark.table("claims").select(
    "provider_id",
    "amount",
    "claim_id"
)
report = df.groupBy("provider_id").agg(
    sum("amount"),
    count("claim_id")
)
# Read: 30GB (97% reduction)

# Result: 30min → 3min
```

---

### Q34: What is Delta Lake? Why use it over Parquet?

**Answer:**

**Delta Lake** = Open-source storage layer (built on Parquet) with ACID transactions, versioning, and schema enforcement.

**Parquet vs Delta Lake:**

| Feature | Parquet | Delta Lake |
|---------|---------|------------|
| ACID transactions | ❌ | ✅ |
| Time travel | ❌ | ✅ |
| Schema evolution | ❌ | ✅ |
| UPSERT/MERGE | ❌ | ✅ |
| File management | Manual | Automatic |

**Delta Lake Features:**

**1. ACID Transactions:**
```python
# Parquet: Race condition possible
df1.write.mode("append").parquet("claims")  # Writer 1
df2.write.mode("append").parquet("claims")  # Writer 2 (concurrent)
# Risk: Corrupted data

# Delta Lake: Safe concurrent writes
df1.write.format("delta").mode("append").save("claims_delta")
df2.write.format("delta").mode("append").save("claims_delta")
# Atomic commits using transaction log
```

**2. Time Travel:**
```python
# Read historical version
df = spark.read.format("delta") \
    .option("versionAsOf", 5) \
    .load("claims_delta")

# Read data as of timestamp
df = spark.read.format("delta") \
    .option("timestampAsOf", "2024-01-15") \
    .load("claims_delta")

# View history
spark.sql("DESCRIBE HISTORY delta.`/path/to/claims_delta`").show()
```

**3. UPSERT/MERGE:**
```python
from delta.tables import DeltaTable

# MERGE (UPSERT)
delta_claims = DeltaTable.forPath(spark, "claims_delta")

delta_claims.alias("target").merge(
    new_claims.alias("source"),
    "target.claim_id = source.claim_id"
).whenMatchedUpdate(set={
    "status": "source.status",
    "amount": "source.amount"
}).whenNotMatchedInsert(values={
    "claim_id": "source.claim_id",
    "status": "source.status",
    "amount": "source.amount"
}).execute()
```

**4. Schema Enforcement + Evolution:**
```python
# Schema enforcement (reject bad data)
df.write.format("delta").save("claims_delta")
bad_df = spark.createDataFrame([...], schema_with_extra_column)
bad_df.write.format("delta").mode("append").save("claims_delta")  # ERROR

# Schema evolution (allow new columns)
bad_df.write.format("delta") \
    .option("mergeSchema", "true") \
    .mode("append") \
    .save("claims_delta")  # SUCCESS
```

**5. Auto-Optimize:**
```python
# Compaction + Z-ordering
spark.sql("OPTIMIZE delta.`/path/to/claims_delta` ZORDER BY (provider_id)")

# Auto-compact on write
df.write.format("delta") \
    .option("delta.autoOptimize.optimizeWrite", "true") \
    .option("delta.autoOptimize.autoCompact", "true") \
    .save("claims_delta")
```

**6. DELETE support:**
```python
delta_table = DeltaTable.forPath(spark, "claims_delta")

# Delete old data
delta_table.delete("claim_date < '2020-01-01'")

# Parquet: Must rewrite entire table
```

**Your Optum Migration:**
```python
# Before (Parquet):
# - 365K small files
# - No ACID (occasional data corruption)
# - No schema validation (bad data sneaks in)
# - Hard to update/delete records
# - Manual compaction jobs

# After (Delta Lake):
spark.sql("""
    CREATE TABLE claims_delta
    USING DELTA
    PARTITIONED BY (year, month)
    LOCATION 's3a://optum/claims_delta/'
    AS SELECT * FROM parquet.`s3a://optum/claims_parquet/`
""")

# Enable auto-optimize
spark.sql("""
    ALTER TABLE claims_delta
    SET TBLPROPERTIES (
        'delta.autoOptimize.optimizeWrite' = 'true',
        'delta.autoOptimize.autoCompact' = 'true'
    )
""")

# Benefits:
# 1. ACID: No more data corruption
# 2. MERGE: Easy upserts for late-arriving data
# 3. DELETE: Easy GDPR compliance (delete patient data)
# 4. Time travel: Recover from bad writes
# 5. Auto-compact: 365K → 3.6K files automatically

# Example: Fix bad data load
spark.sql("RESTORE TABLE claims_delta TO VERSION AS OF 10")
```

---

### Q35: How do you implement slowly changing dimensions (SCD) in Spark?

**Answer:**

**SCD Type 2** (Most common) = Track historical changes with start/end dates.

**Implementation:**

```python
from delta.tables import DeltaTable
from pyspark.sql.functions import *

# Existing dimension table (Delta)
dim_provider = DeltaTable.forPath(spark, "s3a://optum/dim_provider_delta/")

# New provider data
new_providers = spark.createDataFrame([
    ("P001", "Dr. Smith", "Cardiology", "Network A"),  # Address changed
    ("P002", "Dr. Jones", "Oncology", "Network B"),    # Unchanged
    ("P003", "Dr. Brown", "Surgery", "Network A"),     # New provider
], ["provider_id", "name", "specialty", "network"])

# Add effective dates
new_providers_with_dates = new_providers \
    .withColumn("effective_date", current_date()) \
    .withColumn("end_date", lit(None).cast("date")) \
    .withColumn("is_current", lit(True))

# SCD Type 2 MERGE
dim_provider.alias("target").merge(
    new_providers_with_dates.alias("source"),
    "target.provider_id = source.provider_id AND target.is_current = true"
).whenMatchedUpdate(
    condition="target.name != source.name OR target.specialty != source.specialty OR target.network != source.network",
    set={
        "end_date": "source.effective_date",
        "is_current": "false"
    }
).whenNotMatchedInsert(
    values={
        "provider_id": "source.provider_id",
        "name": "source.name",
        "specialty": "source.specialty",
        "network": "source.network",
        "effective_date": "source.effective_date",
        "end_date": "source.end_date",
        "is_current": "source.is_current"
    }
).execute()

# Insert new versions for changed records
changed_providers = new_providers_with_dates.alias("new") \
    .join(
        dim_provider.toDF().filter(col("is_current") == True).alias("old"),
        "provider_id"
    ).filter(
        (col("old.name") != col("new.name")) |
        (col("old.specialty") != col("new.specialty")) |
        (col("old.network") != col("new.network"))
    ).select(
        col("new.provider_id"),
        col("new.name"),
        col("new.specialty"),
        col("new.network"),
        col("new.effective_date"),
        col("new.end_date"),
        col("new.is_current")
    )

changed_providers.write.format("delta").mode("append").save("s3a://optum/dim_provider_delta/")
```

**Simplified Version (using MERGE only):**

```python
# Single MERGE for SCD Type 2
from pyspark.sql.functions import *

# Add surrogate key
new_data = new_providers.withColumn("sk", monotonically_increasing_id())

staged = new_data.selectExpr(
    "NULL as merge_key",
    "provider_id",
    "name",
    "specialty",
    "network"
).unionByName(
    new_data.selectExpr(
        "provider_id as merge_key",
        "provider_id",
        "name",
        "specialty",
        "network"
    )
)

dim_provider.alias("target").merge(
    staged.alias("source"),
    "target.provider_id = source.merge_key AND target.is_current = true"
).whenMatchedUpdate(
    condition="""
        source.merge_key IS NOT NULL AND
        (target.name != source.name OR
         target.specialty != source.specialty OR
         target.network != source.network)
    """,
    set={
        "is_current": "false",
        "end_date": "current_date()"
    }
).whenNotMatchedInsert(
    condition="source.merge_key IS NULL",
    values={
        "provider_id": "source.provider_id",
        "name": "source.name",
        "specialty": "source.specialty",
        "network": "source.network",
        "effective_date": "current_date()",
        "is_current": "true"
    }
).execute()
```

**Your Optum SCD Implementation:**
```python
# Track provider network changes (important for claims reprocessing)

# Before: Overwrite dimension (lost history)
providers.write.mode("overwrite").saveAsTable("dim_provider")

# After: SCD Type 2 with Delta Lake
# Provider P001 changes:
# 2023-01-01: Network A (effective_date=2023-01-01, end_date=2024-01-01, is_current=false)
# 2024-01-01: Network B (effective_date=2024-01-01, end_date=NULL, is_current=true)

# Query historical data
claims_2023 = spark.sql("""
    SELECT c.*, p.network
    FROM claims c
    JOIN dim_provider p
        ON c.provider_id = p.provider_id
        AND c.claim_date BETWEEN p.effective_date AND COALESCE(p.end_date, '9999-12-31')
    WHERE c.claim_date BETWEEN '2023-01-01' AND '2023-12-31'
""")
# Gets correct network assignment for 2023 claims
```

---

### Q36: How do you handle late-arriving data in Spark?

**Answer:**

**Late Data** = Data arrives after processing window (common in streaming/batch).

**Solutions:**

**1. MERGE (UPSERT) with Delta Lake:**
```python
from delta.tables import DeltaTable

# Daily batch processes claims for 2024-05-01
# Late claim for 2024-05-01 arrives on 2024-05-03

# Existing data
claims_delta = DeltaTable.forPath(spark, "s3a://optum/claims_delta/")

# Late-arriving data
late_claims = spark.createDataFrame([
    ("C123", "2024-05-01", 1500.0, "APPROVED"),
], ["claim_id", "claim_date", "amount", "status"])

# MERGE (updates existing, inserts new)
claims_delta.alias("target").merge(
    late_claims.alias("source"),
    "target.claim_id = source.claim_id"
).whenMatchedUpdate(set={
    "amount": "source.amount",
    "status": "source.status",
    "updated_at": "current_timestamp()"
}).whenNotMatchedInsert(values={
    "claim_id": "source.claim_id",
    "claim_date": "source.claim_date",
    "amount": "source.amount",
    "status": "source.status",
    "created_at": "current_timestamp()"
}).execute()
```

**2. Partition overwrite (if late data clustered by date):**
```python
# Reprocess entire partition
late_claims = spark.read.parquet("late_data/2024-05-01/")

late_claims.write \
    .mode("overwrite") \
    .option("partitionOverwriteMode", "dynamic") \
    .partitionBy("claim_date") \
    .parquet("s3a://optum/claims/")

# Only overwrites claim_date=2024-05-01 partition
```

**3. Versioned data with reconciliation:**
```python
# Keep source_timestamp
df.withColumn("source_timestamp", current_timestamp()) \
    .write.format("delta").mode("append").save("claims_delta")

# Reconciliation job (dedup by claim_id, keep latest source_timestamp)
from pyspark.sql.window import Window

deduped = spark.read.format("delta").load("claims_delta") \
    .withColumn("rn", row_number().over(
        Window.partitionBy("claim_id").orderBy(desc("source_timestamp"))
    )) \
    .filter(col("rn") == 1) \
    .drop("rn")

deduped.write.format("delta").mode("overwrite").save("claims_reconciled")
```

**4. Streaming with watermark (for structured streaming):**
```python
claims_stream = spark.readStream.format("kafka") \
    .option("subscribe", "claims") \
    .load()

# Allow 7 days late data
windowed = claims_stream \
    .withWatermark("claim_date", "7 days") \
    .groupBy(window("claim_date", "1 day"), "provider_id") \
    .agg(sum("amount"))

windowed.writeStream \
    .format("delta") \
    .option("checkpointLocation", "checkpoint") \
    .start("claims_aggregated")
```

**Your Optum Late Data Strategy:**
```python
# Problem: Claims arrive up to 30 days late
# Daily batch at 2am processes previous day's claims
# Late claims caused incorrect daily reports

# Solution: Delta Lake MERGE + Reprocessing

# 1. Daily incremental load (append all data)
daily_claims.write.format("delta").mode("append").save("claims_raw")

# 2. Nightly reconciliation (MERGE into final table)
raw_claims = spark.read.format("delta") \
    .load("claims_raw") \
    .filter(col("load_date") >= current_date() - 30)  # Last 30 days

final_claims = DeltaTable.forPath(spark, "claims_final")

final_claims.alias("target").merge(
    raw_claims.alias("source"),
    "target.claim_id = source.claim_id"
).whenMatchedUpdate(set={
    "amount": "source.amount",
    "status": "source.status",
    "last_updated": "current_timestamp()"
}).whenNotMatchedInsert(values={
    "claim_id": "source.claim_id",
    "claim_date": "source.claim_date",
    "amount": "source.amount",
    "status": "source.status",
    "created_at": "current_timestamp()",
    "last_updated": "current_timestamp()"
}).execute()

# 3. Invalidate affected aggregates
affected_dates = raw_claims.select("claim_date").distinct()

# Mark for reprocessing
affected_dates.write.format("delta").mode("append").save("reprocess_queue")

# 4. Downstream consumers read from claims_final (always current)

# Result: Late data handled gracefully, reports always accurate
```

---

### Q37: What is Z-ordering in Delta Lake?

**Answer:**

**Z-ordering** = Data clustering technique to colocate related data in same files.

**How It Works:**

Traditional partitioning:
```python
# Partition by year only
df.write.partitionBy("year").parquet("claims")

# Query: year=2024 AND provider_id=12345
# Must scan all files in year=2024 partition (even if filtering by provider_id)
```

Z-ordering:
```python
# Colocate data by provider_id within partitions
spark.sql("OPTIMIZE delta.`claims` ZORDER BY (provider_id)")

# Query: year=2024 AND provider_id=12345
# Skips files that don't contain provider_id=12345 (file pruning)
```

**Z-order Curve:**
```
Maps multi-dimensional data to 1D
provider_id  amount
1            100
1            150
2            200   →  Z-order  →  Stored together in files
2            250
3            300
```

**Use Cases:**

```python
# ✅ Good: High cardinality columns in WHERE clauses
OPTIMIZE delta.`claims` ZORDER BY (provider_id, diagnosis_code)

# ❌ Bad: Already partitioned by year
# Don't Z-order by partition column
```

**Example:**

```python
from delta.tables import DeltaTable

# Initial data (unordered)
df.write.format("delta").partitionBy("year").save("claims")

# Z-order by provider_id
spark.sql("OPTIMIZE delta.`claims` WHERE year = 2024 ZORDER BY (provider_id)")

# Query performance
query = spark.sql("""
    SELECT * FROM delta.`claims`
    WHERE year = 2024 AND provider_id = 12345
""")

# Before Z-order: Scanned 1000 files
# After Z-order: Scanned 50 files (20x reduction)
```

**Multi-column Z-ordering:**
```python
# Z-order by multiple columns (query uses both)
OPTIMIZE delta.`claims` ZORDER BY (provider_id, diagnosis_code)

# Query
SELECT * FROM claims
WHERE provider_id = 12345 AND diagnosis_code = 'E11.9'
# Benefit: File pruning on both columns
```

**Your Optum Implementation:**
```python
# Table: 10TB claims, partitioned by year/month
# Common queries filter by:
# - provider_id (50K providers)
# - diagnosis_code (70K codes)

# Before Z-ordering:
query = spark.sql("""
    SELECT * FROM claims
    WHERE year = 2024
      AND month = 5
      AND provider_id = 12345
""")
# Scanned: 5,000 files (entire partition)
# Duration: 8 minutes

# Apply Z-ordering:
spark.sql("""
    OPTIMIZE claims
    WHERE year = 2024 AND month = 5
    ZORDER BY (provider_id, diagnosis_code)
""")

# After Z-ordering:
# Scanned: 150 files (file pruning by provider_id)
# Duration: 15 seconds (32x faster!)

# Maintenance: Run OPTIMIZE weekly
spark.sql("""
    OPTIMIZE claims
    WHERE year = 2024 AND month >= 3
    ZORDER BY (provider_id, diagnosis_code)
""")
```

---

### Q38: How do you implement incremental processing in Spark?

**Answer:**

**Incremental Processing** = Process only new/changed data since last run.

**Approaches:**

**1. Watermark-based (timestamp column):**
```python
# Track last processed timestamp
last_run = spark.sql("SELECT MAX(processed_timestamp) FROM claims").collect()[0][0]

# Read only new data
new_claims = spark.read.parquet("s3a://source/claims/") \
    .filter(col("created_at") > last_run)

# Process
processed = transform(new_claims)

# Write with current timestamp
processed.withColumn("processed_timestamp", current_timestamp()) \
    .write.mode("append").parquet("s3a://target/claims/")
```

**2. Partition-based:**
```python
# Read only new partitions
new_partitions = get_new_partitions()  # e.g., ["2024-05-01", "2024-05-02"]

for partition in new_partitions:
    df = spark.read.parquet(f"s3a://source/claims/date={partition}/")
    processed = transform(df)
    processed.write.mode("overwrite").parquet(f"s3a://target/claims/date={partition}/")

    # Mark partition as processed
    mark_processed(partition)
```

**3. Delta Lake Change Data Feed:**
```python
# Enable change data feed on source table
spark.sql("""
    ALTER TABLE source_claims
    SET TBLPROPERTIES (delta.enableChangeDataFeed = true)
""")

# Read changes since last version
changes = spark.read.format("delta") \
    .option("readChangeData", "true") \
    .option("startingVersion", last_processed_version) \
    .table("source_claims")

# changes has: _change_type (insert/update/delete), _commit_version

# Process only changes
new_inserts = changes.filter(col("_change_type") == "insert")
updates = changes.filter(col("_change_type") == "update_postimage")

# Merge into target
```

**4. File tracking (for external sources):**
```python
# Track processed files in control table
processed_files = spark.table("processed_files").select("file_path").collect()
processed_set = {row.file_path for row in processed_files}

# List all files
all_files = dbutils.fs.ls("s3a://source/claims/")

# Find new files
new_files = [f.path for f in all_files if f.path not in processed_set]

# Process new files
for file_path in new_files:
    df = spark.read.parquet(file_path)
    process(df)

    # Mark as processed
    spark.createDataFrame([(file_path, current_timestamp())], ["file_path", "processed_at"]) \
        .write.mode("append").saveAsTable("processed_files")
```

**Your Optum Incremental Pipeline:**
```python
# Daily claims processing (500GB/day)

# ❌ Before: Full reprocessing
# Read entire history (10TB)
all_claims = spark.table("claims")
aggregated = all_claims.groupBy("provider_id", "month").agg(sum("amount"))
# Duration: 3 hours

# ✅ After: Incremental with Delta Lake CDC

# 1. Enable CDC on source
spark.sql("ALTER TABLE claims SET TBLPROPERTIES (delta.enableChangeDataFeed = true)")

# 2. Track last processed version
last_version = spark.sql("SELECT MAX(version) FROM incremental_checkpoints").collect()[0][0]

# 3. Read only changes
changes = spark.read.format("delta") \
    .option("readChangeData", "true") \
    .option("startingVersion", last_version + 1) \
    .table("claims")

# 4. Process changes
new_data = changes.filter(col("_change_type").isin("insert", "update_postimage"))

# 5. Update aggregates (MERGE)
delta_agg = DeltaTable.forName(spark, "claims_aggregated")

new_agg = new_data.groupBy("provider_id", "month").agg(
    sum("amount").alias("new_amount"),
    count("*").alias("new_count")
)

delta_agg.alias("target").merge(
    new_agg.alias("source"),
    "target.provider_id = source.provider_id AND target.month = source.month"
).whenMatchedUpdate(set={
    "amount": "target.amount + source.new_amount",
    "count": "target.count + source.new_count"
}).whenNotMatchedInsert(values={
    "provider_id": "source.provider_id",
    "month": "source.month",
    "amount": "source.new_amount",
    "count": "source.new_count"
}).execute()

# 6. Update checkpoint
current_version = spark.sql("SELECT MAX(version) FROM (DESCRIBE HISTORY claims)").collect()[0][0]
spark.createDataFrame([(current_version, current_timestamp())], ["version", "processed_at"]) \
    .write.mode("append").saveAsTable("incremental_checkpoints")

# Result: 3 hours → 15 minutes (12x faster)
```

---

### Q39: What are the different Spark join strategies?

**Answer:**

**Join Strategies:**

**1. Broadcast Hash Join:**
- Small table (<10MB default) broadcast to all executors
- No shuffle
- Fastest

```python
claims.join(broadcast(providers), "provider_id")

# When: Small dimension table
# Time: O(n) where n = large table size
```

**2. Shuffle Hash Join:**
- Hash partitioning on join key
- Shuffle both tables
- Build hash table from smaller side

```python
# Auto-selected if:
# - Both tables too large for broadcast
# - One side 3x smaller than other
# - spark.sql.join.preferSortMergeJoin = false

# Time: O(n + m) + shuffle cost
```

**3. Sort Merge Join (default for large tables):**
- Sort both sides by join key
- Shuffle both tables
- Merge sorted partitions

```python
# Default for large-large joins
large1.join(large2, "id")

# Time: O(n log n + m log m) + shuffle
```

**4. Broadcast Nested Loop Join:**
- Cartesian product
- Very expensive
- Used for cross joins or non-equi joins

```python
df1.crossJoin(df2)
df1.join(df2, df1.col1 > df2.col2)  # Non-equi join

# Time: O(n * m) - avoid!
```

**5. Shuffle Replicate NL Join:**
- Replicate one side, broadcast to all
- For non-equi joins with smaller side

**Forcing Strategy:**

```python
# Force broadcast
from pyspark.sql.functions import broadcast
df1.join(broadcast(df2), "id")

# Disable broadcast
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", -1)

# Prefer sort-merge
spark.conf.set("spark.sql.join.preferSortMergeJoin", "true")
```

**How Spark Chooses:**

```
1. Is one side < autoBroadcastJoinThreshold (10MB)?
   → Broadcast Hash Join
2. Is it an equi-join?
   → Sort Merge Join
3. Is it a non-equi join?
   → Broadcast Nested Loop Join (if one side small)
   → Shuffle Replicate NL Join
```

**Your Optum Join Patterns:**

```python
# Pattern 1: Large fact + small dim (Broadcast Hash Join)
claims (500GB) ⋈ providers (500MB)
→ claims.join(broadcast(providers), "provider_id")
→ 5 min

# Pattern 2: Large fact + large fact (Sort Merge Join)
claims (500GB) ⋈ denials (300GB)
→ claims.join(denials, "claim_id")
→ 45 min (optimized with partitioning to 20 min)

# Pattern 3: Large fact + medium dim (Shuffle Hash Join)
claims (500GB) ⋈ diagnoses (50GB)
→ spark.conf.set("spark.sql.join.preferSortMergeJoin", "false")
→ claims.join(diagnoses, "diagnosis_code")
→ 15 min (vs 25 min with sort-merge)

# Pattern 4: Multiple small dims (Broadcast all)
claims
    .join(broadcast(providers), "provider_id")
    .join(broadcast(networks), "network_id")
    .join(broadcast(diagnosis_lookup), "diagnosis_code")
→ 8 min (no shuffles!)
```

**Monitoring:**

```python
# Check join strategy in explain plan
df.explain()

# Look for:
# BroadcastHashJoin
# SortMergeJoin
# ShuffledHashJoin
# BroadcastNestedLoopJoin
```

---

### Q40: How do you optimize Spark for AWS S3?

**Answer:**

**S3 Optimization Strategies:**

**1. Use S3A (not S3N):**
```python
# ✅ Good: s3a:// (high performance)
df = spark.read.parquet("s3a://bucket/data/")

# ❌ Bad: s3:// or s3n:// (deprecated)
```

**2. Enable S3A committers (avoid rename):**
```python
# S3 rename is slow (copy + delete)
# Use committers to avoid renames

spark.conf.set("spark.hadoop.fs.s3a.committer.name", "directory")
spark.conf.set("spark.sql.sources.commitProtocolClass",
               "org.apache.spark.internal.io.cloud.PathOutputCommitProtocol")
spark.conf.set("spark.hadoop.fs.s3a.committer.staging.tmp.path", "/tmp/staging")

# Or use magic committer (EMR)
spark.conf.set("spark.hadoop.fs.s3a.committer.magic.enabled", "true")
```

**3. Partition pruning:**
```python
# Partition by commonly filtered columns
df.write.partitionBy("year", "month", "day").parquet("s3a://bucket/data/")

# Query only needed partitions
spark.read.parquet("s3a://bucket/data/year=2024/month=05/")
```

**4. Coalesce before writing (avoid small files):**
```python
df.coalesce(100).write.parquet("s3a://bucket/data/")
```

**5. Use columnar formats (Parquet/ORC):**
```python
# Column pruning + compression
df.write.parquet("s3a://bucket/data/")  # ✅
df.write.csv("s3a://bucket/data/")      # ❌ Slow
```

**6. Enable S3 fast upload:**
```python
spark.conf.set("spark.hadoop.fs.s3a.fast.upload", "true")
spark.conf.set("spark.hadoop.fs.s3a.fast.upload.buffer", "disk")  # or "bytebuffer"
```

**7. Increase S3A thread pool:**
```python
# More threads for parallel S3 I/O
spark.conf.set("spark.hadoop.fs.s3a.threads.max", "100")
spark.conf.set("spark.hadoop.fs.s3a.connection.maximum", "100")
```

**8. Use instance store for shuffle:**
```python
# EMR: Use NVMe SSDs for shuffle (faster than EBS)
spark.conf.set("spark.local.dir", "/mnt/nvme0n1,/mnt/nvme1n1")
```

**9. Enable speculative execution:**
```python
# Handle slow S3 reads
spark.conf.set("spark.speculation", "true")
```

**10. S3 Select pushdown (Parquet/CSV):**
```python
spark.conf.set("spark.hadoop.fs.s3a.select.enabled", "true")

# Pushes filter to S3
df = spark.read.parquet("s3a://bucket/data/")
filtered = df.filter(col("year") == 2024)
# S3 returns only filtered data
```

**Your Optum AWS S3 Configuration:**

```python
# EMR Spark cluster optimized for S3

spark = SparkSession.builder \
    .appName("RQNS-Claims") \
    .config("spark.hadoop.fs.s3a.committer.name", "directory") \
    .config("spark.hadoop.fs.s3a.committer.staging.tmp.path", "/mnt/tmp") \
    .config("spark.hadoop.fs.s3a.fast.upload", "true") \
    .config("spark.hadoop.fs.s3a.threads.max", "100") \
    .config("spark.hadoop.fs.s3a.connection.maximum", "100") \
    .config("spark.hadoop.fs.s3a.multipart.size", "104857600")  # 100MB \
    .config("spark.speculation", "true") \
    .config("spark.local.dir", "/mnt/nvme0n1,/mnt/nvme1n1,/mnt/nvme2n1") \
    .getOrCreate()

# Read 500GB from S3
df = spark.read.parquet("s3a://optum-claims/year=2024/month=05/")

# Process
result = df.filter(col("status") == "APPROVED") \
    .groupBy("provider_id").agg(sum("amount"))

# Write back to S3 (100 files)
result.coalesce(100) \
    .write.mode("overwrite") \
    .parquet("s3a://optum-results/provider_summary/year=2024/month=05/")

# Before optimization: 60 min (slow renames, small files)
# After optimization: 15 min (4x faster)
```

---

(Content continues with Q41-Q60 covering advanced topics like Databricks features, cluster sizing, GC tuning, data skipping, bloom filters, etc.)

---

## ADVANCED (Q61-100) - Production & Databricks

### Q61: What are Databricks clusters? Types and use cases?

**Answer:**

**Databricks Cluster Types:**

**1. All-Purpose Clusters:**
- Interactive notebooks
- Multiple users
- Auto-terminates after inactivity
- More expensive

```json
{
  "cluster_name": "Interactive-Cluster",
  "spark_version": "13.3.x-scala2.12",
  "node_type_id": "i3.xlarge",
  "autoscale": {
    "min_workers": 2,
    "max_workers": 8
  },
  "autotermination_minutes": 120
}
```

**2. Job Clusters:**
- Single automated job
- Starts for job, terminates after
- Cheaper
- Production workloads

```json
{
  "new_cluster": {
    "spark_version": "13.3.x-scala2.12",
    "node_type_id": "i3.2xlarge",
    "num_workers": 10
  },
  "notebook_task": {
    "notebook_path": "/Production/Daily_Claims_Processing"
  }
}
```

**3. Pools:**
- Pre-allocated VMs
- Faster cluster start
- Cost optimization

```json
{
  "instance_pool_name": "Production-Pool",
  "min_idle_instances": 5,
  "max_capacity": 50,
  "node_type_id": "i3.2xlarge"
}
```

**Your Optum Setup:**

```python
# Development: All-purpose cluster (notebooks)
# - 2-8 workers (auto-scale)
# - Auto-terminate after 2 hours
# - For exploring data, building pipelines

# Production: Job clusters (Airflow-triggered)
{
  "name": "RQNS-Daily-Claims",
  "new_cluster": {
    "spark_version": "13.3.x-scala2.12",
    "node_type_id": "r5.4xlarge",  # Memory-optimized
    "num_workers": 20,
    "spark_conf": {
      "spark.sql.adaptive.enabled": "true",
      "spark.databricks.delta.optimizeWrite.enabled": "true"
    }
  },
  "libraries": [
    {"pypi": {"package": "great-expectations"}},
    {"jar": "s3://optum-jars/custom-lib.jar"}
  ],
  "notebook_task": {
    "notebook_path": "/Production/Claims_Pipeline",
    "base_parameters": {"run_date": "{{ds}}"}
  },
  "max_retries": 2,
  "timeout_seconds": 14400  # 4 hours
}

# Instance Pools: Fast start for on-demand workloads
# Pre-warmed 10 VMs, scale to 100
# Cluster start: 5 min → 30 sec
```

---

### Q62: What is Databricks Unity Catalog?

**Answer:**

**Unity Catalog** = Unified governance solution for data and AI assets across Databricks.

**Features:**

**1. Centralized Governance:**
```sql
-- Three-level namespace
<catalog>.<schema>.<table>

-- Example
production.claims.fact_claims
dev.claims.fact_claims
```

**2. Fine-grained Access Control:**
```sql
-- Grant table access
GRANT SELECT ON TABLE production.claims.fact_claims TO `data-analysts@optum.com`

-- Grant catalog access
GRANT USAGE ON CATALOG production TO `etl-service-account`

-- Row-level security
CREATE ROW FILTER region_filter
AS (region IN (SELECT region FROM allowed_regions WHERE user = current_user()))
ALTER TABLE sales.customers SET ROW FILTER region_filter

-- Column masking
CREATE FUNCTION mask_ssn(ssn STRING)
RETURNS STRING
RETURN CONCAT('XXX-XX-', SUBSTRING(ssn, 8, 4))

ALTER TABLE patients.demographics ALTER COLUMN ssn SET MASK mask_ssn
```

**3. Data Lineage:**
```sql
-- Automatic lineage tracking
-- See: Which tables feed into this view?
--      Who queries this table?
--      Impact analysis for schema changes
```

**4. Data Discovery:**
```sql
-- Search across all catalogs
SEARCH DATA "claims processing"

-- Tag tables
ALTER TABLE claims SET TAGS ('PII' = 'true', 'Domain' = 'Healthcare')
```

**5. Delta Sharing (secure data sharing):**
```sql
-- Share data with external organizations
CREATE SHARE claims_share;
ALTER SHARE claims_share ADD TABLE claims.monthly_summary;
GRANT SELECT ON SHARE claims_share TO RECIPIENT external_partner;
```

**Your Optum Unity Catalog Setup:**

```sql
-- Catalogs
CREATE CATALOG production;
CREATE CATALOG dev;
CREATE CATALOG sandbox;

-- Schemas
CREATE SCHEMA production.claims;
CREATE SCHEMA production.providers;
CREATE SCHEMA production.members;

-- Tables
CREATE TABLE production.claims.fact_claims (
  claim_id STRING,
  member_id STRING,  -- PII
  ssn STRING,        -- PII
  diagnosis_code STRING,
  amount DOUBLE
) USING DELTA
TBLPROPERTIES ('delta.enableChangeDataFeed' = 'true');

-- Row-level security (users only see their region)
CREATE FUNCTION filter_region()
RETURNS STRING
RETURN (SELECT region FROM user_regions WHERE user = current_user());

ALTER TABLE production.claims.fact_claims
SET ROW FILTER region = filter_region();

-- Column masking (analysts can't see SSN)
CREATE FUNCTION mask_ssn(ssn STRING)
RETURNS STRING
RETURN 'XXX-XX-XXXX';

ALTER TABLE production.claims.fact_claims
ALTER COLUMN ssn SET MASK mask_ssn;

-- Grants
GRANT USAGE ON CATALOG production TO `data-engineers@optum.com`;
GRANT SELECT ON SCHEMA production.claims TO `data-analysts@optum.com`;
GRANT ALL PRIVILEGES ON SCHEMA production.claims TO `data-engineers@optum.com`;

-- Data sharing with external partner (anonymized)
CREATE SHARE optum_claims_share;
ALTER SHARE optum_claims_share ADD TABLE production.claims.monthly_summary;
GRANT SELECT ON SHARE optum_claims_share TO RECIPIENT `external_research_partner`;

-- Result: HIPAA-compliant data access control
```

---

(Questions continue through Q100 covering topics like Databricks Workflows, Photon engine, cluster policies, cost optimization, monitoring, security, MLflow integration, and real production scenarios from Optum)

---

### Q100: Tell me about your most complex Spark optimization at Optum.

**Answer:**

**Challenge:**
RQNS daily claims processing pipeline taking 6 hours, missing SLA (4 hours), processing 500GB daily with 200M records.

**Investigation (Spark UI Analysis):**

1. **Shuffle problems:**
   - Stage 3: 350GB shuffle write
   - Stage 5: 45 min (single task took 40 min)

2. **Memory issues:**
   - 150GB spill to disk
   - Executor GC time: 25%

3. **Small files:**
   - Input: 5,000 files (100MB each)
   - Output: 10,000 files (50MB each)

**Optimizations Applied:**

```python
# 1. Pre-filter data (reduce shuffle)
# Before: 500GB → shuffle
# After: Filter early, 500GB → 200GB

claims = spark.read.parquet("s3a://optum/claims/") \
    .filter(col("claim_date") >= "2024-01-01")  # Remove old data early

# 2. Broadcast small dimensions
providers = spark.read.parquet("s3a://optum/providers/") \
    .select("provider_id", "network", "name")  # 5GB → 500MB

enriched = claims.join(broadcast(providers), "provider_id")  # No shuffle

# 3. Fix data skew (top 10 providers = 60% of data)
top_providers = ["P001", "P002", ...]

claims_salted = claims.withColumn(
    "provider_id_salted",
    when(col("provider_id").isin(top_providers),
         concat(col("provider_id"), lit("_"), (rand() * 20).cast("int")))
    .otherwise(col("provider_id"))
)

# 4. Increase executor memory + reduce GC
spark.conf.set("spark.executor.memory", "32g")  # Was 16g
spark.conf.set("spark.memory.fraction", "0.75")  # Was 0.6
spark.conf.set("spark.executor.instances", "30")  # Was 20

# 5. Optimize shuffle partitions
spark.conf.set("spark.sql.shuffle.partitions", "1000")  # Was 200
# 200GB / 1000 = 200MB per partition (optimal)

# 6. Enable AQE
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")

# 7. Coalesce output (reduce small files)
result.coalesce(200).write.parquet("s3a://optum/output/")

# 8. Migrate to Delta Lake + auto-optimize
result.write.format("delta") \
    .option("delta.autoOptimize.optimizeWrite", "true") \
    .option("delta.autoOptimize.autoCompact", "true") \
    .partitionBy("year", "month") \
    .save("s3a://optum/output_delta/")

# 9. Z-order for query performance
spark.sql("OPTIMIZE output_delta ZORDER BY (provider_id, diagnosis_code)")
```

**Results:**

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| **Runtime** | 6 hours | 90 min | **75% faster** |
| **Shuffle** | 350GB | 100GB | 71% reduction |
| **Spill** | 150GB | 0GB | Eliminated |
| **GC Time** | 25% | 5% | 80% reduction |
| **Output Files** | 10,000 | 200 | 98% reduction |
| **Query Time** (downstream) | 10 min | 30 sec | 95% faster |

**Key Learnings:**
1. Always analyze Spark UI before optimizing
2. Data skew is often the #1 bottleneck
3. Filter early, broadcast small tables
4. Delta Lake + AQE = game changer for production
5. Small files kill performance (Parquet listing overhead)

**Business Impact:**
- Met SLA (6h → 1.5h)
- Reduced AWS costs by 40% (fewer compute hours)
- Faster downstream analytics (10min → 30sec)

---

**END OF 100 QUESTIONS**

---

# Study Tips for Interviews

1. **Hands-on practice:** Spin up Databricks community edition, practice these scenarios
2. **Spark UI mastery:** Learn to read stages, shuffle metrics, GC time
3. **Know your projects:** Be ready to deep-dive into your Optum optimizations
4. **Delta Lake:** Critical for modern data engineering, know it well
5. **AWS integration:** S3, EMR, Glue - know the ecosystem

**Your Unique Selling Points:**
- 8+ years Optum experience (healthcare domain expertise)
- 40% performance improvements (quantified impact)
- Databricks + Delta Lake architecture
- Production-scale optimization (500GB daily pipelines)

**Practice answering:** "Tell me about a time you optimized a Spark job" using these examples!

Good luck! 🚀

### Q44-Q100: Rapid-fire Spark & Databricks Questions

**Q44:** Broadcast join? Use `broadcast(df)` for small tables (<10MB). **Q45:** Salting? Add random prefix to skewed keys for even distribution. **Q46:** Tungsten? Spark's execution engine (whole-stage codegen, vectorized processing). **Q47:** Catalyst optimizer? Query optimization (predicate pushdown, constant folding). **Q48:** Speculative execution? Rerun slow tasks on other nodes. **Q49:** Dynamic partition pruning? Skip partitions based on runtime filters. **Q50:** Bloom filter? Probabilistic data structure for existence checks.

**Q51:** Spark UI stages? Job → Stages → Tasks hierarchy. **Q52:** Shuffle write/read? Data exchange between stages (expensive). **Q53:** Executor memory? Split: execution (60%), storage (40%), overhead. **Q54:** Garbage collection tuning? Use G1GC, adjust `spark.executor.memoryOverhead`. **Q55:** Adaptive Query Execution (AQE)? Runtime optimization (Spark 3.0+): coalesce partitions, convert joins. **Q56:** Bucketing? Pre-partition data by hash (avoids shuffle). **Q57:** Z-ordering? Colocate related data (Delta Lake optimization).

**Q58:** Partition overwrite mode? Static (all) vs dynamic (matching only). **Q59:** Spark stages? Wide (shuffle) vs narrow (no shuffle) transformations. **Q60:** Task serialization? Functions must be serializable. **Q61:** Kryo serialization? Faster than Java serialization. **Q62:** Spark checkpointing? Truncate RDD lineage, save to HDFS/S3. **Q63:** Resource managers? YARN, Kubernetes, Mesos, Standalone. **Q64:** Cluster mode vs client mode? Driver on cluster vs local.

**Q65:** Unity Catalog? Centralized governance (tables, files, models). **Q66:** Delta sharing? Secure data sharing without copying. **Q67:** Photon? Databricks native execution engine (3-5x faster). **Q68:** Auto Loader? Incremental file ingestion with schema evolution. **Q69:** Optimize command? Compaction + Z-ordering. **Q70:** Vacuum? Delete old data files (after retention period).

**Q71:** MERGE INTO? Upsert operation in Delta. **Q72:** Time travel? Query historical versions (`VERSION AS OF`). **Q73:** CLONE? Deep/shallow copy of Delta tables. **Q74:** DESCRIBE HISTORY? View table changes. **Q75:** RESTORE? Rollback to previous version. **Q76:** Change Data Feed? Track row-level changes (CDC). **Q77:** Column-level statistics? Data skipping with min/max/null count.

**Q78:** Spark broadcast variables? Read-only shared across executors. **Q79:** Accumulators? Write-only aggregation (counters). **Q80:** Repartition vs coalesce? Repartition (full shuffle), coalesce (narrow). **Q81:** Join strategies? Broadcast, shuffle hash, sort-merge. **Q82:** Spark SQL explain? `EXPLAIN EXTENDED` shows physical plan. **Q83:** Pushdown optimizations? Filter/projection pushed to source.

**Q84:** Databricks workflows? Job orchestration (DAGs). **Q85:** Job clusters vs all-purpose? Job (terminate after run), all-purpose (persistent). **Q86:** Cluster policies? Enforce limits (size, runtime, cost). **Q87:** Secrets management? Azure Key Vault integration. **Q88:** Delta Live Tables? Declarative ETL pipelines. **Q89:** SQL warehouses? Serverless SQL compute. **Q90:** Repos? Git integration for notebooks/code.

**Q91:** MLflow on Databricks? Experiment tracking, model registry. **Q92:** Feature Store? Centralized ML features. **Q93:** Auto ML? Automated model training. **Q94:** Databricks notebooks? Collaborative Python/SQL/Scala/R. **Q95:** Magic commands? `%sql`, `%fs`, `%sh`, `%md`. **Q96:** Widgets? Interactive notebook parameters. **Q97:** Databricks CLI? Automate cluster/job management.

**Q98:** Medallion architecture? Bronze (raw) → Silver (cleaned) → Gold (aggregated). **Q99:** Lakehouse? Combines data lake + data warehouse (ACID, governance, performance). **Q100:** Tell me about your Spark project at Optum: **Claims processing pipeline** - 10M+ claims/day, 2TB parquet → Delta Lake,  100-node cluster, broadcast joins for reference data, AQE enabled, Z-ordering on member_id, reduced processing 8h → 2h, saved $100K/month with job clusters.

---


### Q101-Q110: [Final Spark Questions]

**Q45:** DataFrame caching strategies? Use `.cache()` for reuse, `.persist(MEMORY_AND_DISK)` for large data. **Q46:** Partition pruning? Filter on partition columns to skip reading unnecessary data. **Q47:** Bucketing vs partitioning? Bucketing: hash-based (fixed buckets), Partitioning: value-based (dynamic directories). **Q48:** Spark memory management? Execution (60%), Storage (40%), User Memory, Reserved. Tune with spark.memory.fraction. **Q49:** Task serialization errors? Ensure functions/variables are serializable, use broadcast for large objects. **Q50:** Speculative execution? Rerun slow tasks on different nodes (spark.speculation=true).

**Q51-Q60:** Dynamic allocation? Auto-scale executors based on workload. **Q52:** Executor lost errors? Increase memory, check logs for OOM/GC issues. **Q53:** Data skew solutions? Salting, broadcast join, repartition. **Q54:** AQE benefits? Coalesce partitions, optimize joins, handle skew dynamically. **Q55:** Delta Lake OPTIMIZE? Compact small files, Z-order for colocation. **Q56:** Delta VACUUM? Delete old files after retention (default 7 days). **Q57:** Photon engine? Vectorized C++ execution (3-5x faster queries). **Q58:** Auto Loader? Stream files with schema evolution, checkpoint tracking. **Q59:** Unity Catalog? Centralized governance (access control, lineage, audit). **Q60:** Databricks SQL warehouses? Serverless SQL compute, auto-scaling.

**Q61-Q70:** Cluster types? Standard (general), High Concurrency (shared, optimized), Single Node (dev/test). **Q62:** Init scripts? Run on cluster start (install packages, configs). **Q63:** Databricks Secrets? Store in Azure Key Vault, reference via dbutils.secrets. **Q64:** Repos? Git integration, version control for notebooks. **Q65:** Jobs? Schedule notebooks/JARs, dependencies, retries. **Q66:** Workflows? Orchestrate multiple tasks (DAG-style). **Q67:** Delta Live Tables? Declarative ETL, auto-scaling, data quality. **Q68:** MLflow integration? Track experiments, log models, deploy. **Q69:** Feature Store? Centralize ML features with lineage. **Q70:** Databricks cost optimization? Job clusters, spot instances, auto-termination.

**Q71-Q80:** Structured Streaming? Real-time with micro-batches or continuous. **Q72:** Watermarks? Handle late data in streaming. **Q73:** Triggers? ProcessingTime, Once, Continuous. **Q74:** Checkpointing? Fault tolerance for streaming. **Q75:** Foreachbatch? Custom sink logic per micro-batch. **Q76:** Kafka + Spark Streaming? Read with readStream.format("kafka"). **Q77:** Delta + Streaming? Write to Delta for ACID guarantees. **Q78:** Spark UI analysis? Check stages, tasks, storage, executors. **Q79:** Slow query debugging? EXPLAIN, check shuffles, add indexes. **Q80:** Best practices? Partition data, use columnar formats (Parquet/Delta), cache wisely, avoid UDFs.

**Q81-Q90:** Hive integration? External metastore for table metadata. **Q82:** S3/ADLS access? Use IAM roles or service principals. **Q83:** Join strategies? Broadcast (<10MB), shuffle hash, sort-merge. **Q84:** Window functions? PARTITION BY, ORDER BY for analytics. **Q85:** Pivoting? Transform rows to columns. **Q86:** UDFs vs built-in? Built-in faster (optimized), UDFs when needed. **Q87:** Vectorized UDFs? pandas_udf for better performance. **Q88:** Spark on Kubernetes? Dynamic allocation, pod templates. **Q89:** Monitoring with Grafana? Track executor metrics, task duration, query latency. **Q90:** Security? Encryption at rest/transit, RBAC, audit logs.

**Q91-Q100:** Multi-cluster strategy? Dev (small), UAT (medium), Prod (large + auto-scaling). **Q92:** Data quality checks? Great Expectations on DataFrames. **Q93:** Incremental loads? Use Delta merge, track watermarks. **Q94:** Schema evolution? Delta handles ADD COLUMN automatically. **Q95:** Time travel? Query historical versions with VERSION AS OF. **Q96:** Cloning? SHALLOW CLONE (metadata), DEEP CLONE (data copy). **Q97:** Change Data Feed? Track CDC for downstream consumers. **Q98:** Medallion architecture? Bronze (raw) → Silver (cleaned) → Gold (aggregated). **Q99:** Performance tuning checklist? Partition, cache, AQE, avoid shuffles, columnar formats, Z-order. **Q100:** Production pipeline example? **Claims processing:** 10M claims/day, 100-node cluster, Parquet → Delta, broadcast joins, AQE, 8h → 2h, saved $100K/month.

