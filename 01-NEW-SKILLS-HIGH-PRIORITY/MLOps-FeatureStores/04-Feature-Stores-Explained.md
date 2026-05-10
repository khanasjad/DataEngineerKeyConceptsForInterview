# Chapter 04: Feature Stores Explained

**Centralized Feature Management for ML at Scale**

---

## What is a Feature Store?

### Simple Definition

A **Feature Store** is a centralized repository for storing, managing, and serving features for machine learning models.

### Real-Life Analogy

**Without Feature Store** = Every restaurant creates its own ingredients from scratch
- Bakery: Makes flour, grows wheat
- Pizza place: Makes cheese, milks cows
- Result: Duplicated effort, inconsistent quality

**With Feature Store** = Central grocery store
- All restaurants get ingredients from one place
- Consistent quality
- Reusable across restaurants
- Efficient

---

## Why Feature Stores?

### Problems Without Feature Store

**1. Feature Duplication:**
```
Team A: Computes "customer_lifetime_value"
Team B: Also computes "customer_lifetime_value" (slightly different)
Team C: Also computes "customer_lifetime_value" (different again)

Result: 3x the work, inconsistent features
```

**2. Training-Serving Skew:**
```
Training (Python/Spark):
features = compute_features(data)  # Batch processing

Production (Java API):
features = compute_features_differently(data)  # Real-time
# ↑ Different implementation = Different results!
```

**3. Feature Discovery:**
```
Data Scientist: "Has anyone computed customer churn probability?"
*Asks 10 people, searches Slack, checks repos*
Result: Wastes days, might rebuild existing feature
```

---

## Feature Store Architecture

### High-Level Components

```
┌──────────────────────────────────────────┐
│     DATA SOURCES                         │
│  (Databases, Data Lake, Streams)        │
└────────────┬─────────────────────────────┘
             ↓
┌──────────────────────────────────────────┐
│   FEATURE ENGINEERING PIPELINES          │
│  (Batch: Spark/dbt, Stream: Flink)      │
└────────────┬─────────────────────────────┘
             ↓
┌──────────────────────────────────────────┐
│        FEATURE STORE                     │
│  ┌────────────────────────────────────┐ │
│  │  OFFLINE STORE                     │ │
│  │  (Parquet, Delta Lake, BigQuery)   │ │
│  │  - Training data                    │ │
│  │  - Historical features             │ │
│  └────────────────────────────────────┘ │
│  ┌────────────────────────────────────┐ │
│  │  ONLINE STORE                      │ │
│  │  (Redis, DynamoDB, Cassandra)      │ │
│  │  - Real-time serving               │ │
│  │  - Low latency (<10ms)             │ │
│  └────────────────────────────────────┘ │
│  ┌────────────────────────────────────┐ │
│  │  FEATURE REGISTRY                  │ │
│  │  - Metadata                        │ │
│  │  - Versions                        │ │
│  │  - Lineage                         │ │
│  └────────────────────────────────────┘ │
└────────────┬─────────────────────────────┘
             ↓
┌──────────────────────────────────────────┐
│     CONSUMERS                            │
│  - Model Training                        │
│  - Model Serving                         │
│  - Analytics                             │
└──────────────────────────────────────────┘
```

---

## Offline vs Online Stores

### Offline Store (Training)

**Purpose:** Provide historical features for training

**Storage:** Columnar formats (Parquet, Delta Lake)

**Query Pattern:**
```python
# Get all features for last 6 months
features = feature_store.get_historical_features(
    entity_rows=customers_df,
    feature_refs=[
        "customer_features:lifetime_value",
        "customer_features:days_since_purchase",
        "customer_features:total_orders"
    ],
    start_date="2023-07-01",
    end_date="2024-01-01"
)
```

**Characteristics:**
- Large data volume
- Batch queries
- Latency: seconds to minutes OK
- Point-in-time correctness

---

### Online Store (Serving)

**Purpose:** Provide latest features for real-time predictions

**Storage:** Key-value stores (Redis, DynamoDB)

**Query Pattern:**
```python
# Get features for one customer (real-time)
features = feature_store.get_online_features(
    entity_rows=[{"customer_id": "C123"}],
    feature_refs=[
        "customer_features:lifetime_value",
        "customer_features:days_since_purchase"
    ]
)
```

**Characteristics:**
- Small data volume (latest only)
- Single-row queries
- Latency: <10ms required
- High throughput

---

## Popular Feature Stores

### 1. Feast (Open-Source)

**Installation:**
```bash
pip install feast
```

**Define Features:**
```python
# feature_repo/features.py
from feast import Entity, Feature, FeatureView, ValueType
from feast.data_source import FileSource
from datetime import timedelta

# Define entity
customer = Entity(
    name="customer_id",
    value_type=ValueType.STRING,
    description="Customer ID"
)

# Define data source
customer_source = FileSource(
    path="data/customer_features.parquet",
    event_timestamp_column="event_timestamp"
)

# Define feature view
customer_features = FeatureView(
    name="customer_features",
    entities=["customer_id"],
    ttl=timedelta(days=1),
    features=[
        Feature(name="lifetime_value", dtype=ValueType.FLOAT),
        Feature(name="total_orders", dtype=ValueType.INT64),
        Feature(name="days_since_purchase", dtype=ValueType.INT64)
    ],
    online=True,
    source=customer_source
)
```

**Apply Configuration:**
```bash
feast apply
```

**Materialize to Online Store:**
```bash
feast materialize-incremental $(date +%Y-%m-%d)
```

**Retrieve Features:**
```python
from feast import FeatureStore

store = FeatureStore(repo_path=".")

# Online features
features = store.get_online_features(
    features=[
        "customer_features:lifetime_value",
        "customer_features:total_orders"
    ],
    entity_rows=[{"customer_id": "C123"}]
).to_dict()

print(features)
# {'customer_id': ['C123'], 'lifetime_value': [5000.0], 'total_orders': [15]}
```

---

### 2. Tecton (Managed)

```python
from tecton import Entity, BatchSource, FeatureView
from datetime import datetime, timedelta

# Define entity
customer = Entity(
    name="customer",
    join_keys=["customer_id"]
)

# Define data source
transactions = BatchSource(
    name="transactions",
    batch_config=SparkBatchConfig(
        data_source=ParquetConfig(
            uri="s3://bucket/transactions/"
        )
    )
)

# Define feature view with transformation
@FeatureView(
    sources=[transactions],
    entities=[customer],
    mode="spark_sql",
    online=True,
    offline=True,
    feature_start_time=datetime(2023, 1, 1)
)
def customer_transaction_stats(transactions):
    return f"""
        SELECT
            customer_id,
            COUNT(*) as transaction_count,
            SUM(amount) as total_spent,
            AVG(amount) as avg_transaction_value
        FROM
            {transactions}
        WHERE
            timestamp >= NOW() - INTERVAL 30 DAYS
        GROUP BY
            customer_id
    """
```

---

### 3. Databricks Feature Store

```python
from databricks import feature_store

fs = feature_store.FeatureStoreClient()

# Create feature table
fs.create_table(
    name="customer_features",
    primary_keys=["customer_id"],
    df=features_df,
    schema=features_df.schema,
    description="Customer behavioral features"
)

# Write features
fs.write_table(
    name="customer_features",
    df=new_features_df,
    mode="merge"
)

# Read features (training)
training_set = fs.create_training_set(
    df=labels_df,
    feature_lookups=[
        feature_store.FeatureLookup(
            table_name="customer_features",
            feature_names=["lifetime_value", "total_orders"],
            lookup_key="customer_id"
        )
    ],
    label="churn"
)

training_df = training_set.load_df()
```

---

## Key Concepts

### 1. Entities

**Definition:** Objects that features describe (customer, product, store)

**Example:**
```python
customer = Entity(
    name="customer",
    value_type=ValueType.STRING,
    description="Unique customer identifier"
)

product = Entity(
    name="product",
    value_type=ValueType.STRING,
    description="Product SKU"
)
```

---

### 2. Feature Views

**Definition:** Logical group of features from same source

**Example:**
```python
customer_demographics = FeatureView(
    name="customer_demographics",
    entities=["customer"],
    features=[
        Feature("age", ValueType.INT64),
        Feature("gender", ValueType.STRING),
        Feature("country", ValueType.STRING)
    ],
    source=customer_source
)

customer_behavior = FeatureView(
    name="customer_behavior",
    entities=["customer"],
    features=[
        Feature("lifetime_value", ValueType.FLOAT),
        Feature("purchase_frequency", ValueType.FLOAT)
    ],
    source=transactions_source
)
```

---

### 3. Point-in-Time Correctness

**Problem:**
```
Training time: 2024-01-15
Need features as of that date (not future!)

BAD: Join on customer_id (gets latest features, includes future)
GOOD: Join on customer_id AND timestamp <= 2024-01-15
```

**Implementation:**
```python
# Feature store automatically handles this
historical_features = store.get_historical_features(
    entity_rows=events_df,  # Has customer_id and event_timestamp
    feature_refs=["customer_features:lifetime_value"]
)

# Returns features AS OF each event_timestamp
# No data leakage!
```

---

### 4. Feature Serving Patterns

**Batch Serving:**
```python
# For batch predictions (e.g., daily churn scores)
customers = spark.read.table("customers")

features = store.get_historical_features(
    entity_rows=customers,
    feature_refs=all_features
)

predictions = model.predict(features)
```

**Online Serving:**
```python
# For real-time predictions (API endpoint)
@app.route('/predict', methods=['POST'])
def predict():
    customer_id = request.json['customer_id']
    
    # Get features from online store
    features = store.get_online_features(
        entity_rows=[{"customer_id": customer_id}],
        feature_refs=all_features
    ).to_dict()
    
    # Predict
    prediction = model.predict([features])
    
    return {"churn_probability": prediction[0]}
```

---

## Building a Simple Feature Store

### DIY Feature Store (Learning)

```python
import pandas as pd
import redis
from datetime import datetime

class SimpleFeatureStore:
    def __init__(self, offline_path, redis_host='localhost'):
        self.offline_path = offline_path
        self.redis_client = redis.Redis(host=redis_host, decode_responses=True)
    
    def write_offline_features(self, df, feature_view_name):
        """Write features to offline store (Parquet)"""
        path = f"{self.offline_path}/{feature_view_name}.parquet"
        df.to_parquet(path, index=False)
        print(f"Wrote {len(df)} rows to {path}")
    
    def materialize_to_online(self, feature_view_name, entity_key):
        """Copy latest features to online store (Redis)"""
        # Read from offline
        path = f"{self.offline_path}/{feature_view_name}.parquet"
        df = pd.read_parquet(path)
        
        # Get latest per entity
        df_latest = df.sort_values('timestamp').groupby(entity_key).tail(1)
        
        # Write to Redis
        for _, row in df_latest.iterrows():
            key = f"{feature_view_name}:{row[entity_key]}"
            value = row.drop([entity_key, 'timestamp']).to_json()
            self.redis_client.set(key, value)
        
        print(f"Materialized {len(df_latest)} entities to online store")
    
    def get_online_features(self, feature_view_name, entity_id):
        """Get features from online store"""
        key = f"{feature_view_name}:{entity_id}"
        value = self.redis_client.get(key)
        
        if value:
            return pd.read_json(value, typ='series').to_dict()
        return None
    
    def get_historical_features(self, feature_view_name, entity_ids, as_of_date):
        """Get features from offline store (point-in-time)"""
        path = f"{self.offline_path}/{feature_view_name}.parquet"
        df = pd.read_parquet(path)
        
        # Filter by entities and date
        df_filtered = df[
            (df['customer_id'].isin(entity_ids)) &
            (df['timestamp'] <= as_of_date)
        ]
        
        # Get latest per entity
        df_latest = df_filtered.sort_values('timestamp').groupby('customer_id').tail(1)
        
        return df_latest

# Usage
store = SimpleFeatureStore(offline_path='./features')

# Write features
features_df = pd.DataFrame({
    'customer_id': ['C1', 'C2', 'C3'],
    'lifetime_value': [5000, 3000, 8000],
    'total_orders': [15, 8, 25],
    'timestamp': [datetime.now()] * 3
})

store.write_offline_features(features_df, 'customer_features')
store.materialize_to_online('customer_features', 'customer_id')

# Online serving
features = store.get_online_features('customer_features', 'C1')
print(features)  # {'lifetime_value': 5000, 'total_orders': 15}
```

---

## Best Practices

### 1. Feature Naming Convention

```python
# Format: {entity}_{metric}_{timewindow}_{aggregation}
customer_purchases_30d_sum
customer_purchases_30d_count
customer_purchases_30d_avg

product_views_7d_count
product_purchases_all_time_sum
```

---

### 2. Feature Documentation

```python
Feature(
    name="customer_lifetime_value",
    dtype=ValueType.FLOAT,
    labels={
        "owner": "data-team",
        "description": "Total revenue from customer",
        "sla": "daily",
        "pii": "false"
    }
)
```

---

### 3. Feature Monitoring

```python
def monitor_features(feature_store, feature_name):
    """Monitor feature quality"""
    
    features = feature_store.get_latest_features(feature_name)
    
    # Check nulls
    null_pct = features.isnull().sum() / len(features)
    if null_pct > 0.05:
        alert(f"{feature_name} has {null_pct:.2%} nulls")
    
    # Check distribution drift
    historical_mean = get_historical_stats(feature_name)['mean']
    current_mean = features.mean()
    
    if abs(current_mean - historical_mean) / historical_mean > 0.2:
        alert(f"{feature_name} drift detected")
```

---

## Summary

### Key Takeaways:

✅ **Feature Store:** Central repository for ML features
✅ **Benefits:** Reusability, consistency, discovery, governance
✅ **Components:** Offline store (training), Online store (serving), Registry (metadata)
✅ **Popular Tools:** Feast (open-source), Tecton (managed), Databricks
✅ **Key Concepts:** Entities, feature views, point-in-time correctness

### For Data Engineers:

- Build feature computation pipelines
- Maintain offline/online sync
- Ensure data quality
- Monitor feature freshness
- Optimize serving latency

---

**Continue to Chapter 05 to learn Model Deployment! 🚀**
