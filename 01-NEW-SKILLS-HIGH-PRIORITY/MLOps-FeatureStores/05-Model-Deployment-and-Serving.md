# Chapter 05: Model Deployment and Serving

**Deploying ML Models to Production**

## Deployment Patterns

### 1. Batch Prediction
**Use case:** Daily/hourly predictions for all users
**Example:** Churn prediction for all customers overnight

```python
# Load model
model = mlflow.sklearn.load_model("models:/churn_model/Production")

# Get all customers
customers = spark.read.table("customers")

# Get features
features = feature_store.get_features(customers)

# Predict
predictions = model.predict(features)

# Save results
predictions.write.table("predictions.daily_churn")
```

### 2. Real-Time (Online) Prediction
**Use case:** API endpoint for instant predictions
**Example:** Fraud detection at transaction time

```python
from fastapi import FastAPI
import mlflow

app = FastAPI()
model = mlflow.sklearn.load_model("models:/fraud_model/Production")

@app.post("/predict")
def predict(transaction: dict):
    features = extract_features(transaction)
    prediction = model.predict([features])
    return {"fraud_probability": float(prediction[0])}
```

### 3. Streaming Prediction
**Use case:** Process events from queue
**Example:** Real-time recommendations from clickstream

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

# Read stream
stream = spark.readStream.format("kafka") \
    .option("subscribe", "user_events") \
    .load()

# Apply model (batch UDF)
predictions = stream.withColumn(
    "recommendation",
    predict_udf("user_id", "context")
)

# Write results
predictions.writeStream \
    .format("kafka") \
    .option("topic", "recommendations") \
    .start()
```

## Model Serving Frameworks

### 1. FastAPI (Python)
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class PredictionRequest(BaseModel):
    features: list

@app.post("/predict")
def predict(request: PredictionRequest):
    prediction = model.predict([request.features])
    return {"prediction": prediction[0]}

# Run: uvicorn main:app --reload
```

### 2. Flask (Python)
```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/predict', methods=['POST'])
def predict():
    data = request.json
    prediction = model.predict([data['features']])
    return jsonify({'prediction': prediction.tolist()})

# Run: python app.py
```

### 3. BentoML
```python
import bentoml
from bentoml.io import JSON

# Save model to BentoML
bentoml.sklearn.save_model("churn_model", model)

# Create service
@bentoml.service(name="churn_predictor")
class ChurnPredictor:
    model = bentoml.sklearn.get("churn_model:latest")
    
    @bentoml.api
    def predict(self, input_data: JSON) -> JSON:
        return self.model.predict(input_data)

# Deploy
# bentoml serve service:ChurnPredictor
```

### 4. Seldon Core (Kubernetes)
```yaml
apiVersion: machinelearning.seldon.io/v1
kind: SeldonDeployment
metadata:
  name: churn-model
spec:
  predictors:
  - graph:
      name: classifier
      implementation: SKLEARN_SERVER
      modelUri: s3://models/churn-model
    name: default
    replicas: 3
```

## Model Packaging

### 1. Docker Container
```dockerfile
FROM python:3.9-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy model and code
COPY model.pkl .
COPY app.py .

# Run API
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 2. MLflow Model
```python
import mlflow.pyfunc

class ModelWrapper(mlflow.pyfunc.PythonModel):
    def load_context(self, context):
        import joblib
        self.model = joblib.load(context.artifacts["model"])
    
    def predict(self, context, model_input):
        return self.model.predict(model_input)

# Log with dependencies
mlflow.pyfunc.log_model(
    artifact_path="model",
    python_model=ModelWrapper(),
    artifacts={"model": "model.pkl"},
    conda_env={
        "dependencies": [
            "python=3.9",
            "scikit-learn=1.0.2",
            "pandas=1.4.0"
        ]
    }
)
```

## A/B Testing

```python
import random

def route_traffic(user_id):
    """Route 10% to model B, 90% to model A"""
    if hash(user_id) % 100 < 10:
        return "model_b"
    return "model_a"

@app.post("/predict")
def predict(user_id: str, features: list):
    model_version = route_traffic(user_id)
    
    if model_version == "model_b":
        model = load_model("model_b")
        log_experiment(user_id, "model_b")
    else:
        model = load_model("model_a")
        log_experiment(user_id, "model_a")
    
    prediction = model.predict([features])
    return {"prediction": prediction[0], "model": model_version}
```

## Model Registry

```python
import mlflow

# Register model
mlflow.register_model(
    model_uri="runs:/abc123/model",
    name="churn_predictor"
)

# Transition stages
client = mlflow.tracking.MlflowClient()

# Staging
client.transition_model_version_stage(
    name="churn_predictor",
    version=2,
    stage="Staging"
)

# Production (after validation)
client.transition_model_version_stage(
    name="churn_predictor",
    version=2,
    stage="Production"
)

# Archive old version
client.transition_model_version_stage(
    name="churn_predictor",
    version=1,
    stage="Archived"
)
```

## Optimization

### 1. Model Compression
```python
# Quantization (reduce precision)
import torch

model_fp32 = torch.load("model.pth")
model_int8 = torch.quantization.quantize_dynamic(
    model_fp32,
    {torch.nn.Linear},
    dtype=torch.qint8
)

# Result: 4x smaller, 2-4x faster
```

### 2. Caching
```python
from functools import lru_cache
import redis

redis_client = redis.Redis()

def predict_with_cache(features):
    # Create cache key
    key = hash(str(features))
    
    # Check cache
    cached = redis_client.get(key)
    if cached:
        return cached
    
    # Predict
    prediction = model.predict([features])
    
    # Cache (TTL = 1 hour)
    redis_client.setex(key, 3600, prediction)
    
    return prediction
```

### 3. Batch Prediction API
```python
@app.post("/predict_batch")
def predict_batch(requests: List[dict]):
    """Handle multiple predictions at once"""
    features = [r['features'] for r in requests]
    predictions = model.predict(features)  # Vectorized
    return {"predictions": predictions.tolist()}
```

## Summary

✅ **Deployment Patterns:** Batch, real-time, streaming
✅ **Serving Frameworks:** FastAPI, BentoML, Seldon
✅ **Model Packaging:** Docker, MLflow
✅ **A/B Testing:** Traffic routing and experimentation
✅ **Optimization:** Compression, caching, batching

**Continue to Chapter 06 for Monitoring! 🚀**
