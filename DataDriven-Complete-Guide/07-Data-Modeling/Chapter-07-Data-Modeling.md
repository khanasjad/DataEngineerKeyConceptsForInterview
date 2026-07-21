# Chapter 7: Data Modeling for Data Engineering

**Master dimensional modeling, normalization, and data warehouse design**

---

## Table of Contents

1. [Introduction to Data Modeling](#1-introduction-to-data-modeling)
2. [Normalization vs Denormalization](#2-normalization-vs-denormalization)
3. [Dimensional Modeling Fundamentals](#3-dimensional-modeling-fundamentals)
4. [Star Schema](#4-star-schema)
5. [Snowflake Schema](#5-snowflake-schema)
6. [Fact Tables](#6-fact-tables)
7. [Dimension Tables](#7-dimension-tables)
8. [Slowly Changing Dimensions (SCD)](#8-slowly-changing-dimensions-scd)
9. [Preventing Double-Counting](#9-preventing-double-counting)
10. [Data Warehouse Design Patterns](#10-data-warehouse-design-patterns)
11. [Practice Problems](#11-practice-problems)
12. [Summary and Self-Assessment](#12-summary-and-self-assessment)

---

## 1. Introduction to Data Modeling

### What is Data Modeling?

**Data modeling** is the process of designing how data is structured, stored, and related in a database or data warehouse.

**Goal:** Create a structure that is:
- Easy to query
- Performs well
- Maintainable
- Accurate

### Types of Data Models

**1. Transactional (OLTP - Online Transaction Processing)**
```
Purpose: Support day-to-day operations
Example: E-commerce order system
Characteristics:
- Highly normalized (3NF)
- Fast writes
- Small transactions
- Avoids redundancy
```

**2. Analytical (OLAP - Online Analytical Processing)**
```
Purpose: Support analysis and reporting
Example: Sales data warehouse
Characteristics:
- Denormalized (star/snowflake schema)
- Fast reads
- Large aggregations
- Optimized for queries
```

**Comparison:**

| Aspect | OLTP (Transactional) | OLAP (Analytical) |
|--------|----------------------|-------------------|
| **Purpose** | Day-to-day operations | Analysis & reporting |
| **Schema** | Normalized (3NF) | Denormalized (star/snowflake) |
| **Operations** | INSERT, UPDATE, DELETE | SELECT, aggregations |
| **Users** | Many (thousands) | Few (analysts, BI) |
| **Data Volume** | Current data | Historical data |
| **Response Time** | Milliseconds | Seconds to minutes |
| **Example** | Shopping cart | Sales trends dashboard |

### Why Data Modeling Matters for Interviews

**Interview Frequency:** 20-30% of data engineering interviews

**Common Questions:**
- "Design a data model for an e-commerce warehouse"
- "What's the difference between star and snowflake schema?"
- "How do you handle slowly changing dimensions?"
- "How would you prevent double-counting in this scenario?"

**Skills Tested:**
- Understanding of dimensional modeling
- Trade-offs (normalization vs performance)
- Real-world problem-solving
- SQL proficiency

---

## 2. Normalization vs Denormalization

### Normalization

**Definition:** Organizing data to reduce redundancy and dependency.

**Normal Forms:**

**1st Normal Form (1NF):**
- No repeating groups
- Atomic values only
- Each cell contains single value

**Bad (Not 1NF):**
```
Orders Table:
| order_id | customer | products                        |
|----------|----------|---------------------------------|
| 1        | Alice    | "Apple, Banana, Orange"         |
| 2        | Bob      | "Laptop, Mouse"                 |
```

**Good (1NF):**
```
Orders Table:
| order_id | customer | product  |
|----------|----------|----------|
| 1        | Alice    | Apple    |
| 1        | Alice    | Banana   |
| 1        | Alice    | Orange   |
| 2        | Bob      | Laptop   |
| 2        | Bob      | Mouse    |
```

**2nd Normal Form (2NF):**
- Must be in 1NF
- No partial dependencies (all non-key attributes depend on entire primary key)

**Bad (Not 2NF):**
```
OrderItems Table:
| order_id | product_id | product_name | price | quantity |
|----------|------------|--------------|-------|----------|
| 1        | 101        | Laptop       | 999   | 1        |
| 1        | 102        | Mouse        | 25    | 2        |
```
Problem: `product_name` and `price` depend only on `product_id`, not on full key `(order_id, product_id)`

**Good (2NF):**
```
OrderItems Table:
| order_id | product_id | quantity |
|----------|------------|----------|
| 1        | 101        | 1        |
| 1        | 102        | 2        |

Products Table:
| product_id | product_name | price |
|------------|--------------|-------|
| 101        | Laptop       | 999   |
| 102        | Mouse        | 25    |
```

**3rd Normal Form (3NF):**
- Must be in 2NF
- No transitive dependencies (non-key attributes don't depend on other non-key attributes)

**Bad (Not 3NF):**
```
Orders Table:
| order_id | customer_id | customer_name | customer_city | customer_state |
|----------|-------------|---------------|---------------|----------------|
| 1        | 100         | Alice         | Seattle       | WA             |
| 2        | 101         | Bob           | Portland      | OR             |
```
Problem: `customer_city` and `customer_state` depend on `customer_id`, not directly on `order_id`

**Good (3NF):**
```
Orders Table:
| order_id | customer_id | order_date |
|----------|-------------|------------|
| 1        | 100         | 2024-01-15 |
| 2        | 101         | 2024-01-16 |

Customers Table:
| customer_id | customer_name | city     | state |
|-------------|---------------|----------|-------|
| 100         | Alice         | Seattle  | WA    |
| 101         | Bob           | Portland | OR    |
```

**Benefits of Normalization:**
- ✅ No data redundancy
- ✅ Easy to update (update in one place)
- ✅ Data integrity maintained
- ✅ Smaller storage footprint

**Drawbacks:**
- ❌ Many JOINs needed for queries
- ❌ Slower query performance
- ❌ Complex queries

### Denormalization

**Definition:** Intentionally introducing redundancy to improve query performance.

**Example:**
```sql
-- Normalized (requires JOIN)
SELECT o.order_id, c.customer_name, o.total
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;

-- Denormalized (no JOIN needed)
SELECT order_id, customer_name, total
FROM orders_denormalized;
```

**Denormalized Table:**
```
OrdersDenormalized:
| order_id | customer_id | customer_name | customer_city | total |
|----------|-------------|---------------|---------------|-------|
| 1        | 100         | Alice         | Seattle       | 1049  |
| 2        | 101         | Bob           | Portland      | 75    |
```

**Benefits of Denormalization:**
- ✅ Faster queries (fewer JOINs)
- ✅ Simpler SQL
- ✅ Better for analytics

**Drawbacks:**
- ❌ Data redundancy
- ❌ Update anomalies (must update multiple places)
- ❌ Larger storage

### When to Use Each

**Use Normalization (OLTP):**
- Transactional systems
- Frequent updates
- Data integrity critical
- Storage cost matters

**Use Denormalization (OLAP):**
- Data warehouses
- Read-heavy workloads
- Query performance critical
- Historical data (rarely updated)

---

## 3. Dimensional Modeling Fundamentals

### What is Dimensional Modeling?

**Dimensional modeling** is a design technique for data warehouses that organizes data into:
- **Fact tables:** Measurements/metrics (sales, clicks, revenue)
- **Dimension tables:** Context/attributes (who, what, when, where, why)

**Created by:** Ralph Kimball (considered the father of dimensional modeling)

**Philosophy:** Design for query simplicity and performance, not normalization.

### Core Concepts

**1. Facts (Measurements)**
- Numeric values you want to analyze
- Examples: revenue, quantity, count, duration

**2. Dimensions (Context)**
- Descriptive attributes
- Examples: customer, product, date, location

**3. Grain**
- Level of detail in fact table
- Must be clearly defined
- Example: "One row per order line item"

### Benefits of Dimensional Modeling

**1. Query Simplicity**
```sql
-- Simple star schema query
SELECT
    d.year,
    p.category,
    SUM(f.revenue) AS total_revenue
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
JOIN dim_product p ON f.product_key = p.product_key
WHERE d.year = 2024
GROUP BY d.year, p.category;
```

**2. Performance**
- Fewer JOINs than normalized schema
- Optimized for aggregations
- Indexes on dimension keys

**3. Understandable**
- Business users can understand structure
- Matches mental model of business

**4. Extensible**
- Easy to add new dimensions
- New facts can reference existing dimensions

---

## 4. Star Schema

### What is Star Schema?

**Star schema** is the simplest dimensional model, with one central fact table surrounded by denormalized dimension tables.

**Structure:**
```
        Dim_Product
              |
              |
Dim_Customer -- Fact_Sales -- Dim_Date
              |
              |
         Dim_Store
```

### Example: E-Commerce Star Schema

**Fact Table: Fact_Sales**
```
| sale_key | date_key | customer_key | product_key | store_key | quantity | revenue | cost |
|----------|----------|--------------|-------------|-----------|----------|---------|------|
| 1        | 20240115 | 1001         | 501         | 10        | 2        | 50.00   | 30.00|
| 2        | 20240115 | 1002         | 502         | 10        | 1        | 999.00  | 700.00|
| 3        | 20240116 | 1001         | 503         | 11        | 3        | 75.00   | 45.00|
```

**Dimension Table: Dim_Date**
```
| date_key | full_date  | day | month | year | quarter | day_of_week | is_weekend |
|----------|------------|-----|-------|------|---------|-------------|------------|
| 20240115 | 2024-01-15 | 15  | 1     | 2024 | Q1      | Monday      | No         |
| 20240116 | 2024-01-16 | 16  | 1     | 2024 | Q1      | Tuesday     | No         |
```

**Dimension Table: Dim_Customer**
```
| customer_key | customer_id | name  | city     | state | country | segment   |
|--------------|-------------|-------|----------|-------|---------|-----------|
| 1001         | C1001       | Alice | Seattle  | WA    | USA     | Premium   |
| 1002         | C1002       | Bob   | Portland | OR    | USA     | Standard  |
```

**Dimension Table: Dim_Product**
```
| product_key | product_id | name   | category    | subcategory | brand  | price |
|-------------|------------|--------|-------------|-------------|--------|-------|
| 501         | P501       | Mouse  | Electronics | Accessories | Logitech| 25.00|
| 502         | P502       | Laptop | Electronics | Computers   | Dell   | 999.00|
| 503         | P503       | Chair  | Furniture   | Office      | Herman | 25.00|
```

**Dimension Table: Dim_Store**
```
| store_key | store_id | store_name       | city     | state | region |
|-----------|----------|------------------|----------|-------|--------|
| 10        | S10      | Downtown Seattle | Seattle  | WA    | West   |
| 11        | S11      | Portland Mall    | Portland | OR    | West   |
```

### Querying Star Schema

**Example: Monthly revenue by product category**
```sql
SELECT
    d.year,
    d.month,
    p.category,
    SUM(f.revenue) AS total_revenue,
    SUM(f.quantity) AS total_quantity
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
JOIN dim_product p ON f.product_key = p.product_key
WHERE d.year = 2024
  AND d.quarter = 'Q1'
GROUP BY d.year, d.month, p.category
ORDER BY d.month, total_revenue DESC;
```

**Example: Customer segmentation analysis**
```sql
SELECT
    c.segment,
    c.state,
    COUNT(DISTINCT f.customer_key) AS num_customers,
    SUM(f.revenue) AS total_revenue,
    AVG(f.revenue) AS avg_order_value
FROM fact_sales f
JOIN dim_customer c ON f.customer_key = c.customer_key
JOIN dim_date d ON f.date_key = d.date_key
WHERE d.year = 2024
GROUP BY c.segment, c.state
ORDER BY total_revenue DESC;
```

### Advantages of Star Schema

**1. Simple Queries**
- Easy to understand and write
- Predictable JOIN pattern

**2. Fast Query Performance**
- Fewer JOINs (only fact to dimensions)
- Database optimizers handle well
- Good for aggregations

**3. Easy to Maintain**
- Simple structure
- Easy to add dimensions

**4. BI Tool Friendly**
- Tools like Tableau, Power BI work well with star schemas
- Automatic relationship detection

### Disadvantages

**1. Data Redundancy**
- Denormalized dimensions repeat data
- Example: Product category stored in every product row

**2. Storage Space**
- Larger than normalized schema
- Modern storage is cheap, so less of concern

**3. Update Complexity**
- Dimension updates require careful handling
- Slowly Changing Dimensions (covered later)

---

## 5. Snowflake Schema

### What is Snowflake Schema?

**Snowflake schema** is a normalized version of star schema where dimension tables are split into sub-dimensions.

**Structure:**
```
    Dim_Category
         |
    Dim_Product
         |
         |
Dim_Customer -- Fact_Sales -- Dim_Date
         |
         |
    Dim_City
         |
    Dim_State
```

### Example: Snowflake vs Star

**Star Schema (Denormalized):**
```
Dim_Product:
| product_key | product_name | category | subcategory | brand |
|-------------|--------------|----------|-------------|-------|
| 1           | Mouse        | Electronics | Accessories | Logitech |
| 2           | Laptop       | Electronics | Computers   | Dell     |
```

**Snowflake Schema (Normalized):**
```
Dim_Product:
| product_key | product_name | category_key | brand |
|-------------|--------------|--------------|-------|
| 1           | Mouse        | 10           | Logitech |
| 2           | Laptop       | 20           | Dell     |

Dim_Category:
| category_key | category    | subcategory |
|--------------|-------------|-------------|
| 10           | Electronics | Accessories |
| 20           | Electronics | Computers   |
```

### Querying Snowflake Schema

**Star Schema Query:**
```sql
SELECT
    p.category,
    SUM(f.revenue)
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
GROUP BY p.category;
```

**Snowflake Schema Query (Requires Extra JOIN):**
```sql
SELECT
    c.category,
    SUM(f.revenue)
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_category c ON p.category_key = c.category_key  -- Extra JOIN
GROUP BY c.category;
```

### Star vs Snowflake Comparison

| Aspect | Star Schema | Snowflake Schema |
|--------|-------------|------------------|
| **Normalization** | Denormalized dimensions | Normalized dimensions |
| **JOINs** | Fewer (fact to dims) | More (dims to sub-dims) |
| **Query Complexity** | Simpler | More complex |
| **Query Performance** | Faster | Slower (more JOINs) |
| **Storage** | More (redundancy) | Less (normalized) |
| **Maintenance** | Harder (update in many places) | Easier (update once) |
| **Use Case** | Most data warehouses | Large dimensions, storage critical |

### When to Use Snowflake Schema

**Use Snowflake When:**
- Very large dimensions (millions of rows)
- Storage cost is critical
- Dimensions change frequently
- Need to enforce referential integrity

**Example:** Product catalog with 10 million products
- Snowflake: Store category once, reference from products
- Star: Store category in every product row (10M × category data)

**However:** Modern data warehouses (Snowflake, BigQuery) have cheap storage, so **star schema is preferred** in most cases.

---

## 6. Fact Tables

### What is a Fact Table?

**Fact table** stores measurable events or transactions. Each row represents a business event.

**Characteristics:**
- Many rows (millions to billions)
- Foreign keys to dimension tables
- Numeric measures (facts)
- Grain must be clearly defined

### Types of Facts

**1. Additive Facts**
Can be summed across all dimensions.

```sql
-- Revenue is additive
SELECT
    SUM(revenue) AS total_revenue  -- Can sum across any dimension
FROM fact_sales
WHERE year = 2024;
```

**Examples:** revenue, cost, quantity, count

**2. Semi-Additive Facts**
Can be summed across some dimensions, but not all.

```sql
-- Account balance is semi-additive
SELECT
    customer_id,
    SUM(balance)  -- Can sum across customers
FROM fact_account_balance
WHERE date = '2024-01-15'
GROUP BY customer_id;

-- WRONG (can't sum balance across time)
SELECT SUM(balance) FROM fact_account_balance;  -- Incorrect!

-- CORRECT (use AVG or snapshot)
SELECT AVG(balance) FROM fact_account_balance WHERE date = '2024-01-15';
```

**Examples:** balance, inventory level, head count

**3. Non-Additive Facts**
Cannot be summed across any dimension.

```sql
-- Unit price is non-additive
SELECT AVG(unit_price)  -- Use AVG, MIN, MAX instead
FROM fact_sales;
```

**Examples:** price, ratio, percentage

### Types of Fact Tables

**1. Transaction Fact Table**
- One row per transaction/event
- Most common type
- Additive facts

```
Fact_Sales:
| sale_key | date_key | customer_key | product_key | quantity | revenue |
|----------|----------|--------------|-------------|----------|---------|
| 1        | 20240115 | 100          | 500         | 2        | 50.00   |
| 2        | 20240115 | 101          | 501         | 1        | 999.00  |
```

**Grain:** One row per sale transaction

**2. Periodic Snapshot Fact Table**
- One row per time period
- Accumulates metrics over period
- Semi-additive facts

```
Fact_Monthly_Inventory:
| snapshot_key | date_key | product_key | warehouse_key | beginning_balance | ending_balance | units_sold |
|--------------|----------|-------------|---------------|-------------------|----------------|------------|
| 1            | 20240131 | 500         | 10            | 100               | 75             | 25         |
| 2            | 20240229 | 500         | 10            | 75                | 50             | 25         |
```

**Grain:** One row per product per warehouse per month

**3. Accumulating Snapshot Fact Table**
- One row per process/workflow
- Updated as milestones reached
- Multiple date columns

```
Fact_Order_Fulfillment:
| order_key | customer_key | order_date | payment_date | ship_date | delivery_date | order_amount | days_to_ship |
|-----------|--------------|------------|--------------|-----------|---------------|--------------|--------------|
| 1         | 100          | 2024-01-15 | 2024-01-15   | 2024-01-16| 2024-01-18    | 1049.00      | 1            |
| 2         | 101          | 2024-01-16 | 2024-01-16   | NULL      | NULL          | 75.00        | NULL         |
```

**Grain:** One row per order (updated as order progresses)

### Fact Table Design Best Practices

**1. Clearly Define Grain**
```sql
-- Good: Clear grain
-- "One row per order line item per day"
Fact_Daily_Sales:
| date_key | order_id | line_item | product_key | quantity | revenue |

-- Bad: Mixed grain
-- "One row per order... or maybe per product?"
Fact_Confused:
| date_key | order_id | product_key | ??? |
```

**2. Use Surrogate Keys**
```sql
-- Good: Surrogate key for dimensions
| sale_key | date_key | customer_key | product_key |
|----------|----------|--------------|-------------|
| 1        | 20240115 | 1001         | 501         |

-- Bad: Natural keys (vulnerable to changes)
| sale_key | date | customer_id | product_id |
|----------|------|-------------|------------|
| 1        | 2024-01-15 | C1001 | P501    |
```

**3. Keep Facts Numeric**
```sql
-- Good: All facts are numeric
| quantity | revenue | cost | profit |
|----------|---------|------|--------|
| 2        | 50.00   | 30.00| 20.00  |

-- Bad: Text in fact table
| quantity | revenue | status   |  ❌
|----------|---------|----------|
| 2        | 50.00   | "shipped"|
```

**4. Denormalize Carefully**
```sql
-- Sometimes OK to denormalize frequently used attributes
Fact_Sales:
| date_key | customer_key | product_key | quantity | revenue | product_name |
-- product_name duplicated for query convenience
```

---

## 7. Dimension Tables

### What is a Dimension Table?

**Dimension table** provides context for facts. Contains descriptive attributes used for filtering and grouping.

**Characteristics:**
- Fewer rows than fact tables (thousands to millions)
- Wide (many columns)
- Text attributes
- Denormalized in star schema

### Dimension Design

**Example: Dim_Customer**
```
| customer_key | customer_id | name  | email           | city     | state | country | segment | lifetime_value | created_date |
|--------------|-------------|-------|-----------------|----------|-------|---------|---------|----------------|--------------|
| 1001         | C1001       | Alice | alice@email.com | Seattle  | WA    | USA     | Premium | 5000.00        | 2020-01-15   |
| 1002         | C1002       | Bob   | bob@email.com   | Portland | OR    | USA     | Standard| 1200.00        | 2021-06-10   |
```

### Types of Dimensions

**1. Conformed Dimension**
- Shared across multiple fact tables
- Ensures consistency

```
Dim_Date (used by multiple facts):
    ├── Fact_Sales
    ├── Fact_Inventory
    └── Fact_Website_Traffic
```

**2. Role-Playing Dimension**
- Same dimension used multiple times with different context

```
Dim_Date:
    ├── order_date_key
    ├── ship_date_key
    └── delivery_date_key

Fact_Orders:
| order_key | order_date_key | ship_date_key | delivery_date_key |
```

**3. Degenerate Dimension**
- Dimension data stored in fact table (no separate dimension)
- Example: Order number, invoice number

```
Fact_Sales:
| sale_key | order_number | invoice_number | ... |
--         ^^^^^^^^^^^^^ ^^^^^^^^^^^^^^^ degenerate dimensions
```

**4. Junk Dimension**
- Combines low-cardinality flags/indicators

**Bad (Many binary columns in fact):**
```
Fact_Sales:
| sale_key | is_promo | is_weekend | is_online | payment_type | ... |
```

**Good (Junk dimension):**
```
Fact_Sales:
| sale_key | transaction_type_key | ... |

Dim_Transaction_Type:
| transaction_type_key | is_promo | is_weekend | is_online | payment_type |
|----------------------|----------|------------|-----------|--------------|
| 1                    | Yes      | No         | Yes       | Credit       |
| 2                    | No       | Yes        | Yes       | Debit        |
| 3                    | No       | No         | No        | Cash         |
```

### Date Dimension

**Most important dimension in data warehouse!**

**Example: Dim_Date**
```sql
CREATE TABLE dim_date (
    date_key INT PRIMARY KEY,  -- 20240115
    full_date DATE,             -- 2024-01-15
    day_of_month INT,           -- 15
    day_of_week INT,            -- 1 (Monday)
    day_name VARCHAR(10),       -- 'Monday'
    day_of_year INT,            -- 15
    week_of_year INT,           -- 3
    month INT,                  -- 1
    month_name VARCHAR(10),     -- 'January'
    quarter INT,                -- 1
    quarter_name VARCHAR(10),   -- 'Q1'
    year INT,                   -- 2024
    is_weekend BOOLEAN,         -- FALSE
    is_holiday BOOLEAN,         -- FALSE
    holiday_name VARCHAR(50),   -- NULL
    fiscal_year INT,            -- 2024
    fiscal_quarter INT          -- 1
);
```

**Why Date Dimension?**
- Enables rich date-based analysis
- Pre-computed attributes (no DATE functions in queries)
- Fiscal calendars
- Holiday tracking

**Query Example:**
```sql
-- Without date dimension (uses DATE functions)
SELECT
    EXTRACT(YEAR FROM sale_date) AS year,
    EXTRACT(QUARTER FROM sale_date) AS quarter,
    SUM(revenue)
FROM fact_sales
GROUP BY year, quarter;

-- With date dimension (simpler, faster)
SELECT
    d.year,
    d.quarter_name,
    SUM(f.revenue)
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
GROUP BY d.year, d.quarter_name;
```

### Surrogate Keys vs Natural Keys

**Natural Key:**
- Business identifier (customer_id, product_code)
- Can change
- May be alphanumeric

**Surrogate Key:**
- System-generated integer
- Never changes
- Sequential (1, 2, 3, ...)

**Example:**
```
Dim_Product:
| product_key | product_id | product_name | ... |
  ^^^^^^^^^     ^^^^^^^^^^
  Surrogate     Natural

| 1           | P-1001     | Laptop       |
| 2           | P-1002     | Mouse        |
```

**Why Use Surrogate Keys?**
1. **Handle changes:** Natural keys can change (product P-1001 renamed to P-2001)
2. **Performance:** Integer JOINs faster than string JOINs
3. **Slowly Changing Dimensions:** Track history without breaking foreign keys
4. **Independence:** Warehouse independent of source system key changes

---

## 8. Slowly Changing Dimensions (SCD)

### What are Slowly Changing Dimensions?

**SCD** handles how dimension attributes change over time.

**Example:** Customer moves from Seattle to Portland
- How do we track this change?
- Do we preserve history?
- How do old facts relate to new customer data?

### SCD Type 1: Overwrite

**Strategy:** Overwrite old value with new value (no history).

**Example:**
```sql
-- Before: Customer moves
| customer_key | customer_id | name  | city    | state |
|--------------|-------------|-------|---------|-------|
| 1001         | C1001       | Alice | Seattle | WA    |

-- After: Update city
UPDATE dim_customer
SET city = 'Portland', state = 'OR'
WHERE customer_key = 1001;

| customer_key | customer_id | name  | city     | state |
|--------------|-------------|-------|----------|-------|
| 1001         | C1001       | Alice | Portland | OR    |
```

**Characteristics:**
- Simple
- No history preserved
- Old facts now point to updated dimension

**Use When:**
- History not important
- Corrections (fixing errors)
- Attributes that don't affect analysis (email, phone)

**Example Query Impact:**
```sql
-- Historical sales in Seattle will now show as Portland!
SELECT
    c.city,
    SUM(f.revenue)
FROM fact_sales f
JOIN dim_customer c ON f.customer_key = c.customer_key
WHERE f.date_key = 20240115  -- Sale was in Seattle
GROUP BY c.city;

-- Result: Portland (even though sale was in Seattle!)
```

### SCD Type 2: Add New Row

**Strategy:** Add new row with new values, keep old row for history.

**Example:**
```sql
-- Before: Customer moves
| customer_key | customer_id | name  | city    | state | effective_date | expiration_date | is_current |
|--------------|-------------|-------|---------|-------|----------------|-----------------|------------|
| 1001         | C1001       | Alice | Seattle | WA    | 2020-01-01     | 9999-12-31      | TRUE       |

-- After: Add new row, expire old row
| customer_key | customer_id | name  | city     | state | effective_date | expiration_date | is_current |
|--------------|-------------|-------|----------|-------|----------------|-----------------|------------|
| 1001         | C1001       | Alice | Seattle  | WA    | 2020-01-01     | 2024-01-15      | FALSE      |
| 1002         | C1001       | Alice | Portland | OR    | 2024-01-16     | 9999-12-31      | TRUE       |
```

**Key Fields:**
- `effective_date`: When this row became active
- `expiration_date`: When this row expired (9999-12-31 = current)
- `is_current`: Flag for current row

**Characteristics:**
- Complete history preserved
- New surrogate key for each change
- Old facts point to old version

**Use When:**
- History is critical
- Need point-in-time accuracy
- Regulatory requirements (finance, healthcare)

**Example Query:**
```sql
-- Sales by customer's current city
SELECT
    c.city,
    SUM(f.revenue)
FROM fact_sales f
JOIN dim_customer c ON f.customer_key = c.customer_key
WHERE c.is_current = TRUE
GROUP BY c.city;

-- Sales by customer's city at time of sale (point-in-time)
SELECT
    c.city,
    SUM(f.revenue)
FROM fact_sales f
JOIN dim_customer c ON f.customer_key = c.customer_key
  AND f.date_key BETWEEN c.effective_date AND c.expiration_date
GROUP BY c.city;
-- Result: Seattle (correct historical city!)
```

**Implementation:**
```sql
-- When customer moves, run this logic:

-- 1. Expire old record
UPDATE dim_customer
SET expiration_date = '2024-01-15',
    is_current = FALSE
WHERE customer_id = 'C1001'
  AND is_current = TRUE;

-- 2. Insert new record
INSERT INTO dim_customer (customer_id, name, city, state, effective_date, expiration_date, is_current)
VALUES ('C1001', 'Alice', 'Portland', 'OR', '2024-01-16', '9999-12-31', TRUE);
```

### SCD Type 3: Add New Column

**Strategy:** Add column to track previous value.

**Example:**
```sql
-- Before: Customer moves
| customer_key | customer_id | name  | current_city | current_state | prior_city | prior_state |
|--------------|-------------|-------|--------------|---------------|------------|-------------|
| 1001         | C1001       | Alice | Seattle      | WA            | NULL       | NULL        |

-- After: Update current, save prior
UPDATE dim_customer
SET prior_city = current_city,
    prior_state = current_state,
    current_city = 'Portland',
    current_state = 'OR'
WHERE customer_key = 1001;

| customer_key | customer_id | name  | current_city | current_state | prior_city | prior_state |
|--------------|-------------|-------|--------------|---------------|------------|-------------|
| 1001         | C1001       | Alice | Portland     | OR            | Seattle    | WA          |
```

**Characteristics:**
- Limited history (only previous value)
- No new rows
- Same surrogate key

**Use When:**
- Need to track one previous value
- "Before and after" comparisons
- Limited history requirements

**Example Query:**
```sql
-- Customers who moved states
SELECT
    customer_id,
    name,
    prior_state,
    current_state
FROM dim_customer
WHERE prior_state != current_state
  AND prior_state IS NOT NULL;
```

### SCD Type 4: History Table

**Strategy:** Separate current and historical tables.

**Example:**
```sql
-- Current table (Type 1)
Dim_Customer_Current:
| customer_key | customer_id | name  | city     | state |
|--------------|-------------|-------|----------|-------|
| 1001         | C1001       | Alice | Portland | OR    |

-- History table (Type 2)
Dim_Customer_History:
| customer_key | customer_id | name  | city    | state | effective_date | expiration_date |
|--------------|-------------|-------|---------|-------|----------------|-----------------|
| 1001         | C1001       | Alice | Seattle | WA    | 2020-01-01     | 2024-01-15      |
| 1002         | C1001       | Alice | Portland| OR    | 2024-01-16     | 9999-12-31      |
```

**Use When:**
- Most queries use current data
- Historical queries rare
- Want to optimize for common case

### SCD Comparison

| Type | History | Storage | Complexity | Use Case |
|------|---------|---------|------------|----------|
| **Type 1** | None | Low | Simple | Corrections, non-critical attributes |
| **Type 2** | Complete | High | Complex | Regulatory, point-in-time analysis |
| **Type 3** | Limited (1 prior) | Medium | Medium | Before/after comparisons |
| **Type 4** | Separate table | Medium | Medium | Mostly current queries |

**Most Common:** Type 1 (simple attributes) and Type 2 (important attributes)

**Hybrid Approach:**
- Type 1 for email, phone (corrections)
- Type 2 for address, segment (history matters)

---

## 9. Preventing Double-Counting

### What is Double-Counting?

**Double-counting** occurs when the same metric is counted multiple times due to JOIN multiplicities or incorrect aggregation.

**Common Scenarios:**
1. One-to-many relationships
2. Multiple fact tables at different grains
3. Incorrect aggregation order

### Scenario 1: One-to-Many JOIN

**Problem:**
```sql
-- Orders table
Orders:
| order_id | customer_id | order_amount |
|----------|-------------|--------------|
| 1        | 100         | 50           |
| 2        | 100         | 75           |

-- Order Items table (one order has multiple items)
OrderItems:
| order_id | item_id | quantity |
|----------|---------|----------|
| 1        | 1       | 2        |
| 1        | 2       | 1        |
| 2        | 1       | 3        |

-- WRONG: Join causes duplication
SELECT
    SUM(o.order_amount) AS total_revenue
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id;

-- Result: 200 (should be 125)
-- Order 1 counted twice (50 + 50 = 100)
-- Order 2 counted once (75)
```

**Why:** Order 1 joins with 2 items, so order_amount (50) appears twice.

**Solution 1: Aggregate Before JOIN**
```sql
SELECT
    SUM(o.order_amount) AS total_revenue
FROM orders o;
-- Result: 125 ✓
```

**Solution 2: DISTINCT or Subquery**
```sql
SELECT
    SUM(order_amount)
FROM (
    SELECT DISTINCT
        o.order_id,
        o.order_amount
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
) subquery;
-- Result: 125 ✓
```

**Solution 3: COUNT DISTINCT**
```sql
-- If you need both order count and item count
SELECT
    COUNT(DISTINCT o.order_id) AS num_orders,
    COUNT(oi.item_id) AS num_items
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id;
-- num_orders: 2, num_items: 3
```

### Scenario 2: Multiple Fact Tables

**Problem:**
```sql
-- Fact_Sales (daily grain)
Fact_Sales:
| date_key | product_key | daily_revenue |
|----------|-------------|---------------|
| 20240115 | 100         | 500           |
| 20240116 | 100         | 600           |

-- Fact_Inventory (daily snapshot)
Fact_Inventory:
| date_key | product_key | units_in_stock |
|----------|-------------|----------------|
| 20240115 | 100         | 1000           |
| 20240116 | 100         | 900            |

-- WRONG: Join fact tables
SELECT
    p.product_name,
    SUM(s.daily_revenue) AS total_revenue,
    SUM(i.units_in_stock) AS total_stock
FROM fact_sales s
JOIN fact_inventory i ON s.date_key = i.date_key
  AND s.product_key = i.product_key
JOIN dim_product p ON s.product_key = p.product_key
GROUP BY p.product_name;

-- total_stock: 1900 (should not sum across dates!)
```

**Why:** Inventory is semi-additive (can't sum across time).

**Solution: Separate Queries or Use Window Functions**
```sql
-- Correct approach 1: Separate queries
SELECT
    p.product_name,
    SUM(s.daily_revenue) AS total_revenue
FROM fact_sales s
JOIN dim_product p ON s.product_key = p.product_key
GROUP BY p.product_name;

SELECT
    p.product_name,
    i.units_in_stock AS current_stock
FROM fact_inventory i
JOIN dim_product p ON i.product_key = p.product_key
WHERE i.date_key = 20240116;  -- Latest date only

-- Correct approach 2: Use latest inventory in one query
WITH latest_inventory AS (
    SELECT
        product_key,
        units_in_stock,
        ROW_NUMBER() OVER (PARTITION BY product_key ORDER BY date_key DESC) AS rn
    FROM fact_inventory
)
SELECT
    p.product_name,
    SUM(s.daily_revenue) AS total_revenue,
    MAX(li.units_in_stock) AS current_stock
FROM fact_sales s
JOIN dim_product p ON s.product_key = p.product_key
LEFT JOIN latest_inventory li ON s.product_key = li.product_key
  AND li.rn = 1
GROUP BY p.product_name;
```

### Scenario 3: Dimension with Multiple Attributes

**Problem:**
```sql
-- Dim_Product (snowflake style)
Dim_Product:
| product_key | product_name | category_key |
|-------------|--------------|--------------|
| 100         | Laptop       | 10           |
| 101         | Mouse        | 10           |

Dim_Category:
| category_key | category    | region    |
|--------------|-------------|-----------|
| 10           | Electronics | North     |
| 10           | Electronics | South     |  -- Category operates in multiple regions!

-- WRONG: Join causes duplication
SELECT
    c.category,
    SUM(f.revenue) AS total_revenue
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_category c ON p.category_key = c.category_key
GROUP BY c.category;

-- Revenue doubled because Electronics appears twice in category table!
```

**Solution: Fix Dimension Model**
```sql
-- Correct: Separate category and region
Dim_Category:
| category_key | category    |
|--------------|-------------|
| 10           | Electronics |

Dim_Category_Region:
| category_key | region |
|--------------|--------|
| 10           | North  |
| 10           | South  |

-- Now query works correctly
SELECT
    c.category,
    SUM(f.revenue) AS total_revenue
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_category c ON p.category_key = c.category_key
GROUP BY c.category;
```

### Best Practices to Avoid Double-Counting

**1. Understand Grain**
- Know the grain of each fact table
- Don't JOIN facts with different grains

**2. COUNT DISTINCT When Needed**
```sql
-- Always use DISTINCT for counts across JOINs
COUNT(DISTINCT customer_id) AS num_customers
```

**3. Aggregate Before JOIN**
```sql
-- Pre-aggregate to avoid duplication
WITH order_totals AS (
    SELECT
        order_id,
        SUM(item_amount) AS total_amount
    FROM order_items
    GROUP BY order_id
)
SELECT ...
FROM orders o
JOIN order_totals ot ON o.order_id = ot.order_id;
```

**4. Test with Known Data**
```sql
-- Always verify results with manual calculation
SELECT SUM(revenue) FROM fact_sales WHERE date_key = 20240115;
-- Check against source system
```

**5. Use Assertions**
```sql
-- Add checks to ETL
-- Assert: fact table rows = source rows
SELECT
    CASE
        WHEN (SELECT COUNT(*) FROM fact_sales WHERE date_key = 20240115) =
             (SELECT COUNT(*) FROM staging_sales WHERE date = '2024-01-15')
        THEN 'PASS'
        ELSE 'FAIL - Row count mismatch!'
    END AS row_count_check;
```

---

## 10. Data Warehouse Design Patterns

### Pattern 1: Kimball Methodology (Bottom-Up)

**Approach:** Build data marts first, then integrate.

**Steps:**
1. Identify business process (sales, inventory, etc.)
2. Declare grain (one row per order line item)
3. Identify dimensions (customer, product, date)
4. Identify facts (quantity, revenue, cost)
5. Build dimensional model (star schema)
6. Repeat for each business process
7. Use conformed dimensions to integrate

**Advantages:**
- Fast time-to-value (deliver one data mart at a time)
- Business-focused
- Easy to understand

**Disadvantages:**
- Can lead to data silos
- Integration challenges

### Pattern 2: Inmon Methodology (Top-Down)

**Approach:** Build normalized enterprise data warehouse first, then create data marts.

**Steps:**
1. Design normalized enterprise model (3NF)
2. Build centralized data warehouse
3. Create denormalized data marts from warehouse

**Advantages:**
- Single source of truth
- Data consistency
- Better long-term scalability

**Disadvantages:**
- Longer time to delivery
- More complex
- Higher upfront cost

### Pattern 3: Data Vault

**Approach:** Agile, scalable warehouse design with Hubs, Links, and Satellites.

**Components:**
- **Hub:** Business keys (customer_id, product_id)
- **Link:** Relationships between hubs (customer bought product)
- **Satellite:** Attributes and history (customer name, address - changes over time)

**Example:**
```
Hub_Customer:
| customer_hub_key | customer_id | load_date |
|------------------|-------------|-----------|
| 1                | C1001       | 2024-01-01|

Satellite_Customer:
| customer_hub_key | name  | city    | load_date  | load_end_date |
|------------------|-------|---------|------------|---------------|
| 1                | Alice | Seattle | 2024-01-01 | 2024-01-15    |
| 1                | Alice | Portland| 2024-01-16 | 9999-12-31    |

Link_Order:
| order_link_key | customer_hub_key | product_hub_key | load_date |
|----------------|------------------|-----------------|-----------|
| 1              | 1                | 100             | 2024-01-15|
```

**Advantages:**
- Handles complex changes well
- Audit trail built-in
- Parallel loading

**Disadvantages:**
- Complex to query
- Requires transformation layer for BI
- Steeper learning curve

### Pattern 4: One Big Table (OBT)

**Approach:** Denormalize everything into one wide table.

**Example:**
```sql
CREATE TABLE one_big_table AS
SELECT
    f.sale_key,
    f.quantity,
    f.revenue,
    d.full_date,
    d.year,
    d.quarter,
    c.customer_name,
    c.city AS customer_city,
    c.state AS customer_state,
    p.product_name,
    p.category,
    p.brand
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
JOIN dim_customer c ON f.customer_key = c.customer_key
JOIN dim_product p ON f.product_key = p.product_key;
```

**Advantages:**
- Simplest queries (no JOINs)
- Fast for specific use case
- Good for BI tools

**Disadvantages:**
- Massive data redundancy
- Expensive storage
- Update complexity
- Not flexible

**Use When:**
- Small dataset
- Single use case (specific dashboard)
- BI tool limitation (can't handle JOINs well)

---

## 11. Practice Problems

### Problem 1: Design E-Commerce Data Warehouse

**Scenario:**
Design a star schema for an e-commerce company that sells products online. The business wants to analyze:
- Sales by product category
- Sales by customer segment
- Sales trends over time
- Store performance (they have physical stores too)

**Requirements:**
- Track customer purchases over time
- Handle customer address changes (need history)
- Daily sales reporting

<details>
<summary>Click for Solution</summary>

**Star Schema Design:**

**Fact Table: Fact_Sales**
```sql
CREATE TABLE fact_sales (
    sale_key BIGINT PRIMARY KEY,
    date_key INT NOT NULL,
    customer_key INT NOT NULL,
    product_key INT NOT NULL,
    store_key INT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    revenue DECIMAL(10,2) NOT NULL,  -- quantity * unit_price
    cost DECIMAL(10,2) NOT NULL,
    profit DECIMAL(10,2) NOT NULL,   -- revenue - cost
    FOREIGN KEY (date_key) REFERENCES dim_date(date_key),
    FOREIGN KEY (customer_key) REFERENCES dim_customer(customer_key),
    FOREIGN KEY (product_key) REFERENCES dim_product(product_key),
    FOREIGN KEY (store_key) REFERENCES dim_store(store_key)
);
```

**Grain:** One row per order line item

**Dimension: Dim_Date**
```sql
CREATE TABLE dim_date (
    date_key INT PRIMARY KEY,  -- YYYYMMDD format
    full_date DATE NOT NULL,
    day INT NOT NULL,
    month INT NOT NULL,
    year INT NOT NULL,
    quarter VARCHAR(2) NOT NULL,
    day_of_week VARCHAR(10) NOT NULL,
    is_weekend BOOLEAN NOT NULL,
    is_holiday BOOLEAN NOT NULL
);
```

**Dimension: Dim_Customer (SCD Type 2)**
```sql
CREATE TABLE dim_customer (
    customer_key INT PRIMARY KEY,      -- Surrogate key
    customer_id VARCHAR(50) NOT NULL,  -- Natural key
    customer_name VARCHAR(100) NOT NULL,
    email VARCHAR(100),
    street_address VARCHAR(200),
    city VARCHAR(50),
    state VARCHAR(50),
    country VARCHAR(50),
    zip_code VARCHAR(20),
    segment VARCHAR(20),  -- Premium, Standard, etc.
    effective_date DATE NOT NULL,
    expiration_date DATE NOT NULL,
    is_current BOOLEAN NOT NULL
);
```

**Dimension: Dim_Product**
```sql
CREATE TABLE dim_product (
    product_key INT PRIMARY KEY,
    product_id VARCHAR(50) NOT NULL,
    product_name VARCHAR(100) NOT NULL,
    category VARCHAR(50) NOT NULL,
    subcategory VARCHAR(50),
    brand VARCHAR(50),
    unit_cost DECIMAL(10,2) NOT NULL,
    list_price DECIMAL(10,2) NOT NULL
);
```

**Dimension: Dim_Store**
```sql
CREATE TABLE dim_store (
    store_key INT PRIMARY KEY,
    store_id VARCHAR(50) NOT NULL,
    store_name VARCHAR(100) NOT NULL,
    store_type VARCHAR(20),  -- Physical, Online
    city VARCHAR(50),
    state VARCHAR(50),
    country VARCHAR(50),
    region VARCHAR(20),  -- West, East, etc.
    open_date DATE
);
```

**Example Queries:**

```sql
-- Sales by product category and quarter
SELECT
    d.year,
    d.quarter,
    p.category,
    SUM(f.revenue) AS total_revenue,
    SUM(f.profit) AS total_profit
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
JOIN dim_product p ON f.product_key = p.product_key
WHERE d.year = 2024
GROUP BY d.year, d.quarter, p.category
ORDER BY d.quarter, total_revenue DESC;

-- Customer segment analysis
SELECT
    c.segment,
    COUNT(DISTINCT f.customer_key) AS num_customers,
    SUM(f.revenue) AS total_revenue,
    AVG(f.revenue) AS avg_order_value
FROM fact_sales f
JOIN dim_customer c ON f.customer_key = c.customer_key
WHERE c.is_current = TRUE
GROUP BY c.segment
ORDER BY total_revenue DESC;

-- Store performance
SELECT
    s.store_name,
    s.region,
    SUM(f.revenue) AS total_revenue,
    SUM(f.quantity) AS total_units_sold
FROM fact_sales f
JOIN dim_store s ON f.store_key = s.store_key
JOIN dim_date d ON f.date_key = d.date_key
WHERE d.year = 2024 AND d.month = 1
GROUP BY s.store_name, s.region
ORDER BY total_revenue DESC;
```

</details>

---

### Problem 2: Handle Slowly Changing Customer Address

**Scenario:**
A customer's address changes from Seattle to Portland on January 16, 2024. The customer made purchases on:
- Jan 15 in Seattle
- Jan 20 in Portland

Implement SCD Type 2 and write a query to show revenue by customer city (point-in-time accurate).

<details>
<summary>Click for Solution</summary>

**Initial Dimension State (Jan 15):**
```sql
Dim_Customer:
| customer_key | customer_id | name  | city    | state | effective_date | expiration_date | is_current |
|--------------|-------------|-------|---------|-------|----------------|-----------------|------------|
| 1001         | C1001       | Alice | Seattle | WA    | 2020-01-01     | 9999-12-31      | TRUE       |
```

**Fact Table (Jan 15 Sale):**
```sql
Fact_Sales:
| sale_key | date_key | customer_key | product_key | revenue |
|----------|----------|--------------|-------------|---------|
| 1        | 20240115 | 1001         | 500         | 100.00  |
```

**Update Process (Jan 16 - Address Change):**
```sql
-- Step 1: Expire old record
UPDATE dim_customer
SET expiration_date = '2024-01-15',
    is_current = FALSE
WHERE customer_key = 1001;

-- Step 2: Insert new record with new customer_key
INSERT INTO dim_customer (
    customer_key, customer_id, name, city, state,
    effective_date, expiration_date, is_current
)
VALUES (
    1002, 'C1001', 'Alice', 'Portland', 'OR',
    '2024-01-16', '9999-12-31', TRUE
);
```

**After Update:**
```sql
Dim_Customer:
| customer_key | customer_id | name  | city     | state | effective_date | expiration_date | is_current |
|--------------|-------------|-------|----------|-------|----------------|-----------------|------------|
| 1001         | C1001       | Alice | Seattle  | WA    | 2020-01-01     | 2024-01-15      | FALSE      |
| 1002         | C1001       | Alice | Portland | OR    | 2024-01-16     | 9999-12-31      | TRUE       |
```

**New Sale (Jan 20) Uses New Key:**
```sql
Fact_Sales:
| sale_key | date_key | customer_key | product_key | revenue |
|----------|----------|--------------|-------------|---------|
| 1        | 20240115 | 1001         | 500         | 100.00  |
| 2        | 20240120 | 1002         | 501         | 150.00  |  -- New customer_key!
```

**Query: Revenue by City (Point-in-Time):**
```sql
SELECT
    c.city,
    SUM(f.revenue) AS total_revenue
FROM fact_sales f
JOIN dim_customer c ON f.customer_key = c.customer_key
GROUP BY c.city;

-- Result:
-- Seattle:  100.00 (sale on Jan 15 correctly attributed)
-- Portland: 150.00 (sale on Jan 20 correctly attributed)
```

**Query: Revenue by Customer (Current Name):**
```sql
SELECT
    c_current.customer_id,
    c_current.name,
    SUM(f.revenue) AS total_revenue
FROM fact_sales f
JOIN dim_customer c_historical ON f.customer_key = c_historical.customer_key
JOIN dim_customer c_current ON c_historical.customer_id = c_current.customer_id
WHERE c_current.is_current = TRUE
GROUP BY c_current.customer_id, c_current.name;

-- Result:
-- C1001 | Alice | 250.00 (total across both addresses)
```

</details>

---

### Problem 3: Prevent Double-Counting

**Scenario:**
You need to report:
1. Total number of orders
2. Total number of order items
3. Total revenue

Tables:
```sql
Orders:
| order_id | customer_id | order_date | order_amount |
|----------|-------------|------------|--------------|
| 1        | 100         | 2024-01-15 | 125          |
| 2        | 101         | 2024-01-16 | 50           |

OrderItems:
| order_item_id | order_id | product_id | quantity | item_amount |
|---------------|----------|------------|----------|-------------|
| 1             | 1        | 500        | 2        | 50          |
| 2             | 1        | 501        | 1        | 75          |
| 3             | 2        | 500        | 1        | 50          |
```

Write a query to get correct metrics without double-counting.

<details>
<summary>Click for Solution</summary>

**Wrong Query (Double-Counts):**
```sql
SELECT
    COUNT(o.order_id) AS num_orders,       -- WRONG: 3 (should be 2)
    COUNT(oi.order_item_id) AS num_items,  -- Correct: 3
    SUM(o.order_amount) AS total_revenue   -- WRONG: 300 (should be 175)
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id;
```

**Correct Query:**
```sql
SELECT
    COUNT(DISTINCT o.order_id) AS num_orders,  -- Correct: 2
    COUNT(oi.order_item_id) AS num_items,      -- Correct: 3
    SUM(DISTINCT o.order_amount) AS total_revenue  -- Correct: 175
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id;
```

**Better Approach (Separate Aggregations):**
```sql
WITH order_metrics AS (
    SELECT
        COUNT(*) AS num_orders,
        SUM(order_amount) AS total_revenue
    FROM orders
),
item_metrics AS (
    SELECT
        COUNT(*) AS num_items
    FROM order_items
)
SELECT
    om.num_orders,
    im.num_items,
    om.total_revenue
FROM order_metrics om, item_metrics im;
```

**Best Approach (Use Fact Table at Correct Grain):**
```sql
-- Design fact table at item grain
CREATE TABLE fact_order_items (
    item_key BIGINT PRIMARY KEY,
    order_id INT,
    product_key INT,
    quantity INT,
    item_amount DECIMAL(10,2),
    order_amount DECIMAL(10,2)  -- Denormalized for convenience
);

-- Query without double-counting
SELECT
    COUNT(DISTINCT order_id) AS num_orders,
    COUNT(*) AS num_items,
    SUM(DISTINCT order_amount) AS total_revenue
FROM fact_order_items;
```

</details>

---

## 12. Summary and Self-Assessment

### Key Takeaways

**1. Data Modeling Fundamentals**
- OLTP (normalized) vs OLAP (denormalized)
- Normalization reduces redundancy but requires JOINs
- Denormalization improves query performance

**2. Dimensional Modeling**
- Fact tables: Measurements (revenue, quantity)
- Dimension tables: Context (who, what, when, where)
- Star schema: Denormalized dimensions (preferred)
- Snowflake schema: Normalized dimensions (rare)

**3. Fact Tables**
- Additive: Can sum across all dimensions (revenue)
- Semi-additive: Can sum across some dimensions (balance)
- Non-additive: Cannot sum (unit price)
- Grain must be clearly defined

**4. Dimension Tables**
- Surrogate keys (system-generated) vs natural keys (business)
- Date dimension is critical
- Conformed dimensions shared across facts
- Role-playing dimensions used multiple times

**5. Slowly Changing Dimensions**
- Type 1: Overwrite (no history)
- Type 2: Add new row (complete history) - **Most common**
- Type 3: Add column (limited history)
- Type 4: Separate history table

**6. Preventing Double-Counting**
- Understand grain of fact tables
- Use COUNT DISTINCT for counts across JOINs
- Aggregate before JOIN when possible
- Never JOIN facts at different grains

**7. Design Patterns**
- Kimball (bottom-up): Build data marts first
- Inmon (top-down): Build enterprise warehouse first
- Data Vault: Hubs, Links, Satellites
- One Big Table: Denormalize everything (rare)

### Self-Assessment Checklist

**After completing this chapter, you should be able to:**

- [ ] Explain difference between OLTP and OLAP
- [ ] Design a star schema for a business scenario
- [ ] Choose between star and snowflake schema
- [ ] Identify fact vs dimension attributes
- [ ] Implement SCD Type 1, 2, and 3
- [ ] Prevent double-counting in JOINs
- [ ] Create a comprehensive date dimension
- [ ] Use surrogate keys correctly
- [ ] Understand additive vs semi-additive vs non-additive facts
- [ ] Design for query performance
- [ ] Handle slowly changing dimensions in queries
- [ ] Explain conformed dimensions

### Common Interview Questions

**1. "Design a data warehouse for Uber"**

<details>
<summary>Answer Framework</summary>

**Business Processes:** Rides, Payments, Driver Activity

**Fact Table: Fact_Rides**
```sql
Grain: One row per completed ride

fact_rides:
- ride_key (PK)
- date_key (FK)
- pickup_time_key (FK - role-playing date dim)
- dropoff_time_key (FK - role-playing date dim)
- rider_key (FK)
- driver_key (FK)
- pickup_location_key (FK)
- dropoff_location_key (FK)
- distance_miles (fact - additive)
- duration_minutes (fact - additive)
- base_fare (fact - additive)
- surge_multiplier (fact - non-additive)
- total_fare (fact - additive)
- driver_payout (fact - additive)
```

**Dimensions:**
- dim_date (with time-of-day attributes)
- dim_rider (SCD Type 2 for rating, status)
- dim_driver (SCD Type 2 for rating, status, vehicle)
- dim_location (city, neighborhood, lat/long)

**Key Design Decisions:**
- Role-playing date dimension for pickup/dropoff
- Track surge multiplier (non-additive, use AVG)
- SCD Type 2 for driver (track vehicle changes)
- Location dimension for geographic analysis

</details>

**2. "How do you handle a product price change?"**

<details>
<summary>Answer Framework</summary>

**Option 1: SCD Type 2 (Track History)**
```sql
dim_product:
| product_key | product_id | name   | price | effective_date | expiration_date | is_current |
|-------------|------------|--------|-------|----------------|-----------------|------------|
| 1           | P100       | Laptop | 999   | 2023-01-01     | 2024-01-14      | FALSE      |
| 2           | P100       | Laptop | 1099  | 2024-01-15     | 9999-12-31      | TRUE       |
```

Benefits:
- Point-in-time accuracy
- Historical analysis (how did price changes affect sales?)

**Option 2: Store Price in Fact**
```sql
fact_sales:
| sale_key | product_key | unit_price | quantity | revenue |

-- unit_price from transaction time
```

Benefits:
- Simpler dimension
- Exact transaction price always available

**Recommendation:** Use Option 2 (price in fact) for transaction systems. Price is part of the transaction event, not product attribute.

</details>

**3. "What's the difference between star and snowflake schema?"**

<details>
<summary>Answer</summary>

**Star Schema:**
- Denormalized dimensions
- Fewer JOINs (fact → dimension only)
- Faster queries
- More storage (redundancy)
- Preferred for most data warehouses

**Snowflake Schema:**
- Normalized dimensions
- More JOINs (fact → dimension → sub-dimension)
- Slower queries
- Less storage
- Used when dimensions are very large or frequently updated

**Example:**
Star: Fact → Product (with category, subcategory columns)
Snowflake: Fact → Product → Category → Subcategory

**When to use Snowflake:**
- Dimension is huge (millions of rows)
- Storage cost is critical
- Dimension updates are frequent

**Most Common:** Star schema (storage is cheap, query performance matters)

</details>

### Next Steps

**Practice:**
1. Design a star schema for a real business (Netflix, Airbnb, etc.)
2. Implement SCD Type 2 with SQL scripts
3. Create a date dimension for 10 years
4. Practice JOIN queries without double-counting

**DataDriven.io Problems:**
- Filter by "Data Modeling" tag
- Practice dimensional modeling scenarios
- Focus on:
  - Grain definition
  - Fact vs dimension identification
  - SCD implementation
  - Double-counting prevention

**Further Reading:**
- "The Data Warehouse Toolkit" by Ralph Kimball
- "Building the Data Warehouse" by Bill Inmon
- Kimball Group website (kimballgroup.com)

---

**Congratulations!** You now understand data modeling at a level sufficient for mid-level data engineering interviews. You can design star schemas, handle slowly changing dimensions, and prevent double-counting.

**Next Chapter:** Practice Problems & Solutions

---

**Chapter 7 Complete** | [Back to Main Guide](../README.md) | [Next: Chapter 8 - Practice Problems →](../08-Practice-Problems/Chapter-08-Practice-Problems.md)
