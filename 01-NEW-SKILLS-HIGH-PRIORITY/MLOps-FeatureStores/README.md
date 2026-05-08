# MLOps & Feature Stores - High Priority

**Why This Matters:** MLOps Engineers earn $200k-$250k. Feature engineering is critical for ML at scale.

**Study Time:** Week 2 (Day 3-5) - 6-8 hours

**Good News:** You have strong data pipeline skills - MLOps is similar but for ML models!

---

## What is MLOps?

**Definition:** MLOps applies DevOps principles to Machine Learning - deploying, monitoring, and managing ML models in production.

**MLOps vs DataOps:**
| DataOps | MLOps |
|---------|-------|
| Data pipelines | ML model pipelines |
| Data quality | Model quality (accuracy, drift) |
| ETL monitoring | Model monitoring |
| Data versioning | Model + data versioning |

**Your Role as Data Engineer:**
- Build feature pipelines (real-time + batch)
- Create feature stores
- Ensure data quality for models
- Monitor data drift
- Scale inference pipelines

**You DON'T need to:** Build ML models (that's data scientists' job)
**You DO need to:** Build infrastructure for ML models to run in production

---

## Core MLOps Concepts

### 1. Feature Engineering
**What:** Transforming raw data into features (inputs) for ML models

**Example:**
```python
# Raw data
customer_id, purchase_timestamp, amount

# Features for churn prediction model
- total_purchases_last_30_days
- avg_purchase_amount_last_90_days
- days_since_last_purchase
- customer_lifetime_value
```

**Challenges:**
- **Feature/Training Skew:** Features in production different from training
- **Real-time requirements:** Need features with low latency
- **Consistency:** Same logic for batch (training) and real-time (serving)

### 2. Feature Store
**What:** Centralized repository for feature data, serving both training and inference

**Why Need It:**
- **Reusability:** Multiple teams/models use same features
- **Consistency:** Same features for training and serving
- **Low latency:** Pre-computed features ready for inference
- **Monitoring:** Track feature drift

**Architecture:**
```
Raw Data → Feature Pipeline → Feature Store → Training (batch)
                                           → Serving (real-time)
```

**Popular Feature Stores:**
- **Feast** (Open source, most popular)
- **Tecton** (Managed, built on Feast)
- **Databricks Feature Store**
- **AWS SageMaker Feature Store**
- **GCP Vertex AI Feature Store**

### 3. Model Deployment Patterns
**Batch Inference:**
- Run predictions on schedule (daily, hourly)
- Store results in database
- Use when real-time not needed
- Example: Churn predictions, recommendations

**Real-time Inference:**
- Predict on-demand via API
- Low latency required (<100ms often)
- Example: Fraud detection, dynamic pricing

**Streaming Inference:**
- Predict on streaming data (Kafka)
- Combine batch features + real-time features
- Example: Real-time recommendations

### 4. Model Monitoring
**What to Monitor:**
- **Data Drift:** Input distribution changes (features look different)
- **Concept Drift:** Relationship between features and target changes
- **Model Performance:** Accuracy degrades over time
- **System Metrics:** Latency, throughput, errors

**Your Responsibility:**
- Monitor feature quality
- Detect data drift
- Alert when features look anomalous
- Ensure feature pipeline SLAs

---

## Study Resources (8 hours total)

### Day 3: MLOps Fundamentals (3 hours)

**Watch/Read (2 hours):**
- [Course] "Introduction to Machine Learning in Production" - Coursera (Week 1 only)
  https://www.coursera.org/learn/introduction-to-machine-learning-in-production

- [Article] "MLOps: What It Is and Why It Matters" (30 min)
  https://ml-ops.org/content/mlops-principles

- [Video] "MLOps Explained" by Databricks (30 min)
  https://www.youtube.com/watch?v=Nev07rp4iI0

**Hands-On (1 hour):**
- Set up simple ML pipeline with scikit-learn
- Deploy model with Flask API
- Test inference endpoint

### Day 4: Feature Stores (3 hours)

**Watch/Read (1.5 hours):**
- [Docs] Feast documentation - Concepts (1 hour)
  https://docs.feast.dev/getting-started/concepts

- [Article] "What is a Feature Store?" by Tecton (30 min)
  https://www.tecton.ai/blog/what-is-a-feature-store/

**Hands-On (1.5 hours):**
- Install Feast locally
- Follow Feast quickstart
- Create feature repository
- Register features
- Materialize features to online store
- Retrieve features for inference

### Day 5: Advanced Topics + Integration (2 hours)

**Watch/Read (1 hour):**
- [Article] "Real-time Feature Engineering with Kafka and Feast" (30 min)
  Search recent articles

- [Docs] Databricks Feature Store guide (30 min)
  https://docs.databricks.com/machine-learning/feature-store/

**Hands-On (1 hour):**
- Design feature pipeline for your domain
- Create architecture diagram
- Document feature definitions
- Plan batch + real-time features

---

## Hands-On Project

### Project: Customer Churn Prediction Features

**Scenario:** Build feature pipeline for churn prediction model

**Features to Build:**
```python
# Batch Features (Spark/dbt)
- total_purchases_30d
- avg_purchase_amount_90d
- customer_lifetime_value
- days_since_registration
- preferred_category

# Real-time Features (Kafka/Flink)
- days_since_last_purchase
- last_login_timestamp
- cart_abandonment_count_7d
```

**Architecture:**
```
Batch Path:
Orders DB → Spark → Feature Pipeline → Feast (Offline Store: Parquet)
                                     → Model Training

Real-time Path:
Kafka Events → Flink → Feature Pipeline → Feast (Online Store: Redis)
                                        → Model Serving API
```

**Implementation Steps:**

1. **Set up Feast** (30 min)
```python
# feature_repo/features.py
from feast import Entity, Feature, FeatureView, FileSource, ValueType

customer = Entity(name="customer_id", value_type=ValueType.INT64)

customer_features = FeatureView(
    name="customer_stats",
    entities=["customer_id"],
    features=[
        Feature(name="total_purchases_30d", dtype=ValueType.INT64),
        Feature(name="avg_purchase_amount", dtype=ValueType.DOUBLE),
    ],
    batch_source=FileSource(
        path="data/customer_features.parquet",
        timestamp_field="event_timestamp",
    ),
)
```

2. **Create batch feature pipeline** (1.5 hours)
```python
# Spark job to compute features
from pyspark.sql import functions as F

customer_features = (
    orders
    .groupBy("customer_id")
    .agg(
        F.count("order_id").alias("total_purchases_30d"),
        F.avg("order_amount").alias("avg_purchase_amount"),
        F.max("order_timestamp").alias("last_purchase_date")
    )
)

# Write to feature store
customer_features.write.parquet("feature_store/customer_features/")
```

3. **Create real-time feature pipeline** (1.5 hours)
```python
# Flink or Spark Streaming
kafka_stream
  .groupBy("customer_id")
  .window(tumbling(days=7))
  .agg(count("cart_id").as("cart_abandonment_count_7d"))
  .to(redis_sink)  # Online feature store
```

4. **Retrieve features** (30 min)
```python
from feast import FeatureStore

store = FeatureStore(repo_path=".")

# For training (batch)
training_df = store.get_historical_features(
    entity_df=customer_ids_with_timestamps,
    features=["customer_stats:total_purchases_30d",
              "customer_stats:avg_purchase_amount"],
).to_df()

# For inference (real-time)
online_features = store.get_online_features(
    features=["customer_stats:total_purchases_30d"],
    entity_rows=[{"customer_id": 12345}],
).to_dict()
```

**Time:** 4-5 hours

**Outcome:** Working feature store demo to discuss in interviews

---

## Interview Preparation

### Common Interview Questions:

**1. "What is a feature store and why use it?"**

**Answer:**
A feature store is a centralized repository that:
1. Stores feature data (offline for training, online for serving)
2. Ensures consistency between training and serving
3. Enables feature reusability across teams/models
4. Provides low-latency feature access for real-time inference

**Why use it:**
- **Consistency:** Same features for training and serving (prevents training/serving skew)
- **Reusability:** Data science team A computes features once, team B reuses them
- **Speed:** Pre-computed features ready for inference
- **Monitoring:** Track feature drift, data quality

**Optum example:** "For RQNS, feature store would centralize member/provider features used by multiple ML models (risk score, fraud detection, recommendations)."

---

**2. "How would you design a real-time feature pipeline?"**

**Answer:**

**Architecture:**
```
Real-time Events (Kafka) → Stream Processing (Flink/Spark)
                        → Feature Store (Redis)
                        → ML Model API
```

**Components:**
1. **Event Stream:** Kafka topics with user events, transactions, etc.
2. **Stream Processor:** Flink or Spark Streaming
   - Aggregate features (windowing)
   - Join with batch features
   - Compute derived features
3. **Online Store:** Redis or DynamoDB (low latency)
4. **Feature Serving:** API retrieves features for inference

**Challenges & Solutions:**
- **Latency:** Use Redis for <10ms lookups, caching
- **Freshness:** Streaming updates features continuously
- **Scale:** Partition by entity_id (customer_id), horizontal scaling
- **Consistency:** Use same transformation logic as batch (e.g., shared dbt macros)

**Example:**
```python
# Flink SQL for real-time feature
SELECT
    customer_id,
    COUNT(*) as purchases_last_hour,
    AVG(amount) as avg_amount_last_hour,
    TUMBLE_END(event_time, INTERVAL '1' HOUR) as window_end
FROM kafka_purchases
GROUP BY customer_id, TUMBLE(event_time, INTERVAL '1' HOUR)
```

---

**3. "Explain training/serving skew and how to prevent it"**

**Answer:**

**What it is:**
Features computed differently for training vs serving, causing model performance degradation in production.

**Common Causes:**
- Different code for batch (training) vs real-time (serving)
- Different data sources
- Different transformation logic
- Different aggregation windows

**Prevention:**
1. **Use Feature Store:** Same feature definitions for training/serving
2. **Shared Code:** Use same transformation logic (dbt models, Python functions)
3. **Testing:** Compare batch vs real-time feature outputs
4. **Monitoring:** Alert on feature distribution drift

**Example at Optum:**
```
❌ Bad:
Training: SQL query computes "avg_claim_amount_30d"
Serving: Python code computes "avg_claim_amount_30d" (different logic!)

✓ Good:
Both use Feast feature "claim_stats:avg_claim_amount_30d"
Single Spark pipeline computes it, serves via online store
```

---

**4. "How do you monitor ML models in production?"**

**Answer:**

**Monitor 4 Layers:**

**1. Data Quality:**
- Null rates, schema changes
- Feature distribution (compare to training)
- Data drift (statistical tests)

**2. Model Performance:**
- Accuracy, precision, recall (if labels available)
- Prediction distribution
- Concept drift

**3. System Metrics:**
- Latency (p50, p95, p99)
- Throughput (requests/sec)
- Error rates
- Resource usage (CPU, memory)

**4. Business Metrics:**
- ROI of model predictions
- User engagement
- Cost savings

**Tools:**
- **Data Drift:** Great Expectations, Monte Carlo
- **Model Monitoring:** MLflow, Weights & Biases
- **System:** Prometheus + Grafana
- **Logging:** ELK stack

**Alerting:**
- Feature drift > threshold → retrain model
- Latency > SLA → scale up
- Error rate spike → rollback

**Your responsibility as DE:**
- Build feature quality monitoring
- Detect data drift
- Ensure feature pipeline SLAs
- Alert data scientists when drift detected

---

**5. "Design an ML platform for fraud detection"**

**Answer:**

**Requirements:**
- Real-time predictions (<100ms)
- High throughput (thousands of transactions/sec)
- Low false positives (don't block legitimate transactions)

**Architecture:**
```
Transaction Event (Kafka)
  ↓
Feature Pipeline (Flink)
  ├─→ Batch features from Feature Store (Redis)
  ├─→ Real-time features (last 1h activity)
  └─→ Combined feature vector
       ↓
ML Model (deployed on Kubernetes)
  ↓
Fraud Score → Decision Engine
  ↓
Block/Allow + Monitoring
```

**Components:**

**1. Feature Pipeline:**
- **Batch Features (pre-computed):**
  - Customer risk score
  - Historical fraud rate
  - Device fingerprint
- **Real-time Features (streaming):**
  - Transactions in last hour
  - Velocity (amount/time)
  - Location changes

**2. Feature Store:**
- **Offline (S3/Delta):** Training data
- **Online (Redis):** Low-latency serving

**3. Model Serving:**
- REST API on Kubernetes
- Autoscaling based on load
- Model versioning (A/B testing)

**4. Monitoring:**
- Latency dashboard (target: p99 <100ms)
- False positive rate
- Feature drift detection
- Model performance (precision/recall)

**5. Feedback Loop:**
- Labeled fraud cases → retrain model weekly
- Feature importance → add new features

**Technologies:**
- Kafka + Flink (real-time)
- Feast (feature store)
- Redis (online store)
- MLflow (model registry)
- Kubernetes (serving)
- Prometheus + Grafana (monitoring)

**Optum Example:** "Similar to what we'd build for healthcare fraud detection in claims processing."

---

## Key Technologies

### Must Know:
- **Feast** - Open source feature store
- **MLflow** - Model registry, experiment tracking
- **Kubernetes** - Model deployment
- **Prometheus + Grafana** - Monitoring

### Good to Know:
- **Tecton** - Managed feature store
- **Databricks Feature Store** - If using Databricks
- **Kubeflow** - ML on Kubernetes
- **SageMaker** - AWS ML platform

---

## Quick Reference Cheat Sheet

### Feast Workflow:
```bash
# 1. Initialize
feast init feature_repo
cd feature_repo

# 2. Define features (features.py)
# 3. Apply to registry
feast apply

# 4. Materialize features to online store
feast materialize-incremental $(date +%Y-%m-%d)

# 5. Get features for training
python get_historical_features.py

# 6. Get features for serving
python get_online_features.py
```

### Feature Engineering Patterns:
```python
# Aggregation features
- count, sum, avg, min, max over time windows
- Example: total_purchases_30d, avg_amount_90d

# Trend features
- Change over time
- Example: purchases_this_month vs last_month

# Recency features
- Days since event
- Example: days_since_last_login

# Categorical features
- One-hot encoding, embeddings
- Example: preferred_category, user_segment
```

---

## Practice Problems

**Problem 1:** Design feature pipeline for recommendation system (2 hours)
- User features: viewing history, preferences
- Item features: popularity, category
- Real-time: current session activity

**Problem 2:** Implement feature monitoring (1.5 hours)
- Detect distribution drift
- Alert on anomalies
- Dashboard for feature health

**Problem 3:** Batch + Real-time feature consistency (2 hours)
- Implement same feature in batch (Spark) and streaming (Flink)
- Test for consistency
- Use Feast for serving

---

## Week 2 Success Metrics

**Knowledge:**
- [ ] Understand MLOps vs DataOps
- [ ] Know what feature store is and why use it
- [ ] Familiar with training/serving skew
- [ ] Can explain ML monitoring

**Hands-On:**
- [ ] Installed and used Feast
- [ ] Created feature definitions
- [ ] Retrieved historical features
- [ ] Served online features

**Interview Ready:**
- [ ] Can design real-time feature pipeline
- [ ] Explain feature store architecture
- [ ] Describe ML monitoring strategy
- [ ] Have feature store demo project

---

**Resume Addition:**
```
• Built feature pipeline using Feast for ML model serving with sub-100ms
  latency, supporting both batch and real-time feature computation
```

**Next:** Combine this with your Kafka/Spark expertise for powerful interview examples
