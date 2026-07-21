# Chapter 1: SQL Fundamentals

**Duration:** 3-4 days | **Difficulty:** Beginner to Intermediate
**Problems to Solve:** 15-20 easy problems

---

## 📚 Table of Contents

1. [Introduction](#introduction)
2. [Relational Database Theory](#relational-theory)
3. [SELECT and FROM: The Foundation](#select-from)
4. [WHERE Clause: Filtering Data](#where-clause)
5. [Joins: Combining Tables](#joins)
6. [Aggregations: Summarizing Data](#aggregations)
7. [GROUP BY and HAVING](#group-by-having)
8. [Subqueries](#subqueries)
9. [Set Operations](#set-operations)
10. [Practice Problems](#practice-problems)
11. [Solutions](#solutions)
12. [Summary](#summary)

---

## 1. Introduction {#introduction}

### What You'll Learn

By the end of this chapter, you will:
- ✅ Understand relational database fundamentals
- ✅ Write SELECT queries with confidence
- ✅ Master all types of joins
- ✅ Perform aggregations and grouping
- ✅ Use subqueries effectively
- ✅ Solve 15-20 DataDriven easy problems

### Why SQL Matters

SQL (Structured Query Language) is the **universal language of data**. Whether you're:
- Analyzing business metrics
- Building data pipelines
- Creating reports
- Debugging data quality issues

...you'll use SQL daily as a data engineer.

### Real-World Application

**Example:** You're a data engineer at an e-commerce company. Your stakeholders need:
- Daily sales reports (aggregation)
- Customer purchase history (joins)
- Top-selling products by category (ranking + grouping)
- Revenue trends over time (temporal analysis)

All of these require SQL fundamentals.

---

## 2. Relational Database Theory {#relational-theory}

### What is a Relational Database?

A relational database organizes data into **tables** (also called relations). Each table consists of:
- **Rows** (also called tuples or records): Individual data entries
- **Columns** (also called attributes or fields): Data properties

**Example: E-commerce Database**

**Table: customers**
```
customer_id | name      | email             | signup_date
------------|-----------|-------------------|------------
1           | Alice     | alice@email.com   | 2024-01-15
2           | Bob       | bob@email.com     | 2024-02-20
3           | Charlie   | charlie@email.com | 2024-03-10
```

**Table: orders**
```
order_id | customer_id | order_date | total_amount
---------|-------------|------------|-------------
101      | 1           | 2024-01-20 | 150.00
102      | 2           | 2024-02-25 | 200.00
103      | 1           | 2024-03-05 | 75.00
```

### Key Concepts

#### **Primary Key**
A column (or combination of columns) that uniquely identifies each row.

- `customer_id` is the primary key of `customers` table
- `order_id` is the primary key of `orders` table

**Properties:**
- Must be unique
- Cannot be NULL
- One per table (though can be composite)

#### **Foreign Key**
A column that references the primary key of another table, establishing relationships.

- `customer_id` in `orders` is a foreign key referencing `customers(customer_id)`

**Purpose:** Maintains **referential integrity** (ensures orders can't reference non-existent customers)

#### **Relationships**

**One-to-Many (1:N):**
- One customer → many orders
- One category → many products

**Many-to-Many (M:N):**
- Requires a junction table
- Example: Students ↔ Courses (needs enrollment table)

**One-to-One (1:1):**
- Less common
- Example: User ↔ UserProfile

### Normal Forms (Brief Overview)

**Why normalize?**
- Reduce data redundancy
- Prevent update anomalies
- Maintain data integrity

**First Normal Form (1NF):**
- Each column contains atomic (indivisible) values
- No repeating groups

**Bad (Not 1NF):**
```sql
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    product_ids VARCHAR(100)  -- "1,2,3" BAD!
);
```

**Good (1NF):**
```sql
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);
```

**Second Normal Form (2NF):** 1NF + No partial dependencies (all non-key attributes depend on the whole primary key)

**Third Normal Form (3NF):** 2NF + No transitive dependencies (non-key attributes don't depend on other non-key attributes)

*We'll dive deeper into normalization in Chapter 7: Data Modeling*

---

## 3. SELECT and FROM: The Foundation {#select-from}

### Basic Syntax

```sql
SELECT column1, column2, ...
FROM table_name;
```

### SELECT *: All Columns

```sql
-- Get all columns from customers table
SELECT *
FROM customers;
```

**Result:**
```
customer_id | name      | email             | signup_date
------------|-----------|-------------------|------------
1           | Alice     | alice@email.com   | 2024-01-15
2           | Bob       | bob@email.com     | 2024-02-20
3           | Charlie   | charlie@email.com | 2024-03-10
```

**Best Practice:** ⚠️ Avoid `SELECT *` in production code
- Slower (retrieves unnecessary data)
- Breaks if table schema changes
- Unclear what data you need

**Use this instead:**
```sql
SELECT customer_id, name, email
FROM customers;
```

### SELECT Specific Columns

```sql
-- Get only names and emails
SELECT name, email
FROM customers;
```

**Result:**
```
name      | email
----------|------------------
Alice     | alice@email.com
Bob       | bob@email.com
Charlie   | charlie@email.com
```

### Column Aliases

Make column names more readable:

```sql
SELECT
    customer_id AS id,
    name AS customer_name,
    signup_date AS registered_on
FROM customers;
```

**Result:**
```
id | customer_name | registered_on
---|---------------|------------
1  | Alice         | 2024-01-15
2  | Bob           | 2024-02-20
3  | Charlie       | 2024-03-10
```

**Alias with spaces (use quotes):**
```sql
SELECT
    name AS "Customer Name",
    email AS "Email Address"
FROM customers;
```

### Expressions in SELECT

You can perform calculations:

```sql
SELECT
    order_id,
    total_amount,
    total_amount * 0.1 AS tax,
    total_amount * 1.1 AS total_with_tax
FROM orders;
```

**Result:**
```
order_id | total_amount | tax   | total_with_tax
---------|--------------|-------|---------------
101      | 150.00       | 15.00 | 165.00
102      | 200.00       | 20.00 | 220.00
103      | 75.00        | 7.50  | 82.50
```

### DISTINCT: Remove Duplicates

```sql
-- Get unique cities from customers
SELECT DISTINCT city
FROM customers;
```

**Example:**
```sql
-- customers table
customer_id | name    | city
------------|---------|----------
1           | Alice   | New York
2           | Bob     | Los Angeles
3           | Charlie | New York
4           | David   | New York

-- Query
SELECT DISTINCT city
FROM customers;

-- Result
city
-------------
New York
Los Angeles
```

### LIMIT: Restrict Rows Returned

```sql
-- Get first 10 customers
SELECT *
FROM customers
LIMIT 10;
```

**With OFFSET (pagination):**
```sql
-- Get customers 11-20 (page 2)
SELECT *
FROM customers
LIMIT 10 OFFSET 10;
```

---

## 4. WHERE Clause: Filtering Data {#where-clause}

### Basic Syntax

```sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

### Comparison Operators

```sql
=   -- Equal
<>  -- Not equal (also !=)
<   -- Less than
<=  -- Less than or equal
>   -- Greater than
>=  -- Greater than or equal
```

**Examples:**

```sql
-- Customers who signed up in 2024
SELECT *
FROM customers
WHERE signup_date >= '2024-01-01';

-- Orders with amount exactly $150
SELECT *
FROM orders
WHERE total_amount = 150.00;

-- Orders with amount NOT $150
SELECT *
FROM orders
WHERE total_amount <> 150.00;
```

### Logical Operators

#### AND (Both conditions must be true)

```sql
-- Orders in January 2024 with amount > $100
SELECT *
FROM orders
WHERE order_date >= '2024-01-01'
  AND order_date < '2024-02-01'
  AND total_amount > 100;
```

#### OR (At least one condition must be true)

```sql
-- Customers named Alice OR Bob
SELECT *
FROM customers
WHERE name = 'Alice'
   OR name = 'Bob';
```

#### NOT (Negates a condition)

```sql
-- Customers NOT named Alice
SELECT *
FROM customers
WHERE NOT name = 'Alice';

-- Equivalent to:
WHERE name <> 'Alice';
```

#### Combining AND/OR (Use parentheses!)

```sql
-- High-value orders in Jan OR any orders in Feb
SELECT *
FROM orders
WHERE (order_date >= '2024-01-01' AND order_date < '2024-02-01' AND total_amount > 200)
   OR (order_date >= '2024-02-01' AND order_date < '2024-03-01');
```

**Without parentheses (WRONG!):**
```sql
-- This doesn't work as intended!
WHERE order_date >= '2024-01-01'
  AND order_date < '2024-02-01'
  AND total_amount > 200
   OR order_date >= '2024-02-01'
  AND order_date < '2024-03-01';
-- OR takes precedence in unexpected ways
```

**Rule:** Always use parentheses to make logic clear!

### IN: Match Any Value in List

```sql
-- Customers in specific cities
SELECT *
FROM customers
WHERE city IN ('New York', 'Los Angeles', 'Chicago');

-- Equivalent to:
WHERE city = 'New York'
   OR city = 'Los Angeles'
   OR city = 'Chicago';
```

**NOT IN:**
```sql
-- Exclude certain cities
SELECT *
FROM customers
WHERE city NOT IN ('New York', 'Los Angeles');
```

### BETWEEN: Range Check

```sql
-- Orders between $100 and $200 (inclusive)
SELECT *
FROM orders
WHERE total_amount BETWEEN 100 AND 200;

-- Equivalent to:
WHERE total_amount >= 100
  AND total_amount <= 200;
```

**Dates:**
```sql
-- Orders in January 2024
SELECT *
FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-01-31';
```

### LIKE: Pattern Matching

**Wildcards:**
- `%` : Zero or more characters
- `_` : Exactly one character

**Examples:**

```sql
-- Names starting with 'A'
SELECT * FROM customers
WHERE name LIKE 'A%';

-- Names ending with 'e'
SELECT * FROM customers
WHERE name LIKE '%e';

-- Names containing 'li'
SELECT * FROM customers
WHERE name LIKE '%li%';

-- Names with exactly 5 characters
SELECT * FROM customers
WHERE name LIKE '_____';

-- Names starting with 'A' and ending with 'e'
SELECT * FROM customers
WHERE name LIKE 'A%e';
```

**Case sensitivity:**
- PostgreSQL: `LIKE` is case-sensitive, `ILIKE` is case-insensitive
- MySQL: `LIKE` is case-insensitive by default

```sql
-- Case-insensitive search (PostgreSQL)
SELECT * FROM customers
WHERE name ILIKE 'alice';  -- Matches 'Alice', 'ALICE', 'alice'
```

### IS NULL / IS NOT NULL

```sql
-- Customers without email
SELECT *
FROM customers
WHERE email IS NULL;

-- Customers with email
SELECT *
FROM customers
WHERE email IS NOT NULL;
```

**⚠️ Common mistake:**
```sql
-- WRONG! This doesn't work
WHERE email = NULL;

-- CORRECT
WHERE email IS NULL;
```

**Why?** NULL means "unknown", and unknown = unknown is unknown (not true!)

---

## 5. Joins: Combining Tables {#joins}

### Why Joins?

Real data is split across multiple tables (normalization). Joins let us combine related data.

**Example:**
- `customers` table has customer info
- `orders` table has order info
- To see "customer names with their orders", we need to JOIN

### Sample Data

**customers:**
```
customer_id | name
------------|--------
1           | Alice
2           | Bob
3           | Charlie
```

**orders:**
```
order_id | customer_id | total_amount
---------|-------------|-------------
101      | 1           | 150.00
102      | 1           | 200.00
103      | 2           | 75.00
```

### INNER JOIN

**Returns only matching rows from both tables**

**Syntax:**
```sql
SELECT columns
FROM table1
INNER JOIN table2 ON table1.key = table2.key;
```

**Example:**
```sql
SELECT
    c.name,
    o.order_id,
    o.total_amount
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;
```

**Result:**
```
name    | order_id | total_amount
--------|----------|-------------
Alice   | 101      | 150.00
Alice   | 102      | 200.00
Bob     | 103      | 75.00
```

**Note:** Charlie not included (no orders)

**Visual Representation:**
```
customers          orders
---------         -------
1 Alice      ←→   101 (cust=1)
2 Bob        ←→   102 (cust=1)
3 Charlie         103 (cust=2)

INNER JOIN = Intersection
Result: 3 rows (only matched records)
```

### LEFT JOIN (LEFT OUTER JOIN)

**Returns all rows from left table + matching rows from right**
**If no match, right table columns are NULL**

**Example:**
```sql
SELECT
    c.name,
    o.order_id,
    o.total_amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
```

**Result:**
```
name    | order_id | total_amount
--------|----------|-------------
Alice   | 101      | 150.00
Alice   | 102      | 200.00
Bob     | 103      | 75.00
Charlie | NULL     | NULL
```

**Note:** Charlie included with NULL order values

**Visual:**
```
LEFT JOIN = All from left + matches from right
Result: 4 rows (all customers, even without orders)
```

### RIGHT JOIN (RIGHT OUTER JOIN)

**Returns all rows from right table + matching rows from left**

**Example:**
```sql
SELECT
    c.name,
    o.order_id,
    o.total_amount
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id;
```

**Result:** Same as INNER JOIN in this case (all orders have customers)

**When different:** If `orders` had orphaned records (customer_id not in customers):
```
orders:
order_id | customer_id | total_amount
---------|-------------|-------------
101      | 1           | 150.00
104      | 99          | 300.00  ← customer_id 99 doesn't exist

RIGHT JOIN result:
name    | order_id | total_amount
--------|----------|-------------
Alice   | 101      | 150.00
NULL    | 104      | 300.00  ← No customer name
```

### FULL OUTER JOIN

**Returns all rows from both tables**
**NULL where no match**

**Example:**
```sql
SELECT
    c.name,
    o.order_id,
    o.total_amount
FROM customers c
FULL OUTER JOIN orders o ON c.customer_id = o.customer_id;
```

**Result:**
```
name    | order_id | total_amount
--------|----------|-------------
Alice   | 101      | 150.00
Alice   | 102      | 200.00
Bob     | 103      | 75.00
Charlie | NULL     | NULL
NULL    | 104      | 300.00  (if orphaned order exists)
```

**Use case:** Finding mismatches (customers without orders, orders without customers)

### CROSS JOIN

**Cartesian product: Every row from table1 × every row from table2**

**Example:**
```sql
SELECT
    c.name,
    p.product_name
FROM customers c
CROSS JOIN products p;
```

**If customers has 3 rows and products has 5 rows:**
**Result: 3 × 5 = 15 rows**

**Use case:**
- Generate all combinations
- Create date series × other dimension
- Usually NOT what you want (very large result sets!)

### Self Join

**Join a table to itself**

**Use case:** Hierarchical data (employees and managers)

**employees table:**
```
employee_id | name    | manager_id
------------|---------|------------
1           | Alice   | NULL
2           | Bob     | 1
3           | Charlie | 1
4           | David   | 2
```

**Query:** Find employees with their manager's name

```sql
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

**Result:**
```
employee | manager
---------|--------
Alice    | NULL
Bob      | Alice
Charlie  | Alice
David    | Bob
```

### Multiple Joins

**Example:** Customers → Orders → Products

```sql
SELECT
    c.name AS customer_name,
    o.order_date,
    p.product_name,
    oi.quantity,
    oi.quantity * p.price AS line_total
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
WHERE o.order_date >= '2024-01-01';
```

**Join order matters for readability, but optimizer handles execution order**

### Join Conditions: More Than Equality

**Non-equi join:**
```sql
-- Find overlapping time periods
SELECT
    e1.event_name AS event1,
    e2.event_name AS event2
FROM events e1
INNER JOIN events e2
    ON e1.event_id < e2.event_id  -- Avoid duplicate pairs
    AND e1.end_time > e2.start_time  -- Overlap condition
    AND e1.start_time < e2.end_time;
```

---

## 6. Aggregations: Summarizing Data {#aggregations}

### Aggregate Functions

**Common functions:**
- `COUNT()` - Count rows
- `SUM()` - Add up values
- `AVG()` - Average value
- `MIN()` - Minimum value
- `MAX()` - Maximum value
- `STDDEV()` - Standard deviation
- `VARIANCE()` - Variance

### COUNT

**Count all rows:**
```sql
SELECT COUNT(*) AS total_customers
FROM customers;
```

**Count non-NULL values:**
```sql
SELECT COUNT(email) AS customers_with_email
FROM customers;
```

**Count distinct values:**
```sql
SELECT COUNT(DISTINCT city) AS unique_cities
FROM customers;
```

**Example:**
```
customers:
customer_id | email
------------|------------------
1           | alice@email.com
2           | NULL
3           | charlie@email.com

COUNT(*) = 3
COUNT(email) = 2
COUNT(DISTINCT email) = 2
```

### SUM

**Total of all values:**
```sql
SELECT SUM(total_amount) AS total_revenue
FROM orders;
```

**Example:**
```
orders:
order_id | total_amount
---------|-------------
101      | 150.00
102      | 200.00
103      | 75.00

SUM(total_amount) = 425.00
```

### AVG

**Average value:**
```sql
SELECT AVG(total_amount) AS average_order_value
FROM orders;
```

**Result:** 141.67 (425 / 3)

**⚠️ NULL handling:** NULL values are excluded from AVG calculation

### MIN and MAX

```sql
SELECT
    MIN(total_amount) AS smallest_order,
    MAX(total_amount) AS largest_order
FROM orders;
```

**Result:**
```
smallest_order | largest_order
---------------|---------------
75.00          | 200.00
```

### Multiple Aggregations

```sql
SELECT
    COUNT(*) AS total_orders,
    SUM(total_amount) AS total_revenue,
    AVG(total_amount) AS avg_order_value,
    MIN(total_amount) AS min_order,
    MAX(total_amount) AS max_order
FROM orders
WHERE order_date >= '2024-01-01';
```

---

## 7. GROUP BY and HAVING {#group-by-having}

### GROUP BY: Aggregating by Groups

**Syntax:**
```sql
SELECT column, aggregate_function(column)
FROM table
GROUP BY column;
```

**Example:** Total sales per customer

```sql
SELECT
    customer_id,
    COUNT(*) AS order_count,
    SUM(total_amount) AS total_spent
FROM orders
GROUP BY customer_id;
```

**Result:**
```
customer_id | order_count | total_spent
------------|-------------|------------
1           | 2           | 350.00
2           | 1           | 75.00
```

**How it works:**
1. Database groups rows by `customer_id`
2. For each group, calculates aggregates
3. Returns one row per group

### GROUP BY Multiple Columns

```sql
SELECT
    customer_id,
    EXTRACT(YEAR FROM order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    COUNT(*) AS orders,
    SUM(total_amount) AS revenue
FROM orders
GROUP BY customer_id, EXTRACT(YEAR FROM order_date), EXTRACT(MONTH FROM order_date)
ORDER BY customer_id, year, month;
```

**Result:** Orders and revenue per customer per month

### Rules for GROUP BY

**⚠️ Important Rule:**
**Every column in SELECT (except aggregates) MUST be in GROUP BY**

**WRONG:**
```sql
-- This will error!
SELECT customer_id, name, COUNT(*)
FROM orders
GROUP BY customer_id;
-- name is not in GROUP BY
```

**CORRECT:**
```sql
SELECT customer_id, COUNT(*)
FROM orders
GROUP BY customer_id;
```

**Or if you need name:**
```sql
SELECT o.customer_id, c.name, COUNT(*)
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
GROUP BY o.customer_id, c.name;
```

### HAVING: Filtering Groups

**WHERE filters rows BEFORE grouping**
**HAVING filters groups AFTER grouping**

**Example:** Customers with more than 1 order

```sql
SELECT
    customer_id,
    COUNT(*) AS order_count,
    SUM(total_amount) AS total_spent
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

**Result:**
```
customer_id | order_count | total_spent
------------|-------------|------------
1           | 2           | 350.00
```

**WHERE vs HAVING:**

```sql
-- WHERE: Filter rows before grouping
SELECT customer_id, COUNT(*)
FROM orders
WHERE total_amount > 100  -- Filter individual orders
GROUP BY customer_id;

-- HAVING: Filter groups after aggregation
SELECT customer_id, COUNT(*)
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 1;  -- Filter customers with >1 order

-- Both together
SELECT customer_id, COUNT(*)
FROM orders
WHERE total_amount > 100  -- Only count orders > $100
GROUP BY customer_id
HAVING COUNT(*) > 1;  -- Only customers with >1 large order
```

### Order of Execution

```sql
SELECT customer_id, COUNT(*) AS order_count
FROM orders
WHERE order_date >= '2024-01-01'  -- 1. Filter rows
GROUP BY customer_id               -- 2. Group
HAVING COUNT(*) > 1                -- 3. Filter groups
ORDER BY COUNT(*) DESC             -- 4. Sort
LIMIT 10;                          -- 5. Limit results
```

**Execution order:**
1. FROM - Get table
2. WHERE - Filter rows
3. GROUP BY - Create groups
4. HAVING - Filter groups
5. SELECT - Calculate columns
6. ORDER BY - Sort results
7. LIMIT - Limit output

---

## 8. Subqueries {#subqueries}

### What is a Subquery?

A query nested inside another query.

### Types of Subqueries

1. **Scalar subquery** - Returns single value
2. **Row subquery** - Returns single row
3. **Column subquery** - Returns single column
4. **Table subquery** - Returns multiple rows/columns

### Scalar Subquery

**Returns one value, used in SELECT or WHERE**

**Example:** Compare each order to average

```sql
SELECT
    order_id,
    total_amount,
    (SELECT AVG(total_amount) FROM orders) AS avg_amount,
    total_amount - (SELECT AVG(total_amount) FROM orders) AS diff_from_avg
FROM orders;
```

**Result:**
```
order_id | total_amount | avg_amount | diff_from_avg
---------|--------------|------------|---------------
101      | 150.00       | 141.67     | 8.33
102      | 200.00       | 141.67     | 58.33
103      | 75.00        | 141.67     | -66.67
```

### IN / NOT IN with Subquery

**Example:** Customers who have placed orders

```sql
SELECT name
FROM customers
WHERE customer_id IN (
    SELECT DISTINCT customer_id
    FROM orders
);
```

**Equivalent with JOIN (usually faster):**
```sql
SELECT DISTINCT c.name
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;
```

**NOT IN:** Customers who haven't ordered

```sql
SELECT name
FROM customers
WHERE customer_id NOT IN (
    SELECT DISTINCT customer_id
    FROM orders
);
```

### EXISTS / NOT EXISTS

**Checks if subquery returns any rows (Boolean)**

**Example:** Customers with orders

```sql
SELECT name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

**Why EXISTS is better than IN:**
- Stops at first match (faster)
- Handles NULL better

**NOT EXISTS:** Customers without orders

```sql
SELECT name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

### Subquery in FROM (Derived Table)

**Treat subquery result as a table**

```sql
SELECT
    customer_category,
    COUNT(*) AS customer_count,
    AVG(total_spent) AS avg_spent
FROM (
    SELECT
        customer_id,
        SUM(total_amount) AS total_spent,
        CASE
            WHEN SUM(total_amount) >= 500 THEN 'VIP'
            WHEN SUM(total_amount) >= 200 THEN 'Premium'
            ELSE 'Standard'
        END AS customer_category
    FROM orders
    GROUP BY customer_id
) customer_segments
GROUP BY customer_category;
```

**Better with CTE (Chapter 3)**

### Correlated Subquery

**Subquery references outer query (runs for each row)**

**Example:** Orders above customer's average

```sql
SELECT
    order_id,
    customer_id,
    total_amount
FROM orders o1
WHERE total_amount > (
    SELECT AVG(total_amount)
    FROM orders o2
    WHERE o2.customer_id = o1.customer_id
);
```

**⚠️ Performance:** Correlated subqueries can be slow (runs N times for N rows)

---

## 9. Set Operations {#set-operations}

### UNION: Combine Results (Remove Duplicates)

**Syntax:**
```sql
SELECT column FROM table1
UNION
SELECT column FROM table2;
```

**Example:**
```sql
-- All cities from customers and suppliers
SELECT city FROM customers
UNION
SELECT city FROM suppliers;
```

**Result:**
```
city
-------------
New York
Los Angeles
Chicago
-- Duplicates removed
```

### UNION ALL: Keep Duplicates

```sql
SELECT city FROM customers
UNION ALL
SELECT city FROM suppliers;
```

**Faster than UNION (no duplicate check)**

### INTERSECT: Common Values

**Values in BOTH queries**

```sql
-- Cities with both customers and suppliers
SELECT city FROM customers
INTERSECT
SELECT city FROM suppliers;
```

### EXCEPT (MINUS in Oracle): Difference

**Values in first query but NOT in second**

```sql
-- Cities with customers but no suppliers
SELECT city FROM customers
EXCEPT
SELECT city FROM suppliers;
```

**Rules for Set Operations:**
1. Same number of columns
2. Compatible data types
3. Column names from first query

---

## 10. Practice Problems {#practice-problems}

### Problem 1: "30-Day Page View Counts" (Easy)

**Scenario:** You have a `page_views` table with columns:
- `view_id` (INT)
- `user_id` (INT)
- `page_url` (VARCHAR)
- `view_date` (DATE)

**Task:** Count total page views in the last 30 days (from '2024-03-01' to '2024-03-30')

**Hints:**
- Use COUNT(*)
- Filter with WHERE
- Date range: BETWEEN or >= AND <=

**Try yourself before looking at solution!**

---

### Problem 2: "Above Average" (Easy)

**Scenario:** `products` table:
- `product_id` (INT)
- `product_name` (VARCHAR)
- `price` (DECIMAL)
- `category` (VARCHAR)

**Task:** Find all products with price above the average price of ALL products

**Hints:**
- Subquery to calculate average
- WHERE price > (subquery)

---

### Problem 3: Customer Order Summary (Easy)

**Scenario:** `customers` and `orders` tables (as defined earlier)

**Task:** Show each customer's name, total number of orders, and total amount spent. Include customers with no orders.

**Hints:**
- LEFT JOIN
- GROUP BY customer
- Use COALESCE for NULLs

---

### Problem 4: Top 5 Customers (Easy-Medium)

**Task:** Find top 5 customers by total spending

**Hints:**
- GROUP BY customer
- ORDER BY total DESC
- LIMIT 5

---

### Problem 5: Monthly Sales (Medium)

**Scenario:** `orders` table with `order_date` and `total_amount`

**Task:** Calculate total sales for each month in 2024

**Hints:**
- EXTRACT(YEAR FROM date), EXTRACT(MONTH FROM date)
- GROUP BY year, month
- WHERE year = 2024

---

## 11. Solutions {#solutions}

### Solution 1: "30-Day Page View Counts"

```sql
SELECT COUNT(*) AS total_page_views
FROM page_views
WHERE view_date BETWEEN '2024-03-01' AND '2024-03-30';
```

**Alternative:**
```sql
SELECT COUNT(*) AS total_page_views
FROM page_views
WHERE view_date >= '2024-03-01'
  AND view_date <= '2024-03-30';
```

**Explanation:**
- `COUNT(*)` counts all rows
- `BETWEEN` is inclusive (includes both '2024-03-01' and '2024-03-30')
- Returns a single number

---

### Solution 2: "Above Average"

```sql
SELECT
    product_id,
    product_name,
    price
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

**With more context:**
```sql
SELECT
    product_id,
    product_name,
    category,
    price,
    (SELECT AVG(price) FROM products) AS avg_price,
    price - (SELECT AVG(price) FROM products) AS above_avg_by
FROM products
WHERE price > (SELECT AVG(price) FROM products)
ORDER BY price DESC;
```

**Explanation:**
- Subquery `(SELECT AVG(price) FROM products)` calculates average price
- Main query filters products above that average
- Subquery runs once, result used for all comparisons

---

### Solution 3: Customer Order Summary

```sql
SELECT
    c.customer_id,
    c.name,
    COUNT(o.order_id) AS order_count,
    COALESCE(SUM(o.total_amount), 0) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name
ORDER BY total_spent DESC;
```

**Explanation:**
- `LEFT JOIN` includes customers without orders
- `COUNT(o.order_id)` counts orders (NULL for customers without orders → 0)
- `COALESCE(SUM(...), 0)` replaces NULL with 0 for display
- `GROUP BY c.customer_id, c.name` groups by customer

**Result:**
```
customer_id | name    | order_count | total_spent
------------|---------|-------------|------------
1           | Alice   | 2           | 350.00
2           | Bob     | 1           | 75.00
3           | Charlie | 0           | 0.00
```

---

### Solution 4: Top 5 Customers

```sql
SELECT
    c.customer_id,
    c.name,
    SUM(o.total_amount) AS total_spent
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name
ORDER BY total_spent DESC
LIMIT 5;
```

**Explanation:**
- `INNER JOIN` excludes customers without orders
- `GROUP BY` aggregates per customer
- `ORDER BY total_spent DESC` sorts highest first
- `LIMIT 5` returns top 5

---

### Solution 5: Monthly Sales

```sql
SELECT
    EXTRACT(YEAR FROM order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    COUNT(*) AS order_count,
    SUM(total_amount) AS monthly_revenue
FROM orders
WHERE EXTRACT(YEAR FROM order_date) = 2024
GROUP BY EXTRACT(YEAR FROM order_date), EXTRACT(MONTH FROM order_date)
ORDER BY year, month;
```

**Alternative (cleaner with CTEs - Chapter 3):**
```sql
WITH monthly_data AS (
    SELECT
        DATE_TRUNC('month', order_date) AS month_start,
        total_amount
    FROM orders
    WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01'
)
SELECT
    month_start,
    COUNT(*) AS order_count,
    SUM(total_amount) AS monthly_revenue
FROM monthly_data
GROUP BY month_start
ORDER BY month_start;
```

**Result:**
```
year | month | order_count | monthly_revenue
-----|-------|-------------|----------------
2024 | 1     | 15          | 2500.00
2024 | 2     | 20          | 3200.00
2024 | 3     | 18          | 2800.00
```

---

## 12. Summary {#summary}

### Key Concepts Mastered

✅ **SELECT and FROM:** Retrieving data from tables
✅ **WHERE:** Filtering rows with conditions
✅ **Joins:** Combining related tables (INNER, LEFT, RIGHT, FULL, CROSS, SELF)
✅ **Aggregations:** Summarizing data (COUNT, SUM, AVG, MIN, MAX)
✅ **GROUP BY:** Aggregating by groups
✅ **HAVING:** Filtering groups
✅ **Subqueries:** Nested queries for complex logic
✅ **Set Operations:** UNION, INTERSECT, EXCEPT

### Self-Assessment Questions

Can you answer these confidently?

1. What's the difference between INNER JOIN and LEFT JOIN?
2. When do you use WHERE vs HAVING?
3. What does EXISTS do differently than IN?
4. Why use COALESCE with aggregations?
5. What's the order of SQL execution (FROM, WHERE, GROUP BY, etc.)?

### Common Mistakes to Avoid

❌ Using `WHERE` to filter after `GROUP BY` (use `HAVING`)
❌ Forgetting `DISTINCT` in `COUNT DISTINCT`
❌ Not handling NULLs (use `IS NULL`, not `= NULL`)
❌ Using `SELECT *` in production
❌ Not aliasing subqueries in FROM
❌ Mixing AND/OR without parentheses

### Next Steps

**You're ready for Chapter 2: SQL Window Functions!**

Window functions will let you:
- Rank rows
- Calculate running totals
- Compare with previous/next rows
- Compute moving averages

**Before moving on:**
- [ ] Solved 15+ easy problems on DataDriven.io
- [ ] Understand all join types
- [ ] Comfortable with GROUP BY and aggregations
- [ ] Can write subqueries confidently

### Practice Recommendations

**This week:**
- Solve 20-30 easy SQL problems on DataDriven
- Focus on: aggregations, joins, filtering
- Aim for 80%+ success rate
- Time yourself (target: 15-20 min per problem)

**Keep a learning journal:**
- What patterns did you see?
- What was confusing?
- What questions do you have?

---

**Congratulations on completing Chapter 1!** 🎉

**Next:** `02-SQL-Window-Functions/Chapter-02-Window-Functions.md`

---
