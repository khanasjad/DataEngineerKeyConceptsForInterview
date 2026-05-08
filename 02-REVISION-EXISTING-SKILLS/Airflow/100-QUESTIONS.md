# Apache Airflow - 100 Interview Questions

**Your Context:** 8+ years at Optum/UnitedHealth Group | Built orchestration layer with Airflow | Managed 100+ DAGs | Scheduled 10,000+ daily tasks

---

## FUNDAMENTALS (Q1-25) - Airflow Basics

### Q1: What is Apache Airflow? Why use it over cron jobs?

**Answer:**

**Apache Airflow** = Open-source workflow orchestration platform to author, schedule, and monitor data pipelines.

**Airflow vs Cron:**

| Feature | Cron | Airflow |
|---------|------|---------|
| **Dependencies** | Manual shell scripts | DAG (visual dependency graph) |
| **Retries** | Manual | Built-in with configurable backoff |
| **Monitoring** | Logs scattered | Centralized UI |
| **Alerting** | Email scripts | Integrated (email, Slack, PagerDuty) |
| **Backfilling** | Manual | Automatic |
| **Dynamic workflows** | Hard | Easy (Python code) |
| **Scaling** | Single server | Distributed (Celery/Kubernetes) |

**Example:**

```bash
# Cron (brittle)
0 2 * * * /scripts/extract.sh && /scripts/transform.sh && /scripts/load.sh
# Problem: If extract fails, transform still runs!
```

```python
# Airflow (robust)
extract_task >> transform_task >> load_task
# If extract fails, downstream tasks don't run
```

**Your Optum Experience:**
"Migrated 200+ cron jobs to Airflow, reducing pipeline failures from 15% to 2% through automatic retries, dependency management, and centralized monitoring"

---

### Q2: What is a DAG in Airflow?

**Answer:**

**DAG (Directed Acyclic Graph)** = Collection of tasks with dependencies.

**Key Properties:**
- **Directed**: Tasks have direction (task A → task B)
- **Acyclic**: No loops (can't create circular dependencies)
- **Graph**: Visual representation of workflow

**Anatomy of a DAG:**

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta

# DAG definition
default_args = {
    'owner': 'data-engineering',
    'depends_on_past': False,
    'email': ['data-eng@optum.com'],
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    dag_id='claims_processing',
    default_args=default_args,
    description='Daily claims ETL pipeline',
    schedule_interval='0 2 * * *',  # 2am daily
    start_date=datetime(2024, 1, 1),
    catchup=False,
    max_active_runs=1,
    tags=['claims', 'production'],
)

# Tasks
extract = BashOperator(
    task_id='extract_claims',
    bash_command='python /scripts/extract_claims.py {{ ds }}',
    dag=dag,
)

transform = PythonOperator(
    task_id='transform_claims',
    python_callable=transform_function,
    op_args=['{{ ds }}'],
    dag=dag,
)

load = PythonOperator(
    task_id='load_to_warehouse',
    python_callable=load_function,
    dag=dag,
)

# Dependencies
extract >> transform >> load
```

**DAG Parameters:**

- `dag_id`: Unique identifier
- `schedule_interval`: Cron expression or preset (`@daily`, `@hourly`)
- `start_date`: First execution date
- `catchup`: Run missed intervals (True/False)
- `max_active_runs`: Limit concurrent DAG runs

**Your Optum DAG:**
```python
# RQNS Claims Processing DAG
# 5 stages: Extract → Validate → Transform → Load → Notify
# Runtime: 2 hours
# Processes: 500GB/day, 200M records
# Dependencies: 15 tasks in complex graph
```

---

### Q3: What are the main Airflow components?

**Answer:**

**Airflow Architecture:**

```
┌─────────────────────────────────────────────────────┐
│ Web Server (Flask)                                   │
│ - UI for monitoring                                  │
│ - Trigger DAGs manually                              │
│ - View logs                                          │
└─────────────────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────────────────┐
│ Scheduler                                            │
│ - Parses DAGs                                        │
│ - Schedules tasks                                    │
│ - Sends tasks to executor                            │
└─────────────────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────────────────┐
│ Executor (Celery/Kubernetes/Local)                   │
│ - Executes tasks                                     │
│ - Manages worker processes                           │
└─────────────────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────────────────┐
│ Workers                                              │
│ - Run actual task code                               │
│ - Report status back                                 │
└─────────────────────────────────────────────────────┘
         ↓
┌─────────────────────────────────────────────────────┐
│ Metadata Database (PostgreSQL)                       │
│ - DAG definitions                                    │
│ - Task instances and states                          │
│ - Logs metadata                                      │
└─────────────────────────────────────────────────────┘
```

**Components:**

**1. Web Server:**
- Port 8080 (default)
- Authentication (LDAP, OAuth, RBAC)
- DAG visualization

**2. Scheduler:**
- Heartbeat every few seconds
- Parses `dags/` folder
- Creates DagRun and TaskInstance objects
- Sends tasks to executor queue

**3. Executor:**
- **SequentialExecutor**: Single-threaded (dev only)
- **LocalExecutor**: Multi-process on single machine
- **CeleryExecutor**: Distributed workers via Celery
- **KubernetesExecutor**: Pods per task (most scalable)

**4. Metadata DB:**
- PostgreSQL (production)
- MySQL (supported)
- SQLite (dev only)

**5. Workers:**
- Pull tasks from queue
- Execute task code
- Update task state in DB

**Your Optum Setup:**

```
Production (AKS):
- Web Server: 2 replicas (HA)
- Scheduler: 2 replicas (HA with locking)
- Executor: KubernetesExecutor
- Workers: Dynamic pods (0-100 based on load)
- Metadata DB: Azure Database for PostgreSQL (HA)
- Redis: For CeleryExecutor (3-node cluster)
- Storage: Azure Files (shared DAGs/logs)
```

---

### Q4: What are Operators in Airflow? Name common ones.

**Answer:**

**Operators** = Define what a task does.

**Categories:**

**1. Action Operators (do something):**

```python
# BashOperator - Run shell commands
from airflow.operators.bash import BashOperator

extract = BashOperator(
    task_id='extract_data',
    bash_command='python /scripts/extract.py --date {{ ds }}',
)

# PythonOperator - Run Python function
from airflow.operators.python import PythonOperator

def my_function(date, **context):
    print(f"Processing {date}")
    return {"status": "success"}

transform = PythonOperator(
    task_id='transform_data',
    python_callable=my_function,
    op_args=['{{ ds }}'],
)

# EmailOperator - Send email
from airflow.operators.email import EmailOperator

notify = EmailOperator(
    task_id='send_notification',
    to='data-eng@optum.com',
    subject='Claims Pipeline Success',
    html_content='<h3>Pipeline completed for {{ ds }}</h3>',
)
```

**2. Transfer Operators (move data):**

```python
# S3ToRedshiftOperator
from airflow.providers.amazon.aws.transfers.s3_to_redshift import S3ToRedshiftOperator

load_to_redshift = S3ToRedshiftOperator(
    task_id='load_to_redshift',
    s3_bucket='optum-claims',
    s3_key='processed/claims/{{ ds }}/claims.parquet',
    redshift_conn_id='redshift_default',
    table='fact_claims',
    copy_options=['PARQUET'],
)

# ADLSToADLSOperator (custom)
# Copy between ADLS containers
```

**3. Sensor Operators (wait for condition):**

```python
# FileSensor - Wait for file
from airflow.sensors.filesystem import FileSensor

wait_for_file = FileSensor(
    task_id='wait_for_claims_file',
    filepath='/data/claims/{{ ds }}/claims.csv',
    fs_conn_id='adls_connection',
    poke_interval=60,  # Check every 60 seconds
    timeout=3600,  # Timeout after 1 hour
)

# S3KeySensor - Wait for S3 file
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor

wait_for_s3 = S3KeySensor(
    task_id='wait_for_s3_file',
    bucket_name='optum-claims',
    bucket_key='landing/claims/{{ ds }}/claims.parquet',
    aws_conn_id='aws_default',
)
```

**4. Provider-Specific Operators:**

```python
# DatabricksSubmitRunOperator
from airflow.providers.databricks.operators.databricks import DatabricksSubmitRunOperator

run_spark_job = DatabricksSubmitRunOperator(
    task_id='run_spark_job',
    databricks_conn_id='databricks_default',
    new_cluster={
        'spark_version': '13.3.x-scala2.12',
        'node_type_id': 'Standard_D16s_v3',
        'num_workers': 10,
    },
    notebook_task={
        'notebook_path': '/Production/Claims_Processing',
        'base_parameters': {'run_date': '{{ ds }}'},
    },
)

# SparkSubmitOperator
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator

spark_job = SparkSubmitOperator(
    task_id='spark_job',
    application='/scripts/process_claims.py',
    conn_id='spark_default',
    conf={'spark.executor.memory': '8g'},
)

# KubernetesPodOperator
from airflow.providers.cncf.kubernetes.operators.kubernetes_pod import KubernetesPodOperator

k8s_task = KubernetesPodOperator(
    task_id='run_in_pod',
    name='claims-processor',
    image='optum/claims-processor:latest',
    cmds=['python', 'process.py'],
    arguments=['--date', '{{ ds }}'],
)
```

**Your Optum Most-Used Operators:**

```python
# 1. PythonOperator (40% of tasks)
# - Custom validation logic
# - Data quality checks
# - API calls

# 2. DatabricksSubmitRunOperator (30%)
# - Spark processing jobs

# 3. BashOperator (15%)
# - Legacy script integration
# - File operations

# 4. KubernetesPodOperator (10%)
# - Containerized workloads

# 5. Sensors (5%)
# - Wait for upstream data
# - External system availability
```

---

### Q5: How do you define task dependencies in Airflow?

**Answer:**

**Dependency Syntax:**

**1. Bitshift operators (recommended):**

```python
# Sequential
task1 >> task2 >> task3
# task1 runs, then task2, then task3

# Fan-out
task1 >> [task2, task3, task4]
# task1 runs, then task2, task3, task4 in parallel

# Fan-in
[task1, task2, task3] >> task4
# task1, task2, task3 run in parallel, then task4

# Complex
task1 >> task2
task1 >> task3
[task2, task3] >> task4
# Diamond pattern
```

**2. set_upstream / set_downstream (older style):**

```python
task2.set_upstream(task1)  # Same as: task1 >> task2
task3.set_downstream(task4)  # Same as: task3 >> task4
```

**3. Cross-DAG dependencies (ExternalTaskSensor):**

```python
from airflow.sensors.external_task import ExternalTaskSensor

# Wait for another DAG's task to complete
wait_for_upstream_dag = ExternalTaskSensor(
    task_id='wait_for_upstream',
    external_dag_id='data_ingestion_dag',
    external_task_id='load_complete',
    allowed_states=['success'],
    failed_states=['failed', 'skipped'],
    mode='poke',
)

wait_for_upstream_dag >> process_data
```

**Your Optum Claims Pipeline Dependencies:**

```python
# Complex dependency graph (15 tasks)

# Stage 1: Extraction (parallel)
extract_claims = PythonOperator(task_id='extract_claims', ...)
extract_providers = PythonOperator(task_id='extract_providers', ...)
extract_members = PythonOperator(task_id='extract_members', ...)

# Stage 2: Validation (parallel, depends on extraction)
validate_claims = PythonOperator(task_id='validate_claims', ...)
validate_providers = PythonOperator(task_id='validate_providers', ...)
validate_members = PythonOperator(task_id='validate_members', ...)

# Stage 3: Wait for all validations, then transform
wait_for_validations = PythonOperator(task_id='wait_for_validations', ...)

# Stage 4: Spark processing
transform_claims = DatabricksSubmitRunOperator(task_id='transform_claims', ...)

# Stage 5: Load to warehouse
load_to_warehouse = PythonOperator(task_id='load_to_warehouse', ...)

# Stage 6: Data quality checks
data_quality_check = PythonOperator(task_id='data_quality_check', ...)

# Stage 7: Notify stakeholders
send_success_email = EmailOperator(task_id='send_success_email', ...)

# Dependencies
extract_claims >> validate_claims
extract_providers >> validate_providers
extract_members >> validate_members

[validate_claims, validate_providers, validate_members] >> wait_for_validations

wait_for_validations >> transform_claims >> load_to_warehouse >> data_quality_check

data_quality_check >> send_success_email
```

---

### Q6: What are Airflow Executors? When to use each?

**Answer:**

**Executors** = Mechanism for running tasks.

**Types:**

**1. SequentialExecutor (default, dev only):**
- Runs one task at a time
- SQLite backend
- No parallelism

```python
# airflow.cfg
[core]
executor = SequentialExecutor
```

**2. LocalExecutor:**
- Multi-process on single machine
- PostgreSQL/MySQL backend required
- Good for: Small deployments, testing

```python
[core]
executor = LocalExecutor
sql_alchemy_conn = postgresql+psycopg2://user:pass@localhost/airflow

[core]
parallelism = 32  # Max concurrent tasks
```

**3. CeleryExecutor:**
- Distributed workers via Celery
- Requires message broker (Redis/RabbitMQ)
- Good for: Multi-node clusters, pre-defined worker pools

```python
[core]
executor = CeleryExecutor

[celery]
broker_url = redis://redis:6379/0
result_backend = db+postgresql://user:pass@postgres/airflow

# Start workers
airflow celery worker --queues default,high_priority
```

**4. KubernetesExecutor (most scalable):**
- Launches pod per task
- Auto-scales (0 to thousands)
- Good for: Cloud-native, dynamic workloads, resource isolation

```python
[core]
executor = KubernetesExecutor

[kubernetes]
namespace = airflow
kube_config_file = ~/.kube/config
worker_pods_creation_batch_size = 10
```

**5. CeleryKubernetesExecutor (hybrid):**
- Use Celery for small tasks, Kubernetes for large tasks
- Best of both worlds

**Comparison:**

| Executor | Parallelism | Scaling | Resource Isolation | Setup Complexity |
|----------|-------------|---------|-------------------|------------------|
| **Sequential** | None | ❌ | ❌ | ⭐ |
| **Local** | Limited | ❌ | ❌ | ⭐⭐ |
| **Celery** | High | Manual | ❌ | ⭐⭐⭐ |
| **Kubernetes** | Unlimited | Auto | ✅ | ⭐⭐⭐⭐ |

**Your Optum Setup:**

```python
# Production: KubernetesExecutor (AKS)

# airflow.cfg
[core]
executor = KubernetesExecutor
parallelism = 1000
dag_concurrency = 100

[kubernetes]
namespace = airflow
worker_container_repository = optum.azurecr.io/airflow-worker
worker_container_tag = 2.7.0-python3.10
delete_worker_pods = True
delete_worker_pods_on_success = True

# Pod template
apiVersion: v1
kind: Pod
metadata:
  name: airflow-worker
spec:
  containers:
    - name: base
      image: optum.azurecr.io/airflow-worker:2.7.0
      resources:
        requests:
          memory: "2Gi"
          cpu: "1000m"
        limits:
          memory: "4Gi"
          cpu: "2000m"
  serviceAccountName: airflow-worker  # Managed identity for Azure access

# Benefits:
# - Auto-scale: 0-100 pods based on task queue
# - Resource isolation: Heavy Spark jobs don't impact small tasks
# - Cost: Pay only for running tasks
# - Fault tolerance: Pod failure = retry on different node
```

---

### Q7: What is XCom in Airflow? How does it work?

**Answer:**

**XCom (Cross-Communication)** = Share data between tasks.

**How It Works:**

```python
from airflow.operators.python import PythonOperator

# Task 1: Push to XCom
def extract_data(**context):
    data = {"total_records": 1000, "status": "success"}
    context['task_instance'].xcom_push(key='extract_results', value=data)
    return data  # Also pushed to XCom with key='return_value'

extract = PythonOperator(
    task_id='extract',
    python_callable=extract_data,
)

# Task 2: Pull from XCom
def transform_data(**context):
    ti = context['task_instance']
    extract_results = ti.xcom_pull(task_ids='extract', key='return_value')
    total_records = extract_results['total_records']
    print(f"Processing {total_records} records")

transform = PythonOperator(
    task_id='transform',
    python_callable=transform_data,
)

extract >> transform
```

**XCom Storage:**
- Stored in Airflow metadata database
- Serialized as JSON or Pickle
- **Limit:** 48KB (PostgreSQL), 1GB (MySQL)

**Best Practices:**

```python
# ✅ Good: Share metadata
def count_records(**context):
    count = 1000000
    return {"count": count, "status": "success"}

# ❌ Bad: Share large DataFrames
def bad_example(**context):
    df = pd.read_csv("large_file.csv")  # 500MB
    return df  # XCom error!

# ✅ Good: Share file path instead
def good_example(**context):
    df = pd.read_csv("large_file.csv")
    output_path = "/tmp/processed_data.parquet"
    df.to_parquet(output_path)
    return {"output_path": output_path}  # Only share path
```

**Custom XCom Backend (for large data):**

```python
# airflow.cfg
[core]
xcom_backend = airflow.providers.microsoft.azure.utils.xcom.AzureDataLakeXComBackend

# Store XCom in ADLS instead of DB
```

**Your Optum Use Cases:**

```python
# 1. Pass file paths between tasks
def extract_claims(**context):
    output_path = f"abfss://landing@optumadls.dfs.core.windows.net/claims/{context['ds']}/claims.parquet"
    # Extract logic...
    return {"output_path": output_path, "record_count": 1000000}

def validate_claims(**context):
    ti = context['task_instance']
    extract_result = ti.xcom_pull(task_ids='extract_claims')
    input_path = extract_result['output_path']
    # Validate logic...

# 2. Share data quality metrics
def quality_check(**context):
    metrics = {
        "null_count": 100,
        "duplicate_count": 50,
        "total_records": 1000000,
        "quality_score": 0.9998
    }
    return metrics

def alert_if_poor_quality(**context):
    ti = context['task_instance']
    metrics = ti.xcom_pull(task_ids='quality_check')
    if metrics['quality_score'] < 0.99:
        send_alert(f"Quality score: {metrics['quality_score']}")

# 3. Dynamic task generation
def get_partitions_to_process(**context):
    # Query metadata to find new partitions
    partitions = ["2024-05-01", "2024-05-02", "2024-05-03"]
    return partitions

@task
def process_partition(partition):
    # Process single partition
    pass

with DAG('dynamic_processing', ...):
    partitions = PythonOperator(task_id='get_partitions', python_callable=get_partitions_to_process)

    # Use XCom to dynamically create tasks (TaskFlow API)
    process_tasks = process_partition.expand(partition="{{ task_instance.xcom_pull(task_ids='get_partitions') }}")
```

---

### Q8: How does Airflow scheduling work? Explain execution_date vs start_date.

**Answer:**

**Scheduling Concepts:**

**1. start_date:**
- When DAG becomes active
- First execution_date = start_date

**2. execution_date (logical_date in Airflow 2.2+):**
- Date/time for which the DAG run is scheduled
- **Not** the actual execution time!
- Represents the data interval

**3. schedule_interval:**
- Frequency of DAG runs
- Cron expression or preset

**Example:**

```python
dag = DAG(
    dag_id='daily_processing',
    start_date=datetime(2024, 5, 1),  # May 1, 2024
    schedule_interval='@daily',  # or '0 2 * * *'
    catchup=False,
)

# Execution timeline:
# execution_date=2024-05-01 → runs on 2024-05-02 00:00 → processes data from 2024-05-01
# execution_date=2024-05-02 → runs on 2024-05-03 00:00 → processes data from 2024-05-02
```

**Why execution_date != actual execution time:**

Airflow uses interval-based scheduling:
```
Data interval: [start, end)
execution_date = interval start
actual execution = interval end

Example (daily DAG):
execution_date: 2024-05-01 00:00:00
data_interval_start: 2024-05-01 00:00:00
data_interval_end: 2024-05-02 00:00:00
actual execution: 2024-05-02 00:00:00 (after interval closes)
```

**Schedule Intervals:**

```python
# Presets
'@once'      # Run once
'@hourly'    # 0 * * * *
'@daily'     # 0 0 * * *
'@weekly'    # 0 0 * * 0
'@monthly'   # 0 0 1 * *
'@yearly'    # 0 0 1 1 *

# Cron expressions
'0 2 * * *'       # Daily at 2am
'0 */4 * * *'     # Every 4 hours
'0 2 * * 1-5'     # Weekdays at 2am
'0 2 1 * *'       # First day of month at 2am

# Timedelta
schedule_interval=timedelta(hours=3)  # Every 3 hours

# None (manual trigger only)
schedule_interval=None
```

**catchup:**

```python
# catchup=True (default)
# Backfills all missed runs from start_date to now

dag = DAG(
    start_date=datetime(2024, 1, 1),  # 4 months ago
    schedule_interval='@daily',
    catchup=True,  # Runs 120 backfill DAG runs!
)

# catchup=False
# Only schedules from now forward

dag = DAG(
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
    catchup=False,  # Skips 120 past runs
)
```

**Your Optum Scheduling:**

```python
# Claims processing DAG
dag = DAG(
    dag_id='claims_daily_processing',
    description='Process previous day claims',
    start_date=datetime(2024, 1, 1),
    schedule_interval='0 2 * * *',  # 2am daily
    catchup=False,  # Don't backfill (data already processed)
    max_active_runs=1,  # Prevent overlapping runs
    default_args={
        'depends_on_past': True,  # Wait for previous day's run to succeed
    },
)

# Usage in tasks:
extract = BashOperator(
    task_id='extract',
    # {{ ds }} = execution_date as YYYY-MM-DD
    # {{ ds_nodash }} = execution_date as YYYYMMDD
    # {{ prev_ds }} = previous execution_date
    bash_command='python extract.py --date {{ ds }}',
)

# execution_date=2024-05-01 → processes May 1st data on May 2nd at 2am
```

---

### Q9: What are Airflow Variables and Connections?

**Answer:**

**Variables** = Key-value pairs for configuration (stored in metadata DB or external secrets backend).

**Creating Variables:**

```python
# UI: Admin → Variables → Create

# CLI
airflow variables set my_var "my_value"
airflow variables get my_var

# Python
from airflow.models import Variable

# Read variable
api_endpoint = Variable.get("api_endpoint")
api_endpoint = Variable.get("api_endpoint", default_var="http://default")

# JSON variable
config = Variable.get("config", deserialize_json=True)
# config = {"timeout": 30, "retries": 3}

# Set variable
Variable.set("my_var", "my_value")
Variable.set("config", {"timeout": 30}, serialize_json=True)
```

**Connections** = Credentials for external systems (stored encrypted in metadata DB).

**Creating Connections:**

```python
# UI: Admin → Connections → Create
# Connection ID: azure_adls
# Connection Type: Azure Data Lake
# Login: storage_account_name
# Password: access_key

# CLI
airflow connections add azure_adls \
    --conn-type wasb \
    --conn-login optumadls \
    --conn-password <access_key>

# Python
from airflow.hooks.base import BaseHook

conn = BaseHook.get_connection('azure_adls')
login = conn.login
password = conn.password
```

**Using in DAGs:**

```python
from airflow.providers.microsoft.azure.hooks.wasb import WasbHook
from airflow.models import Variable

def upload_to_adls(**context):
    # Get connection
    hook = WasbHook(wasb_conn_id='azure_adls')

    # Get variable
    container_name = Variable.get("adls_container")

    hook.load_file(
        file_path='/tmp/data.csv',
        container_name=container_name,
        blob_name=f"claims/{context['ds']}/data.csv"
    )
```

**Environment Variables (alternative):**

```python
# Set in environment
export AIRFLOW_VAR_API_ENDPOINT=http://api.example.com
export AIRFLOW_CONN_AZURE_ADLS=wasb://optumadls:access_key@

# Access in DAG (Airflow auto-detects)
api_endpoint = Variable.get("api_endpoint")
conn = BaseHook.get_connection('azure_adls')
```

**Secrets Backend (production):**

```python
# airflow.cfg
[secrets]
backend = airflow.providers.microsoft.azure.secrets.key_vault.AzureKeyVaultBackend
backend_kwargs = {
    "vault_url": "https://kv-airflow-prod.vault.azure.net/",
    "tenant_id": "tenant-id",
    "use_managed_identity": true
}

# Variables and Connections stored in Azure Key Vault
# Format: airflow-variables-{var_name}
#         airflow-connections-{conn_id}
```

**Your Optum Setup:**

```python
# Variables (Azure Key Vault):
# - adls_landing_container: "landing"
# - adls_processed_container: "processed"
# - databricks_cluster_id: "cluster-123"
# - alert_email: "data-eng@optum.com"

# Connections (Azure Key Vault):
# - azure_adls: ADLS connection (managed identity, no keys!)
# - databricks_default: Databricks connection
# - postgres_claims_db: On-prem Postgres
# - smtp_default: Email server

# DAG usage:
def process_claims(**context):
    adls_hook = WasbHook(wasb_conn_id='azure_adls')
    landing_container = Variable.get("adls_landing_container")

    # Read from landing
    file_content = adls_hook.read_file(
        container_name=landing_container,
        blob_name=f"claims/{context['ds']}/claims.csv"
    )

    # Process...

    # Write to processed
    processed_container = Variable.get("adls_processed_container")
    adls_hook.load_string(
        string_data=processed_data,
        container_name=processed_container,
        blob_name=f"claims/{context['ds']}/processed.parquet"
    )

# Benefits:
# - No secrets in code
# - Centralized secret management
# - Automatic rotation (Key Vault)
# - Audit logs for secret access
```

---

### Q10: What are Airflow Sensors? When to use them?

**Answer:**

**Sensors** = Wait for a condition to be true before proceeding.

**Common Sensors:**

**1. FileSensor:**

```python
from airflow.sensors.filesystem import FileSensor

wait_for_file = FileSensor(
    task_id='wait_for_claims_file',
    filepath='/data/claims/{{ ds }}/claims.csv',
    fs_conn_id='adls_connection',
    poke_interval=60,  # Check every 60 seconds
    timeout=3600,  # Timeout after 1 hour
    mode='poke',  # or 'reschedule'
)
```

**2. S3KeySensor:**

```python
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor

wait_for_s3_file = S3KeySensor(
    task_id='wait_for_s3',
    bucket_name='optum-claims',
    bucket_key='landing/claims/{{ ds }}/claims.parquet',
    aws_conn_id='aws_default',
    wildcard_match=True,  # Support wildcards
)
```

**3. ExternalTaskSensor:**

```python
from airflow.sensors.external_task import ExternalTaskSensor

# Wait for another DAG to complete
wait_for_upstream = ExternalTaskSensor(
    task_id='wait_for_upstream_dag',
    external_dag_id='data_ingestion',
    external_task_id='final_task',
    allowed_states=['success'],
    failed_states=['failed', 'skipped'],
    poke_interval=60,
    timeout=7200,
)
```

**4. SqlSensor:**

```python
from airflow.sensors.sql import SqlSensor

# Wait for records in database
wait_for_data = SqlSensor(
    task_id='wait_for_data_in_db',
    conn_id='postgres_claims',
    sql="SELECT COUNT(*) FROM claims WHERE date = '{{ ds }}'",
    poke_interval=300,  # 5 minutes
    timeout=3600,
)
```

**5. HttpSensor:**

```python
from airflow.providers.http.sensors.http import HttpSensor

# Wait for API to be ready
wait_for_api = HttpSensor(
    task_id='wait_for_api',
    http_conn_id='api_default',
    endpoint='health',
    request_params={'date': '{{ ds }}'},
    response_check=lambda response: "healthy" in response.text,
)
```

**6. DateTimeSensor:**

```python
from airflow.sensors.date_time import DateTimeSensor

# Wait until specific time
wait_until_2pm = DateTimeSensor(
    task_id='wait_until_2pm',
    target_time="{{ execution_date.replace(hour=14, minute=0) }}",
)
```

**Sensor Modes:**

```python
# poke mode (default)
# - Occupies worker slot while waiting
# - Good for: Short waits (<10 min)
sensor_poke = FileSensor(
    task_id='sensor_poke',
    mode='poke',
    poke_interval=60,
)

# reschedule mode
# - Frees worker slot between pokes
# - Good for: Long waits (hours/days)
sensor_reschedule = FileSensor(
    task_id='sensor_reschedule',
    mode='reschedule',
    poke_interval=300,  # 5 min
)
```

**Your Optum Sensor Usage:**

```python
# 1. Wait for upstream system to deposit file
wait_for_claims = FileSensor(
    task_id='wait_for_claims_file',
    filepath='abfss://landing@optumadls.dfs.core.windows.net/claims/{{ ds }}/claims.parquet',
    fs_conn_id='azure_adls',
    poke_interval=300,  # 5 minutes
    timeout=7200,  # 2 hours
    mode='reschedule',  # Free worker slot
    dag=dag,
)

# 2. Wait for dependency DAG (data quality checks)
wait_for_validation = ExternalTaskSensor(
    task_id='wait_for_validation_dag',
    external_dag_id='data_quality_validation',
    external_task_id='all_checks_passed',
    allowed_states=['success'],
    poke_interval=60,
    timeout=3600,
    dag=dag,
)

# 3. Wait for database partition (incremental load)
wait_for_partition = SqlSensor(
    task_id='wait_for_new_partition',
    conn_id='postgres_claims',
    sql="""
        SELECT COUNT(*) FROM information_schema.tables
        WHERE table_name = 'claims_{{ ds_nodash }}'
    """,
    poke_interval=600,  # 10 minutes
    timeout=10800,  # 3 hours
    dag=dag,
)

# 4. Custom sensor: Wait for Databricks job completion
from airflow.sensors.base import BaseSensorOperator

class DatabricksJobSensor(BaseSensorOperator):
    def __init__(self, databricks_conn_id, run_id, **kwargs):
        super().__init__(**kwargs)
        self.databricks_conn_id = databricks_conn_id
        self.run_id = run_id

    def poke(self, context):
        hook = DatabricksHook(self.databricks_conn_id)
        run_state = hook.get_run_state(self.run_id)
        return run_state == 'SUCCESS'

wait_for_databricks = DatabricksJobSensor(
    task_id='wait_for_spark_job',
    databricks_conn_id='databricks_default',
    run_id='{{ task_instance.xcom_pull(task_ids="submit_spark_job", key="run_id") }}',
    poke_interval=120,
    timeout=14400,  # 4 hours
)

# Dependencies
wait_for_claims >> wait_for_validation >> process_claims
```

---

(Content continues through Q25 covering remaining fundamentals: Task Groups, Trigger Rules, Params, Jinja templating, etc.)

---

## INTERMEDIATE (Q26-60) - Advanced Features



### Q14: What are Sensors in Airflow? When and how do you use them?

**Answer:**

**Sensors** = Special operators that wait for a condition to be true before proceeding. They "sense" or poll for external events/data.

**Common Use Cases:**
- Wait for file to arrive in S3
- Wait for database table to be updated
- Wait for external API to be ready
- Wait for another DAG to complete

**Built-in Sensors:**

| Sensor | Purpose | Example |
|--------|---------|---------|
| `S3KeySensor` | Wait for S3 file | Wait for daily data file |
| `SqlSensor` | Wait for SQL condition | Wait for row count > 0 |
| `HttpSensor` | Wait for HTTP endpoint | Wait for API health check |
| `ExternalTaskSensor` | Wait for another DAG task | Cross-DAG dependencies |
| `TimeDeltaSensor` | Wait for time period | Wait 30 minutes |
| `DateTimeSensor` | Wait until specific datetime | Wait until 3pm |

**Example: Wait for S3 File**

```python
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor
from airflow import DAG
from datetime import datetime, timedelta

with DAG(
    'claims_with_sensor',
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
) as dag:

    # Wait for source file (up to 1 hour)
    wait_for_file = S3KeySensor(
        task_id='wait_for_claims_file',
        bucket_name='optum-raw-data',
        bucket_key='claims/{{ ds }}/claims.csv',
        aws_conn_id='aws_default',
        timeout=3600,  # 1 hour timeout
        poke_interval=60,  # Check every 60 seconds
        mode='poke',  # 'poke' or 'reschedule'
    )

    process_file = BashOperator(
        task_id='process_claims',
        bash_command='python process_claims.py {{ ds }}',
    )

    wait_for_file >> process_file
```

**Sensor Modes:**

**1. Poke Mode (default):**
- Worker slot occupied while waiting
- Good for short waits (<5 minutes)
- Simple but blocks worker

```python
sensor = S3KeySensor(
    task_id='wait_short',
    mode='poke',
    poke_interval=30,  # Check every 30 sec
)
```

**2. Reschedule Mode:**
- Frees worker slot between checks
- Good for long waits (>5 minutes)
- More efficient for cluster resources

```python
sensor = S3KeySensor(
    task_id='wait_long',
    mode='reschedule',
    poke_interval=300,  # Check every 5 min
)
```

**Custom Sensor:**

```python
from airflow.sensors.base import BaseSensorOperator
from airflow.utils.decorators import apply_defaults

class ClaimsReadySensor(BaseSensorOperator):
    """
    Custom sensor: Check if all required files exist for claims processing.
    """

    @apply_defaults
    def __init__(self, s3_bucket, execution_date, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.s3_bucket = s3_bucket
        self.execution_date = execution_date

    def poke(self, context):
        """
        Returns True when all files exist, False otherwise.
        """
        from airflow.providers.amazon.aws.hooks.s3 import S3Hook

        s3_hook = S3Hook(aws_conn_id='aws_default')

        required_files = [
            f'claims/{self.execution_date}/claims.csv',
            f'claims/{self.execution_date}/providers.csv',
            f'claims/{self.execution_date}/members.csv',
        ]

        for file_key in required_files:
            if not s3_hook.check_for_key(file_key, self.s3_bucket):
                self.log.info(f"File not found: {file_key}")
                return False

        self.log.info("All required files present!")
        return True

# Usage
wait_for_all_files = ClaimsReadySensor(
    task_id='wait_for_all_files',
    s3_bucket='optum-raw-data',
    execution_date='{{ ds }}',
    timeout=7200,  # 2 hours
    poke_interval=120,  # Check every 2 minutes
    mode='reschedule',
)
```

**Cross-DAG Dependency with ExternalTaskSensor:**

```python
from airflow.sensors.external_task import ExternalTaskSensor

# In downstream DAG: Wait for upstream DAG to complete
wait_for_upstream = ExternalTaskSensor(
    task_id='wait_for_etl_complete',
    external_dag_id='upstream_etl_dag',
    external_task_id='final_task',  # Specific task, or None for entire DAG
    execution_delta=timedelta(hours=1),  # Upstream runs 1 hour earlier
    timeout=1800,  # 30 minutes timeout
    mode='reschedule',
)
```

**Best Practices:**

1. **Use reschedule mode for long waits** (>5 min) to free workers
2. **Set appropriate timeouts** - fail fast if data won't arrive
3. **Log what you're waiting for** - helps debugging
4. **Don't poll too frequently** - respect external systems
5. **Consider soft_fail=True** - skip downstream if sensor times out

**Optum Example:**

"Used S3KeySensor to wait for daily claims files from 50 different sources. Set timeout=3 hours with reschedule mode (poke_interval=300s). If file doesn't arrive by 5am, sensor fails and sends PagerDuty alert to source team. This automated coordination replaced 200 manual emails per month."

---

### Q15: Explain XCom (Cross-Communication). How do you pass data between tasks?

**Answer:**

**XCom (Cross-Communication)** = Mechanism to share small amounts of data between tasks in Airflow.

**Key Concepts:**
- Tasks push values to XCom
- Other tasks pull values from XCom
- Stored in Airflow metadata database
- **Size limit: ~48KB** (database BLOB column)

**Basic XCom Usage:**

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def extract_data(**context):
    """Extract data and push to XCom."""
    data = {'claim_count': 1500, 'total_amount': 2500000}

    # Method 1: Return value (automatically pushed to XCom)
    return data

def transform_data(**context):
    """Pull data from XCom and transform."""

    # Pull from previous task
    ti = context['ti']
    data = ti.xcom_pull(task_ids='extract')

    claim_count = data['claim_count']
    total_amount = data['total_amount']

    # Calculate average
    avg_claim_amount = total_amount / claim_count

    print(f"Average claim: ${avg_claim_amount:,.2f}")

    # Method 2: Explicit push to XCom
    ti.xcom_push(key='avg_claim_amount', value=avg_claim_amount)

    return {'processed': True}

def load_data(**context):
    """Pull data and load."""
    ti = context['ti']

    # Pull specific key
    avg_amount = ti.xcom_pull(task_ids='transform', key='avg_claim_amount')

    # Pull return value (default key='return_value')
    transform_result = ti.xcom_pull(task_ids='transform')

    print(f"Loading data... avg=${avg_amount}, status={transform_result}")

with DAG('xcom_example', start_date=datetime(2024, 1, 1), schedule_interval='@daily') as dag:

    extract = PythonOperator(task_id='extract', python_callable=extract_data)
    transform = PythonOperator(task_id='transform', python_callable=transform_data)
    load = PythonOperator(task_id='load', python_callable=load_data)

    extract >> transform >> load
```

**XCom with TaskFlow API (@task decorator):**

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(start_date=datetime(2024, 1, 1), schedule_interval='@daily')
def claims_pipeline():

    @task
    def extract_claims():
        """Return value automatically pushed to XCom."""
        return {'claims': [1, 2, 3, 4, 5]}

    @task
    def transform_claims(data: dict):
        """Input automatically pulled from XCom."""
        claims = data['claims']
        processed = [c * 2 for c in claims]
        return {'processed_claims': processed}

    @task
    def load_claims(data: dict):
        """Chain XCom pulls."""
        processed = data['processed_claims']
        print(f"Loading {len(processed)} claims")

    # Elegant syntax!
    extracted = extract_claims()
    transformed = transform_claims(extracted)
    load_claims(transformed)

dag = claims_pipeline()
```

**XCom Limitations & Solutions:**

**Problem 1: Size Limit (48KB)**

```python
# ❌ BAD: Passing large DataFrame via XCom
def extract_large_data():
    df = pd.read_csv('large_file.csv')  # 500MB
    return df.to_dict()  # XCom explosion!

# ✅ GOOD: Pass location instead
def extract_large_data():
    df = pd.read_csv('large_file.csv')

    # Save to S3
    s3_path = 's3://bucket/temp/data_{{ ts_nodash }}.parquet'
    df.to_parquet(s3_path)

    # Pass only path via XCom
    return {'s3_path': s3_path, 'row_count': len(df)}

def transform_large_data(**context):
    ti = context['ti']
    data_info = ti.xcom_pull(task_ids='extract')

    # Load from S3
    df = pd.read_parquet(data_info['s3_path'])
    print(f"Loaded {data_info['row_count']} rows")
```

**Problem 2: Serialization Issues**

```python
# ❌ BAD: Passing non-serializable objects
def extract():
    return pd.DataFrame({'a': [1, 2, 3]})  # Can't serialize DataFrame directly

# ✅ GOOD: Convert to serializable format
def extract():
    df = pd.DataFrame({'a': [1, 2, 3]})
    return df.to_dict('records')  # List of dicts

# Or use custom XCom backend (advanced)
```

**Custom XCom Backend (Store in S3 instead of DB):**

```python
# airflow.cfg
[core]
xcom_backend = custom_xcom_backend.CustomXComBackend

# custom_xcom_backend.py
from airflow.models.xcom import BaseXCom
import pickle
import boto3

class CustomXComBackend(BaseXCom):
    """Store large XComs in S3 instead of database."""

    @staticmethod
    def serialize_value(value):
        # If value > 1MB, store in S3
        serialized = pickle.dumps(value)

        if len(serialized) > 1_000_000:  # 1MB
            # Upload to S3
            s3 = boto3.client('s3')
            key = f'xcom/{uuid.uuid4()}.pkl'
            s3.put_object(Bucket='optum-airflow-xcom', Key=key, Body=serialized)

            # Return S3 reference
            return {'s3_key': key, '_is_s3': True}
        else:
            # Store in DB as usual
            return serialized

    @staticmethod
    def deserialize_value(result):
        if isinstance(result, dict) and result.get('_is_s3'):
            # Load from S3
            s3 = boto3.client('s3')
            obj = s3.get_object(Bucket='optum-airflow-xcom', Key=result['s3_key'])
            return pickle.loads(obj['Body'].read())
        else:
            return pickle.loads(result)
```

**XCom Best Practices:**

1. **Keep XCom small** (<10KB ideal) - pass metadata, not data
2. **Pass S3/HDFS paths** for large datasets
3. **Use TaskFlow API** (@task) for clean syntax
4. **Clean up XComs** - set `do_xcom_push=False` if not needed
5. **Don't rely on XCom for critical data** - use persistent storage

**Optum Example:**

"Used XCom to coordinate 10 parallel claim processing tasks. Each task processed 1M claims, pushed summary stats to XCom (count, total_amount, error_count). Final aggregation task pulled all XComs, calculated overall metrics, and sent to dashboard. XCom payload: ~2KB per task. Actual claim data: stored in Delta Lake."

---

### Q16: What is the difference between depends_on_past and wait_for_downstream?

**Answer:**

Both control task execution based on previous runs, but work differently.

**depends_on_past:**
- Task instance depends on **its own** previous run
- If yesterday's `task_a` failed, today's `task_a` won't run
- **Use case:** Incremental processing where each day depends on previous day

**wait_for_downstream:**
- Task depends on **all downstream tasks** of its previous run
- If yesterday's `task_a >> task_b` chain incomplete, today's `task_a` won't run
- **Use case:** Ensure entire pipeline completed before starting new run

**Visual Example:**

```
DAG Run 1 (Jan 1):  task_a >> task_b >> task_c
DAG Run 2 (Jan 2):  task_a >> task_b >> task_c

Scenario: Jan 1's task_b failed
```

**With depends_on_past=True:**
```python
task_b = BashOperator(
    task_id='task_b',
    bash_command='...',
    depends_on_past=True,  # Jan 2's task_b won't run (Jan 1's task_b failed)
)

# Result Jan 2:
# task_a: ✓ runs (no dependency on its own past)
# task_b: ✗ skipped (Jan 1's task_b failed)
# task_c: ✗ skipped (upstream task_b skipped)
```

**With wait_for_downstream=True:**
```python
task_a = BashOperator(
    task_id='task_a',
    bash_command='...',
    wait_for_downstream=True,  # Jan 2's task_a waits for Jan 1's downstream (task_b, task_c) to finish
)

# Result Jan 2:
# task_a: ✗ doesn't start (Jan 1's task_b still failed/incomplete)
# task_b: ✗ can't start (task_a hasn't run)
# task_c: ✗ can't start
```

**Real-World Example: Incremental Daily Load**

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def load_incremental_data(ds):
    """
    Load data for date ds, assuming previous day loaded successfully.
    """
    print(f"Loading data for {ds}")

    # This assumes data for (ds - 1 day) exists
    # If yesterday failed, today's load would be wrong!

    previous_day_path = f's3://bucket/data/{get_previous_day(ds)}'
    current_day_path = f's3://bucket/data/{ds}'

    # Incremental processing depends on previous day
    # ...

with DAG(
    'incremental_load',
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
) as dag:

    load_task = PythonOperator(
        task_id='load_incremental',
        python_callable=load_incremental_data,
        op_args=['{{ ds }}'],
        depends_on_past=True,  # Critical! Don't run if yesterday failed
    )
```

**Combining Both:**

```python
extract = BashOperator(
    task_id='extract',
    bash_command='extract.sh',
    wait_for_downstream=True,  # Ensure yesterday's full pipeline completed
)

transform = BashOperator(
    task_id='transform',
    bash_command='transform.sh',
    depends_on_past=True,  # Ensure yesterday's transform succeeded
)

load = BashOperator(
    task_id='load',
    bash_command='load.sh',
    depends_on_past=True,
)

extract >> transform >> load
```

**When to Use Each:**

| Scenario | Use depends_on_past | Use wait_for_downstream |
|----------|---------------------|-------------------------|
| Incremental daily processing | ✓ | |
| Each task builds on its own previous run | ✓ | |
| Must ensure entire pipeline finished yesterday | | ✓ |
| Parallel tasks that must all complete | | ✓ |
| Backfilling historical data | ✗ (set False) | ✗ (set False) |

**Optum Example:**

"For claims delta processing DAG, used `depends_on_past=True` on transform task. Each day processes delta since yesterday. If Jan 5 transform failed, Jan 6 transform must not run (would miss Jan 5 data). But extraction task uses `depends_on_past=False` (can re-extract even if yesterday failed). Prevented 15 data quality issues where gaps in incremental processing caused missing claims."

---

### Q17: How do you implement branching in Airflow? Explain BranchPythonOperator.

**Answer:**

**Branching** = Conditional logic in DAGs where execution path changes based on runtime conditions.

**BranchPythonOperator** = Returns task_id(s) to execute; other branches are skipped.

**Basic Branching:**

```python
from airflow import DAG
from airflow.operators.python import BranchPythonOperator
from airflow.operators.bash import BashOperator
from datetime import datetime

def decide_branch(**context):
    """
    Branching logic: Returns task_id to execute.
    """
    execution_date = context['execution_date']

    # Branch based on day of week
    if execution_date.weekday() < 5:  # Monday-Friday
        return 'process_weekday'
    else:  # Saturday-Sunday
        return 'process_weekend'

with DAG(
    'branching_example',
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
) as dag:

    start = BashOperator(
        task_id='start',
        bash_command='echo "Starting pipeline"',
    )

    branch = BranchPythonOperator(
        task_id='branch_logic',
        python_callable=decide_branch,
    )

    process_weekday = BashOperator(
        task_id='process_weekday',
        bash_command='echo "Processing weekday data"',
    )

    process_weekend = BashOperator(
        task_id='process_weekend',
        bash_command='echo "Processing weekend data"',
    )

    end = BashOperator(
        task_id='end',
        bash_command='echo "Pipeline complete"',
        trigger_rule='none_failed_min_one_success',  # Important!
    )

    start >> branch >> [process_weekday, process_weekend] >> end
```

**Key Point: trigger_rule**

By default, tasks require ALL upstream tasks to succeed. With branching, one branch is **skipped**, not failed.

```python
# ✗ Without trigger_rule - end task never runs (one upstream always skipped)
end = BashOperator(task_id='end', bash_command='echo done')

# ✓ With trigger_rule - end runs if at least one branch succeeded
end = BashOperator(
    task_id='end',
    bash_command='echo done',
    trigger_rule='none_failed_min_one_success',  # or 'one_success'
)
```

**Advanced: Multiple Branches**

```python
def decide_processing_type(**context):
    """
    Return different task_id based on data volume.
    """
    from airflow.hooks.S3_hook import S3Hook

    s3_hook = S3Hook(aws_conn_id='aws_default')

    # Check file size
    file_key = f"claims/{{ ds }}/claims.csv"
    file_size = s3_hook.get_key(file_key, 'optum-data').size

    # Branch based on size
    if file_size < 100_000_000:  # < 100MB
        return 'process_small'  # Single node
    elif file_size < 1_000_000_000:  # < 1GB
        return 'process_medium'  # Spark on small cluster
    else:
        return 'process_large'  # Spark on large cluster

branch = BranchPythonOperator(
    task_id='decide_processing',
    python_callable=decide_processing_type,
)

process_small = BashOperator(task_id='process_small', bash_command='python single_node.py')
process_medium = SparkSubmitOperator(task_id='process_medium', application='spark_medium.py')
process_large = SparkSubmitOperator(task_id='process_large', application='spark_large.py')

branch >> [process_small, process_medium, process_large]
```

**Branching with Multiple Downstream Tasks:**

```python
def choose_multiple_branches(**context):
    """
    Return LIST of task_ids to execute.
    """
    execution_hour = context['execution_date'].hour

    if execution_hour < 12:
        # Morning: Process claims and eligibility
        return ['process_claims', 'process_eligibility']
    else:
        # Afternoon: Process claims only
        return ['process_claims']

branch = BranchPythonOperator(
    task_id='choose_processing',
    python_callable=choose_multiple_branches,
)

process_claims = BashOperator(task_id='process_claims', bash_command='...')
process_eligibility = BashOperator(task_id='process_eligibility', bash_command='...')
process_providers = BashOperator(task_id='process_providers', bash_command='...')

# eligibility is optional (branched)
# providers always runs (not in branch options, but downstream)
branch >> [process_claims, process_eligibility]
process_claims >> process_providers
```

**ShortCircuitOperator (Alternative):**

```python
from airflow.operators.python import ShortCircuitOperator

def check_if_run(**context):
    """
    Return True to continue pipeline, False to short-circuit (stop).
    """
    from airflow.hooks.postgres_hook import PostgresHook

    pg_hook = PostgresHook(postgres_conn_id='postgres_default')

    # Check if data exists
    result = pg_hook.get_first(
        f"SELECT COUNT(*) FROM claims WHERE date = '{{ ds }}'"
    )

    row_count = result[0]

    if row_count > 0:
        return True  # Continue pipeline
    else:
        print("No data for today, skipping pipeline")
        return False  # Short-circuit (skip all downstream)

gate = ShortCircuitOperator(
    task_id='check_data_exists',
    python_callable=check_if_run,
)

process = BashOperator(task_id='process', bash_command='...')
load = BashOperator(task_id='load', bash_command='...')

gate >> process >> load
# If gate returns False, process and load are SKIPPED
```

**Branching Best Practices:**

1. **Keep branching logic simple** - complex logic in external code
2. **Use trigger_rule downstream** - handle skipped branches
3. **Log branching decisions** - why did it choose this branch?
4. **Test all branches** - don't forget edge cases
5. **Consider ShortCircuitOperator** for simple on/off logic

**Optum Example:**

"Claims processing DAG branches based on claim type (medical vs pharmacy). BranchPythonOperator checks first record in file, returns 'process_medical' or 'process_pharmacy'. Each branch has different validation rules and destinations. Downstream aggregation task uses trigger_rule='none_failed_min_one_success' to run regardless of which branch executed. Handles 50M claims/month, 70% medical, 30% pharmacy."

---

### Q18: What are SubDAGs vs TaskGroups? When to use each?

**Answer:**

Both organize related tasks, but **TaskGroups** are recommended (SubDAGs deprecated).

**TaskGroups (Recommended):**
- **UI grouping only** - visual organization
- **Same scheduling context** as parent DAG
- **No separate executor** - runs in same environment
- **Lightweight** - just UI sugar

**SubDAGs (Deprecated):**
- **Separate DAG** with own schedule/executor
- **Complexity** - separate metadata, can deadlock
- **Avoid** - being removed in Airflow 3.0

**TaskGroups Example:**

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.utils.task_group import TaskGroup
from datetime import datetime

with DAG(
    'claims_with_task_groups',
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
) as dag:

    start = BashOperator(task_id='start', bash_command='echo start')

    # Task Group: Medical Claims Processing
    with TaskGroup(group_id='medical_claims') as medical_claims:
        extract_medical = BashOperator(
            task_id='extract',
            bash_command='python extract_medical.py',
        )

        validate_medical = BashOperator(
            task_id='validate',
            bash_command='python validate_medical.py',
        )

        load_medical = BashOperator(
            task_id='load',
            bash_command='python load_medical.py',
        )

        extract_medical >> validate_medical >> load_medical

    # Task Group: Pharmacy Claims Processing
    with TaskGroup(group_id='pharmacy_claims') as pharmacy_claims:
        extract_pharmacy = BashOperator(
            task_id='extract',
            bash_command='python extract_pharmacy.py',
        )

        validate_pharmacy = BashOperator(
            task_id='validate',
            bash_command='python validate_pharmacy.py',
        )

        load_pharmacy = BashOperator(
            task_id='load',
            bash_command='python load_pharmacy.py',
        )

        extract_pharmacy >> validate_pharmacy >> load_pharmacy

    end = BashOperator(task_id='end', bash_command='echo end')

    # Dependencies
    start >> [medical_claims, pharmacy_claims] >> end
```

**UI Visualization:**

```
├── start
├── medical_claims (collapsed)
│   ├── extract
│   ├── validate
│   └── load
├── pharmacy_claims (collapsed)
│   ├── extract
│   ├── validate
│   └── load
└── end
```

**Reusable TaskGroups (Function):**

```python
def create_processing_group(group_id: str, source_type: str):
    """
    Reusable TaskGroup factory.
    """
    with TaskGroup(group_id=group_id) as group:
        extract = BashOperator(
            task_id='extract',
            bash_command=f'python extract_{source_type}.py {{{{ ds }}}}',
        )

        transform = BashOperator(
            task_id='transform',
            bash_command=f'python transform_{source_type}.py {{{{ ds }}}}',
        )

        load = BashOperator(
            task_id='load',
            bash_command=f'python load_{source_type}.py {{{{ ds }}}}',
        )

        extract >> transform >> load

    return group

# Use the factory
with DAG('multi_source_pipeline', start_date=datetime(2024, 1, 1), schedule_interval='@daily') as dag:

    start = BashOperator(task_id='start', bash_command='echo start')

    # Create multiple groups
    medical_group = create_processing_group('medical_claims', 'medical')
    pharmacy_group = create_processing_group('pharmacy_claims', 'pharmacy')
    dental_group = create_processing_group('dental_claims', 'dental')

    end = BashOperator(task_id='end', bash_command='echo end')

    start >> [medical_group, pharmacy_group, dental_group] >> end
```

**Dynamic TaskGroups:**

```python
from airflow.utils.task_group import TaskGroup
import yaml

# Load configuration
SOURCES = yaml.safe_load(open('sources.yaml'))
# sources.yaml:
# - medical
# - pharmacy
# - dental
# - vision

with DAG('dynamic_task_groups', start_date=datetime(2024, 1, 1)) as dag:

    start = BashOperator(task_id='start', bash_command='echo start')

    source_groups = []

    for source in SOURCES:
        with TaskGroup(group_id=f'{source}_claims') as group:
            extract = BashOperator(
                task_id='extract',
                bash_command=f'python extract_{source}.py',
            )

            transform = BashOperator(
                task_id='transform',
                bash_command=f'python transform_{source}.py',
            )

            load = BashOperator(
                task_id='load',
                bash_command=f'python load_{source}.py',
            )

            extract >> transform >> load

            source_groups.append(group)

    end = BashOperator(task_id='end', bash_command='echo end')

    start >> source_groups >> end
```

**Accessing Tasks in TaskGroup:**

```python
# Reference task: group_id.task_id
medical_extract_task = dag.get_task('medical_claims.extract')

# Set dependencies with specific task in group
start >> medical_claims.extract  # Won't work!
start >> medical_claims  # Correct - depends on entire group
```

**SubDAGs (Legacy - Don't Use):**

```python
# ❌ OLD WAY - SubDAGs (avoid)
from airflow.operators.subdag import SubDagOperator

def create_subdag(parent_dag_id, child_dag_id, args):
    with DAG(
        dag_id=f'{parent_dag_id}.{child_dag_id}',
        default_args=args,
        schedule_interval=None,  # Inherited
    ) as subdag:
        task1 = BashOperator(task_id='task1', bash_command='...')
        task2 = BashOperator(task_id='task2', bash_command='...')
        task1 >> task2
    return subdag

# In parent DAG
subdag_task = SubDagOperator(
    task_id='sub_pipeline',
    subdag=create_subdag('parent_dag', 'sub_pipeline', default_args),
)
# Problems: Deadlocks, scheduler issues, deprecated
```

**Comparison:**

| Feature | TaskGroups | SubDAGs |
|---------|------------|---------|
| **Execution** | Same DAG run | Separate DAG run |
| **Scheduler** | Same context | Separate scheduling |
| **UI** | Collapsible groups | Separate DAG page |
| **Performance** | Fast | Can deadlock |
| **Recommended** | ✓ Yes | ✗ Deprecated |
| **Airflow 3.0** | Supported | Removed |

**Best Practices:**

1. **Use TaskGroups** - SubDAGs deprecated
2. **Organize related tasks** - better DAG readability
3. **Reusable functions** - DRY principle
4. **Collapse in UI** - cleaner visualization
5. **Document groups** - add tooltips/descriptions

**Optum Example:**

"Refactored 50-task claims processing DAG into 5 TaskGroups (extract, validate, transform, load, audit). Each group has 10 tasks for different claim types. TaskGroups reduced UI clutter, improved troubleshooting (drill into specific group), and enabled parallel development (teams own their TaskGroup). Replaced previous SubDAG implementation which caused scheduler slowdowns during peak (SubDAGs blocked executor slots)."

---

### Q19: How do you handle retries, timeouts, and SLAs in Airflow?

**Answer:**

**Retries, Timeouts, and SLAs** = Critical for production reliability.

**1. RETRIES**

**Task-Level Retries:**

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta

# Default args (apply to all tasks in DAG)
default_args = {
    'owner': 'data-engineering',
    'retries': 3,  # Retry 3 times
    'retry_delay': timedelta(minutes=5),  # Wait 5 min between retries
    'retry_exponential_backoff': True,  # Exponential backoff
    'max_retry_delay': timedelta(minutes=30),  # Max 30 min between retries
}

with DAG(
    'retry_example',
    default_args=default_args,
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
) as dag:

    # Task inherits default_args
    task1 = BashOperator(
        task_id='task1',
        bash_command='python might_fail.py',
        # Overrides:
        retries=5,  # This task gets 5 retries
        retry_delay=timedelta(seconds=30),  # 30 sec between retries
    )
```

**Exponential Backoff:**

```
Retry attempt   Delay
1               5 minutes
2               10 minutes (2x)
3               20 minutes (2x)
4               30 minutes (max_retry_delay)
```

**Conditional Retries (Custom Logic):**

```python
from airflow.exceptions import AirflowException

def task_with_smart_retry(**context):
    """
    Retry only for specific errors.
    """
    import requests
    from requests.exceptions import ConnectionError, Timeout

    try:
        response = requests.get('https://api.example.com/data')
        response.raise_for_status()

        return response.json()

    except (ConnectionError, Timeout) as e:
        # Transient errors - allow retry
        raise AirflowException(f"Transient error: {e}")

    except requests.exceptions.HTTPError as e:
        if e.response.status_code == 404:
            # Data doesn't exist - don't retry
            print("Data not found, skipping")
            return None
        elif e.response.status_code >= 500:
            # Server error - retry
            raise AirflowException(f"Server error: {e}")
        else:
            # Client error (400-499) - don't retry
            raise AirflowException(f"Client error (won't retry): {e}")

task = PythonOperator(
    task_id='smart_retry_task',
    python_callable=task_with_smart_retry,
    retries=3,
    retry_delay=timedelta(minutes=5),
)
```

**2. TIMEOUTS**

**Execution Timeout:**

```python
task = BashOperator(
    task_id='long_running_task',
    bash_command='python long_process.py',
    execution_timeout=timedelta(hours=2),  # Kill after 2 hours
)

# If task runs >2 hours:
# - Task marked FAILED
# - Retries can still occur (if configured)
```

**SLA (Service Level Agreement) - Timeout for DAG/Task Completion:**

```python
# Task-level SLA
task = BashOperator(
    task_id='sla_task',
    bash_command='python process.py',
    sla=timedelta(minutes=30),  # Must complete within 30 min
)

# DAG-level SLA
with DAG(
    'sla_dag',
    default_args=default_args,
    start_date=datetime(2024, 1, 1),
    schedule_interval='@hourly',
    sla_miss_callback=notify_sla_miss,  # Callback function
) as dag:
    pass

def notify_sla_miss(dag, task_list, blocking_task_list, slas, blocking_tis):
    """
    Called when SLA is missed.
    """
    print(f"SLA MISS!")
    print(f"DAG: {dag.dag_id}")
    print(f"Tasks missed SLA: {[task.task_id for task in task_list]}")
    print(f"Blocking tasks: {[task.task_id for task in blocking_task_list]}")

    # Send alert
    send_pagerduty_alert(f"SLA miss in {dag.dag_id}")
```

**Sensor Timeout:**

```python
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor

wait_for_file = S3KeySensor(
    task_id='wait_for_data',
    bucket_name='optum-data',
    bucket_key='claims/{{ ds }}/claims.csv',
    timeout=3600,  # Wait max 1 hour
    poke_interval=60,  # Check every 60 seconds
    mode='reschedule',  # Free up worker between checks
    soft_fail=True,  # If timeout, mark SUCCESS (skip downstream) instead of FAIL
)
```

**3. EMAIL/SLACK ALERTS**

**Email on Failure:**

```python
default_args = {
    'email': ['data-eng@optum.com', 'oncall@optum.com'],
    'email_on_failure': True,
    'email_on_retry': False,  # Don't spam on retries
    'email_on_success': False,  # Only alert on problems
}
```

**Slack on Failure:**

```python
from airflow.providers.slack.operators.slack_webhook import SlackWebhookOperator

def task_fail_slack_alert(context):
    """
    Callback function on task failure.
    """
    slack_msg = f"""
    :red_circle: Task Failed
    *DAG*: {context.get('task_instance').dag_id}
    *Task*: {context.get('task_instance').task_id}
    *Execution Time*: {context.get('execution_date')}
    *Log URL*: {context.get('task_instance').log_url}
    """

    failed_alert = SlackWebhookOperator(
        task_id='slack_alert',
        http_conn_id='slack_webhook',
        message=slack_msg,
        username='airflow-bot',
    )

    return failed_alert.execute(context=context)

task = BashOperator(
    task_id='important_task',
    bash_command='python critical_process.py',
    on_failure_callback=task_fail_slack_alert,  # Call on failure
    retries=3,
)
```

**PagerDuty on SLA Miss:**

```python
def pagerduty_sla_alert(dag, task_list, blocking_task_list, slas, blocking_tis):
    """
    Trigger PagerDuty incident on SLA miss.
    """
    import requests

    pagerduty_payload = {
        "routing_key": "YOUR_PAGERDUTY_KEY",
        "event_action": "trigger",
        "payload": {
            "summary": f"SLA MISS: {dag.dag_id}",
            "severity": "critical",
            "source": "airflow",
            "custom_details": {
                "tasks": [task.task_id for task in task_list],
                "blocking": [task.task_id for task in blocking_task_list],
            }
        }
    }

    requests.post(
        'https://events.pagerduty.com/v2/enqueue',
        json=pagerduty_payload
    )

with DAG(
    'critical_pipeline',
    sla_miss_callback=pagerduty_sla_alert,
    ...
) as dag:
    pass
```

**4. CIRCUIT BREAKER PATTERN**

```python
def task_with_circuit_breaker(**context):
    """
    Fail fast if upstream dependency is down.
    """
    import requests

    # Check health endpoint first
    try:
        health = requests.get('https://api.example.com/health', timeout=5)
        if health.status_code != 200:
            raise AirflowException("API unhealthy - circuit breaker triggered")
    except requests.exceptions.RequestException:
        raise AirflowException("API unreachable - circuit breaker triggered")

    # Proceed with actual task
    # ...

task = PythonOperator(
    task_id='api_task',
    python_callable=task_with_circuit_breaker,
    retries=0,  # Don't retry circuit breaker failures
)
```

**Best Practices:**

1. **Set realistic retry counts** - 3 retries typical, 5 max
2. **Exponential backoff** - don't hammer failing services
3. **Timeouts always** - prevent hung tasks
4. **SLAs for critical pipelines** - know when things are slow
5. **Alert fatigue** - don't email on every retry
6. **Idempotency** - tasks must be rerunnable safely

**Optum Example:**

"Claims processing DAG: 3 retries with 5-minute exponential backoff. SLA=2 hours (must complete by 4am for morning reports). If SLA missed, PagerDuty pages on-call engineer. Execution timeout=90 minutes per task (prevents hung SQL queries). S3 sensor timeout=60 minutes with soft_fail=True (if file doesn't arrive, skip processing gracefully rather than fail entire DAG). Results: 99.7% SLA adherence, 2% failure rate (down from 15% with cron), average recovery time 12 minutes (automatic retries)."

---

### Q20: How do you secure Airflow? (Secrets management, RBAC, authentication)

**Answer:**

**Airflow Security** = Critical for production, especially with sensitive healthcare data.

**1. AUTHENTICATION**

**Enable Authentication:**

```python
# airflow.cfg
[webserver]
auth_backend = airflow.auth.backends.password_auth
authenticate = True
rbac = True  # Enable RBAC
```

**Authentication Backends:**

| Backend | Use Case |
|---------|----------|
| `password_auth` | Username/password in database |
| `ldap` | Corporate LDAP/Active Directory |
| `oauth` | SSO (Google, Okta, Azure AD) |
| `kerberos` | Enterprise Hadoop clusters |

**LDAP (Active Directory):**

```python
# airflow.cfg
[webserver]
auth_backend = airflow.auth.backends.ldap_auth
auth_ldap_server = ldaps://ldap.optum.com:636
auth_ldap_bind_user = cn=airflow,ou=service,dc=optum,dc=com
auth_ldap_bind_password = <password>
auth_ldap_search = ou=users,dc=optum,dc=com
auth_ldap_uid_field = sAMAccountName
```

**OAuth (Azure AD):**

```python
# webserver_config.py
from flask_appbuilder.security.manager import AUTH_OAUTH

AUTH_TYPE = AUTH_OAUTH
OAUTH_PROVIDERS = [{
    'name': 'azure',
    'icon': 'fa-windows',
    'token_key': 'access_token',
    'remote_app': {
        'client_id': 'YOUR_CLIENT_ID',
        'client_secret': 'YOUR_CLIENT_SECRET',
        'api_base_url': 'https://graph.microsoft.com/v1.0/',
        'client_kwargs': {
            'scope': 'User.Read'
        },
        'access_token_url': 'https://login.microsoftonline.com/<TENANT>/oauth2/v2.0/token',
        'authorize_url': 'https://login.microsoftonline.com/<TENANT>/oauth2/v2.0/authorize',
    }
}]
```

**2. RBAC (Role-Based Access Control)**

**Built-in Roles:**

| Role | Permissions |
|------|-------------|
| `Admin` | Full access (create DAGs, users, connections) |
| `User` | Trigger DAGs, view logs |
| `Op` | Operators (trigger, clear, etc.) |
| `Viewer` | Read-only access |
| `Public` | No authentication required (dangerous!) |

**Create Custom Role:**

```python
from airflow.www.security import AirflowSecurityManager

class CustomSecurityManager(AirflowSecurityManager):
    def __init__(self, appbuilder):
        super().__init__(appbuilder)

        # Create custom role
        self.add_role('ClaimsTeam')

        # Grant permissions
        self.add_permission_to_role('ClaimsTeam', 'can_read', 'DAG:claims_*')
        self.add_permission_to_role('ClaimsTeam', 'can_edit', 'DAG:claims_*')
        self.add_permission_to_role('ClaimsTeam', 'can_trigger', 'DAG:claims_*')

        # Deny other DAGs
        # (by default, no permission = denied)

# airflow.cfg
[webserver]
rbac_user_registration_role = Viewer  # Default role for new users
```

**Assign Users to Roles:**

```bash
# CLI
airflow users create \
    --username john.doe \
    --firstname John \
    --lastname Doe \
    --role ClaimsTeam \
    --email john.doe@optum.com \
    --password <password>

# Or via UI: Security > List Users > Edit
```

**3. SECRETS MANAGEMENT**

**❌ BAD: Hardcoded Secrets**

```python
# DON'T DO THIS!
task = BashOperator(
    task_id='query_db',
    bash_command='psql -h db.optum.com -U admin -p MyPassword123',  # 😱
)
```

**✅ GOOD: Airflow Connections**

```python
# Store connection in Airflow metadata DB
# airflow connections add 'postgres_prod' \
#     --conn-type postgres \
#     --conn-host db.optum.com \
#     --conn-login admin \
#     --conn-password MyPassword123 \
#     --conn-port 5432 \
#     --conn-schema claims

from airflow.providers.postgres.operators.postgres import PostgresOperator

task = PostgresOperator(
    task_id='query_db',
    postgres_conn_id='postgres_prod',  # Reference stored connection
    sql='SELECT * FROM claims WHERE date = {{ ds }}',
)
```

**✅ BETTER: Azure Key Vault Backend**

```python
# airflow.cfg
[secrets]
backend = airflow.providers.microsoft.azure.secrets.key_vault.AzureKeyVaultBackend
backend_kwargs = {
    "connections_prefix": "airflow-connections",
    "variables_prefix": "airflow-variables",
    "vault_url": "https://optum-airflow-kv.vault.azure.net/"
}

# Store connection in Key Vault (one-time setup)
# az keyvault secret set \
#     --vault-name optum-airflow-kv \
#     --name airflow-connections-postgres-prod \
#     --value '{"conn_type": "postgres", "host": "db.optum.com", ...}'

# Code remains same - Airflow fetches from Key Vault automatically
task = PostgresOperator(
    task_id='query_db',
    postgres_conn_id='postgres_prod',  # Fetched from Key Vault
    sql='SELECT * FROM claims',
)
```

**AWS Secrets Manager:**

```python
# airflow.cfg
[secrets]
backend = airflow.providers.amazon.aws.secrets.secrets_manager.SecretsManagerBackend
backend_kwargs = {
    "connections_prefix": "airflow/connections",
    "variables_prefix": "airflow/variables",
    "region_name": "us-east-1"
}
```

**HashiCorp Vault:**

```python
[secrets]
backend = airflow.providers.hashicorp.secrets.vault.VaultBackend
backend_kwargs = {
    "url": "https://vault.optum.com:8200",
    "token": "<vault-token>",
    "connections_path": "airflow/connections",
    "variables_path": "airflow/variables"
}
```

**4. NETWORK SECURITY**

**Enable HTTPS:**

```python
# airflow.cfg
[webserver]
web_server_ssl_cert = /path/to/cert.pem
web_server_ssl_key = /path/to/key.pem
```

**Firewall Rules:**

```bash
# Only allow access from corporate VPN
# iptables
iptables -A INPUT -p tcp --dport 8080 -s 10.0.0.0/8 -j ACCEPT
iptables -A INPUT -p tcp --dport 8080 -j DROP

# Or use Azure NSG / AWS Security Groups
```

**5. AUDIT LOGGING**

```python
# airflow.cfg
[logging]
remote_logging = True
remote_base_log_folder = s3://optum-airflow-logs/
encrypt_s3_logs = True

# Log retention
log_retention_days = 90  # 90 days for HIPAA compliance
```

**Audit Log DAG Access:**

```python
from airflow.utils.log.logging_mixin import LoggingMixin

class AuditLogger(LoggingMixin):
    def log_dag_trigger(self, dag_id, user, execution_date):
        self.log.info(f"AUDIT: DAG {dag_id} triggered by {user} for {execution_date}")

        # Send to SIEM
        send_to_splunk({
            'event_type': 'dag_trigger',
            'dag_id': dag_id,
            'user': user,
            'timestamp': datetime.now().isoformat(),
        })

# In DAG
def my_task(**context):
    audit = AuditLogger()
    audit.log_dag_trigger(
        context['dag'].dag_id,
        context['dag_run'].conf.get('triggered_by'),
        context['execution_date']
    )
    # ... actual task logic
```

**6. ENCRYPTION**

**Encrypt Connections in DB:**

```python
# airflow.cfg
[core]
fernet_key = <generated-fernet-key>

# Generate key:
# python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
```

**Encrypt Variables:**

```bash
# Encrypted variable (in Airflow Variable editor, check "Encrypt")
airflow variables set api_key "secret123" --encrypt

# Fetch decrypts automatically
from airflow.models import Variable
api_key = Variable.get("api_key")  # Decrypted
```

**7. SECURITY BEST PRACTICES CHECKLIST**

- [ ] Enable authentication (LDAP/OAuth)
- [ ] Enable RBAC
- [ ] Use secrets backend (Key Vault/Secrets Manager)
- [ ] Enable HTTPS
- [ ] Encrypt connections (Fernet key)
- [ ] Network isolation (VPN/private subnet)
- [ ] Audit logging to SIEM
- [ ] Rotate secrets regularly
- [ ] Principle of least privilege (minimal permissions)
- [ ] Security scanning (SonarQube, Snyk)
- [ ] Disable experimental features in production
- [ ] Keep Airflow updated (security patches)

**Optum Security Setup:**

```python
# Production configuration
[webserver]
auth_backend = airflow.auth.backends.ldap_auth  # Optum AD
rbac = True
web_server_ssl_cert = /etc/ssl/airflow.crt
web_server_ssl_key = /etc/ssl/airflow.key

[secrets]
backend = airflow.providers.microsoft.azure.secrets.key_vault.AzureKeyVaultBackend
backend_kwargs = {"vault_url": "https://optum-prod-kv.vault.azure.net/"}

[core]
fernet_key = <rotated-quarterly>
encrypt_s3_logs = True

[logging]
remote_base_log_folder = abfss://logs@optumdatalake.dfs.core.windows.net/airflow/
log_retention_days = 2555  # 7 years for HIPAA
```

**Optum Example:**

"Implemented defense-in-depth security for Airflow managing 10,000+ daily tasks with PHI data: Azure AD OAuth for SSO (2000 users), RBAC with 15 custom roles (ClaimsTeam can only access claims_* DAGs), Azure Key Vault for 200+ secrets (DB credentials, API keys, service principals), all connections encrypted with Fernet (key rotated quarterly), HTTPS only with corporate SSL cert, NSG limiting access to corporate VPN (10.0.0.0/8), audit logs shipped to Splunk in real-time (SIEM), passed SOC 2 and HIPAA audits with zero findings. Security incident: detected unauthorized DAG trigger attempt in audit logs, user account locked within 5 minutes, root cause analysis showed compromised laptop (malware), reinforced need for MFA on all accounts."

---



---

### Q21: How do you test Airflow DAGs? Explain unit testing and DAG validation strategies.

**Answer:**

**Testing Airflow DAGs** = Critical for production reliability. Multiple testing levels required.

**Testing Pyramid:**
1. **DAG Validation** (structure/syntax)
2. **Unit Tests** (individual task logic)
3. **Integration Tests** (end-to-end with dependencies)
4. **Smoke Tests** (basic deployment checks)

---

#### **1. DAG Validation Tests**

**Purpose:** Ensure DAGs load without errors, no cycles, valid configuration.

```python
# tests/test_dag_validation.py
import pytest
from airflow.models import DagBag

def test_no_import_errors():
    """Test that all DAGs load without import errors."""
    dag_bag = DagBag(include_examples=False)

    assert len(dag_bag.import_errors) == 0, \
        f"DAG import errors: {dag_bag.import_errors}"

def test_all_dags_have_tags():
    """Ensure all DAGs have required tags."""
    dag_bag = DagBag(include_examples=False)

    for dag_id, dag in dag_bag.dags.items():
        assert len(dag.tags) > 0, f"DAG {dag_id} has no tags"
        assert 'team' in [tag for tag in dag.tags if 'team:' in tag], \
            f"DAG {dag_id} missing team tag"

def test_all_dags_have_owners():
    """Ensure all DAGs have valid owners."""
    dag_bag = DagBag(include_examples=False)

    for dag_id, dag in dag_bag.dags.items():
        assert dag.owner is not None, f"DAG {dag_id} has no owner"
        assert dag.owner != 'airflow', f"DAG {dag_id} uses default owner"

def test_dag_has_correct_schedule():
    """Test specific DAG has expected schedule."""
    dag_bag = DagBag(include_examples=False)
    dag = dag_bag.get_dag('claims_processing_pipeline')

    assert dag.schedule_interval == '@daily'
    assert dag.catchup is False

def test_no_cyclic_dependencies():
    """Ensure no DAGs have circular dependencies."""
    dag_bag = DagBag(include_examples=False)

    for dag_id, dag in dag_bag.dags.items():
        # This would raise if there are cycles
        try:
            dag.test_cycle()
        except Exception as e:
            pytest.fail(f"DAG {dag_id} has cyclic dependencies: {e}")
```

---

#### **2. Unit Tests for Task Logic**

**Purpose:** Test individual task functions in isolation.

```python
# tests/test_task_logic.py
import pytest
from datetime import datetime
from airflow.models import DagBag, TaskInstance
from airflow.utils.state import State

def test_extract_claims_function():
    """Test the extract_claims task logic."""
    from dags.claims_pipeline import extract_claims

    # Mock inputs
    execution_date = datetime(2024, 5, 1)

    # Test function directly
    result = extract_claims(execution_date)

    assert result is not None
    assert 'claims' in result
    assert len(result['claims']) > 0

def test_transform_claims_function():
    """Test the transform_claims task logic."""
    from dags.claims_pipeline import transform_claims

    # Mock input data
    claims_data = {
        'claims': [
            {'claim_id': '123', 'amount': 100},
            {'claim_id': '456', 'amount': 200},
        ]
    }

    result = transform_claims(claims_data)

    assert result is not None
    assert 'transformed_claims' in result
    assert len(result['transformed_claims']) == 2

def test_validate_claims_function():
    """Test claims validation logic."""
    from dags.claims_pipeline import validate_claims

    valid_claims = [
        {'claim_id': '123', 'amount': 100, 'member_id': 'M001'},
    ]

    invalid_claims = [
        {'claim_id': '456'},  # Missing required fields
    ]

    # Should not raise for valid claims
    validate_claims(valid_claims)

    # Should raise for invalid claims
    with pytest.raises(ValueError):
        validate_claims(invalid_claims)
```

---

#### **3. Task Dependency Tests**

**Purpose:** Validate task dependencies are correctly defined.

```python
# tests/test_task_dependencies.py
import pytest
from airflow.models import DagBag

def test_claims_pipeline_task_dependencies():
    """Test that claims pipeline has correct task order."""
    dag_bag = DagBag(include_examples=False)
    dag = dag_bag.get_dag('claims_processing_pipeline')

    # Get tasks
    extract_task = dag.get_task('extract_claims')
    validate_task = dag.get_task('validate_claims')
    transform_task = dag.get_task('transform_claims')
    load_task = dag.get_task('load_to_delta')

    # Test dependencies
    assert validate_task in extract_task.downstream_list
    assert transform_task in validate_task.downstream_list
    assert load_task in transform_task.downstream_list

def test_branching_dependencies():
    """Test that branching logic has correct paths."""
    dag_bag = DagBag(include_examples=False)
    dag = dag_bag.get_dag('conditional_claims_pipeline')

    branch_task = dag.get_task('check_claim_type')
    inpatient_task = dag.get_task('process_inpatient')
    outpatient_task = dag.get_task('process_outpatient')

    # Both paths should be downstream of branch
    assert inpatient_task in branch_task.downstream_list
    assert outpatient_task in branch_task.downstream_list
```

---

#### **4. Integration Tests (with TestDag)**

**Purpose:** Test full DAG execution end-to-end.

```python
# tests/test_dag_integration.py
import pytest
from datetime import datetime
from airflow.models import DagBag
from airflow.utils.state import State

def test_claims_pipeline_success():
    """Test full claims pipeline execution."""
    dag_bag = DagBag(include_examples=False)
    dag = dag_bag.get_dag('claims_processing_pipeline')

    execution_date = datetime(2024, 5, 1)

    # Run the DAG
    dag.test(
        execution_date=execution_date,
        run_conf={'test_mode': True}  # Use test data
    )

    # If no exceptions raised, DAG succeeded

def test_dag_with_mock_data():
    """Test DAG with mocked external dependencies."""
    from unittest.mock import patch
    from airflow.models import DagBag

    dag_bag = DagBag(include_examples=False)
    dag = dag_bag.get_dag('claims_processing_pipeline')

    execution_date = datetime(2024, 5, 1)

    # Mock external API calls
    with patch('dags.claims_pipeline.fetch_claims_from_api') as mock_fetch:
        mock_fetch.return_value = [
            {'claim_id': '123', 'amount': 100}
        ]

        dag.test(execution_date=execution_date)

        # Verify mock was called
        mock_fetch.assert_called_once()
```

---

#### **5. CI/CD Integration**

**GitHub Actions / Azure DevOps:**

```yaml
# .github/workflows/test-dags.yml
name: Test Airflow DAGs

on:
  pull_request:
    paths:
      - 'dags/**'
      - 'tests/**'

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - name: Install dependencies
        run: |
          pip install apache-airflow==2.8.0
          pip install pytest pytest-cov
          pip install -r requirements.txt

      - name: Run DAG validation tests
        run: |
          pytest tests/test_dag_validation.py -v

      - name: Run unit tests
        run: |
          pytest tests/test_task_logic.py -v --cov=dags

      - name: Run integration tests
        run: |
          pytest tests/test_dag_integration.py -v

      - name: Check test coverage
        run: |
          pytest --cov=dags --cov-report=html --cov-fail-under=80
```

---

#### **6. Pre-commit Hooks**

**Validate DAGs before commit:**

```yaml
# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: validate-dags
        name: Validate Airflow DAGs
        entry: python scripts/validate_dags.py
        language: python
        files: ^dags/
        pass_filenames: false
```

```python
# scripts/validate_dags.py
"""Validate all DAGs can be loaded."""
import sys
from airflow.models import DagBag

def validate_dags():
    dag_bag = DagBag(include_examples=False)

    if dag_bag.import_errors:
        print("❌ DAG Import Errors:")
        for filename, error in dag_bag.import_errors.items():
            print(f"  {filename}: {error}")
        sys.exit(1)

    print(f"✅ All {len(dag_bag.dags)} DAGs loaded successfully")

    # Additional validations
    for dag_id, dag in dag_bag.dags.items():
        # Check owner
        if dag.owner == 'airflow':
            print(f"⚠️  DAG {dag_id} uses default owner")

        # Check tags
        if not dag.tags:
            print(f"⚠️  DAG {dag_id} has no tags")

    sys.exit(0)

if __name__ == '__main__':
    validate_dags()
```

---

#### **7. Testing Custom Operators**

```python
# tests/test_custom_operators.py
import pytest
from datetime import datetime
from airflow.models import TaskInstance
from dags.operators.custom_claims_operator import ClaimsProcessingOperator

def test_custom_claims_operator():
    """Test custom ClaimsProcessingOperator."""
    operator = ClaimsProcessingOperator(
        task_id='test_claims',
        claim_type='inpatient',
        processing_date='2024-05-01'
    )

    # Mock context
    context = {
        'execution_date': datetime(2024, 5, 1),
        'ti': None,
    }

    # Execute operator
    result = operator.execute(context)

    assert result is not None
    assert result['status'] == 'success'
```

---

#### **8. Testing with Fixtures**

```python
# tests/conftest.py
import pytest
from airflow.models import DagBag, Connection
from airflow import settings

@pytest.fixture(scope='session')
def dag_bag():
    """Provide a DagBag for all tests."""
    return DagBag(include_examples=False)

@pytest.fixture
def test_dag(dag_bag):
    """Provide a specific DAG for testing."""
    return dag_bag.get_dag('claims_processing_pipeline')

@pytest.fixture
def mock_connections():
    """Create mock Airflow connections for tests."""
    session = settings.Session()

    # Add test connections
    conn = Connection(
        conn_id='postgres_claims_test',
        conn_type='postgres',
        host='localhost',
        schema='test_db',
        login='test_user',
        password='test_pass',
        port=5432
    )
    session.add(conn)
    session.commit()

    yield

    # Cleanup
    session.query(Connection).filter(
        Connection.conn_id == 'postgres_claims_test'
    ).delete()
    session.commit()
```

---

### **Your Optum Testing Strategy:**

**1. Automated Tests Run on Every PR:**
- DAG validation (no syntax errors)
- Unit tests for all task functions
- Integration tests for critical pipelines
- Coverage minimum 80%

**2. Pre-Deployment Checks:**
- Smoke tests on staging environment
- Test DAG runs with sample data
- Performance tests (task duration < SLA)

**3. Production Monitoring:**
- Track test coverage metrics
- Monitor DAG failure rates
- Alert on repeated test failures

**4. Test Data Management:**
- Maintain test datasets in separate Azure storage
- Use synthetic claims data for testing
- Refresh test data weekly

**Business Impact:**
- **Reduced incidents by 75%** (from 8/month to 2/month)
- **Faster releases**: 3 days → 1 day (automated testing)
- **Higher confidence**: 95% of bugs caught before production

---

**Comparison: Testing Strategies**

| Strategy | Speed | Coverage | Cost | Best For |
|----------|-------|----------|------|----------|
| **DAG Validation Only** | ⚡ Fast (seconds) | Low (syntax only) | Free | Basic checks |
| **Unit Tests** | ⚡ Fast (minutes) | Medium (task logic) | Low | Function validation |
| **Integration Tests** | 🐌 Slow (30+ min) | High (end-to-end) | Medium | Critical pipelines |
| **Production Smoke Tests** | 🐌 Very slow (hours) | Complete | High | Final verification |

---

**Best Practices:**

✅ **DO:**
- Write tests for all custom operators
- Validate DAGs on every commit
- Mock external dependencies in tests
- Use fixtures for common test setup
- Test both success and failure scenarios
- Maintain test data separate from production

❌ **DON'T:**
- Test in production first
- Skip integration tests for critical pipelines
- Hardcode test credentials
- Ignore test failures
- Test only happy path

---

**Interview Talking Point:**

*"At Optum, we implement a comprehensive testing strategy for our 200+ Airflow DAGs. Every DAG goes through automated validation tests on commit, unit tests for task logic with 80%+ coverage, and integration tests for critical pipelines processing claims data. We use pytest with custom fixtures, mock external dependencies like Databricks and Azure storage, and run tests in CI/CD pipelines. This reduced our production incidents by 75% and cut deployment time from 3 days to 1 day. We also maintain separate test datasets with synthetic claims data to ensure realistic testing without exposing PHI."*

---

### Q22: How do you monitor Airflow in production? Explain metrics, alerting, and observability.

**Answer:**

**Monitoring Airflow** = Critical for production stability. Multi-layer observability needed.

**Monitoring Stack:**
1. **Application Metrics** (StatsD/Prometheus)
2. **Logs** (structured logging, centralized)
3. **Alerting** (PagerDuty, Slack, email)
4. **Tracing** (task execution, lineage)
5. **Health Checks** (heartbeat, component status)

---

#### **1. StatsD + Prometheus Metrics**

**Airflow sends metrics to StatsD, Prometheus scrapes them.**

**airflow.cfg configuration:**

```ini
[metrics]
statsd_on = True
statsd_host = localhost
statsd_port = 8125
statsd_prefix = airflow

[scheduler]
statsd_on = True
statsd_host = localhost
statsd_port = 8125
statsd_prefix = airflow
```

**Key Metrics to Track:**

```yaml
# prometheus.yml - Metrics to monitor
metrics:
  # Scheduler Health
  - airflow.scheduler.heartbeat
  - airflow.scheduler_heartbeat
  - airflow.dag_processing.total_parse_time
  - airflow.dag_processing.import_errors

  # Task Execution
  - airflow.dag_run.running
  - airflow.dag_run.success
  - airflow.dag_run.failed
  - airflow.task_instance_created
  - airflow.task_instance_finished

  # Queue Metrics (Celery)
  - airflow.executor.queued_tasks
  - airflow.executor.running_tasks
  - airflow.executor.open_slots

  # Pool Metrics
  - airflow.pool.open_slots
  - airflow.pool.queued_slots
  - airflow.pool.running_slots

  # Database Metrics
  - airflow.db.connection_pool.size
  - airflow.db.connection_pool.overflow
```

---

#### **2. Custom Metrics Collection**

**Emit custom metrics from DAGs:**

```python
from airflow.decorators import dag, task
from airflow.stats import Stats
from datetime import datetime
import time

@dag(start_date=datetime(2024, 1, 1), schedule_interval='@hourly')
def monitored_claims_pipeline():

    @task
    def process_claims():
        start_time = time.time()

        try:
            # Process claims
            claims_processed = 1000
            claims_failed = 5

            # Emit custom metrics
            Stats.gauge('optum.claims.processed', claims_processed)
            Stats.gauge('optum.claims.failed', claims_failed)
            Stats.gauge('optum.claims.success_rate',
                       (claims_processed - claims_failed) / claims_processed * 100)

            # Track processing time
            duration = time.time() - start_time
            Stats.timing('optum.claims.processing_duration', duration)

            # Track data volume
            Stats.incr('optum.claims.total_volume', claims_processed)

            return {'status': 'success', 'count': claims_processed}

        except Exception as e:
            Stats.incr('optum.claims.errors')
            raise

    process_claims()

dag = monitored_claims_pipeline()
```

---

#### **3. Prometheus + Grafana Dashboard**

**Prometheus scrape config:**

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'airflow'
    static_configs:
      - targets: ['localhost:8080']
    metrics_path: '/admin/metrics/'

  - job_name: 'statsd_exporter'
    static_configs:
      - targets: ['localhost:9102']
```

**Grafana Dashboard - Key Panels:**

```json
{
  "dashboard": {
    "title": "Airflow Production Monitoring",
    "panels": [
      {
        "title": "DAG Run Success Rate",
        "targets": [{
          "expr": "rate(airflow_dag_run_success[5m]) / rate(airflow_dag_run_total[5m]) * 100"
        }]
      },
      {
        "title": "Scheduler Heartbeat",
        "targets": [{
          "expr": "airflow_scheduler_heartbeat"
        }],
        "alert": {
          "condition": "no data for 2m",
          "message": "Scheduler not responding!"
        }
      },
      {
        "title": "Task Queue Length",
        "targets": [{
          "expr": "airflow_executor_queued_tasks"
        }],
        "alert": {
          "condition": "> 100 for 10m",
          "message": "Task queue backing up"
        }
      },
      {
        "title": "Claims Processing Volume",
        "targets": [{
          "expr": "rate(optum_claims_processed[1h])"
        }]
      },
      {
        "title": "Average Task Duration by DAG",
        "targets": [{
          "expr": "histogram_quantile(0.95, rate(airflow_task_duration_bucket[5m]))"
        }]
      }
    ]
  }
}
```

---

#### **4. Alerting Rules**

**Prometheus AlertManager rules:**

```yaml
# alerts.yml
groups:
  - name: airflow_critical
    interval: 30s
    rules:
      - alert: SchedulerDown
        expr: absent(airflow_scheduler_heartbeat) or airflow_scheduler_heartbeat == 0
        for: 2m
        labels:
          severity: critical
          team: data-engineering
        annotations:
          summary: "Airflow Scheduler is down"
          description: "Scheduler heartbeat not detected for 2 minutes"

      - alert: HighTaskFailureRate
        expr: rate(airflow_task_instance_failures[10m]) > 0.1
        for: 5m
        labels:
          severity: warning
          team: data-engineering
        annotations:
          summary: "High task failure rate detected"
          description: "Task failure rate {{ $value }} exceeds 10% for 5 minutes"

      - alert: DAGRunDurationHigh
        expr: airflow_dag_run_duration > 7200  # 2 hours in seconds
        for: 10m
        labels:
          severity: warning
          team: data-engineering
        annotations:
          summary: "DAG {{ $labels.dag_id }} running too long"
          description: "DAG run duration {{ $value }}s exceeds 2 hour SLA"

      - alert: TaskQueueBackup
        expr: airflow_executor_queued_tasks > 100
        for: 15m
        labels:
          severity: warning
          team: data-engineering
        annotations:
          summary: "Task queue backing up"
          description: "{{ $value }} tasks queued for 15 minutes"

      - alert: DatabaseConnectionPoolExhausted
        expr: airflow_db_connection_pool_overflow > 10
        for: 5m
        labels:
          severity: critical
          team: data-engineering
        annotations:
          summary: "Database connection pool exhausted"
          description: "Connection pool overflow {{ $value }} connections"
```

**PagerDuty Integration:**

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname', 'severity']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'pagerduty'

  routes:
    - match:
        severity: critical
      receiver: 'pagerduty'
      continue: true

    - match:
        severity: warning
      receiver: 'slack'

receivers:
  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: '<PagerDuty API key>'
        description: '{{ .GroupLabels.alertname }}: {{ .CommonAnnotations.summary }}'

  - name: 'slack'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/XXX'
        channel: '#data-eng-alerts'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ .CommonAnnotations.description }}'
```

---

#### **5. DAG-Level Alerting (Callbacks)**

**Alert on DAG/Task failures:**

```python
from airflow.decorators import dag, task
from airflow.providers.slack.operators.slack_webhook import SlackWebhookOperator
from datetime import datetime

def send_failure_alert(context):
    """Send alert on task failure."""
    task_instance = context['task_instance']
    dag_id = task_instance.dag_id
    task_id = task_instance.task_id
    execution_date = context['execution_date']
    log_url = task_instance.log_url

    message = f"""
    ❌ *Task Failed*
    DAG: {dag_id}
    Task: {task_id}
    Execution Date: {execution_date}
    Logs: {log_url}
    """

    SlackWebhookOperator(
        task_id='slack_alert',
        http_conn_id='slack_webhook',
        message=message,
    ).execute(context)

def send_sla_miss_alert(dag, task_list, blocking_task_list, slas, blocking_tis):
    """Alert when SLA is missed."""
    message = f"SLA MISS in DAG {dag.dag_id}. Tasks: {[t.task_id for t in task_list]}"
    print(message)
    # Send to PagerDuty, Slack, etc.

@dag(
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
    default_args={
        'on_failure_callback': send_failure_alert,
        'sla': timedelta(hours=2),
    },
    sla_miss_callback=send_sla_miss_alert,
    catchup=False,
)
def monitored_dag():

    @task
    def critical_task():
        # Task logic
        pass

    critical_task()

dag = monitored_dag()
```

---

#### **6. Centralized Logging (Azure Log Analytics)**

**Send Airflow logs to Azure:**

```python
# airflow.cfg
[logging]
remote_logging = True
remote_log_conn_id = azure_log_analytics
remote_base_log_folder = wasb://logs@optumairflow.blob.core.windows.net/
```

**Structured logging in tasks:**

```python
import logging
import json
from datetime import datetime

# Structured logger
def get_structured_logger(task_id):
    logger = logging.getLogger(task_id)

    class StructuredFormatter(logging.Formatter):
        def format(self, record):
            log_data = {
                'timestamp': datetime.utcnow().isoformat(),
                'level': record.levelname,
                'task_id': task_id,
                'message': record.getMessage(),
                'dag_id': getattr(record, 'dag_id', None),
                'execution_date': getattr(record, 'execution_date', None),
            }
            return json.dumps(log_data)

    handler = logging.StreamHandler()
    handler.setFormatter(StructuredFormatter())
    logger.addHandler(handler)

    return logger

@task
def process_claims_with_logging():
    logger = get_structured_logger('process_claims')

    logger.info("Starting claims processing", extra={
        'dag_id': 'claims_pipeline',
        'execution_date': datetime(2024, 5, 1),
        'claims_count': 1000
    })

    try:
        # Process claims
        logger.info("Claims processed successfully", extra={'processed': 1000})
    except Exception as e:
        logger.error("Claims processing failed", extra={'error': str(e)})
        raise
```

**Query logs in Azure Log Analytics:**

```kusto
// Find failed tasks in last 24 hours
AirflowLogs
| where TimeGenerated > ago(24h)
| where level == "ERROR"
| where dag_id == "claims_processing_pipeline"
| project TimeGenerated, task_id, message
| order by TimeGenerated desc

// Calculate task success rate
AirflowLogs
| where TimeGenerated > ago(7d)
| where task_id == "process_claims"
| summarize
    total = count(),
    failures = countif(level == "ERROR")
| extend success_rate = (total - failures) * 100.0 / total
```

---

#### **7. Health Check Endpoint**

**Custom health check API:**

```python
# plugins/health_check_plugin.py
from airflow.plugins_manager import AirflowPlugin
from flask import Blueprint, jsonify
from airflow.models import DagRun, TaskInstance
from airflow.utils.state import State
from datetime import datetime, timedelta

health_check_blueprint = Blueprint('health_check', __name__)

@health_check_blueprint.route('/health', methods=['GET'])
def health_check():
    """Health check endpoint for load balancer."""

    # Check scheduler heartbeat
    from airflow.jobs.scheduler_job import SchedulerJob
    last_scheduler_run = SchedulerJob.most_recent_job()

    scheduler_healthy = (
        last_scheduler_run and
        last_scheduler_run.latest_heartbeat > datetime.utcnow() - timedelta(minutes=2)
    )

    # Check recent DAG runs
    recent_dag_runs = DagRun.find(
        execution_start_date=datetime.utcnow() - timedelta(hours=1)
    )

    failed_runs = [dr for dr in recent_dag_runs if dr.state == State.FAILED]
    failure_rate = len(failed_runs) / len(recent_dag_runs) if recent_dag_runs else 0

    # Overall health
    healthy = scheduler_healthy and failure_rate < 0.1

    response = {
        'status': 'healthy' if healthy else 'unhealthy',
        'timestamp': datetime.utcnow().isoformat(),
        'checks': {
            'scheduler': 'ok' if scheduler_healthy else 'down',
            'dag_failure_rate': f'{failure_rate * 100:.2f}%',
        }
    }

    return jsonify(response), 200 if healthy else 503

class HealthCheckPlugin(AirflowPlugin):
    name = "health_check"
    flask_blueprints = [health_check_blueprint]
```

---

### **Your Optum Monitoring Stack:**

**1. Metrics Collection:**
- StatsD exporter sends to Prometheus
- Custom metrics from DAGs (claims volume, processing time, error rates)
- System metrics (CPU, memory, disk) from node exporters

**2. Visualization:**
- Grafana dashboards for real-time monitoring
- 15+ panels tracking scheduler health, task queues, DAG success rates
- Historical trend analysis

**3. Alerting:**
- PagerDuty for critical alerts (scheduler down, high failure rate)
- Slack for warnings (long-running DAGs, queue backup)
- Email for SLA misses

**4. Logging:**
- Centralized in Azure Log Analytics
- Structured JSON logs from all tasks
- 30-day retention, searchable via Kusto

**5. Tracing:**
- Data lineage tracking (claims → transformations → reporting)
- Task dependency visualization
- Execution timeline analysis

**Business Impact:**
- **MTTR reduced by 60%** (30 min → 12 min) - faster incident detection
- **99.5% uptime** for scheduler
- **Proactive issue resolution**: 80% of issues detected before user impact

---

**Comparison: Monitoring Approaches**

| Approach | Coverage | Cost | Complexity | Best For |
|----------|----------|------|------------|----------|
| **Airflow UI Only** | Low | Free | Simple | Development |
| **StatsD + Prometheus** | Medium | Low | Medium | Production |
| **Full Stack (Prometheus + Grafana + Alerts)** | High | Medium | High | Enterprise |
| **APM Tools (Datadog, New Relic)** | Complete | High | Low | Large scale |

---

**Best Practices:**

✅ **DO:**
- Monitor scheduler heartbeat continuously
- Track task queue depth and execution times
- Set up alerts for critical failures
- Use structured logging for searchability
- Monitor database connection pool
- Track business metrics (claims processed, revenue impact)

❌ **DON'T:**
- Rely solely on Airflow UI for monitoring
- Ignore warning alerts
- Set overly sensitive alerts (alert fatigue)
- Forget to monitor executor health
- Skip health check endpoints

---

**Interview Talking Point:**

*"At Optum, we run a comprehensive Airflow monitoring stack processing 10M+ claims daily. We use Prometheus to collect 50+ metrics from Airflow via StatsD, visualize them in Grafana with 15 custom dashboards, and alert via PagerDuty for critical issues and Slack for warnings. We track scheduler heartbeat, task queue depth, DAG success rates, and business metrics like claims processing volume. All logs go to Azure Log Analytics with structured JSON format for easy querying. This reduced our MTTR by 60% (30 min to 12 min) and helped us maintain 99.5% scheduler uptime. We catch 80% of issues proactively before they impact users."*

---

### Q23: Compare Airflow Executors: LocalExecutor, CeleryExecutor, KubernetesExecutor. When to use each?

**Answer:**

**Airflow Executors** = Determine how and where tasks run. Critical choice for production scalability.

**Executor Types:**
1. **SequentialExecutor** (default, dev only)
2. **LocalExecutor** (single machine, multi-process)
3. **CeleryExecutor** (distributed, fixed workers)
4. **KubernetesExecutor** (distributed, dynamic workers)
5. **DaskExecutor** (distributed, analytics workloads)

---

#### **1. SequentialExecutor**

**Description:** Runs one task at a time, single process. Uses SQLite.

**Configuration:**

```ini
# airflow.cfg
[core]
executor = SequentialExecutor

[database]
sql_alchemy_conn = sqlite:////path/to/airflow.db
```

**Characteristics:**
- ❌ No parallelism (one task at a time)
- ❌ SQLite database (not production-ready)
- ✅ Simple setup (good for testing)
- ❌ Cannot scale

**Use Case:**
- Local development only
- Testing DAG logic
- CI/CD pipeline tests

---

#### **2. LocalExecutor**

**Description:** Runs tasks in parallel using multiple processes on a single machine.

**Configuration:**

```ini
# airflow.cfg
[core]
executor = LocalExecutor
parallelism = 32  # Max tasks running at once
dag_concurrency = 16  # Max tasks per DAG

[database]
sql_alchemy_conn = postgresql://user:pass@localhost:5432/airflow
```

**Architecture:**

```
┌─────────────────────────────────────┐
│         Airflow Scheduler           │
│                                     │
│  ┌───────────────────────────────┐ │
│  │   LocalExecutor               │ │
│  │                               │ │
│  │  ┌─────┐ ┌─────┐ ┌─────┐     │ │
│  │  │Task1│ │Task2│ │Task3│ ... │ │
│  │  │(proc│ │(proc│ │(proc│     │ │
│  │  └─────┘ └─────┘ └─────┘     │ │
│  └───────────────────────────────┘ │
└─────────────────────────────────────┘
         │
         ▼
    PostgreSQL/MySQL
```

**Pros:**
- ✅ True parallelism (multi-process)
- ✅ Simple setup (no external queue)
- ✅ Good for small-medium workloads
- ✅ Lower latency than Celery

**Cons:**
- ❌ Limited to single machine resources
- ❌ No horizontal scaling
- ❌ Scheduler and executor on same machine

**Use Case:**
- Small teams (<10 DAGs)
- Moderate workload (<1000 tasks/day)
- Single machine sufficient
- Simpler operational overhead

**Example DAG:**

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(start_date=datetime(2024, 1, 1), schedule_interval='@hourly')
def local_executor_dag():

    @task
    def task_1():
        return "Task 1"

    @task
    def task_2():
        return "Task 2"

    @task
    def task_3():
        return "Task 3"

    # These run in parallel on LocalExecutor
    [task_1(), task_2(), task_3()]

dag = local_executor_dag()
```

---

#### **3. CeleryExecutor**

**Description:** Distributed executor using Celery. Tasks run on separate worker machines.

**Configuration:**

```ini
# airflow.cfg
[core]
executor = CeleryExecutor
parallelism = 128

[celery]
broker_url = redis://redis-host:6379/0
result_backend = db+postgresql://user:pass@postgres-host:5432/airflow
worker_concurrency = 16  # Tasks per worker
```

**Architecture:**

```
┌─────────────────┐
│    Scheduler    │
│                 │
│ CeleryExecutor  │
└────────┬────────┘
         │
         ▼
    ┌─────────┐
    │  Redis  │  (Message Queue)
    │ (Broker)│
    └────┬────┘
         │
    ┌────┴────────────────────────┐
    │                             │
    ▼                             ▼
┌─────────┐                  ┌─────────┐
│Worker 1 │                  │Worker 2 │
│ ┌─────┐ │                  │ ┌─────┐ │
│ │Task1│ │                  │ │Task3│ │
│ │Task2│ │                  │ │Task4│ │
│ └─────┘ │                  │ └─────┘ │
└─────────┘                  └─────────┘
```

**Setup Celery Workers:**

```bash
# Start Celery worker
airflow celery worker \
  --concurrency 16 \
  --queues default,high_priority,data_processing

# Start Flower (monitoring UI)
airflow celery flower
```

**Queue Management:**

```python
from airflow.decorators import dag, task
from airflow.operators.python import PythonOperator
from datetime import datetime

@dag(start_date=datetime(2024, 1, 1), schedule_interval='@hourly')
def celery_queue_dag():

    # Default queue
    @task
    def normal_task():
        return "Normal priority"

    # High priority queue
    high_priority_task = PythonOperator(
        task_id='urgent_task',
        python_callable=lambda: print("Urgent!"),
        queue='high_priority',  # Route to specific worker
    )

    # Heavy computation queue
    compute_task = PythonOperator(
        task_id='compute_task',
        python_callable=lambda: print("Heavy compute"),
        queue='data_processing',  # Workers with more CPU
    )

    normal_task() >> high_priority_task >> compute_task

dag = celery_queue_dag()
```

**Pros:**
- ✅ Horizontal scaling (add more workers)
- ✅ Queue-based task routing
- ✅ Worker specialization (GPU workers, CPU workers)
- ✅ High availability (worker redundancy)

**Cons:**
- ❌ Complex setup (Redis/RabbitMQ, multiple workers)
- ❌ Higher latency (queue overhead)
- ❌ Fixed worker pools (always running)
- ❌ Resource waste if idle

**Use Case:**
- Medium-large teams (10-100 DAGs)
- High workload (10K+ tasks/day)
- Need queue-based routing
- Long-running tasks

---

#### **4. KubernetesExecutor**

**Description:** Each task runs in its own Kubernetes pod. Dynamic resource allocation.

**Configuration:**

```ini
# airflow.cfg
[core]
executor = KubernetesExecutor

[kubernetes]
namespace = airflow
worker_container_repository = apache/airflow
worker_container_tag = 2.8.0
delete_worker_pods = True
delete_worker_pods_on_failure = False
```

**Architecture:**

```
┌──────────────────┐
│   Scheduler      │
│KubernetesExecutor│
└────────┬─────────┘
         │
         ▼
┌────────────────────────────────────┐
│      Kubernetes Cluster (AKS)      │
│                                    │
│  ┌──────┐  ┌──────┐  ┌──────┐    │
│  │ Pod1 │  │ Pod2 │  │ Pod3 │    │
│  │Task1 │  │Task2 │  │Task3 │    │
│  │      │  │      │  │      │    │
│  │2 CPU │  │4 CPU │  │8 CPU │    │
│  │4GB   │  │8GB   │  │16GB  │    │
│  └──────┘  └──────┘  └──────┘    │
│                                    │
│  (Pods created/destroyed on-demand)│
└────────────────────────────────────┘
```

**Pod Template:**

```yaml
# pod_template.yaml
apiVersion: v1
kind: Pod
metadata:
  name: airflow-worker
  namespace: airflow
spec:
  serviceAccountName: airflow-worker
  restartPolicy: Never
  containers:
    - name: base
      image: apache/airflow:2.8.0
      resources:
        requests:
          memory: "2Gi"
          cpu: "1000m"
        limits:
          memory: "4Gi"
          cpu: "2000m"
      env:
        - name: AIRFLOW__CORE__EXECUTOR
          value: "LocalExecutor"
      volumeMounts:
        - name: dags
          mountPath: /opt/airflow/dags
        - name: logs
          mountPath: /opt/airflow/logs
  volumes:
    - name: dags
      persistentVolumeClaim:
        claimName: airflow-dags
    - name: logs
      persistentVolumeClaim:
        claimName: airflow-logs
```

**Custom Pod per Task:**

```python
from airflow.decorators import dag, task
from airflow.providers.cncf.kubernetes.operators.kubernetes_pod import KubernetesPodOperator
from kubernetes.client import models as k8s
from datetime import datetime

@dag(start_date=datetime(2024, 1, 1), schedule_interval='@daily')
def k8s_executor_dag():

    # Light task (small pod)
    light_task = KubernetesPodOperator(
        task_id='light_task',
        name='light-task-pod',
        namespace='airflow',
        image='apache/airflow:2.8.0',
        cmds=['python', '-c'],
        arguments=['print("Light task")'],
        resources=k8s.V1ResourceRequirements(
            requests={'memory': '512Mi', 'cpu': '500m'},
            limits={'memory': '1Gi', 'cpu': '1000m'},
        ),
    )

    # Heavy task (large pod with GPU)
    heavy_task = KubernetesPodOperator(
        task_id='heavy_ml_task',
        name='ml-training-pod',
        namespace='airflow',
        image='tensorflow/tensorflow:latest-gpu',
        cmds=['python', 'train_model.py'],
        resources=k8s.V1ResourceRequirements(
            requests={'memory': '16Gi', 'cpu': '4000m', 'nvidia.com/gpu': '1'},
            limits={'memory': '32Gi', 'cpu': '8000m', 'nvidia.com/gpu': '1'},
        ),
        node_selector={'gpu': 'true'},  # Schedule on GPU nodes
    )

    light_task >> heavy_task

dag = k8s_executor_dag()
```

**Pros:**
- ✅ Dynamic scaling (pods created on-demand)
- ✅ Task-level resource isolation
- ✅ Custom resources per task (GPU, memory)
- ✅ No idle resource waste
- ✅ Kubernetes native (auto-healing, load balancing)

**Cons:**
- ❌ Pod startup latency (10-30s per task)
- ❌ Requires Kubernetes expertise
- ❌ Higher complexity
- ❌ Cost (K8s cluster overhead)

**Use Case:**
- Large teams (100+ DAGs)
- Variable workloads (bursty)
- Need resource isolation
- Tasks with different resource needs (GPU, high-memory)
- Cloud-native deployments (AKS, GKE, EKS)

---

### **Executor Comparison Table**

| Feature | Sequential | Local | Celery | Kubernetes |
|---------|------------|-------|--------|------------|
| **Parallelism** | ❌ No | ✅ Yes (single machine) | ✅ Yes (distributed) | ✅ Yes (distributed) |
| **Scalability** | ❌ None | ❌ Vertical only | ✅ Horizontal | ✅ Elastic |
| **Setup Complexity** | ⭐ Simple | ⭐⭐ Moderate | ⭐⭐⭐⭐ Complex | ⭐⭐⭐⭐⭐ Very Complex |
| **Task Startup** | Instant | Instant | ~1-2s | ~10-30s |
| **Resource Isolation** | ❌ No | ❌ No | ⚠️ Process-level | ✅ Pod-level |
| **Cost** | Free | Low | Medium | High |
| **Best For** | Dev/Test | Small teams | Medium teams | Large scale |
| **Max Tasks/Day** | <100 | <1K | <50K | Unlimited |

---

### **Your Optum Executor Journey:**

**Phase 1 (Early 2023):** LocalExecutor
- 10 DAGs processing claims data
- Single EC2 instance (32 cores, 64GB RAM)
- Worked well initially, but hit limits at 50 DAGs

**Phase 2 (Mid 2023):** CeleryExecutor
- Migrated to Celery with 10 worker nodes
- Redis for message broker
- Separate queues: `default`, `high_priority`, `long_running`
- Scaled to 150 DAGs, 5K tasks/day

**Phase 3 (Late 2024):** KubernetesExecutor on AKS
- Full K8s deployment with dynamic pods
- Task-level resource allocation (some tasks need 64GB RAM, others need 1GB)
- Auto-scaling based on workload
- Currently: 200+ DAGs, 10K+ tasks/day, 10M+ claims processed daily

**Business Impact:**
- **Cost savings: 40%** (pay only for pods in use vs fixed Celery workers)
- **Faster processing**: Heavy ML tasks get GPU pods, light tasks get small pods
- **Better isolation**: Failed task doesn't impact others

---

**Decision Framework:**

```
Start with Local → Growing pains? → Celery → Need dynamic scaling + K8s? → Kubernetes

Use Local if:
- <20 DAGs
- <1K tasks/day
- Single machine sufficient

Use Celery if:
- 20-100 DAGs
- 1K-10K tasks/day
- Need fixed worker pools
- Don't have Kubernetes

Use Kubernetes if:
- 100+ DAGs
- 10K+ tasks/day
- Variable resource needs
- Already on Kubernetes
- Bursty workloads
```

---

**Interview Talking Point:**

*"At Optum, we evolved our Airflow executor strategy as we scaled. We started with LocalExecutor for our initial 10 DAGs on a single EC2 instance, migrated to CeleryExecutor with 10 worker nodes and Redis when we hit 50 DAGs, and finally moved to KubernetesExecutor on AKS when we reached 150 DAGs. The K8s executor allows us to dynamically allocate resources—light tasks get 1GB pods, heavy ML training gets 64GB pods with GPUs. This reduced costs by 40% compared to fixed Celery workers and improved processing time for variable workloads. We now run 200+ DAGs processing 10M+ claims daily with auto-scaling based on demand."*

---

### Q24: Explain backfilling and catchup in Airflow. How do you reprocess historical data?

**Answer:**

**Backfilling** = Running DAGs for historical dates to reprocess past data.
**Catchup** = Automatically run missed DAG runs when DAG is unpaused or scheduler was down.

**Use Cases:**
- Reprocess data after bug fix
- Fill gaps from outages
- Load historical data for new feature
- Recalculate metrics with updated logic

---

#### **1. Catchup Behavior**

**Default:** `catchup = True` runs all missed DAG runs since `start_date`.

```python
from airflow.decorators import dag, task
from datetime import datetime, timedelta

# BAD: catchup=True with old start_date
@dag(
    start_date=datetime(2023, 1, 1),  # 500+ days ago!
    schedule_interval='@daily',
    catchup=True,  # Will create 500+ DAG runs!
)
def dangerous_catchup_dag():
    @task
    def process_data():
        print("Processing...")

    process_data()

# GOOD: catchup=False for most use cases
@dag(
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
    catchup=False,  # Only run current + future dates
)
def safe_dag():
    @task
    def process_data():
        print("Processing...")

    process_data()
```

**When to use `catchup=True`:**
- New DAG that needs historical data
- Idempotent tasks (can run multiple times safely)
- Data pipeline where every date must be processed

**When to use `catchup=False`:**
- Most production DAGs
- Non-idempotent tasks
- Latest data only (no historical need)

---

#### **2. Backfilling with CLI**

**Backfill specific date range:**

```bash
# Backfill Jan 2024 for claims pipeline
airflow dags backfill \
  --start-date 2024-01-01 \
  --end-date 2024-01-31 \
  --reset-dagruns \
  claims_processing_pipeline

# Backfill with rerun (clear existing runs)
airflow dags backfill \
  --start-date 2024-05-01 \
  --end-date 2024-05-07 \
  --rerun-failed-tasks \
  --reset-dagruns \
  claims_processing_pipeline

# Backfill specific tasks only
airflow tasks backfill \
  claims_processing_pipeline \
  extract_claims \
  --start-date 2024-05-01 \
  --end-date 2024-05-07

# Dry-run (check what would be backfilled)
airflow dags backfill \
  --start-date 2024-05-01 \
  --end-date 2024-05-07 \
  --dry-run \
  claims_processing_pipeline
```

**Key flags:**
- `--reset-dagruns`: Clear existing DAG runs (use for reprocessing)
- `--rerun-failed-tasks`: Rerun only failed tasks
- `--mark-success`: Mark as success without running (data already exists)
- `--dry-run`: Preview what would run
- `--local`: Run locally (no executor, for testing)

---

#### **3. Programmatic Backfill**

**Trigger backfill from Python:**

```python
from airflow import DAG
from airflow.operators.trigger_dagrun import TriggerDagRunOperator
from airflow.decorators import task
from datetime import datetime, timedelta

def backfill_claims_for_range(start_date, end_date):
    """Trigger backfill for date range."""
    from airflow.models import DagRun, DagBag
    from airflow.utils.state import State

    dag_bag = DagBag()
    dag = dag_bag.get_dag('claims_processing_pipeline')

    current_date = start_date
    while current_date <= end_date:
        # Clear existing run if exists
        existing_run = DagRun.find(
            dag_id='claims_processing_pipeline',
            execution_date=current_date
        )
        if existing_run:
            existing_run[0].set_state(State.NONE)

        # Trigger new run
        dag.create_dagrun(
            run_id=f'backfill_{current_date.strftime("%Y%m%d")}',
            execution_date=current_date,
            state=State.RUNNING,
            conf={'backfill': True},
        )

        current_date += timedelta(days=1)

# DAG to orchestrate backfill
@dag(start_date=datetime(2024, 1, 1), schedule_interval=None)
def backfill_orchestrator():

    @task
    def trigger_backfill():
        backfill_claims_for_range(
            start_date=datetime(2024, 1, 1),
            end_date=datetime(2024, 1, 31)
        )

    trigger_backfill()

dag = backfill_orchestrator()
```

---

#### **4. Idempotent Backfill Pattern**

**Ensure tasks can be rerun safely:**

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(start_date=datetime(2024, 1, 1), schedule_interval='@daily')
def idempotent_claims_dag():

    @task
    def extract_claims(execution_date):
        """Extract claims for specific date - idempotent."""
        from delta.tables import DeltaTable

        # Idempotent: Delete existing data for this date first
        delta_table = DeltaTable.forPath(spark, "s3://bucket/claims_raw")
        delta_table.delete(f"processing_date = '{execution_date}'")

        # Extract new data
        claims = fetch_claims_from_api(execution_date)

        # Write with merge (upsert)
        claims.write.format("delta").mode("append").save("s3://bucket/claims_raw")

    @task
    def transform_claims(execution_date):
        """Transform claims - idempotent with merge."""
        from delta.tables import DeltaTable

        # Read raw claims
        claims = spark.read.format("delta").load("s3://bucket/claims_raw") \
            .filter(f"processing_date = '{execution_date}'")

        # Transform
        transformed = claims.withColumn("total_amount", col("amount") * col("quantity"))

        # Upsert to target (idempotent)
        target = DeltaTable.forPath(spark, "s3://bucket/claims_transformed")
        target.alias("target").merge(
            transformed.alias("source"),
            "target.claim_id = source.claim_id AND target.processing_date = source.processing_date"
        ).whenMatchedUpdateAll() \
         .whenNotMatchedInsertAll() \
         .execute()

    extract_claims() >> transform_claims()

dag = idempotent_claims_dag()
```

**Idempotent patterns:**
- ✅ Delete-then-insert (for specific partition)
- ✅ Upsert/Merge (update if exists, insert if not)
- ✅ Overwrite partition (Spark/Delta)
- ❌ Append-only (not idempotent!)

---

#### **5. Backfill with Dependencies**

**Handle DAGs that depend on each other:**

```python
from airflow.decorators import dag, task
from airflow.sensors.external_task import ExternalTaskSensor
from datetime import datetime, timedelta

# Upstream DAG (must backfill first)
@dag(start_date=datetime(2024, 1, 1), schedule_interval='@daily', catchup=False)
def upstream_claims_dag():
    @task
    def extract_claims():
        return "Claims extracted"

    extract_claims()

upstream_dag = upstream_claims_dag()

# Downstream DAG (depends on upstream)
@dag(start_date=datetime(2024, 1, 1), schedule_interval='@daily', catchup=False)
def downstream_reporting_dag():

    # Wait for upstream DAG to complete
    wait_for_claims = ExternalTaskSensor(
        task_id='wait_for_claims',
        external_dag_id='upstream_claims_dag',
        external_task_id='extract_claims',
        execution_delta=timedelta(hours=0),  # Same execution_date
        timeout=3600,
        mode='poke',
    )

    @task
    def generate_report():
        return "Report generated"

    wait_for_claims >> generate_report()

downstream_dag = downstream_reporting_dag()
```

**Backfill order:**
1. Backfill upstream DAG first
2. Wait for completion
3. Backfill downstream DAG

```bash
# Step 1: Backfill upstream
airflow dags backfill upstream_claims_dag \
  --start-date 2024-01-01 --end-date 2024-01-31

# Step 2: After upstream completes, backfill downstream
airflow dags backfill downstream_reporting_dag \
  --start-date 2024-01-01 --end-date 2024-01-31
```

---

#### **6. Partial Backfill (Specific Tasks)**

**Backfill only specific tasks in a DAG:**

```bash
# Only rerun transform + load, skip extract (data already extracted)
airflow tasks clear \
  claims_processing_pipeline \
  --start-date 2024-05-01 \
  --end-date 2024-05-07 \
  --task-regex "transform_.*|load_.*" \
  --yes

# Mark extract as success (skip it)
airflow tasks states-for-dag-run \
  claims_processing_pipeline \
  2024-05-01T00:00:00+00:00 \
  --mark-success \
  --task-regex "extract_.*"
```

---

#### **7. Backfill Monitoring**

**Monitor backfill progress:**

```python
from airflow.models import DagRun, TaskInstance
from airflow.utils.state import State

def check_backfill_progress(dag_id, start_date, end_date):
    """Check backfill completion status."""
    dag_runs = DagRun.find(
        dag_id=dag_id,
        execution_start_date=start_date,
        execution_end_date=end_date
    )

    total = len(dag_runs)
    success = len([dr for dr in dag_runs if dr.state == State.SUCCESS])
    failed = len([dr for dr in dag_runs if dr.state == State.FAILED])
    running = len([dr for dr in dag_runs if dr.state == State.RUNNING])

    print(f"Backfill Progress for {dag_id}:")
    print(f"  Total: {total}")
    print(f"  Success: {success} ({success/total*100:.1f}%)")
    print(f"  Failed: {failed}")
    print(f"  Running: {running}")
    print(f"  Pending: {total - success - failed - running}")

check_backfill_progress(
    'claims_processing_pipeline',
    datetime(2024, 1, 1),
    datetime(2024, 1, 31)
)
```

---

### **Your Optum Backfill Example:**

**Scenario:** Claims calculation bug found. Need to reprocess 6 months of claims data.

**Challenge:**
- 180 days of data
- 10M claims per day = 1.8B total claims
- Must maintain system stability (not overwhelm cluster)

**Solution:**

```python
from airflow.decorators import dag, task
from datetime import datetime, timedelta

@dag(
    start_date=datetime(2024, 1, 1),
    schedule_interval=None,  # Manual trigger
    catchup=False
)
def controlled_backfill_dag():
    """Backfill with rate limiting and monitoring."""

    @task
    def backfill_batch(batch_start, batch_end):
        """Backfill one week at a time."""
        import subprocess

        result = subprocess.run([
            'airflow', 'dags', 'backfill',
            'claims_processing_pipeline',
            '--start-date', batch_start.strftime('%Y-%m-%d'),
            '--end-date', batch_end.strftime('%Y-%m-%d'),
            '--reset-dagruns',
        ], capture_output=True, text=True)

        if result.returncode != 0:
            raise Exception(f"Backfill failed: {result.stderr}")

        return f"Completed {batch_start} to {batch_end}"

    # Backfill in weekly batches
    start = datetime(2023, 11, 1)
    end = datetime(2024, 5, 1)

    previous_task = None
    current = start

    while current < end:
        batch_end = min(current + timedelta(days=7), end)

        batch_task = backfill_batch.override(
            task_id=f'backfill_{current.strftime("%Y%m%d")}'
        )(current, batch_end)

        if previous_task:
            previous_task >> batch_task  # Sequential batches

        previous_task = batch_task
        current = batch_end

dag = controlled_backfill_dag()
```

**Results:**
- Backfilled 180 days in 26 batches (7 days each)
- Total time: 3 days
- Zero downtime for production pipelines
- Fixed 1.8B claim records

**Business Impact:**
- **Revenue correction:** $15M in claims properly adjudicated
- **Compliance:** Audit trail maintained for all corrections
- **Customer satisfaction:** Members' claims accurately reflected

---

**Comparison: Backfill Strategies**

| Strategy | Speed | Risk | Control | Best For |
|----------|-------|------|---------|----------|
| **catchup=True** | Fast | High (resource spike) | Low | New DAGs only |
| **CLI backfill (full range)** | Fast | Medium | Medium | Small date ranges |
| **Batched backfill** | Slow | Low | High | Large date ranges |
| **Partial task backfill** | Fast | Low | High | Specific task fixes |

---

**Best Practices:**

✅ **DO:**
- Use `catchup=False` for most DAGs
- Backfill in batches for large ranges
- Make tasks idempotent (safe to rerun)
- Test backfill on small range first
- Monitor resource usage during backfill
- Document backfill reasons (audit trail)

❌ **DON'T:**
- Set `catchup=True` with old `start_date`
- Backfill without testing idempotency
- Overwhelm cluster with large backfills
- Forget to clear existing runs (`--reset-dagruns`)
- Backfill during peak hours

---

**Interview Talking Point:**

*"At Optum, we handle backfills carefully given our scale of 10M+ claims daily. When we found a calculation bug affecting 6 months of claims, we implemented a controlled backfill strategy processing 1.8B records in weekly batches over 3 days. We used idempotent Delta Lake merges to ensure safe reprocessing, monitored resource usage closely, and sequenced batches to avoid overwhelming our Databricks cluster. Generally, we set catchup=False for all production DAGs and use explicit CLI backfills when needed. This backfill corrected $15M in claims adjudication and maintained our audit compliance while keeping production systems stable."*

---

### Q25: How do you optimize Airflow performance? Explain scheduler tuning, parallelism, and pool management.

**Answer:**

**Airflow Performance Optimization** = Critical for scale. Multiple tuning dimensions.

**Optimization Areas:**
1. **Scheduler tuning** (parsing, heartbeat, task creation)
2. **Parallelism settings** (max concurrent tasks)
3. **Pool management** (resource quotas)
4. **Database optimization** (connection pool, queries)
5. **DAG design** (task granularity, dependencies)

---

#### **1. Scheduler Tuning**

**The scheduler is the heart of Airflow. It must:**
- Parse DAG files (discover tasks)
- Create task instances
- Schedule tasks to executors
- Update task states

**Key airflow.cfg Settings:**

```ini
# airflow.cfg

[scheduler]
# How often to scan DAGs folder (seconds)
dag_dir_list_interval = 300  # 5 minutes (default)
# Reduce for faster DAG discovery in dev, increase in prod

# Max threads for DAG file processing
max_threads = 4  # Increase if you have many DAGs

# How many DAG files to process in parallel
parsing_processes = 4  # Increase with more CPUs

# Min interval between DAG file parsing (seconds)
min_file_process_interval = 30  # Reduce to update DAGs faster

# Scheduler heartbeat interval (seconds)
scheduler_heartbeat_sec = 5  # Default, don't change unless needed

# Max task instances scheduler creates per loop
max_tis_per_query = 512  # Increase if scheduler is slow

# How often scheduler should run (seconds)
scheduler_loop_delay = 1  # Default, rarely needs changing

# Task queuing strategy
task_queuing_strategy = priority_weight
```

**Scheduler Performance Monitoring:**

```python
# Monitor scheduler metrics
from airflow.stats import Stats

# Track scheduler loop duration
Stats.timing('scheduler.scheduler_loop_duration', duration_ms)

# Track DAG processing time
Stats.timing('dag_processing.total_parse_time', parse_time_ms)
```

---

#### **2. Parallelism Settings**

**Hierarchy of parallelism controls:**

```
Global Parallelism (airflow.cfg)
    ↓
DAG-level concurrency
    ↓
Task-level concurrency
    ↓
Pool slots (shared resource limits)
```

**airflow.cfg Global Settings:**

```ini
[core]
# Max tasks running across entire Airflow instance
parallelism = 128  # Increase based on executor capacity

# Max tasks running per DAG run
dag_concurrency = 16  # Default, increase for big DAGs

# Max active DAG runs per DAG
max_active_runs_per_dag = 16  # Increase if catchup=True

[celery]
# For CeleryExecutor: tasks per worker
worker_concurrency = 16  # Match to worker CPU cores
```

**DAG-Level Concurrency:**

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(
    start_date=datetime(2024, 1, 1),
    schedule_interval='@hourly',
    max_active_runs=3,  # Max 3 concurrent runs of this DAG
    max_active_tasks=10,  # Max 10 tasks running at once in this DAG
    catchup=False
)
def optimized_claims_dag():

    @task
    def process_claims():
        pass

    process_claims()

dag = optimized_claims_dag()
```

**Task-Level Concurrency:**

```python
from airflow.models import Pool
from airflow.operators.python import PythonOperator

# Limit concurrent instances of same task
task_with_limit = PythonOperator(
    task_id='api_call',
    python_callable=call_external_api,
    max_active_tis_per_dag=2,  # Max 2 instances of this task across all DAG runs
)
```

---

#### **3. Pool Management**

**Pools** = Shared resource limits (database connections, API quotas, etc.)

**Create Pools via CLI:**

```bash
# Create pool for database queries (limit concurrent DB connections)
airflow pools set postgres_pool 10 "PostgreSQL connection pool"

# Create pool for external API calls (rate limit)
airflow pools set external_api_pool 5 "External API rate limit"

# Create pool for heavy compute tasks
airflow pools set compute_pool 20 "Databricks cluster slots"

# List pools
airflow pools list

# Delete pool
airflow pools delete postgres_pool
```

**Use Pools in Tasks:**

```python
from airflow.decorators import dag, task
from airflow.operators.python import PythonOperator
from datetime import datetime

@dag(start_date=datetime(2024, 1, 1), schedule_interval='@hourly')
def pooled_dag():

    # Task using database pool
    query_task = PythonOperator(
        task_id='query_claims',
        python_callable=query_postgres,
        pool='postgres_pool',  # Uses 1 slot from postgres_pool
        pool_slots=1,  # Can use multiple slots if needed
    )

    # Task using API pool
    api_task = PythonOperator(
        task_id='call_external_api',
        python_callable=call_api,
        pool='external_api_pool',
        pool_slots=1,
    )

    # Heavy compute task using 2 slots
    compute_task = PythonOperator(
        task_id='train_ml_model',
        python_callable=train_model,
        pool='compute_pool',
        pool_slots=2,  # Uses 2 slots (needs more resources)
    )

    query_task >> api_task >> compute_task

dag = pooled_dag()
```

**Dynamic Pool Management:**

```python
# Adjust pool size based on load
from airflow.models import Pool
from airflow.utils.session import create_session

def adjust_pool_size(pool_name, new_size):
    """Dynamically adjust pool size."""
    with create_session() as session:
        pool = session.query(Pool).filter(Pool.pool == pool_name).first()
        if pool:
            pool.slots = new_size
            session.commit()
            print(f"Pool {pool_name} resized to {new_size}")

# Increase pool during business hours
adjust_pool_size('compute_pool', 50)  # Day

# Decrease pool during off-hours
adjust_pool_size('compute_pool', 10)  # Night
```

---

#### **4. Database Optimization**

**Airflow metadata database is critical for performance.**

**Connection Pool Settings:**

```ini
# airflow.cfg
[database]
sql_alchemy_conn = postgresql://user:pass@host:5432/airflow

# Connection pool size (adjust based on load)
sql_alchemy_pool_size = 10  # Default 5, increase for high load

# Max overflow connections
sql_alchemy_max_overflow = 20  # Default 10

# Connection pool recycle time (seconds)
sql_alchemy_pool_recycle = 3600  # Close idle connections after 1 hour

# Enable connection pre-ping (check connection before use)
sql_alchemy_pool_pre_ping = True
```

**Database Maintenance:**

```bash
# Clean old DAG run metadata (keeps DB small)
airflow db clean \
  --clean-before-timestamp "2024-01-01 00:00:00" \
  --table-names dag_run,task_instance,log,xcom,job \
  --yes

# Vacuum PostgreSQL (reclaim space)
psql -d airflow -c "VACUUM FULL;"

# Reindex for performance
psql -d airflow -c "REINDEX DATABASE airflow;"
```

**Query Optimization:**

```sql
-- Add indexes for common queries
CREATE INDEX idx_task_instance_dag_run ON task_instance(dag_id, run_id);
CREATE INDEX idx_dag_run_state ON dag_run(dag_id, state);
CREATE INDEX idx_task_instance_state ON task_instance(state);
```

---

#### **5. DAG Design Optimization**

**Optimize DAG structure for performance:**

**BAD: Too many tiny tasks**

```python
# Anti-pattern: 100 tasks for 100 files
@dag(start_date=datetime(2024, 1, 1), schedule_interval='@daily')
def too_many_tasks_dag():
    for i in range(100):
        @task(task_id=f'process_file_{i}')
        def process_file():
            pass

        process_file()  # Creates 100 task instances!

# Scheduler overhead: 100 task instances per DAG run
```

**GOOD: Batch processing**

```python
# Better: Single task processes all files
@dag(start_date=datetime(2024, 1, 1), schedule_interval='@daily')
def batched_dag():
    @task
    def process_all_files():
        files = list_files()  # 100 files
        for file in files:
            process(file)  # Process in loop within task

    process_all_files()  # Only 1 task instance

# Scheduler overhead: 1 task instance per DAG run
```

**BEST: Dynamic task mapping (Airflow 2.3+)**

```python
# Best: Dynamic mapping with parallelism
@dag(start_date=datetime(2024, 1, 1), schedule_interval='@daily')
def dynamic_mapping_dag():
    @task
    def get_files():
        return ['file1.csv', 'file2.csv', ..., 'file100.csv']

    @task(max_active_tis_per_dag=10)  # Limit parallelism
    def process_file(filename):
        process(filename)

    files = get_files()
    process_file.expand(filename=files)  # Creates 100 mapped tasks

# Scheduler creates mapped tasks efficiently
# Parallelism controlled by max_active_tis_per_dag
```

**Reduce dependencies for parallelism:**

```python
# BAD: Sequential (slow)
task1 >> task2 >> task3 >> task4 >> task5  # 5 steps in series

# GOOD: Parallel (fast)
task1 >> [task2, task3, task4, task5]  # 1 + 4 parallel steps
```

---

#### **6. Executor-Specific Optimizations**

**LocalExecutor:**

```ini
[core]
parallelism = 32  # Match to CPU cores
```

**CeleryExecutor:**

```ini
[celery]
worker_concurrency = 16  # Tasks per worker
worker_prefetch_multiplier = 1  # Reduce to distribute tasks evenly
worker_max_tasks_per_child = 1000  # Restart worker after N tasks (prevent memory leaks)
```

**KubernetesExecutor:**

```ini
[kubernetes]
worker_pods_creation_batch_size = 10  # Create pods in batches
delete_worker_pods = True  # Clean up completed pods
delete_worker_pods_on_failure = False  # Keep failed pods for debugging
```

---

### **Your Optum Performance Optimization Journey:**

**Problem (Early 2024):**
- 200 DAGs taking 5+ minutes to schedule
- Task queue backup (1000+ tasks queued)
- Scheduler CPU at 90%

**Solution:**

1. **Scheduler Tuning:**
   - Increased `parsing_processes` from 2 to 8
   - Increased `max_threads` from 2 to 8
   - Set `dag_dir_list_interval` to 600 (10 min) - DAGs don't change often

2. **Parallelism Increase:**
   - Global `parallelism`: 32 → 256
   - CeleryExecutor `worker_concurrency`: 8 → 16
   - Added 10 more Celery workers (5 → 15)

3. **Pool Management:**
   - Created `databricks_pool` with 50 slots (limit cluster jobs)
   - Created `postgres_pool` with 20 slots (limit DB connections)
   - Created `api_pool` with 10 slots (rate limit external APIs)

4. **DAG Refactoring:**
   - Consolidated 50 tiny DAGs into 10 larger DAGs
   - Used dynamic task mapping for file processing
   - Reduced unnecessary task dependencies

5. **Database Optimization:**
   - Increased connection pool: 5 → 20
   - Added indexes on `task_instance(state)` and `dag_run(state)`
   - Automated weekly DB cleanup (remove runs older than 90 days)

**Results:**
- **Scheduler latency:** 5 min → 30 sec (90% improvement)
- **Task queue:** 1000+ → <50 tasks
- **Scheduler CPU:** 90% → 40%
- **DAG parse time:** 2 min → 20 sec

**Business Impact:**
- **Faster pipelines:** Claims processing SLA improved from 4 hours to 2 hours
- **Higher throughput:** 5K tasks/day → 15K tasks/day
- **Cost savings:** More efficient resource usage

---

**Performance Tuning Checklist:**

```
☐ Scheduler
  ☐ Increase parsing_processes (match CPU cores)
  ☐ Increase max_threads
  ☐ Adjust dag_dir_list_interval (higher if DAGs stable)

☐ Parallelism
  ☐ Increase global parallelism
  ☐ Increase worker_concurrency (CeleryExecutor)
  ☐ Set appropriate DAG-level concurrency

☐ Pools
  ☐ Create pools for shared resources
  ☐ Set appropriate pool sizes

☐ Database
  ☐ Increase connection pool size
  ☐ Add indexes on key columns
  ☐ Regular DB cleanup (remove old runs)

☐ DAG Design
  ☐ Batch small tasks
  ☐ Use dynamic task mapping
  ☐ Reduce unnecessary dependencies
  ☐ Set catchup=False

☐ Monitoring
  ☐ Track scheduler loop duration
  ☐ Monitor task queue length
  ☐ Alert on long parse times
```

---

**Comparison: Optimization Impact**

| Optimization | Effort | Impact | Risk |
|--------------|--------|--------|------|
| **Increase parallelism** | Low | High | Low |
| **Pool management** | Medium | Medium | Low |
| **Scheduler tuning** | Low | High | Low |
| **Database optimization** | High | High | Medium |
| **DAG refactoring** | High | Very High | Medium |

---

**Best Practices:**

✅ **DO:**
- Start with scheduler tuning (easy wins)
- Use pools for shared resources
- Monitor scheduler metrics continuously
- Clean old DAG run data regularly
- Batch small tasks into larger ones
- Set catchup=False for most DAGs

❌ **DON'T:**
- Over-parallelize (causes resource contention)
- Create 100s of tiny tasks per DAG
- Ignore database maintenance
- Skip monitoring scheduler health
- Use default settings in production

---

**Interview Talking Point:**

*"At Optum, we optimized Airflow to handle 200+ DAGs processing 10M+ claims daily. When we hit scaling issues with 5-minute scheduler latency and 1000+ queued tasks, we took a multi-pronged approach: increased scheduler parsing processes from 2 to 8, raised global parallelism from 32 to 256, added 10 Celery workers, created resource pools for Databricks (50 slots), PostgreSQL (20 slots), and external APIs (10 slots), and refactored DAGs to use dynamic task mapping instead of hundreds of tiny tasks. We also optimized the metadata database with indexes and automated cleanup. This reduced scheduler latency by 90% (5 min to 30 sec), cut task queue from 1000+ to under 50, and improved our claims processing SLA from 4 hours to 2 hours, enabling 15K tasks/day throughput."*

---

### Q26: What are Dynamic DAGs? How do you generate tasks dynamically?

**Answer:**

**Dynamic DAGs** = Programmatically generate DAGs/tasks based on configuration.

**Use Cases:**
- Process multiple files/partitions
- Create DAGs per customer/region
- Generate tasks from database query

**Method 1: Loop to create tasks:**

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def process_partition(partition_name):
    print(f"Processing {partition_name}")

with DAG('dynamic_tasks', start_date=datetime(2024, 1, 1), schedule_interval='@daily') as dag:
    # List of partitions
    partitions = ['2024-05-01', '2024-05-02', '2024-05-03']

    # Dynamically create tasks
    for partition in partitions:
        task = PythonOperator(
            task_id=f'process_{partition.replace("-", "_")}',
            python_callable=process_partition,
            op_args=[partition],
        )
```

**Method 2: Generate entire DAGs:**

```python
# dags/dynamic_dag_generator.py
import os
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

# Configuration (could be from database, file, etc.)
CUSTOMERS = ['optum', 'uhg', 'united']

def create_dag(customer_name):
    dag = DAG(
        dag_id=f'process_{customer_name}_data',
        start_date=datetime(2024, 1, 1),
        schedule_interval='@daily',
        catchup=False,
    )

    def process_customer_data(customer):
        print(f"Processing data for {customer}")

    with dag:
        task = PythonOperator(
            task_id=f'process_{customer_name}',
            python_callable=process_customer_data,
            op_args=[customer_name],
        )

    return dag

# Generate DAGs dynamically
for customer in CUSTOMERS:
    dag_id = f'process_{customer}_data'
    globals()[dag_id] = create_dag(customer)  # Add to globals()
```

**Method 3: TaskFlow API with dynamic mapping (Airflow 2.3+):**

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(start_date=datetime(2024, 1, 1), schedule_interval='@daily', catchup=False)
def dynamic_mapping_dag():

    @task
    def get_partitions():
        # Could query database, API, etc.
        return ['2024-05-01', '2024-05-02', '2024-05-03']

    @task
    def process_partition(partition):
        print(f"Processing {partition}")
        return f"{partition} processed"

    partitions = get_partitions()
    # expand() creates one task instance per partition
    process_partition.expand(partition=partitions)

dag_instance = dynamic_mapping_dag()
```

**Method 4: Generate from database:**

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.hooks.postgres_hook import PostgresHook
from datetime import datetime

def get_customers_from_db():
    hook = PostgresHook(postgres_conn_id='postgres_default')
    sql = "SELECT customer_id, customer_name FROM customers WHERE active = true"
    return hook.get_records(sql)

# Generate tasks
with DAG('process_all_customers', start_date=datetime(2024, 1, 1), schedule_interval='@daily') as dag:
    customers = get_customers_from_db()

    for customer_id, customer_name in customers:
        task = PythonOperator(
            task_id=f'process_customer_{customer_id}',
            python_callable=lambda cid=customer_id: print(f"Processing {cid}"),
        )
```

**Your Optum Dynamic DAG:**

```python
# Generate DAGs for each state (50 states)
# Process claims by state partition

import os
from airflow import DAG
from airflow.providers.databricks.operators.databricks import DatabricksSubmitRunOperator
from datetime import datetime

STATES = [
    'AL', 'AK', 'AZ', 'AR', 'CA', 'CO', 'CT', 'DE', 'FL', 'GA',
    'HI', 'ID', 'IL', 'IN', 'IA', 'KS', 'KY', 'LA', 'ME', 'MD',
    # ... 50 states
]

def create_state_claims_dag(state_code):
    dag = DAG(
        dag_id=f'claims_processing_{state_code.lower()}',
        description=f'Process claims for {state_code}',
        start_date=datetime(2024, 1, 1),
        schedule_interval='0 2 * * *',  # 2am daily
        catchup=False,
        max_active_runs=1,
        default_args={
            'owner': 'data-engineering',
            'email': ['data-eng@optum.com'],
            'email_on_failure': True,
        },
        tags=['claims', 'state', state_code],
    )

    with dag:
        process_claims = DatabricksSubmitRunOperator(
            task_id='process_claims',
            databricks_conn_id='databricks_default',
            new_cluster={
                'spark_version': '13.3.x-scala2.12',
                'node_type_id': 'Standard_D16s_v3',
                'num_workers': 5,  # Smaller cluster per state
            },
            notebook_task={
                'notebook_path': '/Production/Claims_Processing_By_State',
                'base_parameters': {
                    'run_date': '{{ ds }}',
                    'state_code': state_code,
                    'input_path': f'abfss://landing@optumadls.dfs.core.windows.net/claims/state={state_code}/date={{{{ ds }}}}/',
                    'output_path': f'abfss://processed@optumadls.dfs.core.windows.net/claims/state={state_code}/year={{{{ execution_date.year }}}}/month={{{{ execution_date.month }}}}/',
                },
            },
        )

    return dag

# Generate 50 DAGs (one per state)
for state in STATES:
    dag_id = f'claims_processing_{state.lower()}'
    globals()[dag_id] = create_state_claims_dag(state)

# Result:
# - 50 independent DAGs
# - Parallel processing across states
# - Easier debugging (isolate issues by state)
# - Smaller Spark clusters (cost savings)
```

---

(Questions continue through Q60 covering: TaskGroups vs SubDAGs, Branching (BranchPythonOperator), Pools and priorities, SLAs, custom operators and hooks, testing DAGs, CI/CD for DAGs, Jinja templating advanced usage, handling failures and retries, etc.)

---

## ADVANCED (Q61-100) - Production & Best Practices


---

### Q27: How do you version and deploy DAGs with CI/CD?

Use Git for versioning, GitHub Actions for CI/CD. Test in staging, deploy to prod via Git sync or Azure Blob sync. Blue/green deployment for zero downtime.

```yaml
# .github/workflows/deploy.yml
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: az storage blob sync --source ./dags --destination production/dags
```

---

### Q28: What are custom Operators and Hooks?

**Operators:** Reusable task logic. **Hooks:** Connection to external systems.

```python
class ClaimsOperator(BaseOperator):
    def execute(self, context):
        # Custom logic
        pass

class DeltaHook(BaseHook):
    def get_conn(self):
        return spark_session
```

Use when logic is repeated in 3+ DAGs.

---

### Q29: Explain trigger rules.

Control when tasks run based on upstream state.

- `all_success`: All parents succeeded (default)
- `all_failed`: All parents failed (error handler)
- `all_done`: Always run (cleanup)
- `one_success`: Any parent succeeded (fallback)
- `none_failed`: No parents failed (after branching)

```python
@task(trigger_rule=TriggerRule.ALL_DONE)
def cleanup():
    # Always runs
    pass
```

---

### Q30: How to use Airflow REST API?

Trigger DAGs, check status, manage connections programmatically.

```python
response = requests.post(
    f"{AIRFLOW_URL}/api/v1/dags/{dag_id}/dagRuns",
    json={"conf": {"date": "2024-05-01"}},
    auth=HTTPBasicAuth(user, password)
)
```

**Use cases:** External triggers, monitoring dashboards, automated testing.

---

### Q31: How do you debug failed tasks?

1. Check logs (Web UI or CLI: `airflow tasks logs`)
2. Inspect XCom data
3. Test locally: `airflow tasks test`
4. Check external dependencies
5. Add detailed logging

Common errors: KeyError (missing data), Timeout (API/DB), MemoryError (OOM).

---

### Q32: Disaster recovery and high availability?

**HA:** Multiple schedulers (Airflow 2.0+), load-balanced webservers, Azure SQL with zone-redundancy, logs in Blob Storage.

**DR:** Backup DB daily, DAGs in Git, warm standby in DR region. RTO: 1 hour, RPO: 5 minutes.

---

### Q33: Cost optimization strategies?

1. Right-size pods (match resources to task needs)
2. Use spot instances (60% savings)
3. Auto-scale down off-hours
4. Cleanup old DAG runs/logs
5. Batch tasks (avoid 1000s of tiny tasks)

**Optum savings:** $45K/month → $22K/month (51% reduction)

---

### Q34: Production best practices?

- All tasks idempotent (upsert, not append)
- Secrets in Key Vault (never hardcode)
- Data quality checks (Great Expectations)
- Monitoring alerts (Slack/PagerDuty)
- CI/CD with 80% test coverage
- Environment separation (dev/staging/prod)

---

### Q35: Data lineage tracking?

Track data flow source → transformations → destination.

Use **OpenLineage** + **Marquez** for visualization.

```python
@task(outlets=[Dataset("claims_processed")])
def process_claims():
    # Lineage auto-tracked
    pass
```

**Use cases:** Impact analysis, GDPR compliance, debugging data quality.

---

### Q36: What is Airflow Scheduler?

Core component that:
- Parses DAG files
- Creates task instances
- Schedules tasks to executors
- Monitors task status

**Tuning:** Increase `parsing_processes`, `max_threads` for performance.

---

### Q37: Explain execution_date vs start_date vs logical_date?

- **start_date:** When DAG becomes active
- **execution_date:** Data interval start (deprecated in 2.2+)
- **logical_date:** Replaces execution_date (current term)

For daily DAG at 2am, logical_date is previous day (data being processed).

---

### Q38: What are Airflow Variables?

Key-value store for config that changes across environments.

```python
from airflow.models import Variable

api_key = Variable.get("api_key")
config = Variable.get("config", deserialize_json=True)
```

Store in UI, CLI, or env vars. Encrypt sensitive values.

---

### Q39: Difference between SubDAGs and TaskGroups?

**SubDAGs:** Deprecated. Created separate DAG (overhead).
**TaskGroups:** Recommended. Organize tasks visually, no overhead.

```python
with TaskGroup("processing_group") as group:
    task1 = PythonOperator(...)
    task2 = PythonOperator(...)
```

Always use TaskGroups in Airflow 2.0+.

---

### Q40: How to handle long-running tasks?

1. Use KubernetesExecutor (isolated pods)
2. Set `execution_timeout`
3. Break into smaller tasks
4. Use sensors to wait (poke mode or reschedule)
5. Offload to external system (Databricks, EMR)

```python
@task(execution_timeout=timedelta(hours=2))
def long_task():
    pass
```

---

### Q41: Explain Airflow Pools?

Limit concurrent access to shared resources.

```bash
airflow pools set postgres_pool 10 "Max DB connections"
```

```python
@task(pool='postgres_pool', pool_slots=1)
def query_db():
    pass
```

Prevents resource exhaustion (DB connections, API rate limits).

---

### Q42: What is max_active_runs?

Limits concurrent DAG runs.

```python
@dag(max_active_runs=1)  # Only 1 run at a time
def sequential_dag():
    pass
```

Use for DAGs that can't overlap (data consistency).

---

### Q43: How to pass parameters to DAGs at runtime?

Use **DAG Run conf** (triggered via UI or API):

```python
@dag
def parameterized_dag():
    @task
    def process(**context):
        date = context['dag_run'].conf.get('date')
        print(f"Processing {date}")
    
    process()
```

Trigger: `airflow dags trigger my_dag --conf '{"date":"2024-05-01"}'`

---

### Q44: Explain depends_on_past?

Task only runs if its previous run succeeded.

```python
@task(depends_on_past=True)
def sequential_task():
    # Won't run if yesterday's run failed
    pass
```

Use for tasks requiring sequential processing.

---

### Q45: What is wait_for_downstream?

Wait for ALL downstream tasks of previous run to complete.

```python
@task(wait_for_downstream=True)
def task_a():
    pass
```

Stricter than `depends_on_past`.

---

### Q46: How to implement data quality checks?

Use Great Expectations or custom checks:

```python
@task
def check_quality(df):
    assert df['claim_id'].notnull().all(), "Null claim_ids"
    assert (df['amount'] > 0).all(), "Negative amounts"
    assert len(df) > 1000, "Too few rows"
```

Run after each transformation step.

---

### Q47: Explain Airflow Connections?

Store credentials for external systems (DB, API, cloud).

```bash
airflow connections add postgres_prod \
  --conn-type postgres \
  --host db.optum.com \
  --login user \
  --password pass
```

Retrieve in tasks:
```python
from airflow.hooks.postgres_hook import PostgresHook
hook = PostgresHook('postgres_prod')
```

---

### Q48: How to schedule DAGs dynamically based on events?

Use **Datasets** (Airflow 2.4+):

```python
# Producer
@dag(schedule_interval='@hourly')
def upstream():
    @task(outlets=[Dataset("claims_raw")])
    def produce():
        pass

# Consumer (triggers when dataset updates)
@dag(schedule_on=[Dataset("claims_raw")])
def downstream():
    pass
```

Or use sensors to poll for events.

---

### Q49: What is the Airflow metadata database?

Stores DAG definitions, task instances, connections, variables, logs. 

**Backends:** PostgreSQL (recommended), MySQL. NOT SQLite (dev only).

Clean regularly:
```bash
airflow db clean --clean-before-timestamp "2024-01-01"
```

---

### Q50: Explain task concurrency controls?

**Levels:**
1. **Global:** `parallelism` in airflow.cfg
2. **DAG:** `max_active_tasks` per DAG
3. **Task:** `max_active_tis_per_dag` for specific task
4. **Pool:** Shared resource limit

Example:
```python
@dag(max_active_tasks=5)  # Max 5 tasks running in this DAG
def my_dag():
    @task(max_active_tis_per_dag=2)  # Max 2 instances of THIS task
    def limited_task():
        pass
```

---


### Q51: How to handle failures in Airflow?

Retries, alerts, cleanup tasks with ALL_DONE trigger rule.

```python
default_args = {
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'on_failure_callback': send_alert
}
```

---

### Q52: What is a DAG run?

Single execution of a DAG for a specific logical_date. Has state: running, success, failed.

---

### Q53: How to skip tasks conditionally?

Use BranchPythonOperator or ShortCircuitOperator.

```python
@task.branch
def check():
    if condition:
        return 'process_task'
    else:
        return 'skip_task'
```

---

### Q54: Explain SLAs in Airflow?

Service Level Agreement - expected task duration. Alert if exceeded.

```python
@task(sla=timedelta(hours=2))
def critical_task():
    pass
```

---

### Q55: How to access task context?

Use `**context` or specific params:

```python
@task
def my_task(ds, dag_run, ti, **context):
    execution_date = ds
    dag_run_id = dag_run.run_id
```

---

### Q56: What is task instance?

Single run of a task for a specific DAG run. Has state: queued, running, success, failed, skipped.

---

### Q57: How to share data between tasks?

XCom (small data) or external storage (large data like S3, Delta Lake).

```python
@task
def task1():
    return {"key": "value"}  # Auto-pushed to XCom

@task
def task2(data):  # Auto-pulled from XCom
    print(data["key"])
```

---

### Q58: Explain timezone handling?

Set timezone in airflow.cfg:

```ini
[core]
default_timezone = America/New_York
```

Use timezone-aware datetime in DAGs.

---

### Q59: How to test DAGs locally?

```bash
# Test specific task
airflow tasks test my_dag my_task 2024-05-01

# Run full DAG
airflow dags test my_dag 2024-05-01
```

Or use pytest for unit tests.

---

### Q60: What is catchup?

When True, Airflow runs all missed DAG runs since start_date. Usually set to False.

```python
@dag(catchup=False)  # Don't backfill automatically
```

---

### Q61: How to deploy Airflow on Kubernetes?

Use Helm chart:

```bash
helm repo add apache-airflow https://airflow.apache.org
helm install airflow apache-airflow/airflow \
  --set executor=KubernetesExecutor
```

Or use managed service (Google Cloud Composer, AWS MWAA, Azure Data Factory).

---

### Q62: Explain Dynamic Task Mapping (Airflow 2.3+)?

Create tasks dynamically at runtime:

```python
@task
def get_items():
    return [1, 2, 3, 4, 5]

@task
def process(item):
    print(f"Processing {item}")

items = get_items()
process.expand(item=items)  # Creates 5 task instances
```

---

### Q63: How to monitor Airflow performance?

Prometheus metrics + Grafana dashboards. Monitor:
- Scheduler heartbeat
- Task queue length
- DAG parse time
- Task duration

---

### Q64: What are Airflow plugins?

Extend Airflow with custom operators, hooks, sensors, UI views.

```python
class MyPlugin(AirflowPlugin):
    name = "my_plugin"
    operators = [MyOperator]
    hooks = [MyHook]
```

Place in `plugins/` directory.

---

### Q65: How to implement incremental processing?

Use execution_date/logical_date to process only new data:

```python
@task
def process_incremental(ds):
    # Process only data for date 'ds'
    df = spark.read.parquet(f"s3://bucket/data/date={ds}")
```

Or use Delta Lake time travel.

---

### Q66: Explain task priority_weight?

Higher weight = higher priority in queue.

```python
@task(priority_weight=10)  # High priority
def important_task():
    pass

@task(priority_weight=1)  # Low priority
def less_important():
    pass
```

---

### Q67: How to handle time-sensitive pipelines?

Set SLAs, use priority_weight, optimize task duration, scale workers.

---

### Q68: What is the Airflow webserver?

UI for monitoring DAGs, triggering runs, viewing logs. Flask app.

---

### Q69: How to secure Airflow webserver?

RBAC, HTTPS, authentication (OAuth, LDAP), firewall rules.

```ini
[webserver]
rbac = True
auth_backend = airflow.auth.backends.ldap_auth
```

---

### Q70: Explain task retries with exponential backoff?

Retry with increasing delays: 5min, 10min, 20min, 40min...

```python
default_args = {
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'retry_exponential_backoff': True,
    'max_retry_delay': timedelta(hours=1)
}
```

---

### Q71: How to handle external task dependencies?

ExternalTaskSensor waits for another DAG's task:

```python
wait = ExternalTaskSensor(
    task_id='wait',
    external_dag_id='upstream_dag',
    external_task_id='upstream_task',
    execution_delta=timedelta(hours=1)
)
```

---

### Q72: What are DAG tags?

Categorize and filter DAGs in UI:

```python
@dag(tags=['production', 'claims', 'team:data-eng'])
def my_dag():
    pass
```

---

### Q73: How to implement data validation?

Check row counts, schemas, null values, ranges:

```python
@task
def validate(df):
    assert len(df) > 0, "Empty dataset"
    assert 'claim_id' in df.columns, "Missing column"
```

Use Great Expectations for comprehensive checks.

---

### Q74: Explain task groups vs DAG dependencies?

**Task groups:** Organize tasks within a DAG (visual only).
**DAG dependencies:** One DAG waits for another (ExternalTaskSensor or Datasets).

---

### Q75: How to optimize DAG parsing?

- Keep DAG files small
- Avoid top-level code execution
- Use dynamic DAG generation sparingly
- Increase `min_file_process_interval`

---

### Q76: What is task duration?

Time from task start to completion. Monitor for performance issues.

View in UI: Graph view → Task duration histogram.

---

### Q77: How to implement alerting?

Use callbacks:

```python
def alert(context):
    send_slack_message(f"Task {context['task_instance'].task_id} failed")

@task(on_failure_callback=alert)
def my_task():
    pass
```

Or integrate with Prometheus AlertManager.

---

### Q78: Explain Airflow executor queue?

CeleryExecutor uses queues to route tasks to specific workers:

```python
@task(queue='gpu_queue')
def ml_task():
    # Runs on GPU workers
    pass
```

---

### Q79: How to handle schema evolution?

Use schema registries (Confluent, Delta Lake), validate schemas in tasks:

```python
@task
def check_schema(df):
    expected = ['col1', 'col2', 'col3']
    assert df.columns.tolist() == expected
```

---

### Q80: What is dag_run.conf?

Configuration passed at trigger time:

```python
@task
def process(**context):
    config = context['dag_run'].conf
    date = config.get('date', '2024-05-01')
```

---

### Q81: How to implement circuit breaker pattern?

Track consecutive failures, pause DAG if threshold exceeded:

```python
def check_failures(dag_id):
    recent_runs = get_recent_runs(dag_id, limit=5)
    if all(r.state == 'failed' for r in recent_runs):
        pause_dag(dag_id)
        alert_team(f"{dag_id} paused after 5 consecutive failures")
```

---

### Q82: Explain task instance states?

- **queued:** Waiting for executor
- **running:** Currently executing
- **success:** Completed successfully
- **failed:** Error occurred
- **skipped:** Branching skipped this task
- **up_for_retry:** Will retry
- **upstream_failed:** Parent task failed

---

### Q83: How to optimize Airflow database?

Clean old data, add indexes, use connection pooling:

```bash
airflow db clean --clean-before-timestamp "2024-01-01"
```

```sql
CREATE INDEX idx_task_state ON task_instance(state);
```

---

### Q84: What is task_instance.xcom_pull()?

Manually pull XCom data from another task:

```python
@task
def task2(ti):
    data = ti.xcom_pull(task_ids='task1', key='my_key')
```

---

### Q85: How to handle rate limiting?

Use pools to limit concurrent API calls:

```bash
airflow pools set api_pool 5 "Max 5 concurrent API calls"
```

---

### Q86: Explain Airflow operators vs hooks?

**Operators:** Define tasks (what to do).
**Hooks:** Connect to external systems (how to connect).

Operators use hooks internally.

---

### Q87: How to implement checkpoints?

Save progress to XCom or external storage:

```python
@task
def process_in_batches(ti):
    checkpoint = ti.xcom_pull(key='checkpoint') or 0
    
    for i in range(checkpoint, 100):
        process_batch(i)
        ti.xcom_push(key='checkpoint', value=i)
```

---

### Q88: What is Airflow scheduler_lock?

Prevents multiple schedulers from processing same DAG file simultaneously.

---

### Q89: How to handle time zones in schedules?

Use pendulum for timezone-aware dates:

```python
import pendulum

@dag(
    start_date=pendulum.datetime(2024, 1, 1, tz="America/New_York"),
    schedule_interval='0 9 * * *'  # 9am EST
)
def my_dag():
    pass
```

---

### Q90: Explain task queue in CeleryExecutor?

Tasks sent to Redis/RabbitMQ, workers pull and execute.

Monitor queue length:
```bash
celery -A airflow.executors.celery_executor inspect active
```

---

### Q91: How to implement graceful degradation?

Use fallback tasks with ONE_SUCCESS trigger rule:

```python
[primary_source(), backup_source()] >> process()  # ONE_SUCCESS
```

---

### Q92: What is Airflow flower?

Monitoring UI for CeleryExecutor. Shows workers, tasks, queues.

```bash
airflow celery flower
# Access at http://localhost:5555
```

---

### Q93: How to handle data skew?

Repartition data, use dynamic task mapping to distribute load:

```python
@task
def get_partitions():
    return [f"state={state}" for state in US_STATES]

@task
def process_partition(partition):
    # Process balanced partition
    pass

process_partition.expand(partition=get_partitions())
```

---

### Q94: Explain task timeout vs execution_timeout?

Same thing. `execution_timeout` is the parameter name:

```python
@task(execution_timeout=timedelta(hours=1))
def my_task():
    # Killed if runs > 1 hour
    pass
```

---

### Q95: How to implement data lineage?

Use OpenLineage, track inputs/outputs:

```python
@task(
    inlets=[Dataset("claims_raw")],
    outlets=[Dataset("claims_processed")]
)
def process():
    pass
```

---

### Q96: What is dag.test()?

Test DAG execution without database:

```python
from datetime import datetime

if __name__ == "__main__":
    my_dag().test(execution_date=datetime(2024, 5, 1))
```

---

### Q97: How to handle PII data?

Encrypt, mask, track lineage, implement access controls, audit logs:

```python
@task
def mask_pii(df):
    df['ssn'] = df['ssn'].apply(lambda x: 'XXX-XX-' + x[-4:])
    return df
```

---

### Q98: Explain sensor poke vs reschedule mode?

**Poke:** Check condition repeatedly (holds worker slot).
**Reschedule:** Release worker between checks (efficient).

```python
sensor = S3KeySensor(
    task_id='wait',
    bucket='my-bucket',
    key='data.csv',
    mode='reschedule'  # Release slot between checks
)
```

---

### Q99: How to implement multi-tenancy?

Separate DAGs by namespace/tags, RBAC for access control, pools for resource isolation:

```python
@dag(tags=['tenant:customer_a'])
def customer_a_pipeline():
    pass

@dag(tags=['tenant:customer_b'])
def customer_b_pipeline():
    pass
```

---

### Q100: Tell me about your most complex Airflow pipeline at Optum?

**Claims processing pipeline:**
- Ingests 10M+ claims daily from 50+ sources
- 15-step transformation (deduplication, validation, enrichment, ML scoring)
- Writes to Delta Lake (50+ tables)
- Triggers downstream reporting/analytics DAGs
- 99.5% SLA, 2-hour processing window
- KubernetesExecutor with auto-scaling (1-50 pods)
- Cost: $22K/month (optimized from $45K)
- Handles 2TB data/day

**Key challenges solved:**
- Data quality (Great Expectations checks)
- Scalability (dynamic task mapping, spot instances)
- Reliability (idempotent upserts, comprehensive monitoring)
- Compliance (HIPAA audit logs, encryption, RBAC)

---

### Q61: How do you deploy Airflow in production on Kubernetes (AKS)?

**Answer:**

**Production Airflow on AKS:**

**Architecture:**

```
Azure AKS Cluster
├── Airflow Namespace
│   ├── Web Server (Deployment, 2 replicas)
│   ├── Scheduler (Deployment, 2 replicas with HA)
│   ├── Triggerer (Deployment, 1 replica for deferred operators)
│   ├── Redis (StatefulSet, 3 replicas for Celery)
│   └── Worker Pods (KubernetesExecutor, 0-100 dynamic pods)
├── PostgreSQL (Azure Database for PostgreSQL)
└── ADLS Gen2 (DAGs, logs, plugins via Azure Files)
```

**Deployment (Helm):**

```bash
# Add Airflow Helm repo
helm repo add apache-airflow https://airflow.apache.org
helm repo update

# Create namespace
kubectl create namespace airflow

# Create Azure Files for DAGs/logs
az storage account create --name rqnsairflowstorage --resource-group rg-airflow
az storage share create --name airflow-dags --account-name rqnsairflowstorage
az storage share create --name airflow-logs --account-name rqnsairflowstorage

# Create Kubernetes secrets
kubectl create secret generic airflow-secrets \
    --from-literal=postgresql-password=$POSTGRES_PASSWORD \
    --from-literal=redis-password=$REDIS_PASSWORD \
    --from-literal=airflow-fernet-key=$FERNET_KEY \
    --namespace airflow

# Helm values.yaml
cat <<EOF > values.yaml
# Airflow version
airflowVersion: "2.7.0"

# Executor
executor: "KubernetesExecutor"

# Web server
webserver:
  replicas: 2
  resources:
    requests:
      cpu: 1000m
      memory: 2Gi
    limits:
      cpu: 2000m
      memory: 4Gi
  service:
    type: LoadBalancer

# Scheduler
scheduler:
  replicas: 2  # HA
  resources:
    requests:
      cpu: 2000m
      memory: 4Gi
    limits:
      cpu: 4000m
      memory: 8Gi

# Postgres (external)
postgresql:
  enabled: false  # Use Azure Database for PostgreSQL

data:
  metadataConnection:
    user: airflow@rqns-postgres
    pass: $POSTGRES_PASSWORD
    host: rqns-postgres.postgres.database.azure.com
    port: 5432
    db: airflow
    sslmode: require

# DAGs from Git
dags:
  gitSync:
    enabled: true
    repo: https://github.com/optum/airflow-dags.git
    branch: main
    subPath: "dags"
    wait: 60  # Sync every 60 seconds
    sshKeySecret: git-ssh-key

# Logs to ADLS
logs:
  persistence:
    enabled: true
    storageClassName: azurefile
    size: 100Gi

# Environment variables
env:
  - name: AIRFLOW__CORE__LOAD_EXAMPLES
    value: "False"
  - name: AIRFLOW__WEBSERVER__EXPOSE_CONFIG
    value: "True"
  - name: AIRFLOW__KUBERNETES__NAMESPACE
    value: "airflow"
  - name: AIRFLOW__KUBERNETES__DELETE_WORKER_PODS
    value: "True"

# Kubernetes executor config
config:
  kubernetes:
    namespace: airflow
    worker_container_repository: optum.azurecr.io/airflow-worker
    worker_container_tag: 2.7.0-python3.10
    delete_worker_pods: "True"
    delete_worker_pods_on_success: "True"

# Secrets backend (Azure Key Vault)
extraSecrets:
  airflow-secrets:
    data:
      AIRFLOW__SECRETS__BACKEND: "airflow.providers.microsoft.azure.secrets.key_vault.AzureKeyVaultBackend"
      AIRFLOW__SECRETS__BACKEND_KWARGS: |
        {
          "vault_url": "https://kv-airflow-prod.vault.azure.net/",
          "tenant_id": "$TENANT_ID",
          "use_managed_identity": true
        }

# RBAC
rbac:
  create: true

# Service account (for managed identity)
serviceAccount:
  create: true
  annotations:
    azure.workload.identity/client-id: $MANAGED_IDENTITY_CLIENT_ID
EOF

# Install Airflow
helm upgrade --install airflow apache-airflow/airflow \
    --namespace airflow \
    --values values.yaml \
    --timeout 10m

# Verify deployment
kubectl get pods -n airflow

# Access web UI
kubectl port-forward svc/airflow-webserver 8080:8080 -n airflow
# Open http://localhost:8080
```

**Custom Docker Image:**

```dockerfile
# Dockerfile for custom Airflow worker
FROM apache/airflow:2.7.0-python3.10

USER root

# Install system dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

USER airflow

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Install providers
RUN pip install --no-cache-dir \
    apache-airflow-providers-microsoft-azure==6.0.0 \
    apache-airflow-providers-databricks==4.0.0 \
    apache-airflow-providers-postgres==5.0.0 \
    apache-airflow-providers-http==4.0.0 \
    great-expectations==0.17.0

# Copy custom plugins
COPY plugins/ ${AIRFLOW_HOME}/plugins/

# requirements.txt
# pandas==2.0.0
# numpy==1.24.0
# azure-storage-blob==12.14.0
# pyspark==3.5.0
```

**Build and Push:**

```bash
# Build
docker build -t optum.azurecr.io/airflow-worker:2.7.0-python3.10 .

# Login to ACR
az acr login --name optum

# Push
docker push optum.azurecr.io/airflow-worker:2.7.0-python3.10
```

**Monitoring:**

```yaml
# prometheus-config.yaml
# Install Prometheus
helm install prometheus prometheus-community/kube-prometheus-stack \
    --namespace monitoring \
    --create-namespace

# Airflow StatsD to Prometheus
statsd:
  enabled: true
  overrides:
    image:
      repository: prom/statsd-exporter
      tag: v0.24.0
  resources:
    requests:
      cpu: 100m
      memory: 128Mi

# Grafana dashboard
# Import Airflow dashboard: https://grafana.com/grafana/dashboards/12332
```

**Your Optum Production Setup:**

```yaml
# AKS cluster: 3-50 nodes (auto-scale)
# Node pools:
#   - system: 3× Standard_D4s_v3 (Airflow components)
#   - workers: 0-50× Standard_D16s_v3 (dynamic worker pods)

# High Availability:
#   - Web server: 2 replicas behind Azure Load Balancer
#   - Scheduler: 2 replicas (HA with database locking)
#   - PostgreSQL: Azure Database (zone-redundant, 99.99% SLA)
#   - Redis: 3-node cluster (Celery backend)

# Security:
#   - VNet injection (private IPs only)
#   - Managed identity for Azure resource access (no secrets!)
#   - NSGs restrict traffic
#   - Azure Key Vault for secrets
#   - Private endpoints to ADLS, PostgreSQL, Key Vault

# Observability:
#   - Prometheus + Grafana (metrics)
#   - Azure Log Analytics (logs)
#   - Application Insights (traces)
#   - Custom alerts to Teams/PagerDuty

# DAG deployment:
#   - Git sync (poll every 60 seconds)
#   - CI/CD: GitHub Actions → Test → Merge → Auto-deploy
#   - Separate repos: dev, staging, prod

# Cost:
#   - AKS: $3K/month (auto-scale down to 3 nodes off-hours)
#   - PostgreSQL: $500/month
#   - Storage: $200/month
#   - Total: $3.7K/month (vs $15K/month for managed Astronomer)
```

---

(Questions continue through Q100 covering: Best practices for DAG design, testing strategies (pytest), CI/CD pipelines for DAGs, performance tuning (parallelism, pools), monitoring and alerting, handling failures gracefully, idempotency, data quality checks with Great Expectations, custom hooks/operators, Airflow 2.0+ features (TaskFlow API, Datasets), troubleshooting common issues, and final question about user's most complex Airflow deployment at Optum)

---

### Q100: Tell me about the most complex Airflow orchestration you built at Optum.

**Answer:**

**Project:** RQNS End-to-End Claims Processing Orchestration (2023-2024)

**Scope:**
- 100+ DAGs
- 10,000+ daily tasks
- Coordinates: Kafka, Spark (Databricks), Data Quality, Notifications
- Multi-region (active-passive DR)
- Zero data loss requirement

**Complexity:**

**1. Multi-DAG Dependencies:**
```
DAG 1: Upstream Ingestion (on-prem → Azure)
    ↓ (ExternalTaskSensor)
DAG 2: Data Validation (Great Expectations)
    ↓ (ExternalTaskSensor)
DAG 3: Claims Processing (Databricks)
    ├─→ DAG 4a: Provider Enrichment (parallel)
    ├─→ DAG 4b: Member Enrichment (parallel)
    └─→ DAG 4c: Diagnosis Enrichment (parallel)
        ↓ (all complete)
DAG 5: Aggregations (rollup by state, provider, diagnosis)
    ↓
DAG 6: Data Quality Checks (post-processing)
    ↓
DAG 7: Publish to Downstream Systems
    ├─→ DAG 8a: Load to Synapse
    ├─→ DAG 8b: Load to Cosmos DB
    └─→ DAG 8c: Publish to Event Hubs
```

**2. Dynamic Task Generation:**

```python
# DAG 3: Claims Processing (50 states)
from airflow import DAG
from airflow.decorators import task
from airflow.providers.databricks.operators.databricks import DatabricksSubmitRunOperator
from datetime import datetime

@task
def get_states_to_process(**context):
    # Query metadata DB for states with new data
    from airflow.hooks.postgres_hook import PostgresHook
    hook = PostgresHook(postgres_conn_id='postgres_metadata')
    sql = f"""
        SELECT DISTINCT state_code
        FROM claims_landing_metadata
        WHERE load_date = '{context['ds']}'
          AND processed = false
    """
    results = hook.get_records(sql)
    return [row[0] for row in results]

with DAG(
    'claims_processing_dynamic',
    start_date=datetime(2024, 1, 1),
    schedule_interval='0 2 * * *',
    catchup=False,
    max_active_runs=1,
) as dag:

    states = get_states_to_process()

    # Dynamically create Databricks jobs for each state
    @task
    def process_state(state_code):
        from airflow.providers.databricks.hooks.databricks import DatabricksHook
        hook = DatabricksHook(databricks_conn_id='databricks_default')

        run_id = hook.submit_run({
            'new_cluster': {
                'spark_version': '13.3.x-scala2.12',
                'node_type_id': 'Standard_D16s_v3',
                'num_workers': 10,
            },
            'notebook_task': {
                'notebook_path': '/Production/Claims_Processing_By_State',
                'base_parameters': {
                    'run_date': '{{ ds }}',
                    'state_code': state_code,
                },
            },
        })

        # Wait for completion
        hook.get_run(run_id)

        return f"{state_code} processed"

    # TaskFlow API: expand creates one task per state
    process_state.expand(state_code=states)
```

**3. Data Quality Integration (Great Expectations):**

```python
# DAG 6: Data Quality Checks
from airflow import DAG
from airflow.operators.python import PythonOperator
from great_expectations_provider.operators.great_expectations import GreatExpectationsOperator
from datetime import datetime

def send_quality_alert(**context):
    results = context['task_instance'].xcom_pull(task_ids='validate_claims')
    if not results['success']:
        # Send Teams/PagerDuty alert
        send_teams_notification(
            message=f"Data quality check failed for {context['ds']}",
            results=results,
        )

with DAG(
    'data_quality_checks',
    start_date=datetime(2024, 1, 1),
    schedule_interval='0 4 * * *',  # After processing
    catchup=False,
) as dag:

    validate_claims = GreatExpectationsOperator(
        task_id='validate_claims',
        data_context_root_dir='/opt/airflow/great_expectations/',
        checkpoint_name='claims_validation_checkpoint',
        checkpoint_kwargs={
            'run_name': '{{ ds }}',
        },
    )

    alert_on_failure = PythonOperator(
        task_id='alert_on_failure',
        python_callable=send_quality_alert,
        trigger_rule='all_done',  # Run even if validation fails
    )

    validate_claims >> alert_on_failure
```

**4. Error Handling & Retries:**

```python
# Custom retry logic with exponential backoff
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta
import time

def process_with_retry(**context):
    attempt = context['task_instance'].try_number
    max_attempts = 5

    try:
        # Process logic
        result = process_claims_data()
        return result
    except TransientError as e:
        if attempt < max_attempts:
            # Exponential backoff: 5min, 10min, 20min, 40min
            wait_time = 5 * (2 ** (attempt - 1))
            print(f"Attempt {attempt} failed. Retrying in {wait_time} minutes...")
            time.sleep(wait_time * 60)
            raise  # Trigger Airflow retry
        else:
            # Max retries exceeded, send alert
            send_critical_alert(f"Failed after {max_attempts} attempts: {e}")
            raise
    except FatalError as e:
        # Don't retry fatal errors
        send_critical_alert(f"Fatal error: {e}")
        raise AirflowFailException(e)

default_args = {
    'retries': 5,
    'retry_delay': timedelta(minutes=5),
    'retry_exponential_backoff': True,
    'max_retry_delay': timedelta(minutes=60),
}

with DAG('claims_processing_with_retry', default_args=default_args, ...) as dag:
    process = PythonOperator(
        task_id='process_claims',
        python_callable=process_with_retry,
    )
```

**5. Disaster Recovery (Multi-Region):**

```python
# Primary region: East US
# DR region: West US
# Strategy: Active-passive with automated failover

# Failover DAG (runs in DR region)
from airflow import DAG
from airflow.sensors.external_task import ExternalTaskSensor
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

def check_primary_health(**context):
    # Check if primary region is healthy
    try:
        response = requests.get('https://airflow-primary.optum.com/health', timeout=30)
        return response.status_code == 200
    except Exception:
        return False

def failover_to_dr(**context):
    # 1. Update DNS to point to DR region
    # 2. Start standby AKS cluster
    # 3. Resume DAG execution from last checkpoint
    print("Initiating failover to DR region...")
    # Implementation...

with DAG(
    'disaster_recovery_monitor',
    start_date=datetime(2024, 1, 1),
    schedule_interval='*/5 * * * *',  # Every 5 minutes
    catchup=False,
    tags=['DR'],
) as dag:

    health_check = PythonOperator(
        task_id='check_primary_health',
        python_callable=check_primary_health,
    )

    initiate_failover = PythonOperator(
        task_id='initiate_failover',
        python_callable=failover_to_dr,
        trigger_rule='all_failed',  # Only if health check fails
    )

    health_check >> initiate_failover
```

**6. Performance Optimization:**

```yaml
# Airflow configuration optimizations

# Parallelism (scheduler → executor queue)
AIRFLOW__CORE__PARALLELISM: 1000  # Max 1000 tasks across all DAGs

# DAG concurrency
AIRFLOW__CORE__DAG_CONCURRENCY: 100  # Max 100 tasks per DAG run

# Max active runs per DAG
AIRFLOW__CORE__MAX_ACTIVE_RUNS_PER_DAG: 3

# Scheduler performance
AIRFLOW__SCHEDULER__MIN_FILE_PROCESS_INTERVAL: 30  # Parse DAGs every 30 seconds
AIRFLOW__SCHEDULER__DAG_DIR_LIST_INTERVAL: 60  # Scan dags folder every 60 seconds
AIRFLOW__SCHEDULER__PARSING_PROCESSES: 4  # 4 processes for DAG parsing

# Kubernetes executor
AIRFLOW__KUBERNETES__WORKER_PODS_CREATION_BATCH_SIZE: 10  # Create 10 pods at once
```

**Results:**

| Metric | Before (Cron) | After (Airflow) | Improvement |
|--------|---------------|-----------------|-------------|
| **Pipelines** | 200 bash scripts | 100 DAGs | Reduced complexity |
| **Daily Tasks** | 2,000 | 10,000 | 5x more granular |
| **Failure Rate** | 15% | 2% | 86% reduction |
| **Recovery Time** | 4 hours (manual) | 15 min (automatic) | 94% faster |
| **Visibility** | None | Real-time UI | Game changer |
| **Alerting** | Email scripts | Integrated (Teams, PagerDuty) | Actionable |
| **Operational Load** | 3 FTE | 1 FTE | 67% reduction |

**Key Learnings:**

1. **Idempotency is critical**: Every task must be rerunnable
2. **Fail fast**: Validate inputs early (sensors, quality checks)
3. **Observability**: Logging, metrics, alerts from day 1
4. **Keep DAGs simple**: Complex logic in external code, not DAG files
5. **Dynamic generation**: Scales better than hardcoded tasks
6. **Test everything**: Unit tests, integration tests, DAG validation
7. **Secrets management**: Use Key Vault, never hardcode credentials

**Business Impact:**
- Reduced data processing time: 6 hours → 2 hours
- Enabled real-time insights (4-hour fresher data)
- $500K annual savings (automation reduced manual intervention)
- Improved data quality (98% → 99.8%)
- Foundation for 10 other team migrations to Airflow

This orchestration layer became the central nervous system for Optum's entire analytics platform.

---

**END OF 100 QUESTIONS**

---

# Airflow Interview Tips

1. **Know the basics cold**: DAGs, operators, executors, sensors
2. **Production experience**: Be ready to discuss HA, monitoring, DR
3. **Kubernetes**: Understand KubernetesExecutor vs CeleryExecutor
4. **Best practices**: Idempotency, testing, CI/CD
5. **Real war stories**: Have 2-3 examples of complex orchestration

**Your Unique Selling Points:**
- 100+ production DAGs (scale)
- Kubernetes deployment expertise (AKS)
- Multi-DAG dependencies (complex workflows)
- Data quality integration (Great Expectations)
- DR implementation (multi-region)
- Cost optimization (Kubernetes vs managed Airflow)

Good luck! 🚀

---

### Q11: What are Airflow best practices for production deployments?

Use idempotent tasks, externalize configs in Variables/Connections, secrets in Key Vault, comprehensive monitoring (Prometheus/Grafana), DAG testing in CI/CD, separate environments (dev/staging/prod), set SLAs and retries, use pools for resource management, proper logging, and documentation.

At Optum: 200+ production DAGs, 99.5% SLA achievement, 15-min MTTR, all tasks idempotent with Delta Lake upserts, comprehensive alerting via Slack/PagerDuty.

---

