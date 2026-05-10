# Chapter 02: ML Lifecycle Management

**Managing the Complete Machine Learning Workflow**

---

## Table of Contents
1. [ML Project Workflow](#ml-project-workflow)
2. [Experiment Tracking](#experiment-tracking)
3. [Model Versioning](#model-versioning)
4. [CI/CD for ML](#cicd-for-ml)
5. [Pipeline Orchestration](#pipeline-orchestration)
6. [Best Practices](#best-practices)

---

## ML Project Workflow

### Complete Lifecycle

```
1. Problem Definition
   ├─ Business goal: Increase customer retention
   ├─ ML task: Predict churn probability
   └─ Success metric: 20% reduction in churn

2. Data Collection
   ├─ Customer demographics
   ├─ Purchase history
   ├─ Support interactions
   └─ Product usage logs

3. Exploratory Data Analysis (EDA)
   ├─ Data quality checks
   ├─ Statistical analysis
   ├─ Feature distributions
   └─ Correlation analysis

4. Data Preparation
   ├─ Clean missing values
   ├─ Handle outliers
   ├─ Encode categorical variables
   └─ Train/validation/test split

5. Feature Engineering
   ├─ Create derived features
   ├─ Feature selection
   ├─ Feature scaling
   └─ Feature validation

6. Model Development
   ├─ Baseline model
   ├─ Try multiple algorithms
   ├─ Hyperparameter tuning
   └─ Cross-validation

7. Model Evaluation
   ├─ Test set performance
   ├─ Business metrics
   ├─ Fairness checks
   └─ Error analysis

8. Model Deployment
   ├─ Package model
   ├─ Deploy to staging
   ├─ A/B testing
   └─ Deploy to production

9. Monitoring & Maintenance
   ├─ Performance tracking
   ├─ Drift detection
   ├─ Retraining
   └─ Model updates
```

---

## Experiment Tracking

### Why Track Experiments?

**Problem without tracking:**
```
Data Scientist: "I ran 50 experiments last week"
Manager: "Which one was best?"
Data Scientist: "Um... I think it was... maybe the XGBoost one?"
Manager: "What were the parameters?"
Data Scientist: "I don't remember..."
```

**Solution: Experiment tracking**

---

### What to Track

```
For each experiment:
├─ Code version (Git commit hash)
├─ Data version (dataset identifier)
├─ Parameters
│  ├─ learning_rate: 0.01
│  ├─ n_estimators: 100
│  └─ max_depth: 5
├─ Metrics
│  ├─ accuracy: 0.92
│  ├─ precision: 0.89
│  ├─ recall: 0.94
│  └─ f1_score: 0.91
├─ Artifacts
│  ├─ Model file
│  ├─ Feature importance plot
│  └─ Confusion matrix
├─ Environment
│  ├─ Python: 3.9
│  ├─ scikit-learn: 1.2.0
│  └─ Hardware: 8 CPU, 16GB RAM
└─ Metadata
   ├─ Experiment name
   ├─ Timestamp
   └─ Author
```

---

### MLflow - Experiment Tracking

**Installation:**
```bash
pip install mlflow
```

**Basic Usage:**
```python
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, precision_score

# Start MLflow run
with mlflow.start_run(run_name="rf_experiment_1"):
    
    # Log parameters
    n_estimators = 100
    max_depth = 10
    mlflow.log_param("n_estimators", n_estimators)
    mlflow.log_param("max_depth", max_depth)
    
    # Train model
    model = RandomForestClassifier(
        n_estimators=n_estimators,
        max_depth=max_depth,
        random_state=42
    )
    model.fit(X_train, y_train)
    
    # Evaluate
    y_pred = model.predict(X_test)
    accuracy = accuracy_score(y_test, y_pred)
    precision = precision_score(y_test, y_pred)
    
    # Log metrics
    mlflow.log_metric("accuracy", accuracy)
    mlflow.log_metric("precision", precision)
    
    # Log model
    mlflow.sklearn.log_model(model, "model")
    
    # Log artifacts (plots, files)
    import matplotlib.pyplot as plt
    plt.figure()
    # ... create plot ...
    plt.savefig("feature_importance.png")
    mlflow.log_artifact("feature_importance.png")
    
    print(f"Accuracy: {accuracy:.4f}")
```

---

**View Results:**
```bash
# Start MLflow UI
mlflow ui

# Open browser: http://localhost:5000
```

**UI shows:**
- All experiments and runs
- Compare metrics across runs
- View parameters and artifacts
- Download models

---

### Advanced MLflow Features

**1. Nested Runs (for pipelines):**
```python
with mlflow.start_run(run_name="full_pipeline") as parent_run:
    
    # Data preprocessing run
    with mlflow.start_run(run_name="preprocessing", nested=True):
        mlflow.log_param("scaler", "StandardScaler")
        X_scaled = preprocess(X)
    
    # Model training run
    with mlflow.start_run(run_name="training", nested=True):
        mlflow.log_param("model", "RandomForest")
        model = train(X_scaled, y)
        mlflow.log_metric("accuracy", 0.92)
```

---

**2. Autologging:**
```python
import mlflow
import mlflow.sklearn

# Enable autologging
mlflow.sklearn.autolog()

# Train model - MLflow logs everything automatically!
model = RandomForestClassifier()
model.fit(X_train, y_train)

# Automatically logs:
# - All parameters
# - Training metrics
# - Model
# - Feature importance
```

---

**3. Model Registry:**
```python
# Register model
mlflow.register_model(
    model_uri="runs:/abc123/model",
    name="churn_predictor"
)

# Transition to staging
client = mlflow.tracking.MlflowClient()
client.transition_model_version_stage(
    name="churn_predictor",
    version=1,
    stage="Staging"
)

# After testing, promote to production
client.transition_model_version_stage(
    name="churn_predictor",
    version=1,
    stage="Production"
)
```

---

## Model Versioning

### Why Version Models?

**Scenario:**
```
Monday: Deploy model v1 (accuracy: 90%)
Friday: Model performance drops to 70%!
Team: "Let's rollback to previous version"
Problem: Which version? What were the parameters?
```

**Solution: Proper versioning**

---

### Versioning Strategies

**1. Semantic Versioning**
```
v1.0.0 - Initial model
v1.1.0 - Added new features
v1.1.1 - Bug fix in preprocessing
v2.0.0 - Major algorithm change
```

**2. Timestamp-Based**
```
model_20240115_143022
model_20240116_091545
```

**3. Git Hash + Timestamp**
```
model_abc123_20240115
```

---

### Complete Example

```python
import mlflow
from datetime import datetime

class ModelVersionManager:
    def __init__(self, model_name):
        self.model_name = model_name
        self.client = mlflow.tracking.MlflowClient()
    
    def save_model(self, model, metrics, metadata):
        """Save model with version tracking"""
        
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        run_name = f"{self.model_name}_{timestamp}"
        
        with mlflow.start_run(run_name=run_name):
            # Log all metadata
            for key, value in metadata.items():
                mlflow.log_param(key, value)
            
            # Log metrics
            for metric_name, metric_value in metrics.items():
                mlflow.log_metric(metric_name, metric_value)
            
            # Log model
            mlflow.sklearn.log_model(
                model,
                artifact_path="model",
                registered_model_name=self.model_name
            )
            
            run_id = mlflow.active_run().info.run_id
            
        return run_id
    
    def get_production_model(self):
        """Load current production model"""
        
        model_uri = f"models:/{self.model_name}/Production"
        return mlflow.sklearn.load_model(model_uri)
    
    def compare_versions(self, version1, version2):
        """Compare two model versions"""
        
        v1_metrics = self.client.get_run(version1).data.metrics
        v2_metrics = self.client.get_run(version2).data.metrics
        
        comparison = {}
        for metric in v1_metrics.keys():
            comparison[metric] = {
                "version1": v1_metrics[metric],
                "version2": v2_metrics.get(metric, None),
                "improvement": v2_metrics.get(metric, 0) - v1_metrics[metric]
            }
        
        return comparison

# Usage
manager = ModelVersionManager("customer_churn")

# Save new model
run_id = manager.save_model(
    model=trained_model,
    metrics={"accuracy": 0.92, "f1": 0.90},
    metadata={"algorithm": "XGBoost", "features": 25}
)

# Load production model
prod_model = manager.get_production_model()
```

---

## CI/CD for ML

### Traditional CI/CD vs ML CI/CD

**Traditional Software:**
```
Code → Build → Test → Deploy
```

**ML System:**
```
Code → Data Validation → Model Training → Model Validation → Deploy
  ↑         ↑                   ↑               ↑              ↑
 Git    Schema checks      Metrics check    A/B test      Serving
```

---

### ML Pipeline Stages

**1. Continuous Integration (CI)**
```yaml
# .github/workflows/ml_ci.yml
name: ML CI Pipeline

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Python
        uses: actions/setup-python@v2
        with:
          python-version: 3.9
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
      
      - name: Run data validation tests
        run: |
          pytest tests/test_data_validation.py
      
      - name: Run model training tests
        run: |
          pytest tests/test_model.py
      
      - name: Check code quality
        run: |
          flake8 src/
          black --check src/
```

---

**2. Continuous Training (CT)**
```python
# Automated training pipeline
def training_pipeline():
    """Runs on schedule or trigger"""
    
    # 1. Validate data
    data = load_data()
    if not validate_data_schema(data):
        raise ValueError("Data validation failed")
    
    # 2. Check data quality
    quality_report = check_data_quality(data)
    if quality_report['null_percentage'] > 5:
        alert_team("High null percentage in data")
        return
    
    # 3. Train model
    model = train_model(data)
    
    # 4. Evaluate
    metrics = evaluate_model(model)
    
    # 5. Compare with production
    prod_model = load_production_model()
    prod_metrics = evaluate_model(prod_model)
    
    # 6. Deploy if better
    if metrics['f1'] > prod_metrics['f1'] + 0.02:  # 2% improvement threshold
        deploy_model(model, stage="staging")
        notify_team("New model deployed to staging")
    else:
        log_message("New model did not improve performance")
```

---

**3. Continuous Deployment (CD)**
```python
# Automated deployment pipeline
def deployment_pipeline(model_run_id):
    """Deploy model after validation"""
    
    # 1. Load model from MLflow
    model = mlflow.sklearn.load_model(f"runs:/{model_run_id}/model")
    
    # 2. Run integration tests
    if not run_integration_tests(model):
        raise Exception("Integration tests failed")
    
    # 3. Deploy to staging
    deploy_to_staging(model)
    
    # 4. Run A/B test
    ab_results = run_ab_test(
        model_a="production",
        model_b=model_run_id,
        duration_hours=24,
        traffic_split=0.1  # 10% to new model
    )
    
    # 5. Evaluate A/B test
    if ab_results['model_b_f1'] > ab_results['model_a_f1']:
        # Promote to production
        promote_to_production(model_run_id)
        notify_team("Model promoted to production")
    else:
        # Rollback
        rollback_deployment(model_run_id)
        notify_team("A/B test failed, rolled back")
```

---

## Pipeline Orchestration

### Why Orchestration?

**Manual process:**
```
1. Data scientist runs training script
2. Waits for it to finish
3. Manually evaluates results
4. Emails results to team
5. If good, manually deploys
```

**Problems:** Slow, error-prone, not reproducible

---

### Airflow for ML Pipelines

**DAG Structure:**
```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'ml-team',
    'depends_on_past': False,
    'email_on_failure': True,
    'email': ['ml-team@company.com'],
    'retries': 1,
    'retry_delay': timedelta(minutes=5)
}

dag = DAG(
    'ml_training_pipeline',
    default_args=default_args,
    description='Daily model retraining',
    schedule_interval='0 2 * * *',  # 2 AM daily
    start_date=datetime(2024, 1, 1),
    catchup=False
)

# Task 1: Extract data
extract_data_task = PythonOperator(
    task_id='extract_data',
    python_callable=extract_data,
    dag=dag
)

# Task 2: Validate data
validate_data_task = PythonOperator(
    task_id='validate_data',
    python_callable=validate_data_quality,
    dag=dag
)

# Task 3: Feature engineering
feature_engineering_task = PythonOperator(
    task_id='feature_engineering',
    python_callable=engineer_features,
    dag=dag
)

# Task 4: Train model
train_model_task = PythonOperator(
    task_id='train_model',
    python_callable=train_and_log_model,
    dag=dag
)

# Task 5: Evaluate model
evaluate_model_task = PythonOperator(
    task_id='evaluate_model',
    python_callable=evaluate_and_compare,
    dag=dag
)

# Task 6: Deploy if better
deploy_model_task = PythonOperator(
    task_id='deploy_model',
    python_callable=conditional_deploy,
    dag=dag
)

# Define dependencies
extract_data_task >> validate_data_task >> feature_engineering_task
feature_engineering_task >> train_model_task >> evaluate_model_task
evaluate_model_task >> deploy_model_task
```

---

### Task Functions

```python
def extract_data(**context):
    """Extract data from sources"""
    from sqlalchemy import create_engine
    
    engine = create_engine('postgresql://...')
    query = """
        SELECT * FROM customers
        WHERE updated_at > NOW() - INTERVAL '1 day'
    """
    
    df = pd.read_sql(query, engine)
    
    # Save to XCom for next task
    context['task_instance'].xcom_push(key='data_path', value='/tmp/data.csv')
    df.to_csv('/tmp/data.csv', index=False)
    
    return len(df)

def validate_data_quality(**context):
    """Run data quality checks"""
    data_path = context['task_instance'].xcom_pull(
        task_ids='extract_data',
        key='data_path'
    )
    
    df = pd.read_csv(data_path)
    
    # Checks
    assert df.isnull().sum().sum() / len(df) < 0.05, "Too many nulls"
    assert len(df) > 1000, "Insufficient data"
    assert df['age'].between(0, 120).all(), "Invalid age values"
    
    print("Data quality checks passed")

def train_and_log_model(**context):
    """Train model and log to MLflow"""
    import mlflow
    
    # Load data
    data_path = context['task_instance'].xcom_pull(
        task_ids='extract_data',
        key='data_path'
    )
    df = pd.read_csv(data_path)
    
    X = df.drop('target', axis=1)
    y = df['target']
    
    # Train
    with mlflow.start_run():
        model = RandomForestClassifier(n_estimators=100)
        model.fit(X, y)
        
        # Log
        mlflow.sklearn.log_model(model, "model")
        mlflow.log_metric("train_size", len(X))
        
        run_id = mlflow.active_run().info.run_id
    
    # Pass to next task
    context['task_instance'].xcom_push(key='model_run_id', value=run_id)
    
    return run_id

def conditional_deploy(**context):
    """Deploy only if model is better"""
    run_id = context['task_instance'].xcom_pull(
        task_ids='train_model',
        key='model_run_id'
    )
    
    # Load metrics
    client = mlflow.tracking.MlflowClient()
    new_model_metrics = client.get_run(run_id).data.metrics
    
    # Compare with production
    prod_models = client.get_latest_versions("customer_churn", stages=["Production"])
    
    if prod_models:
        prod_metrics = client.get_run(prod_models[0].run_id).data.metrics
        
        if new_model_metrics['f1'] > prod_metrics['f1']:
            # Promote to production
            mlflow.register_model(
                f"runs:/{run_id}/model",
                "customer_churn"
            )
            print("New model deployed to production")
        else:
            print("New model did not improve, keeping current production model")
    else:
        # First model, deploy directly
        mlflow.register_model(
            f"runs:/{run_id}/model",
            "customer_churn"
        )
```

---

## Best Practices

### 1. Reproducibility Checklist

```python
# Always include:
✓ requirements.txt (dependency versions)
✓ Random seeds
✓ Data version/snapshot
✓ Code version (git commit)
✓ Environment specs

# Example:
import random
import numpy as np

def set_seed(seed=42):
    random.seed(seed)
    np.random.seed(seed)
    # For deep learning:
    # torch.manual_seed(seed)
    # tf.random.set_seed(seed)

set_seed(42)
```

---

### 2. Data Versioning

```python
# Use DVC (Data Version Control)
# Install: pip install dvc

# Track data
$ dvc add data/train.csv
$ git add data/train.csv.dvc .gitignore
$ git commit -m "Add training data"

# Push data to remote storage
$ dvc remote add -d storage s3://mybucket/dvc-store
$ dvc push
```

---

### 3. Config Management

```yaml
# config.yaml
model:
  algorithm: "RandomForest"
  n_estimators: 100
  max_depth: 10
  random_state: 42

data:
  train_path: "data/train.csv"
  test_path: "data/test.csv"
  target_column: "churn"

training:
  test_size: 0.2
  cv_folds: 5

deployment:
  model_name: "customer_churn"
  registry_uri: "sqlite:///mlflow.db"
```

```python
# Load config
import yaml

with open('config.yaml', 'r') as f:
    config = yaml.safe_load(f)

# Use in code
model = RandomForestClassifier(**config['model'])
```

---

### 4. Automated Testing

```python
# tests/test_model.py
import pytest
from src.model import train_model, predict

def test_model_training():
    """Test model can be trained"""
    X_train = [[1, 2], [3, 4], [5, 6]]
    y_train = [0, 1, 0]
    
    model = train_model(X_train, y_train)
    assert model is not None

def test_model_prediction():
    """Test model makes valid predictions"""
    X_test = [[2, 3]]
    predictions = predict(model, X_test)
    
    assert len(predictions) == 1
    assert predictions[0] in [0, 1]

def test_model_performance():
    """Test model meets minimum performance"""
    accuracy = evaluate_model(model, X_test, y_test)
    assert accuracy > 0.7, "Model accuracy below threshold"
```

---

## Summary

### Key Takeaways:

✅ **Experiment Tracking:** Use MLflow to log everything
✅ **Model Versioning:** Track models, compare versions, enable rollback
✅ **CI/CD for ML:** Automate training, validation, deployment
✅ **Orchestration:** Use Airflow/Prefect for complex pipelines
✅ **Best Practices:** Reproducibility, testing, config management

### For Data Engineers:

- Build robust data pipelines for ML
- Implement data validation checks
- Version datasets properly
- Integrate with ML training workflows
- Monitor data quality continuously

---

**Continue to Chapter 03 to learn Feature Engineering! 🚀**
