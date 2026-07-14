# dbt (data build tool) - High Priority

**Why This Matters:** Industry standard for data transformations. Mentioned in 70% of modern data engineer job postings.

**Study Time:** Week 2 (Day 1-2) - 4-6 hours

**Good News:** You already know SQL and data modeling - dbt just adds structure and best practices!

---

## What is dbt?

**Definition:** dbt transforms data in your warehouse using SELECT statements. It's "analytics engineering" - bringing software engineering practices to data transformations.

**Core Idea:**t
```
Raw Data → dbt (SQL + tests + docs) → Analytics-Ready Tables
```

**What dbt does:**
- Transforms data using SQL
- Tests data quality
- Documents data models
- Manages dependencies (DAG)
- Version controls transformations

 
**What dbt does NOT do:**
- Extract or load data (E and L of ETL)
- Orchestrate pipelines (use Airflow for that)
- Replace your data warehouse

**You already have:** SQL skills, data modeling experience, Airflow
**dbt adds:** Structure, testing, documentation, version control for transformations

---

## Why Companies Want dbt

**Traditional Approach (What you might be doing):**
- SQL scripts scattered everywhere
- No clear dependencies
- Manual testing
- Documentation outdated or missing

**dbt Approach:**
- All transformations in code (version controlled)
- Auto-generated documentation
- Built-in testing
- Clear lineage and dependencies

**Result:** Faster development, better quality, easier collaboration

---

## Core Concepts (Must Know for Interviews)

### 1. Models
**What:** SQL SELECT statements that create tables/views

```sql
-- models/staging/stg_customers.sql
SELECT
    customer_id,
    LOWER(email) as email,
    created_at
FROM {{ source('raw', 'customers') }}
WHERE deleted_at IS NULL
```

**Materialization Options:**
- **view:** Creates view (default, fast builds, slow queries)
- **table:** Creates table (slower builds, fast queries)
- **incremental:** Only processes new data (efficient for large tables)
- **ephemeral:** CTE, not materialized

### 2. Sources
**What:** Raw data tables in your warehouse

```yaml
# models/staging/sources.yml
sources:
  - name: raw
    tables:
      - name: customers
      - name: orders
```

### 3. Tests
**What:** Data quality assertions

```yaml
# models/staging/stg_customers.yml
models:
  - name: stg_customers
    columns:
      - name: customer_id
        tests:
          - unique
          - not_null
      - name: email
        tests:
          - unique
```

**Built-in tests:** unique, not_null, accepted_values, relationships
**Custom tests:** Write your own SQL

### 4. Documentation
**What:** Auto-generated docs from descriptions

```yaml
models:
  - name: stg_customers
    description: "Cleaned customer data from raw source"
    columns:
      - name: customer_id
        description: "Unique identifier for customer"
```

**Result:** `dbt docs generate` creates beautiful website with lineage graphs

### 5. Macros
**What:** Reusable Jinja templates (like functions)

```sql
-- macros/cents_to_dollars.sql
{% macro cents_to_dollars(column_name) %}
    ({{ column_name }} / 100.0)::decimal(10,2)
{% endmacro %}

-- Use in model:
SELECT {{ cents_to_dollars('price_cents') }} as price_dollars
FROM orders
```

---

## Study Resources (6 hours total)

### Day 1: Fundamentals (3 hours)

**Priority 1 - Course:**
- [dbt Fundamentals] Official free course (2 hours)
  https://courses.getdbt.com/courses/fundamentals

**Priority 2 - Read:**
- [Docs] "Introduction to dbt" (30 min)
  https://docs.getdbt.com/docs/introduction

- [Article] "What is dbt?" by dbt Labs (30 min)
  https://www.getdbt.com/blog/what-is-dbt

### Day 2: Hands-On Practice (3 hours)

**Project:** Set up dbt with sample data

**Steps:**
1. Install dbt (30 min)
   ```bash
   pip install dbt-core dbt-postgres  # or dbt-snowflake, dbt-databricks
   dbt init my_project
   ```

2. Set up profiles.yml with local Postgres or DuckDB (30 min)

3. Create 3-5 models (1 hour):
   - Staging models (clean raw data)
   - Intermediate models (business logic)
   - Mart models (final analytics tables)

4. Add tests and documentation (30 min)

5. Run `dbt docs generate` and explore (30 min)

---

## Hands-On Project

### Project: E-Commerce Data Transformation

**Scenario:** Transform raw orders, customers, products into analytics-ready tables

**Raw Tables:**
- raw_customers (id, email, created_at)
- raw_orders (id, customer_id, order_date, status)
- raw_order_items (id, order_id, product_id, quantity, price)

**Your dbt Project:**

```
models/
├── staging/
│   ├── stg_customers.sql
│   ├── stg_orders.sql
│   └── stg_order_items.sql
├── intermediate/
│   └── int_customer_orders.sql
└── marts/
    └── fct_customer_metrics.sql
```

**Example Model - fct_customer_metrics.sql:**
```sql
WITH customer_orders AS (
    SELECT * FROM {{ ref('int_customer_orders') }}
)

SELECT
    customer_id,
    COUNT(DISTINCT order_id) as total_orders,
    SUM(order_total) as lifetime_value,
    MIN(order_date) as first_order_date,
    MAX(order_date) as last_order_date
FROM customer_orders
GROUP BY customer_id
```

**Add Tests:**
```yaml
models:
  - name: fct_customer_metrics
    tests:
      - dbt_utils.unique_combination_of_columns:
          combination_of_columns:
            - customer_id
    columns:
      - name: total_orders
        tests:
          - not_null
          - dbt_utils.expression_is_true:
              expression: ">= 0"
```

**Time:** 3-4 hours

---

## Interview Preparation

### Common Interview Questions:

**1. "What is dbt and why use it?"**

**Answer:**
dbt is a transformation tool that enables analytics engineers to transform data using SQL with software engineering best practices. It adds:
- Version control for transformations
- Automated testing for data quality
- Auto-generated documentation
- Dependency management (DAG)
- Reusable code (macros)

**Why use it:**
- Faster development (reusable code, clear structure)
- Better quality (built-in testing)
- Better collaboration (version control, docs)
- Easier maintenance (clear dependencies)

**Optum example:** "Instead of scattered SQL scripts for RQNS transformations, dbt would give us version-controlled models with tests and documentation."

---

**2. "Explain dbt project structure and best practices"**

**Answer:**
```
models/
├── staging/       # Clean raw data, 1:1 with sources
├── intermediate/  # Business logic, reusable pieces
└── marts/        # Final analytics tables (facts, dimensions)
```

**Best Practices:**
- **Staging:** Clean column names, basic filtering, no joins
- **Intermediate:** Reusable logic, can join tables
- **Marts:** Final tables for BI tools, organized by department
- **One model = one file**
- **Ref() for dependencies** (never hardcode table names)
- **Test everything** (especially primary keys and foreign keys)

---

**3. "What's the difference between dbt and traditional ETL tools?"**

**Answer:**

| Aspect | Traditional ETL | dbt |
|--------|----------------|-----|
| Language | GUI or Python | SQL |
| What it does | Extract, Transform, Load | Transform only (T) |
| Runs where | Separate server | Inside data warehouse |
| Version control | Often missing | Git-based |
| Testing | Manual or custom | Built-in |
| Docs | Manual | Auto-generated |

**dbt Philosophy:** "Push transformation logic into the warehouse where data already lives, using SQL that analysts know."

---

**4. "How do you handle incremental models in dbt?"**

**Answer:**
Incremental models only process new/changed data to save time and cost.

```sql
{{
    config(
        materialized='incremental',
        unique_key='order_id'
    )
}}

SELECT *
FROM {{ source('raw', 'orders') }}

{% if is_incremental() %}
    WHERE updated_at > (SELECT MAX(updated_at) FROM {{ this }})
{% endif %}
```

**When to use:**
- Large fact tables that grow over time
- Event data (logs, clickstream)
- When full refresh is too slow/expensive

**Trade-offs:**
- More complex logic
- Need unique_key for updates
- Harder to backfill

---

**5. "How would you integrate dbt with your current Spark/Databricks workflow?"**

**Answer:**
Use dbt-databricks or dbt-spark adapter:

**Architecture:**
```
Kafka → Spark Streaming → Delta Lake (Bronze/Silver)
         ↓
    dbt (SQL transformations)
         ↓
    Delta Lake (Gold layer) → BI Tools
```

**Integration Points:**
- dbt runs on Databricks cluster (serverless or job cluster)
- Orchestrated by Airflow:
  ```python
  spark_job >> dbt_run >> dbt_test >> bi_refresh
  ```
- dbt reads from Delta tables, writes back to Delta
- Use incremental models to process only new partitions

**Benefits:**
- dbt handles transformations (better than notebooks)
- Tests ensure data quality
- Docs for downstream users
- Version control for business logic

**Optum example:** "For RQNS, dbt would handle Silver → Gold transformations with tests, while Spark handles Bronze → Silver streaming."

---

## dbt + Your Existing Skills

**How dbt complements what you know:**

| Your Skill | How dbt Enhances It |
|-----------|-------------------|
| **SQL** | Adds structure, testing, docs |
| **Data Modeling** | Formalizes dimensional modeling with marts |
| **Airflow** | dbt task in DAG, better separation of concerns |
| **Spark/Databricks** | dbt-spark for transformations, Spark for heavy lifting |
| **Azure/Cloud** | dbt Cloud or self-hosted on AKS |
| **CI/CD** | dbt test in pipeline, slim CI for PR testing |

---

## Quick Reference Cheat Sheet

### Essential Commands:
```bash
dbt run              # Run all models
dbt run --select stg_customers  # Run specific model
dbt test             # Run all tests
dbt docs generate    # Generate documentation
dbt docs serve       # View docs in browser
dbt build            # Run + test + snapshot
```

### Jinja Essentials:
```sql
{{ ref('model_name') }}              # Reference another model
{{ source('source_name', 'table') }} # Reference raw table
{{ config(materialized='table') }}   # Configure model

{% if is_incremental() %}            # Incremental logic
{% endif %}
```

### Testing:
```yaml
tests:
  - unique
  - not_null
  - accepted_values:
      values: ['pending', 'completed', 'cancelled']
  - relationships:
      to: ref('customers')
      field: customer_id
```

---

## Practice Problems

**Problem 1:** Create dimensional model (2 hours)
- Build fact_orders and dim_customers
- Add tests for referential integrity
- Generate documentation

**Problem 2:** Incremental model (1.5 hours)
- Create incremental fact table for events
- Handle late-arriving data
- Test uniqueness

**Problem 3:** Custom test (1 hour)
- Write SQL test for "revenue must be positive"
- Apply to multiple models
- Run dbt test

---

## Week 2 Success Metrics

**Knowledge:**
- [ ] Understand what dbt is and why it's used
- [ ] Know model materialization types
- [ ] Familiar with ref() and source()
- [ ] Can explain staging → intermediate → marts structure

**Hands-On:**
- [ ] Installed dbt locally
- [ ] Created 3+ models
- [ ] Added tests and documentation
- [ ] Ran dbt docs generate

**Interview Ready:**
- [ ] Can compare dbt vs traditional ETL
- [ ] Explain incremental models
- [ ] Describe how to integrate dbt with Spark/Databricks
- [ ] Have sample dbt project on GitHub

---

**Resume Addition:**
```
• Implemented dbt for data transformations with automated testing and
  documentation, improving code maintainability and data quality
```

**Next:** Integrate dbt into your portfolio project with Databricks
