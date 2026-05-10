# Airflow - Workflow Orchestration & Data Pipeline Management

**Master Apache Airflow for data engineering interviews**

---

## 📚 What's in This Folder

### **1. 100-QUESTIONS.md**
Comprehensive Q&A covering all Airflow concepts:
- DAG design and best practices
- Operators and sensors
- Executors and scaling
- Scheduling and dependencies
- Monitoring and alerting
- Production patterns
- Troubleshooting
- Real-world scenarios

### **2. CHEATSHEET.md**
Quick reference guide with:
- One-line definitions of key concepts
- Common commands and syntax
- DAG configuration examples
- Operator reference
- Best practices checklist
- Quick tips for interviews

---

## 🎯 What You'll Learn

### **Core Concepts**
- **DAGs (Directed Acyclic Graphs):** Workflow definition, no cycles
- **Tasks:** Units of work (instances of operators)
- **Operators:** Templates for tasks (PythonOperator, BashOperator, etc.)
- **Sensors:** Wait for conditions before proceeding
- **Hooks:** Interfaces to external systems
- **XComs:** Cross-communication between tasks
- **Executors:** How tasks are executed (Sequential, Local, Celery, Kubernetes)

### **Advanced Topics**
- Dynamic DAG generation
- Task dependencies and branching
- SubDAGs and TaskGroups
- Callbacks and alerting
- SLA monitoring
- Backfilling and catchup
- Pool and queue management
- Custom operators and sensors

### **Production Patterns**
- Idempotency
- Error handling and retries
- Data quality checks
- Incremental processing
- External task sensors
- Multi-environment deployment
- CI/CD for DAGs
- Performance optimization

---

## 💼 Why Airflow Matters for Data Engineers

### **Industry Adoption**
- Used by: Airbnb, Twitter, Lyft, Reddit, Spotify, Adobe
- De facto standard for workflow orchestration
- 35,000+ GitHub stars
- Strong community and ecosystem

### **Key Responsibilities**
- Orchestrate ETL/ELT pipelines
- Schedule batch jobs
- Monitor data pipeline health
- Coordinate multi-step workflows
- Manage dependencies across systems
- Handle failures and retries

### **Interview Focus Areas**
1. **DAG Design:** How to structure complex workflows
2. **Execution Model:** Understanding how Airflow runs tasks
3. **Scaling:** Executor choices for different scenarios
4. **Production Issues:** Debugging, monitoring, optimization
5. **Real-world Scenarios:** "How would you build X pipeline?"

---

## 🚀 Quick Start Guide

### **1. Read the Cheatsheet (15 minutes)**
Start with `CHEATSHEET.md` to get familiar with core concepts and terminology.

### **2. Study 100 Questions (3-5 hours)**
Work through `100-QUESTIONS.md` in sections:
- **Q1-Q20:** Fundamentals (DAGs, tasks, operators)
- **Q21-Q40:** Scheduling and execution
- **Q41-Q60:** Advanced features and patterns
- **Q61-Q80:** Production and scaling
- **Q81-Q100:** Real-world scenarios and troubleshooting

### **3. Practice Hands-On**
```bash
# Install Airflow
pip install apache-airflow

# Initialize database
airflow db init

# Create example DAG
# See 100-QUESTIONS.md for sample DAGs

# Start webserver
airflow webserver --port 8080

# Start scheduler
airflow scheduler
```

### **4. Key Interview Questions to Master**
- Explain Airflow architecture (scheduler, executor, webserver, metadata DB)
- Design a DAG for incremental data loading
- How do you handle task failures?
- Difference between executors (Local vs Celery vs Kubernetes)
- How to prevent duplicate runs?
- Backfill strategy for historical data

---

## 📖 Topic Coverage

### **Fundamentals (Q1-Q20)**
- DAG definition and structure
- Task dependencies (>>, <<, set_upstream, set_downstream)
- Operators overview (Python, Bash, SQL, etc.)
- Scheduling basics (schedule_interval, start_date)
- Execution date vs run date

### **Intermediate (Q21-Q60)**
- Sensors and waiting patterns
- XCom for data passing
- Branching (BranchPythonOperator)
- Trigger rules (all_success, one_failed, etc.)
- SubDAGs and TaskGroups
- Pools and concurrency
- SLA monitoring

### **Advanced (Q61-Q80)**
- Executors (Sequential, Local, Celery, Kubernetes)
- Dynamic DAG generation
- Custom operators
- Airflow variables and connections
- Testing DAGs (unit tests, integration tests)
- CI/CD for Airflow
- Multi-tenancy

### **Production (Q81-Q100)**
- Monitoring and alerting
- Performance optimization
- Common pitfalls and solutions
- Debugging failed tasks
- Scaling strategies
- Security best practices
- Real Optum use cases

---

## 🎓 Interview Preparation Strategy

### **Week 1: Fundamentals**
- Read CHEATSHEET.md
- Study Q1-Q30 from 100-QUESTIONS.md
- Set up local Airflow instance
- Create 2-3 simple DAGs

### **Week 2: Advanced Concepts**
- Study Q31-Q60
- Practice dynamic DAG generation
- Implement error handling patterns
- Explore different operators

### **Week 3: Production & Scale**
- Study Q61-Q100
- Review Optum use cases
- Practice explaining architectural decisions
- Mock interview scenarios

### **Interview Day**
- Review CHEATSHEET.md (30 minutes)
- Key talking points: idempotency, scalability, monitoring
- Be ready to whiteboard DAG designs
- Prepare 2-3 stories about production Airflow experience

---

## 💡 Common Interview Questions

### **Conceptual Questions**
1. **"What is Airflow and why use it?"**
   - Workflow orchestration platform
   - Programmatic DAG definition (Python)
   - Rich UI for monitoring
   - Scalable and extensible
   - Battle-tested in production

2. **"Explain Airflow architecture"**
   - **Scheduler:** Triggers tasks, monitors DAGs
   - **Executor:** Determines how tasks run
   - **Webserver:** UI for monitoring
   - **Metadata Database:** Stores state
   - **Workers:** Execute tasks (Celery/Kubernetes)

3. **"How does Airflow scheduling work?"**
   - Based on `start_date` + `schedule_interval`
   - Execution date = start of interval
   - Scheduler creates DagRuns
   - Tasks queued based on dependencies

### **Scenario Questions**
1. **"Design a DAG for daily incremental data load"**
   - See Q100 in 100-QUESTIONS.md
   - Key: Use execution_date for watermarking
   - Implement idempotency
   - Add data quality checks

2. **"How to handle task failures?"**
   - Retries (retry_delay, max_retries)
   - Alerts (email, Slack, PagerDuty)
   - On-failure callbacks
   - Manual vs automatic recovery

3. **"Scale Airflow for 1000s of DAGs?"**
   - Use Celery or Kubernetes executor
   - Optimize DAG parsing (reduce top-level code)
   - Separate DAG folder for different teams
   - Use pools to limit concurrency
   - Monitor scheduler performance

---

## 🔗 Useful Resources

### **Official Documentation**
- [Apache Airflow Docs](https://airflow.apache.org/docs/)
- [Best Practices](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html)
- [Airflow Concepts](https://airflow.apache.org/docs/apache-airflow/stable/concepts/)

### **Community**
- [Airflow Slack](https://apache-airflow-slack.herokuapp.com/)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/airflow)
- [GitHub Issues](https://github.com/apache/airflow/issues)

### **Tutorials**
- [Astronomer Guides](https://www.astronomer.io/guides/)
- [Airflow Summit Videos](https://www.youtube.com/c/Airflow)

---

## 🎯 Key Takeaways

### **What Interviewers Look For**
✅ Understanding of DAG design principles
✅ Knowledge of different operators and when to use them
✅ Experience with production Airflow (scaling, monitoring)
✅ Debugging skills (reading logs, understanding failures)
✅ Best practices (idempotency, retries, SLAs)

### **Red Flags to Avoid**
❌ Not understanding execution_date vs current time
❌ Putting heavy logic in DAG file top-level code
❌ Not implementing retries or error handling
❌ No monitoring or alerting strategy
❌ Treating Airflow as a data processing engine (use Spark instead)

---

## 📝 Quick Reference Card

```python
# Basic DAG
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'data-team',
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'email_on_failure': True,
    'email': ['alerts@company.com']
}

with DAG(
    dag_id='example_dag',
    default_args=default_args,
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
    catchup=False,
    tags=['example']
) as dag:

    task1 = PythonOperator(
        task_id='extract',
        python_callable=extract_data
    )

    task2 = PythonOperator(
        task_id='transform',
        python_callable=transform_data
    )

    task3 = PythonOperator(
        task_id='load',
        python_callable=load_data
    )

    task1 >> task2 >> task3  # Dependencies
```

**Schedule Intervals:**
- `@once` - Run once
- `@hourly` - Every hour
- `@daily` - Every day at midnight
- `@weekly` - Every Sunday
- `@monthly` - First day of month
- `0 10 * * *` - Daily at 10 AM (cron)

**Common Operators:**
- `PythonOperator` - Run Python function
- `BashOperator` - Run bash command
- `EmailOperator` - Send email
- `HttpSensor` - Wait for HTTP endpoint
- `ExternalTaskSensor` - Wait for another DAG

---

**Good luck with your Airflow interviews! You've got this! 🚀**

*For detailed answers and examples, see 100-QUESTIONS.md*
*For quick review before interviews, see CHEATSHEET.md*
