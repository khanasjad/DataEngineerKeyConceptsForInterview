# 50 SQL Interview Questions with Answers - Quick Reference

**Condensed format for fast review and practice**

---

## Q1: Find the Second Highest Salary

**Question:** Write a SQL query to find the second highest salary from an Employee table.

**Answer:**
```sql
SELECT DISTINCT salary AS second_highest_salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as rank
    FROM employees
) ranked
WHERE rank = 2;
```

---

## Q2: Employees Earning More Than Their Managers

**Question:** Find employees who earn more than their managers.

**Answer:**
```sql
SELECT e1.name AS employee
FROM employees e1
INNER JOIN employees e2 ON e1.manager_id = e2.id
WHERE e1.salary > e2.salary;
```

---

## Q3: Customers Who Never Ordered

**Question:** Find all customers who never placed an order.

**Answer:**
```sql
SELECT c.name
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL;
```

---

## Q4: Find Duplicate Emails

**Question:** Find all duplicate emails in a Person table.

**Answer:**
```sql
SELECT email
FROM person
GROUP BY email
HAVING COUNT(*) > 1;
```

---

## Q5: Delete Duplicates (Keep First)

**Question:** Delete duplicate emails, keeping only the record with the lowest ID.

**Answer:**
```sql
DELETE FROM users
WHERE id IN (
    SELECT id FROM (
        SELECT id, ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS rn
        FROM users
    ) t
    WHERE rn > 1
);
```

---

## Q6: Nth Highest Salary

**Question:** Write a query to find the Nth highest salary.

**Answer:**
```sql
-- For N = 3
SELECT DISTINCT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as rank
    FROM employees
) ranked
WHERE rank = 3;
```

---

## Q7: Department with Highest Average Salary

**Question:** Find the department with the highest average salary.

**Answer:**
```sql
SELECT dept, AVG(salary) as avg_salary
FROM employees
GROUP BY dept
ORDER BY avg_salary DESC
LIMIT 1;
```

---

## Q8: Consecutive Numbers

**Question:** Find all numbers that appear at least three times consecutively.

**Answer:**
```sql
SELECT DISTINCT l1.num AS ConsecutiveNums
FROM logs l1
JOIN logs l2 ON l1.id = l2.id - 1 AND l1.num = l2.num
JOIN logs l3 ON l2.id = l3.id - 1 AND l2.num = l3.num;
```

---

## Q9: Rising Temperature

**Question:** Find dates with temperature higher than previous day.

**Answer:**
```sql
SELECT w1.date
FROM weather w1
JOIN weather w2 ON w1.date = DATE_ADD(w2.date, INTERVAL 1 DAY)
WHERE w1.temperature > w2.temperature;
```

---

## Q10: Rank Scores

**Question:** Rank scores from highest to lowest. If there is a tie, assign same rank.

**Answer:**
```sql
SELECT
    score,
    DENSE_RANK() OVER (ORDER BY score DESC) as rank
FROM scores;
```

---

## Q11: Top 3 Salaries per Department

**Question:** Find the top 3 highest-paid employees in each department.

**Answer:**
```sql
SELECT dept, name, salary
FROM (
    SELECT
        dept,
        name,
        salary,
        DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC) as rank
    FROM employees
) ranked
WHERE rank <= 3;
```

---

## Q12: Running Total

**Question:** Calculate running total of daily sales.

**Answer:**
```sql
SELECT
    date,
    sales,
    SUM(sales) OVER (ORDER BY date) AS running_total
FROM daily_sales;
```

---

## Q13: Moving Average (7-day)

**Question:** Calculate 7-day moving average of sales.

**Answer:**
```sql
SELECT
    date,
    sales,
    AVG(sales) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7day
FROM daily_sales;
```

---

## Q14: Month-over-Month Growth

**Question:** Calculate month-over-month revenue growth percentage.

**Answer:**
```sql
SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue,
    ROUND((revenue - LAG(revenue) OVER (ORDER BY month)) * 100.0 /
          LAG(revenue) OVER (ORDER BY month), 2) AS mom_growth_pct
FROM monthly_revenue;
```

---

## Q15: Year-over-Year Growth

**Question:** Calculate year-over-year revenue growth.

**Answer:**
```sql
SELECT
    year,
    month,
    revenue,
    LAG(revenue, 12) OVER (ORDER BY year, month) AS prev_year_revenue,
    ROUND((revenue - LAG(revenue, 12) OVER (ORDER BY year, month)) * 100.0 /
          LAG(revenue, 12) OVER (ORDER BY year, month), 2) AS yoy_growth_pct
FROM monthly_revenue;
```

---

## Q16: First and Last Purchase Date

**Question:** Find first and last purchase date for each customer.

**Answer:**
```sql
SELECT
    customer_id,
    MIN(order_date) AS first_purchase,
    MAX(order_date) AS last_purchase,
    COUNT(*) AS total_orders
FROM orders
GROUP BY customer_id;
```

---

## Q17: Customers with Consecutive Purchases

**Question:** Find customers who made purchases on consecutive days.

**Answer:**
```sql
SELECT DISTINCT customer_id
FROM (
    SELECT
        customer_id,
        order_date,
        LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS prev_date
    FROM orders
) t
WHERE DATEDIFF(order_date, prev_date) = 1;
```

---

## Q18: Gaps in Dates

**Question:** Find missing dates in a sequence.

**Answer:**
```sql
WITH RECURSIVE date_range AS (
    SELECT MIN(date) AS date FROM sales
    UNION ALL
    SELECT DATE_ADD(date, INTERVAL 1 DAY)
    FROM date_range
    WHERE date < (SELECT MAX(date) FROM sales)
)
SELECT dr.date
FROM date_range dr
LEFT JOIN sales s ON dr.date = s.date
WHERE s.date IS NULL;
```

---

## Q19: Active Users (DAU/MAU)

**Question:** Calculate Daily Active Users and Monthly Active Users.

**Answer:**
```sql
SELECT
    date,
    COUNT(DISTINCT user_id) AS dau,
    COUNT(DISTINCT user_id) OVER (
        ORDER BY date
        ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
    ) AS mau
FROM user_activity
GROUP BY date;
```

---

## Q20: Retention Rate

**Question:** Calculate user retention rate by cohort.

**Answer:**
```sql
WITH cohorts AS (
    SELECT user_id, DATE_TRUNC('month', signup_date) AS cohort_month
    FROM users
),
activity AS (
    SELECT
        c.cohort_month,
        DATE_TRUNC('month', a.activity_date) AS activity_month,
        COUNT(DISTINCT a.user_id) AS active_users
    FROM user_activity a
    JOIN cohorts c ON a.user_id = c.user_id
    GROUP BY c.cohort_month, activity_month
)
SELECT
    cohort_month,
    activity_month,
    active_users,
    ROUND(active_users * 100.0 / FIRST_VALUE(active_users) OVER (
        PARTITION BY cohort_month ORDER BY activity_month
    ), 2) AS retention_rate
FROM activity;
```

---

## Q21: Cumulative Distribution

**Question:** Calculate cumulative percentage of total sales.

**Answer:**
```sql
SELECT
    product_id,
    sales,
    SUM(sales) OVER (ORDER BY sales DESC) AS cumulative_sales,
    ROUND(SUM(sales) OVER (ORDER BY sales DESC) * 100.0 /
          SUM(sales) OVER (), 2) AS cumulative_pct
FROM product_sales;
```

---

## Q22: Median Value

**Question:** Calculate median salary by department.

**Answer:**
```sql
SELECT
    dept,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median_salary
FROM employees
GROUP BY dept;
```

---

## Q23: Self-JOIN for Pairs

**Question:** Find all pairs of employees in the same department.

**Answer:**
```sql
SELECT
    e1.name AS employee1,
    e2.name AS employee2,
    e1.dept
FROM employees e1
JOIN employees e2 ON e1.dept = e2.dept AND e1.id < e2.id;
```

---

## Q24: Products Never Sold

**Question:** Find products that were never sold.

**Answer:**
```sql
SELECT p.product_name
FROM products p
WHERE NOT EXISTS (
    SELECT 1 FROM order_items oi WHERE oi.product_id = p.id
);
```

---

## Q25: Customer Lifetime Value

**Question:** Calculate total spending per customer (LTV).

**Answer:**
```sql
SELECT
    customer_id,
    COUNT(*) AS total_orders,
    SUM(amount) AS lifetime_value,
    AVG(amount) AS avg_order_value
FROM orders
GROUP BY customer_id
ORDER BY lifetime_value DESC;
```

---

## Q26: Products Bought Together

**Question:** Find products frequently bought together.

**Answer:**
```sql
SELECT
    oi1.product_id AS product_a,
    oi2.product_id AS product_b,
    COUNT(*) AS times_bought_together
FROM order_items oi1
JOIN order_items oi2
    ON oi1.order_id = oi2.order_id
    AND oi1.product_id < oi2.product_id
GROUP BY oi1.product_id, oi2.product_id
HAVING COUNT(*) >= 5
ORDER BY times_bought_together DESC;
```

---

## Q27: Sessionization

**Question:** Group user events into sessions (30-minute timeout).

**Answer:**
```sql
WITH gaps AS (
    SELECT
        user_id,
        event_time,
        CASE
            WHEN TIMESTAMPDIFF(MINUTE,
                LAG(event_time) OVER (PARTITION BY user_id ORDER BY event_time),
                event_time) > 30
            THEN 1
            ELSE 0
        END AS new_session
    FROM events
)
SELECT
    user_id,
    SUM(new_session) OVER (PARTITION BY user_id ORDER BY event_time) AS session_id,
    event_time
FROM gaps;
```

---

## Q28: Pivot Table

**Question:** Pivot monthly sales by year (columns).

**Answer:**
```sql
SELECT
    month,
    SUM(CASE WHEN year = 2023 THEN sales ELSE 0 END) AS sales_2023,
    SUM(CASE WHEN year = 2024 THEN sales ELSE 0 END) AS sales_2024,
    SUM(CASE WHEN year = 2025 THEN sales ELSE 0 END) AS sales_2025
FROM monthly_sales
GROUP BY month;
```

---

## Q29: Unpivot Table

**Question:** Convert columns to rows.

**Answer:**
```sql
SELECT customer_id, 'Jan' AS month, jan_sales AS sales FROM sales_wide
UNION ALL
SELECT customer_id, 'Feb', feb_sales FROM sales_wide
UNION ALL
SELECT customer_id, 'Mar', mar_sales FROM sales_wide;
```

---

## Q30: Running Difference

**Question:** Calculate difference from previous row.

**Answer:**
```sql
SELECT
    date,
    value,
    value - LAG(value) OVER (ORDER BY date) AS difference
FROM daily_metrics;
```

---

## Q31: Percent Change

**Question:** Calculate percent change from previous period.

**Answer:**
```sql
SELECT
    period,
    revenue,
    ROUND((revenue - LAG(revenue) OVER (ORDER BY period)) * 100.0 /
          LAG(revenue) OVER (ORDER BY period), 2) AS pct_change
FROM quarterly_revenue;
```

---

## Q32: First Value in Group

**Question:** Get first order date for each customer within their orders.

**Answer:**
```sql
SELECT
    customer_id,
    order_date,
    FIRST_VALUE(order_date) OVER (
        PARTITION BY customer_id ORDER BY order_date
    ) AS first_order_date
FROM orders;
```

---

## Q33: Recursive CTE - Hierarchy

**Question:** Show organizational hierarchy from CEO down.

**Answer:**
```sql
WITH RECURSIVE org_tree AS (
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    SELECT e.id, e.name, e.manager_id, ot.level + 1
    FROM employees e
    JOIN org_tree ot ON e.manager_id = ot.id
)
SELECT * FROM org_tree ORDER BY level, name;
```

---

## Q34: Recursive CTE - Number Series

**Question:** Generate numbers from 1 to 100.

**Answer:**
```sql
WITH RECURSIVE numbers AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM numbers WHERE n < 100
)
SELECT n FROM numbers;
```

---

## Q35: Dense vs Sparse Ranking

**Question:** Show difference between RANK and DENSE_RANK.

**Answer:**
```sql
SELECT
    name,
    score,
    RANK() OVER (ORDER BY score DESC) AS rank_with_gaps,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM exam_scores;
```

---

## Q36: NTILE for Quartiles

**Question:** Divide customers into 4 quartiles by spending.

**Answer:**
```sql
SELECT
    customer_id,
    total_spent,
    NTILE(4) OVER (ORDER BY total_spent DESC) AS quartile
FROM customer_spending;
```

---

## Q37: String Aggregation

**Question:** Concatenate multiple rows into comma-separated list.

**Answer:**
```sql
-- PostgreSQL
SELECT
    student_id,
    STRING_AGG(course, ', ' ORDER BY course) AS courses
FROM enrollments
GROUP BY student_id;

-- MySQL
SELECT
    student_id,
    GROUP_CONCAT(course ORDER BY course SEPARATOR ', ') AS courses
FROM enrollments
GROUP BY student_id;
```

---

## Q38: Conditional Aggregation

**Question:** Count orders by status in columns.

**Answer:**
```sql
SELECT
    customer_id,
    COUNT(CASE WHEN status = 'completed' THEN 1 END) AS completed,
    COUNT(CASE WHEN status = 'pending' THEN 1 END) AS pending,
    COUNT(CASE WHEN status = 'cancelled' THEN 1 END) AS cancelled
FROM orders
GROUP BY customer_id;
```

---

## Q39: Multiple Aggregations

**Question:** Get various statistics for each department.

**Answer:**
```sql
SELECT
    dept,
    COUNT(*) AS num_employees,
    AVG(salary) AS avg_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary,
    STDDEV(salary) AS salary_stddev
FROM employees
GROUP BY dept;
```

---

## Q40: HAVING vs WHERE

**Question:** Filter groups vs individual rows.

**Answer:**
```sql
SELECT
    dept,
    AVG(salary) AS avg_salary
FROM employees
WHERE salary > 40000  -- Filter rows BEFORE grouping
GROUP BY dept
HAVING AVG(salary) > 60000;  -- Filter groups AFTER aggregation
```

---

## Q41: Date Difference in Days

**Question:** Calculate days between order and shipment.

**Answer:**
```sql
SELECT
    order_id,
    order_date,
    ship_date,
    DATEDIFF(ship_date, order_date) AS days_to_ship
FROM orders;
```

---

## Q42: Extract Date Parts

**Question:** Extract year, month, day from timestamp.

**Answer:**
```sql
SELECT
    order_timestamp,
    EXTRACT(YEAR FROM order_timestamp) AS year,
    EXTRACT(MONTH FROM order_timestamp) AS month,
    EXTRACT(DAY FROM order_timestamp) AS day,
    EXTRACT(HOUR FROM order_timestamp) AS hour
FROM orders;
```

---

## Q43: Date Truncation

**Question:** Truncate timestamp to day/month/year.

**Answer:**
```sql
SELECT
    event_timestamp,
    DATE_TRUNC('day', event_timestamp) AS day,
    DATE_TRUNC('month', event_timestamp) AS month,
    DATE_TRUNC('year', event_timestamp) AS year
FROM events;
```

---

## Q44: Last N Days

**Question:** Get orders from last 7 days.

**Answer:**
```sql
SELECT *
FROM orders
WHERE order_date >= CURRENT_DATE - INTERVAL '7 days';
```

---

## Q45: Business Days

**Question:** Count business days (exclude weekends) between dates.

**Answer:**
```sql
SELECT
    DATEDIFF(end_date, start_date) + 1
    - (FLOOR(DATEDIFF(end_date, start_date) / 7) * 2) AS business_days
FROM date_ranges;
```

---

## Q46: Age Calculation

**Question:** Calculate age from date of birth.

**Answer:**
```sql
SELECT
    name,
    date_of_birth,
    TIMESTAMPDIFF(YEAR, date_of_birth, CURRENT_DATE) AS age
FROM customers;
```

---

## Q47: NULL Handling

**Question:** Replace NULL with default value.

**Answer:**
```sql
SELECT
    name,
    COALESCE(phone, 'No phone') AS phone,
    COALESCE(email, 'No email') AS email,
    IFNULL(address, 'No address') AS address
FROM customers;
```

---

## Q48: CASE for Categorization

**Question:** Categorize customers by age group.

**Answer:**
```sql
SELECT
    name,
    age,
    CASE
        WHEN age < 18 THEN 'Minor'
        WHEN age BETWEEN 18 AND 30 THEN 'Young Adult'
        WHEN age BETWEEN 31 AND 50 THEN 'Adult'
        ELSE 'Senior'
    END AS age_group
FROM customers;
```

---

## Q49: Correlated Subquery

**Question:** Find employees earning more than department average.

**Answer:**
```sql
SELECT name, dept, salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.dept = e1.dept
);
```

---

## Q50: Window Function Alternative to Subquery

**Question:** Optimize correlated subquery with window function.

**Answer:**
```sql
-- Instead of correlated subquery (slow):
-- WHERE salary > (SELECT AVG(salary) FROM employees e2 WHERE e2.dept = e1.dept)

-- Use window function (fast):
SELECT name, dept, salary, dept_avg
FROM (
    SELECT
        name,
        dept,
        salary,
        AVG(salary) OVER (PARTITION BY dept) AS dept_avg
    FROM employees
) t
WHERE salary > dept_avg;
```

---

## 🎯 QUICK REFERENCE GUIDE

### Window Function Syntax
```sql
<FUNCTION> OVER (
    [PARTITION BY column]
    [ORDER BY column]
    [ROWS/RANGE BETWEEN ... AND ...]
)
```

### Common Window Functions
- **ROW_NUMBER()** - Unique sequential number
- **RANK()** - Rank with gaps for ties
- **DENSE_RANK()** - Rank without gaps
- **NTILE(n)** - Divide into n buckets
- **LAG(col, n)** - Value from n rows before
- **LEAD(col, n)** - Value from n rows ahead
- **FIRST_VALUE()** - First value in window
- **LAST_VALUE()** - Last value in window

### Frame Clauses
```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW  -- Running total
ROWS BETWEEN 6 PRECEDING AND CURRENT ROW          -- 7-day window
ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING  -- Remaining rows
```

### CTE Syntax
```sql
WITH cte1 AS (SELECT ...),
     cte2 AS (SELECT ... FROM cte1)
SELECT ... FROM cte2;
```

### Recursive CTE
```sql
WITH RECURSIVE cte AS (
    SELECT ... -- Base case
    UNION ALL
    SELECT ... FROM cte -- Recursive case
)
SELECT ... FROM cte;
```

### Join Types
- **INNER JOIN** - Only matching rows
- **LEFT JOIN** - All from left + matches from right
- **RIGHT JOIN** - All from right + matches from left
- **FULL OUTER JOIN** - All rows from both tables
- **CROSS JOIN** - Cartesian product

### Date Functions
```sql
CURRENT_DATE / CURRENT_TIMESTAMP / NOW()
DATE_ADD(date, INTERVAL n DAY/MONTH/YEAR)
DATE_SUB(date, INTERVAL n DAY/MONTH/YEAR)
DATEDIFF(date1, date2)
DATE_TRUNC('day/month/year', timestamp)
EXTRACT(YEAR/MONTH/DAY FROM date)
```

### Aggregate Functions
```sql
COUNT(*) / COUNT(column) / COUNT(DISTINCT column)
SUM() / AVG() / MIN() / MAX()
STRING_AGG(column, separator)  -- PostgreSQL
GROUP_CONCAT(column SEPARATOR ',')  -- MySQL
PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY column)
```

### NULL Handling
```sql
COALESCE(col1, col2, 'default')  -- First non-NULL
IFNULL(column, 'default')        -- MySQL
NULLIF(col1, col2)               -- NULL if equal
```

---

## 📚 Study Tips

### By Difficulty
**Easy (Practice First):** Q1-Q10, Q16, Q21, Q25, Q41-Q44, Q47-Q48
**Medium (Core Skills):** Q11-Q15, Q17-Q19, Q22-Q24, Q26-Q32, Q37-Q40, Q45-Q46, Q49
**Hard (Advanced):** Q20, Q27, Q33-Q36, Q50

### By Topic
**JOINs:** Q2, Q3, Q8, Q9, Q23, Q24, Q26
**Window Functions:** Q10-Q15, Q20-Q22, Q32, Q35-Q36, Q50
**Aggregations:** Q4, Q7, Q21, Q25, Q37-Q40
**Date/Time:** Q9, Q16-Q19, Q41-Q46
**CTEs:** Q18, Q20, Q27, Q33-Q34
**Subqueries:** Q24, Q49, Q50
**Data Manipulation:** Q5, Q28-Q31, Q47-Q48

### Practice Plan
- **Week 1:** Q1-15 (Basics + Window Functions)
- **Week 2:** Q16-30 (Aggregations + Date/Time)
- **Week 3:** Q31-45 (CTEs + Advanced)
- **Week 4:** Q46-50 + Review all

---

## 💡 Interview Tips

1. **Clarify first** - Ask about NULL handling, data volume, expected output format
2. **Explain approach** - Talk through logic before writing SQL
3. **Start simple** - Write basic query first, then optimize
4. **Test edge cases** - Consider NULL, empty results, duplicates
5. **Discuss performance** - Mention indexes, explain plans, alternatives
6. **Know your database** - Syntax varies (MySQL vs PostgreSQL vs BigQuery)

---

## 🚀 Common Optimization Techniques

### Use Window Functions Instead of Correlated Subqueries
❌ Slow: `WHERE salary > (SELECT AVG(salary) FROM emp e2 WHERE e2.dept = e1.dept)`
✅ Fast: `AVG(salary) OVER (PARTITION BY dept)`

### Use EXISTS Instead of IN (for large datasets)
❌ Slow: `WHERE id IN (SELECT customer_id FROM orders)`
✅ Fast: `WHERE EXISTS (SELECT 1 FROM orders WHERE customer_id = c.id)`

### Avoid Functions on Indexed Columns
❌ Slow: `WHERE YEAR(order_date) = 2024`
✅ Fast: `WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01'`

### Use UNION ALL Instead of UNION (if duplicates ok)
❌ Slower: `SELECT ... UNION SELECT ...` (removes duplicates)
✅ Faster: `SELECT ... UNION ALL SELECT ...` (keeps all)

### Use Specific Columns Instead of SELECT *
❌ Slow: `SELECT * FROM orders`
✅ Fast: `SELECT id, customer_id, total FROM orders`

---

**You're now ready for SQL interviews! Practice these 50 questions and you'll be confident in any data engineering interview. Good luck! 🎯**
