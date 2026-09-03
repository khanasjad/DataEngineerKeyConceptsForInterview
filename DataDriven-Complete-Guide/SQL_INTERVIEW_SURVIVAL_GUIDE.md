# SQL INTERVIEW SURVIVAL GUIDE
## "I Can Barely Remember SQL But I'll Pass This Interview"

> **Philosophy**: 95% of SQL interviews ask the same 15 patterns. Memorize templates, not theory.

---

## 🎯 THE GOLDEN RULE

**Every SQL question = SELECT + FROM + (some combination of WHERE/JOIN/GROUP BY)**

Just fill in the blanks!

---

## 📚 TABLE OF CONTENTS
1. [The Basic Template (Start Here)](#basic-template)
2. [The 7 JOINs You Need](#joins)
3. [GROUP BY & Aggregations](#group-by)
4. [Window Functions (Scary But Easy)](#window-functions)
5. [Subqueries Made Simple](#subqueries)
6. [Common Question Patterns](#common-patterns)
7. [SQL Syntax Cheat Sheet](#syntax-cheat)
8. [When You're Stuck](#stuck)
9. [Final Checklist](#final-checklist)

---

## 🚀 THE BASIC TEMPLATE (START HERE) {#basic-template}

### Every SQL Query Follows This:
```sql
SELECT column1, column2, ...
FROM table_name
WHERE condition
ORDER BY column1 DESC;
```

**That's it. Everything else is just adding to this skeleton.**

### Step-by-Step:
1. **SELECT** = What columns do you want?
2. **FROM** = Which table?
3. **WHERE** = Filter rows (BEFORE grouping)
4. **GROUP BY** = Group rows together
5. **HAVING** = Filter groups (AFTER grouping)
6. **ORDER BY** = Sort the result
7. **LIMIT** = How many rows?

### The Order (MEMORIZE THIS):
```sql
SELECT          -- 5. Select these columns
FROM            -- 1. From this table
WHERE           -- 2. Filter rows
GROUP BY        -- 3. Group rows
HAVING          -- 4. Filter groups
ORDER BY        -- 6. Sort results
LIMIT           -- 7. Limit rows
```

**Execution order is DIFFERENT from writing order!**

---

## 🔗 THE 7 JOINS YOU NEED {#joins}

### Visual Memory Trick:
```
Table A          Table B
1 Alice          1 NYC
2 Bob            2 LA
3 Charlie        4 SF
```

### 1. INNER JOIN (Most Common - 70% of interviews)
**"Only matching rows from both tables"**

```sql
SELECT A.name, B.city
FROM table_a A
INNER JOIN table_b B ON A.id = B.id;

-- Result:
-- Alice, NYC
-- Bob, LA
```

**When to use**: "Find employees AND their departments" (must exist in both)

---

### 2. LEFT JOIN (30% of interviews)
**"All from left table, matching from right table"**

```sql
SELECT A.name, B.city
FROM table_a A
LEFT JOIN table_b B ON A.id = B.id;

-- Result:
-- Alice, NYC
-- Bob, LA
-- Charlie, NULL
```

**When to use**: "Find all employees, show department if they have one"

---

### 3. RIGHT JOIN (Rarely used, just do LEFT JOIN instead)
**"All from right table, matching from left table"**

```sql
SELECT A.name, B.city
FROM table_a A
RIGHT JOIN table_b B ON A.id = B.id;

-- Result:
-- Alice, NYC
-- Bob, LA
-- NULL, SF
```

---

### 4. FULL OUTER JOIN (Rare)
**"Everything from both tables"**

```sql
SELECT A.name, B.city
FROM table_a A
FULL OUTER JOIN table_b B ON A.id = B.id;

-- Result:
-- Alice, NYC
-- Bob, LA
-- Charlie, NULL
-- NULL, SF
```

---

### 5. SELF JOIN (When a table joins itself)
**"Join a table to itself"**

```sql
-- Example: Find employees and their managers
SELECT e.name AS employee, m.name AS manager
FROM employees e
JOIN employees m ON e.manager_id = m.id;
```

**Common question**: "Find employees who earn more than their manager"

---

### 6. CROSS JOIN (Cartesian Product)
**"Every row from A with every row from B"**

```sql
SELECT A.name, B.city
FROM table_a A
CROSS JOIN table_b B;

-- Result: 3 × 3 = 9 rows (every combination)
```

**Rarely used, but good for generating combinations**

---

### 7. Multiple JOINs (Chain them)
```sql
SELECT e.name, d.dept_name, l.city
FROM employees e
JOIN departments d ON e.dept_id = d.id
JOIN locations l ON d.location_id = l.id;
```

**Key**: Just keep chaining with JOIN ... ON

---

## 📊 GROUP BY & AGGREGATIONS {#group-by}

### The 5 Aggregate Functions (MEMORIZE):
1. **COUNT()** - Count rows
2. **SUM()** - Add up values
3. **AVG()** - Average
4. **MIN()** - Minimum value
5. **MAX()** - Maximum value

### Template:
```sql
SELECT column, AGGREGATE_FUNCTION(column)
FROM table
WHERE condition           -- Filter BEFORE grouping
GROUP BY column
HAVING condition          -- Filter AFTER grouping
ORDER BY column;
```

---

### Pattern 1: Count Per Group
```sql
-- Count employees per department
SELECT department_id, COUNT(*) AS employee_count
FROM employees
GROUP BY department_id;
```

---

### Pattern 2: Sum Per Group
```sql
-- Total salary per department
SELECT department_id, SUM(salary) AS total_salary
FROM employees
GROUP BY department_id;
```

---

### Pattern 3: Average Per Group
```sql
-- Average salary per department
SELECT department_id, AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id;
```

---

### Pattern 4: Multiple Aggregations
```sql
SELECT
    department_id,
    COUNT(*) AS num_employees,
    AVG(salary) AS avg_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary
FROM employees
GROUP BY department_id;
```

---

### Pattern 5: HAVING (Filter Groups)
```sql
-- Departments with more than 10 employees
SELECT department_id, COUNT(*) AS employee_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 10;
```

**Key Difference**:
- **WHERE** = Filter rows BEFORE grouping
- **HAVING** = Filter groups AFTER grouping

---

### Pattern 6: GROUP BY Multiple Columns
```sql
-- Count employees by department AND job title
SELECT department_id, job_title, COUNT(*) AS count
FROM employees
GROUP BY department_id, job_title;
```

---

## 🪟 WINDOW FUNCTIONS (Scary But Easy) {#window-functions}

### Why Window Functions?
- GROUP BY collapses rows
- Window functions keep all rows but add calculated columns

### The Template:
```sql
SELECT
    column,
    WINDOW_FUNCTION() OVER (PARTITION BY column ORDER BY column)
FROM table;
```

**Think of it as**: "For each partition, calculate something"

---

### Pattern 1: ROW_NUMBER() (Most Common)
**"Assign a unique number to each row within a group"**

```sql
-- Rank employees by salary within each department
SELECT
    name,
    department_id,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rank
FROM employees;

-- Result:
-- Alice, 1, 100000, 1
-- Bob, 1, 90000, 2
-- Charlie, 2, 95000, 1
-- David, 2, 85000, 2
```

**Use Case**: "Find top N per group"

---

### Pattern 2: RANK() vs DENSE_RANK()
```sql
-- RANK() - Gaps in ranking (1, 2, 2, 4)
SELECT
    name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- DENSE_RANK() - No gaps (1, 2, 2, 3)
SELECT
    name,
    salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;
```

**When salaries are**: 100k, 90k, 90k, 80k
- RANK: 1, 2, 2, 4
- DENSE_RANK: 1, 2, 2, 3
- ROW_NUMBER: 1, 2, 3, 4

---

### Pattern 3: Running Total (SUM with ROWS BETWEEN)
```sql
-- Running total of sales
SELECT
    date,
    sales,
    SUM(sales) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM sales;

-- Simplified:
SELECT
    date,
    sales,
    SUM(sales) OVER (ORDER BY date) AS running_total
FROM sales;
```

---

### Pattern 4: LAG() and LEAD() (Previous/Next Row)
```sql
-- Compare current salary with previous employee
SELECT
    name,
    salary,
    LAG(salary) OVER (ORDER BY salary) AS previous_salary,
    LEAD(salary) OVER (ORDER BY salary) AS next_salary
FROM employees;
```

**Common Use**: "Calculate difference from previous month/row"

```sql
-- Month-over-month growth
SELECT
    month,
    sales,
    sales - LAG(sales) OVER (ORDER BY month) AS growth
FROM monthly_sales;
```

---

### Pattern 5: Find Top N Per Group
```sql
-- Find top 3 highest paid employees per department
SELECT *
FROM (
    SELECT
        name,
        department_id,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rank
    FROM employees
) ranked
WHERE rank <= 3;
```

**THIS IS THE MOST COMMON INTERVIEW QUESTION!**

---

## 🎯 SUBQUERIES MADE SIMPLE {#subqueries}

### What is a Subquery?
**A query inside a query.** That's it.

---

### Pattern 1: Subquery in WHERE Clause
```sql
-- Find employees who earn more than average
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

**Template**: `WHERE column > (SELECT ...)`

---

### Pattern 2: Subquery in FROM Clause (Derived Table)
```sql
-- Find average of departmental averages
SELECT AVG(avg_salary) AS company_avg
FROM (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) dept_averages;
```

**Template**: `FROM (SELECT ...) AS alias`

---

### Pattern 3: IN / NOT IN with Subquery
```sql
-- Find employees in departments located in NYC
SELECT name
FROM employees
WHERE department_id IN (
    SELECT id
    FROM departments
    WHERE city = 'NYC'
);
```

---

### Pattern 4: EXISTS / NOT EXISTS
```sql
-- Find departments that have employees
SELECT dept_name
FROM departments d
WHERE EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.id
);
```

**EXISTS is faster than IN for large datasets**

---

### Pattern 5: Correlated Subquery (References Outer Query)
```sql
-- Find employees who earn more than average in their department
SELECT name, salary, department_id
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id
);
```

**Note**: Subquery uses `e1.department_id` from outer query

---

## 🔥 COMMON QUESTION PATTERNS {#common-patterns}

### Pattern 1: "Find Nth Highest/Lowest"
```sql
-- Find 2nd highest salary
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;

-- OR using window function
SELECT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
) ranked
WHERE rank = 2;
```

---

### Pattern 2: "Find Duplicates"
```sql
-- Find duplicate emails
SELECT email, COUNT(*) AS count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

---

### Pattern 3: "Remove Duplicates"
```sql
-- Keep only first occurrence of duplicate
DELETE FROM users
WHERE id NOT IN (
    SELECT MIN(id)
    FROM users
    GROUP BY email
);
```

---

### Pattern 4: "Top N Per Group"
```sql
-- Top 3 products by sales in each category
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

**THIS SHOWS UP IN 40% OF INTERVIEWS!**

---

### Pattern 5: "Self Join - Compare Within Same Table"
```sql
-- Find employees who earn more than their manager
SELECT e.name AS employee, m.name AS manager
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

---

### Pattern 6: "Date Calculations"
```sql
-- Find users who logged in within last 30 days
SELECT *
FROM users
WHERE last_login >= CURRENT_DATE - INTERVAL '30 days';

-- Month-over-month comparison
SELECT
    EXTRACT(MONTH FROM order_date) AS month,
    COUNT(*) AS orders
FROM orders
WHERE order_date >= CURRENT_DATE - INTERVAL '1 year'
GROUP BY EXTRACT(MONTH FROM order_date);
```

---

### Pattern 7: "Cumulative/Running Total"
```sql
-- Running total of sales
SELECT
    date,
    sales,
    SUM(sales) OVER (ORDER BY date) AS running_total
FROM daily_sales;
```

---

### Pattern 8: "Percentage Calculations"
```sql
-- Percentage of total sales per product
SELECT
    product,
    sales,
    ROUND(100.0 * sales / SUM(sales) OVER (), 2) AS percentage
FROM product_sales;
```

---

### Pattern 9: "Find Missing Numbers/IDs"
```sql
-- Find missing IDs in sequence
WITH RECURSIVE numbers AS (
    SELECT 1 AS num
    UNION ALL
    SELECT num + 1 FROM numbers WHERE num < 100
)
SELECT num
FROM numbers
WHERE num NOT IN (SELECT id FROM users);
```

---

### Pattern 10: "First/Last Record Per Group"
```sql
-- Most recent order per customer
SELECT customer_id, order_id, order_date
FROM (
    SELECT
        customer_id,
        order_id,
        order_date,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rn
    FROM orders
) latest
WHERE rn = 1;
```

---

## 📖 SQL SYNTAX CHEAT SHEET {#syntax-cheat}

### Common Functions

#### String Functions
```sql
CONCAT(str1, str2)              -- 'Hello' + 'World' = 'HelloWorld'
UPPER(str)                       -- 'hello' → 'HELLO'
LOWER(str)                       -- 'HELLO' → 'hello'
LENGTH(str)                      -- Length of string
SUBSTRING(str, start, length)    -- Extract substring
TRIM(str)                        -- Remove spaces
REPLACE(str, 'old', 'new')      -- Replace text
```

#### Date Functions
```sql
CURRENT_DATE                     -- Today's date
CURRENT_TIMESTAMP                -- Current date and time
EXTRACT(YEAR FROM date)          -- Get year
DATE_TRUNC('month', date)        -- First day of month
DATEDIFF(date1, date2)          -- Difference in days
DATE_ADD(date, INTERVAL 1 DAY)   -- Add days
```

#### Math Functions
```sql
ROUND(number, decimals)          -- Round number
CEIL(number)                     -- Round up
FLOOR(number)                    -- Round down
ABS(number)                      -- Absolute value
MOD(number, divisor)             -- Modulo
```

#### Conditional Logic
```sql
-- CASE Statement (SQL's if-else)
SELECT
    name,
    salary,
    CASE
        WHEN salary > 100000 THEN 'High'
        WHEN salary > 50000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_category
FROM employees;

-- COALESCE (First non-null value)
SELECT COALESCE(phone, email, 'No contact') AS contact
FROM users;

-- NULLIF (Return NULL if values are equal)
SELECT NULLIF(column, 0)  -- Avoid division by zero
FROM table;
```

### Common Clauses

#### DISTINCT
```sql
-- Unique values only
SELECT DISTINCT city FROM users;
```

#### LIMIT / OFFSET
```sql
-- Pagination
SELECT * FROM users
LIMIT 10 OFFSET 20;  -- Skip 20, get next 10
```

#### UNION / UNION ALL
```sql
-- Combine results (remove duplicates)
SELECT name FROM customers
UNION
SELECT name FROM employees;

-- Combine results (keep duplicates)
SELECT name FROM customers
UNION ALL
SELECT name FROM employees;
```

#### WITH (Common Table Expression - CTE)
```sql
-- Create temporary result set
WITH high_earners AS (
    SELECT * FROM employees WHERE salary > 100000
)
SELECT * FROM high_earners WHERE department = 'Engineering';
```

**CTEs make complex queries readable!**

---

## 😱 WHEN YOU'RE STUCK {#stuck}

### Step 1: Say this to buy time:
> "Let me break this down step by step..."

### Step 2: Use this framework:

**Question**: "Find the top 3 products by sales in each category"

**Your thought process OUT LOUD**:
1. "I need to group by category"
2. "I need to rank products within each category"
3. "Window function with PARTITION BY sounds right"
4. "Then filter for top 3"

### Step 3: Start with simple query, build up:
```sql
-- Step 1: Get all data
SELECT * FROM products;

-- Step 2: Add ranking
SELECT
    product_name,
    category,
    sales,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC) AS rank
FROM products;

-- Step 3: Filter for top 3
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

### Step 4: If completely stuck, use these hints:

**"Find duplicates"** → GROUP BY + HAVING COUNT(*) > 1

**"Top N per group"** → Window function + PARTITION BY

**"Compare with previous/next"** → LAG() / LEAD()

**"Running total"** → SUM() OVER (ORDER BY ...)

**"Percentage of total"** → value / SUM() OVER ()

**"Self comparison"** → Self join

---

## ✅ FINAL CHECKLIST {#final-checklist}

### Before Interview:
- [ ] Memorize the 5 aggregate functions (COUNT, SUM, AVG, MIN, MAX)
- [ ] Know the difference between WHERE and HAVING
- [ ] Understand INNER JOIN vs LEFT JOIN
- [ ] Practice one window function example (ROW_NUMBER)
- [ ] Review "Top N per group" pattern (most common!)

### During Interview:
- [ ] **Clarify the question**: "So I need to find... is that correct?"
- [ ] **Check table structure**: "Can I see the table schema?"
- [ ] **Start simple**: Write basic SELECT first, then build up
- [ ] **Test with example data**: "Let me trace through an example..."
- [ ] **Explain as you write**: "I'm using GROUP BY here because..."

### Common Mistakes to Avoid:
1. **Forgetting GROUP BY**: If you use aggregate function, you need GROUP BY
2. **WHERE vs HAVING**: WHERE = before grouping, HAVING = after grouping
3. **COUNT(*) vs COUNT(column)**: COUNT(*) includes NULLs, COUNT(column) doesn't
4. **JOIN without ON**: Always specify ON condition
5. **Subquery without alias**: Subquery in FROM needs an alias
6. **Comparing with NULL**: Use `IS NULL` not `= NULL`
7. **Forgetting DISTINCT**: If you want unique values, use DISTINCT

### SQL Execution Order (REMEMBER THIS):
```
1. FROM          (Get data from tables)
2. JOIN          (Combine tables)
3. WHERE         (Filter rows)
4. GROUP BY      (Group rows)
5. HAVING        (Filter groups)
6. SELECT        (Choose columns)
7. DISTINCT      (Remove duplicates)
8. ORDER BY      (Sort results)
9. LIMIT/OFFSET  (Limit rows)
```

---

## 🎓 INTERVIEW DAY TIPS

### What to Say:

**When starting**:
> "Let me first understand the table structure. Can you show me the schema?"

**While writing**:
> "I'll start with a basic query and build up from there..."

**When testing**:
> "Let me verify this with a sample input..."

**If stuck**:
> "I'm thinking this could be solved with a window function, but let me explore
> another approach with subqueries first..."

### The 3-Step Formula:

**Step 1: Simple Query**
```sql
SELECT * FROM table;
```

**Step 2: Add Filters/Joins**
```sql
SELECT columns
FROM table1
JOIN table2 ON condition
WHERE filter;
```

**Step 3: Add Aggregations/Window Functions**
```sql
SELECT
    columns,
    AGGREGATE_FUNCTION() OVER (PARTITION BY ...)
FROM ...
GROUP BY ...
```

---

## 🏆 TOP 20 INTERVIEW QUESTIONS

Practice these and you'll be ready for 80% of interviews:

1. [ ] Find Nth highest salary
2. [ ] Find duplicate records
3. [ ] Remove duplicate records
4. [ ] Find employees earning more than their manager
5. [ ] Find top 3 products per category
6. [ ] Calculate running total
7. [ ] Month-over-month growth
8. [ ] Find customers who never ordered
9. [ ] Find second most recent order per customer
10. [ ] Calculate percentage of total
11. [ ] Find gaps in sequential IDs
12. [ ] Rank employees by salary within department
13. [ ] Find departments with more than N employees
14. [ ] Consecutive login days
15. [ ] Find users who logged in today but not yesterday
16. [ ] Average salary per department (multiple ways)
17. [ ] Self-join to find pairs
18. [ ] UNION to combine results
19. [ ] CTE for complex queries
20. [ ] Date range queries (last 30 days, etc.)

---

## 🚀 YOU GOT THIS!

**Remember**:
1. SQL is just filling in templates
2. Start simple, build up complexity
3. Talk through your thought process
4. It's okay to not remember exact syntax - say "I'd look up the exact syntax for..."

**Most Important**:
- Know your JOINs (especially INNER and LEFT)
- Know GROUP BY + aggregate functions
- Know one window function (ROW_NUMBER)
- Practice "Top N per group" (shows up EVERYWHERE)

**Final Truth**:
You don't need to be a SQL expert. You need to show you can:
1. Break down a problem
2. Write basic queries
3. Think logically

That's it. Now go crush that interview! 💪

---

## 📝 QUICK REFERENCE CARD (Print This!)

```
SELECT columns
FROM table1
[JOIN table2 ON condition]
WHERE row_filter
GROUP BY columns
HAVING group_filter
ORDER BY columns [DESC]
LIMIT n;

AGGREGATES: COUNT, SUM, AVG, MIN, MAX

WINDOW: ROW_NUMBER() OVER (PARTITION BY col ORDER BY col)

JOINS: INNER (matching), LEFT (all from left), RIGHT (all from right)

WHERE vs HAVING: WHERE = before GROUP BY, HAVING = after GROUP BY

NULL: Use IS NULL, not = NULL

Top N per group:
ROW_NUMBER() OVER (PARTITION BY group ORDER BY value DESC)
WHERE rank <= N
```

**You're ready. Believe in yourself!** ✨
