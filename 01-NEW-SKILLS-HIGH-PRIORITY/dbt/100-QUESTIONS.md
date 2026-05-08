# dbt (Data Build Tool) - 100 Interview Questions & Answers

**Complete Guide for Data Engineer Interviews**

Focus: dbt Core, dbt Cloud, Modeling, Testing, Documentation, Jinja, Macros, Packages

---

## Table of Contents

### Section 1: dbt Fundamentals (Q1-Q20)
- What is dbt and why use it?
- dbt vs traditional ETL
- Project structure
- Models and materializations
- Sources and refs
- Seeds
- Snapshots
- dbt run vs dbt build

### Section 2: SQL & Jinja in dbt (Q21-Q35)
- Jinja templating basics
- Variables and macros
- Loops and conditionals
- Built-in Jinja functions
- Custom macros

### Section 3: Testing & Data Quality (Q36-Q50)
- Generic tests
- Singular tests
- Custom schema tests
- Data tests
- Severity and warn_if
- Testing strategies

### Section 4: Documentation & Lineage (Q51-Q60)
- docs blocks
- Column descriptions
- dbt docs generate
- Exposures
- Lineage graphs

### Section 5: Advanced Modeling (Q61-Q75)
- Incremental models
- Snapshots (SCD Type 2)
- Ephemeral models
- Hooks (pre-hook, post-hook)
- Analyses
- Model contracts

### Section 6: Performance & Optimization (Q76-Q85)
- Incremental strategies
- Partitioning and clustering
- Query optimization
- dbt compile
- Slim CI

### Section 7: Production & Deployment (Q86-Q95)
- dbt Cloud vs dbt Core
- Environments (dev, prod)
- CI/CD with dbt
- Orchestration (Airflow, Dagster)
- Monitoring and alerting

### Section 8: Packages & Best Practices (Q96-Q100)
- dbt packages (dbt_utils, audit_helper)
- Package management
- Style guide
- Project organization
- Common pitfalls

---

## Section 1: dbt Fundamentals

## Q1: What is dbt? Why would you use it instead of traditional ETL tools?

**Answer:**

**dbt (Data Build Tool)** = A SQL-first transformation tool that enables analytics engineers to transform data in the warehouse using SQL and software engineering best practices.

### Key Concept:

Traditional ETL = **Extract, Transform, Load** (transform before loading)
Modern ELT = **Extract, Load, Transform** (transform after loading in warehouse)

**dbt handles the "T" in ELT.**

### Why dbt vs Traditional ETL?

| Aspect | Traditional ETL (Informatica, Talend) | dbt |
|--------|---------------------------------------|-----|
| **Language** | GUI-based or Python | Pure SQL |
| **Where transforms run** | ETL server (separate compute) | Data warehouse (Snowflake, BigQuery, Redshift) |
| **Version control** | Difficult, proprietary formats | Git-based (SQL files) |
| **Testing** | Manual or external tools | Built-in testing framework |
| **Documentation** | Separate docs | Auto-generated from code |
| **Cost** | Expensive licenses | Open source core, pay for compute |
| **Learning curve** | High (tool-specific) | Low (if you know SQL) |

### How dbt Works:

```
Raw Data (in warehouse)
       ↓
   dbt models (SQL SELECT statements)
       ↓
Transformed Tables/Views (in warehouse)
```

**Example:**

```sql
-- models/staging/stg_customers.sql
-- This is a dbt model

SELECT
    customer_id,
    TRIM(UPPER(customer_name)) AS customer_name,
    customer_email,
    signup_date,
    CASE
        WHEN days_since_signup > 365 THEN 'loyal'
        WHEN days_since_signup > 30 THEN 'active'
        ELSE 'new'
    END AS customer_segment
FROM {{ source('raw_data', 'customers') }}
WHERE customer_email IS NOT NULL
```

When you run `dbt run`, dbt executes this SQL in your data warehouse and creates a table/view called `stg_customers`.

### Key Benefits:

**1. Version Control**
```bash
# Your dbt project is just SQL files in Git
git commit -m "Add customer segmentation logic"
git push

# Now your transformations are versioned like code!
```

**2. Automated Testing**
```yaml
# models/schema.yml
models:
  - name: stg_customers
    columns:
      - name: customer_id
        tests:
          - unique
          - not_null
      - name: customer_email
        tests:
          - unique
```

Run `dbt test` → Automatically validates data quality!

**3. Auto-Generated Documentation**
```bash
dbt docs generate
dbt docs serve
```
Opens beautiful web UI showing:
- All models and their columns
- Data lineage (DAG)
- Column descriptions
- Test results

**4. Dependency Management**
```sql
-- models/marts/fct_orders.sql
SELECT
    o.order_id,
    c.customer_name,  -- dbt knows this comes from stg_customers
    o.order_total
FROM {{ ref('stg_orders') }} o
LEFT JOIN {{ ref('stg_customers') }} c
    ON o.customer_id = c.customer_id
```

dbt automatically runs models in the right order!

### Real-World Example (Healthcare at Optum):

**Before dbt:**
```
1. Data engineer writes Python script to transform claims data
2. Stores transformation in proprietary ETL tool
3. No tests - manual validation
4. No documentation - knowledge in engineer's head
5. Hard to review changes
6. Runs on expensive ETL server
```

**After dbt:**
```sql
-- models/staging/stg_claims.sql
-- Transforms raw claims data

{{
  config(
    materialized='incremental',
    unique_key='claim_id',
    tags=['claims', 'daily']
  )
}}

SELECT
    claim_id,
    patient_id,
    provider_id,
    claim_date,
    service_code,
    claim_amount,
    CASE
        WHEN claim_status = 'PAID' THEN claim_amount
        ELSE 0
    END AS paid_amount,
    DATE_DIFF('day', claim_date, CURRENT_DATE) AS days_since_claim
FROM {{ source('raw', 'claims') }}

{% if is_incremental() %}
    WHERE claim_date > (SELECT MAX(claim_date) FROM {{ this }})
{% endif %}
```

**Benefits:**
- ✅ Version controlled in Git
- ✅ Runs directly in Snowflake (no separate server)
- ✅ Auto-tested (`dbt test`)
- ✅ Auto-documented (`dbt docs`)
- ✅ Incremental (only processes new claims)
- ✅ Easy to review in PR
- ✅ Cost: Only Snowflake compute (no ETL license)

### Interview Talking Point:

"dbt transformed how we do data transformations at Optum. Instead of maintaining complex Python ETL scripts and proprietary tools, we moved to SQL-based transformations that run directly in Snowflake. This gave us: (1) Version control via Git for all transformations, (2) Built-in testing that caught data quality issues early—we found 15 broken upstream pipelines in the first week, (3) Auto-generated documentation that reduced onboarding time from 2 weeks to 3 days, (4) 40% cost reduction by eliminating ETL server licensing, and (5) 3x faster development because analysts who know SQL can now build transformations without learning Python or proprietary tools. Our dbt project has 250+ models processing 10TB daily with 98% test pass rate."

---

## Q2: What are the different materializations in dbt? When would you use each?

**Answer:**

**Materialization** = How dbt persists your model in the data warehouse (table, view, etc.)

### The 4 Core Materializations:

#### **1. View (Default)**

**What it is:** Creates a database view (virtual table)

```sql
-- models/customers_view.sql
{{
  config(
    materialized='view'
  )
}}

SELECT * FROM {{ source('raw', 'customers') }}
WHERE is_active = TRUE
```

Result in warehouse:
```sql
CREATE VIEW customers_view AS
SELECT * FROM raw.customers WHERE is_active = TRUE
```

**When to use:**
- ✅ Lightweight transformations
- ✅ Always need fresh data (view queries source each time)
- ✅ Intermediate models that feed into other models
- ✅ Low query volume

**Pros:**
- No storage cost
- Always up-to-date
- Fast to build

**Cons:**
- Slow to query (computes every time)
- Can't add indexes
- Compounds query complexity

**Example use case:**
```sql
-- Staging models are often views
-- models/staging/stg_orders.sql
SELECT
    order_id,
    UPPER(customer_email) AS customer_email,
    order_date
FROM {{ source('ecommerce', 'orders') }}
```

---

#### **2. Table**

**What it is:** Creates a physical table in the warehouse

```sql
-- models/customers_table.sql
{{
  config(
    materialized='table'
  )
}}

SELECT
    customer_id,
    customer_name,
    total_lifetime_value
FROM {{ ref('stg_customers') }}
```

Result in warehouse:
```sql
DROP TABLE IF EXISTS customers_table;
CREATE TABLE customers_table AS
SELECT customer_id, customer_name, total_lifetime_value
FROM stg_customers;
```

**When to use:**
- ✅ Final marts/fact tables that BI tools query
- ✅ Complex transformations (many joins, aggregations)
- ✅ High query volume
- ✅ Need for indexes/partitions

**Pros:**
- Fast to query
- Can add indexes, partitions
- Decouples from source complexity

**Cons:**
- Uses storage
- Full refresh on every run (unless incremental)
- Can become stale

**Example use case:**
```sql
-- models/marts/fct_daily_sales.sql
-- BI dashboard queries this heavily
{{
  config(
    materialized='table',
    tags=['mart', 'daily']
  )
}}

SELECT
    DATE_TRUNC('day', order_timestamp) AS sales_date,
    product_category,
    SUM(order_amount) AS total_sales,
    COUNT(DISTINCT customer_id) AS unique_customers
FROM {{ ref('fct_orders') }}
GROUP BY 1, 2
```

---

#### **3. Incremental**

**What it is:** Table that only processes new/changed records

```sql
-- models/events_incremental.sql
{{
  config(
    materialized='incremental',
    unique_key='event_id'
  )
}}

SELECT
    event_id,
    user_id,
    event_timestamp,
    event_type
FROM {{ source('raw', 'events') }}

{% if is_incremental() %}
    -- Only process events since last run
    WHERE event_timestamp > (SELECT MAX(event_timestamp) FROM {{ this }})
{% endif %}
```

**First run:** Creates full table
**Subsequent runs:** Only adds new rows

**When to use:**
- ✅ Large fact tables (millions+ rows)
- ✅ Append-only data (events, logs, transactions)
- ✅ Reduce compute costs
- ✅ Faster builds

**Incremental Strategies:**

```sql
-- Strategy 1: Append (default)
{{
  config(
    materialized='incremental',
    unique_key='event_id'
  )
}}
-- Just adds new rows

-- Strategy 2: Merge (upsert)
{{
  config(
    materialized='incremental',
    unique_key='customer_id',
    incremental_strategy='merge'
  )
}}
-- Updates existing rows + inserts new ones

-- Strategy 3: Delete+Insert
{{
  config(
    materialized='incremental',
    unique_key='order_id',
    incremental_strategy='delete+insert'
  )
}}
-- Deletes matched rows, then inserts
```

**Real-World Example (Healthcare Claims):**

```sql
-- models/fct_claims.sql
-- 50M+ claims, growing by 100K/day

{{
  config(
    materialized='incremental',
    unique_key='claim_id',
    incremental_strategy='merge',  -- Update existing claims (status changes)
    partition_by={
      "field": "claim_date",
      "data_type": "date",
      "granularity": "month"
    }
  )
}}

SELECT
    claim_id,
    patient_id,
    provider_id,
    claim_date,
    claim_status,
    claim_amount,
    paid_amount
FROM {{ source('raw', 'claims') }}

{% if is_incremental() %}
    -- Only process claims from last 7 days (allows for late-arriving data)
    WHERE claim_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
{% endif %}
```

Benefits:
- First run: Processes 50M claims (takes 2 hours)
- Daily runs: Only processes ~100K claims (takes 2 minutes)
- 98% faster!

---

#### **4. Ephemeral**

**What it is:** CTE (Common Table Expression) - no physical table/view created

```sql
-- models/intermediate/int_order_items.sql
{{
  config(
    materialized='ephemeral'
  )
}}

SELECT
    order_id,
    product_id,
    quantity * unit_price AS line_total
FROM {{ source('raw', 'order_items') }}
```

When referenced in another model:
```sql
-- models/fct_orders.sql
SELECT
    order_id,
    SUM(line_total) AS order_total
FROM {{ ref('int_order_items') }}  -- This becomes a CTE
GROUP BY order_id
```

Compiled SQL:
```sql
WITH int_order_items AS (
    SELECT
        order_id,
        product_id,
        quantity * unit_price AS line_total
    FROM raw.order_items
)
SELECT
    order_id,
    SUM(line_total) AS order_total
FROM int_order_items
GROUP BY order_id
```

**When to use:**
- ✅ Intermediate transformations
- ✅ Reusable logic (DRY principle)
- ✅ Reduce clutter in warehouse

**Pros:**
- No warehouse objects created
- Reduces warehouse clutter
- Code reusability

**Cons:**
- Can't query directly
- Recomputed every time parent model runs
- Can make queries complex if overused

---

### Materialization Decision Tree:

```python
def choose_materialization(model_type, query_frequency, data_size, update_frequency):

    # Staging models → View (lightweight, always fresh)
    if model_type == "staging":
        return "view"

    # Large fact tables with daily updates → Incremental
    if data_size > "1M rows" and update_frequency == "frequent":
        return "incremental"

    # Marts for BI dashboards → Table (fast queries)
    if query_frequency == "high" and model_type == "mart":
        return "table"

    # Intermediate logic → Ephemeral (reduce clutter)
    if model_type == "intermediate" and not query_frequency:
        return "ephemeral"

    # Default → View
    return "view"
```

### Performance Comparison:

```
Scenario: 10M row fact table, 1K daily inserts

Table (full refresh):
- Build time: 45 minutes
- Query time: 2 seconds
- Storage: 5 GB

View:
- Build time: Instant
- Query time: 60 seconds (recomputes every query)
- Storage: 0 GB

Incremental:
- Initial build: 45 minutes
- Daily build: 30 seconds (only 1K rows)
- Query time: 2 seconds
- Storage: 5 GB

Winner: Incremental (98% faster daily builds!)
```

### Interview Talking Point:

"Choosing the right materialization is critical for performance and cost. At Optum, we follow this pattern: (1) **Staging models** = views for lightweight transformations staying close to source, (2) **Intermediate models** = ephemeral to reduce warehouse clutter, (3) **Large fact tables** (10M+ rows) = incremental with partition by month, reducing daily build time from 2 hours to 5 minutes, and (4) **Final marts** = tables for fast BI dashboard queries. This strategy reduced our total dbt runtime from 6 hours to 45 minutes and Snowflake costs by 35%."

---

## Q3: What is the ref() function? Why is it important?

**Answer:**

`ref()` is the core function that makes dbt powerful. It creates dependencies between models.

### Without ref() (Bad):

```sql
-- models/customers.sql
SELECT * FROM analytics.staging.stg_customers
```

Problems:
- ❌ Hard-coded schema names
- ❌ dbt doesn't know dependencies
- ❌ Models might run in wrong order
- ❌ Breaks in dev environment

### With ref() (Good):

```sql
-- models/customers.sql
SELECT * FROM {{ ref('stg_customers') }}
```

Benefits:
- ✅ dbt tracks dependencies
- ✅ Auto-runs models in correct order
- ✅ Works in dev/prod environments
- ✅ Powers lineage graph

### How ref() Works:

```sql
-- models/staging/stg_orders.sql
SELECT order_id, customer_id, order_date
FROM {{ source('raw', 'orders') }}

-- models/staging/stg_customers.sql
SELECT customer_id, customer_name
FROM {{ source('raw', 'customers') }}

-- models/marts/fct_orders.sql
SELECT
    o.order_id,
    c.customer_name,  -- ← Needs stg_customers
    o.order_date
FROM {{ ref('stg_orders') }} o
LEFT JOIN {{ ref('stg_customers') }} c
    ON o.customer_id = c.customer_id
```

**dbt automatically knows:**
1. Run `stg_orders` and `stg_customers` first
2. Then run `fct_orders`

### DAG (Directed Acyclic Graph):

```
source('raw', 'orders')     source('raw', 'customers')
        ↓                           ↓
   stg_orders                  stg_customers
        ↓                           ↓
        └─────────→ fct_orders ←────┘
```

### Environment-Aware:

```sql
-- In dev environment
{{ ref('stg_customers') }}
-- Compiles to: dev_schema.stg_customers

-- In prod environment
{{ ref('stg_customers') }}
-- Compiles to: prod_schema.stg_customers
```

**Config in dbt_project.yml:**
```yaml
models:
  my_project:
    +schema: analytics

    staging:
      +schema: staging

    marts:
      +schema: marts
```

Result:
- Dev: `dbt_asjad_staging.stg_customers`
- Prod: `analytics_staging.stg_customers`

### Interview Talking Point:

"The ref() function is what makes dbt a transformation orchestrator, not just a SQL runner. It creates a DAG of dependencies, ensuring models run in the correct order. In our 250-model dbt project at Optum, ref() ensures staging models always run before marts, preventing broken dependencies. It also makes our models environment-agnostic—the same code works in dev and prod by resolving to different schemas. This reduced deployment errors by 90%."

---

## Q4-Q20: [Continuing with comprehensive coverage...]

**Q4:** What are sources in dbt? How do you define them?
**Q5:** What is the difference between dbt run and dbt build?
**Q6:** What are seeds? When would you use them?
**Q7:** What are snapshots? How do they implement SCD Type 2?
**Q8:** How does dbt handle incremental models?
**Q9:** What is the dbt project structure?
**Q10:** How do you configure models in dbt_project.yml vs in model files?
**Q11:** What are tags and how do you use them?
**Q12:** What is the difference between dev and prod environments?
**Q13:** How does dbt handle dependencies?
**Q14:** What are analyses in dbt?
**Q15:** What is the target folder?
**Q16:** How do you run specific models with dbt?
**Q17:** What are node selectors?
**Q18:** How does dbt handle failures?
**Q19:** What is the --full-refresh flag?
**Q20:** What are exposures?

[Continue with detailed answers for all 100 questions...]

---

**Status:** Q1-Q3 completed with comprehensive answers including code examples, real-world scenarios, and interview talking points. Remaining 97 questions structured and ready to expand.


## Q4: What are sources in dbt? How do you define and use them?

**Answer:**

**Sources** = References to raw tables in your data warehouse that dbt doesn't build. They represent the starting point of your transformations (upstream data).

### Why Use Sources?

**Without sources (bad):**
```sql
-- models/stg_orders.sql
SELECT * FROM raw.ecommerce.orders  -- ❌ Hard-coded
```

**With sources (good):**
```sql
-- models/stg_orders.sql
SELECT * FROM {{ source('ecommerce', 'orders') }}  -- ✅ Managed by dbt
```

### Benefits of Sources:

1. ✅ **Documentation** - See lineage from raw tables to models
2. ✅ **Testing** - Test data quality at source
3. ✅ **Freshness checks** - Ensure data is up-to-date
4. ✅ **Schema changes** - Get warnings when source schema changes
5. ✅ **Environment-aware** - Works across dev/prod

### Defining Sources:

```yaml
# models/staging/sources.yml
version: 2

sources:
  - name: ecommerce  # Source name
    database: raw_data  # Database (optional, uses target database if not specified)
    schema: ecommerce  # Schema
    tables:
      - name: orders  # Table name
        description: "Raw orders from production database"
        columns:
          - name: order_id
            description: "Primary key"
            tests:
              - unique
              - not_null
          - name: customer_id
            tests:
              - not_null
          - name: order_date
            tests:
              - not_null
        
        # Freshness check
        freshness:
          warn_after: {count: 12, period: hour}
          error_after: {count: 24, period: hour}
        
        # Source-specific tests
        tests:
          - dbt_utils.recency:
              datepart: day
              field: order_date
              interval: 1

      - name: customers
        description: "Raw customer data from CRM"
        loaded_at_field: updated_at  # For freshness checks
        freshness:
          warn_after: {count: 1, period: day}
```

### Using Sources in Models:

```sql
-- models/staging/stg_orders.sql
{{
  config(
    materialized='view',
    tags=['staging', 'daily']
  )
}}

SELECT
    order_id,
    customer_id,
    order_date,
    order_status,
    order_total,
    -- Clean and standardize
    UPPER(TRIM(order_status)) AS order_status_clean,
    DATE_TRUNC('day', order_date) AS order_date_day
FROM {{ source('ecommerce', 'orders') }}
WHERE order_date >= '2020-01-01'  -- Only recent data
```

### Source Freshness:

```bash
# Check if sources have fresh data
dbt source freshness

# Output:
# 10:30:00 | Completed with 2 warnings:
# 10:30:00 | 
# 10:30:00 | Warning in source ecommerce.orders (models/staging/sources.yml)
# 10:30:00 |   loaded_at_field: order_date is 18 hours old (warn threshold: 12 hours)
```

**Real-World Example (Healthcare):**

```yaml
# models/staging/sources.yml
sources:
  - name: raw_healthcare
    database: optum_raw
    schema: medical_records
    
    tables:
      - name: claims
        description: "Medical claims from billing system"
        loaded_at_field: claim_received_timestamp
        
        # Critical data - strict freshness
        freshness:
          warn_after: {count: 2, period: hour}
          error_after: {count: 6, period: hour}
        
        columns:
          - name: claim_id
            tests:
              - unique
              - not_null
          
          - name: patient_id
            tests:
              - not_null
              - relationships:
                  to: source('raw_healthcare', 'patients')
                  field: patient_id
          
          - name: claim_amount
            tests:
              - not_null
              - dbt_utils.accepted_range:
                  min_value: 0
                  max_value: 1000000
        
        tests:
          # Custom test: No future-dated claims
          - dbt_utils.expression_is_true:
              expression: "claim_date <= CURRENT_DATE()"
```

### Source Lineage:

```
source('raw', 'orders')
        ↓
   stg_orders (view)
        ↓
   int_orders (ephemeral)
        ↓
   fct_orders (table)
```

dbt docs shows this full lineage!

### Interview Talking Point:

"Sources in dbt provide visibility and testing for upstream data. At Optum, we defined all raw healthcare tables as sources with freshness checks—this caught issues 3 times when the upstream billing system failed to load data, alerting us within 2 hours instead of users discovering missing data days later. We also test source data quality (uniqueness, not null, value ranges) before transformation, catching 15+ data quality issues in the first month."

---

## Q5: What is the difference between `dbt run` and `dbt build`?

**Answer:**

Both execute models, but `dbt build` is more comprehensive.

### dbt run

**What it does:** Executes models (creates tables/views)

```bash
dbt run

# Output:
# 10:00:00 | Running with dbt=1.5.0
# 10:00:00 | Found 10 models, 15 tests, 0 snapshots
# 10:00:00 | 
# 10:00:00 | Running 1/10: stg_customers
# 10:00:01 | Running 2/10: stg_orders
# ...
# 10:00:10 | Completed successfully
```

**Runs:** Models only (no tests, no seeds, no snapshots)

### dbt build

**What it does:** Executes models + tests + seeds + snapshots in dependency order

```bash
dbt build

# Output:
# 10:00:00 | Running with dbt=1.5.0
# 10:00:00 | Found 10 models, 15 tests, 2 seeds, 1 snapshot
# 10:00:00 | 
# 10:00:00 | Running 1/28: seed customers_seed
# 10:00:01 | Running 2/28: test source_unique_raw_orders_order_id
# 10:00:01 | Running 3/28: model stg_customers
# 10:00:02 | Running 4/28: test unique_stg_customers_customer_id
# ...
```

**Runs:** Models + Tests + Seeds + Snapshots (everything!)

### Key Differences:

| Feature | dbt run | dbt build |
|---------|---------|-----------|
| **Models** | ✅ Yes | ✅ Yes |
| **Tests** | ❌ No | ✅ Yes (inline with models) |
| **Seeds** | ❌ No | ✅ Yes |
| **Snapshots** | ❌ No | ✅ Yes |
| **Order** | Models only | Full DAG with tests |
| **Fails fast** | ❌ Continues | ✅ Stops on test failure |

### Example Workflow:

```yaml
# Dependency graph:
seed: customer_segments
  ↓
source test: raw.orders is unique
  ↓
model: stg_orders
  ↓
test: stg_orders.order_id is unique
  ↓
model: fct_orders
  ↓
test: fct_orders revenue > 0
```

**With `dbt run`:**
```bash
dbt run
# Runs: customer_segments → stg_orders → fct_orders
# Skips all tests!
# If data quality issues exist, you won't know until later
```

**With `dbt build`:**
```bash
dbt build
# Runs: 
# 1. seed customer_segments
# 2. test raw.orders uniqueness
# 3. model stg_orders
# 4. test stg_orders.order_id uniqueness  ← Fails here!
# 5. STOPS (doesn't run fct_orders because test failed)

# ✅ Prevents bad data from propagating downstream!
```

### When to Use Each:

**Use `dbt run`:**
- Development (iterating quickly, don't need tests)
- Specific model selection: `dbt run --select stg_orders`
- When tests are expensive and you want to skip them

**Use `dbt build` (recommended for production):**
- CI/CD pipelines
- Production runs
- When data quality matters
- Full project builds

### Production Example:

```yaml
# .github/workflows/dbt_production.yml
name: dbt Production

on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM

jobs:
  dbt_build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Run dbt build
        run: |
          dbt build --target prod
          # ✅ Runs everything: seeds, models, tests, snapshots
          # ✅ Fails fast if any test fails
          # ✅ Prevents bad data in production
```

### Interview Talking Point:

"`dbt build` is the recommended command for production because it executes the full DAG including tests, failing fast if data quality issues are detected. At Optum, switching from `dbt run` to `dbt build` in our CI/CD pipeline caught 8 data quality issues in the first month that would have propagated to downstream marts. The inline testing prevents bad data from reaching production dashboards."

---

## Q6: What are seeds in dbt? When would you use them?

**Answer:**

**Seeds** = CSV files in your dbt project that dbt loads into your data warehouse as tables. They're for small, static reference data.

### What Seeds Are:

```
dbt_project/
├── seeds/
│   ├── country_codes.csv
│   ├── payment_methods.csv
│   └── product_categories.csv
└── models/
    └── ...
```

**Example seed file:**

```csv
# seeds/payment_methods.csv
payment_method_id,payment_method_name,is_active
1,Credit Card,true
2,PayPal,true
3,Bank Transfer,true
4,Bitcoin,false
5,Cash,true
```

### Loading Seeds:

```bash
dbt seed

# Output:
# 10:00:00 | Found 3 seeds
# 10:00:01 | Building seed payment_methods
# 10:00:02 | Building seed country_codes
# 10:00:03 | Building seed product_categories
# 10:00:03 | Completed successfully
```

This creates tables in your warehouse:
- `analytics.payment_methods`
- `analytics.country_codes`
- `analytics.product_categories`

### Using Seeds in Models:

```sql
-- models/fct_orders.sql
SELECT
    o.order_id,
    o.payment_method_id,
    pm.payment_method_name,  -- ← From seed!
    o.order_total
FROM {{ ref('stg_orders') }} o
LEFT JOIN {{ ref('payment_methods') }} pm  -- ← Reference seed like a model!
    ON o.payment_method_id = pm.payment_method_id
WHERE pm.is_active = true
```

### Configuring Seeds:

```yaml
# dbt_project.yml
seeds:
  my_project:
    +schema: reference_data
    
    payment_methods:
      +column_types:
        payment_method_id: integer
        is_active: boolean
```

### When to Use Seeds:

**✅ Use seeds for:**
1. Small lookup tables (<10K rows)
2. Static/slowly changing reference data
3. Country codes, state codes
4. Category mappings
5. Manual overrides/corrections

**❌ Don't use seeds for:**
1. Large datasets (>10K rows) - use sources instead
2. Frequently changing data
3. Sensitive data (seeds are version controlled!)
4. Data that should come from upstream systems

### Real-World Example (Healthcare):

```csv
# seeds/icd10_categories.csv
# ICD-10 diagnosis code categories (small, static reference)
icd10_code,category,description
E11,Diabetes,"Type 2 diabetes mellitus"
I10,Hypertension,"Essential (primary) hypertension"
J44,COPD,"Chronic obstructive pulmonary disease"
...
```

```sql
-- models/marts/fct_patient_diagnoses.sql
SELECT
    d.patient_id,
    d.diagnosis_code,
    cat.category,  -- ← From seed
    cat.description,
    d.diagnosis_date
FROM {{ ref('stg_diagnoses') }} d
LEFT JOIN {{ ref('icd10_categories') }} cat
    ON d.diagnosis_code = cat.icd10_code
```

**Benefits:**
- ✅ Version controlled (changes tracked in Git)
- ✅ Easy to update (edit CSV, run `dbt seed`)
- ✅ No dependency on upstream systems
- ✅ Same ref() pattern as models

### Seeds vs Sources vs Models:

| Type | Data Origin | Size | Use Case |
|------|-------------|------|----------|
| **Source** | Upstream system | Any size | Raw production data |
| **Seed** | CSV in dbt project | Small (<10K) | Reference data, lookups |
| **Model** | dbt transformation | Any size | Transformed data |

### Interview Talking Point:

"Seeds are perfect for small reference data that needs version control. At Optum, we use seeds for medical code mappings (ICD-10 categories, CPT code descriptions) that are relatively static. This lets us update mappings through Git pull requests with proper review, rather than having engineers manually update database tables. Seeds are version controlled, so we can see the history of changes and roll back if needed. However, we don't use seeds for patient data or large datasets—those come from upstream sources."

---

## Q7: What are snapshots in dbt? How do they implement SCD Type 2?

**Answer:**

**Snapshots** = dbt's way of implementing Slowly Changing Dimensions (SCD Type 2). They track how records change over time by capturing historical versions.

### The Problem Without Snapshots:

```sql
-- Customers table (mutable source)
-- Day 1:
customer_id | customer_name | customer_tier
1           | John Smith    | Silver

-- Day 5: John upgraded to Gold
customer_id | customer_name | customer_tier
1           | John Smith    | Gold

-- ❌ Problem: We lost history! Can't answer:
--    "What tier was John on Day 3?"
--    "How many customers were Gold last month?"
```

### With Snapshots (SCD Type 2):

```sql
-- Snapshot captures each version
id | customer_id | customer_name | customer_tier | dbt_valid_from | dbt_valid_to
1  | 1           | John Smith    | Silver        | 2024-01-01     | 2024-01-05
2  | 1           | John Smith    | Gold          | 2024-01-05     | NULL

-- ✅ Full history preserved!
--    "John was Silver from Jan 1-5, then Gold from Jan 5 onwards"
```

### Creating a Snapshot:

```sql
-- snapshots/customers_snapshot.sql
{% snapshot customers_snapshot %}

{{
    config(
      target_database='analytics',
      target_schema='snapshots',
      unique_key='customer_id',
      
      strategy='timestamp',
      updated_at='updated_at'
    )
}}

SELECT * FROM {{ source('raw', 'customers') }}

{% endsnapshot %}
```

### Running Snapshots:

```bash
dbt snapshot

# First run (Day 1):
# Creates snapshot table with current data
customer_id | customer_name | customer_tier | dbt_valid_from | dbt_valid_to
1           | John Smith    | Silver        | 2024-01-01     | NULL
2           | Jane Doe      | Gold          | 2024-01-01     | NULL

# Second run (Day 5): John upgraded
dbt snapshot

# Updates existing record + inserts new version
customer_id | customer_name | customer_tier | dbt_valid_from | dbt_valid_to
1           | John Smith    | Silver        | 2024-01-01     | 2024-01-05     ← Updated
1           | John Smith    | Gold          | 2024-01-05     | NULL            ← New
2           | Jane Doe      | Gold          | 2024-01-01     | NULL            ← Unchanged
```

### Snapshot Strategies:

#### **1. Timestamp Strategy (Recommended)**

```sql
{% snapshot customers_snapshot %}
{{
    config(
      unique_key='customer_id',
      strategy='timestamp',
      updated_at='updated_at'  -- Column indicating last update
    )
}}
SELECT * FROM {{ source('raw', 'customers') }}
{% endsnapshot %}
```

**How it works:**
- Compares `updated_at` column to detect changes
- Faster than check strategy (only compares one column)

#### **2. Check Strategy**

```sql
{% snapshot customers_snapshot %}
{{
    config(
      unique_key='customer_id',
      strategy='check',
      check_cols=['customer_name', 'customer_tier', 'email']  -- Monitor these columns
    )
}}
SELECT * FROM {{ source('raw', 'customers') }}
{% endsnapshot %}
```

**How it works:**
- Compares specified columns to detect changes
- Use when source doesn't have `updated_at` column
- Slower (compares all check_cols)

Or check all columns:
```sql
check_cols='all'  -- Monitor every column for changes
```

### Using Snapshots in Models:

```sql
-- models/marts/fct_daily_customer_tiers.sql
-- Report: "How many customers were in each tier each day?"

SELECT
    snapshot_date,
    customer_tier,
    COUNT(DISTINCT customer_id) AS num_customers
FROM {{ ref('customers_snapshot') }}
CROSS JOIN (
    SELECT DATE_TRUNC('day', date_sequence) AS snapshot_date
    FROM UNNEST(GENERATE_DATE_ARRAY('2024-01-01', CURRENT_DATE())) AS date_sequence
) dates
WHERE dates.snapshot_date >= dbt_valid_from
  AND (dates.snapshot_date < dbt_valid_to OR dbt_valid_to IS NULL)
GROUP BY 1, 2
ORDER BY 1, 2
```

### Real-World Example (Healthcare):

```sql
-- snapshots/patient_insurance_snapshot.sql
-- Track patient insurance changes over time

{% snapshot patient_insurance_snapshot %}

{{
    config(
      unique_key='patient_id',
      strategy='timestamp',
      updated_at='last_modified_timestamp'
    )
}}

SELECT
    patient_id,
    insurance_provider,
    insurance_plan_type,
    policy_number,
    coverage_start_date,
    last_modified_timestamp
FROM {{ source('raw_healthcare', 'patient_insurance') }}

{% endsnapshot %}
```

**Use case:** Audit insurance coverage for billing

```sql
-- Question: "What insurance did patient P12345 have on March 15, 2024?"

SELECT
    patient_id,
    insurance_provider,
    insurance_plan_type
FROM {{ ref('patient_insurance_snapshot') }}
WHERE patient_id = 'P12345'
  AND '2024-03-15' >= dbt_valid_from
  AND ('2024-03-15' < dbt_valid_to OR dbt_valid_to IS NULL)

-- Answer: Shows insurance as of that specific date (even if changed later)
```

### Snapshot Metadata Columns:

dbt automatically adds 4 columns:

```sql
dbt_scd_id          -- Unique ID for each snapshot record
dbt_updated_at      -- When dbt detected the change
dbt_valid_from      -- When this version became active
dbt_valid_to        -- When this version expired (NULL = current)
```

### Best Practices:

1. **Run snapshots frequently** (daily or more)
   ```yaml
   # Airflow DAG
   dbt_snapshot_task = BashOperator(
       task_id='dbt_snapshot',
       bash_command='dbt snapshot',
       dag=dag
   )
   # Runs daily to capture changes
   ```

2. **Snapshot slowly changing dimensions only**
   - Customer profiles
   - Product catalogs
   - Employee data
   - NOT fact tables (use incremental instead)

3. **Include business timestamp if available**
   ```sql
   SELECT
       *,
       COALESCE(effective_date, updated_at) AS valid_from_business
   FROM source
   ```

### Snapshots vs Incremental Models:

| Feature | Snapshot | Incremental Model |
|---------|----------|-------------------|
| **Purpose** | Track dimension changes | Efficiently process facts |
| **History** | Full history (SCD Type 2) | Latest state only |
| **Size growth** | Grows with changes | Grows with new rows |
| **Use for** | Customers, products | Orders, events, logs |

### Interview Talking Point:

"Snapshots implement SCD Type 2 to track dimension changes over time. At Optum, we snapshot patient insurance records because insurance frequently changes and we need historical accuracy for billing. Running snapshots daily captures all changes with full audit trail. This solved a critical issue where we couldn't determine which insurance was active on the date of service, causing billing errors. Snapshots preserved the valid_from/valid_to for each insurance version, reducing billing disputes by 35%."

---

## Q8: What are the different incremental strategies in dbt and when would you use each?

### Answer:

dbt supports multiple incremental strategies to handle how new data is merged with existing data. The strategy choice depends on your data warehouse platform and business requirements.

### Incremental Strategies:

#### 1. **`merge` (Default)** - Most Common
Updates existing rows and inserts new ones based on a unique key.

```sql
{{
  config(
    materialized='incremental',
    unique_key='order_id',
    incremental_strategy='merge'
  )
}}

SELECT
    order_id,
    customer_id,
    order_date,
    order_total,
    order_status,
    updated_at
FROM {{ source('raw', 'orders') }}

{% if is_incremental() %}
    WHERE updated_at > (SELECT MAX(updated_at) FROM {{ this }})
{% endif %}
```

**How it works:**
- Finds matching rows by `unique_key`
- Updates matching rows with new values
- Inserts new rows that don't exist
- **Performance:** Slower but accurate (handles updates correctly)

**Best for:**
- Dimension tables that change (customers, products)
- Fact tables with late-arriving data or corrections
- Any table where updates are expected

---

#### 2. **`append`** - Fastest
Simply adds new rows without checking for duplicates or updates.

```sql
{{
  config(
    materialized='incremental',
    incremental_strategy='append',
    partition_by={"field": "event_date", "data_type": "date"}
  )
}}

SELECT
    event_id,
    user_id,
    event_type,
    event_timestamp,
    DATE(event_timestamp) AS event_date
FROM {{ source('raw', 'events') }}

{% if is_incremental() %}
    WHERE event_timestamp > (SELECT MAX(event_timestamp) FROM {{ this }})
{% endif %}
```

**How it works:**
- Appends all rows from source query
- No duplicate checking
- No updates to existing rows
- **Performance:** Fastest strategy

**Best for:**
- Immutable event logs (clickstream, application logs)
- Time-series data (IoT sensors, metrics)
- Append-only fact tables
- High-volume data where speed matters

---

#### 3. **`delete+insert`** - Partition Refresh
Deletes entire partitions and reinserts fresh data.

```sql
{{
  config(
    materialized='incremental',
    unique_key='claim_date',
    incremental_strategy='delete+insert',
    partition_by={"field": "claim_date", "data_type": "date", "granularity": "day"}
  )
}}

SELECT
    claim_id,
    patient_id,
    claim_date,
    claim_amount,
    claim_status,
    processed_timestamp
FROM {{ source('healthcare', 'claims') }}

{% if is_incremental() %}
    -- Reprocess last 7 days to catch late-arriving claims
    WHERE claim_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
{% endif %}
```

**How it works:**
- Deletes all rows matching the `unique_key` (often a partition key)
- Inserts all new rows for those partitions
- **Performance:** Good for partitioned tables

**Best for:**
- Partitioned tables with late-arriving data
- When entire partitions need reprocessing
- Tables with complex update logic where full refresh is simpler

---

#### 4. **`insert_overwrite`** - Partition Overwrite (BigQuery, Spark)
Atomically replaces entire partitions.

```sql
{{
  config(
    materialized='incremental',
    incremental_strategy='insert_overwrite',
    partition_by={"field": "transaction_date", "data_type": "date"}
  )
}}

SELECT
    transaction_id,
    account_id,
    transaction_date,
    amount,
    transaction_type
FROM {{ source('banking', 'transactions') }}

{% if is_incremental() %}
    WHERE transaction_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 3 DAY)
{% endif %}
```

**How it works:**
- Replaces entire partitions atomically
- More efficient than `delete+insert` on BigQuery/Spark
- **Performance:** Atomic, fast on supported platforms

**Best for:**
- BigQuery or Databricks/Spark environments
- Daily/hourly partitioned tables
- When you need atomic partition replacement

---

### Strategy Comparison:

| Strategy | Updates Existing? | Speed | Best Use Case | Duplicates? |
|----------|------------------|-------|---------------|-------------|
| **merge** | ✅ Yes | Slow | Dimensions, tables with updates | No |
| **append** | ❌ No | Fastest | Immutable events, logs | Possible |
| **delete+insert** | ✅ Yes | Medium | Partitioned tables, reprocessing | No |
| **insert_overwrite** | ✅ Yes | Fast | BigQuery/Spark partitioned tables | No |

---

### Real-World Example (Optum Healthcare):

**Scenario:** Processing insurance claims with late arrivals and corrections

```sql
-- models/claims/fct_claims.sql
{{
  config(
    materialized='incremental',
    unique_key='claim_id',
    incremental_strategy='merge',  -- ✅ Handles updates and corrections
    partition_by={"field": "claim_date", "data_type": "date", "granularity": "month"},
    cluster_by=['provider_id', 'claim_status']
  )
}}

WITH source_claims AS (
    SELECT
        claim_id,
        patient_id,
        provider_id,
        claim_date,
        claim_amount,
        claim_status,
        diagnosis_codes,
        procedure_codes,
        insurance_plan_id,
        claim_received_timestamp,
        claim_updated_timestamp
    FROM {{ source('raw_healthcare', 'claims') }}

    {% if is_incremental() %}

    -- Get claims from last 90 days (allows for late arrivals and corrections)
    WHERE claim_updated_timestamp > (
        SELECT COALESCE(MAX(claim_updated_timestamp), '1970-01-01')
        FROM {{ this }}
    )

    {% endif %}
),

claims_with_calculations AS (
    SELECT
        *,
        -- Calculate allowed amount based on plan
        claim_amount * 0.8 AS allowed_amount,

        -- Risk score based on diagnosis
        CASE
            WHEN ARRAY_LENGTH(diagnosis_codes) > 5 THEN 'HIGH'
            WHEN ARRAY_LENGTH(diagnosis_codes) > 2 THEN 'MEDIUM'
            ELSE 'LOW'
        END AS complexity_level
    FROM source_claims
)

SELECT * FROM claims_with_calculations
```

**Why `merge` strategy?**
- Claims can be corrected after initial submission (updates needed)
- Late arrivals common (claims submitted weeks after service date)
- Need to maintain data integrity (no duplicates)
- `unique_key='claim_id'` ensures each claim appears only once with latest values

**Performance optimization:**
- Partitioned by `claim_date` (monthly) reduces scan size
- Clustered by `provider_id` and `claim_status` for fast filtering
- Looks back 90 days to catch late updates (business requirement)

**Result:** Handles 50M+ claims/month with updates completing in 15 minutes instead of 2+ hours with full refresh.

---

### Interview Talking Point:

"Choosing the right incremental strategy is critical for performance and correctness. At Optum, we use `merge` for claims because they're frequently updated with corrections or status changes—a claim submitted as 'pending' might update to 'approved' or 'denied'. Using `append` would create duplicates. However, for immutable event logs like patient portal logins, we use `append` strategy for 10x faster processing. The key decision factors are: (1) Does the data update? Use merge. (2) Is it append-only? Use append. (3) Are you on BigQuery with partitions? Use insert_overwrite for atomic updates. We saw 80% performance improvement by switching our event logs from merge to append once we confirmed they were truly immutable."

---

## Q9: What are dbt tests and how do you implement them? What's the difference between generic and singular tests?

### Answer:

dbt tests validate data quality at build time. Tests run SQL queries that return failing rows—if any rows are returned, the test fails. Tests are essential for ensuring data reliability and catching issues before they reach downstream users.

### Types of dbt Tests:

#### 1. **Generic Tests** (Reusable Tests)

Built-in or custom tests that can be applied to any column.

**Built-in Generic Tests:**
- `unique` - Column values are unique
- `not_null` - Column has no nulls
- `accepted_values` - Column contains only specified values
- `relationships` - Foreign key validation

```yaml
# models/schema.yml
version: 2

models:
  - name: fct_claims
    description: "Insurance claims fact table"
    columns:
      - name: claim_id
        description: "Unique claim identifier"
        tests:
          - unique
          - not_null

      - name: claim_status
        description: "Current status of the claim"
        tests:
          - not_null
          - accepted_values:
              values: ['PENDING', 'APPROVED', 'DENIED', 'RESUBMITTED']

      - name: patient_id
        description: "Foreign key to patient dimension"
        tests:
          - not_null
          - relationships:
              to: ref('dim_patients')
              field: patient_id

      - name: claim_amount
        description: "Claimed amount in dollars"
        tests:
          - not_null
          - dbt_utils.expression_is_true:
              expression: ">= 0"
              config:
                severity: error
```

---

#### 2. **Singular Tests** (One-Off SQL Tests)

Custom SQL tests in the `tests/` folder that return failing rows.

```sql
-- tests/claims_amount_matches_line_items.sql
-- Test: Total claim amount should equal sum of line items

WITH claim_totals AS (
    SELECT
        claim_id,
        claim_amount
    FROM {{ ref('fct_claims') }}
),

line_item_totals AS (
    SELECT
        claim_id,
        SUM(line_item_amount) AS total_line_items
    FROM {{ ref('fct_claim_line_items') }}
    GROUP BY claim_id
)

SELECT
    c.claim_id,
    c.claim_amount,
    l.total_line_items,
    ABS(c.claim_amount - l.total_line_items) AS difference
FROM claim_totals c
INNER JOIN line_item_totals l
    ON c.claim_id = l.claim_id
WHERE ABS(c.claim_amount - l.total_line_items) > 0.01  -- Allow 1 cent rounding
```

This test fails if any claims have mismatched totals (returns failing rows).

---

### Advanced Testing Examples:

#### Custom Generic Test (Reusable):

```sql
-- tests/generic/test_valid_date_range.sql
{% test valid_date_range(model, column_name, min_date, max_date) %}

SELECT *
FROM {{ model }}
WHERE {{ column_name }} < '{{ min_date }}'
   OR {{ column_name }} > '{{ max_date }}'

{% endtest %}
```

**Usage:**
```yaml
models:
  - name: fct_claims
    columns:
      - name: claim_date
        tests:
          - valid_date_range:
              min_date: '2020-01-01'
              max_date: '2030-12-31'
```

---

#### Complex Singular Test (Business Logic):

```sql
-- tests/patients_have_recent_activity.sql
-- Business rule: Active patients should have activity in last 12 months

WITH active_patients AS (
    SELECT patient_id
    FROM {{ ref('dim_patients') }}
    WHERE patient_status = 'ACTIVE'
),

recent_activity AS (
    SELECT DISTINCT patient_id
    FROM {{ ref('fct_appointments') }}
    WHERE appointment_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 12 MONTH)
)

-- Return patients marked active but with no recent activity (failing rows)
SELECT
    ap.patient_id,
    'No activity in last 12 months' AS failure_reason
FROM active_patients ap
LEFT JOIN recent_activity ra
    ON ap.patient_id = ra.patient_id
WHERE ra.patient_id IS NULL
```

---

### Test Severity and Configuration:

```yaml
models:
  - name: fct_claims
    tests:
      - dbt_utils.recency:
          datepart: day
          field: claim_date
          interval: 2
          config:
            severity: warn  # Don't fail build, just warn
            error_if: ">100"  # Fail if more than 100 failing rows
            warn_if: ">10"    # Warn if more than 10 failing rows

    columns:
      - name: claim_id
        tests:
          - unique:
              config:
                severity: error  # Fail build
                where: "claim_status != 'DELETED'"  # Exclude deleted claims
```

---

### Running Tests:

```bash
# Run all tests
dbt test

# Run tests for specific model
dbt test --select fct_claims

# Run only generic tests
dbt test --select test_type:generic

# Run only singular tests
dbt test --select test_type:singular

# Run tests downstream of a model
dbt test --select fct_claims+

# Run tests with increased verbosity
dbt test --select fct_claims --store-failures
```

---

### Test Results Storage:

```yaml
# dbt_project.yml
tests:
  +store_failures: true  # Store failing rows in database
  +schema: test_failures  # Schema for failure tables
```

When enabled, failing rows are stored in tables like:
`test_failures.unique_fct_claims_claim_id`

**Query failing rows:**
```sql
SELECT *
FROM test_failures.unique_fct_claims_claim_id
WHERE test_execution_time = (SELECT MAX(test_execution_time) FROM test_failures.unique_fct_claims_claim_id)
```

---

### Real-World Example (Optum Healthcare):

**Testing Strategy for Claims Pipeline:**

```yaml
# models/claims/schema.yml
version: 2

models:
  - name: fct_claims
    description: "Claims fact table with comprehensive data quality tests"

    # Model-level tests
    tests:
      - dbt_utils.recency:
          datepart: hour
          field: claim_received_timestamp
          interval: 6
          config:
            severity: error
            error_if: ">0"

    columns:
      - name: claim_id
        tests:
          - unique
          - not_null

      - name: patient_id
        tests:
          - not_null
          - relationships:
              to: ref('dim_patients')
              field: patient_id
              config:
                where: "claim_status != 'DELETED'"

      - name: provider_id
        tests:
          - not_null
          - relationships:
              to: ref('dim_providers')
              field: provider_id

      - name: claim_amount
        tests:
          - not_null
          - dbt_utils.expression_is_true:
              expression: ">= 0 AND <= 1000000"
              config:
                severity: error

      - name: claim_status
        tests:
          - not_null
          - accepted_values:
              values: ['PENDING', 'APPROVED', 'DENIED', 'RESUBMITTED', 'DELETED']

      - name: claim_date
        tests:
          - not_null
          - valid_date_range:
              min_date: '2015-01-01'
              max_date: '2030-12-31'
```

**Singular Tests:**

```sql
-- tests/claims_deny_reasons_required.sql
-- Business rule: Denied claims must have a denial reason

SELECT
    claim_id,
    claim_status,
    denial_reason
FROM {{ ref('fct_claims') }}
WHERE claim_status = 'DENIED'
  AND (denial_reason IS NULL OR denial_reason = '')
```

```sql
-- tests/claims_provider_specialties_match.sql
-- Business rule: Procedure codes must match provider specialties

WITH claims_with_providers AS (
    SELECT
        c.claim_id,
        c.procedure_codes,
        p.specialty
    FROM {{ ref('fct_claims') }} c
    INNER JOIN {{ ref('dim_providers') }} p
        ON c.provider_id = p.provider_id
    WHERE c.claim_status IN ('APPROVED', 'PENDING')
)

SELECT
    claim_id,
    procedure_codes,
    specialty,
    'Cardiology procedure by non-cardiologist' AS issue
FROM claims_with_providers
WHERE specialty != 'CARDIOLOGY'
  AND EXISTS (
      SELECT 1
      FROM UNNEST(procedure_codes) AS code
      WHERE code LIKE '92%' OR code LIKE '93%'  -- Cardiology procedure codes
  )
```

**Result:** Caught 1,200 data quality issues in first week:
- 450 claims with missing denial reasons
- 180 invalid foreign keys (patients not in dimension)
- 95 claims exceeding reasonable amounts
- Prevented $2.3M in incorrect claim processing

---

### Generic vs Singular Tests Comparison:

| Feature | Generic Tests | Singular Tests |
|---------|---------------|----------------|
| **Location** | `models/schema.yml` | `tests/*.sql` |
| **Reusability** | ✅ Reusable across models | ❌ One-off test |
| **Syntax** | YAML configuration | SQL query |
| **Scope** | Column or model level | Any SQL logic |
| **Complexity** | Simple validation rules | Complex business logic |
| **Best for** | Common validations (unique, not null) | Custom business rules |

---

### Testing Best Practices:

1. **Test critical business logic** - Focus on data that drives decisions
2. **Test at the source** - Validate raw data early in the pipeline
3. **Use `store_failures`** - Debug failing tests easily
4. **Set appropriate severity** - Not all tests should fail the build
5. **Test relationships** - Validate foreign keys between models
6. **Test business rules** - Use singular tests for domain logic
7. **Run tests in CI/CD** - Automated testing before production deployment

---

### Interview Talking Point:

"dbt testing is critical for data quality. At Optum, we implement a three-tier testing strategy: (1) Generic tests on all fact/dimension tables for basic quality (unique keys, not nulls, referential integrity), (2) Singular tests for complex business rules like 'denied claims must have denial reasons', and (3) Model-level tests for freshness and row counts. When we added comprehensive tests to our claims pipeline, we caught 1,200+ data issues in the first week that would have caused billing errors. The key is balancing coverage with speed—we use `warn` severity for non-critical tests so builds don't fail unnecessarily. Tests run in CI/CD, and failing rows are stored in a `test_failures` schema for debugging. This reduced production data incidents by 70% and gave stakeholders confidence in our data quality."

---

## Q10: How do you structure a dbt project? What are best practices for organizing models, tests, and documentation?

### Answer:

A well-structured dbt project is critical for maintainability, collaboration, and scalability. dbt recommends a layered approach (staging, intermediate, marts) with clear naming conventions and organization.

### Standard dbt Project Structure:

```
dbt_project/
├── dbt_project.yml           # Project configuration
├── packages.yml              # dbt package dependencies
├── profiles.yml              # Connection profiles (not in git!)
│
├── models/                   # All dbt models
│   ├── staging/              # Layer 1: Raw source transformations
│   │   ├── _sources.yml      # Source definitions
│   │   ├── crm/
│   │   │   ├── _crm_models.yml
│   │   │   ├── stg_crm__customers.sql
│   │   │   └── stg_crm__orders.sql
│   │   ├── payments/
│   │   │   ├── _payments_models.yml
│   │   │   ├── stg_payments__transactions.sql
│   │   │   └── stg_payments__refunds.sql
│   │   └── healthcare/
│   │       ├── _healthcare_sources.yml
│   │       ├── stg_healthcare__patients.sql
│   │       ├── stg_healthcare__claims.sql
│   │       └── stg_healthcare__providers.sql
│   │
│   ├── intermediate/         # Layer 2: Business logic transformations
│   │   ├── claims/
│   │   │   ├── _int_claims_models.yml
│   │   │   ├── int_claims__with_risk_scores.sql
│   │   │   └── int_claims__with_denials.sql
│   │   └── patients/
│   │       ├── _int_patients_models.yml
│   │       └── int_patients__with_demographics.sql
│   │
│   └── marts/                # Layer 3: Business-defined entities
│       ├── core/             # Core business entities
│       │   ├── _core_models.yml
│       │   ├── dim_patients.sql
│       │   ├── dim_providers.sql
│       │   ├── dim_diagnoses.sql
│       │   ├── fct_claims.sql
│       │   ├── fct_appointments.sql
│       │   └── fct_prescriptions.sql
│       │
│       ├── finance/          # Finance-specific marts
│       │   ├── _finance_models.yml
│       │   ├── fct_revenue.sql
│       │   └── fct_payments.sql
│       │
│       └── analytics/        # Analytics/reporting marts
│           ├── _analytics_models.yml
│           ├── patient_360.sql
│           └── provider_performance.sql
│
├── tests/                    # Singular tests (custom SQL tests)
│   ├── generic/              # Custom generic tests
│   │   └── test_valid_date_range.sql
│   │
│   ├── claims/
│   │   ├── claims_amount_matches_line_items.sql
│   │   └── denied_claims_have_reasons.sql
│   │
│   └── patients/
│       └── active_patients_have_recent_activity.sql
│
├── macros/                   # Reusable SQL macros
│   ├── _macros.yml
│   ├── generate_schema_name.sql
│   ├── cents_to_dollars.sql
│   └── calculate_age.sql
│
├── analyses/                 # Ad-hoc analytical queries (not models)
│   └── claim_approval_rate_analysis.sql
│
├── seeds/                    # Static CSV data
│   ├── diagnosis_code_lookup.csv
│   ├── specialty_mappings.csv
│   └── state_abbreviations.csv
│
├── snapshots/                # SCD Type 2 snapshots
│   ├── patient_insurance_snapshot.sql
│   └── provider_status_snapshot.sql
│
└── docs/                     # Custom documentation
    ├── overview.md
    └── data_dictionary.md
```

---

### Naming Conventions:

#### Model Prefixes:

| Layer | Prefix | Example | Description |
|-------|--------|---------|-------------|
| **Staging** | `stg_<source>__` | `stg_healthcare__claims.sql` | Raw source with light transformation |
| **Intermediate** | `int_<entity>__` | `int_claims__with_risk.sql` | Business logic, not exposed to end users |
| **Dimension** | `dim_` | `dim_patients.sql` | Slowly changing dimensions |
| **Fact** | `fct_` | `fct_claims.sql` | Fact tables (events, transactions) |
| **Report** | `rpt_` | `rpt_monthly_claims.sql` | Reporting/aggregated tables |

#### File Naming:
- Use **snake_case**: `stg_healthcare__patients.sql`
- Source prefix: `stg_<source>__<entity>`
- Double underscore `__` separates source from entity
- Intermediate: `int_<entity>__<description>`

---

### Layer Structure (Staging → Intermediate → Marts):

#### Layer 1: Staging (stg_)

**Purpose:** Light transformation of raw sources. One staging model per source table.

```sql
-- models/staging/healthcare/stg_healthcare__claims.sql
-- Light cleaning and renaming only

WITH source AS (
    SELECT *
    FROM {{ source('raw_healthcare', 'claims') }}
),

renamed AS (
    SELECT
        -- Primary keys
        claim_id,

        -- Foreign keys
        patient_id,
        provider_id,

        -- Attributes
        clm_date AS claim_date,
        clm_amt AS claim_amount,
        clm_status AS claim_status,

        -- Metadata
        _loaded_at AS source_loaded_at
    FROM source
)

SELECT * FROM renamed
```

**Staging layer rules:**
- ✅ Rename columns to standard names
- ✅ Cast data types
- ✅ Basic data cleaning (trim, lowercase)
- ❌ No business logic
- ❌ No joins to other tables
- ❌ No aggregations

---

#### Layer 2: Intermediate (int_)

**Purpose:** Business logic transformations. Not exposed to end users.

```sql
-- models/intermediate/claims/int_claims__with_risk_scores.sql

WITH claims AS (
    SELECT * FROM {{ ref('stg_healthcare__claims') }}
),

patients AS (
    SELECT * FROM {{ ref('stg_healthcare__patients') }}
),

claims_with_patient_info AS (
    SELECT
        c.*,
        p.date_of_birth,
        p.chronic_conditions
    FROM claims c
    LEFT JOIN patients p
        ON c.patient_id = p.patient_id
),

claims_with_risk AS (
    SELECT
        *,

        -- Calculate risk score
        CASE
            WHEN ARRAY_LENGTH(chronic_conditions) >= 3 THEN 'HIGH'
            WHEN ARRAY_LENGTH(chronic_conditions) >= 1 THEN 'MEDIUM'
            ELSE 'LOW'
        END AS patient_risk_level,

        -- Calculate age at time of claim
        DATE_DIFF(claim_date, date_of_birth, YEAR) AS patient_age_at_claim
    FROM claims_with_patient_info
)

SELECT * FROM claims_with_risk
```

**Intermediate layer rules:**
- ✅ Business logic and calculations
- ✅ Join multiple staging models
- ✅ Complex transformations
- ❌ Not exposed to end users (ephemeral or views)
- ❌ No aggregations (leave for marts)

---

#### Layer 3: Marts (dim_, fct_, rpt_)

**Purpose:** Business-defined entities for end users (analysts, BI tools).

```sql
-- models/marts/core/fct_claims.sql
-- Final fact table exposed to end users

{{
  config(
    materialized='incremental',
    unique_key='claim_id',
    partition_by={"field": "claim_date", "data_type": "date"},
    cluster_by=['provider_id', 'claim_status']
  )
}}

WITH int_claims AS (
    SELECT * FROM {{ ref('int_claims__with_risk_scores') }}
),

dim_providers AS (
    SELECT * FROM {{ ref('dim_providers') }}
),

final AS (
    SELECT
        -- Keys
        c.claim_id,
        c.patient_id,
        c.provider_id,

        -- Dates
        c.claim_date,

        -- Measures
        c.claim_amount,
        c.allowed_amount,
        c.paid_amount,

        -- Attributes
        c.claim_status,
        c.patient_risk_level,
        c.patient_age_at_claim,
        p.provider_specialty,
        p.provider_network_tier,

        -- Calculations
        c.claim_amount - c.paid_amount AS outstanding_amount,

        -- Metadata
        CURRENT_TIMESTAMP() AS dbt_updated_at
    FROM int_claims c
    LEFT JOIN dim_providers p
        ON c.provider_id = p.provider_id

    {% if is_incremental() %}
    WHERE c.claim_date > (SELECT MAX(claim_date) FROM {{ this }})
    {% endif %}
)

SELECT * FROM final
```

**Marts layer rules:**
- ✅ Exposed to end users
- ✅ Aggregations and metrics
- ✅ Optimized for querying (partitioned, clustered)
- ✅ Comprehensive tests and documentation
- ✅ Use incremental or table materialization

---

### Configuration in dbt_project.yml:

```yaml
# dbt_project.yml
name: 'optum_healthcare_analytics'
version: '1.0.0'
config-version: 2

profile: 'optum_healthcare'

model-paths: ["models"]
analysis-paths: ["analyses"]
test-paths: ["tests"]
seed-paths: ["seeds"]
macro-paths: ["macros"]
snapshot-paths: ["snapshots"]

clean-targets:
  - "target"
  - "dbt_packages"

models:
  optum_healthcare_analytics:

    # Staging layer
    staging:
      +materialized: view
      +schema: staging
      +tags: ["staging"]

    # Intermediate layer
    intermediate:
      +materialized: ephemeral  # Not materialized (CTE only)
      +schema: intermediate
      +tags: ["intermediate"]

    # Marts layer
    marts:
      +materialized: table
      +schema: marts
      +tags: ["marts"]

      core:
        +schema: marts_core
        +tags: ["marts", "core"]

        # Dimensions as views (small, frequently changing)
        dim_*:
          +materialized: view

        # Facts as incremental (large, append-heavy)
        fct_*:
          +materialized: incremental
          +on_schema_change: "append_new_columns"

      finance:
        +schema: marts_finance
        +tags: ["marts", "finance"]

      analytics:
        +schema: marts_analytics
        +tags: ["marts", "analytics"]

# Seeds configuration
seeds:
  optum_healthcare_analytics:
    +schema: seeds
    +tags: ["seeds"]

# Snapshots configuration
snapshots:
  optum_healthcare_analytics:
    +target_schema: snapshots
    +tags: ["snapshots"]

# Tests configuration
tests:
  optum_healthcare_analytics:
    +store_failures: true
    +schema: test_failures
```

---

### Real-World Example (Optum Healthcare dbt Project):

**Project stats:**
- 180+ models (45 staging, 35 intermediate, 100 marts)
- 15 data sources
- 500+ tests
- 8 team members contributing

**Organization:**

```
models/
├── staging/
│   ├── claims_system/         # Source: Claims processing system
│   │   ├── stg_claims__claims.sql
│   │   ├── stg_claims__line_items.sql
│   │   └── stg_claims__denials.sql
│   │
│   ├── emr/                   # Source: Electronic medical records
│   │   ├── stg_emr__patients.sql
│   │   ├── stg_emr__encounters.sql
│   │   └── stg_emr__diagnoses.sql
│   │
│   └── providers/             # Source: Provider network system
│       ├── stg_providers__providers.sql
│       └── stg_providers__networks.sql
│
├── intermediate/
│   ├── claims/
│   │   ├── int_claims__enriched.sql
│   │   ├── int_claims__with_risk.sql
│   │   └── int_claims__with_denials.sql
│   │
│   └── patients/
│       ├── int_patients__with_demographics.sql
│       └── int_patients__with_conditions.sql
│
└── marts/
    ├── core/
    │   ├── dim_patients.sql
    │   ├── dim_providers.sql
    │   ├── fct_claims.sql
    │   └── fct_encounters.sql
    │
    ├── finance/
    │   ├── fct_revenue_daily.sql
    │   └── rpt_claims_aging.sql
    │
    └── quality/
        ├── rpt_quality_measures.sql
        └── rpt_readmission_rates.sql
```

---

### Best Practices Summary:

#### Folder Structure:
1. ✅ Use **three-layer** approach (staging → intermediate → marts)
2. ✅ Organize by **source system** in staging
3. ✅ Organize by **business domain** in marts
4. ✅ One model per file, file name matches model name

#### Naming:
1. ✅ Use consistent **prefixes** (stg_, int_, dim_, fct_)
2. ✅ Use **snake_case** for all names
3. ✅ Include **source** in staging names: `stg_<source>__<entity>`
4. ✅ Descriptive names: `int_claims__with_risk_scores` not `int_claims_2`

#### Configuration:
1. ✅ Set defaults in **dbt_project.yml** by folder
2. ✅ Staging as **views** (fast, always fresh)
3. ✅ Intermediate as **ephemeral** (not materialized, CTEs only)
4. ✅ Marts as **tables** or **incremental**
5. ✅ Use **tags** for selective execution

#### Documentation & Tests:
1. ✅ Document **all** models in schema.yml files
2. ✅ Test **all** primary keys (unique, not_null)
3. ✅ Test **all** foreign keys (relationships)
4. ✅ Store **test failures** for debugging

---

### Interview Talking Point:

"A well-structured dbt project is essential for team collaboration and long-term maintainability. At Optum, we follow the standard three-layer approach: (1) Staging models do light transformation of raw sources—just renaming and type casting, one model per source table; (2) Intermediate models contain business logic and joins but aren't exposed to end users, usually ephemeral; (3) Marts are the final business entities exposed to analysts and BI tools, optimized with incremental loads and partitioning. We organize by source system in staging (claims_system/, emr/, providers/) and by business domain in marts (core/, finance/, quality/). Naming conventions are strict—stg_<source>__<entity>, int_<entity>__<description>, dim_/fct_ prefixes. This structure lets our 8-person team work without conflicts, makes code reviews easier, and allows new team members to understand the project in days not weeks. When we migrated from legacy ETL to dbt, this structure reduced development time by 60% and made our data lineage crystal clear."

---


## Q11-Q100: Complete dbt Coverage

**Q11-Q20 (Core Concepts):** Incremental models (append/merge), dbt tests (unique/not_null/relationships/accepted_values), Custom tests, Sources vs seeds, Snapshots (SCD Type 2), Macros (Jinja), Packages (dbt_utils), Hooks (pre/post), Operations, Documentation.

**Q21-Q30 (Materializations):** Table (full refresh), View (virtual), Incremental (delta), Ephemeral (CTE), Custom materializations, Materialization configs, Performance trade-offs, Storage costs, Query patterns, Best practices.

**Q32-Q40 (Data Quality):** dbt test types, Schema tests, Data tests, Custom singular tests, Great Expectations integration, dbt_expectations package, Test severity (warn/error), Test selection, CI/CD testing, Production alerts.

**Q41-Q50 (Jinja & Macros):** Jinja basics ({% %}), Variables ({{ var() }}), Loops/conditions, Custom macros, Cross-database macros, dbt_utils macros (date_spine, surrogate_key), Macro arguments, dbt_project.yml macros, Package macros, Macro testing.

**Q51-Q60 (Orchestration):** Airflow + dbt, Dagster integration, dbt Cloud scheduler, dbt run selectors (--select, --exclude, tags), State-based selection (--state), Slim CI, Parallel execution, Retry logic, Alerting on failure, Cost optimization.

**Q61-Q70 (Advanced Patterns):** dbt Mesh (cross-project refs), Exposures, Metrics layer, Semantic layer, dbt Server, Multi-project setup, Monorepo vs multi-repo, Version control, Environment management (dev/prod), Feature flags.

**Q71-Q80 (Performance):** Query optimization, Incremental strategy selection, Partition pruning, Materialized views, dbt compile analysis, Profile performance, Reduce test time, Parallel threads, Cloud warehouse optimization (Snowflake/BigQuery/Redshift/Databricks), Cost monitoring.

**Q81-Q90 (Production & Governance):** CI/CD pipelines, PR checks, dbt docs deployment, Data catalog integration, Column-level lineage, Access controls, PII tagging, Data classification, Audit logging, Compliance (GDPR/HIPAA).

**Q91-Q100 (Best Practices & Real-world):** Naming conventions, Folder structure, Model layers (staging/intermediate/marts), Config inheritance, DRY principles, Error handling, Debugging (`dbt --debug`), Performance profiling, Team collaboration, **Q100: Optum implementation -** 500+ dbt models (Bronze/Silver/Gold), Databricks backend, Airflow orchestration, 10M+ claims transformed daily, Great Expectations tests, 99.5% test pass rate, Git-based CI/CD, documented in dbt docs, reduced transformation time 6h → 1h.

