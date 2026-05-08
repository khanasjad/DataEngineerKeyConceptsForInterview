# Airflow Cheatsheet - Quick Reference

## Core Concepts
- **Airflow**: Workflow orchestration platform for scheduling and monitoring data pipelines
- **DAG (Directed Acyclic Graph)**: Collection of tasks with dependencies, no cycles
- **Task**: Single unit of work (instance of an operator)
- **Operator**: Template for a task (PythonOperator, BashOperator, etc.)
- **TaskInstance**: Single execution of a task for a specific DAG run
- **DAG Run**: Single execution of a DAG for a specific date/time

## Scheduling
- **start_date**: When DAG becomes active
- **schedule_interval**: How often DAG runs (@daily, cron expression)
- **execution_date / logical_date**: Data interval being processed (not current time)
- **catchup**: Auto-run missed DAG runs since start_date (default True, set False)
- **max_active_runs**: Limit concurrent DAG runs

## Task Dependencies
- **>>**: Downstream dependency (task1 >> task2 means task2 runs after task1)
- **<<**: Upstream dependency
- **depends_on_past**: Task only runs if previous run succeeded
- **wait_for_downstream**: Wait for all downstream tasks of previous run

## Executors
- **SequentialExecutor**: One task at a time, SQLite, dev only
- **LocalExecutor**: Multi-process on single machine, PostgreSQL/MySQL
- **CeleryExecutor**: Distributed with Redis/RabbitMQ queue, fixed workers
- **KubernetesExecutor**: Each task in separate pod, dynamic scaling

## Data Passing
- **XCom**: Cross-communication between tasks (max 48KB recommended)
- **xcom_push()**: Send data
- **xcom_pull()**: Retrieve data
- **TaskFlow API**: @task decorator auto-handles XCom

## Sensors
- **BaseSensor**: Wait for condition to be true
- **poke mode**: Check continuously (holds worker slot)
- **reschedule mode**: Release worker between checks (efficient)
- **timeout**: Max wait time
- **poke_interval**: Time between checks

## Trigger Rules
- **all_success**: All parents succeeded (default)
- **all_failed**: All parents failed
- **all_done**: All parents finished (success or failed)
- **one_success**: At least one parent succeeded
- **one_failed**: At least one parent failed
- **none_failed**: No parents failed (skipped OK)

## Branching
- **BranchPythonOperator**: Choose which downstream task(s) to execute
- **ShortCircuitOperator**: Stop pipeline if condition False
- **@task.branch**: Decorator for branch logic

## Configuration
- **Variables**: Key-value store for config (Airflow UI or CLI)
- **Connections**: Store credentials for external systems
- **Pools**: Limit concurrent tasks accessing shared resources
- **priority_weight**: Task priority in queue (higher = first)

## Task Groups
- **TaskGroup**: Organize tasks visually (no overhead)
- **SubDAG**: Deprecated, creates separate DAG (use TaskGroups instead)

## Retries & Timeouts
- **retries**: Number of retry attempts
- **retry_delay**: Wait time between retries
- **retry_exponential_backoff**: Increase delay exponentially
- **execution_timeout**: Max task runtime

## Monitoring
- **SLA**: Expected task duration, alert if exceeded
- **on_failure_callback**: Function called when task fails
- **on_success_callback**: Function called when task succeeds
- **sla_miss_callback**: Function called when SLA missed

## Performance
- **parallelism**: Global max tasks running (airflow.cfg)
- **dag_concurrency / max_active_tasks**: Max tasks per DAG
- **max_active_tis_per_dag**: Max instances of same task
- **Pools**: Shared resource limits

## Best Practices
- **Idempotency**: Tasks produce same result when rerun
- **Atomic tasks**: Small, single-purpose tasks
- **External storage**: Use S3/Delta Lake for large data, not XCom
- **Secrets in Key Vault**: Never hardcode credentials
- **catchup=False**: For most production DAGs

## CLI Commands
```bash
airflow dags list                          # List all DAGs
airflow tasks test <dag> <task> <date>     # Test task locally
airflow dags trigger <dag>                 # Manual trigger
airflow dags backfill <dag> --start --end  # Backfill date range
airflow db clean --clean-before <date>     # Clean old data
```

## REST API Endpoints
- `POST /api/v1/dags/{dag_id}/dagRuns` - Trigger DAG
- `GET /api/v1/dags/{dag_id}/dagRuns/{run_id}` - Get run status
- `PATCH /api/v1/dags/{dag_id}` - Pause/unpause

## Production Checklist
✅ Idempotent tasks | ✅ Secrets in Key Vault | ✅ Monitoring alerts | ✅ CI/CD pipeline | ✅ Data quality checks | ✅ Pool management | ✅ SLAs defined | ✅ Documentation | ✅ Error handling | ✅ Cost optimization
