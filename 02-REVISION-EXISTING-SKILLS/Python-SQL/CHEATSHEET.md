# Python & SQL Cheatsheet - Quick Reference

## Python Data Structures
- **List**: Ordered, mutable, allows duplicates `[1, 2, 3]`
- **Tuple**: Ordered, immutable `(1, 2, 3)`
- **Set**: Unordered, unique elements `{1, 2, 3}`
- **Dict**: Key-value pairs `{"key": "value"}`
- **List comprehension**: `[x**2 for x in range(10) if x % 2 == 0]`

## Python OOP
- **Class**: Blueprint for objects
- **Instance**: Object created from class
- **Inheritance**: Child class inherits from parent
- **Polymorphism**: Same interface, different implementations
- **Encapsulation**: Hide internal details (private: `_var`, `__var`)
- **@property**: Getter decorator
- **@classmethod**: Method bound to class, not instance
- **@staticmethod**: Independent method in class namespace

## Python Functions
- **args**: Variable positional arguments `*args`
- **kwargs**: Variable keyword arguments `**kwargs`
- **Lambda**: Anonymous function `lambda x: x**2`
- **Decorator**: Modify function behavior `@timer`
- **Generator**: Yield values lazily `yield item`
- **Context manager**: Resource management `with open() as f:`

## Python Advanced
- **List vs Generator**: List stores all, generator computes on-demand
- **map/filter/reduce**: Functional programming (use comprehensions instead)
- **enumerate**: Loop with index `for i, val in enumerate(list)`
- **zip**: Combine iterables `zip(list1, list2)`
- **itertools**: Advanced iteration tools (chain, combinations, groupby)
- **functools**: Higher-order functions (lru_cache, partial, reduce)

## SQL Basics
- **SELECT**: Retrieve data `SELECT col1, col2 FROM table`
- **WHERE**: Filter rows `WHERE amount > 1000`
- **ORDER BY**: Sort results `ORDER BY date DESC`
- **GROUP BY**: Aggregate by groups `GROUP BY customer_id`
- **HAVING**: Filter groups `HAVING COUNT(*) > 5`
- **LIMIT**: Restrict rows returned `LIMIT 100`

## SQL Joins
- **INNER JOIN**: Only matching rows from both tables
- **LEFT JOIN**: All from left + matching from right (NULLs if no match)
- **RIGHT JOIN**: All from right + matching from left
- **FULL OUTER JOIN**: All from both (NULLs where no match)
- **CROSS JOIN**: Cartesian product (every combination)
- **SELF JOIN**: Join table to itself

## SQL Aggregates
- **COUNT()**: Number of rows
- **SUM()**: Total of values
- **AVG()**: Average
- **MIN() / MAX()**: Minimum / Maximum
- **GROUP_CONCAT / STRING_AGG**: Concatenate values

## SQL Window Functions
- **ROW_NUMBER()**: Sequential number (1, 2, 3...)
- **RANK()**: Rank with gaps (1, 2, 2, 4...)
- **DENSE_RANK()**: Rank without gaps (1, 2, 2, 3...)
- **LAG() / LEAD()**: Access previous/next row value
- **NTILE(n)**: Divide into n buckets
- **Syntax**: `OVER (PARTITION BY col ORDER BY col)`

## SQL CTEs & Subqueries
- **CTE (Common Table Expression)**: `WITH cte AS (SELECT ...) SELECT * FROM cte`
- **Subquery**: Query inside query `SELECT * FROM (SELECT ...) sub`
- **Correlated subquery**: Inner query references outer query

## SQL Performance
- **Index**: Speed up lookups (B-tree, Hash, Bitmap)
- **Composite index**: Index on multiple columns
- **EXPLAIN**: Show query execution plan
- **Partitioning**: Split large table by key (date, region)
- **Materialized view**: Precomputed query result

## SQL Data Manipulation
- **INSERT**: Add rows `INSERT INTO table VALUES (...)` 
- **UPDATE**: Modify rows `UPDATE table SET col = val WHERE condition`
- **DELETE**: Remove rows `DELETE FROM table WHERE condition`
- **MERGE / UPSERT**: Insert or update if exists
- **TRUNCATE**: Delete all rows (faster than DELETE, can't rollback)

## SQL Constraints
- **PRIMARY KEY**: Unique identifier (not null, unique)
- **FOREIGN KEY**: Reference to another table
- **UNIQUE**: No duplicates allowed
- **NOT NULL**: Value required
- **CHECK**: Validate condition `CHECK (amount > 0)`
- **DEFAULT**: Default value if not provided

## Python-SQL Integration
- **psycopg2**: PostgreSQL driver
- **pymysql**: MySQL driver
- **SQLAlchemy**: ORM and SQL toolkit
- **pandas.read_sql()**: Query to DataFrame
- **pandas.to_sql()**: DataFrame to table
- **Connection pooling**: Reuse connections for performance

## Best Practices
✅ Use indexes on WHERE/JOIN columns | ✅ Avoid SELECT * | ✅ Use EXPLAIN for optimization | ✅ Parameterize queries (prevent SQL injection) | ✅ Close connections | ✅ Use transactions for consistency | ✅ Normalize data (reduce redundancy) | ✅ Partition large tables | ✅ Regular VACUUM/ANALYZE
