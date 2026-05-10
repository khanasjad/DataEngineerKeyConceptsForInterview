# Top 50 SQL Questions for Data Engineer Interviews

**Complete Guide with Detailed Explanations, Examples, and Solutions**

---

## 📋 TABLE OF CONTENTS

1. [Basic SQL & JOINs](#section-1-basic-sql--joins) (Q1-Q10)
2. [Window Functions](#section-2-window-functions) (Q11-Q20)
3. [Aggregations & GROUP BY](#section-3-aggregations--group-by) (Q21-Q25)
4. [Date/Time Manipulation](#section-4-datetime-manipulation) (Q26-Q30)
5. [CTEs & Subqueries](#section-5-ctes--subqueries) (Q31-Q35)
6. [Advanced Analytics](#section-6-advanced-analytics) (Q36-Q40)
7. [Data Quality & Duplicates](#section-7-data-quality--duplicates) (Q41-Q45)
8. [Performance & Optimization](#section-8-performance--optimization) (Q46-Q50)

---

## SECTION 1: Basic SQL & JOINs

### **Q1: Find the Second Highest Salary**

**Problem:**
Write a SQL query to find the second highest salary from an `Employee` table. If there is no second highest salary, return `NULL`.

**Table: employees**
```
+----+--------+
| id | salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
| 4  | 200    |
+----+--------+
```

**Expected Output:**
```
+---------------------+
| second_highest_salary |
+---------------------+
| 200                 |
+---------------------+
```

**Solution 1: Using LIMIT OFFSET**
```sql
SELECT DISTINCT salary AS second_highest_salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

**Solution 2: Using Subquery (Handles NULL case better)**
```sql
SELECT
    (SELECT DISTINCT salary
     FROM employees
     ORDER BY salary DESC
     LIMIT 1 OFFSET 1) AS second_highest_salary;
```

**Solution 3: Using Window Function**
```sql
SELECT DISTINCT salary AS second_highest_salary
FROM (
    SELECT
        salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) as rank
    FROM employees
) ranked
WHERE rank = 2;
```

**Key Concepts:**
- `DISTINCT` to remove duplicate salaries
- `LIMIT/OFFSET` for pagination
- `DENSE_RANK()` vs `RANK()` (DENSE_RANK doesn't skip numbers)
- Subquery to handle NULL case

**Common Variations:**
- Find Nth highest salary
- Find second highest salary per department

---

### **Q2: Self-JOIN - Employee Manager Hierarchy**

**Problem:**
Find employees who earn more than their managers.

**Table: employees**
```
+----+-------+--------+-----------+
| id | name  | salary | manager_id|
+----+-------+--------+-----------+
| 1  | Joe   | 70000  | 3         |
| 2  | Henry | 80000  | 4         |
| 3  | Sam   | 60000  | NULL      |
| 4  | Max   | 90000  | NULL      |
+----+-------+--------+-----------+
```

**Expected Output:**
```
+----------+
| employee |
+----------+
| Joe      |
+----------+
```

**Solution:**
```sql
SELECT
    e1.name AS employee
FROM employees e1
INNER JOIN employees e2
    ON e1.manager_id = e2.id
WHERE e1.salary > e2.salary;
```

**Detailed Explanation:**
1. **Self-JOIN:** Join table to itself using aliases (e1, e2)
2. **e1** represents employees
3. **e2** represents managers
4. **ON clause:** Links employee's manager_id to manager's id
5. **WHERE clause:** Filters employees earning more than their manager

**Step-by-step execution:**
```
Step 1: Join creates pairs
e1.id | e1.name | e1.salary | e1.manager_id | e2.id | e2.name | e2.salary
1     | Joe     | 70000     | 3             | 3     | Sam     | 60000
2     | Henry   | 80000     | 4             | 4     | Max     | 90000

Step 2: Filter where e1.salary > e2.salary
1     | Joe     | 70000     | 3             | 3     | Sam     | 60000  ✓ (70000 > 60000)
2     | Henry   | 80000     | 4             | 4     | Max     | 90000  ✗ (80000 < 90000)

Result: Joe
```

**Key Concepts:**
- Self-JOIN pattern
- Table aliases
- Understanding foreign key relationships

**Common Variations:**
- Find employees who earn the same as their manager
- Find managers who earn less than all their reports
- Calculate average salary difference between manager and reports

---

### **Q3: Customers Who Never Ordered**

**Problem:**
Find all customers who never placed an order.

**Table: customers**
```
+----+-------+
| id | name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
```

**Table: orders**
```
+----+------------+
| id | customer_id|
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+
```

**Expected Output:**
```
+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+
```

**Solution 1: LEFT JOIN (Recommended)**
```sql
SELECT c.name AS Customers
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL;
```

**Solution 2: NOT IN**
```sql
SELECT name AS Customers
FROM customers
WHERE id NOT IN (
    SELECT customer_id
    FROM orders
);
```

**Solution 3: NOT EXISTS (Most performant for large datasets)**
```sql
SELECT name AS Customers
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

**Comparison:**

| Method | Performance | NULL Handling | Readability |
|--------|-------------|---------------|-------------|
| LEFT JOIN | Good | Explicit | High |
| NOT IN | Poor (full scan) | ⚠️ Breaks with NULL | Medium |
| NOT EXISTS | Best | Safe | Medium |

**Key Concepts:**
- LEFT JOIN returns all rows from left table
- `IS NULL` to find unmatched records
- `NOT IN` fails if subquery contains NULL
- `NOT EXISTS` is generally fastest

**Common Variations:**
- Find products never purchased
- Find users who never logged in
- Find employees with no subordinates

---

### **Q4: Duplicate Emails**

**Problem:**
Find all duplicate emails in a `Person` table.

**Table: person**
```
+----+------------------+
| id | email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |
+----+------------------+
```

**Expected Output:**
```
+------------------+
| email            |
+------------------+
| john@example.com |
+------------------+
```

**Solution 1: GROUP BY + HAVING**
```sql
SELECT email
FROM person
GROUP BY email
HAVING COUNT(*) > 1;
```

**Solution 2: Self-JOIN**
```sql
SELECT DISTINCT p1.email
FROM person p1
INNER JOIN person p2
    ON p1.email = p2.email
    AND p1.id != p2.id;
```

**Solution 3: Window Function**
```sql
SELECT DISTINCT email
FROM (
    SELECT
        email,
        COUNT(*) OVER (PARTITION BY email) as email_count
    FROM person
) counts
WHERE email_count > 1;
```

**Performance Comparison:**
- **HAVING:** Most efficient (single table scan + aggregation)
- **Self-JOIN:** Less efficient (cartesian product)
- **Window Function:** Good for additional analytics

**Key Concepts:**
- `GROUP BY` aggregates rows
- `HAVING` filters aggregated results (vs WHERE for individual rows)
- Self-JOIN for finding duplicates
- Window functions for counting without grouping

**Common Variations:**
- Delete duplicate emails (keep lowest ID)
- Find emails with exactly 3 occurrences
- Find duplicates across multiple columns

---

### **Q5: Multiple Table JOIN**

**Problem:**
Write a query to get customer name, order date, and product name for all orders.

**Table: customers**
```
+----+---------+
| id | name    |
+----+---------+
| 1  | Alice   |
| 2  | Bob     |
+----+---------+
```

**Table: orders**
```
+----+-------------+------------+
| id | customer_id | order_date |
+----+-------------+------------+
| 1  | 1           | 2024-01-15 |
| 2  | 2           | 2024-01-16 |
| 3  | 1           | 2024-01-17 |
+----+-------------+------------+
```

**Table: order_items**
```
+----+----------+------------+
| id | order_id | product_id |
+----+----------+------------+
| 1  | 1        | 101        |
| 2  | 1        | 102        |
| 3  | 2        | 101        |
+----+----------+------------+
```

**Table: products**
```
+-----+--------------+
| id  | name         |
+-----+--------------+
| 101 | Widget       |
| 102 | Gadget       |
+-----+--------------+
```

**Solution:**
```sql
SELECT
    c.name AS customer_name,
    o.order_date,
    p.name AS product_name
FROM customers c
INNER JOIN orders o
    ON c.id = o.customer_id
INNER JOIN order_items oi
    ON o.id = oi.order_id
INNER JOIN products p
    ON oi.product_id = p.id
ORDER BY c.name, o.order_date;
```

**Expected Output:**
```
+---------------+------------+--------------+
| customer_name | order_date | product_name |
+---------------+------------+--------------+
| Alice         | 2024-01-15 | Widget       |
| Alice         | 2024-01-15 | Gadget       |
| Alice         | 2024-01-17 | Widget       |
| Bob           | 2024-01-16 | Widget       |
+---------------+------------+--------------+
```

**Key Concepts:**
- Multiple JOINs in sequence
- Foreign key relationships
- Result row explosion (1 order with 2 items = 2 rows)

**Common Variations:**
- Include customers with no orders (LEFT JOIN)
- Calculate total per order
- Find most popular product

---

### **Q6: INNER vs LEFT vs RIGHT vs FULL JOIN**

**Problem:**
Demonstrate all JOIN types with examples.

**Table: employees**
```
+----+-------+--------+
| id | name  | dept_id|
+----+-------+--------+
| 1  | Alice | 10     |
| 2  | Bob   | 20     |
| 3  | Carol | NULL   |
+----+-------+--------+
```

**Table: departments**
```
+----+-------+
| id | name  |
+----+-------+
| 10 | Sales |
| 30 | IT    |
+----+-------+
```

**INNER JOIN (Only matching records)**
```sql
SELECT e.name, d.name AS department
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;
```
**Result:**
```
+-------+------------+
| name  | department |
+-------+------------+
| Alice | Sales      |
+-------+------------+
```
*Bob and Carol excluded (no match). IT department excluded (no employee).*

---

**LEFT JOIN (All from left table)**
```sql
SELECT e.name, d.name AS department
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;
```
**Result:**
```
+-------+------------+
| name  | department |
+-------+------------+
| Alice | Sales      |
| Bob   | NULL       |
| Carol | NULL       |
+-------+------------+
```
*All employees included, even without matching department.*

---

**RIGHT JOIN (All from right table)**
```sql
SELECT e.name, d.name AS department
FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.id;
```
**Result:**
```
+-------+------------+
| name  | department |
+-------+------------+
| Alice | Sales      |
| NULL  | IT         |
+-------+------------+
```
*All departments included, even without employees.*

---

**FULL OUTER JOIN (All from both tables)**
```sql
SELECT e.name, d.name AS department
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.id;
```
**Result:**
```
+-------+------------+
| name  | department |
+-------+------------+
| Alice | Sales      |
| Bob   | NULL       |
| Carol | NULL       |
| NULL  | IT         |
+-------+------------+
```
*Everything included from both tables.*

---

**Visual Representation:**
```
INNER JOIN:     ┌─────┐
                │ ∩   │  (Only overlap)
                └─────┘

LEFT JOIN:      ┌─────┐───┐
                │ All │ ∩ │  (All left + overlap)
                └─────┴───┘

RIGHT JOIN:     ┌───┬─────┐
                │ ∩ │ All │  (Overlap + all right)
                └───┴─────┘

FULL OUTER:     ┌─────┬───┬─────┐
                │ All │ ∩ │ All │  (Everything)
                └─────┴───┴─────┘
```

**Key Decision Guide:**
- **INNER JOIN:** Only need matching records
- **LEFT JOIN:** Need all from first table + matches
- **RIGHT JOIN:** Need all from second table + matches (rare, use LEFT instead)
- **FULL OUTER:** Need everything (rare in analytics)

---

### **Q7: UNION vs UNION ALL**

**Problem:**
Combine results from two queries, with and without duplicates.

**Table: customers_usa**
```
+----+-------+
| id | name  |
+----+-------+
| 1  | Alice |
| 2  | Bob   |
+----+-------+
```

**Table: customers_canada**
```
+----+-------+
| id | name  |
+----+-------+
| 2  | Bob   |
| 3  | Carol |
+----+-------+
```

**UNION (Removes duplicates)**
```sql
SELECT id, name FROM customers_usa
UNION
SELECT id, name FROM customers_canada;
```
**Result:**
```
+----+-------+
| id | name  |
+----+-------+
| 1  | Alice |
| 2  | Bob   |
| 3  | Carol |
+----+-------+
```
*Bob appears once (duplicate removed).*

---

**UNION ALL (Keeps all records)**
```sql
SELECT id, name FROM customers_usa
UNION ALL
SELECT id, name FROM customers_canada;
```
**Result:**
```
+----+-------+
| id | name  |
+----+-------+
| 1  | Alice |
| 2  | Bob   |
| 2  | Bob   |
| 3  | Carol |
+----+-------+
```
*Bob appears twice (keeps duplicates).*

---

**Performance Comparison:**

| Operation | Performance | Use Case |
|-----------|-------------|----------|
| UNION | Slower (sorts + deduplicates) | Need unique records |
| UNION ALL | Faster (no deduplication) | Know no duplicates OR need all records |

**Best Practice:**
- Use `UNION ALL` when possible (faster)
- Use `UNION` only when duplicates are a problem

**Key Concepts:**
- Both queries must have same number of columns
- Column data types must be compatible
- Column names from first query are used

**Common Variations:**
- Combine data from multiple years
- Merge staging tables
- Append incremental data

---

### **Q8: Cross JOIN (Cartesian Product)**

**Problem:**
Generate all possible combinations of products and sizes.

**Table: products**
```
+----+---------+
| id | name    |
+----+---------+
| 1  | T-Shirt |
| 2  | Jeans   |
+----+---------+
```

**Table: sizes**
```
+----+------+
| id | size |
+----+------+
| 1  | S    |
| 2  | M    |
| 3  | L    |
+----+------+
```

**Solution:**
```sql
SELECT
    p.name AS product,
    s.size
FROM products p
CROSS JOIN sizes s;
```

**Result:**
```
+---------+------+
| product | size |
+---------+------+
| T-Shirt | S    |
| T-Shirt | M    |
| T-Shirt | L    |
| Jeans   | S    |
| Jeans   | M    |
| Jeans   | L    |
+---------+------+
```

**Use Cases:**
1. **Generate date spines:**
```sql
SELECT
    d.date,
    u.user_id
FROM dates d
CROSS JOIN users u
-- Creates row for every user for every day
```

2. **Create all possible combinations:**
```sql
SELECT
    t1.option AS option1,
    t2.option AS option2
FROM options t1
CROSS JOIN options t2
WHERE t1.id < t2.id  -- Avoid duplicates like (A,B) and (B,A)
```

**Warning:**
- Result size = rows_table1 × rows_table2
- 1000 × 1000 = 1,000,000 rows!
- Use carefully to avoid performance issues

---

### **Q9: Case Statement in JOIN**

**Problem:**
Categorize employees based on salary ranges and count per category.

**Table: employees**
```
+----+-------+--------+
| id | name  | salary |
+----+-------+--------+
| 1  | Alice | 30000  |
| 2  | Bob   | 60000  |
| 3  | Carol | 90000  |
| 4  | Dave  | 45000  |
+----+-------+--------+
```

**Solution:**
```sql
SELECT
    CASE
        WHEN salary < 40000 THEN 'Low'
        WHEN salary BETWEEN 40000 AND 70000 THEN 'Medium'
        WHEN salary > 70000 THEN 'High'
        ELSE 'Unknown'
    END AS salary_category,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY
    CASE
        WHEN salary < 40000 THEN 'Low'
        WHEN salary BETWEEN 40000 AND 70000 THEN 'Medium'
        WHEN salary > 70000 THEN 'High'
        ELSE 'Unknown'
    END;
```

**Result:**
```
+-----------------+----------------+------------+
| salary_category | employee_count | avg_salary |
+-----------------+----------------+------------+
| Low             | 1              | 30000      |
| Medium          | 2              | 52500      |
| High            | 1              | 90000      |
+-----------------+----------------+------------+
```

**Pivot Example (CASE for pivoting):**
```sql
SELECT
    department,
    SUM(CASE WHEN year = 2023 THEN revenue ELSE 0 END) AS revenue_2023,
    SUM(CASE WHEN year = 2024 THEN revenue ELSE 0 END) AS revenue_2024,
    SUM(CASE WHEN year = 2025 THEN revenue ELSE 0 END) AS revenue_2025
FROM sales
GROUP BY department;
```

**Key Concepts:**
- CASE for conditional logic
- Can use in SELECT, WHERE, GROUP BY, ORDER BY
- Useful for bucketing/categorization

---

### **Q10: Complex JOIN with Multiple Conditions**

**Problem:**
Find employees in same department with same job title but different salaries.

**Table: employees**
```
+----+-------+--------+------+-----------+
| id | name  | dept   | role | salary    |
+----+-------+--------+------+-----------+
| 1  | Alice | Sales  | Rep  | 50000     |
| 2  | Bob   | Sales  | Rep  | 55000     |
| 3  | Carol | Sales  | Mgr  | 80000     |
| 4  | Dave  | IT     | Dev  | 70000     |
+----+-------+--------+------+-----------+
```

**Solution:**
```sql
SELECT
    e1.name AS employee1,
    e2.name AS employee2,
    e1.dept AS department,
    e1.role,
    e1.salary AS salary1,
    e2.salary AS salary2
FROM employees e1
INNER JOIN employees e2
    ON e1.dept = e2.dept          -- Same department
    AND e1.role = e2.role         -- Same role
    AND e1.salary < e2.salary     -- Different salaries (avoid duplicates)
    AND e1.id < e2.id             -- Avoid mirror pairs (Alice-Bob, Bob-Alice)
ORDER BY e1.dept, e1.role;
```

**Result:**
```
+-----------+-----------+------------+------+---------+---------+
| employee1 | employee2 | department | role | salary1 | salary2 |
+-----------+-----------+------------+------+---------+---------+
| Alice     | Bob       | Sales      | Rep  | 50000   | 55000   |
+-----------+-----------+------------+------+---------+---------+
```

**Key Concepts:**
- Multiple conditions in JOIN
- Avoiding duplicate/mirror pairs with `id < id`
- Self-JOIN pattern

---

## SECTION 2: Window Functions

### **Q11: ROW_NUMBER vs RANK vs DENSE_RANK**

**Problem:**
Understand the differences between ranking functions.

**Table: exam_scores**
```
+----+---------+-------+
| id | student | score |
+----+---------+-------+
| 1  | Alice   | 95    |
| 2  | Bob     | 90    |
| 3  | Carol   | 90    |
| 4  | Dave    | 85    |
| 5  | Eve     | 85    |
| 6  | Frank   | 80    |
+----+---------+-------+
```

**Solution:**
```sql
SELECT
    student,
    score,
    ROW_NUMBER() OVER (ORDER BY score DESC) AS row_num,
    RANK() OVER (ORDER BY score DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM exam_scores
ORDER BY score DESC;
```

**Result:**
```
+---------+-------+---------+------+------------+
| student | score | row_num | rank | dense_rank |
+---------+-------+---------+------+------------+
| Alice   | 95    | 1       | 1    | 1          |
| Bob     | 90    | 2       | 2    | 2          |
| Carol   | 90    | 3       | 2    | 2          |
| Dave    | 85    | 4       | 4    | 3          |
| Eve     | 85    | 5       | 4    | 3          |
| Frank   | 80    | 6       | 6    | 4          |
+---------+-------+---------+------+------------+
```

**Comparison:**

| Function | Ties? | Gaps? | Use Case |
|----------|-------|-------|----------|
| ROW_NUMBER | No (arbitrary order) | No | Unique row identifier |
| RANK | Yes (same rank) | Yes (skips ranks) | Competition ranking (1st, 2nd, 2nd, 4th) |
| DENSE_RANK | Yes (same rank) | No (sequential) | Medal system (gold, silver, bronze) |

**Key Differences:**
- **ROW_NUMBER:** Always unique (1, 2, 3, 4, 5, 6)
- **RANK:** Ties get same rank, next rank skips (1, 2, 2, 4, 4, 6)
- **DENSE_RANK:** Ties get same rank, no gaps (1, 2, 2, 3, 3, 4)

**When to Use:**
- **ROW_NUMBER:** Pagination, deduplication (keep first)
- **RANK:** Traditional ranking, "top N" with ties counting as multiple
- **DENSE_RANK:** Category ranking, "top N" where ties count as one

---

### **Q12: PARTITION BY - Ranking Within Groups**

**Problem:**
Find top 2 highest-paid employees in each department.

**Table: employees**
```
+----+-------+--------+--------+
| id | name  | dept   | salary |
+----+-------+--------+--------+
| 1  | Alice | Sales  | 80000  |
| 2  | Bob   | Sales  | 75000  |
| 3  | Carol | Sales  | 70000  |
| 4  | Dave  | IT     | 90000  |
| 5  | Eve   | IT     | 85000  |
| 6  | Frank | IT     | 80000  |
+----+-------+--------+--------+
```

**Solution:**
```sql
SELECT
    dept,
    name,
    salary,
    rank
FROM (
    SELECT
        dept,
        name,
        salary,
        DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC) as rank
    FROM employees
) ranked
WHERE rank <= 2
ORDER BY dept, rank;
```

**Result:**
```
+-------+-------+--------+------+
| dept  | name  | salary | rank |
+-------+-------+--------+------+
| IT    | Dave  | 90000  | 1    |
| IT    | Eve   | 85000  | 2    |
| Sales | Alice | 80000  | 1    |
| Sales | Bob   | 75000  | 2    |
+-------+-------+--------+------+
```

**Key Concepts:**
- `PARTITION BY` resets ranking for each group
- Without PARTITION BY, ranking is across entire table
- Common pattern: Rank within subquery, filter in outer query

**Visualization:**
```
Without PARTITION BY:
Rank | Name  | Dept  | Salary
1    | Dave  | IT    | 90000
2    | Eve   | IT    | 85000
3    | Alice | Sales | 80000
4    | Frank | IT    | 80000
5    | Bob   | Sales | 75000

With PARTITION BY dept:
Dept  | Rank | Name  | Salary
IT    | 1    | Dave  | 90000
IT    | 2    | Eve   | 85000
IT    | 3    | Frank | 80000
Sales | 1    | Alice | 80000
Sales | 2    | Bob   | 75000
```

---

### **Q13: Running Total (Cumulative Sum)**

**Problem:**
Calculate running total of daily revenue.

**Table: daily_revenue**
```
+----+------------+---------+
| id | date       | revenue |
+----+------------+---------+
| 1  | 2024-01-01 | 100     |
| 2  | 2024-01-02 | 150     |
| 3  | 2024-01-03 | 120     |
| 4  | 2024-01-04 | 180     |
+----+------------+---------+
```

**Solution:**
```sql
SELECT
    date,
    revenue,
    SUM(revenue) OVER (ORDER BY date) AS running_total,
    SUM(revenue) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total_explicit
FROM daily_revenue
ORDER BY date;
```

**Result:**
```
+------------+---------+---------------+
| date       | revenue | running_total |
+------------+---------+---------------+
| 2024-01-01 | 100     | 100           |
| 2024-01-02 | 150     | 250           |
| 2024-01-03 | 120     | 370           |
| 2024-01-04 | 180     | 550           |
+------------+---------+---------------+
```

**Frame Clause Explanation:**
```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```
- **UNBOUNDED PRECEDING:** Start of partition
- **CURRENT ROW:** Current row
- Default behavior when using ORDER BY

**Other Frame Options:**
```sql
-- Last 3 rows (including current)
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW

-- Last 3 rows and next 1 row
ROWS BETWEEN 2 PRECEDING AND 1 FOLLOWING

-- All rows in partition
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
```

**Running Total by Department:**
```sql
SELECT
    dept,
    date,
    revenue,
    SUM(revenue) OVER (
        PARTITION BY dept
        ORDER BY date
    ) AS running_total_by_dept
FROM daily_revenue;
```

---

### **Q14: Moving Average**

**Problem:**
Calculate 7-day moving average of daily active users.

**Table: daily_active_users**
```
+------------+------+
| date       | dau  |
+------------+------+
| 2024-01-01 | 100  |
| 2024-01-02 | 120  |
| 2024-01-03 | 110  |
| 2024-01-04 | 130  |
| 2024-01-05 | 125  |
| 2024-01-06 | 115  |
| 2024-01-07 | 140  |
| 2024-01-08 | 135  |
+------------+------+
```

**Solution:**
```sql
SELECT
    date,
    dau,
    AVG(dau) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7day,
    -- Alternative: Minimum 7 days of data
    CASE
        WHEN ROW_NUMBER() OVER (ORDER BY date) >= 7
        THEN AVG(dau) OVER (
            ORDER BY date
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        )
        ELSE NULL
    END AS moving_avg_7day_min
FROM daily_active_users
ORDER BY date;
```

**Result:**
```
+------------+-----+-----------------+
| date       | dau | moving_avg_7day |
+------------+-----+-----------------+
| 2024-01-01 | 100 | 100.00          |
| 2024-01-02 | 120 | 110.00          |
| 2024-01-03 | 110 | 110.00          |
| 2024-01-04 | 130 | 115.00          |
| 2024-01-05 | 125 | 117.00          |
| 2024-01-06 | 115 | 116.67          |
| 2024-01-07 | 140 | 120.00          |
| 2024-01-08 | 135 | 125.00          |
+------------+-----+-----------------+
```

**Key Concepts:**
- Moving average smooths out fluctuations
- Frame includes current row + N preceding rows
- First N-1 rows have fewer data points (partial average)

**Common Variations:**
- 30-day moving average
- Moving median (use PERCENTILE_CONT)
- Exponential moving average (more complex)

---

### **Q15: LAG and LEAD - Accessing Previous/Next Row**

**Problem:**
Calculate day-over-day revenue change.

**Table: daily_revenue**
```
+------------+---------+
| date       | revenue |
+------------+---------+
| 2024-01-01 | 100     |
| 2024-01-02 | 150     |
| 2024-01-03 | 120     |
| 2024-01-04 | 180     |
+------------+---------+
```

**Solution:**
```sql
SELECT
    date,
    revenue,
    LAG(revenue, 1) OVER (ORDER BY date) AS prev_day_revenue,
    revenue - LAG(revenue, 1) OVER (ORDER BY date) AS revenue_change,
    ROUND(
        (revenue - LAG(revenue, 1) OVER (ORDER BY date)) * 100.0 /
        LAG(revenue, 1) OVER (ORDER BY date),
        2
    ) AS pct_change,
    LEAD(revenue, 1) OVER (ORDER BY date) AS next_day_revenue
FROM daily_revenue
ORDER BY date;
```

**Result:**
```
+------------+---------+------------------+----------------+------------+------------------+
| date       | revenue | prev_day_revenue | revenue_change | pct_change | next_day_revenue |
+------------+---------+------------------+----------------+------------+------------------+
| 2024-01-01 | 100     | NULL             | NULL           | NULL       | 150              |
| 2024-01-02 | 150     | 100              | 50             | 50.00      | 120              |
| 2024-01-03 | 120     | 150              | -30            | -20.00     | 180              |
| 2024-01-04 | 180     | 120              | 60             | 50.00      | NULL             |
+------------+---------+------------------+----------------+------------+------------------+
```

**LAG/LEAD Syntax:**
```sql
LAG(column, offset, default_value) OVER (ORDER BY ...)
LEAD(column, offset, default_value) OVER (ORDER BY ...)
```
- **offset:** How many rows back/forward (default = 1)
- **default_value:** Value when no row exists (default = NULL)

**Example with offset and default:**
```sql
SELECT
    date,
    revenue,
    LAG(revenue, 2, 0) OVER (ORDER BY date) AS revenue_2days_ago,
    LEAD(revenue, 1, revenue) OVER (ORDER BY date) AS next_day_or_current
FROM daily_revenue;
```

**Use Cases:**
- Day-over-day, month-over-month comparisons
- Detecting consecutive events
- Gap analysis

---

### **Q16: FIRST_VALUE and LAST_VALUE**

**Problem:**
For each product, show current price and first/last recorded price.

**Table: price_history**
```
+----+------------+------------+-------+
| id | product_id | date       | price |
+----+------------+------------+-------+
| 1  | 101        | 2024-01-01 | 100   |
| 2  | 101        | 2024-01-02 | 105   |
| 3  | 101        | 2024-01-03 | 103   |
| 4  | 102        | 2024-01-01 | 200   |
| 5  | 102        | 2024-01-02 | 195   |
+----+------------+------------+-------+
```

**Solution:**
```sql
SELECT
    product_id,
    date,
    price AS current_price,
    FIRST_VALUE(price) OVER (
        PARTITION BY product_id
        ORDER BY date
    ) AS first_price,
    LAST_VALUE(price) OVER (
        PARTITION BY product_id
        ORDER BY date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_price,
    price - FIRST_VALUE(price) OVER (
        PARTITION BY product_id
        ORDER BY date
    ) AS price_change_from_first
FROM price_history
ORDER BY product_id, date;
```

**Result:**
```
+------------+------------+---------------+-------------+------------+------------------------+
| product_id | date       | current_price | first_price | last_price | price_change_from_first|
+------------+------------+---------------+-------------+------------+------------------------+
| 101        | 2024-01-01 | 100           | 100         | 103        | 0                      |
| 101        | 2024-01-02 | 105           | 100         | 103        | 5                      |
| 101        | 2024-01-03 | 103           | 100         | 103        | 3                      |
| 102        | 2024-01-01 | 200           | 200         | 195        | 0                      |
| 102        | 2024-01-02 | 195           | 200         | 195        | -5                     |
+------------+------------+---------------+-------------+------------+------------------------+
```

**⚠️ LAST_VALUE Gotcha:**
Without `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`, `LAST_VALUE` only looks at current row!

**Wrong (default frame):**
```sql
LAST_VALUE(price) OVER (PARTITION BY product_id ORDER BY date)
-- Default frame: ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
-- Returns current row value, not last in partition!
```

**Correct:**
```sql
LAST_VALUE(price) OVER (
    PARTITION BY product_id
    ORDER BY date
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
)
```

---

### **Q17: NTH_VALUE**

**Problem:**
Get the 3rd highest salary per department.

**Table: employees**
```
+----+-------+--------+--------+
| id | name  | dept   | salary |
+----+-------+--------+--------+
| 1  | Alice | Sales  | 80000  |
| 2  | Bob   | Sales  | 75000  |
| 3  | Carol | Sales  | 70000  |
| 4  | Dave  | Sales  | 65000  |
| 5  | Eve   | IT     | 90000  |
| 6  | Frank | IT     | 85000  |
| 7  | Grace | IT     | 80000  |
+----+-------+--------+--------+
```

**Solution:**
```sql
SELECT DISTINCT
    dept,
    NTH_VALUE(salary, 3) OVER (
        PARTITION BY dept
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS third_highest_salary
FROM employees;
```

**Result:**
```
+-------+-----------------------+
| dept  | third_highest_salary  |
+-------+-----------------------+
| IT    | 80000                 |
| Sales | 70000                 |
+-------+-----------------------+
```

**Alternative using DENSE_RANK:**
```sql
SELECT
    dept,
    salary AS third_highest_salary
FROM (
    SELECT
        dept,
        salary,
        DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC) as rank
    FROM employees
) ranked
WHERE rank = 3;
```

---

### **Q18: NTILE - Divide into Buckets**

**Problem:**
Divide customers into 4 quartiles based on total spending.

**Table: customers**
```
+----+---------+---------+
| id | name    | total   |
+----+---------+---------+
| 1  | Alice   | 10000   |
| 2  | Bob     | 8000    |
| 3  | Carol   | 6000    |
| 4  | Dave    | 4000    |
| 5  | Eve     | 3000    |
| 6  | Frank   | 2000    |
| 7  | Grace   | 1000    |
| 8  | Henry   | 500     |
+----+---------+---------+
```

**Solution:**
```sql
SELECT
    name,
    total,
    NTILE(4) OVER (ORDER BY total DESC) AS quartile,
    CASE NTILE(4) OVER (ORDER BY total DESC)
        WHEN 1 THEN 'VIP'
        WHEN 2 THEN 'High Value'
        WHEN 3 THEN 'Medium Value'
        WHEN 4 THEN 'Low Value'
    END AS customer_segment
FROM customers
ORDER BY total DESC;
```

**Result:**
```
+-------+-------+----------+------------------+
| name  | total | quartile | customer_segment |
+-------+-------+----------+------------------+
| Alice | 10000 | 1        | VIP              |
| Bob   | 8000  | 1        | VIP              |
| Carol | 6000  | 2        | High Value       |
| Dave  | 4000  | 2        | High Value       |
| Eve   | 3000  | 3        | Medium Value     |
| Frank | 2000  | 3        | Medium Value     |
| Grace | 1000  | 4        | Low Value        |
| Henry | 500   | 4        | Low Value        |
+-------+-------+----------+------------------+
```

**Key Concepts:**
- NTILE(n) divides rows into n equal-sized buckets
- If rows don't divide evenly, first buckets get extra row
- Useful for percentiles, customer segmentation

**Use Cases:**
- Customer tiering (top 10%, next 20%, etc.)
- A/B test group assignment
- Performance banding

---

### **Q19: Gaps and Islands Problem**

**Problem:**
Find consecutive date ranges where user was active.

**Table: user_activity**
```
+----+---------+------------+
| id | user_id | login_date |
+----+---------+------------+
| 1  | 101     | 2024-01-01 |
| 2  | 101     | 2024-01-02 |
| 3  | 101     | 2024-01-03 |
| 4  | 101     | 2024-01-05 |  -- Gap
| 5  | 101     | 2024-01-06 |
| 6  | 101     | 2024-01-10 |  -- Gap
+----+---------+------------+
```

**Expected Output:**
```
+---------+------------+----------+
| user_id | start_date | end_date |
+---------+------------+----------+
| 101     | 2024-01-01 | 2024-01-03 |
| 101     | 2024-01-05 | 2024-01-06 |
| 101     | 2024-01-10 | 2024-01-10 |
+---------+------------+----------+
```

**Solution:**
```sql
WITH date_groups AS (
    SELECT
        user_id,
        login_date,
        -- Calculate difference between row number and date
        -- Consecutive dates have same difference
        DATE_SUB(login_date, INTERVAL ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY login_date
        ) DAY) AS group_id
    FROM user_activity
)
SELECT
    user_id,
    MIN(login_date) AS start_date,
    MAX(login_date) AS end_date,
    COUNT(*) AS consecutive_days
FROM date_groups
GROUP BY user_id, group_id
ORDER BY user_id, start_date;
```

**Explanation:**
```
login_date  | row_num | date - row_num | group_id
2024-01-01  | 1       | 2024-01-01 - 1 | 2023-12-31  ← Same group
2024-01-02  | 2       | 2024-01-02 - 2 | 2023-12-31  ← Same group
2024-01-03  | 3       | 2024-01-03 - 3 | 2023-12-31  ← Same group
2024-01-05  | 4       | 2024-01-05 - 4 | 2024-01-01  ← New group (gap!)
2024-01-06  | 5       | 2024-01-06 - 5 | 2024-01-01  ← Same as previous
2024-01-10  | 6       | 2024-01-10 - 6 | 2024-01-04  ← New group (gap!)
```

**Key Concept:**
For consecutive dates, (date - row_number) stays constant!

---

### **Q20: Percent of Total**

**Problem:**
Calculate each product's revenue as percentage of total revenue.

**Table: product_revenue**
```
+------------+---------+
| product_id | revenue |
+------------+---------+
| 101        | 10000   |
| 102        | 15000   |
| 103        | 25000   |
+------------+---------+
```

**Solution:**
```sql
SELECT
    product_id,
    revenue,
    SUM(revenue) OVER () AS total_revenue,
    ROUND(revenue * 100.0 / SUM(revenue) OVER (), 2) AS pct_of_total,
    ROUND(
        SUM(revenue) OVER (ORDER BY revenue DESC) * 100.0 /
        SUM(revenue) OVER (),
        2
    ) AS cumulative_pct
FROM product_revenue
ORDER BY revenue DESC;
```

**Result:**
```
+------------+---------+---------------+--------------+----------------+
| product_id | revenue | total_revenue | pct_of_total | cumulative_pct |
+------------+---------+---------------+--------------+----------------+
| 103        | 25000   | 50000         | 50.00        | 50.00          |
| 102        | 15000   | 50000         | 30.00        | 80.00          |
| 101        | 10000   | 50000         | 20.00        | 100.00         |
+------------+---------+---------------+--------------+----------------+
```

**Use Cases:**
- Pareto analysis (80/20 rule)
- Contribution analysis
- ABC classification

---

## SECTION 3: Aggregations & GROUP BY

### **Q21: GROUP BY with Multiple Columns**

**Problem:**
Calculate total revenue by country and product category.

**Table: sales**
```
+----+---------+----------+---------+
| id | country | category | revenue |
+----+---------+----------+---------+
| 1  | USA     | Books    | 100     |
| 2  | USA     | Books    | 150     |
| 3  | USA     | Electronics | 200  |
| 4  | Canada  | Books    | 80      |
| 5  | Canada  | Electronics | 120  |
+----+---------+----------+---------+
```

**Solution:**
```sql
SELECT
    country,
    category,
    COUNT(*) AS num_sales,
    SUM(revenue) AS total_revenue,
    AVG(revenue) AS avg_revenue,
    MIN(revenue) AS min_revenue,
    MAX(revenue) AS max_revenue
FROM sales
GROUP BY country, category
ORDER BY country, category;
```

**Result:**
```
+---------+-------------+-----------+---------------+-------------+-------------+-------------+
| country | category    | num_sales | total_revenue | avg_revenue | min_revenue | max_revenue |
+---------+-------------+-----------+---------------+-------------+-------------+-------------+
| Canada  | Books       | 1         | 80            | 80.00       | 80          | 80          |
| Canada  | Electronics | 1         | 120           | 120.00      | 120         | 120         |
| USA     | Books       | 2         | 250           | 125.00      | 100         | 150         |
| USA     | Electronics | 1         | 200           | 200.00      | 200         | 200         |
+---------+-------------+-----------+---------------+-------------+-------------+-------------+
```

**With ROLLUP (Subtotals):**
```sql
SELECT
    country,
    category,
    SUM(revenue) AS total_revenue
FROM sales
GROUP BY ROLLUP(country, category);
```

**Result with ROLLUP:**
```
+---------+-------------+---------------+
| country | category    | total_revenue |
+---------+-------------+---------------+
| USA     | Books       | 250           |
| USA     | Electronics | 200           |
| USA     | NULL        | 450           | ← Subtotal for USA
| Canada  | Books       | 80            |
| Canada  | Electronics | 120           |
| Canada  | NULL        | 200           | ← Subtotal for Canada
| NULL    | NULL        | 650           | ← Grand total
+---------+-------------+---------------+
```

---

### **Q22: HAVING vs WHERE**

**Problem:**
Find departments with average salary > 50000 and more than 2 employees.

**Table: employees**
```
+----+-------+--------+--------+
| id | name  | dept   | salary |
+----+-------+--------+--------+
| 1  | Alice | Sales  | 60000  |
| 2  | Bob   | Sales  | 55000  |
| 3  | Carol | Sales  | 50000  |
| 4  | Dave  | IT     | 70000  |
| 5  | Eve   | IT     | 65000  |
| 6  | Frank | HR     | 45000  |
+----+-------+--------+--------+
```

**Solution:**
```sql
SELECT
    dept,
    COUNT(*) AS num_employees,
    AVG(salary) AS avg_salary
FROM employees
WHERE salary > 40000  -- Filter individual rows BEFORE grouping
GROUP BY dept
HAVING COUNT(*) > 2   -- Filter aggregated results AFTER grouping
   AND AVG(salary) > 50000
ORDER BY avg_salary DESC;
```

**Result:**
```
+-------+---------------+------------+
| dept  | num_employees | avg_salary |
+-------+---------------+------------+
| IT    | 2             | 67500      |
| Sales | 3             | 55000      |
+-------+---------------+------------+
```

**Key Differences:**

| Aspect | WHERE | HAVING |
|--------|-------|--------|
| **When Applied** | Before GROUP BY | After GROUP BY |
| **Filters** | Individual rows | Aggregated results |
| **Can Use Aggregates** | No | Yes |
| **Performance** | Better (filters early) | Slower (filters after) |

**Example Showing Difference:**
```sql
-- WHERE filters rows FIRST
SELECT dept, AVG(salary)
FROM employees
WHERE salary > 50000  -- Only considers employees earning > 50K
GROUP BY dept;

-- HAVING filters groups AFTER calculation
SELECT dept, AVG(salary)
FROM employees
GROUP BY dept
HAVING AVG(salary) > 50000;  -- Considers ALL employees, then filters departments
```

---

### **Q23: COUNT(*) vs COUNT(column) vs COUNT(DISTINCT)**

**Problem:**
Understand different COUNT variations.

**Table: orders**
```
+----+-------------+--------+
| id | customer_id | amount |
+----+-------------+--------+
| 1  | 101         | 100    |
| 2  | 101         | NULL   |
| 3  | 102         | 150    |
| 4  | 103         | NULL   |
| 5  | 101         | 200    |
+----+-------------+--------+
```

**Solution:**
```sql
SELECT
    COUNT(*) AS total_rows,
    COUNT(amount) AS non_null_amounts,
    COUNT(DISTINCT customer_id) AS unique_customers,
    COUNT(DISTINCT amount) AS unique_amounts
FROM orders;
```

**Result:**
```
+------------+-------------------+------------------+----------------+
| total_rows | non_null_amounts  | unique_customers | unique_amounts |
+------------+-------------------+------------------+----------------+
| 5          | 3                 | 3                | 3              |
+------------+-------------------+------------------+----------------+
```

**Explanation:**
- **COUNT(*):** Counts all rows (5)
- **COUNT(amount):** Counts non-NULL values in amount column (3)
- **COUNT(DISTINCT customer_id):** Counts unique customer IDs (101, 102, 103 = 3)
- **COUNT(DISTINCT amount):** Counts unique non-NULL amounts (100, 150, 200 = 3)

**Practical Example:**
```sql
SELECT
    customer_id,
    COUNT(*) AS total_orders,
    COUNT(amount) AS orders_with_amount,
    SUM(amount) AS total_spent,
    AVG(amount) AS avg_order_value  -- Only averages non-NULL
FROM orders
GROUP BY customer_id;
```

**Result:**
```
+-------------+--------------+--------------------+-------------+-----------------+
| customer_id | total_orders | orders_with_amount | total_spent | avg_order_value |
+-------------+--------------+--------------------+-------------+-----------------+
| 101         | 3            | 2                  | 300         | 150.00          |
| 102         | 1            | 1                  | 150         | 150.00          |
| 103         | 1            | 0                  | NULL        | NULL            |
+-------------+--------------+--------------------+-------------+-----------------+
```

---

### **Q24: GROUP_CONCAT / STRING_AGG**

**Problem:**
Aggregate multiple rows into comma-separated list.

**Table: student_courses**
```
+------------+--------+
| student_id | course |
+------------+--------+
| 1          | Math   |
| 1          | Science|
| 1          | History|
| 2          | Math   |
| 2          | Art    |
+------------+--------+
```

**Solution (MySQL):**
```sql
SELECT
    student_id,
    GROUP_CONCAT(course ORDER BY course SEPARATOR ', ') AS courses
FROM student_courses
GROUP BY student_id;
```

**Solution (PostgreSQL):**
```sql
SELECT
    student_id,
    STRING_AGG(course, ', ' ORDER BY course) AS courses
FROM student_courses
GROUP BY student_id;
```

**Result:**
```
+------------+------------------------+
| student_id | courses                |
+------------+------------------------+
| 1          | History, Math, Science |
| 2          | Art, Math              |
+------------+------------------------+
```

**Use Cases:**
- Display tags/categories
- Email lists
- Aggregate IDs for IN clause
- Create denormalized reports

**Advanced Example with Counts:**
```sql
SELECT
    student_id,
    COUNT(*) AS num_courses,
    GROUP_CONCAT(
        CONCAT(course, ' (', grade, ')')
        ORDER BY grade DESC
        SEPARATOR ' | '
    ) AS courses_with_grades
FROM student_grades
GROUP BY student_id;
```

---

### **Q25: Conditional Aggregation (CASE with SUM)**

**Problem:**
Pivot data: Show revenue by year in columns.

**Table: sales**
```
+----+------+---------+
| id | year | revenue |
+----+------+---------+
| 1  | 2023 | 100     |
| 2  | 2023 | 150     |
| 3  | 2024 | 200     |
| 4  | 2024 | 180     |
| 5  | 2025 | 220     |
+----+------+---------+
```

**Solution:**
```sql
SELECT
    SUM(CASE WHEN year = 2023 THEN revenue ELSE 0 END) AS revenue_2023,
    SUM(CASE WHEN year = 2024 THEN revenue ELSE 0 END) AS revenue_2024,
    SUM(CASE WHEN year = 2025 THEN revenue ELSE 0 END) AS revenue_2025,
    SUM(revenue) AS total_revenue
FROM sales;
```

**Result:**
```
+--------------+--------------+--------------+---------------+
| revenue_2023 | revenue_2024 | revenue_2025 | total_revenue |
+--------------+--------------+--------------+---------------+
| 250          | 380          | 220          | 850           |
+--------------+--------------+--------------+---------------+
```

**By Category:**
```sql
SELECT
    category,
    SUM(CASE WHEN year = 2023 THEN revenue ELSE 0 END) AS revenue_2023,
    SUM(CASE WHEN year = 2024 THEN revenue ELSE 0 END) AS revenue_2024,
    SUM(CASE WHEN year = 2025 THEN revenue ELSE 0 END) AS revenue_2025
FROM sales
GROUP BY category;
```

**Use Cases:**
- Pivot tables
- Conditional counts (COUNT with CASE)
- Dynamic bucketing

---

## SECTION 4: Date/Time Manipulation

### **Q26: Date Arithmetic**

**Problem:**
Calculate various date operations.

**Table: events**
```
+----+---------------------+
| id | event_timestamp     |
+----+---------------------+
| 1  | 2024-01-15 14:30:00 |
+----+---------------------+
```

**Solution:**
```sql
SELECT
    event_timestamp,

    -- Extract parts
    EXTRACT(YEAR FROM event_timestamp) AS year,
    EXTRACT(MONTH FROM event_timestamp) AS month,
    EXTRACT(DAY FROM event_timestamp) AS day,
    EXTRACT(HOUR FROM event_timestamp) AS hour,

    -- Date arithmetic
    DATE_ADD(event_timestamp, INTERVAL 7 DAY) AS plus_7_days,
    DATE_SUB(event_timestamp, INTERVAL 1 MONTH) AS minus_1_month,

    -- Date truncation
    DATE_TRUNC('day', event_timestamp) AS truncated_to_day,
    DATE_TRUNC('month', event_timestamp) AS truncated_to_month,

    -- Difference
    DATEDIFF(CURRENT_DATE, event_timestamp) AS days_ago,

    -- Day of week
    DAYOFWEEK(event_timestamp) AS day_of_week,  -- 1=Sunday, 7=Saturday
    DAYNAME(event_timestamp) AS day_name,

    -- Age calculation
    TIMESTAMPDIFF(YEAR, event_timestamp, NOW()) AS years_ago

FROM events;
```

**Common Date Functions by Database:**

| Operation | MySQL | PostgreSQL | BigQuery |
|-----------|-------|------------|----------|
| Add days | DATE_ADD(date, INTERVAL 7 DAY) | date + INTERVAL '7 days' | DATE_ADD(date, INTERVAL 7 DAY) |
| Truncate | DATE(timestamp) | DATE_TRUNC('day', timestamp) | DATE_TRUNC(timestamp, DAY) |
| Extract | EXTRACT(YEAR FROM date) | EXTRACT(YEAR FROM date) | EXTRACT(YEAR FROM date) |
| Difference | DATEDIFF(date1, date2) | date1 - date2 | DATE_DIFF(date1, date2, DAY) |

---

### **Q27: Find Records from Last N Days**

**Problem:**
Find orders from last 7 days.

**Table: orders**
```
+----+------------+--------+
| id | order_date | amount |
+----+------------+--------+
| 1  | 2024-05-01 | 100    |
| 2  | 2024-05-05 | 150    |
| 3  | 2024-05-08 | 200    |  ← Today is 2024-05-08
+----+------------+--------+
```

**Solution:**
```sql
-- Last 7 days (including today)
SELECT *
FROM orders
WHERE order_date >= CURRENT_DATE - INTERVAL 7 DAY
  AND order_date < CURRENT_DATE + INTERVAL 1 DAY;

-- Alternative: Last 7 days using DATEDIFF
SELECT *
FROM orders
WHERE DATEDIFF(CURRENT_DATE, order_date) <= 7;

-- Last complete week (Monday to Sunday)
SELECT *
FROM orders
WHERE order_date >= DATE_TRUNC('week', CURRENT_DATE)
  AND order_date < DATE_TRUNC('week', CURRENT_DATE) + INTERVAL 7 DAY;

-- Last calendar month
SELECT *
FROM orders
WHERE EXTRACT(YEAR FROM order_date) = EXTRACT(YEAR FROM CURRENT_DATE - INTERVAL 1 MONTH)
  AND EXTRACT(MONTH FROM order_date) = EXTRACT(MONTH FROM CURRENT_DATE - INTERVAL 1 MONTH);
```

**Performance Tip:**
```sql
-- BAD (function on column prevents index usage)
WHERE DATE(order_timestamp) = '2024-05-08'

-- GOOD (uses index)
WHERE order_timestamp >= '2024-05-08 00:00:00'
  AND order_timestamp < '2024-05-09 00:00:00'
```

---

### **Q28: Sessionization - Group Events into Sessions**

**Problem:**
Group user events into sessions (gap > 30 minutes = new session).

**Table: user_events**
```
+----+---------+---------------------+
| id | user_id | event_timestamp     |
+----+---------+---------------------+
| 1  | 101     | 2024-01-01 10:00:00 |
| 2  | 101     | 2024-01-01 10:15:00 |
| 3  | 101     | 2024-01-01 10:50:00 | ← Gap > 30 min, new session
| 4  | 101     | 2024-01-01 11:00:00 |
+----+---------+---------------------+
```

**Solution:**
```sql
WITH event_gaps AS (
    SELECT
        user_id,
        event_timestamp,
        LAG(event_timestamp) OVER (
            PARTITION BY user_id
            ORDER BY event_timestamp
        ) AS prev_event_time,
        TIMESTAMPDIFF(
            MINUTE,
            LAG(event_timestamp) OVER (
                PARTITION BY user_id
                ORDER BY event_timestamp
            ),
            event_timestamp
        ) AS minutes_since_last_event
    FROM user_events
),
session_starts AS (
    SELECT
        user_id,
        event_timestamp,
        -- New session if gap > 30 min or first event
        CASE
            WHEN prev_event_time IS NULL THEN 1
            WHEN minutes_since_last_event > 30 THEN 1
            ELSE 0
        END AS is_session_start
    FROM event_gaps
),
sessions_numbered AS (
    SELECT
        user_id,
        event_timestamp,
        -- Cumulative sum of session starts = session ID
        SUM(is_session_start) OVER (
            PARTITION BY user_id
            ORDER BY event_timestamp
        ) AS session_id
    FROM session_starts
)
SELECT
    user_id,
    session_id,
    MIN(event_timestamp) AS session_start,
    MAX(event_timestamp) AS session_end,
    COUNT(*) AS events_in_session,
    TIMESTAMPDIFF(
        MINUTE,
        MIN(event_timestamp),
        MAX(event_timestamp)
    ) AS session_duration_minutes
FROM sessions_numbered
GROUP BY user_id, session_id
ORDER BY user_id, session_id;
```

**Result:**
```
+---------+------------+---------------------+---------------------+-------------------+-------------------------+
| user_id | session_id | session_start       | session_end         | events_in_session | session_duration_minutes|
+---------+------------+---------------------+---------------------+-------------------+-------------------------+
| 101     | 1          | 2024-01-01 10:00:00 | 2024-01-01 10:15:00 | 2                 | 15                      |
| 101     | 2          | 2024-01-01 10:50:00 | 2024-01-01 11:00:00 | 2                 | 10                      |
+---------+------------+---------------------+---------------------+-------------------+-------------------------+
```

**Key Concept:**
- Use LAG to find time since last event
- Mark session starts (gap > threshold)
- Use cumulative SUM to create session IDs

---

### **Q29: Business Days Calculation**

**Problem:**
Calculate business days between two dates (exclude weekends).

**Solution:**
```sql
WITH RECURSIVE date_series AS (
    SELECT '2024-01-01' AS date
    UNION ALL
    SELECT DATE_ADD(date, INTERVAL 1 DAY)
    FROM date_series
    WHERE date < '2024-01-15'
)
SELECT
    COUNT(*) AS business_days
FROM date_series
WHERE DAYOFWEEK(date) NOT IN (1, 7);  -- Exclude Sunday(1) and Saturday(7)
```

**Alternative (Formula-based for better performance):**
```sql
SELECT
    start_date,
    end_date,
    DATEDIFF(end_date, start_date) + 1 AS total_days,
    -- Calculate weekends
    FLOOR(DATEDIFF(end_date, start_date) / 7) * 2  -- Complete weeks
    + CASE
        WHEN DAYOFWEEK(start_date) = 1 THEN 1  -- Starts on Sunday
        WHEN DAYOFWEEK(end_date) = 7 THEN 1    -- Ends on Saturday
        WHEN DAYOFWEEK(start_date) > DAYOFWEEK(end_date) THEN 2  -- Crosses weekend
        ELSE 0
    END AS weekend_days,
    -- Business days
    DATEDIFF(end_date, start_date) + 1
    - (FLOOR(DATEDIFF(end_date, start_date) / 7) * 2
       + CASE
           WHEN DAYOFWEEK(start_date) = 1 THEN 1
           WHEN DAYOFWEEK(end_date) = 7 THEN 1
           WHEN DAYOFWEEK(start_date) > DAYOFWEEK(end_date) THEN 2
           ELSE 0
         END) AS business_days
FROM orders;
```

---

### **Q30: Time Bucketing - Aggregate into Intervals**

**Problem:**
Aggregate events into 15-minute intervals.

**Table: events**
```
+----+---------------------+
| id | event_timestamp     |
+----+---------------------+
| 1  | 2024-01-01 10:05:00 |
| 2  | 2024-01-01 10:12:00 |
| 3  | 2024-01-01 10:18:00 |
| 4  | 2024-01-01 10:35:00 |
+----+---------------------+
```

**Solution:**
```sql
SELECT
    -- Round down to nearest 15 minutes
    TIMESTAMP_TRUNC(event_timestamp, MINUTE, 15) AS time_bucket,
    COUNT(*) AS event_count
FROM events
GROUP BY time_bucket
ORDER BY time_bucket;

-- Alternative (manual calculation)
SELECT
    DATE_FORMAT(
        FROM_UNIXTIME(
            FLOOR(UNIX_TIMESTAMP(event_timestamp) / 900) * 900
        ),
        '%Y-%m-%d %H:%i:00'
    ) AS time_bucket,
    COUNT(*) AS event_count
FROM events
GROUP BY time_bucket
ORDER BY time_bucket;
```

**Result:**
```
+---------------------+-------------+
| time_bucket         | event_count |
+---------------------+-------------+
| 2024-01-01 10:00:00 | 2           |
| 2024-01-01 10:15:00 | 1           |
| 2024-01-01 10:30:00 | 1           |
+---------------------+-------------+
```

**Different Intervals:**
```sql
-- Hourly
SELECT DATE_TRUNC('hour', event_timestamp), COUNT(*)
FROM events
GROUP BY DATE_TRUNC('hour', event_timestamp);

-- Daily
SELECT DATE(event_timestamp), COUNT(*)
FROM events
GROUP BY DATE(event_timestamp);

-- Weekly (Monday start)
SELECT DATE_TRUNC('week', event_timestamp), COUNT(*)
FROM events
GROUP BY DATE_TRUNC('week', event_timestamp);
```

---

## SECTION 5: CTEs & Subqueries

### **Q31: Basic CTE (Common Table Expression)**

**Problem:**
Find customers who ordered more than average.

**Table: customers**
```
+----+-------+
| id | name  |
+----+-------+
| 1  | Alice |
| 2  | Bob   |
| 3  | Carol |
+----+-------+
```

**Table: orders**
```
+----+-------------+--------+
| id | customer_id | amount |
+----+-------------+--------+
| 1  | 1           | 100    |
| 2  | 1           | 150    |
| 3  | 2           | 50     |
| 4  | 3           | 200    |
+----+-------------+--------+
```

**Solution with CTE:**
```sql
WITH customer_totals AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY customer_id
),
overall_avg AS (
    SELECT AVG(total_spent) AS avg_spent
    FROM customer_totals
)
SELECT
    c.name,
    ct.total_spent,
    oa.avg_spent
FROM customers c
JOIN customer_totals ct ON c.id = ct.customer_id
CROSS JOIN overall_avg oa
WHERE ct.total_spent > oa.avg_spent
ORDER BY ct.total_spent DESC;
```

**Same Query with Subqueries (Less Readable):**
```sql
SELECT
    c.name,
    (SELECT SUM(amount) FROM orders WHERE customer_id = c.id) AS total_spent
FROM customers c
WHERE (SELECT SUM(amount) FROM orders WHERE customer_id = c.id) >
      (SELECT AVG(total_spent)
       FROM (SELECT SUM(amount) AS total_spent
             FROM orders
             GROUP BY customer_id) subq);
```

**CTE Advantages:**
- ✅ More readable
- ✅ Can reference multiple times
- ✅ Easier to debug
- ✅ Can chain multiple CTEs

---

### **Q32: Recursive CTE - Organizational Hierarchy**

**Problem:**
Show full organizational hierarchy from CEO down.

**Table: employees**
```
+----+---------+------------+
| id | name    | manager_id |
+----+---------+------------+
| 1  | CEO     | NULL       |
| 2  | VP1     | 1          |
| 3  | VP2     | 1          |
| 4  | Dir1    | 2          |
| 5  | Dir2    | 2          |
| 6  | Mgr1    | 4          |
+----+---------+------------+
```

**Solution:**
```sql
WITH RECURSIVE org_tree AS (
    -- Base case: Start with CEO (no manager)
    SELECT
        id,
        name,
        manager_id,
        1 AS level,
        CAST(name AS CHAR(1000)) AS path
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case: Add direct reports
    SELECT
        e.id,
        e.name,
        e.manager_id,
        ot.level + 1,
        CONCAT(ot.path, ' > ', e.name)
    FROM employees e
    INNER JOIN org_tree ot ON e.manager_id = ot.id
)
SELECT
    level,
    REPEAT('  ', level - 1) || name AS indented_name,
    path
FROM org_tree
ORDER BY path;
```

**Result:**
```
+-------+----------------+---------------------------+
| level | indented_name  | path                      |
+-------+----------------+---------------------------+
| 1     | CEO            | CEO                       |
| 2     |   VP1          | CEO > VP1                 |
| 3     |     Dir1       | CEO > VP1 > Dir1          |
| 4     |       Mgr1     | CEO > VP1 > Dir1 > Mgr1   |
| 3     |     Dir2       | CEO > VP1 > Dir2          |
| 2     |   VP2          | CEO > VP2                 |
+-------+----------------+---------------------------+
```

**Other Recursive CTE Use Cases:**
- Bill of materials (parts hierarchy)
- Category trees
- Graph traversal
- Date/number series generation

---

### **Q33: Multiple CTEs Chained**

**Problem:**
Calculate customer lifetime value with multiple steps.

**Solution:**
```sql
WITH first_orders AS (
    -- Step 1: Find first order date per customer
    SELECT
        customer_id,
        MIN(order_date) AS first_order_date
    FROM orders
    GROUP BY customer_id
),
customer_metrics AS (
    -- Step 2: Calculate metrics per customer
    SELECT
        o.customer_id,
        fo.first_order_date,
        COUNT(DISTINCT o.id) AS total_orders,
        SUM(o.amount) AS total_spent,
        AVG(o.amount) AS avg_order_value,
        MAX(o.order_date) AS last_order_date,
        DATEDIFF(MAX(o.order_date), fo.first_order_date) AS customer_lifetime_days
    FROM orders o
    JOIN first_orders fo ON o.customer_id = fo.customer_id
    GROUP BY o.customer_id, fo.first_order_date
),
customer_segments AS (
    -- Step 3: Segment customers
    SELECT
        *,
        CASE
            WHEN total_spent > 1000 THEN 'VIP'
            WHEN total_spent > 500 THEN 'High Value'
            WHEN total_spent > 100 THEN 'Medium Value'
            ELSE 'Low Value'
        END AS customer_segment,
        -- Predicted LTV (simple model)
        avg_order_value * (total_orders * 1.0 / NULLIF(customer_lifetime_days, 0)) * 365 AS predicted_annual_value
    FROM customer_metrics
)
SELECT
    c.name,
    cs.customer_segment,
    cs.total_orders,
    cs.total_spent,
    cs.avg_order_value,
    cs.customer_lifetime_days,
    ROUND(cs.predicted_annual_value, 2) AS predicted_annual_value
FROM customers c
JOIN customer_segments cs ON c.id = cs.customer_id
ORDER BY cs.total_spent DESC;
```

**Benefits of Chaining CTEs:**
- Each step is clear and testable
- Can debug individual CTEs
- Reusable within same query
- Better than nested subqueries

---

### **Q34: Correlated Subquery**

**Problem:**
Find employees earning more than average in their department.

**Table: employees**
```
+----+-------+--------+--------+
| id | name  | dept   | salary |
+----+-------+--------+--------+
| 1  | Alice | Sales  | 80000  |
| 2  | Bob   | Sales  | 60000  |
| 3  | Carol | IT     | 90000  |
| 4  | Dave  | IT     | 70000  |
+----+-------+--------+--------+
```

**Solution (Correlated Subquery):**
```sql
SELECT
    e1.name,
    e1.dept,
    e1.salary,
    (SELECT AVG(salary)
     FROM employees e2
     WHERE e2.dept = e1.dept) AS dept_avg_salary
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.dept = e1.dept
)
ORDER BY e1.dept, e1.salary DESC;
```

**Same Query with Window Function (Better Performance):**
```sql
SELECT
    name,
    dept,
    salary,
    dept_avg_salary
FROM (
    SELECT
        name,
        dept,
        salary,
        AVG(salary) OVER (PARTITION BY dept) AS dept_avg_salary
    FROM employees
) subq
WHERE salary > dept_avg_salary
ORDER BY dept, salary DESC;
```

**Performance Comparison:**

| Method | Performance | Readability |
|--------|-------------|-------------|
| Correlated Subquery | ❌ Slow (runs subquery per row) | Medium |
| Window Function | ✅ Fast (single pass) | High |
| JOIN with aggregated table | Medium | Medium |

---

### **Q35: EXISTS vs IN**

**Problem:**
Find customers who placed orders.

**Table: customers**
```
+----+-------+
| id | name  |
+----+-------+
| 1  | Alice |
| 2  | Bob   |
| 3  | Carol |
+----+-------+
```

**Table: orders**
```
+----+-------------+
| id | customer_id |
+----+-------------+
| 1  | 1           |
| 2  | 1           |
| 3  | 3           |
+----+-------------+
```

**Solution 1: EXISTS (Recommended)**
```sql
SELECT c.name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

**Solution 2: IN**
```sql
SELECT c.name
FROM customers c
WHERE c.id IN (
    SELECT customer_id
    FROM orders
);
```

**Solution 3: JOIN**
```sql
SELECT DISTINCT c.name
FROM customers c
INNER JOIN orders o ON c.id = o.customer_id;
```

**Performance Comparison:**

| Method | Performance | NULL Handling | Early Exit |
|--------|-------------|---------------|------------|
| EXISTS | ✅ Best | ✅ Safe | ✅ Yes (stops at first match) |
| IN | ⚠️ Medium | ❌ Breaks with NULL | ❌ No (evaluates all) |
| JOIN | ✅ Good | ✅ Safe | ❌ No |

**When IN Breaks:**
```sql
-- This returns NO rows if orders.customer_id contains NULL
SELECT c.name
FROM customers c
WHERE c.id NOT IN (
    SELECT customer_id
    FROM orders  -- Contains NULL
);

-- EXISTS handles NULL correctly
SELECT c.name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

**Rule of Thumb:**
- Use **EXISTS** for: "Does a related record exist?"
- Use **IN** for: Small, hardcoded lists (e.g., `WHERE status IN ('active', 'pending')`)
- Use **JOIN** for: Need columns from both tables

---

## SECTION 6: Advanced Analytics

### **Q36: Cohort Analysis - Retention by Month**

**Problem:**
Calculate monthly retention rate for user cohorts.

**Table: users**
```
+----+------------+
| id | signup_date|
+----+------------+
| 1  | 2024-01-15 |
| 2  | 2024-01-20 |
| 3  | 2024-02-10 |
+----+------------+
```

**Table: logins**
```
+----+---------+------------+
| id | user_id | login_date |
+----+---------+------------+
| 1  | 1       | 2024-01-16 |
| 2  | 1       | 2024-02-10 |
| 3  | 1       | 2024-03-05 |
| 4  | 2       | 2024-02-01 |
| 5  | 3       | 2024-03-15 |
+----+---------+------------+
```

**Solution:**
```sql
WITH user_cohorts AS (
    SELECT
        id AS user_id,
        DATE_TRUNC('month', signup_date) AS cohort_month
    FROM users
),
user_activity AS (
    SELECT DISTINCT
        uc.cohort_month,
        DATE_TRUNC('month', l.login_date) AS activity_month,
        l.user_id
    FROM logins l
    JOIN user_cohorts uc ON l.user_id = uc.user_id
),
cohort_size AS (
    SELECT
        cohort_month,
        COUNT(DISTINCT user_id) AS cohort_size
    FROM user_cohorts
    GROUP BY cohort_month
),
cohort_activity AS (
    SELECT
        cohort_month,
        activity_month,
        COUNT(DISTINCT user_id) AS active_users
    FROM user_activity
    GROUP BY cohort_month, activity_month
)
SELECT
    ca.cohort_month,
    ca.activity_month,
    DATE_DIFF('month', ca.cohort_month, ca.activity_month) AS months_since_signup,
    cs.cohort_size,
    ca.active_users,
    ROUND(ca.active_users * 100.0 / cs.cohort_size, 2) AS retention_rate
FROM cohort_activity ca
JOIN cohort_size cs ON ca.cohort_month = cs.cohort_month
ORDER BY ca.cohort_month, ca.activity_month;
```

**Result:**
```
+--------------+----------------+--------------------+-------------+--------------+---------------+
| cohort_month | activity_month | months_since_signup| cohort_size | active_users | retention_rate|
+--------------+----------------+--------------------+-------------+--------------+---------------+
| 2024-01-01   | 2024-01-01     | 0                  | 2           | 2            | 100.00        |
| 2024-01-01   | 2024-02-01     | 1                  | 2           | 2            | 100.00        |
| 2024-01-01   | 2024-03-01     | 2                  | 2           | 1            | 50.00         |
| 2024-02-01   | 2024-02-01     | 0                  | 1           | 0            | 0.00          |
| 2024-02-01   | 2024-03-01     | 1                  | 1           | 1            | 100.00        |
+--------------+----------------+--------------------+-------------+--------------+---------------+
```

**Pivoted Retention Table:**
```sql
WITH ... (previous CTEs)
SELECT
    cohort_month,
    MAX(CASE WHEN months_since_signup = 0 THEN retention_rate END) AS month_0,
    MAX(CASE WHEN months_since_signup = 1 THEN retention_rate END) AS month_1,
    MAX(CASE WHEN months_since_signup = 2 THEN retention_rate END) AS month_2,
    MAX(CASE WHEN months_since_signup = 3 THEN retention_rate END) AS month_3
FROM cohort_activity ca
JOIN cohort_size cs ON ca.cohort_month = cs.cohort_month
GROUP BY cohort_month;
```

---

### **Q37: Funnel Analysis**

**Problem:**
Calculate conversion rates through signup funnel.

**Table: funnel_events**
```
+----+---------+----------------+---------------------+
| id | user_id | event_type     | event_timestamp     |
+----+---------+----------------+---------------------+
| 1  | 1       | page_view      | 2024-01-01 10:00:00 |
| 2  | 1       | signup_start   | 2024-01-01 10:05:00 |
| 3  | 1       | signup_complete| 2024-01-01 10:10:00 |
| 4  | 2       | page_view      | 2024-01-01 11:00:00 |
| 5  | 2       | signup_start   | 2024-01-01 11:05:00 |
| 6  | 3       | page_view      | 2024-01-01 12:00:00 |
+----+---------+----------------+---------------------+
```

**Solution:**
```sql
WITH funnel_steps AS (
    SELECT
        user_id,
        MAX(CASE WHEN event_type = 'page_view' THEN 1 ELSE 0 END) AS reached_step1,
        MAX(CASE WHEN event_type = 'signup_start' THEN 1 ELSE 0 END) AS reached_step2,
        MAX(CASE WHEN event_type = 'signup_complete' THEN 1 ELSE 0 END) AS reached_step3
    FROM funnel_events
    GROUP BY user_id
)
SELECT
    'Step 1: Page View' AS step,
    SUM(reached_step1) AS users,
    100.0 AS conversion_from_previous,
    100.0 AS conversion_from_start
FROM funnel_steps

UNION ALL

SELECT
    'Step 2: Signup Start',
    SUM(reached_step2),
    SUM(reached_step2) * 100.0 / NULLIF(SUM(reached_step1), 0),
    SUM(reached_step2) * 100.0 / NULLIF(SUM(reached_step1), 0)
FROM funnel_steps

UNION ALL

SELECT
    'Step 3: Signup Complete',
    SUM(reached_step3),
    SUM(reached_step3) * 100.0 / NULLIF(SUM(reached_step2), 0),
    SUM(reached_step3) * 100.0 / NULLIF(SUM(reached_step1), 0)
FROM funnel_steps;
```

**Result:**
```
+------------------------+-------+-------------------------+-----------------------+
| step                   | users | conversion_from_previous| conversion_from_start |
+------------------------+-------+-------------------------+-----------------------+
| Step 1: Page View      | 3     | 100.00                  | 100.00                |
| Step 2: Signup Start   | 2     | 66.67                   | 66.67                 |
| Step 3: Signup Complete| 1     | 50.00                   | 33.33                 |
+------------------------+-------+-------------------------+-----------------------+
```

**Key Insights:**
- 33% drop-off from page view to signup start
- 50% drop-off from signup start to complete
- Overall conversion: 33.33%

---

### **Q38: Time Between Events**

**Problem:**
Calculate average time between order and shipment.

**Table: orders**
```
+----+-------------+------------+
| id | customer_id | order_date |
+----+-------------+------------+
| 1  | 101         | 2024-01-01 |
| 2  | 101         | 2024-01-10 |
| 3  | 102         | 2024-01-05 |
+----+-------------+------------+
```

**Table: shipments**
```
+----+----------+--------------+
| id | order_id | shipment_date|
+----+----------+--------------+
| 1  | 1        | 2024-01-03   |
| 2  | 2        | 2024-01-12   |
| 3  | 3        | 2024-01-06   |
+----+----------+--------------+
```

**Solution:**
```sql
SELECT
    o.id AS order_id,
    o.customer_id,
    o.order_date,
    s.shipment_date,
    DATEDIFF(s.shipment_date, o.order_date) AS days_to_ship,
    AVG(DATEDIFF(s.shipment_date, o.order_date)) OVER () AS avg_days_to_ship,
    CASE
        WHEN DATEDIFF(s.shipment_date, o.order_date) <= 2 THEN 'Fast'
        WHEN DATEDIFF(s.shipment_date, o.order_date) <= 5 THEN 'Normal'
        ELSE 'Slow'
    END AS shipping_speed
FROM orders o
LEFT JOIN shipments s ON o.id = s.order_id;
```

**Aggregate Statistics:**
```sql
SELECT
    AVG(DATEDIFF(s.shipment_date, o.order_date)) AS avg_days,
    MIN(DATEDIFF(s.shipment_date, o.order_date)) AS min_days,
    MAX(DATEDIFF(s.shipment_date, o.order_date)) AS max_days,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY DATEDIFF(s.shipment_date, o.order_date)) AS median_days,
    PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY DATEDIFF(s.shipment_date, o.order_date)) AS p95_days
FROM orders o
JOIN shipments s ON o.id = s.order_id;
```

---

### **Q39: Year-over-Year Growth**

**Problem:**
Calculate YoY revenue growth.

**Table: monthly_revenue**
```
+------+-------+---------+
| year | month | revenue |
+------+-------+---------+
| 2023 | 1     | 10000   |
| 2023 | 2     | 12000   |
| 2024 | 1     | 15000   |
| 2024 | 2     | 14000   |
| 2025 | 1     | 18000   |
+------+-------+---------+
```

**Solution:**
```sql
SELECT
    year,
    month,
    revenue,
    LAG(revenue, 12) OVER (ORDER BY year, month) AS revenue_last_year,
    revenue - LAG(revenue, 12) OVER (ORDER BY year, month) AS yoy_change,
    ROUND(
        (revenue - LAG(revenue, 12) OVER (ORDER BY year, month)) * 100.0 /
        LAG(revenue, 12) OVER (ORDER BY year, month),
        2
    ) AS yoy_growth_pct
FROM monthly_revenue
ORDER BY year, month;
```

**Result:**
```
+------+-------+---------+------------------+------------+----------------+
| year | month | revenue | revenue_last_year| yoy_change | yoy_growth_pct |
+------+-------+---------+------------------+------------+----------------+
| 2023 | 1     | 10000   | NULL             | NULL       | NULL           |
| 2023 | 2     | 12000   | NULL             | NULL       | NULL           |
| 2024 | 1     | 15000   | 10000            | 5000       | 50.00          |
| 2024 | 2     | 14000   | 12000            | 2000       | 16.67          |
| 2025 | 1     | 18000   | 15000            | 3000       | 20.00          |
+------+-------+---------+------------------+------------+----------------+
```

**Month-over-Month (MoM):**
```sql
SELECT
    year,
    month,
    revenue,
    LAG(revenue, 1) OVER (ORDER BY year, month) AS revenue_last_month,
    ROUND(
        (revenue - LAG(revenue, 1) OVER (ORDER BY year, month)) * 100.0 /
        LAG(revenue, 1) OVER (ORDER BY year, month),
        2
    ) AS mom_growth_pct
FROM monthly_revenue;
```

---

### **Q40: Pareto Analysis (80/20 Rule)**

**Problem:**
Find products that make up 80% of revenue.

**Table: product_revenue**
```
+------------+---------+
| product_id | revenue |
+------------+---------+
| 101        | 50000   |
| 102        | 30000   |
| 103        | 10000   |
| 104        | 5000    |
| 105        | 3000    |
| 106        | 2000    |
+------------+---------+
```

**Solution:**
```sql
WITH ranked_products AS (
    SELECT
        product_id,
        revenue,
        SUM(revenue) OVER () AS total_revenue,
        SUM(revenue) OVER (ORDER BY revenue DESC) AS cumulative_revenue,
        ROW_NUMBER() OVER (ORDER BY revenue DESC) AS product_rank,
        COUNT(*) OVER () AS total_products
    FROM product_revenue
),
cumulative_pct AS (
    SELECT
        product_id,
        revenue,
        product_rank,
        total_products,
        ROUND(revenue * 100.0 / total_revenue, 2) AS revenue_pct,
        ROUND(cumulative_revenue * 100.0 / total_revenue, 2) AS cumulative_pct,
        ROUND(product_rank * 100.0 / total_products, 2) AS product_pct_rank
    FROM ranked_products
)
SELECT
    product_id,
    revenue,
    revenue_pct,
    cumulative_pct,
    product_pct_rank,
    CASE
        WHEN cumulative_pct <= 80 THEN 'A (Top 80%)'
        WHEN cumulative_pct <= 95 THEN 'B (Next 15%)'
        ELSE 'C (Bottom 5%)'
    END AS abc_category
FROM cumulative_pct
ORDER BY revenue DESC;
```

**Result:**
```
+------------+---------+-------------+----------------+------------------+--------------+
| product_id | revenue | revenue_pct | cumulative_pct | product_pct_rank | abc_category |
+------------+---------+-------------+----------------+------------------+--------------+
| 101        | 50000   | 50.00       | 50.00          | 16.67            | A (Top 80%)  |
| 102        | 30000   | 30.00       | 80.00          | 33.33            | A (Top 80%)  |
| 103        | 10000   | 10.00       | 90.00          | 50.00            | B (Next 15%) |
| 104        | 5000    | 5.00        | 95.00          | 66.67            | B (Next 15%) |
| 105        | 3000    | 3.00        | 98.00          | 83.33            | C (Bottom 5%)|
| 106        | 2000    | 2.00        | 100.00         | 100.00           | C (Bottom 5%)|
+------------+---------+-------------+----------------+------------------+--------------+
```

**Insight:**
- 33% of products (2 out of 6) generate 80% of revenue
- Focus inventory/marketing on top performers

---

## SECTION 7: Data Quality & Duplicates

### **Q41: Find Duplicate Records**

**Problem:**
Find all duplicate email addresses and show all occurrences.

**Table: users**
```
+----+-------+------------------+
| id | name  | email            |
+----+-------+------------------+
| 1  | Alice | alice@email.com  |
| 2  | Bob   | bob@email.com    |
| 3  | Carol | alice@email.com  |
| 4  | Dave  | dave@email.com   |
| 5  | Eve   | bob@email.com    |
+----+-------+------------------+
```

**Solution 1: Show all duplicate records**
```sql
SELECT u.*
FROM users u
INNER JOIN (
    SELECT email
    FROM users
    GROUP BY email
    HAVING COUNT(*) > 1
) duplicates ON u.email = duplicates.email
ORDER BY u.email, u.id;
```

**Solution 2: Using window function**
```sql
SELECT *
FROM (
    SELECT
        *,
        COUNT(*) OVER (PARTITION BY email) AS email_count
    FROM users
) counted
WHERE email_count > 1
ORDER BY email, id;
```

**Result:**
```
+----+-------+------------------+
| id | name  | email            |
+----+-------+------------------+
| 1  | Alice | alice@email.com  |
| 3  | Carol | alice@email.com  |
| 2  | Bob   | bob@email.com    |
| 5  | Eve   | bob@email.com    |
+----+-------+------------------+
```

---

### **Q42: Delete Duplicates (Keep First)**

**Problem:**
Delete duplicate emails, keeping only the record with the lowest ID.

**Table: users** (same as above)

**Solution (PostgreSQL/MySQL):**
```sql
DELETE u1
FROM users u1
INNER JOIN users u2
    ON u1.email = u2.email
    AND u1.id > u2.id;
```

**Solution using CTE with ROW_NUMBER:**
```sql
-- Step 1: Identify duplicates to delete
WITH duplicates_to_delete AS (
    SELECT
        id,
        ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS row_num
    FROM users
)
DELETE FROM users
WHERE id IN (
    SELECT id
    FROM duplicates_to_delete
    WHERE row_num > 1
);
```

**Safe way - Create clean table:**
```sql
-- Step 1: Create table with unique records
CREATE TABLE users_clean AS
SELECT *
FROM (
    SELECT
        *,
        ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS row_num
    FROM users
) ranked
WHERE row_num = 1;

-- Step 2: Verify counts
SELECT COUNT(*) FROM users;       -- Original count
SELECT COUNT(*) FROM users_clean; -- After dedup

-- Step 3: Replace original table
DROP TABLE users;
ALTER TABLE users_clean RENAME TO users;
```

---

### **Q43: Find Missing Values / Data Quality Check**

**Problem:**
Identify data quality issues in customers table.

**Table: customers**
```
+----+-------+------------------+-------+
| id | name  | email            | age   |
+----+-------+------------------+-------+
| 1  | Alice | alice@email.com  | 30    |
| 2  | Bob   | NULL             | 25    |
| 3  | Carol | carol@email.com  | NULL  |
| 4  | NULL  | dave@email.com   | 35    |
| 5  | Eve   | invalid-email    | -5    |
+----+-------+------------------+-------+
```

**Solution:**
```sql
SELECT
    COUNT(*) AS total_records,

    -- NULL checks
    COUNT(*) - COUNT(name) AS missing_name,
    COUNT(*) - COUNT(email) AS missing_email,
    COUNT(*) - COUNT(age) AS missing_age,

    -- Percentage missing
    ROUND((COUNT(*) - COUNT(name)) * 100.0 / COUNT(*), 2) AS pct_missing_name,
    ROUND((COUNT(*) - COUNT(email)) * 100.0 / COUNT(*), 2) AS pct_missing_email,
    ROUND((COUNT(*) - COUNT(age)) * 100.0 / COUNT(*), 2) AS pct_missing_age,

    -- Invalid data
    SUM(CASE WHEN email NOT LIKE '%@%' THEN 1 ELSE 0 END) AS invalid_email_format,
    SUM(CASE WHEN age < 0 OR age > 120 THEN 1 ELSE 0 END) AS invalid_age,

    -- Duplicates
    COUNT(*) - COUNT(DISTINCT email) AS duplicate_emails
FROM customers;
```

**Detailed quality report:**
```sql
SELECT
    'Missing name' AS issue_type,
    COUNT(*) AS issue_count,
    GROUP_CONCAT(id) AS affected_ids
FROM customers
WHERE name IS NULL

UNION ALL

SELECT
    'Missing email',
    COUNT(*),
    GROUP_CONCAT(id)
FROM customers
WHERE email IS NULL

UNION ALL

SELECT
    'Invalid email format',
    COUNT(*),
    GROUP_CONCAT(id)
FROM customers
WHERE email NOT LIKE '%@%'

UNION ALL

SELECT
    'Invalid age',
    COUNT(*),
    GROUP_CONCAT(id)
FROM customers
WHERE age < 0 OR age > 120 OR age IS NULL;
```

---

### **Q44: Data Validation - Referential Integrity**

**Problem:**
Find orphaned records (orders without corresponding customer).

**Table: customers**
```
+----+-------+
| id | name  |
+----+-------+
| 1  | Alice |
| 2  | Bob   |
+----+-------+
```

**Table: orders**
```
+----+-------------+--------+
| id | customer_id | amount |
+----+-------------+--------+
| 1  | 1           | 100    |
| 2  | 3           | 150    | ← Orphaned! Customer 3 doesn't exist
| 3  | 2           | 200    |
| 4  | NULL        | 50     | ← Missing customer_id
+----+-------------+--------+
```

**Solution:**
```sql
-- Find orphaned orders
SELECT
    o.id AS order_id,
    o.customer_id,
    o.amount,
    'Customer not found' AS issue
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.id
WHERE c.id IS NULL
  AND o.customer_id IS NOT NULL

UNION ALL

-- Find orders with NULL customer_id
SELECT
    o.id,
    o.customer_id,
    o.amount,
    'NULL customer_id' AS issue
FROM orders o
WHERE o.customer_id IS NULL;
```

**Result:**
```
+----------+-------------+--------+--------------------+
| order_id | customer_id | amount | issue              |
+----------+-------------+--------+--------------------+
| 2        | 3           | 150    | Customer not found |
| 4        | NULL        | 50     | NULL customer_id   |
+----------+-------------+--------+--------------------+
```

**Comprehensive Referential Integrity Check:**
```sql
-- Check all foreign key relationships
SELECT
    'orders' AS table_name,
    'customer_id' AS foreign_key,
    COUNT(*) AS orphaned_records
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.id
WHERE c.id IS NULL AND o.customer_id IS NOT NULL

UNION ALL

SELECT
    'order_items',
    'order_id',
    COUNT(*)
FROM order_items oi
LEFT JOIN orders o ON oi.order_id = o.id
WHERE o.id IS NULL AND oi.order_id IS NOT NULL

UNION ALL

SELECT
    'order_items',
    'product_id',
    COUNT(*)
FROM order_items oi
LEFT JOIN products p ON oi.product_id = p.id
WHERE p.id IS NULL AND oi.product_id IS NOT NULL;
```

---

### **Q45: Find Records with Inconsistent Data**

**Problem:**
Find orders where line items don't sum to order total.

**Table: orders**
```
+----+-------------+--------+
| id | customer_id | total  |
+----+-------------+--------+
| 1  | 101         | 100    |
| 2  | 102         | 200    |
+----+-------------+--------+
```

**Table: order_items**
```
+----+----------+-------+
| id | order_id | amount|
+----+----------+-------+
| 1  | 1        | 50    |
| 2  | 1        | 50    | ← Correct: 50 + 50 = 100
| 3  | 2        | 80    |
| 4  | 2        | 100   | ← Incorrect: 80 + 100 = 180 ≠ 200
+----+----------+-------+
```

**Solution:**
```sql
WITH order_calculations AS (
    SELECT
        o.id AS order_id,
        o.total AS recorded_total,
        COALESCE(SUM(oi.amount), 0) AS calculated_total,
        o.total - COALESCE(SUM(oi.amount), 0) AS difference
    FROM orders o
    LEFT JOIN order_items oi ON o.id = oi.order_id
    GROUP BY o.id, o.total
)
SELECT
    order_id,
    recorded_total,
    calculated_total,
    difference,
    CASE
        WHEN ABS(difference) < 0.01 THEN 'OK'
        WHEN difference > 0 THEN 'Undercharged'
        ELSE 'Overcharged'
    END AS status
FROM order_calculations
WHERE ABS(difference) >= 0.01  -- Allow for rounding errors
ORDER BY ABS(difference) DESC;
```

**Result:**
```
+----------+----------------+------------------+------------+--------------+
| order_id | recorded_total | calculated_total | difference | status       |
+----------+----------------+------------------+------------+--------------+
| 2        | 200            | 180              | 20         | Undercharged |
+----------+----------------+------------------+------------+--------------+
```

---

## SECTION 8: Performance & Optimization

### **Q46: Explain Plan Analysis**

**Problem:**
Understand query execution plan to identify bottlenecks.

**Example Query:**
```sql
SELECT c.name, SUM(o.amount) AS total
FROM customers c
JOIN orders o ON c.id = o.customer_id
WHERE o.order_date >= '2024-01-01'
GROUP BY c.id, c.name
HAVING SUM(o.amount) > 1000;
```

**Get Execution Plan:**
```sql
-- PostgreSQL
EXPLAIN ANALYZE
SELECT ...;

-- MySQL
EXPLAIN
SELECT ...;

-- BigQuery
-- Automatically shows execution details after query
```

**Sample EXPLAIN output:**
```
+----+-------------+-------+------+---------------+---------+---------+-------+------+-------------+
| id | select_type | table | type | possible_keys | key     | key_len | ref   | rows | Extra       |
+----+-------------+-------+------+---------------+---------+---------+-------+------+-------------+
|  1 | SIMPLE      | o     | ALL  | NULL          | NULL    | NULL    | NULL  | 1000 | Using where |
|  1 | SIMPLE      | c     | ref  | PRIMARY       | PRIMARY | 4       | o.cid | 1    | Using index |
+----+-------------+-------+------+---------------+---------+---------+-------+------+-------------+
```

**Key Indicators:**

| Indicator | Bad Sign | Good Sign |
|-----------|----------|-----------|
| **type** | ALL (full table scan) | ref, eq_ref, const |
| **key** | NULL (no index used) | Index name shown |
| **rows** | Large number | Small number |
| **Extra** | Using filesort, Using temporary | Using index, Using where |

**Common Problems & Fixes:**

**1. Full Table Scan (type: ALL)**
```sql
-- Problem
EXPLAIN SELECT * FROM orders WHERE YEAR(order_date) = 2024;

-- Fix: Don't use function on indexed column
EXPLAIN SELECT * FROM orders
WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01';
```

**2. Missing Index**
```sql
-- Problem: Slow JOIN
EXPLAIN SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.id;
-- Shows: type=ALL on customers

-- Fix: Add index
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
```

**3. Using filesort**
```sql
-- Problem
EXPLAIN SELECT * FROM orders ORDER BY created_at;
-- Extra: Using filesort

-- Fix: Add index on ORDER BY column
CREATE INDEX idx_orders_created_at ON orders(created_at);
```

---

### **Q47: Index Optimization**

**Problem:**
Choose right indexes for common queries.

**Scenario: E-commerce orders table**
```sql
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    status VARCHAR(20),
    total_amount DECIMAL(10,2)
);
```

**Common Queries:**
```sql
-- Q1: Find orders by customer
SELECT * FROM orders WHERE customer_id = 123;

-- Q2: Find recent orders
SELECT * FROM orders WHERE order_date >= '2024-01-01';

-- Q3: Find customer's pending orders
SELECT * FROM orders
WHERE customer_id = 123 AND status = 'pending';

-- Q4: Get order statistics by date range
SELECT COUNT(*), SUM(total_amount)
FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-01-31';
```

**Optimal Indexes:**
```sql
-- For Q1: Single column index
CREATE INDEX idx_customer_id ON orders(customer_id);

-- For Q2: Single column index
CREATE INDEX idx_order_date ON orders(order_date);

-- For Q3: Composite index (most specific first)
CREATE INDEX idx_customer_status ON orders(customer_id, status);

-- For Q4: Covering index (includes all needed columns)
CREATE INDEX idx_date_amount ON orders(order_date, total_amount);
```

**Index Design Principles:**

**1. Composite Index Column Order:**
```sql
-- GOOD: High cardinality first, low cardinality second
CREATE INDEX idx_customer_status ON orders(customer_id, status);
-- customer_id: millions of values (high cardinality)
-- status: ~5 values (low cardinality)

-- BAD: Low cardinality first
CREATE INDEX idx_status_customer ON orders(status, customer_id);
-- Less efficient for finding specific customer
```

**2. Covering Indexes:**
```sql
-- Query
SELECT order_date, total_amount
FROM orders
WHERE customer_id = 123;

-- Covering index (includes all columns in SELECT + WHERE)
CREATE INDEX idx_covering ON orders(customer_id, order_date, total_amount);
-- Advantage: No need to access table data (index-only scan)
```

**3. When NOT to use index:**
- Low cardinality columns (< 10 distinct values)
- Small tables (< 1000 rows)
- Columns with frequent updates (index maintenance overhead)
- Columns used in calculations/functions

---

### **Q48: Query Rewriting for Performance**

**Problem:**
Optimize slow queries through rewriting.

**Example 1: OR to UNION**

**Slow:**
```sql
SELECT * FROM orders
WHERE customer_id = 123
   OR order_date = '2024-01-01';
-- Problem: Cannot use indexes efficiently
```

**Fast:**
```sql
SELECT * FROM orders WHERE customer_id = 123
UNION
SELECT * FROM orders WHERE order_date = '2024-01-01';
-- Can use separate indexes on each column
```

---

**Example 2: NOT IN to LEFT JOIN**

**Slow:**
```sql
SELECT * FROM customers
WHERE id NOT IN (SELECT customer_id FROM orders);
-- Problem: Full table scan if subquery has NULL
```

**Fast:**
```sql
SELECT c.*
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL;
-- Uses index efficiently
```

---

**Example 3: Avoid Functions on Indexed Columns**

**Slow:**
```sql
SELECT * FROM orders
WHERE YEAR(order_date) = 2024;
-- Problem: Index on order_date not used
```

**Fast:**
```sql
SELECT * FROM orders
WHERE order_date >= '2024-01-01'
  AND order_date < '2025-01-01';
-- Uses index on order_date
```

---

**Example 4: SELECT * to Specific Columns**

**Slow:**
```sql
SELECT * FROM orders
WHERE customer_id = 123;
-- Problem: Fetches all columns (more I/O)
```

**Fast:**
```sql
SELECT id, order_date, total_amount
FROM orders
WHERE customer_id = 123;
-- Fetches only needed columns
-- Can use covering index
```

---

**Example 5: Subquery to JOIN**

**Slow:**
```sql
SELECT
    c.name,
    (SELECT COUNT(*) FROM orders WHERE customer_id = c.id) AS order_count
FROM customers c;
-- Problem: Correlated subquery runs for each row
```

**Fast:**
```sql
SELECT
    c.name,
    COUNT(o.id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name;
-- Single pass through data
```

---

### **Q49: Partitioning Strategy**

**Problem:**
Design table partitioning for large tables.

**Scenario: Orders table with 100M+ rows**

**Range Partitioning by Date:**
```sql
CREATE TABLE orders (
    id INT,
    customer_id INT,
    order_date DATE,
    amount DECIMAL(10,2)
)
PARTITION BY RANGE (YEAR(order_date)) (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

**Benefits:**
```sql
-- Query with partition pruning
SELECT * FROM orders
WHERE order_date >= '2024-01-01'
  AND order_date < '2024-12-31';
-- Only scans p2024 partition (not entire table)

-- Easy old data archival
ALTER TABLE orders DROP PARTITION p2022;
-- Instantly removes all 2022 data
```

**Hash Partitioning (for even distribution):**
```sql
CREATE TABLE orders (
    id INT,
    customer_id INT,
    order_date DATE,
    amount DECIMAL(10,2)
)
PARTITION BY HASH(customer_id)
PARTITIONS 10;
-- Distributes data evenly across 10 partitions
```

**When to Partition:**
- ✅ Table > 10GB
- ✅ Queries often filter by partition key
- ✅ Need to archive/delete old data
- ✅ Time-series data

**When NOT to Partition:**
- ❌ Table < 1GB
- ❌ Queries don't filter by partition key
- ❌ More overhead than benefit

---

### **Q50: Materialized Views for Aggregations**

**Problem:**
Speed up repeated expensive aggregations.

**Scenario: Dashboard showing daily metrics**

**Slow Query (runs every page load):**
```sql
SELECT
    DATE(order_date) AS date,
    COUNT(*) AS total_orders,
    SUM(amount) AS total_revenue,
    AVG(amount) AS avg_order_value,
    COUNT(DISTINCT customer_id) AS unique_customers
FROM orders
GROUP BY DATE(order_date);
-- Scans entire orders table every time
```

**Solution: Materialized View**
```sql
-- Create materialized view (PostgreSQL)
CREATE MATERIALIZED VIEW daily_metrics AS
SELECT
    DATE(order_date) AS date,
    COUNT(*) AS total_orders,
    SUM(amount) AS total_revenue,
    AVG(amount) AS avg_order_value,
    COUNT(DISTINCT customer_id) AS unique_customers
FROM orders
GROUP BY DATE(order_date);

-- Create index on materialized view
CREATE INDEX idx_daily_metrics_date ON daily_metrics(date);

-- Refresh materialized view (run daily)
REFRESH MATERIALIZED VIEW daily_metrics;

-- Fast query
SELECT * FROM daily_metrics WHERE date >= '2024-01-01';
-- Instant results!
```

**BigQuery Alternative (Materialized View auto-refresh):**
```sql
CREATE MATERIALIZED VIEW project.dataset.daily_metrics
AS
SELECT
    DATE(order_timestamp) AS date,
    COUNT(*) AS total_orders,
    SUM(amount) AS total_revenue
FROM project.dataset.orders
GROUP BY date;
-- Auto-refreshes when base table changes
```

**dbt Alternative (Incremental Model):**
```sql
-- models/daily_metrics.sql
{{ config(
    materialized='incremental',
    unique_key='date'
) }}

SELECT
    DATE(order_date) AS date,
    COUNT(*) AS total_orders,
    SUM(amount) AS total_revenue
FROM {{ ref('orders') }}

{% if is_incremental() %}
    WHERE order_date >= (SELECT MAX(date) FROM {{ this }})
{% endif %}

GROUP BY date;
```

**When to Use Materialized Views:**
- ✅ Repeated expensive aggregations
- ✅ Dashboard/reporting queries
- ✅ Data changes infrequently
- ✅ Acceptable staleness (e.g., daily refresh)

**When NOT to Use:**
- ❌ Need real-time data
- ❌ Base data changes frequently
- ❌ Query is already fast
- ❌ Limited storage

---

## 🎯 INTERVIEW PREPARATION CHECKLIST

### **By Difficulty Level**

**Easy (Must Know):**
- Q1, Q2, Q3, Q4, Q5, Q6, Q7, Q21, Q22, Q23

**Medium (Common in Interviews):**
- Q11, Q12, Q13, Q14, Q15, Q26, Q27, Q31, Q41, Q42

**Hard (Senior Level):**
- Q19, Q28, Q32, Q36, Q37, Q40, Q46, Q47, Q48, Q50

### **By Topic Priority**

**1. JOINs (10 questions):** Q1-Q10
**2. Window Functions (10 questions):** Q11-Q20
**3. Aggregations (5 questions):** Q21-Q25
**4. Date/Time (5 questions):** Q26-Q30
**5. CTEs (5 questions):** Q31-Q35
**6. Analytics (5 questions):** Q36-Q40
**7. Data Quality (5 questions):** Q41-Q45
**8. Performance (5 questions):** Q46-Q50

---

## 📚 QUICK REFERENCE

### **Window Functions Syntax**
```sql
<function> OVER (
    [PARTITION BY column]
    [ORDER BY column]
    [ROWS/RANGE BETWEEN ... AND ...]
)
```

### **CTE Syntax**
```sql
WITH cte_name AS (
    SELECT ...
),
another_cte AS (
    SELECT ... FROM cte_name
)
SELECT ... FROM another_cte;
```

### **Date Functions (PostgreSQL)**
```sql
CURRENT_DATE, NOW()
DATE_TRUNC('month', timestamp)
EXTRACT(YEAR FROM date)
date + INTERVAL '7 days'
date1 - date2  -- Returns interval
```

### **Common Aggregates**
```sql
COUNT(*), COUNT(column), COUNT(DISTINCT column)
SUM(), AVG(), MIN(), MAX()
STRING_AGG(column, separator)
PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY column)
```

---

## 💡 FINAL TIPS FOR INTERVIEWS

1. **Always clarify requirements first**
   - "Should I handle NULL values?"
   - "Do you want all columns or specific ones?"
   - "What's the expected data volume?"

2. **Explain your approach before coding**
   - "I'll use a window function to rank within groups"
   - "I'll need a CTE to calculate the intermediate step"

3. **Test with edge cases**
   - Empty result set
   - NULL values
   - Duplicate values
   - Single row

4. **Discuss performance**
   - "This query will do a full table scan"
   - "We could add an index on X column"
   - "For large data, we might need partitioning"

5. **Know your database**
   - Syntax differences (MySQL vs PostgreSQL vs BigQuery)
   - Specific optimizations (Snowflake clustering)

---

**You now have 50 comprehensive SQL questions covering all major topics for Data Engineer interviews!**

**Practice Strategy:**
- Week 1: Q1-Q15 (Basics + Window Functions)
- Week 2: Q16-Q30 (Window Functions + Date/Time)
- Week 3: Q31-Q40 (CTEs + Analytics)
- Week 4: Q41-Q50 (Data Quality + Performance)

**Good luck! 🚀**
