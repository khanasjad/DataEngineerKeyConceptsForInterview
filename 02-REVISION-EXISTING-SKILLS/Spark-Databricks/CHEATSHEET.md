# Spark & Databricks Cheatsheet - Quick Reference

## Spark Core Concepts
- **RDD**: Resilient Distributed Dataset (immutable, distributed, lazy evaluation)
- **DataFrame**: Distributed table with named columns (like Pandas but distributed)
- **Dataset**: Type-safe DataFrame (Scala/Java only)
- **Partition**: Unit of parallelism (data split across executors)
- **Transformation**: Lazy operation (map, filter, groupBy)
- **Action**: Trigger computation (collect, count, show, write)

## Spark Architecture
- **Driver**: Orchestrates execution, holds SparkContext
- **Executor**: Worker process running on nodes, executes tasks
- **Task**: Unit of work on one partition
- **Stage**: Set of tasks (separated by shuffle operations)
- **Job**: Triggered by action, consists of stages
- **Cluster Manager**: YARN, Kubernetes, Mesos, Standalone

## DataFrame Operations
- **select()**: Choose columns
- **filter() / where()**: Filter rows
- **groupBy()**: Aggregate by groups
- **join()**: Combine DataFrames
- **withColumn()**: Add/modify column
- **agg()**: Aggregate functions
- **orderBy()**: Sort

## Transformations
- **Narrow**: No shuffle (map, filter, union) - fast
- **Wide**: Requires shuffle (groupBy, join, repartition) - expensive

## Performance Optimization
- **Cache / Persist**: Store DataFrame in memory for reuse
- **Broadcast join**: Send small table to all executors (avoid shuffle)
- **Partition pruning**: Skip reading unnecessary partitions
- **Predicate pushdown**: Push filters to data source
- **Columnar format**: Use Parquet or Delta (compression, column skipping)
- **Z-ordering**: Colocate related data in Delta Lake
- **AQE (Adaptive Query Execution)**: Runtime optimization (Spark 3.0+)

## Spark SQL
- **Catalyst optimizer**: Query optimization engine
- **Tungsten**: Execution engine (whole-stage codegen, vectorization)
- **Dynamic partition pruning**: Skip partitions based on runtime info
- **Cost-based optimization**: Use statistics for query planning

## Memory Management
- **Execution memory**: For shuffles, joins, sorts (60% default)
- **Storage memory**: For caching (40% default)
- **User memory**: User data structures
- **Reserved memory**: Internal Spark operations
- **Tuning**: `spark.memory.fraction`, `spark.memory.storageFraction`

## Delta Lake
- **ACID transactions**: Atomicity, Consistency, Isolation, Durability
- **Time travel**: Query historical versions (`VERSION AS OF`, `TIMESTAMP AS OF`)
- **Schema evolution**: ADD COLUMN, change types with merge schema
- **MERGE INTO**: Upsert (insert + update)
- **OPTIMIZE**: Compact small files, Z-order
- **VACUUM**: Delete old data files (default 7-day retention)
- **Change Data Feed**: Track row-level changes (CDC)

## Databricks
- **Workspace**: Collaborative environment for notebooks, jobs, data
- **Cluster**: Compute resource (Standard, High Concurrency, Single Node)
- **Job cluster**: Starts for job, terminates after (cheaper)
- **All-purpose cluster**: Persistent for interactive work
- **Notebook**: Collaborative Python/SQL/Scala/R environment
- **Repos**: Git integration for version control
- **Workflows**: Job orchestration (tasks, dependencies, schedules)

## Databricks Features
- **Unity Catalog**: Centralized governance (access control, lineage, audit)
- **Photon**: Native vectorized engine (3-5x faster)
- **Auto Loader**: Incremental file ingestion with schema inference
- **Delta Live Tables**: Declarative ETL pipelines
- **SQL Warehouses**: Serverless SQL compute
- **MLflow**: ML lifecycle management
- **Feature Store**: Centralized ML features

## Common Errors & Solutions
- **OutOfMemoryError**: Increase executor memory, reduce partition size, cache less
- **Task serialization error**: Ensure UDFs and variables are serializable
- **Shuffle errors**: Increase shuffle partitions, tune memory
- **Skewed data**: Use salting, broadcast join, AQE skew handling
- **Small files problem**: Use OPTIMIZE to compact

## Spark Configuration
```python
spark = SparkSession.builder \
    .appName("MyApp") \
    .config("spark.sql.shuffle.partitions", "200") \
    .config("spark.sql.adaptive.enabled", "true") \
    .config("spark.executor.memory", "8g") \
    .config("spark.executor.cores", "4") \
    .getOrCreate()
```

## Best Practices
✅ Use DataFrames (not RDDs) | ✅ Cache wisely (unpersist when done) | ✅ Partition data appropriately | ✅ Use columnar formats (Parquet/Delta) | ✅ Broadcast small tables in joins | ✅ Avoid UDFs (use built-in functions) | ✅ Enable AQE | ✅ Monitor Spark UI | ✅ Compact small files | ✅ Use Delta Lake for ACID
