# Chapter 8: Practice Problems & Solutions

**Comprehensive problem set covering all data engineering topics**

---

## Table of Contents

1. [How to Use This Chapter](#how-to-use-this-chapter)
2. [SQL Fundamentals Problems](#sql-fundamentals-problems)
3. [SQL Window Functions Problems](#sql-window-functions-problems)
4. [Advanced SQL Problems](#advanced-sql-problems)
5. [Python Problems](#python-problems)
6. [Java Problems](#java-problems)
7. [Pipeline Architecture Problems](#pipeline-architecture-problems)
8. [Data Modeling Problems](#data-modeling-problems)
9. [Mixed Interview Problems](#mixed-interview-problems)
10. [Solutions Guide](#solutions-guide)

---

## How to Use This Chapter

### Study Approach

**1. Attempt Problems First**
- Try to solve each problem on your own
- Time yourself (aim for: Easy 15min, Medium 25min, Hard 40min)
- Write complete solutions, not just pseudocode

**2. Check Solutions After Attempting**
- Review the solution only after trying
- Understand the thought process, not just the code
- Learn from alternative approaches

**3. Practice Spaced Repetition**
- Review problems after 3 days
- Re-solve difficult problems after 1 week
- Track which patterns you struggle with

### Problem Organization

Each problem includes:
- **Difficulty:** Easy / Medium / Hard
- **Topic:** SQL Fundamentals, Window Functions, etc.
- **Time Estimate:** Target completion time
- **Hints:** Progressive hints if you're stuck
- **Solution:** Complete solution with explanation
- **Complexity:** Time/space complexity where applicable

### Difficulty Levels

**Easy:**
- Single concept
- Straightforward approach
- 15-20 minutes
- DataDriven.io equivalent: Easy level

**Medium:**
- Multiple concepts
- Requires planning
- 25-30 minutes
- DataDriven.io equivalent: Medium level

**Hard:**
- Complex logic
- Multiple approaches possible
- 35-45 minutes
- DataDriven.io equivalent: Medium-Hard level

---

## SQL Fundamentals Problems

### Problem 1: Active Users (Easy)

**Difficulty:** Easy | **Time:** 15 min | **Topic:** Basic Aggregation

**Problem:**
Given a `users` table and a `logins` table, find the number of active users (users who logged in at least once) in January 2024.

**Tables:**
```sql
users:
| user_id | name  | created_at |
|---------|-------|------------|
| 1       | Alice | 2023-01-15 |
| 2       | Bob   | 2023-06-20 |
| 3       | Carol | 2024-01-10 |

logins:
| login_id | user_id | login_date |
|----------|---------|------------|
| 1        | 1       | 2024-01-15 |
| 2        | 1       | 2024-01-16 |
| 3        | 2       | 2024-02-01 |
| 4        | 3       | 2024-01-11 |
```

**Expected Output:**
```
| active_users |
|--------------|
| 2            |
```

**Hints:**
<details>
<summary>Hint 1</summary>
Use COUNT DISTINCT to count unique users
</details>

<details>
<summary>Hint 2</summary>
Filter login_date for January 2024 using WHERE clause
</details>

**Solution:**
<details>
<summary>Click to View Solution</summary>

```sql
SELECT
    COUNT(DISTINCT user_id) AS active_users
FROM logins
WHERE login_date >= '2024-01-01'
  AND login_date < '2024-02-01';
```

**Explanation:**
- `COUNT(DISTINCT user_id)`: Counts unique users (handles multiple logins per user)
- `WHERE login_date >= '2024-01-01' AND login_date < '2024-02-01'`: Filters for January
- Alternative: `WHERE EXTRACT(YEAR FROM login_date) = 2024 AND EXTRACT(MONTH FROM login_date) = 1`

**Key Concepts:**
- COUNT DISTINCT for unique counts
- Date filtering
- Understanding that users can login multiple times

</details>

---

### Problem 2: Customer Lifetime Value (Medium)

**Difficulty:** Medium | **Time:** 25 min | **Topic:** JOINs & Aggregation

**Problem:**
Calculate the lifetime value (total spending) for each customer. Include customers with zero purchases.

**Tables:**
```sql
customers:
| customer_id | name  | city     |
|-------------|-------|----------|
| 1           | Alice | Seattle  |
| 2           | Bob   | Portland |
| 3           | Carol | Boston   |

orders:
| order_id | customer_id | order_amount | order_date |
|----------|-------------|--------------|------------|
| 1        | 1           | 100.00       | 2024-01-15 |
| 2        | 1           | 150.00       | 2024-01-20 |
| 3        | 2           | 75.00        | 2024-01-18 |
```

**Expected Output:**
```
| customer_id | name  | lifetime_value |
|-------------|-------|----------------|
| 1           | Alice | 250.00         |
| 2           | Bob   | 75.00          |
| 3           | Carol | 0.00           |
```

**Hints:**
<details>
<summary>Hint 1</summary>
Use LEFT JOIN to include customers without orders
</details>

<details>
<summary>Hint 2</summary>
Use COALESCE to handle NULL sums
</details>

**Solution:**
<details>
<summary>Click to View Solution</summary>

```sql
SELECT
    c.customer_id,
    c.name,
    COALESCE(SUM(o.order_amount), 0) AS lifetime_value
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name
ORDER BY lifetime_value DESC;
```

**Explanation:**
- `LEFT JOIN`: Includes all customers, even those without orders
- `COALESCE(SUM(o.order_amount), 0)`: Replaces NULL with 0 for customers without orders
- `GROUP BY c.customer_id, c.name`: Groups by customer to aggregate orders

**Common Mistakes:**
- Using INNER JOIN (misses customers without orders)
- Forgetting COALESCE (shows NULL instead of 0)
- Not including customer_id and name in GROUP BY

</details>

---

### Problem 3: Product Pairs (Hard)

**Difficulty:** Hard | **Time:** 35 min | **Topic:** Self JOINs

**Problem:**
Find all pairs of products that were purchased together in the same order. Show each pair once (avoid duplicates like (A,B) and (B,A)).

**Tables:**
```sql
order_items:
| order_id | product_id | product_name |
|----------|------------|--------------|
| 1        | 100        | Laptop       |
| 1        | 101        | Mouse        |
| 1        | 102        | Keyboard     |
| 2        | 100        | Laptop       |
| 2        | 101        | Mouse        |
| 3        | 103        | Monitor      |
```

**Expected Output:**
```
| product1  | product2  | times_purchased_together |
|-----------|-----------|--------------------------|
| Laptop    | Mouse     | 2                        |
| Laptop    | Keyboard  | 1                        |
| Mouse     | Keyboard  | 1                        |
```

**Solution:**
<details>
<summary>Click to View Solution</summary>

```sql
SELECT
    oi1.product_name AS product1,
    oi2.product_name AS product2,
    COUNT(DISTINCT oi1.order_id) AS times_purchased_together
FROM order_items oi1
JOIN order_items oi2
    ON oi1.order_id = oi2.order_id
    AND oi1.product_id < oi2.product_id  -- Avoid duplicates
GROUP BY oi1.product_name, oi2.product_name
ORDER BY times_purchased_together DESC, product1, product2;
```

**Explanation:**
- **Self JOIN**: `order_items` joined with itself to find products in same order
- **Key condition**: `oi1.product_id < oi2.product_id`
  - Ensures each pair appears once
  - Example: (100, 101) appears, but not (101, 100)
  - Also prevents product paired with itself (100, 100)
- `COUNT(DISTINCT oi1.order_id)`: Counts number of orders with this pair

**Alternative Approach (Using Window Functions):**
```sql
WITH product_pairs AS (
    SELECT
        order_id,
        product_id,
        product_name,
        LEAD(product_id) OVER (PARTITION BY order_id ORDER BY product_id) AS next_product_id,
        LEAD(product_name) OVER (PARTITION BY order_id ORDER BY product_id) AS next_product_name
    FROM order_items
)
SELECT
    product_name AS product1,
    next_product_name AS product2,
    COUNT(*) AS times_purchased_together
FROM product_pairs
WHERE next_product_id IS NOT NULL
GROUP BY product_name, next_product_name;
```

**Key Concepts:**
- Self JOINs for finding relationships within same table
- Inequality JOINs to avoid duplicates
- DISTINCT in aggregations

</details>

---

## SQL Window Functions Problems

### Problem 4: Top 3 Salespersons Per Region (Medium)

**Difficulty:** Medium | **Time:** 20 min | **Topic:** Window Functions (RANK)

**Problem:**
Find the top 3 salespersons by revenue in each region.

**Tables:**
```sql
sales:
| salesperson_id | name   | region | total_revenue |
|----------------|--------|--------|---------------|
| 1              | Alice  | West   | 500000        |
| 2              | Bob    | West   | 450000        |
| 3              | Carol  | West   | 400000        |
| 4              | David  | West   | 350000        |
| 5              | Eve    | East   | 600000        |
| 6              | Frank  | East   | 550000        |
```

**Expected Output:**
```
| salesperson_id | name  | region | total_revenue | rank |
|----------------|-------|--------|---------------|------|
| 1              | Alice | West   | 500000        | 1    |
| 2              | Bob   | West   | 450000        | 2    |
| 3              | Carol | West   | 400000        | 3    |
| 5              | Eve   | East   | 600000        | 1    |
| 6              | Frank | East   | 550000        | 2    |
```

**Solution:**
<details>
<summary>Click to View Solution</summary>

```sql
WITH ranked_sales AS (
    SELECT
        salesperson_id,
        name,
        region,
        total_revenue,
        RANK() OVER (PARTITION BY region ORDER BY total_revenue DESC) AS rank
    FROM sales
)
SELECT
    salesperson_id,
    name,
    region,
    total_revenue,
    rank
FROM ranked_sales
WHERE rank <= 3
ORDER BY region, rank;
```

**Explanation:**
- `PARTITION BY region`: Creates separate rankings for each region
- `ORDER BY total_revenue DESC`: Ranks by revenue (highest first)
- `RANK()`: Assigns rank (handles ties by giving same rank)
- `WHERE rank <= 3`: Filters to top 3

**RANK vs DENSE_RANK vs ROW_NUMBER:**
```sql
-- RANK: 1, 2, 2, 4 (skips after tie)
-- DENSE_RANK: 1, 2, 2, 3 (no skip)
-- ROW_NUMBER: 1, 2, 3, 4 (no ties)
```

</details>

---

### Problem 5: Moving 7-Day Average (Medium)

**Difficulty:** Medium | **Time:** 25 min | **Topic:** Window Functions (Frame Clause)

**Problem:**
Calculate a 7-day moving average of daily sales.

**Tables:**
```sql
daily_sales:
| sale_date  | revenue |
|------------|---------|
| 2024-01-01 | 1000    |
| 2024-01-02 | 1200    |
| 2024-01-03 | 900     |
| 2024-01-04 | 1100    |
| 2024-01-05 | 1300    |
| 2024-01-06 | 1000    |
| 2024-01-07 | 1400    |
| 2024-01-08 | 1200    |
```

**Expected Output:**
```
| sale_date  | revenue | moving_avg_7d |
|------------|---------|---------------|
| 2024-01-01 | 1000    | 1000.00       |
| 2024-01-02 | 1200    | 1100.00       |
| 2024-01-03 | 900     | 1033.33       |
| 2024-01-04 | 1100    | 1050.00       |
| 2024-01-05 | 1300    | 1100.00       |
| 2024-01-06 | 1000    | 1083.33       |
| 2024-01-07 | 1400    | 1128.57       |
| 2024-01-08 | 1200    | 1157.14       |
```

**Solution:**
<details>
<summary>Click to View Solution</summary>

```sql
SELECT
    sale_date,
    revenue,
    ROUND(AVG(revenue) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ), 2) AS moving_avg_7d
FROM daily_sales
ORDER BY sale_date;
```

**Explanation:**
- `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW`: Includes current row + previous 6 rows (7 total)
- `ORDER BY sale_date`: Ensures correct chronological order
- `AVG(revenue)`: Calculates average over the window
- For first few rows, averages over available rows (e.g., row 1 averages only 1 row)

**Frame Clause Options:**
```sql
-- Last 7 rows (including current)
ROWS BETWEEN 6 PRECEDING AND CURRENT ROW

-- Next 7 rows (including current)
ROWS BETWEEN CURRENT ROW AND 6 FOLLOWING

-- Centered 7-day window
ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING
```

</details>

---

### Problem 6: Session Detection (Hard)

**Difficulty:** Hard | **Time:** 40 min | **Topic:** Window Functions (LAG, Session Logic)

**Problem:**
Identify user sessions. A new session starts if there's a gap of more than 30 minutes between consecutive events.

**Tables:**
```sql
events:
| user_id | event_time          |
|---------|---------------------|
| 1       | 2024-01-15 10:00:00 |
| 1       | 2024-01-15 10:15:00 |
| 1       | 2024-01-15 10:50:00 |  -- New session (35 min gap)
| 1       | 2024-01-15 11:00:00 |
| 2       | 2024-01-15 09:00:00 |
| 2       | 2024-01-15 09:40:00 |  -- New session (40 min gap)
```

**Expected Output:**
```
| user_id | session_id | session_start       | session_end         | event_count |
|---------|------------|---------------------|---------------------|-------------|
| 1       | 1          | 2024-01-15 10:00:00 | 2024-01-15 10:15:00 | 2           |
| 1       | 2          | 2024-01-15 10:50:00 | 2024-01-15 11:00:00 | 2           |
| 2       | 1          | 2024-01-15 09:00:00 | 2024-01-15 09:00:00 | 1           |
| 2       | 2          | 2024-01-15 09:40:00 | 2024-01-15 09:40:00 | 1           |
```

**Solution:**
<details>
<summary>Click to View Solution</summary>

```sql
WITH time_diffs AS (
    SELECT
        user_id,
        event_time,
        LAG(event_time) OVER (PARTITION BY user_id ORDER BY event_time) AS prev_event_time
    FROM events
),
session_flags AS (
    SELECT
        user_id,
        event_time,
        prev_event_time,
        CASE
            WHEN prev_event_time IS NULL
                OR EXTRACT(EPOCH FROM (event_time - prev_event_time)) > 1800  -- 30 min = 1800 sec
            THEN 1
            ELSE 0
        END AS is_new_session
    FROM time_diffs
),
session_ids AS (
    SELECT
        user_id,
        event_time,
        SUM(is_new_session) OVER (PARTITION BY user_id ORDER BY event_time) AS session_id
    FROM session_flags
)
SELECT
    user_id,
    session_id,
    MIN(event_time) AS session_start,
    MAX(event_time) AS session_end,
    COUNT(*) AS event_count
FROM session_ids
GROUP BY user_id, session_id
ORDER BY user_id, session_id;
```

**Explanation:**

**Step 1 (time_diffs):**
- `LAG(event_time)`: Gets previous event time for each user

**Step 2 (session_flags):**
- `EXTRACT(EPOCH FROM (event_time - prev_event_time))`: Time difference in seconds
- `> 1800`: Marks new session if gap > 30 minutes
- First event per user always marked as new session (`prev_event_time IS NULL`)

**Step 3 (session_ids):**
- `SUM(is_new_session) OVER (...)`: Running sum creates unique session IDs
- Example: [1, 0, 1, 0, 0] → [1, 1, 2, 2, 2]

**Step 4 (final aggregation):**
- `MIN(event_time)`: Session start
- `MAX(event_time)`: Session end
- `COUNT(*)`: Events in session

**Key Concepts:**
- LAG for comparing consecutive rows
- Cumulative SUM for creating IDs
- Multi-step CTEs for complex logic

</details>

---

## Advanced SQL Problems

### Problem 7: Hierarchical Data (Medium)

**Difficulty:** Medium | **Time:** 30 min | **Topic:** Recursive CTEs

**Problem:**
Given an organization chart, find all employees reporting to a specific manager (directly or indirectly).

**Tables:**
```sql
employees:
| employee_id | name    | manager_id |
|-------------|---------|------------|
| 1           | Alice   | NULL       |  -- CEO
| 2           | Bob     | 1          |
| 3           | Carol   | 1          |
| 4           | David   | 2          |
| 5           | Eve     | 2          |
| 6           | Frank   | 3          |
```

**Expected Output (for manager_id = 2, Bob):**
```
| employee_id | name  | level |
|-------------|-------|-------|
| 4           | David | 1     |
| 5           | Eve   | 1     |
```

**Solution:**
<details>
<summary>Click to View Solution</summary>

```sql
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: Direct reports
    SELECT
        employee_id,
        name,
        manager_id,
        1 AS level
    FROM employees
    WHERE manager_id = 2  -- Bob's direct reports

    UNION ALL

    -- Recursive case: Reports of reports
    SELECT
        e.employee_id,
        e.name,
        e.manager_id,
        eh.level + 1
    FROM employees e
    INNER JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT
    employee_id,
    name,
    level
FROM employee_hierarchy
ORDER BY level, name;
```

**Explanation:**
- **Base case**: Selects direct reports (employees where manager_id = 2)
- **Recursive case**: Joins back to find reports of reports
- `level`: Tracks depth in hierarchy (1 = direct report, 2 = skip-level, etc.)
- Recursion stops when no more employees found

**Full Org Chart (starting from CEO):**
```sql
WITH RECURSIVE org_chart AS (
    SELECT
        employee_id,
        name,
        manager_id,
        name AS reporting_chain,
        0 AS level
    FROM employees
    WHERE manager_id IS NULL  -- CEO

    UNION ALL

    SELECT
        e.employee_id,
        e.name,
        e.manager_id,
        oc.reporting_chain || ' → ' || e.name,
        oc.level + 1
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.employee_id
)
SELECT * FROM org_chart ORDER BY level, name;
```

</details>

---

### Problem 8: Pivot Data (Medium)

**Difficulty:** Medium | **Time:** 25 min | **Topic:** CASE + Aggregation

**Problem:**
Pivot monthly sales data from rows to columns.

**Tables:**
```sql
monthly_sales:
| product  | month | revenue |
|----------|-------|---------|
| Laptop   | Jan   | 10000   |
| Laptop   | Feb   | 12000   |
| Laptop   | Mar   | 11000   |
| Mouse    | Jan   | 2000    |
| Mouse    | Feb   | 2500    |
| Mouse    | Mar   | 2200    |
```

**Expected Output:**
```
| product | Jan   | Feb   | Mar   |
|---------|-------|-------|-------|
| Laptop  | 10000 | 12000 | 11000 |
| Mouse   | 2000  | 2500  | 2200  |
```

**Solution:**
<details>
<summary>Click to View Solution</summary>

```sql
SELECT
    product,
    SUM(CASE WHEN month = 'Jan' THEN revenue ELSE 0 END) AS Jan,
    SUM(CASE WHEN month = 'Feb' THEN revenue ELSE 0 END) AS Feb,
    SUM(CASE WHEN month = 'Mar' THEN revenue ELSE 0 END) AS Mar
FROM monthly_sales
GROUP BY product
ORDER BY product;
```

**Explanation:**
- `CASE WHEN month = 'Jan' THEN revenue ELSE 0 END`: Returns revenue for Jan, 0 otherwise
- `SUM(...)`: Sums the values (only Jan values contribute to Jan column)
- `GROUP BY product`: Aggregates each product

**Alternative (using FILTER):**
```sql
SELECT
    product,
    SUM(revenue) FILTER (WHERE month = 'Jan') AS Jan,
    SUM(revenue) FILTER (WHERE month = 'Feb') AS Feb,
    SUM(revenue) FILTER (WHERE month = 'Mar') AS Mar
FROM monthly_sales
GROUP BY product;
```

**Dynamic Pivot (PostgreSQL):**
```sql
SELECT * FROM crosstab(
    'SELECT product, month, revenue FROM monthly_sales ORDER BY 1,2',
    'SELECT DISTINCT month FROM monthly_sales ORDER BY 1'
) AS ct(product TEXT, Jan INT, Feb INT, Mar INT);
```

</details>

---

## Python Problems

### Problem 9: Frequency Counter (Easy)

**Difficulty:** Easy | **Time:** 15 min | **Topic:** Collections.Counter

**Problem:**
Find the most common word in a text, ignoring case and punctuation.

**Input:**
```python
text = "Hello world! Hello Python. Python is great, Python rocks!"
```

**Expected Output:**
```
'python'
```

**Solution:**
<details>
<summary>Click to View Solution</summary>

```python
from collections import Counter
import re

def most_common_word(text):
    # Remove punctuation and convert to lowercase
    words = re.findall(r'\b\w+\b', text.lower())

    # Count word frequencies
    word_counts = Counter(words)

    # Return most common word
    return word_counts.most_common(1)[0][0]

# Test
text = "Hello world! Hello Python. Python is great, Python rocks!"
print(most_common_word(text))  # Output: 'python'
```

**Explanation:**
- `re.findall(r'\b\w+\b', text.lower())`: Extracts words (alphanumeric), removes punctuation
- `Counter(words)`: Counts frequency of each word
- `most_common(1)`: Returns list of top 1 most common: [('python', 3)]
- `[0][0]`: Extracts the word from tuple

**Alternative (without regex):**
```python
def most_common_word_simple(text):
    words = text.lower().replace('!', '').replace('.', '').replace(',', '').split()
    word_counts = Counter(words)
    return word_counts.most_common(1)[0][0]
```

**Time Complexity:** O(n) where n is number of characters
**Space Complexity:** O(w) where w is number of unique words

</details>

---

### Problem 10: Session Detection in Python (Medium)

**Difficulty:** Medium | **Time:** 30 min | **Topic:** Data Processing

**Problem:**
Given user events, group them into sessions. A new session starts after 30 minutes of inactivity.

**Input:**
```python
events = [
    {'user_id': 1, 'timestamp': '2024-01-15 10:00:00'},
    {'user_id': 1, 'timestamp': '2024-01-15 10:15:00'},
    {'user_id': 1, 'timestamp': '2024-01-15 10:50:00'},  # New session
    {'user_id': 2, 'timestamp': '2024-01-15 09:00:00'},
]
```

**Expected Output:**
```python
[
    {'user_id': 1, 'session_id': 1, 'start': '2024-01-15 10:00:00', 'end': '2024-01-15 10:15:00', 'event_count': 2},
    {'user_id': 1, 'session_id': 2, 'start': '2024-01-15 10:50:00', 'end': '2024-01-15 10:50:00', 'event_count': 1},
    {'user_id': 2, 'session_id': 1, 'start': '2024-01-15 09:00:00', 'end': '2024-01-15 09:00:00', 'event_count': 1},
]
```

**Solution:**
<details>
<summary>Click to View Solution</summary>

```python
from datetime import datetime
from collections import defaultdict

def detect_sessions(events):
    # Sort events by user and timestamp
    sorted_events = sorted(events, key=lambda x: (x['user_id'], x['timestamp']))

    # Group by user
    user_events = defaultdict(list)
    for event in sorted_events:
        user_events[event['user_id']].append(event['timestamp'])

    # Detect sessions
    sessions = []
    session_id = {}

    for user_id, timestamps in user_events.items():
        session_num = 1
        session_start = None
        session_end = None
        event_count = 0

        for i, ts_str in enumerate(timestamps):
            ts = datetime.strptime(ts_str, '%Y-%m-%d %H:%M:%S')

            if session_start is None:
                # Start first session
                session_start = ts_str
                session_end = ts_str
                event_count = 1
            else:
                # Check time gap
                prev_ts = datetime.strptime(timestamps[i-1], '%Y-%m-%d %H:%M:%S')
                gap_minutes = (ts - prev_ts).total_seconds() / 60

                if gap_minutes > 30:
                    # Save previous session
                    sessions.append({
                        'user_id': user_id,
                        'session_id': session_num,
                        'start': session_start,
                        'end': session_end,
                        'event_count': event_count
                    })

                    # Start new session
                    session_num += 1
                    session_start = ts_str
                    session_end = ts_str
                    event_count = 1
                else:
                    # Continue session
                    session_end = ts_str
                    event_count += 1

        # Save last session
        sessions.append({
            'user_id': user_id,
            'session_id': session_num,
            'start': session_start,
            'end': session_end,
            'event_count': event_count
        })

    return sorted(sessions, key=lambda x: (x['user_id'], x['session_id']))

# Test
events = [
    {'user_id': 1, 'timestamp': '2024-01-15 10:00:00'},
    {'user_id': 1, 'timestamp': '2024-01-15 10:15:00'},
    {'user_id': 1, 'timestamp': '2024-01-15 10:50:00'},
    {'user_id': 2, 'timestamp': '2024-01-15 09:00:00'},
]

result = detect_sessions(events)
for session in result:
    print(session)
```

**Time Complexity:** O(n log n) for sorting
**Space Complexity:** O(n) for storing sessions

</details>

---

## Java Problems

### Problem 11: Group Anagrams (Medium)

**Difficulty:** Medium | **Time:** 25 min | **Topic:** HashMap + String Processing

**Problem:**
Group anagrams together from a list of words.

**Input:**
```java
String[] words = {"eat", "tea", "tan", "ate", "nat", "bat"};
```

**Expected Output:**
```
[["eat", "tea", "ate"], ["tan", "nat"], ["bat"]]
```

**Solution:**
<details>
<summary>Click to View Solution</summary>

```java
import java.util.*;

public class GroupAnagrams {
    public static List<List<String>> groupAnagrams(String[] words) {
        Map<String, List<String>> anagramGroups = new HashMap<>();

        for (String word : words) {
            // Sort characters to create key
            char[] chars = word.toCharArray();
            Arrays.sort(chars);
            String sortedWord = new String(chars);

            // Add to group
            anagramGroups.computeIfAbsent(sortedWord, k -> new ArrayList<>()).add(word);
        }

        return new ArrayList<>(anagramGroups.values());
    }

    public static void main(String[] args) {
        String[] words = {"eat", "tea", "tan", "ate", "nat", "bat"};
        List<List<String>> result = groupAnagrams(words);

        for (List<String> group : result) {
            System.out.println(group);
        }
        // Output:
        // [eat, tea, ate]
        // [tan, nat]
        // [bat]
    }
}
```

**Explanation:**
- Sort each word's characters: "eat" → "aet", "tea" → "aet", "ate" → "aet"
- Use sorted string as HashMap key
- All anagrams map to same key
- `computeIfAbsent`: Creates new ArrayList if key doesn't exist

**Time Complexity:** O(n * k log k) where n = number of words, k = max word length
**Space Complexity:** O(n * k) for storing results

</details>

---

## Pipeline Architecture Problems

### Problem 12: Optimize Slow Spark Job (Medium)

**Difficulty:** Medium | **Time:** 20 min | **Topic:** Spark Optimization

**Problem:**
You have a Spark job that joins two DataFrames:
- `users`: 1 billion rows, 500GB
- `countries`: 200 rows, 1MB

The job takes 3 hours. How would you optimize it?

**Current Code:**
```python
users_df = spark.read.parquet("s3://data/users/")
countries_df = spark.read.parquet("s3://data/countries/")

result = users_df.join(countries_df, "country_code")
result.write.parquet("s3://output/")
```

**Solution:**
<details>
<summary>Click to View Solution</summary>

**Optimized Code:**
```python
from pyspark.sql.functions import broadcast

users_df = spark.read.parquet("s3://data/users/")
countries_df = spark.read.parquet("s3://data/countries/")

# Broadcast small table
result = users_df.join(broadcast(countries_df), "country_code")

# Partition output
result.coalesce(200).write.partitionBy("country_code").parquet("s3://output/")
```

**Optimizations Applied:**

1. **Broadcast Join**
   - Small table (1MB) broadcasted to all executors
   - Avoids shuffle of large table (500GB)
   - Expected speedup: 10-20x

2. **Coalesce Output**
   - Reduces number of output files
   - Combines small partitions
   - Better for downstream reads

3. **Partition by country_code**
   - Future queries on country_code read less data
   - Partition pruning enabled

**Additional Optimizations:**
```python
# 4. Cache if reused
users_df.cache()

# 5. Column pruning (read only needed columns)
users_df = spark.read.parquet("s3://data/users/").select("user_id", "country_code", "revenue")

# 6. Filter early
users_df = users_df.filter("country_code IS NOT NULL")

# 7. Configure Spark
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", "10MB")
```

**Expected Result:**
- Original: 3 hours
- After optimization: 10-15 minutes
- **12-18x speedup!**

</details>

---

## Data Modeling Problems

### Problem 13: Design Star Schema for Netflix (Hard)

**Difficulty:** Hard | **Time:** 40 min | **Topic:** Dimensional Modeling

**Problem:**
Design a star schema for Netflix to analyze:
- Viewing patterns (what users watch)
- Content performance (which shows are popular)
- User engagement (watch duration, completion rates)

**Requirements:**
- Track user viewing sessions
- Handle content metadata (genre, rating, release year)
- Support time-based analysis
- Handle multiple content types (movies, TV episodes)

**Solution:**
<details>
<summary>Click to View Solution</summary>

**Star Schema Design:**

**Fact Table: Fact_Viewing_Sessions**
```sql
CREATE TABLE fact_viewing_sessions (
    session_key BIGINT PRIMARY KEY,

    -- Dimension foreign keys
    date_key INT NOT NULL,
    time_key INT NOT NULL,  -- Time of day dimension
    user_key INT NOT NULL,
    content_key INT NOT NULL,
    device_key INT NOT NULL,

    -- Facts (measurements)
    watch_duration_seconds INT NOT NULL,  -- How long watched
    content_duration_seconds INT NOT NULL,  -- Total content length
    completion_percentage DECIMAL(5,2),  -- watch_duration / content_duration
    is_completed BOOLEAN,  -- Watched > 90%
    pause_count INT,
    rewind_count INT,
    video_quality VARCHAR(10),  -- HD, 4K, etc.

    -- Degenerate dimensions
    session_id VARCHAR(100),

    FOREIGN KEY (date_key) REFERENCES dim_date(date_key),
    FOREIGN KEY (user_key) REFERENCES dim_user(user_key),
    FOREIGN KEY (content_key) REFERENCES dim_content(content_key),
    FOREIGN KEY (device_key) REFERENCES dim_device(device_key)
);
```

**Grain:** One row per viewing session (user starts watching a piece of content)

**Dimension: Dim_Date**
```sql
CREATE TABLE dim_date (
    date_key INT PRIMARY KEY,  -- YYYYMMDD
    full_date DATE NOT NULL,
    day INT,
    month INT,
    year INT,
    quarter VARCHAR(2),
    day_of_week VARCHAR(10),
    is_weekend BOOLEAN,
    is_holiday BOOLEAN,
    week_of_year INT
);
```

**Dimension: Dim_Time (for time of day analysis)**
```sql
CREATE TABLE dim_time (
    time_key INT PRIMARY KEY,  -- HHMMSS
    hour INT,
    minute INT,
    time_period VARCHAR(20),  -- Morning, Afternoon, Evening, Night
    is_prime_time BOOLEAN  -- 8PM-11PM
);
```

**Dimension: Dim_User (SCD Type 2)**
```sql
CREATE TABLE dim_user (
    user_key INT PRIMARY KEY,  -- Surrogate key
    user_id VARCHAR(100) NOT NULL,  -- Natural key

    -- User attributes
    subscription_tier VARCHAR(20),  -- Basic, Standard, Premium
    country VARCHAR(50),
    age_group VARCHAR(20),  -- 18-24, 25-34, etc.
    account_created_date DATE,

    -- SCD Type 2 fields
    effective_date DATE NOT NULL,
    expiration_date DATE NOT NULL,
    is_current BOOLEAN NOT NULL
);
```

**Dimension: Dim_Content**
```sql
CREATE TABLE dim_content (
    content_key INT PRIMARY KEY,
    content_id VARCHAR(100) NOT NULL,

    -- Content attributes
    title VARCHAR(500),
    content_type VARCHAR(20),  -- Movie, TV Episode
    genre VARCHAR(100),
    subgenre VARCHAR(100),
    rating VARCHAR(10),  -- PG, PG-13, R, etc.
    release_year INT,
    language VARCHAR(50),
    country_of_origin VARCHAR(50),

    -- TV-specific (NULL for movies)
    series_name VARCHAR(500),
    season_number INT,
    episode_number INT,

    -- Content metrics (updated periodically)
    avg_rating DECIMAL(3,2),
    total_views BIGINT,
    content_duration_minutes INT
);
```

**Dimension: Dim_Device**
```sql
CREATE TABLE dim_device (
    device_key INT PRIMARY KEY,
    device_type VARCHAR(50),  -- TV, Mobile, Tablet, Desktop
    operating_system VARCHAR(50),  -- iOS, Android, Windows, etc.
    browser VARCHAR(50),
    app_version VARCHAR(20)
);
```

**Example Queries:**

**1. Most Watched Genres by Quarter:**
```sql
SELECT
    d.quarter,
    c.genre,
    COUNT(DISTINCT f.session_key) AS total_sessions,
    SUM(f.watch_duration_seconds) / 3600 AS total_hours_watched,
    AVG(f.completion_percentage) AS avg_completion_rate
FROM fact_viewing_sessions f
JOIN dim_date d ON f.date_key = d.date_key
JOIN dim_content c ON f.content_key = c.content_key
WHERE d.year = 2024
GROUP BY d.quarter, c.genre
ORDER BY d.quarter, total_hours_watched DESC;
```

**2. Prime Time Viewing Patterns:**
```sql
SELECT
    t.time_period,
    c.content_type,
    COUNT(DISTINCT f.user_key) AS unique_viewers,
    AVG(f.watch_duration_seconds) / 60 AS avg_watch_minutes
FROM fact_viewing_sessions f
JOIN dim_time t ON f.time_key = t.time_key
JOIN dim_content c ON f.content_key = c.content_key
WHERE f.date_key BETWEEN 20240101 AND 20240131
GROUP BY t.time_period, c.content_type
ORDER BY unique_viewers DESC;
```

**3. User Churn Risk Analysis:**
```sql
WITH user_engagement AS (
    SELECT
        u.user_key,
        u.subscription_tier,
        COUNT(f.session_key) AS sessions_last_30_days,
        AVG(f.completion_percentage) AS avg_completion
    FROM dim_user u
    LEFT JOIN fact_viewing_sessions f ON u.user_key = f.user_key
        AND f.date_key >= 20240101
    WHERE u.is_current = TRUE
    GROUP BY u.user_key, u.subscription_tier
)
SELECT
    subscription_tier,
    COUNT(*) AS total_users,
    COUNT(CASE WHEN sessions_last_30_days < 5 THEN 1 END) AS at_risk_users,
    AVG(avg_completion) AS avg_completion_rate
FROM user_engagement
GROUP BY subscription_tier;
```

**Design Decisions:**

1. **Separate Time Dimension**
   - Allows time-of-day analysis independent of date
   - Identifies prime-time viewing patterns

2. **Content Duration in Fact**
   - Denormalized for easy completion percentage calculation
   - Trade-off: Redundancy vs query simplicity

3. **SCD Type 2 for Users**
   - Track subscription tier changes over time
   - Analyze impact of upgrades/downgrades

4. **Semi-Additive Fact: completion_percentage**
   - Can't sum completion percentages
   - Use AVG for aggregations

5. **Degenerate Dimension: session_id**
   - No separate dimension needed
   - Stored in fact table for traceability

**Schema Diagram:**
```
        Dim_Date          Dim_Time
             \              /
              \            /
               \          /
                \        /
Dim_User -- Fact_Viewing_Sessions -- Dim_Content
                 /          \
                /            \
               /              \
        Dim_Device        (session_id)
```

</details>

---

## Mixed Interview Problems

### Problem 14: Data Quality Check (Medium)

**Difficulty:** Medium | **Time:** 25 min | **Topic:** SQL + Data Validation

**Problem:**
Write SQL queries to validate data quality for a sales table. Check for:
1. NULL values in critical columns
2. Negative revenues
3. Future order dates
4. Duplicate order IDs

**Tables:**
```sql
sales:
| order_id | customer_id | order_date | revenue |
|----------|-------------|------------|---------|
| 1        | 100         | 2024-01-15 | 150.00  |
| 2        | NULL        | 2024-01-16 | 75.00   |  -- Missing customer
| 3        | 101         | 2024-01-17 | -50.00  |  -- Negative revenue
| 4        | 102         | 2025-01-01 | 100.00  |  -- Future date
| 5        | 103         | 2024-01-18 | 200.00  |
| 5        | 104         | 2024-01-18 | 250.00  |  -- Duplicate order_id
```

**Solution:**
<details>
<summary>Click to View Solution</summary>

```sql
-- 1. NULL values in critical columns
SELECT
    'NULL customer_id' AS issue_type,
    COUNT(*) AS issue_count
FROM sales
WHERE customer_id IS NULL

UNION ALL

SELECT
    'NULL revenue' AS issue_type,
    COUNT(*) AS issue_count
FROM sales
WHERE revenue IS NULL

UNION ALL

-- 2. Negative revenues
SELECT
    'Negative revenue' AS issue_type,
    COUNT(*) AS issue_count
FROM sales
WHERE revenue < 0

UNION ALL

-- 3. Future order dates
SELECT
    'Future order date' AS issue_type,
    COUNT(*) AS issue_count
FROM sales
WHERE order_date > CURRENT_DATE

UNION ALL

-- 4. Duplicate order IDs
SELECT
    'Duplicate order_id' AS issue_type,
    COUNT(*) AS issue_count
FROM (
    SELECT order_id
    FROM sales
    GROUP BY order_id
    HAVING COUNT(*) > 1
) duplicates;

-- Output:
-- | issue_type           | issue_count |
-- |----------------------|-------------|
-- | NULL customer_id     | 1           |
-- | NULL revenue         | 0           |
-- | Negative revenue     | 1           |
-- | Future order date    | 1           |
-- | Duplicate order_id   | 1           |
```

**Detailed Issue Report:**
```sql
-- Get specific rows with issues
SELECT
    order_id,
    customer_id,
    order_date,
    revenue,
    CASE
        WHEN customer_id IS NULL THEN 'Missing customer'
        WHEN revenue < 0 THEN 'Negative revenue'
        WHEN order_date > CURRENT_DATE THEN 'Future date'
        ELSE 'OK'
    END AS issue
FROM sales
WHERE customer_id IS NULL
   OR revenue < 0
   OR order_date > CURRENT_DATE;

-- Check duplicates with details
SELECT
    order_id,
    COUNT(*) AS occurrence_count,
    STRING_AGG(customer_id::TEXT, ', ') AS customer_ids
FROM sales
GROUP BY order_id
HAVING COUNT(*) > 1;
```

**Production Data Quality Framework:**
```sql
CREATE TABLE data_quality_checks (
    check_id SERIAL PRIMARY KEY,
    table_name VARCHAR(100),
    check_type VARCHAR(100),
    issue_count INT,
    check_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert check results
INSERT INTO data_quality_checks (table_name, check_type, issue_count)
SELECT
    'sales' AS table_name,
    'NULL customer_id' AS check_type,
    COUNT(*) AS issue_count
FROM sales
WHERE customer_id IS NULL;

-- Alert if issues found
SELECT *
FROM data_quality_checks
WHERE issue_count > 0
  AND check_timestamp >= CURRENT_DATE;
```

</details>

---

### Problem 15: A/B Test Analysis (Hard)

**Difficulty:** Hard | **Time:** 45 min | **Topic:** SQL + Statistical Analysis

**Problem:**
Analyze an A/B test to determine if a new feature increased conversion rate.

**Tables:**
```sql
users:
| user_id | test_group |  -- A = control, B = treatment
|---------|------------|
| 1       | A          |
| 2       | A          |
| 3       | B          |
| 4       | B          |
| 5       | A          |

conversions:
| user_id | converted | conversion_date |
|---------|-----------|-----------------|
| 1       | TRUE      | 2024-01-15      |
| 2       | FALSE     | 2024-01-15      |
| 3       | TRUE      | 2024-01-15      |
| 4       | TRUE      | 2024-01-16      |
| 5       | FALSE     | 2024-01-16      |
```

**Expected Analysis:**
- Conversion rate by group
- Statistical significance
- Sample size per group
- Lift (improvement)

**Solution:**
<details>
<summary>Click to View Solution</summary>

```sql
WITH group_stats AS (
    SELECT
        u.test_group,
        COUNT(DISTINCT u.user_id) AS total_users,
        COUNT(DISTINCT CASE WHEN c.converted = TRUE THEN u.user_id END) AS converted_users,
        ROUND(
            COUNT(DISTINCT CASE WHEN c.converted = TRUE THEN u.user_id END)::NUMERIC /
            COUNT(DISTINCT u.user_id)::NUMERIC * 100,
            2
        ) AS conversion_rate_pct
    FROM users u
    LEFT JOIN conversions c ON u.user_id = c.user_id
    GROUP BY u.test_group
),
comparison AS (
    SELECT
        MAX(CASE WHEN test_group = 'A' THEN conversion_rate_pct END) AS control_rate,
        MAX(CASE WHEN test_group = 'B' THEN conversion_rate_pct END) AS treatment_rate,
        MAX(CASE WHEN test_group = 'A' THEN total_users END) AS control_size,
        MAX(CASE WHEN test_group = 'B' THEN total_users END) AS treatment_size
    FROM group_stats
)
SELECT
    control_rate,
    treatment_rate,
    treatment_rate - control_rate AS absolute_lift,
    ROUND((treatment_rate - control_rate) / control_rate * 100, 2) AS relative_lift_pct,
    control_size,
    treatment_size,
    CASE
        WHEN ABS(treatment_rate - control_rate) > 5 THEN 'Significant'
        ELSE 'Not Significant'
    END AS significance  -- Simplified (real: use chi-square test)
FROM comparison;

-- Output:
-- | control_rate | treatment_rate | absolute_lift | relative_lift_pct | control_size | treatment_size | significance |
-- |--------------|----------------|---------------|-------------------|--------------|----------------|--------------|
-- | 33.33        | 100.00         | 66.67         | 200.00            | 3            | 2              | Significant  |
```

**Detailed Breakdown:**
```sql
SELECT
    u.test_group,
    COUNT(DISTINCT u.user_id) AS total_users,
    COUNT(DISTINCT CASE WHEN c.converted = TRUE THEN u.user_id END) AS conversions,
    COUNT(DISTINCT CASE WHEN c.converted = FALSE OR c.converted IS NULL THEN u.user_id END) AS non_conversions,
    ROUND(
        COUNT(DISTINCT CASE WHEN c.converted = TRUE THEN u.user_id END)::NUMERIC /
        COUNT(DISTINCT u.user_id)::NUMERIC * 100,
        2
    ) AS conversion_rate_pct
FROM users u
LEFT JOIN conversions c ON u.user_id = c.user_id
GROUP BY u.test_group;
```

**Statistical Significance (Chi-Square Test Approximation):**
```sql
-- For proper statistical test, use Python/R
-- This is a simplified SQL approximation

WITH contingency_table AS (
    SELECT
        test_group,
        SUM(CASE WHEN converted = TRUE THEN 1 ELSE 0 END) AS conversions,
        SUM(CASE WHEN converted = FALSE OR converted IS NULL THEN 1 ELSE 0 END) AS non_conversions
    FROM users u
    LEFT JOIN conversions c ON u.user_id = c.user_id
    GROUP BY test_group
)
SELECT
    *,
    -- Expected values and chi-square calculation would go here
    -- For production, export to Python for proper statistical analysis
FROM contingency_table;
```

**Python Statistical Analysis:**
```python
from scipy import stats

# Observed data
control_conversions = 1
control_non_conversions = 2
treatment_conversions = 2
treatment_non_conversions = 0

# Chi-square test
contingency_table = [
    [control_conversions, control_non_conversions],
    [treatment_conversions, treatment_non_conversions]
]

chi2, p_value, dof, expected = stats.chi2_contingency(contingency_table)

print(f"Chi-square statistic: {chi2:.4f}")
print(f"P-value: {p_value:.4f}")
print(f"Significant at 0.05 level: {p_value < 0.05}")
```

</details>

---

## Solutions Guide

### How to Review Solutions

**1. First Attempt**
- Solve problem completely on your own
- Time yourself
- Write full code/SQL, not pseudocode

**2. Compare with Solution**
- Did you get the correct answer?
- Is your approach similar or different?
- Which approach is more efficient?

**3. Learn from Differences**
- If solution is different, understand why
- Learn new patterns (e.g., using LAG for previous row)
- Note time/space complexity trade-offs

**4. Practice Variations**
- Modify the problem slightly
- Change requirements (e.g., "top 3" → "top 5")
- Add constraints (e.g., "only last 7 days")

### Common Patterns Summary

**SQL Patterns:**
1. **TOP N per Group**: `ROW_NUMBER() OVER (PARTITION BY ... ORDER BY ...)`
2. **Running Totals**: `SUM(...) OVER (ORDER BY ... ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)`
3. **Previous Value**: `LAG(...) OVER (PARTITION BY ... ORDER BY ...)`
4. **Session Detection**: `LAG + CASE + Cumulative SUM`
5. **Hierarchy**: `WITH RECURSIVE`
6. **Pivot**: `CASE WHEN ... GROUP BY`
7. **Avoid Double-Count**: `COUNT DISTINCT`, aggregate before JOIN

**Python Patterns:**
1. **Frequency Count**: `Counter`
2. **Grouping**: `defaultdict(list)`
3. **Date Math**: `datetime`, `timedelta`
4. **String Processing**: `re.findall`, `str.lower()`

**Java Patterns:**
1. **Frequency Count**: `HashMap` with `merge()` or `compute()`
2. **Grouping**: `Collectors.groupingBy()`
3. **Sorting**: `Arrays.sort()`, `Collections.sort()`
4. **Stream Processing**: `.stream().filter().map().collect()`

### Next Steps

**Beginner Track (Weeks 1-4):**
- SQL Fundamentals: Problems 1-3
- Window Functions: Problems 4-5
- Python/Java: Problems 9, 11
- Practice 10-15 easy problems on DataDriven.io

**Intermediate Track (Weeks 5-7):**
- Advanced SQL: Problems 7-8
- Window Functions: Problem 6
- Python/Java: Problem 10
- Pipeline: Problem 12
- Practice 20-30 medium problems on DataDriven.io

**Advanced Track (Week 8+):**
- Data Modeling: Problem 13
- Mixed: Problems 14-15
- All hard problems
- Practice 30+ medium-hard problems on DataDriven.io

### DataDriven.io Problem Mapping

**SQL Fundamentals:**
- 30-Day Page View Counts
- New vs Returning Users
- Customer Retention Rate

**Window Functions:**
- 7-Check Rolling Average
- 10 Lowest Uptime Services
- Between the Clicks

**Advanced SQL:**
- Recursive CTEs for hierarchies
- Complex date calculations
- JSON processing

**Python/Java:**
- Activity Time Ledger
- Caesar Shift Check
- Letters in the Noise

**Pipeline Architecture:**
- Spark optimization questions
- ETL design scenarios

**Data Modeling:**
- Star schema design
- SCD implementation

---

## Congratulations!

You've completed the practice problems chapter. You now have:
- ✅ 15 solved problems across all topics
- ✅ Multiple difficulty levels
- ✅ Complete solutions with explanations
- ✅ Patterns and best practices

**Keep practicing on DataDriven.io to build fluency!**

---

**Chapter 8 Complete** | [Back to Main Guide](../README.md)
