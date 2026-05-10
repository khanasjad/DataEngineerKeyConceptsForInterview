# Apache Spark & Databricks - Distributed Data Processing

**Master big data processing with Spark and Databricks for data engineering interviews**

---

## 📚 What's in This Folder

### **1. 100-QUESTIONS.md**
Comprehensive Q&A covering:
- Spark architecture and execution model
- RDDs, DataFrames, and Datasets
- Transformations vs actions
- Spark SQL and optimizations
- Performance tuning (partitioning, caching, broadcasting)
- Delta Lake and lakehouse architecture
- Databricks platform features
- PySpark best practices
- Real-world optimization scenarios

### **2. CHEATSHEET.md**
Quick reference with:
- Core concepts definitions
- Common transformations and actions
- Performance tuning checklist
- Spark configurations
- Delta Lake operations
- Interview talking points

---

## 🎯 What You'll Learn

### **Spark Fundamentals**
- **RDD (Resilient Distributed Dataset):** Low-level distributed data structure
- **DataFrame:** Distributed table with named columns
- **Dataset:** Type-safe DataFrame (Scala/Java)
- **Transformations:** Lazy operations (map, filter, join)
- **Actions:** Trigger computation (count, collect, save)
- **DAG:** Directed Acyclic Graph of operations
- **Partitions:** Units of parallelism
- **Executors:** Workers that run tasks

### **Advanced Concepts**
- **Catalyst Optimizer:** Query optimization engine
- **Tungsten Engine:** Memory and CPU optimization
- **Shuffle:** Data redistribution across partitions
- **Broadcast Joins:** Optimize small table joins
- **Adaptive Query Execution (AQE):** Runtime optimization
- **Partition Pruning:** Skip irrelevant partitions
- **Predicate Pushdown:** Filter at source

### **Delta Lake**
- **ACID Transactions:** Atomicity, consistency, isolation, durability
- **Time Travel:** Query historical versions
- **Schema Evolution:** Add/modify columns safely
- **MERGE (Upsert):** Insert + update in one operation
- **Z-Ordering:** Co-locate related data
- **Optimize:** Compact small files
- **Vacuum:** Delete old files

### **Databricks Features**
- **Unity Catalog:** Unified governance
- **Auto Loader:** Incremental data ingestion
- **Photon Engine:** Native query acceleration
- **Delta Live Tables:** ETL framework
- **MLflow:** ML lifecycle management
- **Collaborative notebooks**

---

## 💼 Why Spark & Databricks Matter

### **Industry Adoption**
- **Spark:** Most popular big data processing engine
- **Databricks:** Fastest-growing data platform (founded by Spark creators)
- Used by: Netflix, Uber, Apple, Comcast, Shell, Adobe

### **Key Use Cases**
- **ETL/ELT:** Transform terabytes of data daily
- **Real-time processing:** Structured Streaming
- **Machine Learning:** MLlib, feature engineering
- **Data Lake/Lakehouse:** Centralized storage and processing
- **SQL Analytics:** Interactive queries on big data
- **Log processing:** Parse and analyze application logs

### **Interview Focus**
1. Understanding Spark execution model
2. Performance optimization techniques
3. Handling data skew
4. Delta Lake operations
5. Production Spark job design

---

## 🚀 Quick Start Guide

### **1. Read Cheatsheet (20 minutes)**
Familiarize yourself with core concepts from `CHEATSHEET.md`.

### **2. Study 100 Questions (5-6 hours)**
- **Q1-Q25:** Fundamentals (RDD, DataFrame, transformations, actions)
- **Q26-Q50:** Spark SQL, optimizations, partitioning
- **Q51-Q75:** Performance tuning, shuffle, caching, broadcasting
- **Q76-Q100:** Delta Lake, Databricks, production patterns

### **3. Hands-On Practice**

**Local Spark Setup:**
```bash
# Install PySpark
pip install pyspark

# Python script
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("Practice") \
    .master("local[*]") \
    .getOrCreate()

# Read data
df = spark.read.csv("data.csv", header=True, inferSchema=True)

# Transform
result = df.filter(df['age'] > 25) \
           .groupBy('city') \
           .count()

# Show
result.show()
```

**Databricks Community Edition:**
- Sign up: https://community.cloud.databricks.com/
- Free tier with limited resources
- Practice notebooks and Delta Lake

---

## 📖 Topic Coverage

### **Spark Architecture**
```
┌─────────────────────────────────────┐
│         Driver Program              │
│  ┌──────────────────────────────┐  │
│  │     SparkContext             │  │
│  └──────────┬───────────────────┘  │
└─────────────┼───────────────────────┘
              │
    ┌─────────┴─────────┐
    │  Cluster Manager  │
    └─────────┬─────────┘
              │
    ┌─────────┴─────────────────┐
    │                           │
┌───▼──────┐           ┌───────▼──┐
│ Executor │           │ Executor │
│  Task    │           │  Task    │
│  Cache   │           │  Cache   │
└──────────┘           └──────────┘
```

**Key Components:**
- **Driver:** Orchestrates execution, creates DAG
- **Executors:** Run tasks, store cached data
- **Cluster Manager:** Allocates resources (YARN, Kubernetes, Standalone)
- **Tasks:** Units of work on partitions

### **DataFrame API**

**Creating DataFrames:**
```python
# From file
df = spark.read.parquet("s3://bucket/data/")

# From SQL
df = spark.sql("SELECT * FROM table WHERE date >= '2024-01-01'")

# From Python data
data = [("Alice", 25), ("Bob", 30)]
df = spark.createDataFrame(data, ["name", "age"])
```

**Common Transformations:**
```python
# Filter
filtered = df.filter(df['age'] > 25)

# Select
selected = df.select('name', 'age')

# Group by
grouped = df.groupBy('department').agg(
    F.count('*').alias('count'),
    F.avg('salary').alias('avg_salary')
)

# Join
joined = df1.join(df2, df1['id'] == df2['customer_id'], 'left')

# Window function
from pyspark.sql import Window
window_spec = Window.partitionBy('department').orderBy('salary')
df.withColumn('rank', F.rank().over(window_spec))
```

**Actions (Trigger Execution):**
```python
df.count()              # Count rows
df.show()               # Display data
df.collect()            # Return all rows to driver (careful!)
df.write.parquet(path)  # Save to disk
```

### **Performance Optimization**

**1. Partitioning:**
```python
# Repartition (full shuffle)
df.repartition(100, 'customer_id')

# Coalesce (reduce partitions, no shuffle)
df.coalesce(10)

# Check partitions
df.rdd.getNumPartitions()
```

**2. Caching:**
```python
# Cache in memory
df.cache()  # Same as persist(MEMORY_AND_DISK)

# Specific storage level
df.persist(pyspark.StorageLevel.MEMORY_ONLY)

# Unpersist when done
df.unpersist()
```

**3. Broadcast Join:**
```python
from pyspark.sql.functions import broadcast

# Small table (< 10 MB) broadcasted to all executors
result = large_df.join(
    broadcast(small_df),
    large_df['id'] == small_df['id']
)
```

**4. Avoid Shuffles:**
```python
# BAD: Multiple shuffles
df.groupBy('col1').count() \
  .join(df.groupBy('col2').count(), ...)

# GOOD: Single transformation
df.groupBy('col1', 'col2').count()
```

### **Delta Lake**

**Create Delta Table:**
```python
# Write as Delta
df.write.format("delta").save("/path/to/delta-table")

# Create managed table
df.write.format("delta").saveAsTable("my_table")
```

**MERGE (Upsert):**
```python
from delta.tables import DeltaTable

delta_table = DeltaTable.forPath(spark, "/path/to/table")

delta_table.alias("target").merge(
    updates_df.alias("source"),
    "target.id = source.id"
).whenMatchedUpdate(set={
    "value": "source.value",
    "updated_at": "source.timestamp"
}).whenNotMatchedInsert(values={
    "id": "source.id",
    "value": "source.value",
    "created_at": "source.timestamp"
}).execute()
```

**Time Travel:**
```python
# Query historical version
df = spark.read.format("delta").option("versionAsOf", 5).load("/path/to/table")

# Query as of timestamp
df = spark.read.format("delta") \
    .option("timestampAsOf", "2024-01-01") \
    .load("/path/to/table")
```

**Optimize:**
```python
# Compact small files
spark.sql("OPTIMIZE my_table")

# Z-order for faster queries
spark.sql("OPTIMIZE my_table ZORDER BY (customer_id, date)")

# Remove old files (default: 7 days retention)
spark.sql("VACUUM my_table RETAIN 168 HOURS")
```

---

## 🎓 Interview Preparation

### **Week 1: Fundamentals**
- Study Q1-Q30
- Understand RDD vs DataFrame
- Practice basic transformations
- Learn Spark architecture

### **Week 2: Optimization**
- Study Q31-Q60
- Deep dive on shuffle, partitioning
- Practice caching strategies
- Learn broadcast joins

### **Week 3: Advanced**
- Study Q61-Q85
- Delta Lake operations
- Adaptive Query Execution
- Handling data skew

### **Week 4: Production**
- Study Q86-Q100
- Real-world scenarios
- Debugging Spark UI
- Best practices

---

## 💡 Common Interview Questions

### **Conceptual**

**Q: "RDD vs DataFrame vs Dataset?"**
- **RDD:** Low-level, no schema, full control
- **DataFrame:** High-level, schema, optimized (use this!)
- **Dataset:** Type-safe DataFrame (Scala/Java only)

**Q: "Transformation vs Action?"**
- **Transformation:** Lazy (map, filter, join) - builds DAG
- **Action:** Eager (count, collect, save) - triggers execution

**Q: "Explain Spark execution model"**
1. User submits code (transformations)
2. Spark builds DAG
3. Action triggers execution
4. DAG converted to stages (based on shuffles)
5. Stages divided into tasks
6. Tasks scheduled on executors

**Q: "What is shuffle?"**
- Data redistribution across partitions
- Expensive (disk I/O, network I/O)
- Triggered by: groupBy, join, repartition
- Minimize: use broadcast, partition wisely

**Q: "How to optimize Spark jobs?"**
1. Cache frequently used DataFrames
2. Use broadcast for small tables
3. Partition data properly
4. Minimize shuffles
5. Avoid UDFs (use built-in functions)
6. Enable AQE (Adaptive Query Execution)
7. Use columnar formats (Parquet, Delta)

### **Scenario Questions**

**Q: "Data skew - some partitions much larger than others?"**
```python
# Solution 1: Salting (add random key)
from pyspark.sql.functions import rand, concat

df_salted = df.withColumn("salt", (rand() * 100).cast("int"))
result = df_salted.groupBy("key", "salt").agg(...)
final = result.groupBy("key").agg(...)  # Remove salt

# Solution 2: Adaptive Query Execution
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
```

**Q: "Join 1TB table with 1MB table?"**
```python
# Use broadcast join
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    large_df['id'] == small_df['id']
)

# Spark auto-broadcasts if < 10 MB (spark.sql.autoBroadcastJoinThreshold)
```

**Q: "Incremental data processing pattern?"**
```python
# Using Delta Lake
from delta.tables import DeltaTable

# Read latest checkpoint
checkpoint = spark.read.format("delta").load("/checkpoint")
last_processed = checkpoint.agg({"timestamp": "max"}).collect()[0][0]

# Read new data
new_data = spark.read.parquet("/source") \
    .filter(f"timestamp > '{last_processed}'")

# Process and write
processed = transform(new_data)
processed.write.format("delta").mode("append").save("/output")

# Update checkpoint
new_checkpoint = spark.createDataFrame([(max_timestamp,)], ["timestamp"])
new_checkpoint.write.format("delta").mode("overwrite").save("/checkpoint")
```

---

## 🔗 Resources

### **Official Docs**
- [Spark Documentation](https://spark.apache.org/docs/latest/)
- [Databricks Documentation](https://docs.databricks.com/)
- [Delta Lake Documentation](https://docs.delta.io/)

### **Learning**
- [Spark: The Definitive Guide](https://www.oreilly.com/library/view/spark-the-definitive/9781491912201/) (book)
- [Databricks Academy](https://academy.databricks.com/) (free courses)
- [Spark by Examples](https://sparkbyexamples.com/)

### **Practice**
- Databricks Community Edition (free)
- AWS EMR or Google Dataproc (paid)
- Local PySpark installation

---

## 🎯 Key Takeaways

### **Must Know**
✅ Transformations vs actions
✅ Partitioning strategies
✅ Caching and persistence
✅ Broadcast joins
✅ Shuffle operations
✅ Delta Lake basics (MERGE, time travel)

### **Should Know**
✅ Spark execution model (DAG, stages, tasks)
✅ Adaptive Query Execution
✅ Handling data skew
✅ Window functions
✅ UDF vs built-in functions

### **Nice to Know**
✅ Catalyst optimizer internals
✅ Tungsten engine
✅ Z-ordering
✅ Delta Live Tables
✅ Structured Streaming

### **Red Flags**
❌ Using `collect()` on large DataFrames
❌ Not caching frequently used DataFrames
❌ Ignoring partition skew
❌ Overusing UDFs instead of built-in functions
❌ Not monitoring Spark UI

---

## 📝 Performance Tuning Checklist

**Before Optimization:**
- [ ] Enable Spark UI and review execution plan
- [ ] Identify bottlenecks (long stages, large shuffles)
- [ ] Check partition sizes (aim for 128 MB - 1 GB)

**Common Optimizations:**
- [ ] Cache DataFrames used multiple times
- [ ] Broadcast small tables (< 100 MB)
- [ ] Repartition before expensive operations
- [ ] Use columnar formats (Parquet, Delta)
- [ ] Enable AQE for runtime optimization
- [ ] Avoid UDFs, use built-in functions
- [ ] Filter early (predicate pushdown)
- [ ] Use partitioned tables for incremental reads

**Spark Configurations:**
```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
spark.conf.set("spark.sql.files.maxPartitionBytes", "134217728")  # 128 MB
spark.conf.set("spark.sql.shuffle.partitions", "200")  # Adjust based on data size
```

---

## 💼 Interview Practice

### **Coding Questions**
1. Read CSV, filter rows, group by column, write Parquet
2. Join two DataFrames on multiple keys
3. Calculate rolling 7-day average using window function
4. Implement deduplication logic
5. Incremental data load with Delta Lake MERGE

### **Scenario Questions**
1. "How would you process 10 TB of logs daily?"
2. "Spark job running for 5 hours. How to optimize?"
3. "Out of memory error. What to check?"
4. "Data skew causing one task to run 10x longer. Solutions?"
5. "Design ETL pipeline for real-time clickstream data"

### **Debugging Questions**
1. "How to read Spark UI?"
2. "What metrics indicate shuffle problems?"
3. "How to identify data skew from Spark UI?"
4. "Task failures - how to debug?"

---

**Master Spark and build scalable big data pipelines! ⚡**

*For detailed answers and examples, see 100-QUESTIONS.md*
*For quick review before interviews, see CHEATSHEET.md*
