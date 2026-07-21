# Chapter 2: SQL Window Functions

**Duration:** 5-7 days | **Difficulty:** Intermediate
**Problems to Solve:** 25-35 medium problems
**Prerequisites:** Chapter 1 (SQL Fundamentals)

---

## 📚 Table of Contents

1. [Introduction to Window Functions](#introduction)
2. [Core Concepts and Theory](#core-concepts)
3. [Ranking Functions](#ranking-functions)
4. [Value Functions (LAG, LEAD, FIRST_VALUE, LAST_VALUE)](#value-functions)
5. [Aggregate Window Functions](#aggregate-window-functions)
6. [Frame Clauses (ROWS vs RANGE)](#frame-clauses)
7. [Distribution Functions (NTILE, PERCENT_RANK)](#distribution-functions)
8. [Advanced Patterns](#advanced-patterns)
9. [Performance Considerations](#performance)
10. [Practice Problems](#practice-problems)
11. [Solutions](#solutions)
12. [Summary](#summary)

---

## 1. Introduction to Window Functions {#introduction}

### What Are Window Functions?

Window functions perform calculations **across a set of rows related to the current row**, without collapsing rows like `GROUP BY` does.

**Key difference from GROUP BY:**
- **GROUP BY:** Collapses rows into groups → fewer rows in output
- **Window Functions:** Keep all rows → same number of rows in output

### Real-World Example

**Scenario:** E-commerce sales analysis

**With GROUP BY:**
```sql
SELECT
    product_id,
    SUM(sales) AS total_sales
FROM sales
GROUP BY product_id;
```

**Result:** One row per product (collapsed)
```
product_id | total_sales
-----------|------------
1          | 1000
2          | 1500
```

**With Window Function:**
```sql
SELECT
    sale_id,
    product_id,
    sales,
    SUM(sales) OVER (PARTITION BY product_id) AS total_product_sales
FROM sales;
```

**Result:** All rows kept, with running context
```
sale_id | product_id | sales | total_product_sales
--------|------------|-------|--------------------
101     | 1          | 200   | 1000
102     | 1          | 300   | 1000
103     | 1          | 500   | 1000
201     | 2          | 600   | 1500
202     | 2          | 900   | 1500
```

**Notice:** Each row still present, but can see total across product

### Why Window Functions Matter

**Common use cases in data engineering:**
1. **Ranking:** Top N per category
2. **Running totals:** Cumulative sales over time
3. **Moving averages:** 7-day rolling average
4. **Time comparisons:** Compare with previous day/week
5. **Percentiles:** 90th percentile latency
6. **Deduplication:** Keep only latest record per user

**60-70% of DataDriven mid-level SQL problems use window functions!**

---

## 2. Core Concepts and Theory {#core-concepts}

### Anatomy of a Window Function

```sql
function_name([arguments]) OVER (
    [PARTITION BY partition_expression]
    [ORDER BY sort_expression]
    [frame_clause]
)
```

### OVER Clause: The Window Definition

**Three components:**

#### 1. PARTITION BY (Optional)

Divides rows into groups (partitions). Function applies separately to each partition.

**Think of it as:** "GROUP BY for window functions"

```sql
SUM(sales) OVER (PARTITION BY product_id)
```

**Meaning:** Calculate sum separately for each product

**Visual:**
```
Partition 1 (product_id = 1):
row 1: sales = 200  →  sum = 1000
row 2: sales = 300  →  sum = 1000
row 3: sales = 500  →  sum = 1000

Partition 2 (product_id = 2):
row 4: sales = 600  →  sum = 1500
row 5: sales = 900  →  sum = 1500
```

**Without PARTITION BY:** Entire table is one partition

```sql
SUM(sales) OVER ()
-- Sum of ALL sales, shown on every row
```

---

#### 2. ORDER BY (Optional for some functions, required for others)

Defines the order within each partition.

**Required for:**
- Ranking functions (ROW_NUMBER, RANK, etc.)
- LAG/LEAD
- Running totals
- Frame clauses

**Not required for:**
- Aggregate over entire partition (no frame clause)

```sql
ROW_NUMBER() OVER (PARTITION BY product_id ORDER BY sale_date)
```

**Meaning:** Number rows 1, 2, 3... within each product, ordered by date

---

#### 3. Frame Clause (Optional)

Defines the **window frame**: which rows within the partition to include in the calculation.

**Examples:**
```sql
-- Current row only
ROWS BETWEEN CURRENT ROW AND CURRENT ROW

-- All rows from start to current
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

-- 7-day rolling window (current + 6 preceding)
ROWS BETWEEN 6 PRECEDING AND CURRENT ROW

-- Centered window (3 before, current, 3 after)
ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING
```

**Default frame (if ORDER BY present):**
```sql
RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

*We'll dive deep into frames in Section 6*

---

### Types of Window Functions

#### 1. Ranking Functions
- `ROW_NUMBER()` - Unique sequential integer
- `RANK()` - Rank with gaps on ties
- `DENSE_RANK()` - Rank without gaps
- `NTILE(n)` - Divide into n buckets

#### 2. Value Functions
- `LAG(col, offset)` - Value from previous row
- `LEAD(col, offset)` - Value from next row
- `FIRST_VALUE(col)` - First value in window
- `LAST_VALUE(col)` - Last value in window

#### 3. Aggregate Functions (as window functions)
- `SUM()`, `AVG()`, `COUNT()`, `MIN()`, `MAX()`
- When used with OVER clause, become window functions

#### 4. Distribution Functions
- `PERCENT_RANK()` - Relative rank (0 to 1)
- `CUME_DIST()` - Cumulative distribution
- `PERCENTILE_CONT(percentile)` - Continuous percentile
- `PERCENTILE_DISC(percentile)` - Discrete percentile

---

## 3. Ranking Functions {#ranking-functions}

### ROW_NUMBER()

**Returns:** Unique sequential integer for each row within partition

**Syntax:**
```sql
ROW_NUMBER() OVER (
    [PARTITION BY col]
    ORDER BY col [ASC|DESC]
)
```

**Example Data:**
```sql
CREATE TABLE sales (
    sale_id INT,
    product_id INT,
    sale_date DATE,
    amount DECIMAL(10,2)
);

INSERT INTO sales VALUES
(1, 1, '2024-01-01', 100),
(2, 1, '2024-01-02', 150),
(3, 1, '2024-01-03', 150),
(4, 2, '2024-01-01', 200),
(5, 2, '2024-01-02', 250);
```

**Query:**
```sql
SELECT
    sale_id,
    product_id,
    sale_date,
    amount,
    ROW_NUMBER() OVER (
        PARTITION BY product_id
        ORDER BY sale_date
    ) AS row_num
FROM sales;
```

**Result:**
```
sale_id | product_id | sale_date  | amount | row_num
--------|------------|------------|--------|--------
1       | 1          | 2024-01-01 | 100    | 1
2       | 1          | 2024-01-02 | 150    | 2
3       | 1          | 2024-01-03 | 150    | 3
4       | 2          | 2024-01-01 | 200    | 1
5       | 2          | 2024-01-02 | 250    | 2
```

**Key properties:**
- Always unique (even for ties)
- Deterministic ordering with ORDER BY
- Use for: Pagination, deduplication, selecting Nth row

**Common pattern: Latest record per group**
```sql
-- Get most recent sale for each product
SELECT *
FROM (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY product_id
            ORDER BY sale_date DESC
        ) AS rn
    FROM sales
) ranked
WHERE rn = 1;
```

---

### RANK()

**Returns:** Rank with gaps for ties

**Example:**
```sql
SELECT
    sale_id,
    product_id,
    amount,
    RANK() OVER (
        PARTITION BY product_id
        ORDER BY amount DESC
    ) AS rank
FROM sales;
```

**Result:**
```
sale_id | product_id | amount | rank
--------|------------|--------|-----
2       | 1          | 150    | 1
3       | 1          | 150    | 1  ← Tie!
1       | 1          | 100    | 3  ← Gap (no rank 2)
5       | 2          | 250    | 1
4       | 2          | 200    | 2
```

**Key difference from ROW_NUMBER:**
- Ties get same rank
- Next rank skips numbers (gap)

**Use when:** Need to show ties, ok with gaps

---

### DENSE_RANK()

**Returns:** Rank without gaps

**Example:**
```sql
SELECT
    sale_id,
    product_id,
    amount,
    DENSE_RANK() OVER (
        PARTITION BY product_id
        ORDER BY amount DESC
    ) AS dense_rank
FROM sales;
```

**Result:**
```
sale_id | product_id | amount | dense_rank
--------|------------|--------|------------
2       | 1          | 150    | 1
3       | 1          | 150    | 1  ← Tie!
1       | 1          | 100    | 2  ← No gap!
5       | 2          | 250    | 1
4       | 2          | 200    | 2
```

**Key difference from RANK:**
- No gaps in numbering
- Next rank is always previous + 1

**Use when:** Need consecutive rank numbers

---

### Comparing ROW_NUMBER, RANK, DENSE_RANK

**Sample data with ties:**
```
amount: 100, 100, 100, 90, 80
```

**Results:**
```
amount | row_number | rank | dense_rank
-------|------------|------|------------
100    | 1          | 1    | 1
100    | 2          | 1    | 1
100    | 3          | 1    | 1
90     | 4          | 4    | 2  ← Note the difference!
80     | 5          | 5    | 3
```

**Decision tree:**
- Need unique numbers? → `ROW_NUMBER()`
- Need ties with gaps? → `RANK()`
- Need consecutive ranks? → `DENSE_RANK()`

---

### Practical Example: "10 Lowest Uptime Services"

**Problem:** Find the 10 services with lowest uptime percentage

**Data:**
```sql
CREATE TABLE services (
    service_id INT,
    service_name VARCHAR(100),
    uptime_percentage DECIMAL(5,2)
);
```

**Solution 1: ORDER BY + LIMIT**
```sql
SELECT
    service_name,
    uptime_percentage
FROM services
ORDER BY uptime_percentage ASC
LIMIT 10;
```

**Problem with this:** If there are ties at position 10, we might miss some

**Solution 2: Window function (handles ties)**
```sql
WITH ranked_services AS (
    SELECT
        service_name,
        uptime_percentage,
        DENSE_RANK() OVER (ORDER BY uptime_percentage ASC) AS rank
    FROM services
)
SELECT
    service_name,
    uptime_percentage
FROM ranked_services
WHERE rank <= 10
ORDER BY rank, service_name;
```

**Why DENSE_RANK here?**
- Want all services with uptime in "bottom 10 distinct values"
- If 5 services have uptime=99.5% (10th lowest), include all 5

---

## 4. Value Functions (LAG, LEAD, FIRST_VALUE, LAST_VALUE) {#value-functions}

### LAG()

**Returns:** Value from a previous row

**Syntax:**
```sql
LAG(column, offset, default_value) OVER (
    [PARTITION BY col]
    ORDER BY col
)
```

**Parameters:**
- `column` - Column to get value from
- `offset` - How many rows back (default: 1)
- `default_value` - Return this if no previous row (default: NULL)

**Example: Day-over-day change**

```sql
SELECT
    sale_date,
    amount,
    LAG(amount, 1) OVER (ORDER BY sale_date) AS prev_day_amount,
    amount - LAG(amount, 1) OVER (ORDER BY sale_date) AS day_over_day_change
FROM daily_sales;
```

**Result:**
```
sale_date  | amount | prev_day_amount | day_over_day_change
-----------|--------|-----------------|--------------------
2024-01-01 | 1000   | NULL            | NULL
2024-01-02 | 1200   | 1000            | 200
2024-01-03 | 1100   | 1200            | -100
2024-01-04 | 1300   | 1100            | 200
```

**With default value:**
```sql
LAG(amount, 1, 0) OVER (ORDER BY sale_date)
-- Returns 0 instead of NULL for first row
```

**Multiple offsets:**
```sql
SELECT
    sale_date,
    amount,
    LAG(amount, 1) OVER (ORDER BY sale_date) AS yesterday,
    LAG(amount, 7) OVER (ORDER BY sale_date) AS last_week
FROM daily_sales;
```

---

### LEAD()

**Returns:** Value from a next row (opposite of LAG)

**Example: Forward-looking metrics**

```sql
SELECT
    sale_date,
    amount,
    LEAD(amount, 1) OVER (ORDER BY sale_date) AS next_day_amount,
    CASE
        WHEN LEAD(amount, 1) OVER (ORDER BY sale_date) > amount THEN 'Increasing'
        WHEN LEAD(amount, 1) OVER (ORDER BY sale_date) < amount THEN 'Decreasing'
        ELSE 'Stable'
    END AS trend
FROM daily_sales;
```

---

### Practical Example: Session Boundary Detection

**Problem:** "Between the Clicks" - Detect session boundaries

A new session starts if >30 minutes gap between events.

**Data:**
```sql
CREATE TABLE events (
    user_id INT,
    event_time TIMESTAMP
);
```

**Solution:**
```sql
WITH event_gaps AS (
    SELECT
        user_id,
        event_time,
        LAG(event_time) OVER (
            PARTITION BY user_id
            ORDER BY event_time
        ) AS prev_event_time,
        event_time - LAG(event_time) OVER (
            PARTITION BY user_id
            ORDER BY event_time
        ) AS time_since_last_event
    FROM events
),
session_starts AS (
    SELECT
        *,
        CASE
            WHEN time_since_last_event IS NULL THEN 1  -- First event
            WHEN time_since_last_event > INTERVAL '30 minutes' THEN 1
            ELSE 0
        END AS is_session_start
    FROM event_gaps
)
SELECT
    user_id,
    event_time,
    SUM(is_session_start) OVER (
        PARTITION BY user_id
        ORDER BY event_time
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS session_id
FROM session_starts;
```

**Explanation:**
1. `LAG` gets previous event time
2. Calculate gap duration
3. Mark session starts (>30 min gap or first event)
4. Running sum of session starts = session_id

---

### FIRST_VALUE() and LAST_VALUE()

**FIRST_VALUE:** First value in the window

**LAST_VALUE:** Last value in the window

**Example:**
```sql
SELECT
    sale_date,
    amount,
    FIRST_VALUE(amount) OVER (
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS first_sale,
    LAST_VALUE(amount) OVER (
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_sale
FROM sales;
```

**⚠️ CRITICAL:** Always use full frame with LAST_VALUE!

**Why?**

**Wrong (common mistake):**
```sql
LAST_VALUE(amount) OVER (ORDER BY sale_date)
-- Default frame: UNBOUNDED PRECEDING to CURRENT ROW
-- "Last value" is always current row!
```

**Correct:**
```sql
LAST_VALUE(amount) OVER (
    ORDER BY sale_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
)
-- Frame includes all rows, so truly gets last
```

**Practical use: Baseline comparison**
```sql
SELECT
    month,
    revenue,
    FIRST_VALUE(revenue) OVER (ORDER BY month) AS baseline_month,
    revenue / FIRST_VALUE(revenue) OVER (ORDER BY month) AS growth_multiple
FROM monthly_revenue;
```

---

## 5. Aggregate Window Functions {#aggregate-window-functions}

### Aggregates as Window Functions

Any aggregate function can be used as a window function with `OVER` clause:
- `SUM()`
- `AVG()`
- `COUNT()`
- `MIN()`
- `MAX()`
- `STDDEV()`
- `VARIANCE()`

### Running Totals (Cumulative Sum)

**Pattern:** "Running total of sales over time"

```sql
SELECT
    sale_date,
    amount,
    SUM(amount) OVER (
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM sales;
```

**Shorthand (same result):**
```sql
SUM(amount) OVER (ORDER BY sale_date)
-- Default frame is UNBOUNDED PRECEDING to CURRENT ROW
```

**Result:**
```
sale_date  | amount | running_total
-----------|--------|---------------
2024-01-01 | 100    | 100
2024-01-02 | 150    | 250
2024-01-03 | 200    | 450
2024-01-04 | 120    | 570
```

**Visual:**
```
Row 1: SUM(100) = 100
Row 2: SUM(100, 150) = 250
Row 3: SUM(100, 150, 200) = 450
Row 4: SUM(100, 150, 200, 120) = 570
```

**With PARTITION BY (running total per group):**
```sql
SUM(amount) OVER (
    PARTITION BY product_id
    ORDER BY sale_date
)
-- Separate running total for each product
```

---

### Moving Averages (Rolling Windows)

**Pattern:** "7-day moving average" (DataDriven problem!)

```sql
SELECT
    check_date,
    value,
    AVG(value) OVER (
        ORDER BY check_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS rolling_avg_7day
FROM metrics;
```

**Explanation:**
- `6 PRECEDING + CURRENT ROW = 7 rows total`
- Window slides forward with each row
- First 6 rows have fewer than 7 days (beginning of data)

**Result:**
```
check_date | value | rolling_avg_7day
-----------|-------|------------------
2024-01-01 | 10    | 10.00  (only 1 row)
2024-01-02 | 12    | 11.00  (2 rows)
2024-01-03 | 11    | 11.00  (3 rows)
2024-01-04 | 13    | 11.50  (4 rows)
2024-01-05 | 14    | 12.00  (5 rows)
2024-01-06 | 15    | 12.50  (6 rows)
2024-01-07 | 16    | 13.00  (7 rows) ← Full window starts
2024-01-08 | 14    | 13.57  (7 rows: 12,11,13,14,15,16,14)
```

**Common windows:**
- 7-day: `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW`
- 30-day: `ROWS BETWEEN 29 PRECEDING AND CURRENT ROW`
- Centered 7-day: `ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING`

---

### COUNT with Windows

**Example: Count of orders in last 30 days**

```sql
SELECT
    order_date,
    COUNT(*) OVER (
        ORDER BY order_date
        RANGE BETWEEN INTERVAL '30 days' PRECEDING AND CURRENT ROW
    ) AS orders_last_30_days
FROM orders;
```

**Use RANGE for date-based windows (more on this in Section 6)**

---

## 6. Frame Clauses (ROWS vs RANGE) {#frame-clauses}

### What is a Frame?

The **frame** defines which rows within the partition are included in the calculation.

**Default frames:**
- If `ORDER BY` present: `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`
- If no `ORDER BY`: All rows in partition

### ROWS vs RANGE

#### ROWS - Physical offset (row count)

Counts rows physically.

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
-- Includes: 2 rows before + current row = 3 rows
```

**Example:**
```
Rows:     amount:
1         10   ← Window for row 3
2         20   ← Window for row 3
3         30   ← Current row (row 3)
4         40
5         50

Window for row 3: (10, 20, 30)
SUM = 60
```

#### RANGE - Logical offset (value-based)

Includes all rows with values within a range.

```sql
RANGE BETWEEN 10 PRECEDING AND CURRENT ROW
-- Includes all rows where value is within (current_value - 10) to current_value
```

**Example:**
```
Rows:     amount:
1         10
2         12   ← Included (20-10=10, and 12 >= 10)
3         15   ← Included
4         20   ← Current row
5         25

Window for row 4 (amount=20):
Includes rows where amount BETWEEN (20-10) AND 20
= rows with amount 10, 12, 15, 20
SUM = 57
```

**Key difference:**
- `ROWS` - Fixed number of rows
- `RANGE` - Variable number of rows based on values

### Frame Boundaries

**Start boundary:**
- `UNBOUNDED PRECEDING` - From start of partition
- `n PRECEDING` - n rows/units before current
- `CURRENT ROW` - Current row only

**End boundary:**
- `CURRENT ROW` - Current row only
- `n FOLLOWING` - n rows/units after current
- `UNBOUNDED FOLLOWING` - To end of partition

**Examples:**

```sql
-- All rows from start to current
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

-- Current row only
ROWS BETWEEN CURRENT ROW AND CURRENT ROW

-- 3 before, current, 3 after (7 total)
ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING

-- Current to end of partition
ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING

-- Everything
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
```

### When to Use ROWS vs RANGE

**Use ROWS when:**
- Need exact number of rows (e.g., 7-day rolling average)
- Working with row numbers
- Performance (ROWS is faster)

**Use RANGE when:**
- Date/time-based windows (e.g., last 30 days)
- Value-based windows
- Need to include ties

**Example: 30-day window with RANGE**

```sql
SELECT
    order_date,
    COUNT(*) OVER (
        ORDER BY order_date
        RANGE BETWEEN INTERVAL '30 days' PRECEDING AND CURRENT ROW
    ) AS orders_last_30_days
FROM orders;
```

**Why RANGE?**
- Includes all orders within 30 days
- If multiple orders on same date, includes all

**With ROWS (wrong):**
```sql
-- This would count 30 rows, not 30 days!
ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
```

---

## 7. Distribution Functions (NTILE, PERCENT_RANK) {#distribution-functions}

### NTILE(n)

**Divides rows into n approximately equal buckets**

**Example: Quartiles (4 buckets)**

```sql
SELECT
    customer_id,
    total_spend,
    NTILE(4) OVER (ORDER BY total_spend DESC) AS quartile
FROM customer_totals;
```

**Result:**
```
customer_id | total_spend | quartile
------------|-------------|----------
1           | 10000       | 1  ← Top 25%
2           | 9000        | 1
3           | 8000        | 1
4           | 7000        | 2  ← 25-50%
5           | 6000        | 2
6           | 5000        | 2
7           | 4000        | 3  ← 50-75%
8           | 3000        | 3
9           | 2000        | 4  ← Bottom 25%
10          | 1000        | 4
```

**Use cases:**
- Customer segmentation (VIP, Premium, Standard, Basic)
- Percentile buckets
- Balanced distribution

**Common values:**
- `NTILE(4)` - Quartiles
- `NTILE(10)` - Deciles
- `NTILE(100)` - Percentiles

---

### PERCENT_RANK()

**Returns:** Relative rank (0 to 1)

**Formula:**
```
(rank - 1) / (total_rows - 1)
```

**Example:**
```sql
SELECT
    student_id,
    score,
    PERCENT_RANK() OVER (ORDER BY score DESC) AS percentile_rank
FROM test_scores;
```

**Result:**
```
student_id | score | percentile_rank
-----------|-------|----------------
1          | 100   | 0.00  (best)
2          | 95    | 0.11
3          | 90    | 0.22
4          | 85    | 0.33
...
10         | 50    | 1.00  (worst)
```

**Interpretation:**
- 0.00 = Top performer
- 0.50 = Median
- 1.00 = Bottom performer

---

### CUME_DIST()

**Returns:** Cumulative distribution (percentage of values ≤ current value)

**Example:**
```sql
SELECT
    score,
    CUME_DIST() OVER (ORDER BY score) AS cumulative_dist
FROM test_scores;
```

**Result:**
```
score | cumulative_dist
------|----------------
50    | 0.10  (10% scored ≤ 50)
60    | 0.20
70    | 0.40
80    | 0.60
90    | 0.80
100   | 1.00  (100% scored ≤ 100)
```

---

### PERCENTILE_CONT() and PERCENTILE_DISC()

**Calculate specific percentiles**

**PERCENTILE_CONT:** Continuous (interpolates between values)
**PERCENTILE_DISC:** Discrete (returns actual value from dataset)

**Example: 90th percentile**

```sql
-- Continuous
SELECT
    PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY latency) AS p90_latency_cont
FROM api_calls;

-- Discrete
SELECT
    PERCENTILE_DISC(0.9) WITHIN GROUP (ORDER BY latency) AS p90_latency_disc
FROM api_calls;
```

**Difference:**
```
Values: 10, 20, 30, 40, 50, 60, 70, 80, 90, 100

PERCENTILE_CONT(0.9):
- 90th percentile falls between 90 and 100
- Interpolates: 0.9 * (100-90) + 90 = 99

PERCENTILE_DISC(0.9):
- Returns actual value closest to 90th percentile
- Returns: 90
```

**DataDriven Problem:** "90th Pctl Model Accuracy Gap"

```sql
WITH model_percentiles AS (
    SELECT
        model_name,
        PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY accuracy) AS p90_accuracy,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY accuracy) AS median_accuracy
    FROM model_scores
    GROUP BY model_name
)
SELECT
    model_name,
    p90_accuracy,
    median_accuracy,
    p90_accuracy - median_accuracy AS gap
FROM model_percentiles
ORDER BY gap DESC;
```

---

## 8. Advanced Patterns {#advanced-patterns}

### Pattern 1: Top N per Group

**Problem:** "Top 3 products by sales in each category"

```sql
WITH ranked_products AS (
    SELECT
        category_id,
        product_id,
        product_name,
        total_sales,
        ROW_NUMBER() OVER (
            PARTITION BY category_id
            ORDER BY total_sales DESC
        ) AS rank
    FROM product_sales
)
SELECT
    category_id,
    product_name,
    total_sales,
    rank
FROM ranked_products
WHERE rank <= 3
ORDER BY category_id, rank;
```

---

### Pattern 2: Deduplication (Keep Latest)

**Problem:** Keep only the most recent record per user

```sql
WITH ranked_records AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY updated_at DESC
        ) AS rn
    FROM user_records
)
SELECT *
FROM ranked_records
WHERE rn = 1;
```

**Why ROW_NUMBER, not RANK?**
- Need exactly one record per user
- ROW_NUMBER guarantees uniqueness (even with ties)

---

### Pattern 3: Gap and Island Detection

**Problem:** Find consecutive days with data

```sql
WITH numbered_days AS (
    SELECT
        event_date,
        ROW_NUMBER() OVER (ORDER BY event_date) AS rn,
        event_date - (ROW_NUMBER() OVER (ORDER BY event_date) * INTERVAL '1 day') AS island_id
    FROM (SELECT DISTINCT event_date FROM events) dates
)
SELECT
    island_id,
    MIN(event_date) AS streak_start,
    MAX(event_date) AS streak_end,
    COUNT(*) AS consecutive_days
FROM numbered_days
GROUP BY island_id
ORDER BY streak_start;
```

**How it works:**
```
date        | rn | date - rn*1day | island_id
------------|----|-----------------|-----------
2024-01-01  | 1  | 2023-12-31     | A  ← Same island
2024-01-02  | 2  | 2023-12-31     | A
2024-01-03  | 3  | 2023-12-31     | A
2024-01-05  | 4  | 2024-01-01     | B  ← Gap! New island
2024-01-06  | 5  | 2024-01-01     | B
```

---

### Pattern 4: Conditional Running Total

**Problem:** Reset running total on condition

```sql
WITH transactions_with_flag AS (
    SELECT
        transaction_id,
        amount,
        CASE WHEN type = 'reset' THEN 1 ELSE 0 END AS is_reset
    FROM transactions
),
reset_groups AS (
    SELECT
        *,
        SUM(is_reset) OVER (ORDER BY transaction_id) AS group_id
    FROM transactions_with_flag
)
SELECT
    transaction_id,
    amount,
    SUM(amount) OVER (PARTITION BY group_id ORDER BY transaction_id) AS running_total
FROM reset_groups;
```

---

## 9. Performance Considerations {#performance}

### Optimization Tips

#### 1. Minimize Window Function Calls

**Bad (computes window 3 times):**
```sql
SELECT
    sale_date,
    amount,
    AVG(amount) OVER (ORDER BY sale_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW),
    MIN(amount) OVER (ORDER BY sale_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW),
    MAX(amount) OVER (ORDER BY sale_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)
FROM sales;
```

**Better (name the window):**
```sql
SELECT
    sale_date,
    amount,
    AVG(amount) OVER w,
    MIN(amount) OVER w,
    MAX(amount) OVER w
FROM sales
WINDOW w AS (ORDER BY sale_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW);
```

#### 2. Use Appropriate Frame

**Slower:**
```sql
SUM(amount) OVER (
    ORDER BY date
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
)
-- Includes all rows every time
```

**Faster:**
```sql
SUM(amount) OVER ()
-- No ORDER BY, computed once
```

#### 3. Filter Before Window Functions

**Bad:**
```sql
SELECT *
FROM (
    SELECT
        *,
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) AS rn
    FROM products
) ranked
WHERE price > 100 AND rn <= 3;
```

**Better:**
```sql
SELECT *
FROM (
    SELECT
        *,
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) AS rn
    FROM products
    WHERE price > 100  -- Filter early!
) ranked
WHERE rn <= 3;
```

#### 4. Index Partition and Order Columns

```sql
CREATE INDEX idx_sales_product_date ON sales(product_id, sale_date);

-- Speeds up:
SELECT
    *,
    ROW_NUMBER() OVER (PARTITION BY product_id ORDER BY sale_date)
FROM sales;
```

---

## 10. Practice Problems {#practice-problems}

Now practice on DataDriven.io! Here are recommended problems:

### Easy-Medium (Start Here)

1. **"7-Check Rolling Average"** - Moving average pattern
2. **"7-Day Token Retention"** - Retention calculation with LAG
3. **"Above Average"** - Compare to average (use window function!)

### Medium

4. **"10 Lowest Uptime Services"** - Ranking
5. **"Proof of Presence"** - Temporal comparison
6. **"The Long Tail"** - Percentile calculation
7. **"90th Pctl Model Accuracy Gap"** - PERCENTILE_CONT

### Challenging

8. Find products that had consecutive days of increased sales
9. Calculate month-over-month growth rate
10. Identify users in top 10% of engagement

---

## 11. Solutions {#solutions}

### Solution: "7-Check Rolling Average"

**Problem:** Calculate 7-day rolling average of health check values

```sql
SELECT
    check_date,
    value,
    AVG(value) OVER (
        ORDER BY check_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS rolling_avg_7day,
    COUNT(*) OVER (
        ORDER BY check_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS window_size
FROM health_checks
ORDER BY check_date;
```

**Explanation:**
- `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` = 7 rows (6 + current)
- For first 6 rows, window is incomplete (< 7 rows)
- `COUNT(*)` shows actual window size

---

### Solution: "10 Lowest Uptime Services"

```sql
WITH ranked_services AS (
    SELECT
        service_id,
        service_name,
        uptime_percentage,
        DENSE_RANK() OVER (ORDER BY uptime_percentage ASC) AS uptime_rank
    FROM services
)
SELECT
    service_name,
    uptime_percentage,
    uptime_rank
FROM ranked_services
WHERE uptime_rank <= 10
ORDER BY uptime_rank, service_name;
```

**Why DENSE_RANK?**
- Want 10 distinct uptime values
- Include all services with those uptimes (handles ties)

---

### Solution: "90th Pctl Model Accuracy Gap"

```sql
WITH model_stats AS (
    SELECT
        model_name,
        PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY accuracy) AS p90,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY accuracy) AS p50
    FROM model_predictions
    GROUP BY model_name
)
SELECT
    model_name,
    ROUND(p90, 4) AS p90_accuracy,
    ROUND(p50, 4) AS median_accuracy,
    ROUND(p90 - p50, 4) AS gap
FROM model_stats
ORDER BY gap DESC;
```

---

## 12. Summary {#summary}

### Key Concepts Mastered

✅ **Window function anatomy:** OVER, PARTITION BY, ORDER BY, frame clause
✅ **Ranking:** ROW_NUMBER, RANK, DENSE_RANK
✅ **Value functions:** LAG, LEAD, FIRST_VALUE, LAST_VALUE
✅ **Aggregate windows:** Running totals, moving averages
✅ **Frame clauses:** ROWS vs RANGE
✅ **Distribution:** NTILE, PERCENT_RANK, PERCENTILE_CONT
✅ **Common patterns:** Top N per group, deduplication, session detection

### Self-Assessment

Can you:
- [ ] Explain the difference between GROUP BY and window functions?
- [ ] Choose between ROW_NUMBER, RANK, and DENSE_RANK?
- [ ] Write a running total query?
- [ ] Calculate a 7-day moving average?
- [ ] Use LAG to find day-over-day change?
- [ ] Explain ROWS vs RANGE?
- [ ] Find top 3 per category?
- [ ] Calculate 90th percentile?

### Common Mistakes

❌ Using LAST_VALUE without full frame
❌ Confusing ROWS and RANGE for date windows
❌ Forgetting ORDER BY for ranking functions
❌ Using wrong ranking function for use case
❌ Not handling NULL with LAG/LEAD default

### Next Steps

**You're ready for Chapter 3: Advanced SQL!**

Topics include:
- CTEs (WITH clauses)
- Recursive queries
- Complex joins
- Query optimization

**Practice goals:**
- [ ] 30-40 medium SQL problems with window functions
- [ ] 80%+ success rate
- [ ] Average time: 25-30 min per problem

---

**Congratulations! Window functions are the most powerful SQL feature for analytics!** 🎉

**Next:** `03-SQL-Advanced-Queries/Chapter-03-Advanced-SQL.md`

---
