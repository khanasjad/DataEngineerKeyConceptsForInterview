# Analytics Engineer Interview Prep - 7-Week Master Plan 🎯

**Goal:** Land Analytics Engineer role with confidence in SQL, dbt, data modeling, and business analytics

**Your Arsenal:**
- ✅ 900 questions across 10 data engineering topics
- ✅ 10 cheatsheets for quick reference
- ✅ Layman's guide for concept clarity
- ✅ Real-world Optum use cases

---

## 📅 WEEK-BY-WEEK BREAKDOWN

---

## **WEEK 1: SQL FUNDAMENTALS → INTERMEDIATE**

### **Daily Schedule (2-3 hours/day)**

#### **Monday: JOINs & Aggregations**
**Morning (60 min):**
- Read: `Python-SQL/CHEATSHEET.md` - JOIN types section
- Study: `Python-SQL/100-QUESTIONS.md` Q15-Q25 (JOINs)

**Practice (60 min):**
```sql
-- Exercise 1: Multi-table JOIN
-- Find customers who purchased in Q1 but not Q2

-- Exercise 2: Self-JOIN
-- Find employees earning more than their managers

-- Exercise 3: Aggregation
-- Calculate total revenue by product category, show only top 10
```

**Resources:**
- LeetCode SQL: Problems 175, 176, 177 (JOIN basics)
- HackerRank: "The Report", "Top Competitors"

---

#### **Tuesday: Window Functions Part 1 (RANK, ROW_NUMBER, DENSE_RANK)**
**Morning (60 min):**
- Study: `Python-SQL/100-QUESTIONS.md` Q30-Q40 (Window functions)
- Review: Different ranking functions and use cases

**Practice (90 min):**
```sql
-- Exercise 1: Find 2nd highest salary per department
SELECT dept, employee, salary
FROM (
    SELECT dept, employee, salary,
           RANK() OVER (PARTITION BY dept ORDER BY salary DESC) as rk
    FROM employees
) WHERE rk = 2

-- Exercise 2: Consecutive appearances
-- Find products sold in 3+ consecutive months

-- Exercise 3: Running totals
-- Calculate cumulative revenue by month
```

**Resources:**
- LeetCode: 178 (Rank Scores), 184 (Department Highest Salary)
- Mode Analytics: Window Functions Tutorial

---

#### **Wednesday: Window Functions Part 2 (LEAD, LAG, NTILE)**
**Morning (60 min):**
- Study: `Python-SQL/100-QUESTIONS.md` Q41-Q50

**Practice (90 min):**
```sql
-- Exercise 1: Month-over-month growth
SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) as prev_month,
    (revenue - LAG(revenue) OVER (ORDER BY month)) /
    LAG(revenue) OVER (ORDER BY month) * 100 as growth_pct
FROM monthly_revenue

-- Exercise 2: Customer churn detection
-- Flag customers who didn't purchase in 90 days after last order

-- Exercise 3: Quartile analysis
-- Divide customers into quartiles by lifetime value
```

---

#### **Thursday: CTEs & Subqueries**
**Morning (60 min):**
- Study: `Python-SQL/100-QUESTIONS.md` Q51-Q60

**Practice (90 min):**
```sql
-- Exercise 1: Recursive CTE - org hierarchy
WITH RECURSIVE org_tree AS (
    SELECT employee_id, manager_id, name, 1 as level
    FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.manager_id, e.name, ot.level + 1
    FROM employees e
    JOIN org_tree ot ON e.manager_id = ot.employee_id
)
SELECT * FROM org_tree

-- Exercise 2: Complex CTE - cohort analysis
-- Calculate retention rate by signup month

-- Exercise 3: Multiple CTEs
-- Find customers with above-average orders in their segment
```

---

#### **Friday: Date/Time Manipulation**
**Morning (60 min):**
- Study: `Python-SQL/100-QUESTIONS.md` Q61-Q70

**Practice (90 min):**
```sql
-- Exercise 1: Sessionization
-- Group events into 30-minute sessions

-- Exercise 2: Business days calculation
-- Calculate days between order and delivery (excluding weekends)

-- Exercise 3: Time bucketing
-- Aggregate events into 15-minute intervals
```

---

#### **Weekend: LeetCode SQL Marathon**
**Saturday (3 hours):**
- Solve 10 Medium SQL problems on LeetCode
- Focus on: window functions, CTEs, date manipulation
- Track time: aim for 20-30 min per problem

**Sunday (2 hours):**
- Review: All `Python-SQL/CHEATSHEET.md`
- Redo 3 hardest problems from the week
- Create personal SQL cheatsheet with patterns

**📊 Week 1 Checkpoint:**
- [ ] Solved 20+ SQL problems
- [ ] Comfortable with window functions
- [ ] Can write complex CTEs
- [ ] Fast with JOIN logic

---

## **WEEK 2: ADVANCED SQL & ANALYTICS QUERIES**

### **Daily Schedule (2-3 hours/day)**

#### **Monday: Cohort Analysis**
**Study (60 min):**
- Concept: Grouping users by common characteristic (signup date)
- Metrics: Retention rate, revenue per cohort, LTV

**Practice (90 min):**
```sql
-- Exercise: E-commerce retention analysis
WITH cohorts AS (
    SELECT
        user_id,
        DATE_TRUNC('month', first_order_date) as cohort_month
    FROM users
),
cohort_activity AS (
    SELECT
        c.cohort_month,
        DATE_TRUNC('month', o.order_date) as activity_month,
        COUNT(DISTINCT o.user_id) as active_users
    FROM cohorts c
    JOIN orders o ON c.user_id = o.user_id
    GROUP BY 1, 2
)
SELECT
    cohort_month,
    DATEDIFF('month', cohort_month, activity_month) as months_since_signup,
    active_users,
    active_users * 100.0 / FIRST_VALUE(active_users) OVER (
        PARTITION BY cohort_month ORDER BY activity_month
    ) as retention_rate
FROM cohort_activity
```

---

#### **Tuesday: Funnel Analysis**
**Study (60 min):**
- Concept: Track user journey through conversion steps
- Metrics: Conversion rate, drop-off points

**Practice (90 min):**
```sql
-- Exercise: Signup funnel
WITH funnel AS (
    SELECT
        COUNT(DISTINCT CASE WHEN event = 'page_view' THEN user_id END) as viewed,
        COUNT(DISTINCT CASE WHEN event = 'signup_start' THEN user_id END) as started,
        COUNT(DISTINCT CASE WHEN event = 'signup_complete' THEN user_id END) as completed
    FROM events
    WHERE event_date = '2026-05-01'
)
SELECT
    viewed,
    started,
    completed,
    started * 100.0 / viewed as view_to_start_rate,
    completed * 100.0 / started as start_to_complete_rate,
    completed * 100.0 / viewed as overall_conversion
FROM funnel
```

---

#### **Wednesday: A/B Test Analysis**
**Study (60 min):**
- Concept: Compare metrics between control and treatment groups
- Statistical significance basics

**Practice (90 min):**
```sql
-- Exercise: Email campaign A/B test
SELECT
    variant,
    COUNT(DISTINCT user_id) as users,
    COUNT(DISTINCT CASE WHEN clicked = 1 THEN user_id END) as clickers,
    COUNT(DISTINCT CASE WHEN purchased = 1 THEN user_id END) as purchasers,
    AVG(revenue) as avg_revenue,
    COUNT(*) * 100.0 / SUM(COUNT(*)) OVER () as pct_of_total
FROM ab_test_results
GROUP BY variant
```

---

#### **Thursday: Product Analytics (DAU/MAU, Stickiness)**
**Practice (120 min):**
```sql
-- Exercise 1: DAU/MAU ratio
WITH daily_active AS (
    SELECT DATE_TRUNC('day', event_date) as day, COUNT(DISTINCT user_id) as dau
    FROM events
    GROUP BY 1
),
monthly_active AS (
    SELECT DATE_TRUNC('month', event_date) as month, COUNT(DISTINCT user_id) as mau
    FROM events
    GROUP BY 1
)
SELECT
    d.day,
    d.dau,
    m.mau,
    d.dau * 100.0 / m.mau as stickiness_ratio
FROM daily_active d
JOIN monthly_active m ON DATE_TRUNC('month', d.day) = m.month

-- Exercise 2: Feature adoption rate
-- Calculate % of users who used new feature in first 30 days
```

---

#### **Friday: Query Optimization**
**Study (90 min):**
- Read: `Python-SQL/100-QUESTIONS.md` Q80-Q90 (Performance)
- Topics: EXPLAIN plans, indexes, partitioning

**Practice (90 min):**
```sql
-- Exercise 1: Analyze slow query
EXPLAIN ANALYZE
SELECT c.customer_id, SUM(o.amount)
FROM customers c
JOIN orders o ON c.id = o.customer_id
WHERE o.order_date > '2025-01-01'
GROUP BY c.customer_id

-- Optimize: Add index on order_date, consider materialized view

-- Exercise 2: Partition pruning
-- Rewrite query to leverage date partitioning

-- Exercise 3: Avoiding full table scans
-- Use WHERE clause to filter early
```

---

#### **Weekend: SQL Case Study**
**Saturday (4 hours):**

**Case Study: E-commerce Analytics Dashboard**

You're given these tables:
```sql
-- users (user_id, signup_date, country, plan_type)
-- orders (order_id, user_id, order_date, amount, status)
-- products (product_id, category, price)
-- order_items (order_id, product_id, quantity)
```

**Deliverables:**
1. **Customer Metrics:**
   - Total customers by month
   - Active customers (ordered in last 30 days)
   - Churn rate by cohort

2. **Revenue Metrics:**
   - MRR (Monthly Recurring Revenue) by plan type
   - Revenue by product category
   - Top 10 customers by LTV

3. **Product Metrics:**
   - Products frequently bought together
   - Category mix by customer segment
   - Inventory turnover rate

**Write SQL for each metric + document assumptions**

**Sunday (2 hours):**
- Review all queries from the week
- Optimize 3 slowest queries
- Create personal "Analytics SQL Patterns" doc

**📊 Week 2 Checkpoint:**
- [ ] Can write cohort/funnel/A/B test queries
- [ ] Understand query optimization basics
- [ ] Built end-to-end analytics case study
- [ ] Fast with analytical queries (15-20 min)

---

## **WEEK 3: SQL MASTERY + dbt INTRO**

### **Monday-Tuesday: Advanced SQL Challenges**
**Daily Practice (2 hours):**
- LeetCode SQL Hard problems (5 problems)
- HackerRank Advanced SQL (5 problems)
- Focus: Multi-CTE queries, complex window functions

---

### **Wednesday: dbt Introduction**
**Study (3 hours):**
- Read: `dbt/CHEATSHEET.md` (entire file)
- Read: `dbt/100-QUESTIONS.md` Q1-Q20 (Fundamentals)
- Watch: dbt Fundamentals course (free)

**Key Concepts:**
- What is dbt? Why use it?
- Models, sources, tests, documentation
- Jinja templating basics
- ref() and source() macros

---

### **Thursday: dbt Setup & First Model**
**Hands-on (3 hours):**

1. **Install dbt:**
```bash
pip install dbt-core dbt-bigquery  # or dbt-snowflake, dbt-postgres
dbt init my_analytics_project
```

2. **Configure profiles.yml:**
```yaml
my_analytics_project:
  target: dev
  outputs:
    dev:
      type: bigquery  # or snowflake
      project: your-project
      dataset: dbt_dev
      threads: 4
```

3. **Create first staging model:**
```sql
-- models/staging/stg_orders.sql
{{ config(materialized='view') }}

SELECT
    order_id,
    customer_id,
    order_date,
    amount,
    status,
    -- Standardize column names
    LOWER(status) as order_status,
    -- Add metadata
    CURRENT_TIMESTAMP() as _loaded_at
FROM {{ source('raw', 'orders') }}
WHERE order_date >= '2024-01-01'
```

4. **Define source:**
```yaml
# models/staging/sources.yml
version: 2

sources:
  - name: raw
    database: production
    schema: raw_data
    tables:
      - name: orders
      - name: customers
      - name: products
```

5. **Run model:**
```bash
dbt run --select stg_orders
dbt test --select stg_orders
```

---

### **Friday: More dbt Models**
**Hands-on (3 hours):**

**Create staging layer (3 models):**
- stg_orders.sql
- stg_customers.sql
- stg_products.sql

**Create intermediate model:**
```sql
-- models/intermediate/int_orders_enriched.sql
{{ config(materialized='ephemeral') }}

SELECT
    o.order_id,
    o.customer_id,
    c.customer_name,
    c.signup_date,
    o.order_date,
    o.amount,
    p.product_name,
    p.category
FROM {{ ref('stg_orders') }} o
LEFT JOIN {{ ref('stg_customers') }} c ON o.customer_id = c.customer_id
LEFT JOIN {{ ref('stg_products') }} p ON o.product_id = p.product_id
```

**Create mart (final analytics table):**
```sql
-- models/marts/fct_orders.sql
{{ config(
    materialized='table',
    tags=['daily']
) }}

SELECT
    order_id,
    customer_id,
    order_date,
    amount,
    -- Business logic
    CASE
        WHEN amount > 1000 THEN 'high_value'
        WHEN amount > 500 THEN 'medium_value'
        ELSE 'low_value'
    END as order_tier
FROM {{ ref('int_orders_enriched') }}
```

---

### **Weekend: dbt Project Structure**
**Saturday (4 hours):**

**Build complete dbt project:**
```
my_analytics_project/
├── models/
│   ├── staging/          # 1:1 with source tables
│   │   ├── sources.yml
│   │   ├── stg_orders.sql
│   │   ├── stg_customers.sql
│   │   └── stg_products.sql
│   ├── intermediate/     # Business logic
│   │   └── int_orders_enriched.sql
│   └── marts/           # Final analytics tables
│       ├── fct_orders.sql
│       ├── dim_customers.sql
│       └── metrics/
│           └── customer_metrics.sql
├── tests/
│   └── assert_positive_amounts.sql
├── macros/
│   └── cents_to_dollars.sql
└── dbt_project.yml
```

**Sunday (2 hours):**
- Study: `dbt/100-QUESTIONS.md` Q21-Q40 (Materializations, Testing)
- Add tests to all models
- Generate docs: `dbt docs generate && dbt docs serve`

**📊 Week 3 Checkpoint:**
- [ ] SQL confidence at interview level
- [ ] dbt fundamentals solid
- [ ] Built working dbt project
- [ ] Understand model layering

---

## **WEEK 4: dbt DEEP DIVE**

### **Monday: Incremental Models**
**Study (90 min):**
- Read: `dbt/100-QUESTIONS.md` Q41-Q50 (Incremental models)

**Practice (90 min):**
```sql
-- models/marts/fct_events_incremental.sql
{{ config(
    materialized='incremental',
    unique_key='event_id',
    on_schema_change='append_new_columns'
) }}

SELECT
    event_id,
    user_id,
    event_type,
    event_timestamp,
    properties
FROM {{ source('raw', 'events') }}

{% if is_incremental() %}
    -- Only load new data
    WHERE event_timestamp > (SELECT MAX(event_timestamp) FROM {{ this }})
{% endif %}
```

**Exercise:**
- Create incremental model for large table (orders, events)
- Test backfill: `dbt run --select fct_events_incremental --full-refresh`
- Compare performance: incremental vs full refresh

---

### **Tuesday: Testing & Data Quality**
**Study (90 min):**
- Read: `dbt/100-QUESTIONS.md` Q51-Q60 (Testing)

**Practice (90 min):**

**Schema tests:**
```yaml
# models/marts/schema.yml
version: 2

models:
  - name: fct_orders
    description: "Order facts table"
    columns:
      - name: order_id
        description: "Primary key"
        tests:
          - unique
          - not_null

      - name: customer_id
        description: "Foreign key to customers"
        tests:
          - not_null
          - relationships:
              to: ref('dim_customers')
              field: customer_id

      - name: amount
        description: "Order amount in dollars"
        tests:
          - not_null
          - dbt_utils.expression_is_true:
              expression: ">= 0"

      - name: order_date
        tests:
          - not_null
          - dbt_utils.recency:
              datepart: day
              interval: 7
```

**Custom data test:**
```sql
-- tests/assert_order_amount_matches_items.sql
-- Returns records where order total doesn't match sum of line items
SELECT
    o.order_id,
    o.amount as order_total,
    SUM(oi.quantity * oi.price) as calculated_total
FROM {{ ref('fct_orders') }} o
JOIN {{ ref('fct_order_items') }} oi ON o.order_id = oi.order_id
GROUP BY o.order_id, o.amount
HAVING ABS(o.amount - SUM(oi.quantity * oi.price)) > 0.01
```

---

### **Wednesday: Macros & Jinja**
**Study (90 min):**
- Read: `dbt/100-QUESTIONS.md` Q61-Q70 (Macros, Jinja)

**Practice (90 min):**

**Macro 1: Reusable logic**
```sql
-- macros/cents_to_dollars.sql
{% macro cents_to_dollars(column_name, precision=2) %}
    ROUND({{ column_name }} / 100.0, {{ precision }})
{% endmacro %}

-- Usage in model:
SELECT
    order_id,
    {{ cents_to_dollars('amount_cents') }} as amount_dollars
FROM {{ source('raw', 'orders') }}
```

**Macro 2: Generate SQL dynamically**
```sql
-- macros/generate_date_columns.sql
{% macro generate_date_columns(date_column) %}
    DATE_TRUNC('day', {{ date_column }}) as date_day,
    DATE_TRUNC('week', {{ date_column }}) as date_week,
    DATE_TRUNC('month', {{ date_column }}) as date_month,
    DATE_TRUNC('quarter', {{ date_column }}) as date_quarter,
    DATE_TRUNC('year', {{ date_column }}) as date_year
{% endmacro %}
```

**Macro 3: Loop through list**
```sql
-- macros/pivot_metrics.sql
{% macro pivot_metrics(metrics_list) %}
    {% for metric in metrics_list %}
        SUM(CASE WHEN metric_name = '{{ metric }}' THEN value END) as {{ metric }}
        {%- if not loop.last %},{% endif %}
    {% endfor %}
{% endmacro %}

-- Usage:
SELECT
    user_id,
    {{ pivot_metrics(['revenue', 'orders', 'sessions']) }}
FROM metrics_table
GROUP BY user_id
```

---

### **Thursday: dbt Packages & Advanced Features**
**Study (90 min):**
- Read: `dbt/100-QUESTIONS.md` Q71-Q80

**Practice (90 min):**

**Install packages:**
```yaml
# packages.yml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.1.0
  - package: calogica/dbt_expectations
    version: 0.9.0
  - package: dbt-labs/codegen
    version: 0.10.0
```

```bash
dbt deps
```

**Use dbt_utils:**
```sql
-- Surrogate key
SELECT
    {{ dbt_utils.surrogate_key(['customer_id', 'order_date']) }} as order_key,
    customer_id,
    order_date
FROM {{ ref('stg_orders') }}

-- Date spine
{{ dbt_utils.date_spine(
    datepart="day",
    start_date="cast('2024-01-01' as date)",
    end_date="cast('2024-12-31' as date)"
) }}

-- Union tables
{{ dbt_utils.union_relations(
    relations=[
        ref('orders_2024'),
        ref('orders_2025')
    ]
) }}
```

---

### **Friday: dbt Documentation & Lineage**
**Hands-on (3 hours):**

1. **Document models:**
```yaml
# models/marts/schema.yml
models:
  - name: fct_orders
    description: |
      ## Order Facts Table

      This table contains one row per order with key metrics.

      **Refresh schedule:** Daily at 2am UTC
      **Owner:** Analytics Team

      ### Business Logic
      - Excludes cancelled orders
      - Includes only completed payments
      - Currency normalized to USD

    columns:
      - name: order_id
        description: "Unique order identifier from source system"
      - name: customer_id
        description: "Links to {{ ref('dim_customers') }}"
```

2. **Generate and explore docs:**
```bash
dbt docs generate
dbt docs serve  # Opens browser at localhost:8080
```

**Explore:**
- Lineage graph (DAG visualization)
- Model documentation
- Column-level details
- Test results

3. **Add meta fields:**
```yaml
models:
  - name: fct_orders
    meta:
      owner: analytics@company.com
      priority: high
      contains_pii: false
```

---

### **Weekend: Complete dbt Project**
**Saturday (5 hours):**

**Build production-ready dbt project:**

**Staging Layer (5 models):**
- stg_orders.sql
- stg_customers.sql
- stg_products.sql
- stg_payments.sql
- stg_shipments.sql

**Intermediate Layer (3 models):**
- int_orders_enriched.sql (join orders + customers + products)
- int_customer_order_history.sql (aggregate order stats per customer)
- int_product_performance.sql (product sales metrics)

**Marts Layer (5 models):**
- fct_orders.sql (order facts)
- fct_order_items.sql (line-item facts)
- dim_customers.sql (customer dimension with SCD Type 2)
- dim_products.sql (product dimension)
- customer_metrics.sql (aggregated KPIs)

**Add:**
- Tests for all models (20+ tests)
- Documentation for all models
- 3+ custom macros
- Incremental strategy for fact tables

**Sunday (3 hours):**
- Read: `dbt/100-QUESTIONS.md` Q81-Q100
- Prepare to explain your project
- Practice: "Walk me through your dbt project structure"
- Create presentation slides (5 slides)

**📊 Week 4 Checkpoint:**
- [ ] Proficient with dbt
- [ ] Built production-quality dbt project
- [ ] Understand testing & data quality
- [ ] Can explain architectural decisions

---

## **WEEK 5: DATA MODELING MASTERY**

### **Monday: Dimensional Modeling Fundamentals**
**Study (2 hours):**
- Read: `Data-Modeling/CHEATSHEET.md` (entire file)
- Read: `Data-Modeling/100-QUESTIONS.md` Q1-Q20

**Key Concepts:**
- Fact tables (measurements, metrics)
- Dimension tables (context, attributes)
- Star schema vs Snowflake schema
- Surrogate keys vs natural keys
- Grain (level of detail)

---

### **Tuesday: Star Schema Design Exercise**
**Practice (3 hours):**

**Scenario: E-commerce Data Warehouse**

**Design star schema with:**

**Fact Table: fct_orders**
```sql
CREATE TABLE fct_orders (
    order_key BIGINT PRIMARY KEY,         -- Surrogate key
    order_id VARCHAR(50),                  -- Natural key
    customer_key BIGINT,                   -- FK to dim_customers
    product_key BIGINT,                    -- FK to dim_products
    date_key INT,                          -- FK to dim_date
    order_date TIMESTAMP,
    -- Metrics
    quantity INT,
    unit_price DECIMAL(10,2),
    discount_amount DECIMAL(10,2),
    tax_amount DECIMAL(10,2),
    total_amount DECIMAL(10,2),
    -- Degenerate dimensions (attributes without own dimension)
    payment_method VARCHAR(50),
    shipping_method VARCHAR(50)
)
```

**Dimension: dim_customers (SCD Type 2)**
```sql
CREATE TABLE dim_customers (
    customer_key BIGINT PRIMARY KEY,       -- Surrogate key
    customer_id VARCHAR(50),               -- Natural key
    customer_name VARCHAR(200),
    email VARCHAR(200),
    customer_segment VARCHAR(50),
    country VARCHAR(100),
    -- SCD Type 2 columns
    effective_date DATE,
    expiration_date DATE,
    is_current BOOLEAN
)
```

**Dimension: dim_products**
```sql
CREATE TABLE dim_products (
    product_key BIGINT PRIMARY KEY,
    product_id VARCHAR(50),
    product_name VARCHAR(200),
    category VARCHAR(100),
    subcategory VARCHAR(100),
    brand VARCHAR(100),
    unit_cost DECIMAL(10,2),
    unit_price DECIMAL(10,2)
)
```

**Dimension: dim_date**
```sql
CREATE TABLE dim_date (
    date_key INT PRIMARY KEY,              -- YYYYMMDD format
    full_date DATE,
    day_of_week VARCHAR(20),
    day_of_month INT,
    day_of_year INT,
    week_of_year INT,
    month_name VARCHAR(20),
    month_number INT,
    quarter INT,
    year INT,
    is_weekend BOOLEAN,
    is_holiday BOOLEAN
)
```

**Exercise:**
- Draw ERD (Entity-Relationship Diagram)
- Document grain for each table
- Identify conformed dimensions
- Write INSERT statements to populate

---

### **Wednesday: SCD Type 2 Implementation**
**Study (90 min):**
- Read: `Data-Modeling/100-QUESTIONS.md` Q30-Q40 (SCDs)

**Practice (2 hours):**

**Scenario: Track customer address changes**

```sql
-- Initial load
INSERT INTO dim_customers (
    customer_key,
    customer_id,
    customer_name,
    address,
    effective_date,
    expiration_date,
    is_current
) VALUES (
    1,
    'C001',
    'John Doe',
    '123 Main St',
    '2024-01-01',
    '9999-12-31',
    TRUE
)

-- Customer moves (address change detected)
-- Step 1: Expire old record
UPDATE dim_customers
SET
    expiration_date = '2024-06-30',
    is_current = FALSE
WHERE customer_id = 'C001' AND is_current = TRUE

-- Step 2: Insert new record
INSERT INTO dim_customers (
    customer_key,
    customer_id,
    customer_name,
    address,
    effective_date,
    expiration_date,
    is_current
) VALUES (
    2,
    'C001',
    'John Doe',
    '456 Oak Ave',  -- New address
    '2024-07-01',
    '9999-12-31',
    TRUE
)
```

**dbt SCD Type 2 using snapshot:**
```sql
-- snapshots/customers_snapshot.sql
{% snapshot customers_snapshot %}

{{
    config(
      target_schema='snapshots',
      unique_key='customer_id',
      strategy='timestamp',
      updated_at='updated_at'
    )
}}

SELECT * FROM {{ source('raw', 'customers') }}

{% endsnapshot %}
```

**Exercise:**
- Implement SCD Type 2 for products (track price changes)
- Write queries to analyze historical data
- Create dbt snapshot for customer dimension

---

### **Thursday: Normalization vs Denormalization**
**Study (90 min):**
- Read: `Data-Modeling/100-QUESTIONS.md` Q41-Q55

**Practice (2 hours):**

**Exercise 1: Normalize (OLTP style)**
```sql
-- 3NF: Break into multiple tables to eliminate redundancy

-- Orders table
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2)
)

-- Order_items table
CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10,2)
)

-- Products table
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(200),
    category_id INT
)

-- Categories table
CREATE TABLE categories (
    category_id INT PRIMARY KEY,
    category_name VARCHAR(100)
)
```

**Exercise 2: Denormalize (OLAP style)**
```sql
-- Wide table for analytics (fewer joins, faster queries)
CREATE TABLE orders_denormalized (
    order_id INT,
    customer_id INT,
    customer_name VARCHAR(200),
    customer_segment VARCHAR(50),
    order_date DATE,
    product_id INT,
    product_name VARCHAR(200),
    category_name VARCHAR(100),
    subcategory_name VARCHAR(100),
    quantity INT,
    unit_price DECIMAL(10,2),
    total_amount DECIMAL(10,2)
)
```

**Trade-offs discussion:**
- When to normalize? (Reduce storage, maintain data integrity)
- When to denormalize? (Improve query performance, simplify queries)

---

### **Friday: Data Vault Modeling**
**Study (2 hours):**
- Read: `Data-Modeling/100-QUESTIONS.md` Q56-Q70

**Key Concepts:**
- Hubs (business keys)
- Links (relationships)
- Satellites (attributes, history)

**Example:**
```sql
-- Hub: Customer
CREATE TABLE hub_customer (
    customer_key BIGINT PRIMARY KEY,
    customer_id VARCHAR(50),  -- Business key
    load_date TIMESTAMP,
    record_source VARCHAR(100)
)

-- Hub: Product
CREATE TABLE hub_product (
    product_key BIGINT PRIMARY KEY,
    product_id VARCHAR(50),
    load_date TIMESTAMP,
    record_source VARCHAR(100)
)

-- Link: Order (relationship between customer and product)
CREATE TABLE link_order (
    order_key BIGINT PRIMARY KEY,
    customer_key BIGINT,
    product_key BIGINT,
    order_id VARCHAR(50),
    load_date TIMESTAMP,
    record_source VARCHAR(100)
)

-- Satellite: Customer attributes
CREATE TABLE sat_customer (
    customer_key BIGINT,
    load_date TIMESTAMP,
    customer_name VARCHAR(200),
    email VARCHAR(200),
    segment VARCHAR(50),
    hash_diff VARCHAR(100),  -- For change detection
    PRIMARY KEY (customer_key, load_date)
)
```

---

### **Weekend: Data Modeling Case Studies**
**Saturday (5 hours):**

**Case Study 1: Subscription Business**
Design dimensional model for SaaS company:
- Users sign up for plans (free, pro, enterprise)
- Usage tracked daily (API calls, storage)
- Track upgrades/downgrades (plan changes)
- Billing monthly

**Deliverables:**
- ERD with fact and dimension tables
- Grain statements for each table
- SCD strategy for plan changes
- Sample SQL to calculate MRR, churn

---

**Case Study 2: Healthcare Analytics**
Design dimensional model for hospital:
- Patients (demographics, insurance)
- Visits (admission, discharge, diagnoses)
- Procedures (treatments, costs)
- Providers (doctors, nurses)

**Deliverables:**
- Star schema design
- Handle many-to-many (patient-provider)
- Time dimensions (admission, discharge, procedure)
- Privacy considerations (HIPAA)

---

**Sunday (3 hours):**
- Review: All `Data-Modeling/CHEATSHEET.md`
- Read: `Data-Modeling/100-QUESTIONS.md` Q71-Q100
- Practice: Whiteboard 2 data models (20 min each)

**📊 Week 5 Checkpoint:**
- [ ] Can design star schemas confidently
- [ ] Understand SCD Type 2 implementation
- [ ] Know when to normalize vs denormalize
- [ ] Can explain trade-offs in modeling decisions

---

## **WEEK 6: END-TO-END CASE STUDIES**

### **Daily Focus: Complete Analytics Projects**

---

### **Case Study 1: SaaS Product Analytics (Monday-Tuesday)**

**Scenario:**
You're the Analytics Engineer at a B2B SaaS company. Build complete analytics infrastructure.

**Given Data Sources:**
- users (user_id, signup_date, company_id, plan)
- events (event_id, user_id, event_type, timestamp, properties)
- subscriptions (subscription_id, company_id, plan, mrr, start_date, end_date)
- companies (company_id, company_name, industry, size)

**Your Tasks:**

#### **Day 1: Data Modeling & dbt Setup (4 hours)**

1. **Design dimensional model:**
   - fct_events (event facts)
   - fct_subscriptions (subscription facts)
   - dim_users (user dimension)
   - dim_companies (company dimension)
   - dim_date (date dimension)

2. **Build dbt project:**
   - Staging models (4 models)
   - Intermediate models (2 models)
   - Mart models (5 models)
   - Tests (15+ tests)

3. **SQL transformations:**
```sql
-- models/marts/fct_product_usage.sql
WITH daily_events AS (
    SELECT
        user_id,
        DATE(event_timestamp) as event_date,
        COUNT(*) as event_count,
        COUNT(DISTINCT session_id) as session_count
    FROM {{ ref('stg_events') }}
    GROUP BY 1, 2
),
user_details AS (
    SELECT
        u.user_id,
        u.company_id,
        c.company_name,
        c.industry,
        u.plan,
        u.signup_date
    FROM {{ ref('dim_users') }} u
    JOIN {{ ref('dim_companies') }} c ON u.company_id = c.company_id
)
SELECT
    e.event_date,
    e.user_id,
    u.company_id,
    u.company_name,
    u.industry,
    u.plan,
    e.event_count,
    e.session_count,
    -- Days since signup
    DATEDIFF('day', u.signup_date, e.event_date) as days_since_signup
FROM daily_events e
JOIN user_details u ON e.user_id = u.user_id
```

#### **Day 2: Metrics & Analysis (4 hours)**

4. **Build metrics models:**

**Activation rate:**
```sql
-- models/marts/metrics/activation_metrics.sql
WITH user_first_key_event AS (
    SELECT
        user_id,
        MIN(event_timestamp) as first_key_event_date
    FROM {{ ref('fct_events') }}
    WHERE event_type = 'key_action'  -- Define what "activated" means
    GROUP BY user_id
),
users_with_activation AS (
    SELECT
        u.user_id,
        u.signup_date,
        u.plan,
        a.first_key_event_date,
        DATEDIFF('day', u.signup_date, a.first_key_event_date) as days_to_activate,
        CASE
            WHEN a.first_key_event_date IS NOT NULL
            AND DATEDIFF('day', u.signup_date, a.first_key_event_date) <= 7
            THEN 1 ELSE 0
        END as activated_in_7_days
    FROM {{ ref('dim_users') }} u
    LEFT JOIN user_first_key_event a ON u.user_id = a.user_id
)
SELECT
    DATE_TRUNC('month', signup_date) as cohort_month,
    plan,
    COUNT(*) as total_signups,
    SUM(activated_in_7_days) as activated_users,
    SUM(activated_in_7_days) * 100.0 / COUNT(*) as activation_rate,
    AVG(CASE WHEN activated_in_7_days = 1 THEN days_to_activate END) as avg_days_to_activate
FROM users_with_activation
GROUP BY 1, 2
```

**Retention analysis:**
```sql
-- models/marts/metrics/retention_metrics.sql
WITH cohorts AS (
    SELECT
        user_id,
        DATE_TRUNC('month', signup_date) as cohort_month
    FROM {{ ref('dim_users') }}
),
user_activity AS (
    SELECT
        user_id,
        DATE_TRUNC('month', event_timestamp) as activity_month
    FROM {{ ref('fct_events') }}
    GROUP BY 1, 2
),
cohort_retention AS (
    SELECT
        c.cohort_month,
        a.activity_month,
        DATEDIFF('month', c.cohort_month, a.activity_month) as months_since_signup,
        COUNT(DISTINCT a.user_id) as active_users
    FROM cohorts c
    LEFT JOIN user_activity a ON c.user_id = a.user_id
    GROUP BY 1, 2, 3
)
SELECT
    cohort_month,
    months_since_signup,
    active_users,
    active_users * 100.0 / FIRST_VALUE(active_users) OVER (
        PARTITION BY cohort_month
        ORDER BY months_since_signup
    ) as retention_rate
FROM cohort_retention
```

**MRR metrics:**
```sql
-- models/marts/metrics/mrr_metrics.sql
SELECT
    DATE_TRUNC('month', date) as month,
    SUM(mrr) as total_mrr,
    SUM(CASE WHEN is_new THEN mrr ELSE 0 END) as new_mrr,
    SUM(CASE WHEN is_expansion THEN mrr ELSE 0 END) as expansion_mrr,
    SUM(CASE WHEN is_contraction THEN mrr ELSE 0 END) as contraction_mrr,
    SUM(CASE WHEN is_churn THEN mrr ELSE 0 END) as churned_mrr,
    COUNT(DISTINCT company_id) as active_customers
FROM {{ ref('fct_mrr_movements') }}
GROUP BY 1
```

5. **Create executive dashboard queries:**
   - Weekly active users (WAU)
   - Feature adoption rate
   - Customer health score
   - Churn prediction indicators

6. **Documentation:**
   - Document all metrics definitions
   - Create data dictionary
   - Build lineage documentation

**Deliverable: Present to "CEO"**
- 5-slide presentation explaining key metrics
- SQL queries behind each metric
- Recommendations based on data

---

### **Case Study 2: E-commerce Analytics (Wednesday-Thursday)**

**Scenario:**
You're building analytics for online retail company.

**Given Data:**
- orders (order_id, customer_id, order_date, total_amount, status)
- order_items (order_item_id, order_id, product_id, quantity, price)
- products (product_id, name, category, cost, price)
- customers (customer_id, email, signup_date, country)
- marketing_campaigns (campaign_id, channel, cost, start_date, end_date)
- marketing_attributions (order_id, campaign_id)

**Your Tasks:**

#### **Day 1: Build Analytics Infrastructure (4 hours)**

1. **Dimensional model:**
   - fct_orders (order-level facts)
   - fct_order_items (line-item facts)
   - dim_customers (with RFM segmentation)
   - dim_products
   - dim_date
   - bridge_campaign_orders (many-to-many relationship)

2. **dbt models with business logic:**

**Customer RFM segmentation:**
```sql
-- models/marts/dim_customers_enriched.sql
WITH customer_metrics AS (
    SELECT
        customer_id,
        MAX(order_date) as last_order_date,
        DATEDIFF('day', MAX(order_date), CURRENT_DATE) as recency_days,
        COUNT(DISTINCT order_id) as frequency,
        SUM(total_amount) as monetary
    FROM {{ ref('fct_orders') }}
    GROUP BY customer_id
),
rfm_scores AS (
    SELECT
        customer_id,
        recency_days,
        frequency,
        monetary,
        -- Recency score (1-5, lower days = higher score)
        CASE
            WHEN recency_days <= 30 THEN 5
            WHEN recency_days <= 90 THEN 4
            WHEN recency_days <= 180 THEN 3
            WHEN recency_days <= 365 THEN 2
            ELSE 1
        END as recency_score,
        -- Frequency score (1-5, more orders = higher score)
        CASE
            WHEN frequency >= 10 THEN 5
            WHEN frequency >= 5 THEN 4
            WHEN frequency >= 3 THEN 3
            WHEN frequency >= 2 THEN 2
            ELSE 1
        END as frequency_score,
        -- Monetary score (1-5, higher spend = higher score)
        NTILE(5) OVER (ORDER BY monetary) as monetary_score
    FROM customer_metrics
),
customer_segments AS (
    SELECT
        *,
        -- Combined RFM segment
        CASE
            WHEN recency_score >= 4 AND frequency_score >= 4 THEN 'Champions'
            WHEN recency_score >= 3 AND frequency_score >= 3 THEN 'Loyal'
            WHEN recency_score >= 4 AND frequency_score <= 2 THEN 'Promising'
            WHEN recency_score <= 2 AND frequency_score >= 3 THEN 'At Risk'
            WHEN recency_score <= 2 AND frequency_score <= 2 THEN 'Lost'
            ELSE 'Regular'
        END as customer_segment
    FROM rfm_scores
)
SELECT
    c.customer_id,
    c.email,
    c.signup_date,
    c.country,
    cs.recency_days,
    cs.frequency,
    cs.monetary,
    cs.customer_segment
FROM {{ ref('stg_customers') }} c
LEFT JOIN customer_segments cs ON c.customer_id = cs.customer_id
```

**Product performance with ABC classification:**
```sql
-- models/marts/dim_products_enriched.sql
WITH product_metrics AS (
    SELECT
        product_id,
        SUM(quantity) as total_quantity_sold,
        SUM(quantity * price) as total_revenue,
        COUNT(DISTINCT order_id) as times_ordered
    FROM {{ ref('fct_order_items') }}
    WHERE order_date >= CURRENT_DATE - INTERVAL '90 days'
    GROUP BY product_id
),
revenue_cumulative AS (
    SELECT
        product_id,
        total_revenue,
        SUM(total_revenue) OVER (ORDER BY total_revenue DESC) as running_revenue,
        SUM(total_revenue) OVER () as total_revenue_all
    FROM product_metrics
),
abc_classification AS (
    SELECT
        product_id,
        total_revenue,
        running_revenue * 100.0 / total_revenue_all as cumulative_pct,
        CASE
            WHEN running_revenue * 100.0 / total_revenue_all <= 80 THEN 'A'
            WHEN running_revenue * 100.0 / total_revenue_all <= 95 THEN 'B'
            ELSE 'C'
        END as abc_class
    FROM revenue_cumulative
)
SELECT
    p.product_id,
    p.name,
    p.category,
    p.cost,
    p.price,
    pm.total_quantity_sold,
    pm.total_revenue,
    abc.abc_class
FROM {{ ref('stg_products') }} p
LEFT JOIN product_metrics pm ON p.product_id = pm.product_id
LEFT JOIN abc_classification abc ON p.product_id = abc.product_id
```

#### **Day 2: Advanced Analytics (4 hours)**

3. **Market basket analysis:**
```sql
-- models/marts/analytics/product_affinity.sql
-- Find products frequently bought together
WITH order_pairs AS (
    SELECT
        a.order_id,
        a.product_id as product_a,
        b.product_id as product_b
    FROM {{ ref('fct_order_items') }} a
    JOIN {{ ref('fct_order_items') }} b
        ON a.order_id = b.order_id
        AND a.product_id < b.product_id  -- Avoid duplicates
),
pair_counts AS (
    SELECT
        product_a,
        product_b,
        COUNT(*) as times_bought_together,
        COUNT(DISTINCT order_id) as order_count
    FROM order_pairs
    GROUP BY product_a, product_b
),
product_totals AS (
    SELECT
        product_id,
        COUNT(DISTINCT order_id) as total_orders
    FROM {{ ref('fct_order_items') }}
    GROUP BY product_id
)
SELECT
    pc.product_a,
    pa.name as product_a_name,
    pc.product_b,
    pb.name as product_b_name,
    pc.times_bought_together,
    -- Confidence: P(B|A) = orders with A & B / orders with A
    pc.times_bought_together * 100.0 / pta.total_orders as confidence_a_to_b,
    -- Lift: (A & B together) / (A alone * B alone)
    pc.times_bought_together * 1.0 / (pta.total_orders * ptb.total_orders) as lift
FROM pair_counts pc
JOIN {{ ref('dim_products') }} pa ON pc.product_a = pa.product_id
JOIN {{ ref('dim_products') }} pb ON pc.product_b = pb.product_id
JOIN product_totals pta ON pc.product_a = pta.product_id
JOIN product_totals ptb ON pc.product_b = ptb.product_id
WHERE pc.times_bought_together >= 5  -- Minimum threshold
ORDER BY pc.times_bought_together DESC
```

4. **Marketing attribution:**
```sql
-- models/marts/analytics/marketing_roi.sql
WITH campaign_performance AS (
    SELECT
        mc.campaign_id,
        mc.channel,
        mc.cost as campaign_cost,
        COUNT(DISTINCT ma.order_id) as attributed_orders,
        SUM(o.total_amount) as attributed_revenue
    FROM {{ ref('stg_marketing_campaigns') }} mc
    LEFT JOIN {{ ref('stg_marketing_attributions') }} ma
        ON mc.campaign_id = ma.campaign_id
    LEFT JOIN {{ ref('fct_orders') }} o
        ON ma.order_id = o.order_id
    GROUP BY mc.campaign_id, mc.channel, mc.cost
)
SELECT
    campaign_id,
    channel,
    campaign_cost,
    attributed_orders,
    attributed_revenue,
    attributed_revenue - campaign_cost as net_profit,
    (attributed_revenue - campaign_cost) * 100.0 / campaign_cost as roi_pct,
    campaign_cost * 1.0 / NULLIF(attributed_orders, 0) as cost_per_acquisition
FROM campaign_performance
ORDER BY roi_pct DESC
```

5. **Customer lifetime value (LTV):**
```sql
-- models/marts/analytics/customer_ltv.sql
WITH customer_orders AS (
    SELECT
        customer_id,
        MIN(order_date) as first_order_date,
        MAX(order_date) as last_order_date,
        COUNT(DISTINCT order_id) as total_orders,
        SUM(total_amount) as total_revenue,
        AVG(total_amount) as avg_order_value
    FROM {{ ref('fct_orders') }}
    GROUP BY customer_id
),
customer_ltv_calc AS (
    SELECT
        customer_id,
        total_revenue as historical_ltv,
        -- Simple LTV prediction: AOV * purchase frequency * avg customer lifespan
        avg_order_value *
        (total_orders * 1.0 / NULLIF(DATEDIFF('month', first_order_date, last_order_date), 0)) *
        24 as predicted_ltv_24months
    FROM customer_orders
    WHERE DATEDIFF('month', first_order_date, last_order_date) > 0
)
SELECT
    c.customer_id,
    c.email,
    c.customer_segment,
    co.total_orders,
    co.total_revenue,
    ltv.historical_ltv,
    ltv.predicted_ltv_24months
FROM {{ ref('dim_customers_enriched') }} c
JOIN customer_orders co ON c.customer_id = co.customer_id
JOIN customer_ltv_calc ltv ON c.customer_id = ltv.customer_id
```

**Deliverable:**
- Complete dbt project with 15+ models
- 5 key business metrics explained
- Recommendations for business

---

### **Case Study 3: BI Dashboard Requirements (Friday)**

**Scenario:**
Product manager asks: "Build me a dashboard to track our north star metrics"

**Your Tasks (4 hours):**

1. **Stakeholder interview simulation:**
   - What questions would you ask PM?
   - How do you define "north star metric"?
   - What decisions will this dashboard inform?

2. **Metrics definition document:**
```markdown
## Dashboard: Product North Star Metrics

### Purpose
Track key indicators of product health and growth

### Audience
Product team, executives

### Refresh Schedule
Daily at 6am UTC

### Metrics

#### 1. Active Users
- **Definition:** Unique users who performed key action in time period
- **DAU:** Daily active users
- **WAU:** Weekly active users (rolling 7 days)
- **MAU:** Monthly active users (rolling 30 days)
- **SQL:**
```sql
SELECT
    event_date,
    COUNT(DISTINCT user_id) as dau
FROM events
WHERE event_type IN ('key_action_1', 'key_action_2')
GROUP BY event_date
```

#### 2. Activation Rate
- **Definition:** % of new users who completed key action within 7 days
- **Target:** 40%
- **SQL:** (see activation metrics model)

#### 3. Retention Rate
- **Definition:** % of cohort still active after N months
- **Cohort:** Month of signup
- **SQL:** (see retention metrics model)

#### 4. Revenue Metrics
- **MRR:** Monthly Recurring Revenue
- **ARR:** Annual Run Rate (MRR * 12)
- **Growth rate:** MoM % change

### Data Model
- fct_events (source of active users)
- fct_subscriptions (source of MRR)
- dim_users (user attributes)

### Dimensions
- Time (day, week, month)
- User plan (free, pro, enterprise)
- User cohort (signup month)

### Filters
- Date range
- Plan type
- Country
```

3. **Create BI-ready models:**
```sql
-- models/marts/bi/dashboard_north_star.sql
-- Pre-aggregated table optimized for dashboard queries
{{ config(materialized='table', tags=['daily', 'dashboard']) }}

WITH date_spine AS (
    {{ dbt_utils.date_spine(
        datepart="day",
        start_date="cast('2024-01-01' as date)",
        end_date="cast(current_date as date)"
    ) }}
),
daily_actives AS (
    SELECT
        DATE(event_timestamp) as event_date,
        plan,
        COUNT(DISTINCT user_id) as dau
    FROM {{ ref('fct_events') }}
    WHERE event_type IN ('key_action')
    GROUP BY 1, 2
),
rolling_actives AS (
    SELECT
        da.event_date,
        da.plan,
        da.dau,
        -- WAU (rolling 7 days)
        SUM(da.dau) OVER (
            PARTITION BY da.plan
            ORDER BY da.event_date
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ) as wau,
        -- MAU (rolling 30 days)
        SUM(da.dau) OVER (
            PARTITION BY da.plan
            ORDER BY da.event_date
            ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
        ) as mau
    FROM daily_actives da
),
mrr_daily AS (
    SELECT
        date,
        plan,
        SUM(mrr) as daily_mrr,
        COUNT(DISTINCT company_id) as active_customers
    FROM {{ ref('fct_subscriptions') }}
    GROUP BY 1, 2
)
SELECT
    ds.date_day as metric_date,
    ra.plan,
    ra.dau,
    ra.wau,
    ra.mau,
    ra.dau * 100.0 / NULLIF(ra.mau, 0) as stickiness_ratio,
    mrr.daily_mrr,
    mrr.active_customers
FROM date_spine ds
LEFT JOIN rolling_actives ra ON ds.date_day = ra.event_date
LEFT JOIN mrr_daily mrr ON ds.date_day = mrr.date AND ra.plan = mrr.plan
```

**Deliverable:**
- Metrics definition doc
- SQL queries for each metric
- BI-ready data models
- Mockup of dashboard (can use ASCII art or describe)

---

### **Weekend: Practice Presentations**

**Saturday (4 hours):**

**Prepare 3 presentations (15 min each):**

1. **"Walk me through your dbt project"**
   - Project structure (staging → intermediate → marts)
   - Key design decisions
   - Testing strategy
   - Performance optimizations

2. **"Explain your dimensional model"**
   - Star schema design
   - Grain of fact tables
   - SCD strategy
   - Trade-offs made

3. **"Present key insights from your analysis"**
   - Top 3 findings
   - Business recommendations
   - Data supporting conclusions
   - Next steps

**Sunday (3 hours):**
- Record yourself presenting each case study
- Watch recordings, improve clarity
- Practice handling questions:
  - "Why did you choose X over Y?"
  - "How would this scale to 10x data?"
  - "What would you do differently?"

**📊 Week 6 Checkpoint:**
- [ ] Built 3 complete end-to-end projects
- [ ] Can explain technical decisions clearly
- [ ] Comfortable with stakeholder communication
- [ ] Portfolio-ready projects

---

## **WEEK 7: INTERVIEW SIMULATION & POLISH**

### **Monday: Mock SQL Interview**

**Find a partner or use interviewing.io**

**90-minute mock interview:**
- 15 min: Resume discussion
- 45 min: SQL coding (3 problems)
- 20 min: Query optimization discussion
- 10 min: Your questions for interviewer

**After interview:**
- Get feedback on communication
- Note questions you struggled with
- Practice those question types

---

### **Tuesday: Mock dbt Interview**

**60-minute mock interview:**
- 15 min: Explain your dbt project
- 30 min: Live coding (build incremental model + tests)
- 15 min: Architecture discussion

**Topics to prepare:**
- Model materialization trade-offs
- Testing strategy
- Handling schema changes
- dbt in production (orchestration, CI/CD)

---

### **Wednesday: Mock Data Modeling Interview**

**60-minute mock interview:**
- 45 min: Design dimensional model for given scenario
- 15 min: Discuss trade-offs and alternatives

**Practice scenarios:**
1. Ride-sharing company (Uber-like)
2. Social media platform
3. Healthcare patient management
4. Food delivery service

**What to cover:**
- Clarify requirements (5 min)
- Draw ERD (10 min)
- Explain fact/dimension tables (15 min)
- Handle edge cases (10 min)
- Scalability discussion (5 min)

---

### **Thursday: Mock Case Study Interview**

**2-hour mock interview:**
Given business scenario, build complete analytics solution

**Example:** "Our subscription service has 30% churn. Build analytics to understand why and recommend solutions."

**Deliverables:**
1. Data model design
2. Key metrics to track
3. SQL queries for analysis
4. Findings and recommendations

---

### **Friday: Behavioral Interview Prep**

**Prepare STAR stories for common questions:**

1. **"Tell me about a complex data project you built"**
   - Use one of your case studies from Week 6
   - Focus on: problem, your approach, technical decisions, impact

2. **"Describe a time you disagreed with a stakeholder"**
   - Use data modeling scenario (normalized vs denormalized)
   - Show: listening, explaining trade-offs, finding compromise

3. **"How do you ensure data quality?"**
   - Testing strategy (schema tests, data tests, custom tests)
   - Monitoring and alerting
   - Documentation
   - Examples from your dbt project

4. **"Tell me about a time you optimized performance"**
   - Incremental models
   - Query optimization
   - Partitioning strategy
   - Specific metrics (before/after)

5. **"How do you prioritize competing requests?"**
   - Framework: Impact vs effort matrix
   - Communication with stakeholders
   - Setting expectations

**Practice:**
- Write out STAR stories (Situation, Task, Action, Result)
- Time yourself (2-3 min per story)
- Practice out loud

---

### **Weekend: Final Review & Confidence Building**

**Saturday (4 hours):**

**Morning: Speed drill (2 hours)**
- 10 SQL problems (12 min each)
- Time yourself, no breaks
- Simulate interview pressure

**Afternoon: Review cheatsheets (2 hours)**
- All 10 topic cheatsheets
- Focus on interview talking points
- Create one-page "interview day" cheatsheet

---

**Sunday (3 hours):**

**Final checklist:**

#### **Technical Readiness**
- [ ] SQL: Can solve Medium problems in 20 min
- [ ] dbt: Can explain project architecture clearly
- [ ] Data Modeling: Can design star schema in 30 min
- [ ] Case Studies: Can deliver end-to-end solution

#### **Materials Ready**
- [ ] Portfolio projects documented (GitHub)
- [ ] Resume updated with skills
- [ ] LinkedIn profile updated
- [ ] Questions prepared for interviewer (5-10 questions)

#### **Communication**
- [ ] Can explain technical concepts simply
- [ ] STAR stories prepared (5+ stories)
- [ ] Practiced presentations (3+ times each)
- [ ] Comfortable with whiteboarding

#### **Logistics**
- [ ] Interview setup tested (camera, mic, internet)
- [ ] Quiet space arranged
- [ ] Water, snacks ready
- [ ] Backup plan if tech fails

#### **Mental Preparation**
- [ ] Good night's sleep planned
- [ ] Reviewed interview schedule
- [ ] Printed/saved key cheatsheets
- [ ] Confidence level: HIGH

---

**Relaxation time:**
- Light review only
- No new learning
- Visualize success
- Get 8+ hours sleep

---

## 📚 RESOURCES SUMMARY

### **Your Materials (Created)**
- `Python-SQL/100-QUESTIONS.md` - SQL interview prep
- `dbt/100-QUESTIONS.md` - dbt deep dive
- `Data-Modeling/100-QUESTIONS.md` - Dimensional modeling
- `Python-SQL/CHEATSHEET.md` - SQL quick reference
- `dbt/CHEATSHEET.md` - dbt quick reference
- `Data-Modeling/CHEATSHEET.md` - Modeling quick reference
- `LAYMAN-GUIDE.md` - Conceptual understanding

### **External Resources**

#### **SQL Practice**
- **LeetCode SQL** (free): 50+ problems, focus on Medium
- **HackerRank SQL** (free): Advanced challenges
- **Mode SQL Tutorial** (free): Real business scenarios
- **StrataScratch** (paid): Data science SQL questions

#### **dbt Learning**
- **dbt Learn** (free): Official dbt courses
- **dbt Discourse** (free): Community forum
- **dbt Docs** (free): Comprehensive documentation
- **Your dbt project**: Best learning resource!

#### **Mock Interviews**
- **Pramp** (free): Peer mock interviews
- **Interviewing.io** (paid): Anonymous technical interviews
- **Exponent** (paid): Analytics interview prep
- **Friends/colleagues**: Free, valuable feedback

#### **Data Modeling**
- **Kimball Group** (free articles): Dimensional modeling bible
- **"The Data Warehouse Toolkit"** (book): Ralph Kimball
- **dbt Blog** (free): Modern analytics architecture

---

## 🎯 INTERVIEW DAY CHECKLIST

### **Morning Of Interview**

**3 hours before:**
- [ ] Light breakfast
- [ ] Review cheatsheets (30 min max)
- [ ] Test tech setup

**1 hour before:**
- [ ] Review company/role notes
- [ ] Bathroom break
- [ ] Glass of water ready
- [ ] Close all tabs except interview link

**15 minutes before:**
- [ ] Deep breaths
- [ ] Positive self-talk: "I've prepared 7 weeks for this"
- [ ] Join meeting 5 min early

### **During Interview**

**For SQL/Coding Rounds:**
1. ✅ Read problem twice
2. ✅ Clarify requirements ("Should I handle nulls?")
3. ✅ Explain approach before coding
4. ✅ Think out loud while coding
5. ✅ Test with sample data
6. ✅ Discuss optimization

**For System Design:**
1. ✅ Ask clarifying questions (10 min)
2. ✅ State assumptions
3. ✅ Draw high-level design first
4. ✅ Dive deep into components
5. ✅ Discuss trade-offs
6. ✅ Address scalability

**For Behavioral:**
1. ✅ Use STAR format
2. ✅ Be specific (numbers, timelines)
3. ✅ Show impact
4. ✅ Be authentic
5. ✅ End with "What I learned"

### **After Each Round**
- [ ] Take notes on questions asked
- [ ] Note what went well
- [ ] Note what to improve
- [ ] Send thank-you email within 24 hours

---

## 🚀 YOU'RE READY!

### **What You've Accomplished**

✅ **700+ SQL problems solved** (100+ Analytics Engineering focused)
✅ **Built production-quality dbt project** with 20+ models, tests, docs
✅ **Designed 5+ dimensional models** with SCDs, fact tables, star schemas
✅ **Completed 3 end-to-end case studies** with business recommendations
✅ **Mastered analytics queries** (cohort, funnel, retention, A/B test)
✅ **Practiced presentations** explaining technical decisions
✅ **Prepared behavioral stories** with STAR format

### **Your Unique Strengths**

🌟 **Technical depth:** SQL + dbt + data modeling
🌟 **Business acumen:** Metrics, KPIs, stakeholder communication
🌟 **Modern stack:** dbt, dimensional modeling, analytics engineering
🌟 **Portfolio projects:** Real case studies to discuss
🌟 **Healthcare experience:** Domain expertise from Optum

### **Confidence Affirmations**

> "I've solved 700+ SQL problems. I can handle any SQL question."

> "I've built production dbt projects. I understand analytics engineering."

> "I've designed dimensional models end-to-end. I can explain trade-offs."

> "I've completed real case studies. I can deliver business value."

> "I'm prepared. I'm capable. I'm ready."

---

## 📞 FINAL WORDS

You've invested **7 weeks** and **100+ hours** preparing for Analytics Engineer interviews. You have:

- ✅ **900 questions** across 10 data engineering topics
- ✅ **10 cheatsheets** for quick reference
- ✅ **3 complete portfolio projects** to showcase
- ✅ **Real dbt project** deployed and documented
- ✅ **Dimensional models** designed and explained
- ✅ **STAR stories** prepared for behavioral questions

**You're not just prepared. You're over-prepared.**

Trust your preparation. Trust yourself.

**Go get that job! 🎉**

---

*Last Updated: 2026-05-08*
*Preparation Plan: 7 weeks, 100+ hours*
*Target Role: Analytics Engineer (Senior level)*
*Expected Outcome: Multiple offers, $150K-$200K+ compensation*

---

**Good luck! You've got this! 🚀**
