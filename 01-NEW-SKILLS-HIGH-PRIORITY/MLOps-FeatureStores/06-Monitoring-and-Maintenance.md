# Chapter 06: Monitoring and Maintenance

**Ensuring ML Models Stay Healthy in Production**

## Why Monitor ML Models?

**Problem:** Models degrade over time
- Data distribution changes (drift)
- Concept changes (what we're predicting changes)
- Data quality issues
- Performance degradation

**Without monitoring:** Model silently fails, business impact

## What to Monitor

### 1. Data Quality Metrics

```python
import pandas as pd

def monitor_data_quality(df):
    """Monitor input data quality"""
    
    metrics = {}
    
    # Missing values
    metrics['null_percentage'] = df.isnull().sum().sum() / (len(df) * len(df.columns))
    
    # Duplicate rows
    metrics['duplicate_percentage'] = df.duplicated().sum() / len(df)
    
    # Value ranges (for numeric columns)
    for col in df.select_dtypes(include=['number']).columns:
        metrics[f'{col}_min'] = df[col].min()
        metrics[f'{col}_max'] = df[col].max()
        metrics[f'{col}_mean'] = df[col].mean()
    
    # Alert if issues
    if metrics['null_percentage'] > 0.05:
        alert(f"High null percentage: {metrics['null_percentage']:.2%}")
    
    return metrics
```

### 2. Data Drift Detection

```python
from scipy.stats import ks_2samp

def detect_data_drift(reference_data, current_data, features):
    """Detect distribution changes using Kolmogorov-Smirnov test"""
    
    drift_detected = {}
    
    for feature in features:
        # KS test
        statistic, p_value = ks_2samp(
            reference_data[feature],
            current_data[feature]
        )
        
        # Drift if p < 0.05
        drift_detected[feature] = p_value < 0.05
        
        if drift_detected[feature]:
            alert(f"Drift detected in {feature}: p={p_value:.4f}")
    
    return drift_detected
```

### 3. Model Performance Metrics

```python
from sklearn.metrics import accuracy_score, precision_score, recall_score

def monitor_model_performance(y_true, y_pred):
    """Track model metrics"""
    
    metrics = {
        'accuracy': accuracy_score(y_true, y_pred),
        'precision': precision_score(y_true, y_pred),
        'recall': recall_score(y_true, y_pred),
        'sample_count': len(y_true)
    }
    
    # Log to monitoring system
    log_metrics(metrics, timestamp=datetime.now())
    
    # Alert if degradation
    if metrics['accuracy'] < 0.85:  # Threshold
        alert(f"Model accuracy dropped to {metrics['accuracy']:.2%}")
    
    return metrics
```

### 4. Prediction Distribution

```python
def monitor_prediction_distribution(predictions):
    """Track distribution of predictions"""
    
    import numpy as np
    
    metrics = {
        'mean': np.mean(predictions),
        'std': np.std(predictions),
        'min': np.min(predictions),
        'max': np.max(predictions),
        'percentile_25': np.percentile(predictions, 25),
        'percentile_50': np.percentile(predictions, 50),
        'percentile_75': np.percentile(predictions, 75)
    }
    
    return metrics
```

### 5. System Metrics

```python
import time
import psutil

def monitor_system_metrics():
    """Track infrastructure performance"""
    
    metrics = {
        'cpu_percent': psutil.cpu_percent(),
        'memory_percent': psutil.virtual_memory().percent,
        'disk_percent': psutil.disk_usage('/').percent
    }
    
    return metrics

# Prediction latency
start = time.time()
prediction = model.predict(features)
latency = time.time() - start

log_metric('prediction_latency_ms', latency * 1000)
```

## Monitoring Tools

### 1. Evidently (Data Drift)

```python
from evidently.dashboard import Dashboard
from evidently.tabs import DataDriftTab

# Create drift report
drift_dashboard = Dashboard(tabs=[DataDriftTab()])
drift_dashboard.calculate(reference_data, current_data)
drift_dashboard.save("drift_report.html")
```

### 2. Prometheus + Grafana

```python
from prometheus_client import Counter, Histogram, Gauge, start_http_server

# Define metrics
prediction_counter = Counter('predictions_total', 'Total predictions')
prediction_latency = Histogram('prediction_duration_seconds', 'Prediction latency')
model_accuracy = Gauge('model_accuracy', 'Current model accuracy')

# Track metrics
@prediction_latency.time()
def predict(features):
    prediction = model.predict(features)
    prediction_counter.inc()
    return prediction

# Start metrics server
start_http_server(8000)
```

### 3. MLflow Tracking

```python
import mlflow

def log_production_metrics(predictions, actuals):
    """Log metrics to MLflow"""
    
    with mlflow.start_run():
        accuracy = accuracy_score(actuals, predictions)
        mlflow.log_metric("production_accuracy", accuracy)
        mlflow.log_metric("prediction_count", len(predictions))
```

## Automated Retraining

### Trigger-Based Retraining

```python
def check_retraining_triggers():
    """Decide if model needs retraining"""
    
    triggers = {
        'performance_degradation': check_performance_drop(),
        'data_drift': check_data_drift(),
        'scheduled': check_schedule(),
        'new_data_available': check_data_volume()
    }
    
    if any(triggers.values()):
        trigger_retraining(reason=triggers)
        return True
    
    return False

def check_performance_drop():
    """Check if accuracy dropped below threshold"""
    current_accuracy = get_current_accuracy()
    baseline_accuracy = get_baseline_accuracy()
    
    return current_accuracy < baseline_accuracy - 0.05  # 5% drop

def trigger_retraining(reason):
    """Start retraining pipeline"""
    notify_team(f"Retraining triggered: {reason}")
    
    # Trigger Airflow DAG
    from airflow.api.client.local_client import Client
    client = Client(None, None)
    client.trigger_dag('model_retraining_pipeline')
```

### Scheduled Retraining

```python
# Airflow DAG for weekly retraining
from airflow import DAG
from datetime import datetime, timedelta

dag = DAG(
    'model_retraining',
    schedule_interval='0 2 * * 0',  # Every Sunday at 2 AM
    start_date=datetime(2024, 1, 1)
)

# Tasks: extract_data → train_model → evaluate → deploy_if_better
```

## Alerting

### Alert Rules

```python
class AlertManager:
    def __init__(self):
        self.thresholds = {
            'accuracy': 0.85,
            'latency_ms': 100,
            'null_percentage': 0.05,
            'drift_p_value': 0.05
        }
    
    def check_alerts(self, metrics):
        """Check if any alerts should fire"""
        
        alerts = []
        
        if metrics['accuracy'] < self.thresholds['accuracy']:
            alerts.append({
                'severity': 'HIGH',
                'message': f"Accuracy dropped to {metrics['accuracy']:.2%}",
                'action': 'Consider retraining'
            })
        
        if metrics['latency_ms'] > self.thresholds['latency_ms']:
            alerts.append({
                'severity': 'MEDIUM',
                'message': f"High latency: {metrics['latency_ms']:.0f}ms",
                'action': 'Check infrastructure'
            })
        
        # Send alerts
        for alert in alerts:
            self.send_alert(alert)
        
        return alerts
    
    def send_alert(self, alert):
        """Send alert via email/Slack/PagerDuty"""
        # Implementation depends on alerting system
        pass
```

## Model Versioning & Rollback

```python
def deploy_new_model_with_rollback(new_model_version):
    """Deploy with automatic rollback on failure"""
    
    # Save current production model
    old_model_version = get_production_model_version()
    
    try:
        # Deploy new model
        deploy_model(new_model_version, stage="Production")
        
        # Monitor for 1 hour
        time.sleep(3600)
        
        # Check performance
        metrics = get_production_metrics(hours=1)
        
        if metrics['error_rate'] > 0.01:  # 1% error rate threshold
            raise Exception(f"High error rate: {metrics['error_rate']:.2%}")
        
        if metrics['latency_p95'] > 200:  # 200ms latency threshold
            raise Exception(f"High latency: {metrics['latency_p95']:.0f}ms")
        
        # Success!
        notify_team(f"Model {new_model_version} deployed successfully")
    
    except Exception as e:
        # Rollback
        notify_team(f"Deployment failed: {e}. Rolling back...")
        deploy_model(old_model_version, stage="Production")
        raise
```

## Incident Response

### Runbook Template

```markdown
# Model Performance Degradation Runbook

## Symptoms
- Accuracy < 85%
- Alert: "Model accuracy dropped"

## Diagnosis
1. Check data drift dashboard
2. Review recent data quality metrics
3. Compare current vs historical predictions
4. Check for schema changes in input data

## Resolution
1. If data drift: Trigger retraining
2. If data quality issue: Fix upstream pipeline
3. If schema change: Update feature engineering
4. If infrastructure: Scale resources

## Prevention
- Monitor data drift continuously
- Validate input data schemas
- Set up automated retraining triggers
```

## Summary

✅ **Monitor:** Data quality, drift, performance, predictions, system metrics
✅ **Tools:** Evidently, Prometheus, MLflow, custom dashboards
✅ **Automation:** Trigger-based and scheduled retraining
✅ **Alerting:** Define thresholds, send notifications
✅ **Rollback:** Quick recovery from bad deployments
✅ **Documentation:** Runbooks for incident response

### Monitoring Checklist

- [ ] Data quality checks in place
- [ ] Drift detection configured
- [ ] Performance metrics tracked
- [ ] Alerting rules defined
- [ ] Retraining triggers set
- [ ] Rollback procedure tested
- [ ] Runbooks documented
- [ ] On-call rotation established

**You've completed the MLOps-FeatureStores learning path! 🎉**
