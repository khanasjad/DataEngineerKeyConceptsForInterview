# SQL CONCEPTS QUICK REFERENCE TABLE
## Complete Revision Guide - All Concepts from 50 Interview Questions

> **Use this for last-minute revision before interviews**

---

## 📋 TABLE OF CONTENTS
1. [JOIN Operations](#join-operations)
2. [Window Functions - Ranking](#window-functions-ranking)
3. [Window Functions - Aggregate](#window-functions-aggregate)
4. [Window Functions - Value/Offset](#window-functions-value)
5. [Aggregate Functions](#aggregate-functions)
6. [Date/Time Functions](#datetime-functions)
7. [CTEs & Subqueries](#ctes-subqueries)
8. [Filtering & Grouping](#filtering-grouping)
9. [Set Operations](#set-operations)
10. [Advanced Concepts](#advanced-concepts)

---

## 🔗 JOIN OPERATIONS {#join-operations}

| Concept | What It Does | Syntax | Example Use Case | Question # |
|---------|-------------|--------|------------------|------------|
| **INNER JOIN** | Returns only matching rows from both tables | `FROM A INNER JOIN B ON A.id = B.id` | Find customers who have placed orders | Q1, Q5 |
| **LEFT JOIN** | All rows from left table + matching from right (NULL if no match) | `FROM A LEFT JOIN B ON A.id = B.id` | Find customers with or without orders | Q3 |
| **RIGHT JOIN** | All rows from right table + matching from left (NULL if no match) | `FROM A RIGHT JOIN B ON A.id = B.id` | All departments, even without employees | Q6 |
| **FULL OUTER JOIN** | All rows from both tables (NULL where no match) | `FROM A FULL OUTER JOIN B ON A.id = B.id` | Show all customers and all orders | Q6 |
| **SELF JOIN** | Join table to itself (using aliases) | `FROM emp e1 JOIN emp e2 ON e1.mgr_id = e2.id` | Find employees earning more than manager | Q2, Q10 |
| **CROSS JOIN** | Cartesian product (every row × every row) | `FROM A CROSS JOIN B` | Generate all product-size combinations | Q8 |
| **Multiple JOINs** | Chain multiple tables | `FROM A JOIN B ON ... JOIN C ON ...` | Customer → Order → Product | Q5 |
| **JOIN with Multiple Conditions** | AND/OR conditions in ON clause | `ON A.dept = B.dept AND A.role = B.role` | Match on multiple columns | Q10 |

---

## 🏆 WINDOW FUNCTIONS - RANKING {#window-functions-ranking}

| Function | What It Does | Handles Ties? | Gaps? | Syntax | Use Case | Question # |
|----------|-------------|---------------|-------|--------|----------|------------|
| **ROW_NUMBER()** | Sequential numbering (1, 2, 3...) | No (arbitrary) | No | `ROW_NUMBER() OVER (ORDER BY salary DESC)` | Pagination, deduplication | Q11, Q12 |
| **RANK()** | Ranking with gaps for ties | Yes (same rank) | Yes | `RANK() OVER (ORDER BY score DESC)` | Competition ranking (1, 2, 2, 4) | Q11 |
| **DENSE_RANK()** | Ranking without gaps | Yes (same rank) | No | `DENSE_RANK() OVER (ORDER BY salary DESC)` | **Find Nth highest** (1, 2, 2, 3) | Q1, Q11 |
| **NTILE(n)** | Divide into n equal buckets | N/A | N/A | `NTILE(4) OVER (ORDER BY revenue DESC)` | Quartiles, percentiles, segmentation | Q18, Q40 |

**Frame Syntax**: `OVER ([PARTITION BY col] [ORDER BY col] [ROWS BETWEEN ...])`

**Key Difference**:
```
Score: 95, 90, 90, 85
ROW_NUMBER:  1, 2, 3, 4  (always unique)
RANK:        1, 2, 2, 4  (same rank, skip next)
DENSE_RANK:  1, 2, 2, 3  (same rank, no skip)
NTILE(2):    1, 1, 2, 2  (divide into groups)
```

---

## 📊 WINDOW FUNCTIONS - AGGREGATE {#window-functions-aggregate}

| Function | What It Does | Without ORDER BY | With ORDER BY | Syntax | Question # |
|----------|-------------|------------------|---------------|--------|------------|
| **SUM() OVER** | Cumulative or total sum | Total for partition | Running total | `SUM(revenue) OVER (ORDER BY date)` | Q13, Q20 |
| **AVG() OVER** | Cumulative or partition average | Average for partition | Running average | `AVG(salary) OVER (PARTITION BY dept)` | Q12, Q14 |
| **COUNT() OVER** | Count rows in window | Count in partition | Cumulative count | `COUNT(*) OVER (PARTITION BY category)` | Q23 |
| **MIN() OVER** | Minimum in window | Min in partition | Min so far | `MIN(price) OVER (ORDER BY date)` | Q21 |
| **MAX() OVER** | Maximum in window | Max in partition | Max so far | `MAX(price) OVER (ORDER BY date)` | Q21 |

**Examples**:
```sql
-- Partition average (same for all rows in group)
AVG(salary) OVER (PARTITION BY department)

-- Running total (cumulative)
SUM(revenue) OVER (ORDER BY date)

-- Moving average (last 7 days)
AVG(value) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)

-- Percentage of total
revenue * 100.0 / SUM(revenue) OVER ()
```

---

## 🔄 WINDOW FUNCTIONS - VALUE/OFFSET {#window-functions-value}

| Function | What It Does | Syntax | Common Use | Question # |
|----------|-------------|--------|------------|------------|
| **LAG(col, n)** | Access previous row (n rows back) | `LAG(revenue, 1) OVER (ORDER BY date)` | **Month-over-month growth**, previous value | Q15, Q39 |
| **LEAD(col, n)** | Access next row (n rows forward) | `LEAD(revenue, 1) OVER (ORDER BY date)` | Next value, forward-looking | Q15 |
| **FIRST_VALUE(col)** | First value in window | `FIRST_VALUE(price) OVER (PARTITION BY product ORDER BY date)` | First order date, benchmark | Q16 |
| **LAST_VALUE(col)** | Last value in window | `LAST_VALUE(price) OVER (... ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)` | Last order date | Q16 |
| **NTH_VALUE(col, n)** | Nth value in window | `NTH_VALUE(salary, 2) OVER (...)` | 2nd highest, 3rd lowest | Q17 |

**⚠️ Important Notes**:
- LAG returns NULL for first row (no previous)
- LAST_VALUE needs explicit frame: `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`
- Default frame with ORDER BY: `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`

**Common Patterns**:
```sql
-- Day-over-day change
revenue - LAG(revenue) OVER (ORDER BY date)

-- Percentage change
(revenue - LAG(revenue) OVER (ORDER BY date)) * 100.0 / LAG(revenue) OVER (ORDER BY date)

-- Days since last event
current_date - LAG(event_date) OVER (PARTITION BY user_id ORDER BY event_date)

-- Compare to first value
price - FIRST_VALUE(price) OVER (PARTITION BY product ORDER BY date)
```

---

## 📈 AGGREGATE FUNCTIONS {#aggregate-functions}

| Function | What It Does | Syntax | Ignores NULL? | Use Case | Question # |
|----------|-------------|--------|---------------|----------|------------|
| **COUNT(*)** | Count all rows (including NULL) | `COUNT(*)` | No | Total row count | Q23 |
| **COUNT(col)** | Count non-NULL values in column | `COUNT(email)` | Yes | Count valid values | Q23 |
| **COUNT(DISTINCT col)** | Count unique non-NULL values | `COUNT(DISTINCT customer_id)` | Yes | Unique customers | Q23 |
| **SUM(col)** | Sum of values | `SUM(revenue)` | Yes | Total revenue | Q21 |
| **AVG(col)** | Average of values | `AVG(salary)` | Yes | Average salary | Q21, Q22 |
| **MIN(col)** | Minimum value | `MIN(price)` | Yes | Lowest price | Q21 |
| **MAX(col)** | Maximum value | `MAX(price)` | Yes | Highest price | Q21 |
| **GROUP_CONCAT (MySQL)** | Concatenate values into string | `GROUP_CONCAT(course SEPARATOR ', ')` | Yes | Combine tags/categories | Q24 |
| **STRING_AGG (PostgreSQL)** | Concatenate values into string | `STRING_AGG(course, ', ' ORDER BY course)` | Yes | Combine tags/categories | Q24 |

**Usage with GROUP BY**:
```sql
SELECT
    department,
    COUNT(*) AS num_employees,
    AVG(salary) AS avg_salary,
    SUM(salary) AS total_payroll,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary
FROM employees
GROUP BY department;
```

**Conditional Aggregation** (Pivot):
```sql
SELECT
    SUM(CASE WHEN year = 2023 THEN revenue ELSE 0 END) AS revenue_2023,
    SUM(CASE WHEN year = 2024 THEN revenue ELSE 0 END) AS revenue_2024
FROM sales;
```

---

## 📅 DATE/TIME FUNCTIONS {#datetime-functions}

### PostgreSQL / MySQL / BigQuery Comparison

| Operation | PostgreSQL | MySQL | BigQuery | Use Case | Question # |
|-----------|------------|-------|----------|----------|------------|
| **Current Date** | `CURRENT_DATE` | `CURDATE()` | `CURRENT_DATE()` | Today's date | Q26 |
| **Current Timestamp** | `NOW()` | `NOW()` | `CURRENT_TIMESTAMP()` | Current date and time | Q26 |
| **Extract Year** | `EXTRACT(YEAR FROM date)` | `YEAR(date)` | `EXTRACT(YEAR FROM date)` | Get year component | Q26 |
| **Extract Month** | `EXTRACT(MONTH FROM date)` | `MONTH(date)` | `EXTRACT(MONTH FROM date)` | Get month component | Q26 |
| **Extract Day** | `EXTRACT(DAY FROM date)` | `DAY(date)` | `EXTRACT(DAY FROM date)` | Get day component | Q26 |
| **Truncate to Month** | `DATE_TRUNC('month', date)` | `DATE_FORMAT(date, '%Y-%m-01')` | `DATE_TRUNC(date, MONTH)` | **First day of month** | Q26, Q30 |
| **Truncate to Day** | `DATE_TRUNC('day', timestamp)` | `DATE(timestamp)` | `DATE(timestamp)` | Remove time component | Q30 |
| **Add Days** | `date + INTERVAL '7 days'` | `DATE_ADD(date, INTERVAL 7 DAY)` | `DATE_ADD(date, INTERVAL 7 DAY)` | Future date | Q26, Q27 |
| **Subtract Days** | `date - INTERVAL '7 days'` | `DATE_SUB(date, INTERVAL 7 DAY)` | `DATE_SUB(date, INTERVAL 7 DAY)` | Past date | Q27 |
| **Date Difference** | `date1 - date2` (returns interval) | `DATEDIFF(date1, date2)` (returns days) | `DATE_DIFF(date1, date2, DAY)` | Days between dates | Q26, Q38 |
| **Timestamp Difference** | `EXTRACT(EPOCH FROM (ts1 - ts2))` | `TIMESTAMPDIFF(MINUTE, ts1, ts2)` | `TIMESTAMP_DIFF(ts1, ts2, MINUTE)` | Minutes between timestamps | Q28 |
| **Day of Week** | `EXTRACT(DOW FROM date)` (0=Sun) | `DAYOFWEEK(date)` (1=Sun) | `EXTRACT(DAYOFWEEK FROM date)` (1=Sun) | Get day of week | Q29 |
| **Format Date** | `TO_CHAR(date, 'YYYY-MM-DD')` | `DATE_FORMAT(date, '%Y-%m-%d')` | `FORMAT_DATE('%Y-%m-%d', date)` | Custom format | Q26 |

### Common Date Patterns

```sql
-- Last 7 days (including today)
WHERE order_date >= CURRENT_DATE - INTERVAL '7 days'

-- Last complete month
WHERE DATE_TRUNC('month', order_date) = DATE_TRUNC('month', CURRENT_DATE - INTERVAL '1 month')

-- Same month last year
WHERE EXTRACT(MONTH FROM order_date) = EXTRACT(MONTH FROM CURRENT_DATE)
  AND EXTRACT(YEAR FROM order_date) = EXTRACT(YEAR FROM CURRENT_DATE - INTERVAL '1 year')

-- Business days (exclude weekends)
WHERE EXTRACT(DOW FROM date) NOT IN (0, 6)  -- Not Sunday or Saturday

-- Time bucketing (15-minute intervals)
DATE_TRUNC('hour', timestamp) + INTERVAL '15 min' * FLOOR(EXTRACT(MINUTE FROM timestamp) / 15)
```

**⚠️ Index Usage**:
```sql
-- BAD (function prevents index usage)
WHERE YEAR(order_date) = 2024

-- GOOD (uses index)
WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01'
```

---

## 🔍 CTEs & SUBQUERIES {#ctes-subqueries}

| Concept | What It Does | Syntax | Advantages | Question # |
|---------|-------------|--------|------------|------------|
| **WITH (CTE)** | Named temporary result set | `WITH cte_name AS (SELECT ...) SELECT ... FROM cte_name` | Readable, reusable, debuggable | Q31, Q33 |
| **Multiple CTEs** | Chain multiple CTEs | `WITH cte1 AS (...), cte2 AS (...) SELECT ...` | Break complex logic into steps | Q33 |
| **Recursive CTE** | Self-referencing CTE (loops) | `WITH RECURSIVE cte AS (base UNION ALL recursive) SELECT ...` | Hierarchies, graphs, series | Q32 |
| **Subquery in SELECT** | Scalar subquery | `SELECT (SELECT ... FROM B WHERE ...) AS col FROM A` | Get single value per row | Q34 |
| **Subquery in FROM** | Derived table | `SELECT * FROM (SELECT ...) AS subq` | Create temporary table | Q12, Q34 |
| **Subquery in WHERE** | Filter based on subquery | `WHERE col IN (SELECT ...)` | Dynamic filtering | Q3 |
| **Correlated Subquery** | References outer query | `WHERE salary > (SELECT AVG(salary) FROM emp e2 WHERE e2.dept = e1.dept)` | Row-by-row comparison | Q34 |
| **EXISTS** | Check if subquery returns rows | `WHERE EXISTS (SELECT 1 FROM orders WHERE ...)` | **Faster than IN**, handles NULL | Q3, Q35 |
| **NOT EXISTS** | Check if no rows returned | `WHERE NOT EXISTS (SELECT 1 FROM ...)` | Find missing records | Q35 |

### CTE vs Subquery

| Aspect | CTE | Subquery |
|--------|-----|----------|
| **Readability** | ✅ High | ❌ Low (nested) |
| **Reusability** | ✅ Can reference multiple times | ❌ Must repeat |
| **Debugging** | ✅ Easy (test each CTE) | ❌ Hard |
| **Performance** | Similar | Similar |

### Recursive CTE Pattern

```sql
WITH RECURSIVE hierarchy AS (
    -- Base case (anchor)
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case
    SELECT e.id, e.name, e.manager_id, h.level + 1
    FROM employees e
    JOIN hierarchy h ON e.manager_id = h.id
)
SELECT * FROM hierarchy;
```

---

## 🔎 FILTERING & GROUPING {#filtering-grouping}

| Concept | What It Does | When Applied | Can Use Aggregates? | Syntax | Question # |
|---------|-------------|--------------|---------------------|--------|------------|
| **WHERE** | Filter individual rows | **Before** GROUP BY | ❌ No | `WHERE salary > 50000` | Q22 |
| **HAVING** | Filter aggregated results | **After** GROUP BY | ✅ Yes | `HAVING COUNT(*) > 10` | Q22 |
| **GROUP BY** | Group rows for aggregation | After WHERE | N/A | `GROUP BY department` | Q21, Q22 |
| **ORDER BY** | Sort result set | Last (after all filtering) | ✅ Yes | `ORDER BY salary DESC` | All |
| **LIMIT** | Restrict number of rows | Very last | N/A | `LIMIT 10` | Q1 |
| **OFFSET** | Skip rows (pagination) | With LIMIT | N/A | `LIMIT 10 OFFSET 20` | Q1 |
| **DISTINCT** | Remove duplicate rows | After SELECT | N/A | `SELECT DISTINCT city` | Q4 |

### Execution Order (IMPORTANT!)

```
1. FROM + JOINs      (Get data from tables)
2. WHERE             (Filter individual rows)
3. GROUP BY          (Group rows)
4. HAVING            (Filter groups)
5. SELECT            (Choose columns)
6. DISTINCT          (Remove duplicates)
7. ORDER BY          (Sort results)
8. LIMIT/OFFSET      (Limit rows)
```

### WHERE vs HAVING

```sql
-- WHERE filters rows BEFORE grouping
SELECT department, AVG(salary)
FROM employees
WHERE salary > 50000  -- Only considers employees earning > 50K
GROUP BY department;

-- HAVING filters AFTER grouping
SELECT department, AVG(salary)
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;  -- Only shows departments with avg > 50K
```

### GROUP BY Patterns

```sql
-- Single column
GROUP BY department

-- Multiple columns
GROUP BY country, category

-- With ROLLUP (subtotals)
GROUP BY ROLLUP(country, category)
-- Returns: country-category totals, country totals, grand total

-- By expression
GROUP BY CASE WHEN salary < 50000 THEN 'Low' ELSE 'High' END
```

---

## ➕ SET OPERATIONS {#set-operations}

| Operation | What It Does | Removes Duplicates? | Performance | Syntax | Question # |
|-----------|-------------|---------------------|-------------|--------|------------|
| **UNION** | Combine results, remove duplicates | ✅ Yes | Slower (sorts) | `SELECT ... UNION SELECT ...` | Q7 |
| **UNION ALL** | Combine results, keep all rows | ❌ No | **Faster** | `SELECT ... UNION ALL SELECT ...` | Q7 |
| **INTERSECT** | Only rows in BOTH queries | ✅ Yes | Medium | `SELECT ... INTERSECT SELECT ...` | - |
| **EXCEPT (or MINUS)** | Rows in first query but NOT second | ✅ Yes | Medium | `SELECT ... EXCEPT SELECT ...` | - |

**Rules**:
- All queries must have **same number of columns**
- Column **data types must match** (or be compatible)
- Column names from **first query** are used

**Example**:
```sql
-- UNION (removes duplicates)
SELECT name FROM customers_usa
UNION
SELECT name FROM customers_canada;
-- If "Bob" in both, appears once

-- UNION ALL (keeps all)
SELECT name FROM customers_usa
UNION ALL
SELECT name FROM customers_canada;
-- If "Bob" in both, appears twice
```

**Best Practice**: Use `UNION ALL` when possible (faster)

---

## 🚀 ADVANCED CONCEPTS {#advanced-concepts}

### CASE Expressions

| Type | Syntax | Use Case | Question # |
|------|--------|----------|------------|
| **Simple CASE** | `CASE column WHEN value THEN result END` | Exact match | Q9, Q25 |
| **Searched CASE** | `CASE WHEN condition THEN result END` | Conditional logic | Q9, Q25 |
| **CASE in SELECT** | `SELECT CASE WHEN ... END` | Categorization, bucketing | Q9 |
| **CASE in WHERE** | `WHERE CASE WHEN ... END = value` | Complex filtering | - |
| **CASE in ORDER BY** | `ORDER BY CASE WHEN ... END` | Custom sorting | - |
| **CASE with Aggregates** | `SUM(CASE WHEN year=2024 THEN revenue ELSE 0 END)` | **Conditional aggregation (pivot)** | Q25 |

```sql
-- Categorization
CASE
    WHEN salary < 50000 THEN 'Low'
    WHEN salary BETWEEN 50000 AND 100000 THEN 'Medium'
    ELSE 'High'
END AS salary_category

-- Pivoting
SUM(CASE WHEN year = 2023 THEN revenue ELSE 0 END) AS revenue_2023,
SUM(CASE WHEN year = 2024 THEN revenue ELSE 0 END) AS revenue_2024
```

---

### PARTITION BY (Window Function Clause)

| Concept | What It Does | Example | Question # |
|---------|-------------|---------|------------|
| **PARTITION BY** | Divides rows into groups for window functions | `PARTITION BY department` | Q12, Q13, Q15 |
| **No PARTITION BY** | Entire result set is one partition | `OVER ()` | Q20 |
| **Multiple Columns** | Partition by multiple columns | `PARTITION BY country, category` | - |

```sql
-- Without PARTITION BY (one big group)
AVG(salary) OVER ()  -- Company average (same for all rows)

-- With PARTITION BY (separate groups)
AVG(salary) OVER (PARTITION BY department)  -- Department average (resets per dept)

-- Running total per group
SUM(revenue) OVER (PARTITION BY product ORDER BY date)  -- Resets for each product
```

---

### Window Frames (ROWS BETWEEN)

| Frame | What It Includes | Syntax | Use Case | Question # |
|-------|------------------|--------|----------|------------|
| **Default (with ORDER BY)** | Start to current row | `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` | Running total | Q13 |
| **Default (no ORDER BY)** | All rows in partition | `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` | Partition total | - |
| **Last N rows** | Current + N preceding | `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` | **7-day moving average** | Q14 |
| **Centered window** | N before + current + N after | `ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING` | Centered average | - |
| **All following** | Current to end | `ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING` | Remaining sum | - |

```sql
-- Running total
SUM(revenue) OVER (ORDER BY date)

-- 7-day moving average
AVG(value) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)

-- Total for entire partition
SUM(revenue) OVER (PARTITION BY product)
```

---

### NULL Handling

| Function | Behavior | Example | Question # |
|----------|----------|---------|------------|
| **IS NULL** | Check if NULL | `WHERE email IS NULL` | Q43 |
| **IS NOT NULL** | Check if not NULL | `WHERE email IS NOT NULL` | Q3 |
| **COALESCE(a, b, c)** | First non-NULL value | `COALESCE(phone, email, 'No contact')` | Q43 |
| **NULLIF(a, b)** | NULL if equal, else a | `NULLIF(denominator, 0)` (avoid division by zero) | Q22 |
| **COUNT(column)** | Excludes NULL | `COUNT(email)` vs `COUNT(*)` | Q23 |

**⚠️ NULL Gotchas**:
```sql
-- IN with NULL breaks
WHERE id NOT IN (SELECT customer_id FROM orders)  -- Returns no rows if NULL exists

-- EXISTS handles NULL correctly
WHERE NOT EXISTS (SELECT 1 FROM orders WHERE customer_id = id)

-- NULL in comparisons
NULL = NULL  → NULL (not TRUE!)
IS NULL     → TRUE
```

---

### Data Quality Patterns

| Check | SQL Pattern | Question # |
|-------|-------------|------------|
| **Find duplicates** | `GROUP BY col HAVING COUNT(*) > 1` | Q4, Q41 |
| **Delete duplicates** | `ROW_NUMBER() OVER (PARTITION BY col ORDER BY id)` then delete row_num > 1 | Q42 |
| **Find missing values** | `WHERE col IS NULL` | Q43 |
| **Orphaned records** | `LEFT JOIN ... WHERE right.id IS NULL` | Q44 |
| **Data inconsistency** | Compare aggregated vs stored totals | Q45 |
| **Invalid data** | `WHERE col NOT LIKE pattern` or range checks | Q43 |

---

### Performance Optimization

| Concept | What It Does | Example | Question # |
|---------|-------------|---------|------------|
| **EXPLAIN** | Show query execution plan | `EXPLAIN SELECT ...` | Q46 |
| **Index** | Speed up lookups | `CREATE INDEX idx_name ON table(col)` | Q47 |
| **Covering Index** | Index includes all needed columns | `CREATE INDEX idx ON table(col1, col2, col3)` | Q47 |
| **Avoid functions on indexed columns** | Prevents index usage | Don't: `WHERE YEAR(date) = 2024`<br>Do: `WHERE date >= '2024-01-01'` | Q48 |
| **EXISTS vs IN** | EXISTS is faster, handles NULL | `WHERE EXISTS (...)` instead of `WHERE IN (...)` | Q35 |
| **JOIN vs Subquery** | JOIN usually faster | Prefer JOIN over correlated subquery | Q34 |
| **Partitioning** | Divide large table into chunks | `PARTITION BY RANGE (YEAR(date))` | Q49 |
| **Materialized View** | Pre-computed results | `CREATE MATERIALIZED VIEW ...` | Q50 |

---

## 📊 QUICK LOOKUP TABLES

### When to Use Which JOIN

| Scenario | Use This JOIN |
|----------|---------------|
| Only matching records | INNER JOIN |
| All from left table + matches from right | LEFT JOIN |
| All from right table + matches from left | RIGHT JOIN (rare, use LEFT instead) |
| Everything from both tables | FULL OUTER JOIN |
| Join table to itself | Self JOIN (with aliases) |
| All possible combinations | CROSS JOIN |
| Find records only in left table | LEFT JOIN ... WHERE right.id IS NULL |

---

### When to Use Which Window Function

| Goal | Function | Example |
|------|----------|---------|
| Unique row numbers | ROW_NUMBER() | Pagination, deduplication |
| Ranking (with gaps) | RANK() | Competition ranking |
| **Find Nth highest** | **DENSE_RANK()** | 2nd highest salary |
| Divide into buckets | NTILE(n) | Quartiles, top 25% |
| Running total | SUM() OVER (ORDER BY ...) | Cumulative revenue |
| Group average | AVG() OVER (PARTITION BY ...) | Department average |
| Moving average | AVG() OVER (... ROWS BETWEEN ...) | 7-day average |
| **Compare with previous** | **LAG()** | Month-over-month growth |
| Compare with next | LEAD() | Forward-looking |
| First in group | FIRST_VALUE() | Initial price |
| Last in group | LAST_VALUE() | Final price |

---

### Common SQL Patterns (Copy-Paste Ready)

#### Find Nth Highest
```sql
SELECT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
) ranked
WHERE rank = 2;
```

#### Top N Per Group
```sql
SELECT *
FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY category ORDER BY revenue DESC) AS rank
    FROM products
) ranked
WHERE rank <= 3;
```

#### Month-over-Month Growth
```sql
SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month,
    (revenue - LAG(revenue) OVER (ORDER BY month)) * 100.0 / LAG(revenue) OVER (ORDER BY month) AS growth_pct
FROM monthly_sales;
```

#### Pivot (Rows to Columns)
```sql
SELECT
    SUM(CASE WHEN year = 2023 THEN revenue ELSE 0 END) AS "2023",
    SUM(CASE WHEN year = 2024 THEN revenue ELSE 0 END) AS "2024"
FROM sales;
```

#### Running Total
```sql
SELECT
    date,
    revenue,
    SUM(revenue) OVER (ORDER BY date) AS running_total
FROM daily_sales;
```

#### 7-Day Moving Average
```sql
SELECT
    date,
    value,
    AVG(value) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS avg_7d
FROM metrics;
```

#### Find Duplicates
```sql
SELECT email, COUNT(*) AS count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

#### Delete Duplicates (Keep First)
```sql
DELETE FROM users
WHERE id NOT IN (
    SELECT MIN(id) FROM users GROUP BY email
);
```

#### Gaps and Islands (Consecutive Dates)
```sql
WITH groups AS (
    SELECT
        date,
        DATE_SUB(date, INTERVAL ROW_NUMBER() OVER (ORDER BY date) DAY) AS grp
    FROM activity
)
SELECT MIN(date) AS start, MAX(date) AS end
FROM groups
GROUP BY grp;
```

---

## 🎯 INTERVIEW QUICK TIPS

### Pattern Recognition

| Question Contains... | Use This |
|---------------------|----------|
| "Find Nth highest/lowest" | DENSE_RANK() WHERE rank = N |
| "Top N per group" | ROW_NUMBER() OVER (PARTITION BY ...) |
| "Month-over-month / day-over-day" | LAG() |
| "Running total / cumulative" | SUM() OVER (ORDER BY ...) |
| "Moving average" | AVG() OVER (... ROWS BETWEEN ...) |
| "Never ordered / no matching" | LEFT JOIN ... WHERE NULL or NOT EXISTS |
| "All combinations" | CROSS JOIN |
| "Duplicates" | GROUP BY ... HAVING COUNT(*) > 1 |
| "Pivot" (rows→columns) | CASE with SUM() |
| "Hierarchy / tree" | Recursive CTE |

---

### Performance Red Flags

| ❌ Avoid | ✅ Use Instead |
|---------|---------------|
| `SELECT *` | Select specific columns |
| `WHERE YEAR(date) = 2024` | `WHERE date >= '2024-01-01' AND date < '2025-01-01'` |
| `NOT IN` with subquery | `NOT EXISTS` or `LEFT JOIN ... WHERE NULL` |
| Correlated subquery | Window function or JOIN |
| `OR` on different columns | `UNION` (if can't use index) |
| Multiple queries | Single query with JOIN |

---

### Before Interview

**Memorize These 5 Patterns**:
1. Top N per group (ROW_NUMBER + PARTITION BY)
2. Nth highest value (DENSE_RANK)
3. Month-over-month growth (LAG)
4. Running total (SUM OVER ORDER BY)
5. Moving average (AVG OVER ROWS BETWEEN)

**Know These Differences**:
- WHERE vs HAVING
- RANK vs DENSE_RANK vs ROW_NUMBER
- INNER JOIN vs LEFT JOIN vs FULL OUTER JOIN
- IN vs EXISTS
- UNION vs UNION ALL
- COUNT(*) vs COUNT(column)

---

## 📚 STUDY PLAN

### Week 1: Foundations
- ✅ All JOINs (Q1-Q10)
- ✅ Basic aggregations (Q21-Q25)
- ✅ WHERE vs HAVING

### Week 2: Window Functions
- ✅ Ranking functions (Q11-Q12)
- ✅ LAG/LEAD (Q15)
- ✅ Running totals (Q13-Q14)

### Week 3: Advanced
- ✅ CTEs (Q31-Q35)
- ✅ Date functions (Q26-Q30)
- ✅ Analytics patterns (Q36-Q40)

### Week 4: Interview Prep
- ✅ Data quality (Q41-Q45)
- ✅ Performance (Q46-Q50)
- ✅ Practice all 50 questions

---

## 🚀 YOU'RE READY!

**Print this guide. Review it before your interview. You have everything you need!**

Quick wins:
- Know your JOINs cold (especially INNER vs LEFT)
- Master these 3 window functions: ROW_NUMBER, DENSE_RANK, LAG
- Understand PARTITION BY (it's just GROUP BY for window functions)
- Remember: WHERE filters rows, HAVING filters groups
- Always ask about database (PostgreSQL vs MySQL vs BigQuery)

**Good luck! 💪**
