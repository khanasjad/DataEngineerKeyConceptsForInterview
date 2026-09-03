# SQL WINDOW FUNCTIONS - COMPLETE MEMORIZATION CARD
## Every Function You Need to Know for Interviews

> **Print this. Memorize this. You'll be ready for ANYTHING.**

---

## 📋 TABLE OF CONTENTS
1. [The Universal Template](#universal-template)
2. [All 4 Ranking Functions](#ranking-functions)
3. [All 5 Aggregate Functions as Windows](#aggregate-functions)
4. [All 5 Value/Offset Functions](#value-functions)
5. [PARTITION BY Explained](#partition-by)
6. [ORDER BY Explained](#order-by)
7. [All Window Frame Specifications](#window-frames)
8. [Every Frame Boundary Option](#frame-boundaries)
9. [Complete Syntax Breakdown](#syntax-breakdown)
10. [All Functions Side-by-Side](#all-functions-table)
11. [Interview Question Type → Function Mapping](#question-mapping)
12. [Common Combinations](#common-combinations)

---

## 🎯 THE UNIVERSAL TEMPLATE {#universal-template}

**EVERY window function follows this pattern:**

```sql
FUNCTION_NAME(arguments) OVER (
    [PARTITION BY column1, column2, ...]
    [ORDER BY column3 [ASC|DESC], column4, ...]
    [ROWS|RANGE BETWEEN frame_start AND frame_end]
)
```

### Components:
- **FUNCTION_NAME** - One of 14 functions (see below)
- **OVER** - Required keyword (marks it as a window function)
- **PARTITION BY** - Optional. Divides rows into groups
- **ORDER BY** - Optional for some, required for others
- **ROWS/RANGE BETWEEN** - Optional. Defines window frame

---

## 🏆 ALL 4 RANKING FUNCTIONS {#ranking-functions}

### 1. ROW_NUMBER()

**What**: Assigns unique sequential integer to each row

**Syntax**:
```sql
ROW_NUMBER() OVER ([PARTITION BY col] ORDER BY col)
```

**Requires ORDER BY**: ✅ Yes

**Example**:
```sql
SELECT
    name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num
FROM employees;

-- Result:
-- Alice, 100000, 1
-- Bob, 90000, 2
-- Charlie, 90000, 3    ← Even though tied, gets unique number
-- David, 80000, 4
```

**With PARTITION BY**:
```sql
SELECT
    name,
    department,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;

-- Result:
-- Alice, Sales, 100000, 1       ← Rank 1 in Sales
-- Bob, Sales, 90000, 2           ← Rank 2 in Sales
-- Charlie, IT, 95000, 1          ← Rank 1 in IT (restarts!)
-- David, IT, 85000, 2            ← Rank 2 in IT
```

**Use Cases**:
- Top N per group (most common!)
- Pagination
- Assigning unique IDs
- Breaking ties arbitrarily

**Handles Ties**: No (assigns different numbers even for same values)

---

### 2. RANK()

**What**: Assigns rank with gaps for ties

**Syntax**:
```sql
RANK() OVER ([PARTITION BY col] ORDER BY col)
```

**Requires ORDER BY**: ✅ Yes

**Example**:
```sql
SELECT
    name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- Result:
-- Alice, 100000, 1
-- Bob, 90000, 2
-- Charlie, 90000, 2     ← Same rank as Bob
-- David, 80000, 4       ← SKIPS 3!
-- Eve, 70000, 5
```

**Use Cases**:
- Competition ranking (Olympics style)
- When you want to show gaps for tied values

**Handles Ties**: Yes (same value = same rank, then skip numbers)

**Formula**: If 2 people tie for rank N, next rank is N+2

---

### 3. DENSE_RANK()

**What**: Assigns rank without gaps for ties

**Syntax**:
```sql
DENSE_RANK() OVER ([PARTITION BY col] ORDER BY col)
```

**Requires ORDER BY**: ✅ Yes

**Example**:
```sql
SELECT
    name,
    salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;

-- Result:
-- Alice, 100000, 1
-- Bob, 90000, 2
-- Charlie, 90000, 2     ← Same rank as Bob
-- David, 80000, 3       ← NO GAP! Next consecutive number
-- Eve, 70000, 4
```

**Use Cases**:
- **Find Nth highest/lowest value** (MOST COMMON!)
- Leaderboards with continuous ranks
- When you don't want gaps

**Handles Ties**: Yes (same value = same rank, no gaps)

**Formula**: If 2 people tie for rank N, next rank is N+1

---

### 4. NTILE(n)

**What**: Divides rows into n approximately equal buckets

**Syntax**:
```sql
NTILE(number_of_buckets) OVER ([PARTITION BY col] ORDER BY col)
```

**Requires ORDER BY**: ✅ Yes

**Example**:
```sql
SELECT
    name,
    salary,
    NTILE(4) OVER (ORDER BY salary DESC) AS quartile
FROM employees;

-- Result (for 8 employees divided into 4 groups):
-- Alice, 100000, 1      ← Top quartile
-- Bob, 95000, 1         ← Top quartile
-- Charlie, 90000, 2     ← 2nd quartile
-- David, 85000, 2       ← 2nd quartile
-- Eve, 80000, 3         ← 3rd quartile
-- Frank, 75000, 3       ← 3rd quartile
-- Grace, 70000, 4       ← Bottom quartile
-- Henry, 65000, 4       ← Bottom quartile
```

**With PARTITION BY**:
```sql
-- Quartiles within each department
NTILE(4) OVER (PARTITION BY department ORDER BY salary DESC)
```

**Use Cases**:
- Quartiles (NTILE(4))
- Percentiles (NTILE(100))
- Top 25% of customers (WHERE NTILE(4) ... = 1)
- Divide data into equal groups

**Common Values**:
- NTILE(2) - Median split (top/bottom half)
- NTILE(4) - Quartiles
- NTILE(5) - Quintiles
- NTILE(10) - Deciles
- NTILE(100) - Percentiles

---

## 📊 ALL 5 AGGREGATE FUNCTIONS AS WINDOWS {#aggregate-functions}

**Key Difference from Regular Aggregates**:
- Regular: `SELECT SUM(salary) FROM employees` → 1 row
- Window: `SELECT salary, SUM(salary) OVER () FROM employees` → All rows

---

### 1. SUM() OVER

**What**: Calculates sum over window

**Syntax**:
```sql
SUM(column) OVER ([PARTITION BY col] [ORDER BY col] [frame_clause])
```

**Requires ORDER BY**: ❌ No (but changes behavior)

**Without ORDER BY** (sum of entire partition):
```sql
SELECT
    name,
    department,
    salary,
    SUM(salary) OVER (PARTITION BY department) AS dept_total_salary
FROM employees;

-- Result:
-- Alice, Sales, 100000, 300000   ← Total of all Sales salaries
-- Bob, Sales, 90000, 300000       ← Same total for all Sales rows
-- Charlie, Sales, 110000, 300000  ← Same total
```

**With ORDER BY** (running total):
```sql
SELECT
    order_date,
    revenue,
    SUM(revenue) OVER (ORDER BY order_date) AS running_total
FROM daily_sales;

-- Result:
-- 2024-01-01, 100, 100          ← 100
-- 2024-01-02, 150, 250          ← 100 + 150
-- 2024-01-03, 200, 450          ← 100 + 150 + 200
-- 2024-01-04, 120, 570          ← 100 + 150 + 200 + 120
```

**With PARTITION BY and ORDER BY** (running total per group):
```sql
SELECT
    department,
    employee_name,
    salary,
    SUM(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
    ) AS running_dept_total
FROM employees;

-- Result:
-- Sales, Alice, 100000, 100000
-- Sales, Bob, 90000, 190000       ← Cumulative within Sales
-- IT, Charlie, 80000, 80000       ← RESTARTS for IT
-- IT, David, 60000, 140000        ← Cumulative within IT
```

**Use Cases**:
- Running total / cumulative sum
- Total per group (without ORDER BY)
- Year-to-date totals
- Remaining budget (with ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING)

---

### 2. AVG() OVER

**What**: Calculates average over window

**Syntax**:
```sql
AVG(column) OVER ([PARTITION BY col] [ORDER BY col] [frame_clause])
```

**Requires ORDER BY**: ❌ No (but changes behavior)

**Without ORDER BY** (average of entire partition):
```sql
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg_salary
FROM employees;

-- Result:
-- Alice, Sales, 100000, 95000    ← Avg of all Sales salaries
-- Bob, Sales, 90000, 95000        ← Same for all Sales rows
-- Charlie, IT, 80000, 70000       ← Avg of all IT salaries
-- David, IT, 60000, 70000         ← Same for all IT rows
```

**With ORDER BY** (cumulative average):
```sql
SELECT
    order_date,
    revenue,
    AVG(revenue) OVER (ORDER BY order_date) AS cumulative_avg
FROM daily_sales;

-- Result:
-- 2024-01-01, 100, 100.00        ← 100 / 1
-- 2024-01-02, 150, 125.00        ← (100 + 150) / 2
-- 2024-01-03, 200, 150.00        ← (100 + 150 + 200) / 3
-- 2024-01-04, 120, 142.50        ← (100 + 150 + 200 + 120) / 4
```

**With Frame (Moving Average)**:
```sql
SELECT
    order_date,
    revenue,
    AVG(revenue) OVER (
        ORDER BY order_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3day
FROM daily_sales;

-- Result:
-- 2024-01-01, 100, 100.00        ← Only 1 row
-- 2024-01-02, 150, 125.00        ← Avg of 2 rows
-- 2024-01-03, 200, 150.00        ← Avg of 3 rows (100, 150, 200)
-- 2024-01-04, 120, 156.67        ← Avg of 3 rows (150, 200, 120) ← Window slides!
-- 2024-01-05, 180, 166.67        ← Avg of 3 rows (200, 120, 180)
```

**Use Cases**:
- Department average salary
- Moving average (stock prices, weather)
- Compare individual to group average
- Smoothing data

---

### 3. COUNT() OVER

**What**: Counts rows in window

**Syntax**:
```sql
COUNT(*) OVER ([PARTITION BY col] [ORDER BY col] [frame_clause])
COUNT(column) OVER ([PARTITION BY col] [ORDER BY col] [frame_clause])
```

**Requires ORDER BY**: ❌ No

**Without ORDER BY** (count entire partition):
```sql
SELECT
    name,
    department,
    COUNT(*) OVER (PARTITION BY department) AS dept_employee_count
FROM employees;

-- Result:
-- Alice, Sales, 3        ← 3 employees in Sales
-- Bob, Sales, 3          ← Same for all Sales rows
-- Charlie, Sales, 3
-- David, IT, 2           ← 2 employees in IT
-- Eve, IT, 2             ← Same for all IT rows
```

**With ORDER BY** (cumulative count):
```sql
SELECT
    order_date,
    customer_id,
    COUNT(*) OVER (ORDER BY order_date) AS cumulative_order_count
FROM orders;

-- Result:
-- 2024-01-01, 123, 1
-- 2024-01-02, 456, 2
-- 2024-01-03, 789, 3
-- 2024-01-04, 234, 4
```

**COUNT(*) vs COUNT(column)**:
```sql
-- COUNT(*) counts all rows (including NULLs)
COUNT(*) OVER ()

-- COUNT(column) counts non-NULL values only
COUNT(email) OVER ()
```

**Use Cases**:
- Number of employees per department
- Cumulative count of events
- Running count of active users
- Row numbers within groups

---

### 4. MIN() OVER

**What**: Returns minimum value in window

**Syntax**:
```sql
MIN(column) OVER ([PARTITION BY col] [ORDER BY col] [frame_clause])
```

**Requires ORDER BY**: ❌ No

**Without ORDER BY** (min of entire partition):
```sql
SELECT
    name,
    department,
    salary,
    MIN(salary) OVER (PARTITION BY department) AS dept_min_salary
FROM employees;

-- Result:
-- Alice, Sales, 100000, 70000    ← Lowest salary in Sales
-- Bob, Sales, 90000, 70000        ← Same for all Sales rows
-- Charlie, Sales, 70000, 70000
-- David, IT, 80000, 60000         ← Lowest salary in IT
-- Eve, IT, 60000, 60000
```

**With ORDER BY** (minimum so far):
```sql
SELECT
    order_date,
    price,
    MIN(price) OVER (ORDER BY order_date) AS lowest_price_so_far
FROM stock_prices;

-- Result:
-- 2024-01-01, 100, 100
-- 2024-01-02, 95, 95      ← New minimum
-- 2024-01-03, 98, 95      ← Still 95
-- 2024-01-04, 90, 90      ← New minimum
```

**Use Cases**:
- Minimum/maximum salary in department
- Lowest price so far
- Benchmark values
- Range calculations (MAX - MIN)

---

### 5. MAX() OVER

**What**: Returns maximum value in window

**Syntax**:
```sql
MAX(column) OVER ([PARTITION BY col] [ORDER BY col] [frame_clause])
```

**Requires ORDER BY**: ❌ No

**Without ORDER BY** (max of entire partition):
```sql
SELECT
    name,
    department,
    salary,
    MAX(salary) OVER (PARTITION BY department) AS dept_max_salary
FROM employees;

-- Result:
-- Alice, Sales, 100000, 100000   ← Highest salary in Sales
-- Bob, Sales, 90000, 100000       ← Same for all Sales rows
-- Charlie, Sales, 70000, 100000
-- David, IT, 80000, 80000         ← Highest salary in IT
-- Eve, IT, 60000, 80000
```

**With ORDER BY** (maximum so far):
```sql
SELECT
    order_date,
    price,
    MAX(price) OVER (ORDER BY order_date) AS highest_price_so_far
FROM stock_prices;

-- Result:
-- 2024-01-01, 100, 100
-- 2024-01-02, 95, 100     ← Still 100
-- 2024-01-03, 105, 105    ← New maximum
-- 2024-01-04, 102, 105    ← Still 105
```

**Salary Range Example**:
```sql
SELECT
    name,
    department,
    salary,
    MIN(salary) OVER (PARTITION BY department) AS dept_min,
    MAX(salary) OVER (PARTITION BY department) AS dept_max,
    MAX(salary) OVER (PARTITION BY department) -
    MIN(salary) OVER (PARTITION BY department) AS dept_salary_range
FROM employees;
```

**Use Cases**:
- Highest/lowest values in group
- All-time high (running maximum)
- Range calculations
- Outlier detection

---

## 🔄 ALL 5 VALUE/OFFSET FUNCTIONS {#value-functions}

These functions access values from other rows in the window.

---

### 1. LAG()

**What**: Returns value from PREVIOUS row (look backward)

**Syntax**:
```sql
LAG(column, offset, default_value) OVER ([PARTITION BY col] ORDER BY col)
```

**Parameters**:
- `column` - Which column to get value from
- `offset` - How many rows back (default: 1)
- `default_value` - Value if no previous row exists (default: NULL)

**Requires ORDER BY**: ✅ Yes (must specify order)

**Basic Example**:
```sql
SELECT
    order_date,
    revenue,
    LAG(revenue) OVER (ORDER BY order_date) AS prev_day_revenue
FROM daily_sales;

-- Result:
-- 2024-01-01, 100, NULL       ← No previous day
-- 2024-01-02, 150, 100        ← Previous day: 100
-- 2024-01-03, 200, 150        ← Previous day: 150
-- 2024-01-04, 120, 200        ← Previous day: 200
```

**With Offset**:
```sql
-- Get value from 2 rows back
LAG(revenue, 2) OVER (ORDER BY order_date)

-- Get value from 3 rows back with default
LAG(revenue, 3, 0) OVER (ORDER BY order_date)
```

**With PARTITION BY**:
```sql
SELECT
    customer_id,
    order_date,
    order_total,
    LAG(order_date) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
    ) AS prev_order_date
FROM orders;

-- Result:
-- Customer 123:
-- 123, 2024-01-01, 100, NULL      ← First order for this customer
-- 123, 2024-01-15, 150, 2024-01-01
-- 123, 2024-02-10, 200, 2024-01-15

-- Customer 456:
-- 456, 2024-01-05, 75, NULL       ← RESTARTS for new customer
-- 456, 2024-01-20, 120, 2024-01-05
```

**Calculate Day-over-Day Change**:
```sql
SELECT
    order_date,
    revenue,
    LAG(revenue) OVER (ORDER BY order_date) AS prev_revenue,
    revenue - LAG(revenue) OVER (ORDER BY order_date) AS revenue_change,
    ROUND(
        100.0 * (revenue - LAG(revenue) OVER (ORDER BY order_date)) /
        NULLIF(LAG(revenue) OVER (ORDER BY order_date), 0),
        2
    ) AS pct_change
FROM daily_sales;

-- Result:
-- 2024-01-01, 100, NULL, NULL, NULL
-- 2024-01-02, 150, 100, 50, 50.00%
-- 2024-01-03, 200, 150, 50, 33.33%
-- 2024-01-04, 120, 200, -80, -40.00%
```

**Use Cases**:
- **Month-over-month / day-over-day growth** (SUPER COMMON!)
- Days since last order
- Compare with previous value
- Detect changes/trends
- Session gaps

---

### 2. LEAD()

**What**: Returns value from NEXT row (look forward)

**Syntax**:
```sql
LEAD(column, offset, default_value) OVER ([PARTITION BY col] ORDER BY col)
```

**Parameters**: Same as LAG

**Requires ORDER BY**: ✅ Yes

**Basic Example**:
```sql
SELECT
    order_date,
    revenue,
    LEAD(revenue) OVER (ORDER BY order_date) AS next_day_revenue
FROM daily_sales;

-- Result:
-- 2024-01-01, 100, 150        ← Next day: 150
-- 2024-01-02, 150, 200        ← Next day: 200
-- 2024-01-03, 200, 120        ← Next day: 120
-- 2024-01-04, 120, NULL       ← No next day
```

**With Offset**:
```sql
-- Get value from 2 rows ahead
LEAD(revenue, 2) OVER (ORDER BY order_date)

-- Get value from 3 rows ahead with default
LEAD(revenue, 3, 0) OVER (ORDER BY order_date)
```

**Use Cases**:
- Compare with next value
- Forward-looking analysis
- Identify future changes
- Less common than LAG

---

### 3. FIRST_VALUE()

**What**: Returns value from FIRST row in window

**Syntax**:
```sql
FIRST_VALUE(column) OVER ([PARTITION BY col] ORDER BY col [frame_clause])
```

**Requires ORDER BY**: ✅ Yes (usually)

**Basic Example**:
```sql
SELECT
    name,
    department,
    salary,
    FIRST_VALUE(name) OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS highest_paid_in_dept
FROM employees;

-- Result:
-- Alice, Sales, 100000, Alice      ← Alice is highest in Sales
-- Bob, Sales, 90000, Alice          ← Alice is still highest
-- Charlie, Sales, 70000, Alice      ← Alice is still highest
-- David, IT, 80000, David           ← David is highest in IT
-- Eve, IT, 60000, David             ← David is still highest
```

**Get First Value in Entire Partition**:
```sql
FIRST_VALUE(order_date) OVER (
    PARTITION BY customer_id
    ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
) AS first_order_date
```

**Use Cases**:
- First order date per customer
- Highest/lowest value in group
- Benchmark against first value
- Compare to top performer

---

### 4. LAST_VALUE()

**What**: Returns value from LAST row in window

**Syntax**:
```sql
LAST_VALUE(column) OVER ([PARTITION BY col] ORDER BY col [frame_clause])
```

**Requires ORDER BY**: ✅ Yes

**⚠️ TRICKY - Default Frame Issue**:

**Without explicit frame** (WRONG - returns current row):
```sql
SELECT
    name,
    salary,
    LAST_VALUE(name) OVER (ORDER BY salary DESC) AS last_value
FROM employees;

-- Result (WRONG):
-- Alice, 100000, Alice      ← Returns CURRENT row!
-- Bob, 90000, Bob            ← Returns CURRENT row!
-- Charlie, 70000, Charlie    ← Returns CURRENT row!
```

**With explicit frame** (CORRECT):
```sql
SELECT
    name,
    department,
    salary,
    LAST_VALUE(name) OVER (
        PARTITION BY department
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS lowest_paid_in_dept
FROM employees;

-- Result:
-- Alice, Sales, 100000, Charlie     ← Charlie is lowest in Sales
-- Bob, Sales, 90000, Charlie        ← Charlie is lowest
-- Charlie, Sales, 70000, Charlie    ← Charlie is lowest
-- David, IT, 80000, Eve             ← Eve is lowest in IT
-- Eve, IT, 60000, Eve               ← Eve is lowest
```

**Why the frame is needed**: By default, window frame is "UNBOUNDED PRECEDING AND CURRENT ROW", so LAST_VALUE returns current row!

**Use Cases**:
- Last order date per customer
- Lowest value in group
- Compare to bottom performer
- **Tip**: Often easier to use MIN/MAX or reverse ORDER BY with FIRST_VALUE

---

### 5. NTH_VALUE()

**What**: Returns value from Nth row in window

**Syntax**:
```sql
NTH_VALUE(column, n) OVER ([PARTITION BY col] ORDER BY col [frame_clause])
```

**Parameters**:
- `column` - Which column to get
- `n` - Which row number (1 = first row, 2 = second row, etc.)

**Requires ORDER BY**: ✅ Yes

**Example**:
```sql
SELECT
    name,
    department,
    salary,
    NTH_VALUE(salary, 2) OVER (
        PARTITION BY department
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS second_highest_salary
FROM employees;

-- Result:
-- Alice, Sales, 100000, 90000       ← 2nd highest in Sales
-- Bob, Sales, 90000, 90000           ← 2nd highest is 90000
-- Charlie, Sales, 70000, 90000       ← Same
-- David, IT, 80000, 60000            ← 2nd highest in IT
-- Eve, IT, 60000, 60000              ← 2nd highest is 60000
```

**Use Cases**:
- Get specific position value (2nd highest, 3rd lowest)
- Compare to specific benchmark
- Less common (usually easier to use RANK/DENSE_RANK with filter)

---

## 🎛️ PARTITION BY EXPLAINED {#partition-by}

**What**: Divides rows into separate groups (partitions)

**Syntax**:
```sql
PARTITION BY column1, column2, ...
```

**Optional**: Yes (if omitted, entire result set is one partition)

---

### Without PARTITION BY:
```sql
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER () AS company_avg
FROM employees;

-- Result: Same company average for EVERY row
-- Alice, Sales, 100000, 75000
-- Bob, Sales, 90000, 75000
-- Charlie, IT, 80000, 75000
-- David, IT, 60000, 75000
```

---

### With PARTITION BY:
```sql
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees;

-- Result: Different average per department
-- Alice, Sales, 100000, 95000    ← Sales average
-- Bob, Sales, 90000, 95000        ← Sales average
-- Charlie, IT, 80000, 70000       ← IT average
-- David, IT, 60000, 70000         ← IT average
```

---

### Multiple Columns:
```sql
-- Partition by department AND job_title
PARTITION BY department, job_title

-- Partition by year AND month
PARTITION BY EXTRACT(YEAR FROM date), EXTRACT(MONTH FROM date)
```

---

### Key Concept:
**PARTITION BY is like GROUP BY, but doesn't collapse rows!**

- `GROUP BY department` → 2 rows (one per department)
- `PARTITION BY department` → All rows, calculations reset per department

---

## 📐 ORDER BY EXPLAINED {#order-by}

**What**: Defines the order of rows within each partition

**Syntax**:
```sql
ORDER BY column1 [ASC|DESC], column2 [ASC|DESC], ...
```

**Optional**: Depends on function
- **Required for**: ROW_NUMBER, RANK, DENSE_RANK, NTILE, LAG, LEAD, FIRST_VALUE, LAST_VALUE
- **Optional for**: SUM, AVG, COUNT, MIN, MAX
- **Changes behavior for**: Aggregate functions (running vs total)

---

### Impact on Aggregate Functions:

**Without ORDER BY** (total for entire partition):
```sql
SUM(revenue) OVER (PARTITION BY department)
-- Returns same total for all rows in department
```

**With ORDER BY** (running total):
```sql
SUM(revenue) OVER (PARTITION BY department ORDER BY date)
-- Returns cumulative sum up to current row
```

---

### ASC vs DESC:
```sql
ORDER BY salary DESC    -- Highest to lowest
ORDER BY salary ASC     -- Lowest to highest (default)
ORDER BY salary         -- Same as ASC
```

---

### Multiple Columns:
```sql
-- Sort by department, then by salary within each department
ORDER BY department, salary DESC

-- Sort by date, then by amount for same dates
ORDER BY order_date, order_amount DESC
```

---

### Tie Handling:
```sql
-- If salaries are tied, order is non-deterministic
ORDER BY salary DESC

-- Better: Add tie-breaker
ORDER BY salary DESC, employee_id ASC
```

---

## 🪟 ALL WINDOW FRAME SPECIFICATIONS {#window-frames}

**What**: Defines which rows within the partition to include in calculation

**Syntax**:
```sql
{ROWS | RANGE} BETWEEN frame_start AND frame_end
```

**ROWS vs RANGE**:
- **ROWS** - Physical row count (most common)
- **RANGE** - Logical value range (less common, based on ORDER BY values)

---

### Default Frames:

**If ORDER BY is present**:
- Default: `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`
- Equivalent to: All rows from start up to current row (running total)

**If ORDER BY is NOT present**:
- Default: `RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`
- Equivalent to: Entire partition

---

### Why Frames Matter:

**Running total** (default with ORDER BY):
```sql
SUM(revenue) OVER (ORDER BY date)
-- Implicitly: ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

**Total for entire partition**:
```sql
SUM(revenue) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
```

**Moving average (last 3 rows)**:
```sql
AVG(revenue) OVER (ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)
```

---

## 🎯 EVERY FRAME BOUNDARY OPTION {#frame-boundaries}

### Frame Start Options:
1. **UNBOUNDED PRECEDING** - From the very first row of partition
2. **N PRECEDING** - N rows before current row
3. **CURRENT ROW** - The current row
4. **N FOLLOWING** - N rows after current row (rare for start)

### Frame End Options:
1. **N PRECEDING** - N rows before current row (rare for end)
2. **CURRENT ROW** - The current row
3. **N FOLLOWING** - N rows after current row
4. **UNBOUNDED FOLLOWING** - To the very last row of partition

---

### All Common Frame Patterns:

#### 1. Running Total / Cumulative Sum
```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
-- From start to current row (default with ORDER BY)

-- Example:
-- Row 1: Sum of row 1
-- Row 2: Sum of rows 1-2
-- Row 3: Sum of rows 1-3
```

---

#### 2. Total for Entire Partition
```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
-- All rows in partition (default without ORDER BY)

-- Example:
-- Row 1: Sum of all rows
-- Row 2: Sum of all rows (same)
-- Row 3: Sum of all rows (same)
```

---

#### 3. Moving Average (Last N Rows)
```sql
ROWS BETWEEN (N-1) PRECEDING AND CURRENT ROW
-- Current row + (N-1) previous rows

-- 3-row moving average:
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW

-- Example:
-- Row 1: Avg of row 1 (only 1 row available)
-- Row 2: Avg of rows 1-2 (only 2 rows available)
-- Row 3: Avg of rows 1-3
-- Row 4: Avg of rows 2-4 ← Window slides!
-- Row 5: Avg of rows 3-5
```

---

#### 4. Centered Moving Average
```sql
ROWS BETWEEN N PRECEDING AND N FOLLOWING
-- N rows before, current row, N rows after

-- 3-row centered average:
ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING

-- Example:
-- Row 1: Avg of rows 1-2 (no row before)
-- Row 2: Avg of rows 1-3
-- Row 3: Avg of rows 2-4
-- Row 4: Avg of rows 3-5
-- Row 5: Avg of rows 4-5 (no row after)
```

---

#### 5. Only Following Rows
```sql
ROWS BETWEEN CURRENT ROW AND N FOLLOWING
-- Current row + N rows after

-- Example (next 2 rows):
ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING

-- Row 1: Sum of rows 1-3
-- Row 2: Sum of rows 2-4
-- Row 3: Sum of rows 3-5
```

---

#### 6. Remaining Rows
```sql
ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING
-- Current row to end

-- Example:
-- Row 1: Sum of rows 1-5 (all remaining)
-- Row 2: Sum of rows 2-5
-- Row 3: Sum of rows 3-5
-- Row 4: Sum of rows 4-5
-- Row 5: Sum of row 5
```

---

#### 7. Exclude Current Row
```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING
-- All rows before current (exclude current)

-- Example:
-- Row 1: NULL or 0 (no rows before)
-- Row 2: Sum of row 1
-- Row 3: Sum of rows 1-2
-- Row 4: Sum of rows 1-3
```

---

## 📖 COMPLETE SYNTAX BREAKDOWN {#syntax-breakdown}

### Full Anatomy:
```sql
SELECT
    column1,
    column2,
    FUNCTION_NAME(argument) OVER (
        PARTITION BY partition_column1, partition_column2
        ORDER BY order_column1 [ASC|DESC], order_column2 [ASC|DESC]
        {ROWS | RANGE} BETWEEN frame_start AND frame_end
    ) AS alias
FROM table;
```

---

### Each Component:

**1. FUNCTION_NAME(argument)**
- **Ranking**: ROW_NUMBER(), RANK(), DENSE_RANK(), NTILE(n)
- **Aggregate**: SUM(col), AVG(col), COUNT(col), MIN(col), MAX(col)
- **Value**: LAG(col, n), LEAD(col, n), FIRST_VALUE(col), LAST_VALUE(col), NTH_VALUE(col, n)

**2. OVER ()**
- Keyword that marks this as a window function
- Must always be present
- Can be empty: `COUNT(*) OVER ()`

**3. PARTITION BY** (Optional)
- Divides rows into groups
- Calculation resets for each group
- Like GROUP BY but doesn't collapse rows

**4. ORDER BY** (Required for some, optional for others)
- Defines row order within each partition
- Required: ROW_NUMBER, RANK, DENSE_RANK, NTILE, LAG, LEAD, FIRST_VALUE, LAST_VALUE
- Optional: SUM, AVG, COUNT, MIN, MAX (but changes behavior!)

**5. ROWS/RANGE BETWEEN** (Optional)
- Defines window frame (which rows to include)
- Only relevant when ORDER BY is present
- Default depends on presence of ORDER BY

---

## 📊 ALL FUNCTIONS SIDE-BY-SIDE {#all-functions-table}

### Complete Comparison Table:

| Function | Category | Requires ORDER BY | Default Frame | Use Case |
|----------|----------|-------------------|---------------|----------|
| **ROW_NUMBER()** | Ranking | ✅ Yes | N/A | Top N per group, unique IDs |
| **RANK()** | Ranking | ✅ Yes | N/A | Competition ranking with gaps |
| **DENSE_RANK()** | Ranking | ✅ Yes | N/A | **Nth highest value** |
| **NTILE(n)** | Ranking | ✅ Yes | N/A | Quartiles, percentiles |
| **SUM()** | Aggregate | ❌ No | Running total | Cumulative sum |
| **AVG()** | Aggregate | ❌ No | Running average | Moving average, dept avg |
| **COUNT()** | Aggregate | ❌ No | Running count | Count per group |
| **MIN()** | Aggregate | ❌ No | Min so far | Minimum in group |
| **MAX()** | Aggregate | ❌ No | Max so far | Maximum in group |
| **LAG()** | Value | ✅ Yes | N/A | **M-o-M growth, previous value** |
| **LEAD()** | Value | ✅ Yes | N/A | Next value |
| **FIRST_VALUE()** | Value | ✅ Yes | All rows | First in window |
| **LAST_VALUE()** | Value | ✅ Yes | Up to current | Last in window |
| **NTH_VALUE()** | Value | ✅ Yes | All rows | Specific position |

---

## 🔍 INTERVIEW QUESTION TYPE → FUNCTION MAPPING {#question-mapping}

### Question Contains... → Use This Function

| Question Keywords | Function to Use | Example |
|-------------------|----------------|---------|
| **"Top N per group"** | ROW_NUMBER() + PARTITION BY | Top 3 products per category |
| **"Rank"** | RANK() or DENSE_RANK() | Rank students by score |
| **"Nth highest/lowest"** | DENSE_RANK() | Find 2nd highest salary |
| **"Quartile" / "percentile"** | NTILE(n) | Divide into quartiles |
| **"Running total" / "cumulative"** | SUM() OVER ORDER BY | Cumulative sales |
| **"Average per group"** | AVG() OVER PARTITION BY | Avg salary per dept |
| **"Moving average"** | AVG() OVER + ROWS BETWEEN | 7-day moving average |
| **"Compare to previous"** | LAG() | Month-over-month growth |
| **"Day-over-day" / "M-o-M"** | LAG() | % change from last period |
| **"Compare to next"** | LEAD() | Compare to next value |
| **"First occurrence"** | FIRST_VALUE() or MIN() | First order date |
| **"Last occurrence"** | LAST_VALUE() or MAX() | Most recent order |
| **"Count per group"** | COUNT() OVER PARTITION BY | Employees per dept |
| **"Above/below average"** | AVG() OVER + comparison | Above dept average |
| **"Consecutive"** | ROW_NUMBER() trick | 3+ consecutive days |

---

## 🔗 COMMON COMBINATIONS {#common-combinations}

### 1. Top N Per Group (MOST COMMON!)
```sql
SELECT *
FROM (
    SELECT
        product_name,
        category,
        sales,
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC) AS rank
    FROM products
) ranked
WHERE rank <= 3;
```

---

### 2. Nth Highest Value
```sql
SELECT DISTINCT salary
FROM (
    SELECT
        salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
) ranked
WHERE rank = 2;
```

---

### 3. Month-over-Month Growth
```sql
SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue,
    ROUND(
        100.0 * (revenue - LAG(revenue) OVER (ORDER BY month)) /
        LAG(revenue) OVER (ORDER BY month),
        2
    ) AS growth_pct
FROM monthly_sales;
```

---

### 4. Compare to Group Average
```sql
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg,
    CASE
        WHEN salary > AVG(salary) OVER (PARTITION BY department) THEN 'Above Average'
        WHEN salary < AVG(salary) OVER (PARTITION BY department) THEN 'Below Average'
        ELSE 'At Average'
    END AS salary_position
FROM employees;
```

---

### 5. Running Total with Percentage
```sql
SELECT
    product,
    sales,
    SUM(sales) OVER (ORDER BY sales DESC) AS running_total,
    ROUND(
        100.0 * SUM(sales) OVER (ORDER BY sales DESC) / SUM(sales) OVER (),
        2
    ) AS cumulative_pct
FROM product_sales;
```

---

### 6. 7-Day Moving Average
```sql
SELECT
    date,
    value,
    AVG(value) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7day
FROM daily_metrics;
```

---

### 7. Days Since Last Event
```sql
SELECT
    customer_id,
    order_date,
    LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS prev_order,
    order_date - LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS days_since_last_order
FROM orders;
```

---

### 8. Quartile Analysis
```sql
SELECT
    customer_id,
    total_spending,
    NTILE(4) OVER (ORDER BY total_spending DESC) AS quartile,
    CASE
        WHEN NTILE(4) OVER (ORDER BY total_spending DESC) = 1 THEN 'Top 25%'
        WHEN NTILE(4) OVER (ORDER BY total_spending DESC) = 2 THEN '25-50%'
        WHEN NTILE(4) OVER (ORDER BY total_spending DESC) = 3 THEN '50-75%'
        ELSE 'Bottom 25%'
    END AS spending_tier
FROM customer_spending;
```

---

### 9. First and Last in Group
```sql
SELECT
    customer_id,
    order_date,
    FIRST_VALUE(order_date) OVER (
        PARTITION BY customer_id ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS first_order,
    LAST_VALUE(order_date) OVER (
        PARTITION BY customer_id ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_order
FROM orders;
```

---

### 10. Consecutive Events Detection
```sql
WITH numbered AS (
    SELECT
        user_id,
        activity_date,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY activity_date) AS rn,
        activity_date - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY activity_date) * INTERVAL '1 day' AS grp
    FROM user_activity
),
streaks AS (
    SELECT
        user_id,
        grp,
        MIN(activity_date) AS streak_start,
        MAX(activity_date) AS streak_end,
        COUNT(*) AS streak_length
    FROM numbered
    GROUP BY user_id, grp
)
SELECT *
FROM streaks
WHERE streak_length >= 3;
```

---

## 🎯 FINAL MEMORIZATION CHECKLIST

### Memorize These Patterns:

**Top N per group**:
```sql
ROW_NUMBER() OVER (PARTITION BY group ORDER BY value DESC) WHERE rank <= N
```

**Nth highest**:
```sql
DENSE_RANK() OVER (ORDER BY value DESC) WHERE rank = N
```

**Running total**:
```sql
SUM(amount) OVER (ORDER BY date)
```

**M-o-M growth**:
```sql
(val - LAG(val) OVER (ORDER BY month)) / LAG(val) OVER (ORDER BY month)
```

**Moving average**:
```sql
AVG(val) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)
```

**Compare to average**:
```sql
AVG(val) OVER (PARTITION BY group)
```

**Days since last**:
```sql
date - LAG(date) OVER (PARTITION BY id ORDER BY date)
```

---

## ✅ YOU NOW KNOW EVERY WINDOW FUNCTION!

### Total Functions: 14
- ✅ 4 Ranking Functions
- ✅ 5 Aggregate Functions
- ✅ 5 Value/Offset Functions

### Total Clauses: 3
- ✅ PARTITION BY
- ✅ ORDER BY
- ✅ ROWS/RANGE BETWEEN

### Total Frame Boundaries: 7
- ✅ UNBOUNDED PRECEDING
- ✅ N PRECEDING
- ✅ CURRENT ROW
- ✅ N FOLLOWING
- ✅ UNBOUNDED FOLLOWING
- ✅ ROWS
- ✅ RANGE

---

**PRINT THIS. REVIEW THIS. YOU'RE READY FOR ANY WINDOW FUNCTION QUESTION!** 🚀💪
