# SQL Concepts Explained Simply - Window Functions, CTEs & Performance

**A beginner-friendly guide with real-world examples**

---

## Table of Contents
1. [SQL Window Functions](#sql-window-functions)
2. [CTEs & Subqueries](#ctes--subqueries)
3. [SQL Performance](#sql-performance)

---

# SQL Window Functions

## What Are Window Functions? (Simple Explanation)

Imagine you have a spreadsheet with employee salaries. You want to:
- Number each row
- Rank employees by salary
- Compare each employee's salary with the next person

**Normal SQL**: You can only work with ONE row at a time or GROUP ALL rows together.

**Window Functions**: You can look at OTHER rows while staying on your CURRENT row!

Think of it like looking through a "window" to see neighboring rows without leaving your seat.

---

## The Basic Syntax

```sql
SELECT
    column1,
    WINDOW_FUNCTION() OVER (PARTITION BY column2 ORDER BY column3)
FROM table;
```

**Breaking it down:**
- `WINDOW_FUNCTION()`: What you want to calculate (rank, number, etc.)
- `OVER`: "Hey, I'm using a window function!"
- `PARTITION BY`: "Group data by this column" (optional - like GROUP BY, but doesn't collapse rows)
- `ORDER BY`: "Sort within each group" (required for most functions)

---

## 1. ROW_NUMBER() - Give Each Row a Number

### Real-Life Analogy
Like numbering students in a line: 1, 2, 3, 4, 5...

### Example: Employee Salaries

**Sample Data:**
```
employees table:
| emp_id | name    | department | salary |
|--------|---------|------------|--------|
| 1      | Alice   | Sales      | 70000  |
| 2      | Bob     | Sales      | 60000  |
| 3      | Charlie | Sales      | 70000  |
| 4      | David   | IT         | 80000  |
| 5      | Emma    | IT         | 90000  |
```

### Query:
```sql
SELECT
    name,
    department,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS overall_row_num,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_row_num
FROM employees;
```

### Result:
```
| name    | department | salary | overall_row_num | dept_row_num |
|---------|------------|--------|-----------------|--------------|
| Emma    | IT         | 90000  | 1               | 1            |
| David   | IT         | 80000  | 2               | 2            |
| Alice   | Sales      | 70000  | 3               | 1            |
| Charlie | Sales      | 70000  | 4               | 2            |
| Bob     | Sales      | 60000  | 5               | 3            |
```

### Explanation:
- `overall_row_num`: Numbers ALL employees (1, 2, 3, 4, 5)
- `dept_row_num`: Numbers employees WITHIN each department (restarts at 1 for each dept)
- **Alice and Charlie both have 70000 but get DIFFERENT numbers** (3 and 4)

### When to Use:
- Getting top N records per group
- Pagination (LIMIT/OFFSET alternative)
- Deduplication (keep only ROW_NUMBER = 1)

---

## 2. RANK() - Rank with Gaps

### Real-Life Analogy
Like Olympic medals: If two people tie for gold (1, 1), the next person gets bronze (3) - silver is skipped!

### Example:

```sql
SELECT
    name,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank_with_gaps
FROM employees;
```

### Result:
```
| name    | department | salary | rank_with_gaps |
|---------|------------|--------|----------------|
| Emma    | IT         | 90000  | 1              |
| David   | IT         | 80000  | 2              |
| Alice   | Sales      | 70000  | 1              |
| Charlie | Sales      | 70000  | 1              | ← Tied for 1st
| Bob     | Sales      | 60000  | 3              | ← Skips 2!
```

### Key Point:
- **Alice and Charlie both have rank 1** (same salary)
- **Bob gets rank 3** (not 2) - there's a GAP

### When to Use:
- Sports rankings
- Top performers (ties allowed)
- When you want gaps after ties

---

## 3. DENSE_RANK() - Rank WITHOUT Gaps

### Real-Life Analogy
Like ranking students by grade: If two students tie for 1st place, the next student is 2nd (no gaps).

### Example:

```sql
SELECT
    name,
    department,
    salary,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dense_rank
FROM employees;
```

### Result:
```
| name    | department | salary | dense_rank |
|---------|------------|--------|------------|
| Emma    | IT         | 90000  | 1          |
| David   | IT         | 80000  | 2          |
| Alice   | Sales      | 70000  | 1          |
| Charlie | Sales      | 70000  | 1          | ← Tied for 1st
| Bob     | Sales      | 60000  | 2          | ← NO gap! (not 3)
```

### Key Point:
- **Alice and Charlie both have rank 1**
- **Bob gets rank 2** (continuous ranking, no gaps)

### When to Use:
- When you want continuous rankings
- Product category rankings
- Grade distributions

---

## Comparison: ROW_NUMBER vs RANK vs DENSE_RANK

**Using the same data:**

```sql
SELECT
    name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num,
    RANK() OVER (ORDER BY salary DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;
```

**Result:**
```
| name    | salary | row_num | rank | dense_rank |
|---------|--------|---------|------|------------|
| Emma    | 90000  | 1       | 1    | 1          |
| David   | 80000  | 2       | 2    | 2          |
| Alice   | 70000  | 3       | 3    | 3          | ← Tie starts
| Charlie | 70000  | 4       | 3    | 3          | ← Same rank
| Bob     | 60000  | 5       | 5    | 4          |
```

**Key Differences:**
- **ROW_NUMBER**: Always unique (3, 4, 5)
- **RANK**: Ties get same rank, then GAPS (3, 3, 5)
- **DENSE_RANK**: Ties get same rank, NO gaps (3, 3, 4)

---

## 4. LAG() and LEAD() - Look at Previous/Next Row

### Real-Life Analogy
- **LAG**: Look at the person BEHIND you in line
- **LEAD**: Look at the person IN FRONT of you in line

### Example: Monthly Sales Comparison

**Sample Data:**
```
sales table:
| month   | revenue |
|---------|---------|
| Jan     | 10000   |
| Feb     | 12000   |
| Mar     | 11000   |
| Apr     | 15000   |
```

### Query:
```sql
SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue,
    LEAD(revenue) OVER (ORDER BY month) AS next_month_revenue,
    revenue - LAG(revenue) OVER (ORDER BY month) AS month_over_month_change
FROM sales;
```

### Result:
```
| month | revenue | prev_month_revenue | next_month_revenue | month_over_month_change |
|-------|---------|--------------------|--------------------|-------------------------|
| Jan   | 10000   | NULL               | 12000              | NULL                    |
| Feb   | 12000   | 10000              | 11000              | 2000                    |
| Mar   | 11000   | 12000              | 15000              | -1000                   |
| Apr   | 15000   | 11000              | NULL               | 4000                    |
```

### Explanation:
- **Jan**: No previous month (NULL), next is 12000
- **Feb**: Previous is 10000, next is 11000, growth = +2000
- **Mar**: Previous is 12000, next is 15000, change = -1000 (decline)
- **Apr**: Previous is 11000, no next month (NULL), growth = +4000

### Advanced LAG/LEAD:

```sql
-- Look 2 months back
LAG(revenue, 2) OVER (ORDER BY month)

-- Look 3 months ahead with default value
LEAD(revenue, 3, 0) OVER (ORDER BY month)
```

### When to Use:
- Month-over-month / Year-over-year comparisons
- Calculate differences between consecutive rows
- Find gaps in sequences
- Running totals and deltas

---

## 5. NTILE(n) - Divide into Buckets

### Real-Life Analogy
Like dividing a class into 4 equal groups for team activities. NTILE(4) creates 4 buckets.

### Example: Customer Segmentation

**Sample Data:**
```
customers table:
| customer_id | name    | total_spent |
|-------------|---------|-------------|
| 1           | Alice   | 10000       |
| 2           | Bob     | 500         |
| 3           | Charlie | 8000        |
| 4           | David   | 1200        |
| 5           | Emma    | 15000       |
| 6           | Frank   | 3000        |
```

### Query:
```sql
SELECT
    name,
    total_spent,
    NTILE(3) OVER (ORDER BY total_spent DESC) AS customer_segment
FROM customers;
```

### Result:
```
| name    | total_spent | customer_segment |
|---------|-------------|------------------|
| Emma    | 15000       | 1                | ← Top 33% (High value)
| Alice   | 10000       | 1                |
| Charlie | 8000        | 2                | ← Middle 33% (Medium)
| Frank   | 3000        | 2                |
| David   | 1200        | 3                | ← Bottom 33% (Low)
| Bob     | 500         | 3                |
```

### Explanation:
- **NTILE(3)**: Divide customers into 3 equal groups
- **Segment 1**: Top customers (Emma, Alice)
- **Segment 2**: Medium customers (Charlie, Frank)
- **Segment 3**: Low-value customers (David, Bob)

### Use Case:
```sql
-- Create quartiles (25%, 50%, 75%, 100%)
SELECT
    product_name,
    price,
    CASE NTILE(4) OVER (ORDER BY price)
        WHEN 1 THEN 'Budget'
        WHEN 2 THEN 'Mid-Range'
        WHEN 3 THEN 'Premium'
        WHEN 4 THEN 'Luxury'
    END AS price_category
FROM products;
```

### When to Use:
- Customer segmentation (VIP, Regular, New)
- Percentile calculations
- A/B testing (split into equal groups)
- Performance distribution

---

## Common Use Cases: Real Interview Questions

### Q1: "Find the 2nd highest salary per department"

```sql
WITH ranked_salaries AS (
    SELECT
        department,
        name,
        salary,
        DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
    FROM employees
)
SELECT department, name, salary
FROM ranked_salaries
WHERE rank = 2;
```

**Why DENSE_RANK?** If multiple people have the highest salary, we still want the "second distinct salary."

---

### Q2: "Remove duplicate rows, keep only the latest record"

```sql
WITH numbered_rows AS (
    SELECT
        *,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY created_at DESC) AS rn
    FROM orders
)
SELECT *
FROM numbered_rows
WHERE rn = 1;
```

**Explanation:**
- `PARTITION BY customer_id`: Group by customer
- `ORDER BY created_at DESC`: Latest first
- `WHERE rn = 1`: Keep only the most recent order per customer

---

### Q3: "Calculate running total"

```sql
SELECT
    order_date,
    revenue,
    SUM(revenue) OVER (ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM daily_sales;
```

**Simpler version:**
```sql
SELECT
    order_date,
    revenue,
    SUM(revenue) OVER (ORDER BY order_date) AS running_total
FROM daily_sales;
```

---

### Q4: "Find month-over-month growth percentage"

```sql
SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue,
    ROUND(
        ((revenue - LAG(revenue) OVER (ORDER BY month)) / LAG(revenue) OVER (ORDER BY month)) * 100,
        2
    ) AS growth_pct
FROM monthly_sales;
```

---

## Window Function Cheat Sheet

| Function | What It Does | Ties? | Example |
|----------|--------------|-------|---------|
| `ROW_NUMBER()` | Sequential numbering | Unique numbers | 1, 2, 3, 4 |
| `RANK()` | Ranking with gaps | Same rank, skip next | 1, 2, 2, 4 |
| `DENSE_RANK()` | Ranking without gaps | Same rank, continue | 1, 2, 2, 3 |
| `NTILE(n)` | Divide into n buckets | Equal-ish groups | 1, 1, 2, 2, 3, 3 |
| `LAG(col, n)` | Value n rows before | N/A | Previous row |
| `LEAD(col, n)` | Value n rows ahead | N/A | Next row |
| `SUM() OVER()` | Running total | N/A | Cumulative sum |
| `AVG() OVER()` | Moving average | N/A | Rolling avg |

---

# CTEs & Subqueries

## What Is a Subquery? (Simple Explanation)

A **subquery** is a query INSIDE another query. Like a function that returns data to the main query.

### Real-Life Analogy
Making a sandwich:
1. **Subquery**: First, find the bread (SELECT bread FROM pantry)
2. **Main Query**: Then, make sandwich with that bread (SELECT sandwich FROM kitchen WHERE bread = ...)

---

## Types of Subqueries

### 1. Scalar Subquery (Returns Single Value)

```sql
-- Find employees earning more than average
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

**Step-by-step:**
1. Inner query runs: `SELECT AVG(salary) FROM employees` → Returns `72000`
2. Outer query becomes: `SELECT name, salary FROM employees WHERE salary > 72000`

---

### 2. Column Subquery (Returns List of Values)

```sql
-- Find customers who placed orders
SELECT name
FROM customers
WHERE customer_id IN (SELECT DISTINCT customer_id FROM orders);
```

**Step-by-step:**
1. Inner query: `SELECT DISTINCT customer_id FROM orders` → Returns `[1, 3, 5, 7]`
2. Outer query: `SELECT name FROM customers WHERE customer_id IN (1, 3, 5, 7)`

---

### 3. Table Subquery (Returns Multiple Rows/Columns)

```sql
-- Find departments with average salary > 70000
SELECT dept_name, avg_salary
FROM (
    SELECT department AS dept_name, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
) AS dept_averages
WHERE avg_salary > 70000;
```

---

### 4. Correlated Subquery (References Outer Query)

This is TRICKY! The inner query depends on the outer query.

```sql
-- Find employees earning more than their department's average
SELECT e1.name, e1.salary, e1.department
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department = e1.department  -- References outer query!
);
```

**How it works:**
1. For EACH row in outer query (e1):
   - Inner query calculates average for THAT row's department
   - Compare e1.salary with that average
2. Runs inner query MANY times (once per outer row)

**Warning**: Correlated subqueries are SLOW because they run repeatedly!

---

## What Is a CTE? (Common Table Expression)

A **CTE** is like creating a TEMPORARY named result set. Think of it as a "temporary view" that exists only during the query.

### Syntax:
```sql
WITH cte_name AS (
    SELECT ...
)
SELECT * FROM cte_name;
```

---

## CTE vs Subquery: Same Result, Different Approach

### Using Subquery:
```sql
SELECT dept_name, avg_salary
FROM (
    SELECT department AS dept_name, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
) AS dept_averages
WHERE avg_salary > 70000;
```

### Using CTE (CLEANER):
```sql
WITH dept_averages AS (
    SELECT department AS dept_name, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT dept_name, avg_salary
FROM dept_averages
WHERE avg_salary > 70000;
```

**Why CTE is better:**
- Easier to read
- Can reference multiple times
- Can chain multiple CTEs

---

## Multiple CTEs (Chaining)

```sql
WITH
    high_earners AS (
        SELECT * FROM employees WHERE salary > 80000
    ),
    sales_team AS (
        SELECT * FROM high_earners WHERE department = 'Sales'
    ),
    top_performers AS (
        SELECT * FROM sales_team WHERE performance_score > 90
    )
SELECT name, salary FROM top_performers;
```

**This is like:**
1. Filter high earners
2. From those, get sales team
3. From those, get top performers
4. Show final result

---

## Recursive CTE (Advanced but Important)

**Use case:** Hierarchical data (org charts, folder structures)

### Example: Employee Hierarchy

**Data:**
```
employees table:
| emp_id | name    | manager_id |
|--------|---------|------------|
| 1      | CEO     | NULL       |
| 2      | VP      | 1          |
| 3      | Manager | 2          |
| 4      | Dev     | 3          |
```

### Query (Find all reports under CEO):
```sql
WITH RECURSIVE org_chart AS (
    -- Base case: Start with CEO
    SELECT emp_id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case: Find direct reports
    SELECT e.emp_id, e.name, e.manager_id, oc.level + 1
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.emp_id
)
SELECT * FROM org_chart;
```

**Result:**
```
| emp_id | name    | manager_id | level |
|--------|---------|------------|-------|
| 1      | CEO     | NULL       | 1     |
| 2      | VP      | 1          | 2     |
| 3      | Manager | 2          | 3     |
| 4      | Dev     | 3          | 4     |
```

**How it works:**
1. Start with CEO (manager_id IS NULL)
2. Find all employees reporting to CEO
3. Find all employees reporting to THOSE employees
4. Continue until no more reports found

---

## CTE vs Subquery: When to Use What?

| Scenario | Use CTE | Use Subquery |
|----------|---------|--------------|
| Need to reference result multiple times | ✅ | ❌ |
| Complex multi-step logic | ✅ | ❌ |
| Recursive queries | ✅ | ❌ |
| Simple one-time filter | ⚠️ | ✅ |
| Very large datasets (performance) | ⚠️ | ✅ |

**Rule of thumb:** If query is hard to read, use CTE!

---

## Real Interview Questions

### Q1: "Find employees earning more than their manager"

**Using CTE:**
```sql
WITH emp_manager AS (
    SELECT
        e1.name AS employee,
        e1.salary AS emp_salary,
        e2.name AS manager,
        e2.salary AS mgr_salary
    FROM employees e1
    LEFT JOIN employees e2 ON e1.manager_id = e2.emp_id
)
SELECT employee, emp_salary, manager, mgr_salary
FROM emp_manager
WHERE emp_salary > mgr_salary;
```

---

### Q2: "Get top 3 products per category"

**Using CTE + Window Function:**
```sql
WITH ranked_products AS (
    SELECT
        category,
        product_name,
        revenue,
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY revenue DESC) AS rank
    FROM products
)
SELECT category, product_name, revenue
FROM ranked_products
WHERE rank <= 3;
```

---

### Q3: "Calculate cumulative sum with reset per group"

```sql
WITH ordered_sales AS (
    SELECT
        region,
        sale_date,
        amount,
        SUM(amount) OVER (
            PARTITION BY region
            ORDER BY sale_date
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS cumulative_total
    FROM sales
)
SELECT * FROM ordered_sales;
```

---

# SQL Performance

## Why Care About Performance?

**Scenario:**
- Query on 1,000 rows: Takes 0.1 seconds
- Same query on 1,000,000 rows: Takes 10+ seconds (or crashes!)

**Good performance** = Happy users, lower costs, scalable systems

---

## 1. Indexes - The Phone Book Analogy

### What Is an Index?

**Without Index (Full Table Scan):**
Finding "John Smith" in a phone book by reading EVERY page from start to finish.

**With Index:**
Using the alphabetical tabs to jump directly to "S" section.

### How Indexes Work:

**Table without index:**
```
employees table:
| emp_id | name    | department | salary |
|--------|---------|------------|--------|
| 1      | Alice   | Sales      | 70000  |
| 2      | Bob     | IT         | 80000  |
| 3      | Charlie | Sales      | 60000  |
| ...    | ...     | ...        | ...    |
| 1M     | Zara    | IT         | 90000  |
```

**Query:**
```sql
SELECT * FROM employees WHERE emp_id = 500000;
```

**Without index:** Database reads ALL 1 million rows (SLOW!)

**Creating an index:**
```sql
CREATE INDEX idx_emp_id ON employees(emp_id);
```

**With index:** Database uses index to jump directly to row 500000 (FAST!)

---

### Types of Indexes

#### A. B-Tree Index (Most Common)

Like a tree structure. Great for:
- Exact matches: `WHERE id = 100`
- Range queries: `WHERE salary BETWEEN 50000 AND 80000`
- Sorting: `ORDER BY created_at`

**Example:**
```sql
CREATE INDEX idx_salary ON employees(salary);

-- This is now FAST:
SELECT * FROM employees WHERE salary > 70000;
```

---

#### B. Hash Index

Like a hash table. Great for:
- Exact matches ONLY: `WHERE email = 'alice@company.com'`
- NOT for ranges or sorting

**Example:**
```sql
CREATE INDEX idx_email_hash ON employees USING HASH (email);

-- FAST:
SELECT * FROM employees WHERE email = 'alice@company.com';

-- SLOW (can't use hash index):
SELECT * FROM employees WHERE email LIKE 'alice%';
```

---

#### C. Bitmap Index

Used for low-cardinality columns (few unique values).

**Example: Gender column (only 3 values: Male, Female, Other)**

```sql
CREATE BITMAP INDEX idx_gender ON employees(gender);

-- FAST for columns with few distinct values:
SELECT * FROM employees WHERE gender = 'Female' AND department = 'Sales';
```

**When to use:**
- Data warehouse queries
- Columns with < 100 unique values
- Read-heavy workloads (NOT for frequent updates)

---

### When NOT to Use Indexes

❌ **Small tables** (< 1000 rows): Full scan is faster
❌ **Columns updated frequently**: Index maintenance slows down writes
❌ **Low selectivity**: If query returns 50%+ of rows, full scan is better

**Example of BAD index:**
```sql
-- DON'T index 'is_active' if 95% of rows are TRUE
CREATE INDEX idx_is_active ON users(is_active);
```

---

## 2. Composite Index (Multi-Column Index)

### What Is It?

Index on MULTIPLE columns together. Order matters!

**Real-Life Analogy:**
Phone book sorted by:
1. Last name (primary sort)
2. First name (secondary sort)

You can quickly find "Smith, John" but not "John" alone.

---

### Example:

**Table:**
```
orders table:
| order_id | customer_id | order_date | status    |
|----------|-------------|------------|-----------|
| 1        | 100         | 2024-01-15 | completed |
| 2        | 100         | 2024-02-20 | pending   |
| 3        | 101         | 2024-01-10 | completed |
```

**Create composite index:**
```sql
CREATE INDEX idx_customer_date ON orders(customer_id, order_date);
```

**This index helps these queries:**
```sql
-- ✅ Uses index (starts with customer_id)
SELECT * FROM orders WHERE customer_id = 100;

-- ✅ Uses index (both columns)
SELECT * FROM orders WHERE customer_id = 100 AND order_date > '2024-01-01';

-- ✅ Uses index (both columns)
SELECT * FROM orders WHERE customer_id = 100 AND order_date = '2024-01-15';
```

**This index DOESN'T help:**
```sql
-- ❌ Doesn't use index (starts with order_date, not customer_id)
SELECT * FROM orders WHERE order_date > '2024-01-01';
```

---

### Index Column Order Rules

**Rule:** Put the most selective column FIRST.

**Example:**
```sql
-- GOOD (customer_id is more selective)
CREATE INDEX idx_customer_status ON orders(customer_id, status);

-- BAD (status has only 3 values: pending, completed, cancelled)
CREATE INDEX idx_status_customer ON orders(status, customer_id);
```

**Why?** Database can quickly narrow down by customer (1000 customers) vs status (3 values).

---

## 3. EXPLAIN - See Query Execution Plan

### What Is EXPLAIN?

Shows HOW the database will execute your query (without actually running it).

### Syntax:
```sql
EXPLAIN SELECT * FROM employees WHERE department = 'Sales';
```

### Sample Output (PostgreSQL):
```
Seq Scan on employees  (cost=0.00..20.00 rows=5 width=100)
  Filter: (department = 'Sales'::text)
```

**Translation:**
- **Seq Scan**: Sequential scan (reading ALL rows) - SLOW!
- **cost=0.00..20.00**: Estimated cost (lower is better)
- **rows=5**: Expected to return 5 rows

---

### After Creating Index:
```sql
CREATE INDEX idx_department ON employees(department);

EXPLAIN SELECT * FROM employees WHERE department = 'Sales';
```

**Output:**
```
Index Scan using idx_department on employees  (cost=0.15..8.17 rows=5 width=100)
  Index Cond: (department = 'Sales'::text)
```

**Translation:**
- **Index Scan**: Using index - FAST!
- **cost=0.15..8.17**: Much lower cost (was 20.00)

---

### EXPLAIN ANALYZE (Actually Runs Query)

```sql
EXPLAIN ANALYZE SELECT * FROM employees WHERE department = 'Sales';
```

**Output:**
```
Index Scan using idx_department on employees  (cost=0.15..8.17 rows=5 width=100) (actual time=0.025..0.045 rows=5 loops=1)
  Index Cond: (department = 'Sales'::text)
Planning Time: 0.123 ms
Execution Time: 0.078 ms
```

**Key metrics:**
- **actual time**: Real time taken (0.045 ms)
- **rows**: Actual rows returned (5)
- **Execution Time**: Total time (0.078 ms)

---

### Reading EXPLAIN Output

**Common operations (from fastest to slowest):**

1. **Index Scan / Index Seek** ✅ - Using index (FAST)
2. **Index Only Scan** ✅ - Reading only from index (VERY FAST)
3. **Bitmap Index Scan** ⚠️ - Multiple indexes combined
4. **Seq Scan / Table Scan** ❌ - Reading all rows (SLOW)
5. **Nested Loop** ⚠️ - Join algorithm (can be slow for large tables)
6. **Hash Join** ✅ - Better for large joins
7. **Merge Join** ✅ - Best for sorted data

---

## 4. Partitioning - Split Large Tables

### What Is Partitioning?

Splitting one HUGE table into smaller physical pieces. Like organizing files into folders by year.

**Real-Life Analogy:**
Instead of one giant filing cabinet with 10 years of documents, have 10 separate cabinets (one per year).

---

### Types of Partitioning

#### A. Range Partitioning (Most Common)

Split by date/number ranges.

**Example: Orders table with 100 million rows**

**Without partitioning:**
```sql
-- Scans ALL 100 million rows!
SELECT * FROM orders WHERE order_date >= '2024-01-01';
```

**With partitioning:**
```sql
-- Create partitioned table
CREATE TABLE orders (
    order_id BIGINT,
    customer_id INT,
    order_date DATE,
    amount DECIMAL
) PARTITION BY RANGE (order_date);

-- Create partitions (one per year)
CREATE TABLE orders_2022 PARTITION OF orders
    FOR VALUES FROM ('2022-01-01') TO ('2023-01-01');

CREATE TABLE orders_2023 PARTITION OF orders
    FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');

CREATE TABLE orders_2024 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

**Query:**
```sql
SELECT * FROM orders WHERE order_date >= '2024-01-01';
```

**Result:** Database ONLY scans `orders_2024` partition (1/3 of data) - MUCH FASTER!

---

#### B. List Partitioning

Split by specific values (regions, categories).

**Example:**
```sql
CREATE TABLE sales PARTITION BY LIST (region);

CREATE TABLE sales_north PARTITION OF sales FOR VALUES IN ('North', 'Northeast');
CREATE TABLE sales_south PARTITION OF sales FOR VALUES IN ('South', 'Southeast');
CREATE TABLE sales_west PARTITION OF sales FOR VALUES IN ('West', 'Southwest');
```

**Query:**
```sql
SELECT * FROM sales WHERE region = 'North';
-- Only scans sales_north partition
```

---

#### C. Hash Partitioning

Split by hash of column (even distribution).

**Example:**
```sql
CREATE TABLE users PARTITION BY HASH (user_id);

CREATE TABLE users_p0 PARTITION OF users FOR VALUES WITH (MODULUS 4, REMAINDER 0);
CREATE TABLE users_p1 PARTITION OF users FOR VALUES WITH (MODULUS 4, REMAINDER 1);
CREATE TABLE users_p2 PARTITION OF users FOR VALUES WITH (MODULUS 4, REMAINDER 2);
CREATE TABLE users_p3 PARTITION OF users FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```

**When to use:** Evenly distribute data across partitions when no natural date/category.

---

### Partitioning Benefits

✅ **Faster queries**: Scan only relevant partitions
✅ **Faster deletes**: Drop entire partition instead of DELETE (instant)
✅ **Maintenance**: Rebuild index on one partition at a time
✅ **Archival**: Move old partitions to cheaper storage

**Example - Fast delete:**
```sql
-- SLOW (deletes 10 million rows):
DELETE FROM orders WHERE order_date < '2022-01-01';

-- FAST (instant drop):
DROP TABLE orders_2020;
DROP TABLE orders_2021;
```

---

## 5. Materialized View - Pre-Computed Results

### What Is It?

A **view** is a saved query. A **materialized view** is a saved query result (physical table).

**Real-Life Analogy:**
- **View**: Recipe (instructions to make a cake)
- **Materialized View**: Actual cake (already made, ready to eat)

---

### Regular View vs Materialized View

**Regular View:**
```sql
CREATE VIEW sales_summary AS
SELECT
    product_id,
    SUM(quantity) AS total_quantity,
    SUM(amount) AS total_revenue
FROM orders
GROUP BY product_id;

-- Query the view
SELECT * FROM sales_summary WHERE product_id = 100;
```

**What happens:**
1. Database runs the underlying query EVERY time
2. Aggregates data on-the-fly (SLOW for large tables)

---

**Materialized View:**
```sql
CREATE MATERIALIZED VIEW sales_summary AS
SELECT
    product_id,
    SUM(quantity) AS total_quantity,
    SUM(amount) AS total_revenue
FROM orders
GROUP BY product_id;

-- Query the materialized view
SELECT * FROM sales_summary WHERE product_id = 100;
```

**What happens:**
1. Database stores the RESULT as a physical table
2. Query is INSTANT (just reading from table)

---

### Refreshing Materialized Views

**Problem:** Data becomes stale (underlying table changes).

**Solution:** Refresh the view.

```sql
-- Manual refresh
REFRESH MATERIALIZED VIEW sales_summary;

-- Concurrent refresh (doesn't lock the view)
REFRESH MATERIALIZED VIEW CONCURRENTLY sales_summary;
```

**Scheduled refresh (PostgreSQL + cron):**
```sql
-- Create a job to refresh every night at 2 AM
SELECT cron.schedule('refresh-sales-summary', '0 2 * * *',
    'REFRESH MATERIALIZED VIEW sales_summary');
```

---

### When to Use Materialized Views

✅ **Complex aggregations** (GROUP BY, JOINs across multiple tables)
✅ **Queries run frequently** (dashboards, reports)
✅ **Data doesn't need to be real-time** (hourly/daily refresh is OK)

❌ **Real-time data required** (use regular view or direct query)
❌ **Data changes very frequently** (refresh overhead too high)

---

### Example: Dashboard Report

**Scenario:** CEO dashboard shows total revenue by region (runs every 5 seconds, query takes 30 seconds).

**Bad approach:**
```sql
-- This runs EVERY time dashboard refreshes (slow!)
SELECT region, SUM(amount) AS revenue
FROM orders
JOIN customers ON orders.customer_id = customers.id
GROUP BY region;
```

**Good approach:**
```sql
-- Create materialized view (runs once per day)
CREATE MATERIALIZED VIEW dashboard_revenue AS
SELECT region, SUM(amount) AS revenue
FROM orders
JOIN customers ON orders.customer_id = customers.id
GROUP BY region;

-- Dashboard queries this (instant!)
SELECT * FROM dashboard_revenue;

-- Refresh nightly
REFRESH MATERIALIZED VIEW dashboard_revenue;
```

---

## Performance Optimization Checklist

### Before Writing Query:

1. **Understand the data volume**
   - How many rows?
   - How many rows will query return?

2. **Check existing indexes**
   ```sql
   -- PostgreSQL
   SELECT * FROM pg_indexes WHERE tablename = 'employees';

   -- MySQL
   SHOW INDEX FROM employees;
   ```

3. **Use EXPLAIN to see execution plan**
   ```sql
   EXPLAIN SELECT ... ;
   ```

---

### Writing the Query:

✅ **Select only needed columns** (not `SELECT *`)
```sql
-- BAD
SELECT * FROM employees;

-- GOOD
SELECT emp_id, name, salary FROM employees;
```

✅ **Filter early** (WHERE before JOIN)
```sql
-- BAD
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE o.order_date > '2024-01-01';

-- GOOD
SELECT * FROM
(SELECT * FROM orders WHERE order_date > '2024-01-01') o
JOIN customers c ON o.customer_id = c.id;
```

✅ **Avoid functions on indexed columns**
```sql
-- BAD (can't use index on order_date)
SELECT * FROM orders WHERE YEAR(order_date) = 2024;

-- GOOD (can use index)
SELECT * FROM orders WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01';
```

✅ **Use LIMIT for large result sets**
```sql
SELECT * FROM orders ORDER BY order_date DESC LIMIT 100;
```

---

### After Query:

1. **Check execution time**
   ```sql
   EXPLAIN ANALYZE SELECT ... ;
   ```

2. **Look for:**
   - ❌ Seq Scan on large tables → Add index
   - ❌ High cost → Optimize query
   - ❌ Nested Loop on large tables → Use Hash Join

3. **Monitor:**
   - Slow query logs
   - Database metrics (CPU, I/O, memory)

---

## Common Performance Problems & Solutions

### Problem 1: Slow Query on Large Table

**Symptom:** Query takes 30+ seconds

**Diagnosis:**
```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'pending';
```

**Output:** `Seq Scan on orders (actual time=25000 ms)`

**Solution:** Add index
```sql
CREATE INDEX idx_status ON orders(status);
```

---

### Problem 2: JOIN Kills Performance

**Symptom:** Query with JOIN takes forever

**Bad query:**
```sql
SELECT o.*, c.*
FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE o.order_date > '2024-01-01';
```

**Solution 1:** Index foreign key
```sql
CREATE INDEX idx_customer_id ON orders(customer_id);
```

**Solution 2:** Filter before JOIN (CTE)
```sql
WITH recent_orders AS (
    SELECT * FROM orders WHERE order_date > '2024-01-01'
)
SELECT ro.*, c.*
FROM recent_orders ro
JOIN customers c ON ro.customer_id = c.id;
```

---

### Problem 3: Dashboard Timeout

**Symptom:** Complex aggregation query times out

**Bad approach:**
```sql
-- Runs aggregation every time (slow)
SELECT
    region,
    DATE_TRUNC('month', order_date) AS month,
    SUM(amount) AS revenue
FROM orders
GROUP BY region, month;
```

**Solution:** Materialized view
```sql
CREATE MATERIALIZED VIEW monthly_revenue AS
SELECT
    region,
    DATE_TRUNC('month', order_date) AS month,
    SUM(amount) AS revenue
FROM orders
GROUP BY region, month;

-- Fast query
SELECT * FROM monthly_revenue WHERE region = 'North';
```

---

## Interview Tips

### Common Questions:

**Q: "How do you optimize a slow query?"**

**Answer:**
1. Run `EXPLAIN ANALYZE` to see execution plan
2. Look for full table scans → Add indexes
3. Check if query is returning too many rows → Add filters/LIMIT
4. Optimize JOINs → Index foreign keys, use appropriate join type
5. Consider materialized views for complex aggregations
6. For very large tables → Consider partitioning

---

**Q: "When would you NOT use an index?"**

**Answer:**
- Small tables (< 1000 rows)
- Columns updated frequently (index maintenance overhead)
- Low selectivity (query returns > 50% of rows)
- Columns with very few distinct values (use bitmap index instead)

---

**Q: "Explain partitioning and when to use it"**

**Answer:**
Partitioning splits large table into smaller physical pieces.

**Use when:**
- Table has > 10 million rows
- Queries filter by date/region (partition key)
- Need to archive old data (drop partitions)
- Index rebuilds take too long (partition-level maintenance)

**Example:** Orders table partitioned by year - queries for 2024 data only scan 2024 partition.

---

**Q: "View vs Materialized View?"**

**Answer:**
- **View**: Virtual table, runs query every time (slow, always fresh)
- **Materialized View**: Physical table, stores results (fast, needs refresh)

**Use Materialized View when:**
- Complex aggregations (GROUP BY, multiple JOINs)
- Query runs frequently (dashboards)
- Data doesn't need to be real-time (hourly/daily refresh OK)

---

## Quick Reference Summary

### Indexes:
- **B-Tree**: General purpose (ranges, sorting)
- **Hash**: Exact matches only
- **Bitmap**: Low-cardinality columns (data warehouses)
- **Composite**: Multiple columns (order matters!)

### Query Optimization:
1. Use `EXPLAIN` / `EXPLAIN ANALYZE`
2. Add indexes on filter/join columns
3. Avoid `SELECT *`
4. Filter early (WHERE before JOIN)
5. Use LIMIT for large results

### Partitioning:
- **Range**: Date/number ranges
- **List**: Specific values (regions)
- **Hash**: Even distribution

### Materialized Views:
- Pre-computed query results
- Fast reads, needs periodic refresh
- Perfect for dashboards/reports

---

**Remember:**
- **Index** = Speed up reads, slow down writes
- **Partition** = Split large tables
- **Materialized View** = Cache expensive queries
- **EXPLAIN** = Your best debugging tool

---

**Good luck with your interviews!** 🚀
