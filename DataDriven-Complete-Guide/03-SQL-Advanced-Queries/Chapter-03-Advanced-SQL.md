# Chapter 3: Advanced SQL Queries

**Duration:** 5-7 days | **Difficulty:** Intermediate to Advanced
**Problems to Solve:** 20-25 medium problems
**Prerequisites:** Chapters 1-2 (SQL Fundamentals & Window Functions)

---

## 📚 Table of Contents

1. [Introduction](#introduction)
2. [Common Table Expressions (CTEs)](#ctes)
3. [Recursive CTEs](#recursive-ctes)
4. [Complex Joins and Self-Joins](#complex-joins)
5. [Correlated Subqueries](#correlated-subqueries)
6. [CASE Statements and Conditional Logic](#case-statements)
7. [Date and Time Manipulation](#datetime)
8. [String Functions and Pattern Matching](#string-functions)
9. [JSON and Array Operations](#json-arrays)
10. [Query Optimization](#optimization)
11. [Practice Problems](#practice-problems)
12. [Solutions](#solutions)
13. [Summary](#summary)

---

## 1. Introduction {#introduction}

### What You'll Learn

Advanced SQL techniques that separate good data engineers from great ones:
- ✅ Write readable, maintainable queries with CTEs
- ✅ Handle hierarchical data with recursive queries
- ✅ Master complex multi-table joins
- ✅ Manipulate dates, times, and strings like a pro
- ✅ Optimize slow queries
- ✅ Solve 70-80% of DataDriven medium problems

### Why These Topics Matter

**Real-world scenario:** You're analyzing user behavior across multiple sessions, each session has events, events have attributes stored as JSON, and you need to:
- Join 5+ tables
- Parse JSON fields
- Calculate time differences
- Group by date ranges
- Present in clean, readable format

**This requires:** CTEs, JSON functions, date manipulation, and optimization - all covered in this chapter.

---

## 2. Common Table Expressions (CTEs) {#ctes}

### What is a CTE?

A **Common Table Expression** is a temporary named result set that exists within a single query.

**Syntax:**
```sql
WITH cte_name AS (
    SELECT ...
)
SELECT * FROM cte_name;
```

### Why CTEs?

**Before CTEs (nested subqueries):**
```sql
SELECT *
FROM (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
) customer_totals
WHERE total > 1000;
```

**With CTE (more readable):**
```sql
WITH customer_totals AS (
    SELECT
        customer_id,
        SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT *
FROM customer_totals
WHERE total > 1000;
```

**Benefits:**
1. **Readability:** Named, logical steps
2. **Reusability:** Reference CTE multiple times
3. **Debugging:** Test each CTE independently
4. **Maintainability:** Easy to modify

---

### Single CTE Example

**Problem:** Find customers who spent above average

**Without CTE:**
```sql
SELECT
    c.name,
    o.total_spent
FROM customers c
JOIN (
    SELECT customer_id, SUM(amount) AS total_spent
    FROM orders
    GROUP BY customer_id
) o ON c.customer_id = o.customer_id
WHERE o.total_spent > (
    SELECT AVG(total_spent)
    FROM (
        SELECT customer_id, SUM(amount) AS total_spent
        FROM orders
        GROUP BY customer_id
    ) subquery
);
```

**With CTE:**
```sql
WITH customer_spending AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY customer_id
),
average_spending AS (
    SELECT AVG(total_spent) AS avg_spent
    FROM customer_spending
)
SELECT
    c.name,
    cs.total_spent,
    a.avg_spent,
    cs.total_spent - a.avg_spent AS above_average_by
FROM customers c
JOIN customer_spending cs ON c.customer_id = cs.customer_id
CROSS JOIN average_spending a
WHERE cs.total_spent > a.avg_spent
ORDER BY cs.total_spent DESC;
```

**Much clearer!**

---

### Multiple CTEs (Pipeline Pattern)

**Pattern:** Break complex logic into digestible steps

**Problem:** "Proof of Presence" - Two-factor authentication analysis

Find users who:
1. Requested 2FA code
2. Confirmed within 5 minutes
3. Calculate confirmation rate

```sql
WITH
-- Step 1: Get 2FA requests
requests AS (
    SELECT
        user_id,
        request_id,
        request_time
    FROM two_factor_requests
    WHERE request_time >= '2024-01-01'
),
-- Step 2: Get confirmations
confirmations AS (
    SELECT
        user_id,
        request_id,
        confirm_time
    FROM two_factor_confirmations
    WHERE confirm_time >= '2024-01-01'
),
-- Step 3: Join and calculate time difference
matched_events AS (
    SELECT
        r.user_id,
        r.request_id,
        r.request_time,
        c.confirm_time,
        EXTRACT(EPOCH FROM (c.confirm_time - r.request_time)) / 60 AS minutes_to_confirm
    FROM requests r
    LEFT JOIN confirmations c
        ON r.user_id = c.user_id
        AND r.request_id = c.request_id
),
-- Step 4: Filter valid confirmations (within 5 min)
valid_confirmations AS (
    SELECT
        user_id,
        request_id,
        CASE
            WHEN minutes_to_confirm IS NOT NULL
             AND minutes_to_confirm <= 5
            THEN 1
            ELSE 0
        END AS is_confirmed
    FROM matched_events
)
-- Final: Calculate confirmation rate
SELECT
    COUNT(*) AS total_requests,
    SUM(is_confirmed) AS confirmed_requests,
    ROUND(100.0 * SUM(is_confirmed) / COUNT(*), 2) AS confirmation_rate_pct
FROM valid_confirmations;
```

**Benefits of this approach:**
- Each CTE is testable independently
- Easy to add/remove steps
- Clear business logic flow
- Can add WHERE clauses to any step for debugging

---

### Referencing CTEs Multiple Times

**Problem:** Compare each product's sales to category average

```sql
WITH product_sales AS (
    SELECT
        product_id,
        category_id,
        SUM(amount) AS total_sales
    FROM sales
    GROUP BY product_id, category_id
),
category_averages AS (
    SELECT
        category_id,
        AVG(total_sales) AS avg_category_sales
    FROM product_sales
    GROUP BY category_id
)
SELECT
    ps.product_id,
    ps.category_id,
    ps.total_sales,
    ca.avg_category_sales,
    ps.total_sales - ca.avg_category_sales AS vs_category_avg,
    ROUND(100.0 * ps.total_sales / ca.avg_category_sales, 2) AS pct_of_avg
FROM product_sales ps
JOIN category_averages ca ON ps.category_id = ca.category_id
ORDER BY ps.category_id, ps.total_sales DESC;
```

---

### CTEs vs Subqueries vs Temp Tables

**When to use each:**

**CTEs:**
- ✅ Query-scoped (disappear after query)
- ✅ Better readability
- ✅ Can't be indexed
- ✅ Computed once per query

**Subqueries:**
- ✅ Inline, quick one-offs
- ❌ Less readable when nested
- ❌ Can't reuse

**Temp Tables:**
- ✅ Session-scoped
- ✅ Can be indexed
- ✅ Useful for very large intermediate results
- ❌ Requires cleanup
- ❌ More overhead

**Decision tree:**
- Need to reference result set 2+ times? → CTE
- Result set is huge (millions of rows)? → Temp table with index
- Simple, one-time use? → Subquery
- Complex, multi-step logic? → CTE

---

## 3. Recursive CTEs {#recursive-ctes}

### What is Recursion in SQL?

A **recursive CTE** references itself, allowing you to query hierarchical or graph-like data.

**Syntax:**
```sql
WITH RECURSIVE cte_name AS (
    -- Base case (anchor)
    SELECT ...

    UNION ALL

    -- Recursive case
    SELECT ...
    FROM cte_name
    WHERE termination_condition
)
SELECT * FROM cte_name;
```

**Execution:**
1. Run base case → initial result set
2. Run recursive case using previous result
3. Repeat until no new rows (or max recursion reached)
4. Return all accumulated results

---

### Example 1: Generate Date Series

**Problem:** Generate all dates in January 2024

```sql
WITH RECURSIVE date_series AS (
    -- Base case: Start date
    SELECT DATE '2024-01-01' AS date

    UNION ALL

    -- Recursive case: Add 1 day
    SELECT date + INTERVAL '1 day'
    FROM date_series
    WHERE date < '2024-01-31'
)
SELECT date
FROM date_series;
```

**Result:**
```
date
-----------
2024-01-01
2024-01-02
2024-01-03
...
2024-01-31
```

**Execution trace:**
```
Iteration 0 (base): 2024-01-01
Iteration 1: 2024-01-01 + 1 day = 2024-01-02
Iteration 2: 2024-01-02 + 1 day = 2024-01-03
...
Iteration 30: 2024-01-30 + 1 day = 2024-01-31
Iteration 31: 2024-01-31 + 1 day = 2024-02-01, but WHERE fails → STOP
```

**Use case:** Fill gaps in time series data

---

### Example 2: Organizational Hierarchy

**Problem:** Employee reporting structure

**Data:**
```sql
CREATE TABLE employees (
    employee_id INT,
    name VARCHAR(100),
    manager_id INT
);

INSERT INTO employees VALUES
(1, 'Alice CEO', NULL),
(2, 'Bob VP', 1),
(3, 'Charlie VP', 1),
(4, 'David Manager', 2),
(5, 'Eve Manager', 2),
(6, 'Frank IC', 4),
(7, 'Grace IC', 4);
```

**Query:** Show full reporting chain for each employee

```sql
WITH RECURSIVE org_tree AS (
    -- Base case: Top-level (CEO)
    SELECT
        employee_id,
        name,
        manager_id,
        name AS reporting_chain,
        0 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case: Add direct reports
    SELECT
        e.employee_id,
        e.name,
        e.manager_id,
        ot.reporting_chain || ' → ' || e.name AS reporting_chain,
        ot.level + 1 AS level
    FROM employees e
    INNER JOIN org_tree ot ON e.manager_id = ot.employee_id
)
SELECT
    employee_id,
    name,
    level,
    reporting_chain
FROM org_tree
ORDER BY level, employee_id;
```

**Result:**
```
employee_id | name         | level | reporting_chain
------------|--------------|-------|----------------------------------
1           | Alice CEO    | 0     | Alice CEO
2           | Bob VP       | 1     | Alice CEO → Bob VP
3           | Charlie VP   | 1     | Alice CEO → Charlie VP
4           | David Mgr    | 2     | Alice CEO → Bob VP → David Mgr
5           | Eve Mgr      | 2     | Alice CEO → Bob VP → Eve Mgr
6           | Frank IC     | 3     | Alice CEO → Bob VP → David Mgr → Frank IC
7           | Grace IC     | 3     | Alice CEO → Bob VP → David Mgr → Grace IC
```

---

### Example 3: Graph Traversal (Friends of Friends)

**Problem:** Find all friends within 3 degrees of separation

**Data:**
```sql
CREATE TABLE friendships (
    user_id INT,
    friend_id INT
);
```

**Query:**
```sql
WITH RECURSIVE friend_network AS (
    -- Base case: Direct friends (1 degree)
    SELECT
        user_id,
        friend_id,
        1 AS degree
    FROM friendships
    WHERE user_id = 123  -- Starting user

    UNION

    -- Recursive case: Friends of friends
    SELECT
        fn.user_id,
        f.friend_id,
        fn.degree + 1
    FROM friend_network fn
    INNER JOIN friendships f ON fn.friend_id = f.user_id
    WHERE fn.degree < 3  -- Stop at 3 degrees
)
SELECT DISTINCT friend_id, MIN(degree) AS closest_degree
FROM friend_network
GROUP BY friend_id
ORDER BY closest_degree, friend_id;
```

---

### Recursive CTE Safeguards

**Prevent infinite loops:**

```sql
-- PostgreSQL: Set max recursion depth
WITH RECURSIVE cte AS (
    ...
)
SELECT * FROM cte
OPTION (MAXRECURSION 100);

-- Add termination condition
WHERE level < 10
```

**Common mistakes:**
- ❌ Forgetting WHERE termination condition
- ❌ Recursive part adds same rows infinitely
- ❌ Not using UNION (use UNION ALL for performance, but check for loops)

---

## 4. Complex Joins and Self-Joins {#complex-joins}

### Multi-Table Joins (5+ tables)

**Best practices:**

1. **Start with main table** (largest/most important)
2. **Join related tables progressively**
3. **Use meaningful aliases**
4. **Comment complex joins**

**Example:** E-commerce order details

```sql
SELECT
    o.order_id,
    o.order_date,
    c.customer_name,
    c.email,
    p.product_name,
    cat.category_name,
    oi.quantity,
    oi.unit_price,
    oi.quantity * oi.unit_price AS line_total,
    s.shipment_date,
    s.delivery_date,
    car.carrier_name
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
INNER JOIN categories cat ON p.category_id = cat.category_id
LEFT JOIN shipments s ON o.order_id = s.order_id
LEFT JOIN carriers car ON s.carrier_id = car.carrier_id
WHERE o.order_date >= '2024-01-01'
ORDER BY o.order_date DESC, o.order_id;
```

**Formatting tips:**
- One JOIN per line
- Align ON clauses
- Use LEFT JOIN for optional relationships
- Filter early (WHERE on main table)

---

### Self-Joins (Comparing Rows Within Same Table)

**Pattern 1: Compare with previous/next**

**Problem:** Find products with price increase

```sql
WITH price_changes AS (
    SELECT
        product_id,
        effective_date,
        price,
        LAG(price) OVER (
            PARTITION BY product_id
            ORDER BY effective_date
        ) AS previous_price
    FROM product_prices
)
SELECT
    product_id,
    effective_date,
    previous_price,
    price AS current_price,
    price - previous_price AS price_increase
FROM price_changes
WHERE price > previous_price;
```

**Pattern 2: Find pairs/combinations**

**Problem:** Find employees in same city

```sql
SELECT
    e1.name AS employee1,
    e2.name AS employee2,
    e1.city
FROM employees e1
INNER JOIN employees e2
    ON e1.city = e2.city
    AND e1.employee_id < e2.employee_id  -- Avoid duplicates (A-B and B-A)
ORDER BY e1.city, e1.name;
```

**Key:** `e1.employee_id < e2.employee_id` ensures each pair appears once

---

### Non-Equi Joins

**Join on conditions other than equality**

**Pattern: Overlapping time periods**

**Problem:** Find overlapping meeting room bookings

```sql
SELECT
    b1.booking_id AS booking1,
    b1.start_time AS start1,
    b1.end_time AS end1,
    b2.booking_id AS booking2,
    b2.start_time AS start2,
    b2.end_time AS end2
FROM bookings b1
INNER JOIN bookings b2
    ON b1.room_id = b2.room_id
    AND b1.booking_id < b2.booking_id  -- Avoid self and duplicate pairs
    AND b1.end_time > b2.start_time    -- Overlap condition
    AND b1.start_time < b2.end_time
ORDER BY b1.room_id, b1.start_time;
```

**Overlap logic:**
```
b1:     [-----]
b2: [-----]         ← start2 < end1 AND end2 > start1
b2:     [-----]     ← Overlaps
b2:       [-----]   ← Overlaps
b2:           [-----]  ← start2 >= end1, no overlap
```

---

## 5. Correlated Subqueries {#correlated-subqueries}

### What is a Correlated Subquery?

A subquery that references columns from the outer query. Runs once per outer row.

**Example:** Orders above customer's average

```sql
SELECT
    o.order_id,
    o.customer_id,
    o.amount
FROM orders o
WHERE o.amount > (
    SELECT AVG(amount)
    FROM orders o2
    WHERE o2.customer_id = o.customer_id  -- Correlated!
);
```

**Execution:**
```
For each row in orders:
    1. Get customer_id
    2. Calculate that customer's average
    3. Compare current order to average
    4. Include if greater
```

**Performance:** Can be slow (O(N²) in worst case)

---

### Converting to JOIN (Better Performance)

**Before (correlated subquery):**
```sql
SELECT
    o.order_id,
    o.customer_id,
    o.amount
FROM orders o
WHERE o.amount > (
    SELECT AVG(amount)
    FROM orders o2
    WHERE o2.customer_id = o.customer_id
);
```

**After (JOIN with CTE):**
```sql
WITH customer_avg AS (
    SELECT
        customer_id,
        AVG(amount) AS avg_amount
    FROM orders
    GROUP BY customer_id
)
SELECT
    o.order_id,
    o.customer_id,
    o.amount,
    ca.avg_amount
FROM orders o
INNER JOIN customer_avg ca
    ON o.customer_id = ca.customer_id
WHERE o.amount > ca.avg_amount;
```

**Much faster!** (O(N) vs O(N²))

---

### EXISTS vs IN with Correlated Subqueries

**EXISTS** is usually faster:

```sql
-- Good: EXISTS (stops at first match)
SELECT name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
    AND o.amount > 1000
);

-- Slower: IN (evaluates all)
SELECT name
FROM customers c
WHERE c.customer_id IN (
    SELECT customer_id
    FROM orders
    WHERE amount > 1000
);
```

---

## 6. CASE Statements and Conditional Logic {#case-statements}

### Simple CASE

**Equality check:**

```sql
SELECT
    order_id,
    status,
    CASE status
        WHEN 'pending' THEN 'Awaiting Processing'
        WHEN 'shipped' THEN 'In Transit'
        WHEN 'delivered' THEN 'Complete'
        ELSE 'Unknown'
    END AS status_label
FROM orders;
```

---

### Searched CASE (More Common)

**Multiple conditions:**

```sql
SELECT
    customer_id,
    total_spent,
    CASE
        WHEN total_spent >= 10000 THEN 'VIP'
        WHEN total_spent >= 5000 THEN 'Premium'
        WHEN total_spent >= 1000 THEN 'Standard'
        ELSE 'Basic'
    END AS tier,
    CASE
        WHEN total_spent >= 10000 THEN 0.20
        WHEN total_spent >= 5000 THEN 0.15
        WHEN total_spent >= 1000 THEN 0.10
        ELSE 0.05
    END AS discount_rate
FROM customer_totals;
```

---

### CASE in Aggregations (Pivot Pattern)

**Problem:** Count events by type (pivot)

```sql
SELECT
    user_id,
    COUNT(*) AS total_events,
    SUM(CASE WHEN event_type = 'login' THEN 1 ELSE 0 END) AS login_count,
    SUM(CASE WHEN event_type = 'purchase' THEN 1 ELSE 0 END) AS purchase_count,
    SUM(CASE WHEN event_type = 'logout' THEN 1 ELSE 0 END) AS logout_count,
    SUM(CASE WHEN event_type = 'error' THEN 1 ELSE 0 END) AS error_count
FROM events
GROUP BY user_id;
```

**Result:**
```
user_id | total_events | login_count | purchase_count | logout_count | error_count
--------|--------------|-------------|----------------|--------------|------------
1       | 10           | 2           | 3              | 2            | 3
2       | 8            | 1           | 5              | 1            | 1
```

**Pattern:** Transform rows to columns

---

### Nested CASE

```sql
SELECT
    product_id,
    category,
    price,
    CASE category
        WHEN 'Electronics' THEN
            CASE
                WHEN price > 1000 THEN 'High-End Electronics'
                WHEN price > 500 THEN 'Mid-Range Electronics'
                ELSE 'Budget Electronics'
            END
        WHEN 'Clothing' THEN
            CASE
                WHEN price > 200 THEN 'Designer Clothing'
                ELSE 'Regular Clothing'
            END
        ELSE 'Other'
    END AS product_segment
FROM products;
```

---

## 7. Date and Time Manipulation {#datetime}

### Extracting Date Parts

```sql
SELECT
    order_date,
    EXTRACT(YEAR FROM order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    EXTRACT(DAY FROM order_date) AS day,
    EXTRACT(DOW FROM order_date) AS day_of_week,  -- 0=Sunday
    EXTRACT(WEEK FROM order_date) AS week_of_year,
    EXTRACT(QUARTER FROM order_date) AS quarter
FROM orders;
```

**Alternative functions (PostgreSQL):**
```sql
DATE_PART('year', order_date)
```

---

### Date Arithmetic

```sql
SELECT
    order_date,
    -- Add/subtract
    order_date + INTERVAL '7 days' AS one_week_later,
    order_date - INTERVAL '1 month' AS one_month_ago,
    order_date + INTERVAL '2 hours 30 minutes' AS future_time,

    -- Difference
    current_date - order_date AS days_ago,
    AGE(current_date, order_date) AS time_ago,

    -- Extract epoch (seconds since 1970-01-01)
    EXTRACT(EPOCH FROM (current_timestamp - order_date)) / 86400 AS days_ago_decimal
FROM orders;
```

---

### Date Truncation (Group by Time Period)

```sql
-- Group by month
SELECT
    DATE_TRUNC('month', order_date) AS month,
    COUNT(*) AS orders,
    SUM(amount) AS revenue
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;

-- Other truncations
DATE_TRUNC('year', date)     -- 2024-01-01
DATE_TRUNC('quarter', date)  -- 2024-01-01, 2024-04-01, ...
DATE_TRUNC('week', date)     -- Monday of week
DATE_TRUNC('day', date)      -- Strips time
DATE_TRUNC('hour', timestamp)
```

---

### Date Ranges and Filtering

```sql
-- Last 30 days
WHERE order_date >= CURRENT_DATE - INTERVAL '30 days'

-- This month
WHERE DATE_TRUNC('month', order_date) = DATE_TRUNC('month', CURRENT_DATE)

-- This year
WHERE EXTRACT(YEAR FROM order_date) = EXTRACT(YEAR FROM CURRENT_DATE)

-- Between dates (inclusive)
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31'

-- Specific day of week (Monday)
WHERE EXTRACT(DOW FROM order_date) = 1
```

---

### Generate Time Series

```sql
SELECT generate_series(
    '2024-01-01'::timestamp,
    '2024-01-31'::timestamp,
    '1 day'::interval
) AS date;
```

---

### Common Date Patterns

**Pattern 1: Fill gaps in time series**

```sql
WITH date_series AS (
    SELECT generate_series(
        '2024-01-01'::date,
        '2024-01-31'::date,
        '1 day'::interval
    )::date AS date
)
SELECT
    ds.date,
    COALESCE(COUNT(o.order_id), 0) AS order_count,
    COALESCE(SUM(o.amount), 0) AS revenue
FROM date_series ds
LEFT JOIN orders o ON ds.date = o.order_date::date
GROUP BY ds.date
ORDER BY ds.date;
```

**Pattern 2: Cohort analysis (signup month)**

```sql
WITH user_cohorts AS (
    SELECT
        user_id,
        DATE_TRUNC('month', signup_date) AS cohort_month
    FROM users
),
user_activity AS (
    SELECT
        user_id,
        DATE_TRUNC('month', activity_date) AS activity_month
    FROM user_events
)
SELECT
    uc.cohort_month,
    ua.activity_month,
    COUNT(DISTINCT ua.user_id) AS active_users
FROM user_cohorts uc
LEFT JOIN user_activity ua
    ON uc.user_id = ua.user_id
    AND ua.activity_month >= uc.cohort_month
GROUP BY uc.cohort_month, ua.activity_month
ORDER BY uc.cohort_month, ua.activity_month;
```

---

## 8. String Functions and Pattern Matching {#string-functions}

### Common String Functions

```sql
SELECT
    -- Case conversion
    UPPER(name) AS uppercase,
    LOWER(email) AS lowercase,
    INITCAP(name) AS proper_case,

    -- Trimming
    TRIM(name) AS trimmed,
    LTRIM(name) AS left_trimmed,
    RTRIM(name) AS right_trimmed,
    TRIM(BOTH ' ' FROM name) AS trim_spaces,

    -- Substring
    SUBSTRING(email FROM 1 FOR 5) AS first_5,
    LEFT(email, 5) AS left_5,
    RIGHT(email, 4) AS right_4,

    -- Length and position
    LENGTH(name) AS length,
    POSITION('@' IN email) AS at_position,

    -- Replace
    REPLACE(phone, '-', '') AS phone_no_dashes,
    REPLACE(name, 'Jr.', 'Junior') AS expanded_name,

    -- Concatenation
    name || ' ' || surname AS full_name,
    CONCAT(name, ' ', surname) AS full_name_alt,
    CONCAT_WS(', ', name, city, country) AS comma_separated,

    -- Splitting
    SPLIT_PART(email, '@', 2) AS domain,
    STRING_TO_ARRAY(tags, ',') AS tag_array
FROM users;
```

---

### Pattern Matching with LIKE

```sql
-- Wildcard patterns
WHERE name LIKE 'A%'           -- Starts with A
WHERE name LIKE '%son'         -- Ends with son
WHERE name LIKE '%John%'       -- Contains John
WHERE name LIKE '_____'        -- Exactly 5 characters
WHERE name LIKE 'A%e'          -- Starts with A, ends with e

-- Case insensitive (PostgreSQL)
WHERE name ILIKE 'alice'

-- Multiple patterns
WHERE email LIKE '%@gmail.com' OR email LIKE '%@yahoo.com'

-- NOT LIKE
WHERE name NOT LIKE 'Test%'
```

---

### Regular Expressions (SIMILAR TO, ~)

**PostgreSQL regex operators:**
- `~` - Matches regex (case-sensitive)
- `~*` - Matches regex (case-insensitive)
- `!~` - Does not match regex
- `SIMILAR TO` - SQL standard regex

**Examples:**

```sql
-- Email validation
WHERE email ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$'

-- Phone number (###-###-####)
WHERE phone ~ '^\d{3}-\d{3}-\d{4}$'

-- Extract domain from email
SELECT
    email,
    (regexp_match(email, '@(.+)$'))[1] AS domain
FROM users;

-- Find all emails with numbers
WHERE email ~ '[0-9]'

-- Extract all numbers from string
SELECT regexp_matches('Order #12345 total $67.89', '\d+', 'g');
```

---

### String Aggregation

**Combine multiple rows into one string:**

```sql
-- Aggregate tags per user
SELECT
    user_id,
    STRING_AGG(tag, ', ' ORDER BY tag) AS all_tags,
    ARRAY_AGG(tag ORDER BY tag) AS tag_array
FROM user_tags
GROUP BY user_id;
```

**Result:**
```
user_id | all_tags              | tag_array
--------|----------------------|------------------------
1       | coding, data, python | {coding,data,python}
2       | design, ux           | {design,ux}
```

---

## 9. JSON and Array Operations {#json-arrays}

### JSON Functions (PostgreSQL)

**Sample data:**
```sql
CREATE TABLE events (
    event_id INT,
    user_id INT,
    event_data JSONB
);

INSERT INTO events VALUES
(1, 100, '{"type": "click", "page": "home", "timestamp": "2024-01-15T10:30:00"}'),
(2, 100, '{"type": "purchase", "amount": 49.99, "items": [{"id": 1, "qty": 2}]}');
```

**Accessing JSON fields:**

```sql
SELECT
    event_id,
    -- Extract as text
    event_data->>'type' AS event_type,
    event_data->>'page' AS page,

    -- Extract as JSON
    event_data->'items' AS items_json,

    -- Extract nested
    event_data->'items'->0->>'id' AS first_item_id,

    -- Extract with path
    event_data#>>'{items,0,id}' AS first_item_id_alt
FROM events;
```

**JSON functions:**

```sql
-- Check if key exists
WHERE event_data ? 'amount'

-- Get all keys
SELECT jsonb_object_keys(event_data) FROM events;

-- Build JSON
SELECT jsonb_build_object(
    'user_id', user_id,
    'total', SUM(amount)
) AS user_summary
FROM orders
GROUP BY user_id;

-- Aggregate to JSON
SELECT
    user_id,
    jsonb_agg(event_data) AS all_events
FROM events
GROUP BY user_id;
```

---

### Array Operations

```sql
-- Array construction
SELECT ARRAY[1, 2, 3] AS numbers;
SELECT ARRAY_AGG(product_id) AS product_ids FROM orders;

-- Array access
SELECT tags[1] AS first_tag FROM products;  -- 1-indexed!

-- Array functions
SELECT
    array_length(tags, 1) AS tag_count,
    'python' = ANY(tags) AS has_python_tag,
    tags @> ARRAY['python'] AS contains_python,
    tags && ARRAY['python', 'java'] AS overlaps
FROM products;

-- Unnest (array to rows)
SELECT
    product_id,
    unnest(tags) AS tag
FROM products;
```

---

## 10. Query Optimization {#optimization}

### Understanding Query Execution

**EXPLAIN ANALYZE:**

```sql
EXPLAIN ANALYZE
SELECT
    c.name,
    COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.name;
```

**Look for:**
- Sequential Scan vs Index Scan
- Nested Loop vs Hash Join
- Execution time
- Rows processed

---

### Optimization Techniques

#### 1. Use Indexes

```sql
-- Create index on commonly filtered/joined columns
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_orders_date ON orders(order_date);

-- Composite index
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- Partial index (filtered)
CREATE INDEX idx_active_orders ON orders(order_date)
WHERE status = 'active';
```

**When indexes help:**
- ✅ WHERE clauses
- ✅ JOIN keys
- ✅ ORDER BY columns
- ✅ Columns in window PARTITION BY/ORDER BY

**When indexes don't help:**
- ❌ Small tables (< 1000 rows)
- ❌ Columns with low cardinality (few distinct values)
- ❌ Using functions on indexed column

---

#### 2. SELECT Only Needed Columns

```sql
-- Bad
SELECT * FROM large_table WHERE id = 123;

-- Good
SELECT id, name, email FROM large_table WHERE id = 123;
```

---

#### 3. Filter Early

```sql
-- Bad: Filter after aggregation
SELECT *
FROM (
    SELECT category, COUNT(*) AS cnt
    FROM products
    GROUP BY category
) subq
WHERE cnt > 100;

-- Good: Filter before aggregation
SELECT category, COUNT(*) AS cnt
FROM products
WHERE active = TRUE  -- Filter early!
GROUP BY category
HAVING COUNT(*) > 100;
```

---

#### 4. Use EXISTS Instead of IN for Subqueries

```sql
-- Slower
SELECT * FROM customers
WHERE customer_id IN (
    SELECT customer_id FROM orders WHERE amount > 1000
);

-- Faster
SELECT * FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.customer_id = c.customer_id AND o.amount > 1000
);
```

---

#### 5. Avoid Functions on Indexed Columns

```sql
-- Bad: Can't use index
WHERE UPPER(name) = 'ALICE'
WHERE DATE(created_at) = '2024-01-15'

-- Good: Index can be used
WHERE name = 'Alice'  -- Or create functional index
WHERE created_at >= '2024-01-15' AND created_at < '2024-01-16'
```

---

#### 6. Use UNION ALL Instead of UNION

```sql
-- Slower: Checks for duplicates
SELECT * FROM table1
UNION
SELECT * FROM table2;

-- Faster: Keeps duplicates
SELECT * FROM table1
UNION ALL
SELECT * FROM table2;
```

---

#### 7. Partition Large Tables

```sql
-- Partition by date range
CREATE TABLE orders_2024_01 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

-- Queries only scan relevant partitions
SELECT * FROM orders WHERE order_date = '2024-01-15';
-- Only scans orders_2024_01 partition
```

---

## 11. Practice Problems {#practice-problems}

### Problem 1: "The Long Tail" (Medium)

**Scenario:** Calculate 90th percentile of API latency per endpoint

**Table:** `api_calls` (endpoint, latency_ms, timestamp)

**Task:** Find endpoints where 90th percentile latency > 100ms

**Hints:**
- Use PERCENTILE_CONT
- GROUP BY endpoint
- HAVING to filter

---

### Problem 2: User Retention (Medium)

**Scenario:** Calculate 7-day retention rate by signup cohort

**Tables:** `users` (user_id, signup_date), `user_activity` (user_id, activity_date)

**Task:** For each signup week, what % of users were active 7 days later?

**Hints:**
- DATE_TRUNC to get weeks
- LEFT JOIN activity on day 7
- Calculate percentage

---

### Problem 3: Hierarchical Data (Medium)

**Scenario:** Category tree (Electronics → Computers → Laptops)

**Table:** `categories` (category_id, name, parent_id)

**Task:** Show full category path for each category

**Hints:**
- Recursive CTE
- Build path string
- Start from root (parent_id IS NULL)

---

## 12. Solutions {#solutions}

### Solution 1: "The Long Tail"

```sql
WITH endpoint_stats AS (
    SELECT
        endpoint,
        PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY latency_ms) AS p90_latency,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY latency_ms) AS p50_latency,
        AVG(latency_ms) AS avg_latency,
        COUNT(*) AS request_count
    FROM api_calls
    WHERE timestamp >= CURRENT_DATE - INTERVAL '7 days'
    GROUP BY endpoint
)
SELECT
    endpoint,
    ROUND(p90_latency, 2) AS p90_latency_ms,
    ROUND(p50_latency, 2) AS median_latency_ms,
    ROUND(avg_latency, 2) AS avg_latency_ms,
    request_count,
    ROUND(p90_latency - p50_latency, 2) AS tail_gap
FROM endpoint_stats
WHERE p90_latency > 100
ORDER BY p90_latency DESC;
```

---

### Solution 2: User Retention

```sql
WITH
signup_cohorts AS (
    SELECT
        user_id,
        DATE_TRUNC('week', signup_date) AS cohort_week,
        signup_date
    FROM users
),
day7_activity AS (
    SELECT
        sc.cohort_week,
        sc.user_id,
        CASE
            WHEN ua.activity_date IS NOT NULL THEN 1
            ELSE 0
        END AS was_active_day7
    FROM signup_cohorts sc
    LEFT JOIN user_activity ua
        ON sc.user_id = ua.user_id
        AND ua.activity_date = sc.signup_date + INTERVAL '7 days'
)
SELECT
    cohort_week,
    COUNT(*) AS users_in_cohort,
    SUM(was_active_day7) AS active_day7,
    ROUND(100.0 * SUM(was_active_day7) / COUNT(*), 2) AS retention_rate_pct
FROM day7_activity
GROUP BY cohort_week
ORDER BY cohort_week;
```

---

### Solution 3: Category Paths

```sql
WITH RECURSIVE category_paths AS (
    -- Base: Root categories
    SELECT
        category_id,
        name,
        parent_id,
        name AS full_path,
        1 AS level
    FROM categories
    WHERE parent_id IS NULL

    UNION ALL

    -- Recursive: Child categories
    SELECT
        c.category_id,
        c.name,
        c.parent_id,
        cp.full_path || ' → ' || c.name AS full_path,
        cp.level + 1 AS level
    FROM categories c
    INNER JOIN category_paths cp ON c.parent_id = cp.category_id
)
SELECT
    category_id,
    name,
    level,
    full_path
FROM category_paths
ORDER BY full_path;
```

---

## 13. Summary {#summary}

### Key Concepts Mastered

✅ **CTEs:** Write readable, maintainable queries
✅ **Recursive CTEs:** Handle hierarchies and graphs
✅ **Complex Joins:** Multi-table, self-joins, non-equi
✅ **CASE Statements:** Conditional logic and pivoting
✅ **Date/Time:** Manipulation, truncation, series generation
✅ **String Functions:** Pattern matching, regex, aggregation
✅ **JSON:** Extract, query, and aggregate JSON data
✅ **Optimization:** Indexes, execution plans, best practices

### Self-Assessment

- [ ] Can write multi-step queries with CTEs?
- [ ] Understand when to use recursive CTEs?
- [ ] Comfortable with complex joins (5+ tables)?
- [ ] Can manipulate dates effectively?
- [ ] Know how to optimize slow queries?
- [ ] Can read EXPLAIN ANALYZE output?

### Next Steps

**You're ready for Python!**

**Chapter 4: Python Fundamentals**

**Practice goals:**
- [ ] 25-30 medium SQL problems on DataDriven
- [ ] Focus on CTEs and date manipulation
- [ ] Average time: 25-30 min per problem
- [ ] Success rate: 75%+

**Congratulations on completing Advanced SQL!** 🎉

---
