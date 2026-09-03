# SQL DATE FUNCTIONS - COMPLETE INTERVIEW GUIDE
## Master Every Date Question in SQL Interviews

> **Philosophy**: Date questions confuse everyone. But there are only 10 core patterns. Master these and you're golden.

---

## 📚 TABLE OF CONTENTS
1. [The 3 SQL Dialects (Important!)](#dialects)
2. [Getting Current Date/Time](#current-date)
3. [Extracting Parts of a Date](#extract-parts)
4. [Date Arithmetic (Adding/Subtracting)](#date-arithmetic)
5. [Date Differences](#date-differences)
6. [Grouping by Time Periods (CRITICAL!)](#grouping-periods)
7. [Date Formatting & Conversion](#formatting)
8. [Comparing Dates](#comparing-dates)
9. [Window Functions with Dates](#window-dates)
10. [Common Interview Patterns](#interview-patterns)
11. [Quick Reference Table](#quick-reference)

---

## 🗄️ THE 3 SQL DIALECTS (IMPORTANT!) {#dialects}

**Different databases use different syntax!** I'll show all three:

- **PostgreSQL** (Most common in interviews, used by companies like Airbnb, Reddit)
- **MySQL** (Used by Facebook, Twitter, YouTube)
- **SQL Server** (Used by Microsoft, many enterprises)

**For interviews, focus on PostgreSQL first**, but know the differences.

---

## 📅 GETTING CURRENT DATE/TIME {#current-date}

### Get Today's Date (No Time)

**PostgreSQL:**
```sql
SELECT CURRENT_DATE;
-- Result: 2026-07-26
```

**MySQL:**
```sql
SELECT CURRENT_DATE();
-- OR
SELECT CURDATE();
-- Result: 2026-07-26
```

**SQL Server:**
```sql
SELECT CAST(GETDATE() AS DATE);
-- OR (SQL Server 2012+)
SELECT CONVERT(DATE, GETDATE());
-- Result: 2026-07-26
```

---

### Get Current Date and Time

**PostgreSQL:**
```sql
SELECT CURRENT_TIMESTAMP;
-- OR
SELECT NOW();
-- Result: 2026-07-26 14:30:25
```

**MySQL:**
```sql
SELECT CURRENT_TIMESTAMP();
-- OR
SELECT NOW();
-- Result: 2026-07-26 14:30:25
```

**SQL Server:**
```sql
SELECT GETDATE();
-- OR (SQL Server 2012+)
SELECT SYSDATETIME();
-- Result: 2026-07-26 14:30:25
```

---

### Interview Use Case: "Find users who logged in today"

```sql
-- PostgreSQL & MySQL
SELECT * FROM users
WHERE DATE(last_login) = CURRENT_DATE;

-- SQL Server
SELECT * FROM users
WHERE CAST(last_login AS DATE) = CAST(GETDATE() AS DATE);
```

---

## 🔍 EXTRACTING PARTS OF A DATE {#extract-parts}

### Get Year, Month, Day

**PostgreSQL:**
```sql
SELECT
    EXTRACT(YEAR FROM order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    EXTRACT(DAY FROM order_date) AS day
FROM orders;
```

**MySQL:**
```sql
SELECT
    YEAR(order_date) AS year,
    MONTH(order_date) AS month,
    DAY(order_date) AS day
FROM orders;
```

**SQL Server:**
```sql
SELECT
    YEAR(order_date) AS year,
    MONTH(order_date) AS month,
    DAY(order_date) AS day
FROM orders;
-- OR
SELECT
    DATEPART(YEAR, order_date) AS year,
    DATEPART(MONTH, order_date) AS month,
    DATEPART(DAY, order_date) AS day
FROM orders;
```

---

### Get Day of Week

**PostgreSQL:**
```sql
SELECT
    EXTRACT(DOW FROM order_date) AS day_of_week,  -- 0=Sunday, 6=Saturday
    EXTRACT(ISODOW FROM order_date) AS iso_dow    -- 1=Monday, 7=Sunday
FROM orders;
```

**MySQL:**
```sql
SELECT
    DAYOFWEEK(order_date) AS day_of_week,  -- 1=Sunday, 7=Saturday
    WEEKDAY(order_date) AS weekday          -- 0=Monday, 6=Sunday
FROM orders;
```

**SQL Server:**
```sql
SELECT
    DATEPART(WEEKDAY, order_date) AS day_of_week  -- 1=Sunday, 7=Saturday (depends on DATEFIRST setting)
FROM orders;
```

---

### Get Week Number, Quarter

**PostgreSQL:**
```sql
SELECT
    EXTRACT(WEEK FROM order_date) AS week_number,
    EXTRACT(QUARTER FROM order_date) AS quarter
FROM orders;
```

**MySQL:**
```sql
SELECT
    WEEK(order_date) AS week_number,
    QUARTER(order_date) AS quarter
FROM orders;
```

**SQL Server:**
```sql
SELECT
    DATEPART(WEEK, order_date) AS week_number,
    DATEPART(QUARTER, order_date) AS quarter
FROM orders;
```

---

### Interview Use Case: "Count orders per month"

```sql
-- PostgreSQL
SELECT
    EXTRACT(YEAR FROM order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    COUNT(*) AS order_count
FROM orders
GROUP BY EXTRACT(YEAR FROM order_date), EXTRACT(MONTH FROM order_date)
ORDER BY year, month;

-- MySQL
SELECT
    YEAR(order_date) AS year,
    MONTH(order_date) AS month,
    COUNT(*) AS order_count
FROM orders
GROUP BY YEAR(order_date), MONTH(order_date)
ORDER BY year, month;
```

**⚠️ PROBLEM**: This merges January 2024 and January 2025 into separate rows, but you can't easily compare them as dates.

**Better approach**: Use DATE_TRUNC (see next section!)

---

## 📆 GROUPING BY TIME PERIODS (CRITICAL!) {#grouping-periods}

### The Problem:
`EXTRACT(MONTH FROM date)` gives you just the month number (1-12), which loses the year!

### The Solution: DATE_TRUNC (Round Down to Period Start)

**PostgreSQL (BEST):**
```sql
-- Round down to start of month
SELECT
    DATE_TRUNC('month', order_date) AS month_start,
    COUNT(*) AS order_count
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month_start;

-- Result:
-- 2024-01-01, 150
-- 2024-02-01, 200
-- 2024-03-01, 180
```

**Other options for DATE_TRUNC:**
```sql
DATE_TRUNC('year', order_date)    -- Start of year: 2024-01-01
DATE_TRUNC('month', order_date)   -- Start of month: 2024-07-01
DATE_TRUNC('week', order_date)    -- Start of week: 2024-07-22 (Monday)
DATE_TRUNC('day', order_date)     -- Start of day: 2024-07-26 00:00:00
DATE_TRUNC('hour', order_date)    -- Start of hour: 2024-07-26 14:00:00
```

---

**MySQL (No DATE_TRUNC):**
```sql
-- Fake it with DATE_FORMAT
SELECT
    DATE_FORMAT(order_date, '%Y-%m-01') AS month_start,
    COUNT(*) AS order_count
FROM orders
GROUP BY DATE_FORMAT(order_date, '%Y-%m-01')
ORDER BY month_start;

-- OR use DATE_ADD to go to first day of month
SELECT
    DATE_ADD(
        DATE_ADD(order_date, INTERVAL -DAY(order_date)+1 DAY),
        INTERVAL 0 MONTH
    ) AS month_start,
    COUNT(*) AS order_count
FROM orders
GROUP BY month_start
ORDER BY month_start;
```

---

**SQL Server:**
```sql
-- SQL Server 2022+ has DATE_TRUNC
SELECT
    DATE_TRUNC(month, order_date) AS month_start,
    COUNT(*) AS order_count
FROM orders
GROUP BY DATE_TRUNC(month, order_date)
ORDER BY month_start;

-- Older SQL Server versions:
SELECT
    DATEADD(MONTH, DATEDIFF(MONTH, 0, order_date), 0) AS month_start,
    COUNT(*) AS order_count
FROM orders
GROUP BY DATEADD(MONTH, DATEDIFF(MONTH, 0, order_date), 0)
ORDER BY month_start;
```

---

### **THIS IS THE #1 INTERVIEW PATTERN!**

**Question**: "Show monthly revenue for 2024"

```sql
-- PostgreSQL
SELECT
    DATE_TRUNC('month', order_date) AS month,
    SUM(revenue) AS total_revenue
FROM orders
WHERE EXTRACT(YEAR FROM order_date) = 2024
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;
```

---

## ➕ DATE ARITHMETIC (Adding/Subtracting) {#date-arithmetic}

### Add/Subtract Days, Months, Years

**PostgreSQL:**
```sql
-- Add days
SELECT order_date + INTERVAL '7 days' AS one_week_later FROM orders;

-- Subtract days
SELECT order_date - INTERVAL '30 days' AS thirty_days_ago FROM orders;

-- Add months
SELECT order_date + INTERVAL '3 months' AS three_months_later FROM orders;

-- Add years
SELECT order_date + INTERVAL '1 year' AS next_year FROM orders;

-- Combine multiple intervals
SELECT order_date + INTERVAL '1 year 2 months 15 days' AS future_date FROM orders;
```

---

**MySQL:**
```sql
-- Add days
SELECT DATE_ADD(order_date, INTERVAL 7 DAY) AS one_week_later FROM orders;

-- Subtract days
SELECT DATE_SUB(order_date, INTERVAL 30 DAY) AS thirty_days_ago FROM orders;

-- Add months
SELECT DATE_ADD(order_date, INTERVAL 3 MONTH) AS three_months_later FROM orders;

-- Add years
SELECT DATE_ADD(order_date, INTERVAL 1 YEAR) AS next_year FROM orders;

-- Alternative syntax
SELECT order_date + INTERVAL 7 DAY AS one_week_later FROM orders;
```

---

**SQL Server:**
```sql
-- Add days
SELECT DATEADD(DAY, 7, order_date) AS one_week_later FROM orders;

-- Subtract days
SELECT DATEADD(DAY, -30, order_date) AS thirty_days_ago FROM orders;

-- Add months
SELECT DATEADD(MONTH, 3, order_date) AS three_months_later FROM orders;

-- Add years
SELECT DATEADD(YEAR, 1, order_date) AS next_year FROM orders;
```

---

### Interview Use Case: "Find users active in last 30 days"

```sql
-- PostgreSQL
SELECT * FROM users
WHERE last_active >= CURRENT_DATE - INTERVAL '30 days';

-- MySQL
SELECT * FROM users
WHERE last_active >= DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY);

-- SQL Server
SELECT * FROM users
WHERE last_active >= DATEADD(DAY, -30, CAST(GETDATE() AS DATE));
```

---

### Interview Use Case: "Find orders placed in Q1 2024"

```sql
-- PostgreSQL
SELECT * FROM orders
WHERE order_date >= '2024-01-01'
  AND order_date < '2024-04-01';

-- OR using BETWEEN
SELECT * FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-03-31';

-- OR using date arithmetic
SELECT * FROM orders
WHERE order_date >= DATE_TRUNC('year', '2024-01-01'::date)
  AND order_date < DATE_TRUNC('year', '2024-01-01'::date) + INTERVAL '3 months';
```

**⚠️ Important**: `BETWEEN` is inclusive on both ends!
- `BETWEEN '2024-01-01' AND '2024-03-31'` includes March 31st fully (up to 23:59:59)
- Using `< '2024-04-01'` is safer and more explicit

---

## 📏 DATE DIFFERENCES {#date-differences}

### Calculate Difference Between Two Dates

**PostgreSQL:**
```sql
-- Difference in days (simple subtraction)
SELECT order_date - created_date AS days_difference FROM orders;

-- Get detailed interval
SELECT AGE(order_date, created_date) AS age_interval FROM orders;
-- Result: "2 years 3 months 15 days"

-- Extract specific parts from AGE
SELECT
    EXTRACT(YEAR FROM AGE(order_date, created_date)) AS years,
    EXTRACT(MONTH FROM AGE(order_date, created_date)) AS months,
    EXTRACT(DAY FROM AGE(order_date, created_date)) AS days
FROM orders;
```

---

**MySQL:**
```sql
-- Difference in days
SELECT DATEDIFF(order_date, created_date) AS days_difference FROM orders;

-- Difference in months (approximate)
SELECT TIMESTAMPDIFF(MONTH, created_date, order_date) AS months_difference FROM orders;

-- Difference in years
SELECT TIMESTAMPDIFF(YEAR, created_date, order_date) AS years_difference FROM orders;

-- Difference in hours
SELECT TIMESTAMPDIFF(HOUR, created_date, order_date) AS hours_difference FROM orders;
```

---

**SQL Server:**
```sql
-- Difference in days
SELECT DATEDIFF(DAY, created_date, order_date) AS days_difference FROM orders;

-- Difference in months
SELECT DATEDIFF(MONTH, created_date, order_date) AS months_difference FROM orders;

-- Difference in years
SELECT DATEDIFF(YEAR, created_date, order_date) AS years_difference FROM orders;

-- Difference in hours
SELECT DATEDIFF(HOUR, created_date, order_date) AS hours_difference FROM orders;
```

---

### Interview Use Case: "Find orders delivered late (> 7 days)"

```sql
-- PostgreSQL
SELECT
    order_id,
    order_date,
    delivered_date,
    delivered_date - order_date AS days_to_deliver
FROM orders
WHERE delivered_date - order_date > 7;

-- MySQL
SELECT
    order_id,
    order_date,
    delivered_date,
    DATEDIFF(delivered_date, order_date) AS days_to_deliver
FROM orders
WHERE DATEDIFF(delivered_date, order_date) > 7;

-- SQL Server
SELECT
    order_id,
    order_date,
    delivered_date,
    DATEDIFF(DAY, order_date, delivered_date) AS days_to_deliver
FROM orders
WHERE DATEDIFF(DAY, order_date, delivered_date) > 7;
```

---

### Interview Use Case: "Calculate customer age from date of birth"

```sql
-- PostgreSQL
SELECT
    customer_id,
    date_of_birth,
    EXTRACT(YEAR FROM AGE(CURRENT_DATE, date_of_birth)) AS age
FROM customers;

-- MySQL
SELECT
    customer_id,
    date_of_birth,
    TIMESTAMPDIFF(YEAR, date_of_birth, CURRENT_DATE) AS age
FROM customers;

-- SQL Server
SELECT
    customer_id,
    date_of_birth,
    DATEDIFF(YEAR, date_of_birth, GETDATE()) AS age
FROM customers;
```

---

## 🎨 DATE FORMATTING & CONVERSION {#formatting}

### Convert Date to String (Custom Format)

**PostgreSQL:**
```sql
SELECT
    TO_CHAR(order_date, 'YYYY-MM-DD') AS iso_format,
    TO_CHAR(order_date, 'DD/MM/YYYY') AS european_format,
    TO_CHAR(order_date, 'Month DD, YYYY') AS readable_format,
    TO_CHAR(order_date, 'Day') AS day_name,
    TO_CHAR(order_date, 'Mon') AS month_abbrev
FROM orders;

-- Result:
-- '2024-07-26', '26/07/2024', 'July 26, 2024', 'Friday', 'Jul'
```

**Common PostgreSQL format codes:**
- `YYYY` = 4-digit year (2024)
- `YY` = 2-digit year (24)
- `MM` = 2-digit month (07)
- `Month` = Full month name (July)
- `Mon` = Abbreviated month (Jul)
- `DD` = 2-digit day (26)
- `Day` = Full day name (Friday)
- `Dy` = Abbreviated day (Fri)
- `HH24` = Hour 00-23
- `MI` = Minute
- `SS` = Second

---

**MySQL:**
```sql
SELECT
    DATE_FORMAT(order_date, '%Y-%m-%d') AS iso_format,
    DATE_FORMAT(order_date, '%d/%m/%Y') AS european_format,
    DATE_FORMAT(order_date, '%M %d, %Y') AS readable_format,
    DATE_FORMAT(order_date, '%W') AS day_name,
    DATE_FORMAT(order_date, '%b') AS month_abbrev
FROM orders;
```

**Common MySQL format codes:**
- `%Y` = 4-digit year (2024)
- `%y` = 2-digit year (24)
- `%m` = 2-digit month (07)
- `%M` = Full month name (July)
- `%b` = Abbreviated month (Jul)
- `%d` = 2-digit day (26)
- `%W` = Full day name (Friday)
- `%a` = Abbreviated day (Fri)
- `%H` = Hour 00-23
- `%i` = Minute
- `%s` = Second

---

**SQL Server:**
```sql
SELECT
    FORMAT(order_date, 'yyyy-MM-dd') AS iso_format,
    FORMAT(order_date, 'dd/MM/yyyy') AS european_format,
    FORMAT(order_date, 'MMMM dd, yyyy') AS readable_format,
    FORMAT(order_date, 'dddd') AS day_name,
    FORMAT(order_date, 'MMM') AS month_abbrev
FROM orders;

-- OR use CONVERT with style codes
SELECT
    CONVERT(VARCHAR, order_date, 23) AS iso_format,      -- YYYY-MM-DD
    CONVERT(VARCHAR, order_date, 103) AS european_format, -- DD/MM/YYYY
    CONVERT(VARCHAR, order_date, 107) AS us_format        -- Mon DD, YYYY
FROM orders;
```

---

### Convert String to Date

**PostgreSQL:**
```sql
-- Explicit conversion
SELECT TO_DATE('2024-07-26', 'YYYY-MM-DD') AS date_value;

-- Implicit conversion (if format is standard)
SELECT '2024-07-26'::DATE AS date_value;

-- CAST (portable)
SELECT CAST('2024-07-26' AS DATE) AS date_value;
```

---

**MySQL:**
```sql
-- Explicit conversion
SELECT STR_TO_DATE('26/07/2024', '%d/%m/%Y') AS date_value;

-- Implicit conversion (if format is YYYY-MM-DD)
SELECT CAST('2024-07-26' AS DATE) AS date_value;
```

---

**SQL Server:**
```sql
-- Explicit conversion
SELECT CONVERT(DATE, '2024-07-26', 23) AS date_value;

-- Implicit conversion
SELECT CAST('2024-07-26' AS DATE) AS date_value;
```

---

## 🔀 COMPARING DATES {#comparing-dates}

### Basic Comparisons

```sql
-- All SQL dialects
SELECT * FROM orders WHERE order_date = '2024-07-26';
SELECT * FROM orders WHERE order_date > '2024-01-01';
SELECT * FROM orders WHERE order_date < CURRENT_DATE;
SELECT * FROM orders WHERE order_date >= '2024-01-01' AND order_date < '2024-04-01';
```

---

### BETWEEN (Inclusive on Both Ends)

```sql
-- Includes both '2024-01-01' and '2024-12-31'
SELECT * FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';

-- ⚠️ CAREFUL: This includes timestamps up to 2024-12-31 23:59:59.999
-- If you want only dates in 2024:
SELECT * FROM orders
WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01';
```

---

### Check if Date is NULL

```sql
SELECT * FROM orders WHERE delivered_date IS NULL;
SELECT * FROM orders WHERE delivered_date IS NOT NULL;
```

---

### Date in List

```sql
SELECT * FROM orders
WHERE order_date IN ('2024-07-26', '2024-07-27', '2024-07-28');
```

---

### First Day / Last Day of Month

**PostgreSQL:**
```sql
-- First day of month
SELECT DATE_TRUNC('month', order_date) AS first_day;

-- Last day of month
SELECT (DATE_TRUNC('month', order_date) + INTERVAL '1 month - 1 day')::DATE AS last_day;
```

---

**MySQL:**
```sql
-- First day of month
SELECT DATE_FORMAT(order_date, '%Y-%m-01') AS first_day;

-- Last day of month
SELECT LAST_DAY(order_date) AS last_day;
```

---

**SQL Server:**
```sql
-- First day of month
SELECT DATEADD(MONTH, DATEDIFF(MONTH, 0, order_date), 0) AS first_day;

-- Last day of month
SELECT EOMONTH(order_date) AS last_day;
```

---

## 🪟 WINDOW FUNCTIONS WITH DATES {#window-dates}

### LAG / LEAD (Compare with Previous/Next Row)

**Interview Pattern**: "Calculate days since last order per customer"

```sql
-- PostgreSQL, MySQL, SQL Server (all same syntax)
SELECT
    customer_id,
    order_date,
    LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS previous_order_date,
    order_date - LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS days_since_last_order
FROM orders;
```

---

### Interview Use Case: "Find customers with consecutive daily logins"

```sql
-- PostgreSQL
SELECT
    user_id,
    login_date,
    login_date - LAG(login_date) OVER (PARTITION BY user_id ORDER BY login_date) AS gap_days
FROM user_logins
WHERE login_date - LAG(login_date) OVER (PARTITION BY user_id ORDER BY login_date) = 1;

-- This finds logins that are exactly 1 day apart
```

---

### Interview Use Case: "Month-over-month revenue growth"

```sql
-- PostgreSQL
WITH monthly_revenue AS (
    SELECT
        DATE_TRUNC('month', order_date) AS month,
        SUM(revenue) AS total_revenue
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date)
)
SELECT
    month,
    total_revenue,
    LAG(total_revenue) OVER (ORDER BY month) AS previous_month_revenue,
    total_revenue - LAG(total_revenue) OVER (ORDER BY month) AS revenue_growth,
    ROUND(
        100.0 * (total_revenue - LAG(total_revenue) OVER (ORDER BY month)) /
        NULLIF(LAG(total_revenue) OVER (ORDER BY month), 0),
        2
    ) AS growth_percentage
FROM monthly_revenue
ORDER BY month;
```

---

## 🔥 COMMON INTERVIEW PATTERNS {#interview-patterns}

### Pattern 1: "Active users per month"

**The Problem**: Count distinct users who had any activity in each month

```sql
-- PostgreSQL
SELECT
    DATE_TRUNC('month', activity_date) AS month,
    COUNT(DISTINCT user_id) AS active_users
FROM user_activity
GROUP BY DATE_TRUNC('month', activity_date)
ORDER BY month;
```

---

### Pattern 2: "Retention rate (users active this month AND last month)"

```sql
-- PostgreSQL
WITH monthly_users AS (
    SELECT DISTINCT
        DATE_TRUNC('month', activity_date) AS month,
        user_id
    FROM user_activity
)
SELECT
    curr.month AS current_month,
    COUNT(DISTINCT curr.user_id) AS active_this_month,
    COUNT(DISTINCT prev.user_id) AS active_last_month,
    COUNT(DISTINCT CASE WHEN prev.user_id IS NOT NULL THEN curr.user_id END) AS retained,
    ROUND(
        100.0 * COUNT(DISTINCT CASE WHEN prev.user_id IS NOT NULL THEN curr.user_id END) /
        NULLIF(COUNT(DISTINCT prev.user_id), 0),
        2
    ) AS retention_rate
FROM monthly_users curr
LEFT JOIN monthly_users prev
    ON curr.user_id = prev.user_id
    AND prev.month = curr.month - INTERVAL '1 month'
GROUP BY curr.month
ORDER BY curr.month;
```

---

### Pattern 3: "First and last occurrence per user"

```sql
-- All dialects
SELECT
    user_id,
    MIN(order_date) AS first_order_date,
    MAX(order_date) AS last_order_date,
    MAX(order_date) - MIN(order_date) AS customer_lifetime_days,
    COUNT(*) AS total_orders
FROM orders
GROUP BY user_id;
```

---

### Pattern 4: "Running total by date"

```sql
-- PostgreSQL, MySQL, SQL Server
SELECT
    order_date,
    revenue,
    SUM(revenue) OVER (ORDER BY order_date) AS running_total
FROM orders
ORDER BY order_date;
```

---

### Pattern 5: "Identify gaps in date sequences"

**Find missing dates in a sequence:**

```sql
-- PostgreSQL
WITH date_range AS (
    SELECT generate_series(
        (SELECT MIN(order_date) FROM orders),
        (SELECT MAX(order_date) FROM orders),
        INTERVAL '1 day'
    )::DATE AS date
)
SELECT dr.date AS missing_date
FROM date_range dr
LEFT JOIN orders o ON dr.date = o.order_date
WHERE o.order_date IS NULL;
```

---

### Pattern 6: "Session detection (gap > N minutes)"

**Find user sessions where a new session starts if gap > 30 minutes:**

```sql
-- PostgreSQL
WITH activity_with_gaps AS (
    SELECT
        user_id,
        activity_time,
        LAG(activity_time) OVER (PARTITION BY user_id ORDER BY activity_time) AS prev_activity,
        activity_time - LAG(activity_time) OVER (PARTITION BY user_id ORDER BY activity_time) AS gap
    FROM user_activity
),
session_starts AS (
    SELECT
        user_id,
        activity_time,
        CASE
            WHEN gap IS NULL OR gap > INTERVAL '30 minutes' THEN 1
            ELSE 0
        END AS is_new_session
    FROM activity_with_gaps
),
sessions AS (
    SELECT
        user_id,
        activity_time,
        SUM(is_new_session) OVER (PARTITION BY user_id ORDER BY activity_time) AS session_id
    FROM session_starts
)
SELECT
    user_id,
    session_id,
    MIN(activity_time) AS session_start,
    MAX(activity_time) AS session_end,
    MAX(activity_time) - MIN(activity_time) AS session_duration
FROM sessions
GROUP BY user_id, session_id
ORDER BY user_id, session_id;
```

---

### Pattern 7: "Year-over-year comparison"

```sql
-- PostgreSQL
SELECT
    EXTRACT(MONTH FROM order_date) AS month,
    EXTRACT(YEAR FROM order_date) AS year,
    SUM(revenue) AS total_revenue
FROM orders
WHERE EXTRACT(YEAR FROM order_date) IN (2023, 2024)
GROUP BY EXTRACT(YEAR FROM order_date), EXTRACT(MONTH FROM order_date)
ORDER BY month, year;

-- Better with DATE_TRUNC
SELECT
    DATE_TRUNC('month', order_date) AS month,
    SUM(revenue) AS total_revenue
FROM orders
WHERE order_date >= '2023-01-01' AND order_date < '2025-01-01'
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;
```

---

### Pattern 8: "Business days between two dates"

**Exclude weekends:**

```sql
-- PostgreSQL (approximate - doesn't handle holidays)
SELECT
    order_id,
    order_date,
    delivered_date,
    delivered_date - order_date AS total_days,
    (
        delivered_date - order_date -
        2 * ((delivered_date - order_date) / 7)
    ) AS business_days_approx
FROM orders;
```

For accurate business days, you need a calendar table with holidays.

---

### Pattern 9: "Time buckets (morning, afternoon, evening)"

```sql
-- All dialects
SELECT
    CASE
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 0 AND 11 THEN 'Morning'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END AS time_bucket,
    COUNT(*) AS order_count
FROM orders
GROUP BY time_bucket;
```

---

### Pattern 10: "Cohort analysis (users by signup month)"

```sql
-- PostgreSQL
WITH user_cohorts AS (
    SELECT
        user_id,
        DATE_TRUNC('month', signup_date) AS cohort_month
    FROM users
),
monthly_activity AS (
    SELECT
        user_id,
        DATE_TRUNC('month', activity_date) AS activity_month
    FROM user_activity
)
SELECT
    uc.cohort_month,
    ma.activity_month,
    EXTRACT(MONTH FROM AGE(ma.activity_month, uc.cohort_month)) AS months_since_signup,
    COUNT(DISTINCT ma.user_id) AS active_users
FROM user_cohorts uc
JOIN monthly_activity ma ON uc.user_id = ma.user_id
GROUP BY uc.cohort_month, ma.activity_month
ORDER BY uc.cohort_month, ma.activity_month;
```

---

## 📋 QUICK REFERENCE TABLE {#quick-reference}

| **What You Need** | **PostgreSQL** | **MySQL** | **SQL Server** |
|-------------------|----------------|-----------|----------------|
| **Current Date** | `CURRENT_DATE` | `CURDATE()` | `CAST(GETDATE() AS DATE)` |
| **Current Timestamp** | `NOW()` | `NOW()` | `GETDATE()` |
| **Extract Year** | `EXTRACT(YEAR FROM date)` | `YEAR(date)` | `YEAR(date)` |
| **Extract Month** | `EXTRACT(MONTH FROM date)` | `MONTH(date)` | `MONTH(date)` |
| **Round to Month Start** | `DATE_TRUNC('month', date)` | `DATE_FORMAT(date, '%Y-%m-01')` | `DATE_TRUNC(month, date)` (2022+) |
| **Add Days** | `date + INTERVAL '7 days'` | `DATE_ADD(date, INTERVAL 7 DAY)` | `DATEADD(DAY, 7, date)` |
| **Subtract Days** | `date - INTERVAL '7 days'` | `DATE_SUB(date, INTERVAL 7 DAY)` | `DATEADD(DAY, -7, date)` |
| **Date Difference (days)** | `date1 - date2` | `DATEDIFF(date1, date2)` | `DATEDIFF(DAY, date2, date1)` |
| **Format Date** | `TO_CHAR(date, 'YYYY-MM-DD')` | `DATE_FORMAT(date, '%Y-%m-%d')` | `FORMAT(date, 'yyyy-MM-dd')` |
| **Parse String to Date** | `TO_DATE('2024-07-26', 'YYYY-MM-DD')` | `STR_TO_DATE('2024-07-26', '%Y-%m-%d')` | `CAST('2024-07-26' AS DATE)` |
| **Last Day of Month** | `DATE_TRUNC('month', d) + INTERVAL '1 month - 1 day'` | `LAST_DAY(date)` | `EOMONTH(date)` |
| **Day of Week** | `EXTRACT(DOW FROM date)` | `DAYOFWEEK(date)` | `DATEPART(WEEKDAY, date)` |

---

## 🎯 INTERVIEW PREP CHECKLIST

### Master These 5:
1. ✅ **DATE_TRUNC / round to period** (grouping by month/year)
2. ✅ **Date arithmetic** (+ INTERVAL, DATE_ADD, DATEADD)
3. ✅ **DATEDIFF / date subtraction** (days between dates)
4. ✅ **LAG/LEAD** (compare with previous/next row)
5. ✅ **EXTRACT / date parts** (year, month, day)

### Common Interview Questions:
- [ ] Monthly active users
- [ ] Month-over-month growth
- [ ] Retention rate (M/M)
- [ ] First/last order per customer
- [ ] Days since last activity
- [ ] Year-over-year comparison
- [ ] Identify date gaps
- [ ] Session detection
- [ ] Running total by date
- [ ] Cohort analysis

---

## 💡 TIPS & TRICKS

### Tip 1: Always Use DATE_TRUNC for Period Grouping
❌ **Don't do this:**
```sql
GROUP BY EXTRACT(MONTH FROM date)  -- Loses year!
```

✅ **Do this:**
```sql
GROUP BY DATE_TRUNC('month', date)  -- Preserves year!
```

---

### Tip 2: Be Careful with BETWEEN
❌ **This includes timestamps:**
```sql
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31'
-- Includes 2024-12-31 23:59:59.999
```

✅ **This is clearer:**
```sql
WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01'
```

---

### Tip 3: Use INTERVAL Instead of Numbers
❌ **Confusing:**
```sql
WHERE date > CURRENT_DATE - 30  -- What unit?
```

✅ **Clear:**
```sql
WHERE date > CURRENT_DATE - INTERVAL '30 days'  -- Explicit!
```

---

### Tip 4: Handle NULLs in Date Differences
```sql
-- Use COALESCE to handle NULL dates
SELECT
    order_id,
    COALESCE(delivered_date, CURRENT_DATE) - order_date AS days_pending
FROM orders;
```

---

### Tip 5: Generate Date Series for Missing Dates
```sql
-- PostgreSQL: Generate all dates in range
SELECT generate_series(
    '2024-01-01'::DATE,
    '2024-12-31'::DATE,
    INTERVAL '1 day'
)::DATE AS date;
```

---

## 🚀 YOU'RE READY!

You now know:
- ✅ How to get current date/time
- ✅ How to extract parts of a date
- ✅ How to group by time periods (THE MOST IMPORTANT!)
- ✅ How to add/subtract dates
- ✅ How to calculate date differences
- ✅ How to format dates
- ✅ How to use window functions with dates
- ✅ All common interview patterns

**Remember**: Focus on PostgreSQL syntax first, but know MySQL/SQL Server differ!

**Most Common in Interviews**:
1. `DATE_TRUNC('month', date)` - Grouping by period
2. `date + INTERVAL '30 days'` - Date arithmetic
3. `LAG(date) OVER (...)` - Compare with previous row

**Now go crush those date questions!** 📅💪
