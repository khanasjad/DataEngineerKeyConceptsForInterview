# SQL WINDOW FUNCTIONS - COMPLETE GUIDE
## Master Every Window Function for Interviews

> **Philosophy**: Window functions are scary until you understand the pattern. Then they're incredibly powerful.

---

## 📚 TABLE OF CONTENTS
1. [What Are Window Functions?](#what-are)
2. [The Basic Syntax (OVER Clause)](#basic-syntax)
3. [PARTITION BY vs ORDER BY](#partition-vs-order)
4. [Ranking Functions](#ranking-functions)
5. [Aggregate Window Functions](#aggregate-functions)
6. [Value Functions (LAG, LEAD, FIRST_VALUE, LAST_VALUE)](#value-functions)
7. [Window Frames (ROWS BETWEEN)](#window-frames)
8. [Common Interview Patterns](#interview-patterns)
9. [Comparison Table](#comparison-table)
10. [Practice Questions](#practice-questions)

---

## 🤔 WHAT ARE WINDOW FUNCTIONS? {#what-are}

### The Problem:
You want to calculate something (rank, running total, average) **but keep all your rows**.

**GROUP BY** collapses rows:
```sql
-- This returns only 3 rows (one per department)
SELECT department, AVG(salary)
FROM employees
GROUP BY department;
```

**Window Functions** keep all rows:
```sql
-- This returns ALL employee rows, each with their dept average
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg_salary
FROM employees;
```

---

### Key Concept:
**Window functions perform calculations ACROSS a set of rows related to the current row, but WITHOUT collapsing them.**

Think of it as:
- **GROUP BY** = Collapse rows into groups
- **Window Functions** = Calculate across groups, keep all rows

---

### Why "Window"?
Because you define a "window" (or frame) of rows to perform the calculation on.

---

## 🎯 THE BASIC SYNTAX (OVER Clause) {#basic-syntax}

### Template:
```sql
SELECT
    column1,
    column2,
    WINDOW_FUNCTION() OVER (
        PARTITION BY column3
        ORDER BY column4
        ROWS BETWEEN ... AND ...
    ) AS alias
FROM table;
```

### Breaking It Down:

**1. WINDOW_FUNCTION()** - What you want to calculate:
- `ROW_NUMBER()` - Assign unique numbers
- `RANK()` - Rank with gaps
- `SUM()` - Running total
- `AVG()` - Moving average
- `LAG()` - Previous row value
- etc.

**2. OVER (...)** - Defines the window:
- `PARTITION BY` - Divide into groups (optional)
- `ORDER BY` - Order within each partition (required for some functions)
- `ROWS BETWEEN` - Define the frame (optional)

---

### Simple Example:
```sql
SELECT
    name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS salary_rank
FROM employees;

-- Result:
-- Alice, 100000, 1
-- Bob, 95000, 2
-- Charlie, 90000, 3
```

**What happened?**
- Sorted all employees by salary (descending)
- Assigned numbers 1, 2, 3, ...

---

## 🔀 PARTITION BY vs ORDER BY {#partition-vs-order}

### PARTITION BY - "Group rows into partitions"

**Without PARTITION BY** - One big window (all rows):
```sql
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER () AS company_avg
FROM employees;

-- Result:
-- Alice, Sales, 100000, 75000    (company average)
-- Bob, Sales, 90000, 75000        (same for all)
-- Charlie, IT, 80000, 75000       (same for all)
-- David, IT, 60000, 75000         (same for all)
```

---

**With PARTITION BY** - Separate window per group:
```sql
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees;

-- Result:
-- Alice, Sales, 100000, 95000    (Sales average)
-- Bob, Sales, 90000, 95000        (Sales average)
-- Charlie, IT, 80000, 70000       (IT average)
-- David, IT, 60000, 70000         (IT average)
```

**Key**: `PARTITION BY` restarts the calculation for each group!

---

### ORDER BY - "Sort rows within each partition"

```sql
SELECT
    name,
    department,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;

-- Result:
-- Alice, Sales, 100000, 1        (Rank within Sales)
-- Bob, Sales, 90000, 2            (Rank within Sales)
-- Charlie, IT, 80000, 1           (Rank within IT)
-- David, IT, 60000, 2             (Rank within IT)
```

**Key**: `ORDER BY` determines the sequence for calculations!

---

### Mental Model:

**PARTITION BY** = "For each department separately..."
**ORDER BY** = "...sorted by salary..."
**WINDOW_FUNCTION** = "...calculate the rank/sum/average/etc."

---

## 🏆 RANKING FUNCTIONS {#ranking-functions}

### The 4 Ranking Functions:
1. **ROW_NUMBER()** - Unique numbers (1, 2, 3, 4, ...)
2. **RANK()** - Rank with gaps (1, 2, 2, 4, ...)
3. **DENSE_RANK()** - Rank without gaps (1, 2, 2, 3, ...)
4. **NTILE(n)** - Divide into n buckets (1, 1, 2, 2, 3, 3, ...)

---

### Sample Data:
```
name       | department | salary
-----------|------------|--------
Alice      | Sales      | 100000
Bob        | Sales      | 90000
Charlie    | Sales      | 90000   (same as Bob!)
David      | Sales      | 80000
Eve        | Sales      | 70000
```

---

### 1. ROW_NUMBER() - Always Unique

```sql
SELECT
    name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num
FROM employees;

-- Result:
-- Alice, 100000, 1
-- Bob, 90000, 2
-- Charlie, 90000, 3    ← Even though tied with Bob, gets next number
-- David, 80000, 4
-- Eve, 70000, 5
```

**Use when**: You need unique row identifiers, even for ties.

**Common use**: "Get top 3 per group" (see interview patterns)

---

### 2. RANK() - Gaps for Ties

```sql
SELECT
    name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- Result:
-- Alice, 100000, 1
-- Bob, 90000, 2
-- Charlie, 90000, 2    ← Same rank as Bob
-- David, 80000, 4      ← Skips 3!
-- Eve, 70000, 5
```

**Use when**: You want traditional ranking (like Olympics - two gold medals means next is bronze, not silver).

---

### 3. DENSE_RANK() - No Gaps for Ties

```sql
SELECT
    name,
    salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;

-- Result:
-- Alice, 100000, 1
-- Bob, 90000, 2
-- Charlie, 90000, 2    ← Same rank as Bob
-- David, 80000, 3      ← No gap! Next consecutive number
-- Eve, 70000, 4
```

**Use when**: You want continuous ranking (no gaps).

**Common interview question**: "Find Nth highest salary" → Use DENSE_RANK

---

### 4. NTILE(n) - Divide into Buckets

```sql
SELECT
    name,
    salary,
    NTILE(3) OVER (ORDER BY salary DESC) AS quartile
FROM employees;

-- Result (divides 5 rows into 3 groups):
-- Alice, 100000, 1      (Top third)
-- Bob, 90000, 1         (Top third)
-- Charlie, 90000, 2     (Middle third)
-- David, 80000, 2       (Middle third)
-- Eve, 70000, 3         (Bottom third)
```

**Use when**: You want to divide data into equal buckets (quartiles, percentiles).

**Common use**: "Top 25% of customers by spending" = `NTILE(4) ... WHERE quartile = 1`

---

### Side-by-Side Comparison:

| salary  | ROW_NUMBER | RANK | DENSE_RANK | NTILE(3) |
|---------|------------|------|------------|----------|
| 100000  | 1          | 1    | 1          | 1        |
| 90000   | 2          | 2    | 2          | 1        |
| 90000   | 3          | 2    | 2          | 2        |
| 80000   | 4          | 4    | 3          | 2        |
| 70000   | 5          | 5    | 4          | 3        |

---

## 📊 AGGREGATE WINDOW FUNCTIONS {#aggregate-functions}

Regular aggregates (SUM, AVG, COUNT, MIN, MAX) can be used as window functions!

### Key Difference:
- **Regular aggregate**: Collapses rows
- **Window aggregate**: Keeps all rows, adds calculated column

---

### 1. SUM() OVER - Running Total

```sql
SELECT
    order_date,
    revenue,
    SUM(revenue) OVER (ORDER BY order_date) AS running_total
FROM orders;

-- Result:
-- 2024-01-01, 100, 100        (100)
-- 2024-01-02, 150, 250        (100 + 150)
-- 2024-01-03, 200, 450        (100 + 150 + 200)
-- 2024-01-04, 120, 570        (100 + 150 + 200 + 120)
```

**What's happening**: Each row shows the cumulative sum up to that point.

---

**Running total per group**:
```sql
SELECT
    department,
    employee_name,
    salary,
    SUM(salary) OVER (PARTITION BY department ORDER BY employee_name) AS running_dept_total
FROM employees;

-- Result:
-- Sales, Alice, 100000, 100000
-- Sales, Bob, 90000, 190000       (100000 + 90000)
-- IT, Charlie, 80000, 80000       (Resets for IT!)
-- IT, David, 60000, 140000        (80000 + 60000)
```

---

### 2. AVG() OVER - Moving Average

```sql
SELECT
    order_date,
    revenue,
    AVG(revenue) OVER (ORDER BY order_date) AS cumulative_avg
FROM orders;

-- Result:
-- 2024-01-01, 100, 100           (100 / 1)
-- 2024-01-02, 150, 125           ((100 + 150) / 2)
-- 2024-01-03, 200, 150           ((100 + 150 + 200) / 3)
-- 2024-01-04, 120, 142.5         ((100 + 150 + 200 + 120) / 4)
```

---

**Average per group**:
```sql
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees;

-- Result:
-- Alice, Sales, 100000, 95000    (Average of all Sales salaries)
-- Bob, Sales, 90000, 95000        (Same for all Sales employees)
-- Charlie, IT, 80000, 70000       (Average of all IT salaries)
-- David, IT, 60000, 70000         (Same for all IT employees)
```

**Use case**: "Show each employee's salary and how it compares to department average"

```sql
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg,
    salary - AVG(salary) OVER (PARTITION BY department) AS diff_from_avg
FROM employees;
```

---

### 3. COUNT() OVER - Cumulative Count

```sql
SELECT
    order_date,
    customer_id,
    COUNT(*) OVER (ORDER BY order_date) AS cumulative_orders
FROM orders;

-- Result:
-- 2024-01-01, 123, 1
-- 2024-01-02, 456, 2
-- 2024-01-03, 789, 3
```

---

**Count per group**:
```sql
SELECT
    name,
    department,
    COUNT(*) OVER (PARTITION BY department) AS dept_employee_count
FROM employees;

-- Result:
-- Alice, Sales, 3        (3 employees in Sales)
-- Bob, Sales, 3          (Same for all Sales rows)
-- Charlie, Sales, 3
-- David, IT, 2           (2 employees in IT)
-- Eve, IT, 2             (Same for all IT rows)
```

---

### 4. MIN() and MAX() OVER

```sql
SELECT
    name,
    department,
    salary,
    MIN(salary) OVER (PARTITION BY department) AS dept_min_salary,
    MAX(salary) OVER (PARTITION BY department) AS dept_max_salary
FROM employees;

-- Result:
-- Alice, Sales, 100000, 70000, 100000    (Min and max in Sales)
-- Bob, Sales, 90000, 70000, 100000
-- Charlie, Sales, 70000, 70000, 100000
-- David, IT, 80000, 60000, 80000         (Min and max in IT)
-- Eve, IT, 60000, 60000, 80000
```

**Use case**: "Show salary range within each department"

---

## 🔄 VALUE FUNCTIONS (LAG, LEAD, FIRST_VALUE, LAST_VALUE) {#value-functions}

These functions access values from other rows in the window.

---

### 1. LAG() - Previous Row Value

**Syntax**: `LAG(column, offset, default) OVER (ORDER BY ...)`
- `column` - Which column to get
- `offset` - How many rows back (default: 1)
- `default` - Value if no previous row exists (default: NULL)

```sql
SELECT
    order_date,
    revenue,
    LAG(revenue) OVER (ORDER BY order_date) AS previous_day_revenue
FROM daily_sales;

-- Result:
-- 2024-01-01, 100, NULL       (No previous day)
-- 2024-01-02, 150, 100        (Previous day: 100)
-- 2024-01-03, 200, 150        (Previous day: 150)
-- 2024-01-04, 120, 200        (Previous day: 200)
```

---

**Calculate day-over-day change**:
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
-- 2024-01-02, 150, 100, 50, 50.00
-- 2024-01-03, 200, 150, 50, 33.33
-- 2024-01-04, 120, 200, -80, -40.00
```

---

**LAG with PARTITION BY** (Previous value within group):
```sql
SELECT
    customer_id,
    order_date,
    order_total,
    LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS prev_order_date,
    order_date - LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS days_since_last_order
FROM orders;

-- Result (for customer 123):
-- 123, 2024-01-01, 100, NULL, NULL
-- 123, 2024-01-15, 150, 2024-01-01, 14
-- 123, 2024-02-10, 200, 2024-01-15, 26

-- Result (for customer 456):
-- 456, 2024-01-05, 75, NULL, NULL     (Resets for new customer!)
-- 456, 2024-01-20, 120, 2024-01-05, 15
```

**SUPER COMMON INTERVIEW PATTERN!**

---

### 2. LEAD() - Next Row Value

**Same as LAG, but looks forward instead of backward**

```sql
SELECT
    order_date,
    revenue,
    LEAD(revenue) OVER (ORDER BY order_date) AS next_day_revenue
FROM daily_sales;

-- Result:
-- 2024-01-01, 100, 150        (Next day: 150)
-- 2024-01-02, 150, 200        (Next day: 200)
-- 2024-01-03, 200, 120        (Next day: 120)
-- 2024-01-04, 120, NULL       (No next day)
```

**Use case**: Less common than LAG, but useful for forward-looking comparisons.

---

### 3. FIRST_VALUE() - First Row in Window

```sql
SELECT
    name,
    department,
    salary,
    FIRST_VALUE(name) OVER (PARTITION BY department ORDER BY salary DESC) AS highest_paid_in_dept
FROM employees;

-- Result:
-- Alice, Sales, 100000, Alice      (Alice is highest in Sales)
-- Bob, Sales, 90000, Alice          (Alice is still highest)
-- Charlie, Sales, 70000, Alice      (Alice is still highest)
-- David, IT, 80000, David           (David is highest in IT)
-- Eve, IT, 60000, David             (David is still highest)
```

**Use case**: "Compare each employee to the top performer in their department"

---

### 4. LAST_VALUE() - Last Row in Window

**⚠️ TRICKY!** By default, the window frame is "ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW", so LAST_VALUE returns the current row!

**Wrong**:
```sql
SELECT
    name,
    department,
    salary,
    LAST_VALUE(name) OVER (PARTITION BY department ORDER BY salary DESC) AS last_value
FROM employees;

-- Result (WRONG!):
-- Alice, Sales, 100000, Alice      (Current row!)
-- Bob, Sales, 90000, Bob            (Current row!)
-- Charlie, Sales, 70000, Charlie    (Current row!)
```

---

**Correct** (specify full window frame):
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
-- Alice, Sales, 100000, Charlie     (Charlie is lowest in Sales)
-- Bob, Sales, 90000, Charlie        (Charlie is lowest)
-- Charlie, Sales, 70000, Charlie    (Charlie is lowest)
-- David, IT, 80000, Eve             (Eve is lowest in IT)
-- Eve, IT, 60000, Eve               (Eve is lowest)
```

**Tip**: FIRST_VALUE is easier to use. Avoid LAST_VALUE unless necessary!

---

## 🪟 WINDOW FRAMES (ROWS BETWEEN) {#window-frames}

### What Are Window Frames?

By default, window functions calculate over **all rows from the start to the current row**.

You can change this with **ROWS BETWEEN** or **RANGE BETWEEN**.

---

### Default Frame:
```sql
-- These are equivalent:
SUM(revenue) OVER (ORDER BY order_date)

SUM(revenue) OVER (
    ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

**Translation**: "Sum from the very first row up to the current row" (running total)

---

### Frame Syntax:
```sql
ROWS BETWEEN <start> AND <end>
```

**Options for start/end**:
- `UNBOUNDED PRECEDING` - From the very first row
- `UNBOUNDED FOLLOWING` - To the very last row
- `CURRENT ROW` - The current row
- `N PRECEDING` - N rows before current
- `N FOLLOWING` - N rows after current

---

### Example 1: Moving Average (Last 3 Days)

```sql
SELECT
    order_date,
    revenue,
    AVG(revenue) OVER (
        ORDER BY order_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3_day
FROM daily_sales;

-- Result:
-- 2024-01-01, 100, 100             ((100) / 1)
-- 2024-01-02, 150, 125             ((100 + 150) / 2)
-- 2024-01-03, 200, 150             ((100 + 150 + 200) / 3)
-- 2024-01-04, 120, 156.67          ((150 + 200 + 120) / 3)   ← Only last 3!
-- 2024-01-05, 180, 166.67          ((200 + 120 + 180) / 3)
```

**Translation**: "Average of current row + 2 rows before"

---

### Example 2: Centered Moving Average (1 Before, Current, 1 After)

```sql
SELECT
    order_date,
    revenue,
    AVG(revenue) OVER (
        ORDER BY order_date
        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
    ) AS centered_avg
FROM daily_sales;

-- Result:
-- 2024-01-01, 100, 125             ((100 + 150) / 2)          ← No row before
-- 2024-01-02, 150, 150             ((100 + 150 + 200) / 3)
-- 2024-01-03, 200, 156.67          ((150 + 200 + 120) / 3)
-- 2024-01-04, 120, 166.67          ((200 + 120 + 180) / 3)
-- 2024-01-05, 180, 150             ((120 + 180) / 2)          ← No row after
```

---

### Example 3: Full Partition (All Rows)

```sql
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER (
        PARTITION BY department
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS dept_avg
FROM employees;

-- This is the same as:
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees;
```

**When ORDER BY is not specified, the default frame is the entire partition.**

---

### Common Frame Patterns:

| Frame                                              | Use Case                    |
|----------------------------------------------------|-----------------------------|
| `UNBOUNDED PRECEDING AND CURRENT ROW`              | Running total / cumulative  |
| `UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`      | Total for entire partition  |
| `2 PRECEDING AND CURRENT ROW`                      | 3-row moving average        |
| `1 PRECEDING AND 1 FOLLOWING`                      | Centered 3-row average      |
| `CURRENT ROW AND UNBOUNDED FOLLOWING`              | Remaining sum               |

---

## 🔥 COMMON INTERVIEW PATTERNS {#interview-patterns}

### Pattern 1: "Top N Per Group"

**Question**: "Find top 3 highest-paid employees per department"

```sql
SELECT *
FROM (
    SELECT
        name,
        department,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
    FROM employees
) ranked
WHERE rank <= 3;
```

**⭐ THIS IS THE MOST COMMON WINDOW FUNCTION INTERVIEW QUESTION!**

---

### Pattern 2: "Find Nth Highest Value"

**Question**: "Find 2nd highest salary"

```sql
-- Using DENSE_RANK (handles ties correctly)
SELECT DISTINCT salary
FROM (
    SELECT
        salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
) ranked
WHERE rank = 2;
```

**Why DENSE_RANK instead of ROW_NUMBER?**
- If two people have the highest salary, DENSE_RANK gives both rank 1
- The next salary gets rank 2 (correct!)
- ROW_NUMBER would give them 1 and 2, then 3 (wrong!)

---

### Pattern 3: "Running Total"

**Question**: "Show cumulative revenue by date"

```sql
SELECT
    order_date,
    revenue,
    SUM(revenue) OVER (ORDER BY order_date) AS cumulative_revenue
FROM daily_sales;
```

---

### Pattern 4: "Day-over-Day / Month-over-Month Change"

**Question**: "Calculate daily revenue growth percentage"

```sql
SELECT
    order_date,
    revenue,
    LAG(revenue) OVER (ORDER BY order_date) AS prev_day_revenue,
    ROUND(
        100.0 * (revenue - LAG(revenue) OVER (ORDER BY order_date)) /
        NULLIF(LAG(revenue) OVER (ORDER BY order_date), 0),
        2
    ) AS pct_change
FROM daily_sales;
```

---

### Pattern 5: "Compare to Average"

**Question**: "Find employees earning above department average"

```sql
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg,
    salary - AVG(salary) OVER (PARTITION BY department) AS diff_from_avg
FROM employees
WHERE salary > AVG(salary) OVER (PARTITION BY department);

-- Note: This won't work! Can't use window function in WHERE

-- Correct approach:
SELECT *
FROM (
    SELECT
        name,
        department,
        salary,
        AVG(salary) OVER (PARTITION BY department) AS dept_avg
    FROM employees
) emp_with_avg
WHERE salary > dept_avg;
```

---

### Pattern 6: "Percentile / Quartile"

**Question**: "Divide customers into quartiles by spending"

```sql
SELECT
    customer_id,
    total_spending,
    NTILE(4) OVER (ORDER BY total_spending DESC) AS quartile
FROM customer_spending;

-- Then filter for top 25%:
WHERE NTILE(4) OVER (ORDER BY total_spending DESC) = 1
```

---

### Pattern 7: "Moving Average"

**Question**: "Calculate 7-day moving average of sales"

```sql
SELECT
    order_date,
    revenue,
    AVG(revenue) OVER (
        ORDER BY order_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7_day
FROM daily_sales;
```

---

### Pattern 8: "Consecutive Events (Gap & Islands)"

**Question**: "Find users with 3+ consecutive days of logins"

```sql
WITH login_with_row_num AS (
    SELECT
        user_id,
        login_date,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) AS rn
    FROM user_logins
),
login_with_groups AS (
    SELECT
        user_id,
        login_date,
        login_date - rn * INTERVAL '1 day' AS group_id
    FROM login_with_row_num
),
consecutive_counts AS (
    SELECT
        user_id,
        group_id,
        COUNT(*) AS consecutive_days,
        MIN(login_date) AS streak_start,
        MAX(login_date) AS streak_end
    FROM login_with_groups
    GROUP BY user_id, group_id
)
SELECT *
FROM consecutive_counts
WHERE consecutive_days >= 3;
```

**This is an advanced pattern, but shows up in tech company interviews!**

---

### Pattern 9: "First/Last Event Per Group"

**Question**: "Find first order date for each customer"

```sql
SELECT DISTINCT
    customer_id,
    FIRST_VALUE(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS first_order_date
FROM orders;

-- OR simpler with GROUP BY:
SELECT
    customer_id,
    MIN(order_date) AS first_order_date
FROM orders
GROUP BY customer_id;
```

**When to use window function vs GROUP BY?**
- Use GROUP BY if you only need the aggregate (simpler)
- Use window function if you need other columns from the original rows

---

### Pattern 10: "Dense Rank for Leaderboard"

**Question**: "Create a game leaderboard with ranks (no gaps for ties)"

```sql
SELECT
    player_name,
    score,
    DENSE_RANK() OVER (ORDER BY score DESC) AS rank
FROM game_scores;

-- Result:
-- Alice, 1000, 1
-- Bob, 950, 2
-- Charlie, 950, 2
-- David, 900, 3      ← Rank 3, not 4!
```

---

## 📋 COMPARISON TABLE {#comparison-table}

### All Window Functions at a Glance:

| Function | Category | What It Does | Requires ORDER BY? | Example |
|----------|----------|--------------|-------------------|---------|
| **ROW_NUMBER()** | Ranking | Unique sequential numbers | Yes | 1, 2, 3, 4 |
| **RANK()** | Ranking | Rank with gaps for ties | Yes | 1, 2, 2, 4 |
| **DENSE_RANK()** | Ranking | Rank without gaps | Yes | 1, 2, 2, 3 |
| **NTILE(n)** | Ranking | Divide into n buckets | Yes | 1, 1, 2, 2, 3, 3 |
| **SUM()** | Aggregate | Running/windowed sum | Optional | Cumulative total |
| **AVG()** | Aggregate | Running/windowed average | Optional | Moving average |
| **COUNT()** | Aggregate | Running/windowed count | Optional | Cumulative count |
| **MIN()** | Aggregate | Minimum in window | Optional | Lowest value |
| **MAX()** | Aggregate | Maximum in window | Optional | Highest value |
| **LAG()** | Value | Previous row value | Yes | Look back N rows |
| **LEAD()** | Value | Next row value | Yes | Look ahead N rows |
| **FIRST_VALUE()** | Value | First row in window | Yes | First in partition |
| **LAST_VALUE()** | Value | Last row in window | Yes | Last in partition |

---

### When to Use Which Ranking Function:

| Need | Use | Example |
|------|-----|---------|
| Unique row numbers (no ties) | ROW_NUMBER() | Pagination, Top N |
| Traditional ranking (gaps for ties) | RANK() | Competition ranking |
| Continuous ranking (no gaps) | DENSE_RANK() | Nth highest value |
| Divide into equal groups | NTILE(n) | Quartiles, percentiles |

---

## 📝 PRACTICE QUESTIONS {#practice-questions}

### Easy:
1. ✅ **List all employees with their department's average salary**
2. ✅ **Rank employees by salary (highest to lowest)**
3. ✅ **Find the 2nd highest salary in the company**
4. ✅ **Calculate running total of daily sales**
5. ✅ **Show each order with the previous order date for the same customer**

---

### Medium:
6. ✅ **Find top 3 products by sales in each category**
7. ✅ **Calculate month-over-month revenue growth percentage**
8. ✅ **Find customers whose current order is higher than their previous order**
9. ✅ **Divide customers into quartiles by total spending**
10. ✅ **Calculate 7-day moving average of website traffic**

---

### Hard:
11. ✅ **Find employees earning more than 80% of their department**
12. ✅ **Identify users with 3+ consecutive days of activity**
13. ✅ **Calculate retention rate (users active this month AND last month)**
14. ✅ **Find the longest winning streak for each team**
15. ✅ **Detect session boundaries (new session if gap > 30 minutes)**

---

## 🎯 INTERVIEW CHEAT SHEET

### If the question says...

**"Top N per group"** → ROW_NUMBER() + PARTITION BY
```sql
ROW_NUMBER() OVER (PARTITION BY group_col ORDER BY value_col DESC)
WHERE rank <= N
```

**"Nth highest/lowest"** → DENSE_RANK()
```sql
DENSE_RANK() OVER (ORDER BY value_col DESC)
WHERE rank = N
```

**"Running total"** → SUM() OVER
```sql
SUM(value_col) OVER (ORDER BY date_col)
```

**"Compare with previous"** → LAG()
```sql
value_col - LAG(value_col) OVER (ORDER BY date_col)
```

**"Moving average"** → AVG() with ROWS BETWEEN
```sql
AVG(value_col) OVER (ORDER BY date_col ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)
```

**"Divide into buckets"** → NTILE()
```sql
NTILE(4) OVER (ORDER BY value_col)
```

---

## 💡 KEY TAKEAWAYS

### Remember:
1. **Window functions keep all rows** (unlike GROUP BY)
2. **OVER() defines the window** (PARTITION BY + ORDER BY + ROWS BETWEEN)
3. **PARTITION BY = separate calculations per group**
4. **ORDER BY = sequence matters**
5. **Can't use window functions in WHERE** (use subquery instead)

### The Big 5 for Interviews:
1. **ROW_NUMBER()** - Top N per group
2. **DENSE_RANK()** - Nth highest value
3. **SUM() OVER** - Running total
4. **LAG()** - Compare with previous
5. **AVG() with ROWS BETWEEN** - Moving average

### Common Mistakes:
❌ Using ROW_NUMBER() for Nth highest (use DENSE_RANK)
❌ Forgetting PARTITION BY (applies to all rows instead of per group)
❌ Using window function in WHERE clause (use subquery)
❌ Confusing LAG (previous) with LEAD (next)
❌ Not specifying frame for LAST_VALUE (gets current row!)

---

## 🚀 YOU'RE READY!

You now understand:
- ✅ What window functions are and why they're powerful
- ✅ OVER clause syntax (PARTITION BY, ORDER BY, ROWS BETWEEN)
- ✅ All 4 ranking functions (ROW_NUMBER, RANK, DENSE_RANK, NTILE)
- ✅ Aggregate window functions (SUM, AVG, COUNT, MIN, MAX)
- ✅ Value functions (LAG, LEAD, FIRST_VALUE, LAST_VALUE)
- ✅ Window frames (ROWS BETWEEN)
- ✅ 10 common interview patterns

**Practice 5-10 questions and you'll master this!** 💪

---

**Remember**: Window functions look scary, but follow a simple pattern:
1. **What** do you want? (ROW_NUMBER, SUM, LAG, etc.)
2. **Grouped by what?** (PARTITION BY)
3. **Sorted how?** (ORDER BY)
4. **Over what range?** (ROWS BETWEEN)

Fill in those blanks and you're golden! ✨
