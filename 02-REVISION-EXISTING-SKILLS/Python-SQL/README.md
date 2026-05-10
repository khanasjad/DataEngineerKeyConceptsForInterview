# Python & SQL - Core Data Engineering Skills

**Master programming fundamentals and database querying for data engineering interviews**

---

## 📚 What's in This Folder

### **1. 100-QUESTIONS.md**
Comprehensive Q&A covering:

**Python (Q1-Q50):**
- Data structures (lists, dicts, sets, tuples)
- Functions and decorators
- File I/O and data processing
- Error handling
- OOP concepts
- List comprehensions and generators
- Lambda functions
- Common libraries (pandas, requests)

**SQL (Q51-Q100):**
- SELECT, JOINs, GROUP BY
- Window functions
- CTEs and subqueries
- Aggregations
- Date/time functions
- Query optimization
- Indexes and performance
- Advanced analytics queries

### **2. CHEATSHEET.md**
Quick reference with:
- Python syntax essentials
- Common data structure operations
- SQL query patterns
- Window function templates
- Performance optimization tips
- Interview talking points

---

## 🎯 What You'll Learn

### **Python Essentials**
- **Data Structures:** When to use list vs set vs dict vs tuple
- **Comprehensions:** List/dict/set comprehensions for clean code
- **Generators:** Memory-efficient iteration for large datasets
- **Decorators:** Function wrappers for logging, timing, caching
- **Context Managers:** with statements for resource management
- **Error Handling:** try/except/finally patterns
- **File Processing:** Reading CSV, JSON, parquet files
- **Pandas:** DataFrame operations for data manipulation

### **SQL Mastery**
- **JOINs:** INNER, LEFT, RIGHT, FULL, CROSS, self-joins
- **Window Functions:** ROW_NUMBER, RANK, DENSE_RANK, LAG, LEAD
- **Aggregations:** GROUP BY, HAVING, COUNT, SUM, AVG
- **CTEs:** WITH clauses for readable queries
- **Subqueries:** Correlated vs non-correlated
- **Date Functions:** DATEDIFF, DATE_TRUNC, EXTRACT
- **Query Optimization:** EXPLAIN plans, indexes, query rewriting
- **Advanced Analytics:** Cohort analysis, retention, funnel analysis

---

## 💼 Why Python & SQL Matter

### **Industry Standard**
- **Python:** #1 language for data engineering
- **SQL:** Universal language for databases
- Combined: Required for 95%+ of data engineering roles

### **Daily Use Cases**

**Python:**
- ETL script development
- Data validation and cleaning
- API integration
- Automation tasks
- Spark/PySpark jobs
- Airflow DAG creation

**SQL:**
- Data exploration and analysis
- Writing transformations (dbt models)
- Query optimization
- Data quality checks
- Reporting and dashboards
- Ad-hoc analysis

---

## 🚀 Quick Start Guide

### **1. Python Review (2-3 hours)**
Focus on these topics from 100-QUESTIONS.md:
- **Q1-Q15:** Data structures and operations
- **Q16-Q25:** Functions, lambdas, comprehensions
- **Q26-Q35:** File I/O and pandas
- **Q36-Q50:** OOP, decorators, error handling

### **2. SQL Mastery (3-4 hours)**
Study these sections:
- **Q51-Q60:** JOINs and basic queries
- **Q61-Q75:** Window functions
- **Q76-Q85:** Aggregations and CTEs
- **Q86-Q100:** Advanced analytics and optimization

### **3. Practice Platforms**
- **Python:** LeetCode, HackerRank, CodeSignal
- **SQL:** LeetCode SQL, HackerRank SQL, Mode Analytics

---

## 📖 Python Topic Coverage

### **Data Structures (Q1-Q10)**
```python
# Lists - ordered, mutable
nums = [1, 2, 3, 4, 5]

# Dictionaries - key-value pairs
user = {'name': 'Alice', 'age': 30}

# Sets - unique elements, fast lookup
unique_ids = {1, 2, 3, 4, 5}

# Tuples - immutable
coordinates = (10, 20)
```

### **Common Patterns (Q11-Q25)**
```python
# List comprehension
squares = [x**2 for x in range(10)]

# Dictionary comprehension
word_lengths = {word: len(word) for word in ['a', 'bb', 'ccc']}

# Lambda functions
sorted_items = sorted(items, key=lambda x: x['price'])

# Generators (memory efficient)
def read_large_file(file_path):
    with open(file_path) as f:
        for line in f:
            yield line.strip()
```

### **Pandas Basics (Q26-Q35)**
```python
import pandas as pd

# Read CSV
df = pd.read_csv('data.csv')

# Filter rows
high_value = df[df['amount'] > 1000]

# Group by and aggregate
summary = df.groupby('category').agg({
    'amount': ['sum', 'mean', 'count']
})

# Merge dataframes
result = pd.merge(df1, df2, on='id', how='left')
```

### **Error Handling (Q36-Q50)**
```python
# Try-except pattern
try:
    result = process_data(file_path)
except FileNotFoundError:
    logging.error(f"File not found: {file_path}")
    raise
except Exception as e:
    logging.error(f"Unexpected error: {e}")
    # Handle or re-raise
finally:
    cleanup_resources()
```

---

## 📖 SQL Topic Coverage

### **JOINs (Q51-Q60)**
```sql
-- INNER JOIN (only matching)
SELECT c.name, o.amount
FROM customers c
INNER JOIN orders o ON c.id = o.customer_id;

-- LEFT JOIN (all from left + matches)
SELECT c.name, COUNT(o.id) as order_count
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.name;

-- Self-JOIN
SELECT e1.name, e2.name AS manager
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.id;
```

### **Window Functions (Q61-Q75)**
```sql
-- Ranking
SELECT
    name,
    dept,
    salary,
    DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC) as rank
FROM employees;

-- Running total
SELECT
    date,
    revenue,
    SUM(revenue) OVER (ORDER BY date) as running_total
FROM daily_sales;

-- Moving average
SELECT
    date,
    value,
    AVG(value) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg_7day
FROM metrics;

-- LAG/LEAD (previous/next row)
SELECT
    date,
    revenue,
    LAG(revenue) OVER (ORDER BY date) as prev_day,
    revenue - LAG(revenue) OVER (ORDER BY date) as change
FROM daily_revenue;
```

### **CTEs (Q76-Q85)**
```sql
-- Simple CTE
WITH high_value_customers AS (
    SELECT customer_id, SUM(amount) as total
    FROM orders
    GROUP BY customer_id
    HAVING SUM(amount) > 10000
)
SELECT c.name, hvc.total
FROM customers c
JOIN high_value_customers hvc ON c.id = hvc.customer_id;

-- Multiple CTEs
WITH
    customer_totals AS (
        SELECT customer_id, SUM(amount) as total
        FROM orders
        GROUP BY customer_id
    ),
    customer_segments AS (
        SELECT
            customer_id,
            total,
            CASE
                WHEN total > 10000 THEN 'VIP'
                WHEN total > 1000 THEN 'High'
                ELSE 'Regular'
            END as segment
        FROM customer_totals
    )
SELECT * FROM customer_segments;
```

### **Advanced Analytics (Q86-Q100)**
```sql
-- Cohort analysis
WITH cohorts AS (
    SELECT
        user_id,
        DATE_TRUNC('month', signup_date) as cohort_month
    FROM users
),
user_activity AS (
    SELECT
        c.cohort_month,
        DATE_TRUNC('month', a.activity_date) as activity_month,
        COUNT(DISTINCT a.user_id) as active_users
    FROM activity a
    JOIN cohorts c ON a.user_id = c.user_id
    GROUP BY c.cohort_month, activity_month
)
SELECT
    cohort_month,
    activity_month,
    active_users,
    active_users * 100.0 / FIRST_VALUE(active_users) OVER (
        PARTITION BY cohort_month
        ORDER BY activity_month
    ) as retention_rate
FROM user_activity;
```

---

## 🎓 Interview Preparation Strategy

### **Python Interview Prep**

**Week 1: Fundamentals**
- Data structures (list, dict, set, tuple)
- Time/space complexity basics
- String manipulation
- Common algorithms (sorting, searching)

**Week 2: Advanced**
- Comprehensions and generators
- Decorators and context managers
- Pandas operations
- Error handling patterns

**Common Python Questions:**
1. Reverse a string
2. Find duplicates in a list
3. Flatten nested lists
4. Parse JSON and extract data
5. Read large CSV file efficiently
6. Merge two dictionaries
7. Find most frequent element
8. Implement decorator for timing

### **SQL Interview Prep**

**Week 1: Basics**
- JOINs (all types)
- GROUP BY and HAVING
- Basic window functions
- Subqueries

**Week 2: Advanced**
- Complex window functions
- CTEs and recursive CTEs
- Query optimization
- Advanced analytics

**Common SQL Questions:**
1. Second highest salary
2. Employees earning more than managers
3. Customers who never ordered
4. Find duplicates
5. Top N per group
6. Running total
7. Month-over-month growth
8. Cohort retention analysis

---

## 💡 Interview Tips

### **Python Coding Interview**

**Before Writing Code:**
1. Clarify requirements
2. Ask about input constraints
3. Discuss approach
4. Consider edge cases

**While Coding:**
1. Write clean, readable code
2. Use meaningful variable names
3. Add comments for complex logic
4. Test with sample inputs

**Example:**
```python
def find_duplicates(nums):
    """
    Find duplicate numbers in a list.

    Args:
        nums: List of integers

    Returns:
        Set of duplicate numbers

    Time: O(n), Space: O(n)
    """
    seen = set()
    duplicates = set()

    for num in nums:
        if num in seen:
            duplicates.add(num)
        else:
            seen.add(num)

    return duplicates

# Test
assert find_duplicates([1, 2, 3, 2, 4, 3]) == {2, 3}
assert find_duplicates([1, 2, 3]) == set()
```

### **SQL Interview**

**Query Writing Process:**
1. Understand the question
2. Identify tables and relationships
3. Start with simple SELECT
4. Add JOINs if needed
5. Add WHERE filters
6. Add GROUP BY/aggregations
7. Add window functions if needed
8. Optimize and test

**Performance Considerations:**
- Avoid SELECT *
- Use indexes on JOIN/WHERE columns
- Minimize subqueries
- Use window functions instead of correlated subqueries
- Filter early (WHERE before JOIN when possible)

---

## 🔗 Practice Resources

### **Python**
- [LeetCode Python](https://leetcode.com/problemset/all/?difficulty=EASY&page=1&topicSlugs=array)
- [HackerRank Python](https://www.hackerrank.com/domains/python)
- [CodeSignal](https://codesignal.com/)
- [Exercism Python Track](https://exercism.org/tracks/python)

### **SQL**
- [LeetCode SQL](https://leetcode.com/problemset/database/)
- [HackerRank SQL](https://www.hackerrank.com/domains/sql)
- [Mode SQL Tutorial](https://mode.com/sql-tutorial/)
- [SQLZoo](https://sqlzoo.net/)
- [StrataScratch](https://www.stratascratch.com/)

### **Books**
- **Python:** "Fluent Python" by Luciano Ramalho
- **SQL:** "SQL Performance Explained" by Markus Winand

---

## 🎯 Key Takeaways

### **Python**
✅ Know your data structures (when to use each)
✅ Write Pythonic code (comprehensions, generators)
✅ Understand time/space complexity
✅ Pandas basics for data manipulation
✅ Clean code with proper error handling

### **SQL**
✅ Master JOINs (all types)
✅ Window functions are critical
✅ CTEs for readable queries
✅ Query optimization mindset
✅ Practice advanced analytics queries

### **Combined Skills**
✅ Python for ETL logic, SQL for transformations
✅ Use pandas for small data, SQL for large data
✅ Both languages complement each other
✅ Most data engineering roles require both

---

## 📝 Quick Reference

### **Python Time Complexity**
- List append: O(1)
- List insert at beginning: O(n)
- Dict lookup: O(1)
- Set lookup: O(1)
- List sort: O(n log n)

### **SQL Query Order**
```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

### **Window Function Template**
```sql
<FUNCTION> OVER (
    [PARTITION BY column]
    [ORDER BY column]
    [ROWS/RANGE BETWEEN ... AND ...]
)
```

---

**Master these fundamentals and you'll ace any data engineering interview! 🚀**

*For detailed answers and examples, see 100-QUESTIONS.md*
*For quick review before interviews, see CHEATSHEET.md*
