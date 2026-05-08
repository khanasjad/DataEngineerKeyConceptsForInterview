# Python & SQL - 100 Interview Questions & Answers

**Goal:** Master the most commonly asked Python/SQL questions for data engineering roles

---

## PYTHON QUESTIONS (50 Questions)

### Level 1: Fundamentals (15 Questions)

**Q1: What are the key differences between lists and tuples?**

**Answer:**
| Feature | List | Tuple |
|---------|------|-------|
| Mutability | Mutable (can change) | Immutable (cannot change) |
| Syntax | `[1, 2, 3]` | `(1, 2, 3)` |
| Performance | Slower | Faster (immutable) |
| Use case | Dynamic data | Fixed data, dict keys |
| Methods | append, extend, remove, etc. | count, index only |

**Example:**
```python
# List - mutable
my_list = [1, 2, 3]
my_list.append(4)  # Works
my_list[0] = 10    # Works

# Tuple - immutable
my_tuple = (1, 2, 3)
my_tuple.append(4)  # ERROR!
my_tuple[0] = 10    # ERROR!

# Use tuples for dict keys
locations = {(40.7, -74.0): "New York"}  # Works
locations = {[40.7, -74.0]: "New York"}  # ERROR! Lists aren't hashable
```

---

**Q2: Explain list comprehensions with examples.**

**Answer:**
List comprehensions provide concise way to create lists.

**Syntax:** `[expression for item in iterable if condition]`

**Examples:**
```python
# Basic: squares of numbers
squares = [x**2 for x in range(10)]
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# With condition: even squares
even_squares = [x**2 for x in range(10) if x % 2 == 0]
# [0, 4, 16, 36, 64]

# Nested loops: flatten matrix
matrix = [[1, 2], [3, 4], [5, 6]]
flat = [num for row in matrix for num in row]
# [1, 2, 3, 4, 5, 6]

# String manipulation
names = ['alice', 'bob', 'charlie']
capitalized = [name.capitalize() for name in names]
# ['Alice', 'Bob', 'Charlie']

# Data engineering use case: extract user IDs
events = [
    {'user_id': 1, 'event': 'click'},
    {'user_id': 2, 'event': 'view'},
    {'user_id': 1, 'event': 'purchase'}
]
user_ids = [event['user_id'] for event in events]
# [1, 2, 1]
```

**Performance:** List comprehensions are faster than for loops (~30% faster)

---

**Q3: What is the difference between `==` and `is`?**

**Answer:**
- `==` checks **value equality** (are the values the same?)
- `is` checks **identity** (are they the same object in memory?)

**Examples:**
```python
# Example 1: Lists
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)  # True (same values)
print(a is b)  # False (different objects)
print(a is c)  # True (same object)

# Example 2: Integers (Python caches small integers -5 to 256)
x = 256
y = 256
print(x is y)  # True (cached)

x = 257
y = 257
print(x is y)  # False (not cached, different objects)

# Example 3: None (always use 'is' with None)
value = None
if value is None:  # Correct ✓
    print("Value is None")

if value == None:  # Works but not pythonic ✗
    print("Value is None")
```

**Best Practice:** Use `is` for `None`, `True`, `False`; use `==` for values

---

**Q4: Explain args and kwargs.**

**Answer:**
- `*args`: Pass variable number of **positional** arguments (tuple)
- `**kwargs`: Pass variable number of **keyword** arguments (dict)

**Examples:**
```python
# *args example
def sum_all(*args):
    return sum(args)

print(sum_all(1, 2, 3))        # 6
print(sum_all(1, 2, 3, 4, 5))  # 15

# **kwargs example
def print_user_info(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_user_info(name="Alice", age=30, city="NYC")
# name: Alice
# age: 30
# city: NYC

# Combined: args then kwargs
def process_data(*args, **kwargs):
    print(f"Positional: {args}")
    print(f"Keyword: {kwargs}")

process_data(1, 2, 3, name="test", debug=True)
# Positional: (1, 2, 3)
# Keyword: {'name': 'test', 'debug': True}

# Data engineering use case: flexible log function
def log_event(event_type, *tags, **metadata):
    print(f"Event: {event_type}")
    print(f"Tags: {tags}")
    print(f"Metadata: {metadata}")

log_event("user_login", "auth", "security",
          user_id=123, ip="192.168.1.1", timestamp="2026-05-03")
```

---

**Q5: What are decorators? Give examples.**

**Answer:**
Decorators modify or enhance functions without changing their code. They wrap a function with additional functionality.

**Basic Example:**
```python
# Simple decorator
def timing_decorator(func):
    def wrapper(*args, **kwargs):
        import time
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timing_decorator
def slow_function():
    import time
    time.sleep(1)
    return "Done"

slow_function()
# Output: slow_function took 1.0012 seconds
```

**Data Engineering Examples:**
```python
# 1. Retry decorator (for API calls)
def retry(max_attempts=3):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed: {e}")
            return wrapper
    return decorator

@retry(max_attempts=3)
def fetch_api_data(url):
    # May fail due to network issues
    import requests
    return requests.get(url).json()

# 2. Logging decorator
def log_execution(func):
    def wrapper(*args, **kwargs):
        print(f"Executing {func.__name__} with args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@log_execution
def process_batch(batch_id, debug=False):
    return f"Processed batch {batch_id}"

# 3. Cache decorator (memoization)
def cache(func):
    cache_dict = {}
    def wrapper(*args):
        if args not in cache_dict:
            cache_dict[args] = func(*args)
        return cache_dict[args]
    return wrapper

@cache
def expensive_computation(n):
    # Simulate expensive operation
    return sum(range(n))

expensive_computation(1000000)  # Slow first time
expensive_computation(1000000)  # Instant second time (cached)
```

**Built-in Decorators:**
```python
class DataProcessor:
    _instance = None

    @staticmethod
    def process_static(data):
        # No access to instance/class
        return data.upper()

    @classmethod
    def get_instance(cls):
        # Access to class
        if cls._instance is None:
            cls._instance = cls()
        return cls._instance

    @property
    def status(self):
        # Use like attribute: obj.status instead of obj.status()
        return "running"
```

---

**Q6-Q15: Quick Fire Fundamentals**

**Q6: What is the difference between `append()` and `extend()`?**
```python
# append() adds element as-is
my_list = [1, 2, 3]
my_list.append([4, 5])
# [1, 2, 3, [4, 5]]  ← nested list

# extend() adds each element
my_list = [1, 2, 3]
my_list.extend([4, 5])
# [1, 2, 3, 4, 5]  ← flattened
```

**Q7: Explain shallow copy vs deep copy**
```python
import copy

original = [[1, 2], [3, 4]]

# Shallow copy: copies list but not nested objects
shallow = copy.copy(original)
shallow[0][0] = 999
print(original)  # [[999, 2], [3, 4]] ← CHANGED!

# Deep copy: copies everything recursively
deep = copy.deepcopy(original)
deep[0][0] = 777
print(original)  # [[1, 2], [3, 4]] ← unchanged
```

**Q8: What are lambda functions?**
```python
# Lambda: anonymous single-expression function
square = lambda x: x**2
square(5)  # 25

# Common use: sorting
users = [{'name': 'Bob', 'age': 30}, {'name': 'Alice', 'age': 25}]
sorted_users = sorted(users, key=lambda u: u['age'])
# Alice (25) before Bob (30)

# Map, filter, reduce
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))  # [1, 4, 9, 16, 25]
evens = list(filter(lambda x: x % 2 == 0, numbers))  # [2, 4]
```

**Q9: What is `*` and `/` in function signatures?**
```python
# * forces keyword-only arguments after it
def func(a, b, *, c, d):
    pass

func(1, 2, c=3, d=4)  # OK
func(1, 2, 3, 4)      # ERROR! c and d must be keyword args

# / forces positional-only arguments before it (Python 3.8+)
def func(a, b, /, c, d):
    pass

func(1, 2, 3, 4)      # OK
func(a=1, b=2, c=3, d=4)  # ERROR! a and b must be positional
```

**Q10: Explain `enumerate()`**
```python
# enumerate() adds counter to iterable
names = ['Alice', 'Bob', 'Charlie']

# Without enumerate
for i in range(len(names)):
    print(f"{i}: {names[i]}")

# With enumerate (better!)
for i, name in enumerate(names):
    print(f"{i}: {name}")

# Custom start index
for i, name in enumerate(names, start=1):
    print(f"{i}: {name}")  # 1: Alice, 2: Bob, 3: Charlie
```

**Q11: What is `zip()`?**
```python
# zip() combines multiple iterables
names = ['Alice', 'Bob']
ages = [25, 30]
cities = ['NYC', 'LA']

for name, age, city in zip(names, ages, cities):
    print(f"{name}, {age}, {city}")

# Create dict from two lists
user_dict = dict(zip(names, ages))
# {'Alice': 25, 'Bob': 30}

# Unzip (transpose)
pairs = [(1, 'a'), (2, 'b'), (3, 'c')]
numbers, letters = zip(*pairs)
# numbers = (1, 2, 3), letters = ('a', 'b', 'c')
```

**Q12: Explain `any()` and `all()`**
```python
# any(): True if ANY element is True
any([False, False, True, False])  # True
any([False, False, False])        # False
any([])                           # False (empty)

# all(): True if ALL elements are True
all([True, True, True])   # True
all([True, False, True])  # False
all([])                   # True (empty - vacuous truth)

# Practical use
numbers = [1, 2, 3, 4, 5]
has_even = any(n % 2 == 0 for n in numbers)  # True
all_positive = all(n > 0 for n in numbers)   # True
```

**Q13: What are `set` operations?**
```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

# Union (all elements)
a | b  # {1, 2, 3, 4, 5, 6}
a.union(b)

# Intersection (common elements)
a & b  # {3, 4}
a.intersection(b)

# Difference (in a, not in b)
a - b  # {1, 2}
a.difference(b)

# Symmetric difference (in either, not both)
a ^ b  # {1, 2, 5, 6}
a.symmetric_difference(b)

# Data engineering use: find unique user IDs
user_ids_today = {1, 2, 3, 4, 5}
user_ids_yesterday = {4, 5, 6, 7, 8}
new_users = user_ids_today - user_ids_yesterday  # {1, 2, 3}
```

**Q14: Explain `defaultdict` and `Counter`**
```python
from collections import defaultdict, Counter

# defaultdict: never raises KeyError
# Regular dict
user_events = {}
user_events['alice'] = user_events.get('alice', []) + ['login']  # verbose

# defaultdict
user_events = defaultdict(list)
user_events['alice'].append('login')  # clean!
user_events['bob'].append('purchase')

# Counter: count occurrences
events = ['click', 'view', 'click', 'click', 'purchase', 'view']
count = Counter(events)
# Counter({'click': 3, 'view': 2, 'purchase': 1})

count.most_common(2)  # [('click', 3), ('view', 2)]

# Data engineering: count events by type
from collections import Counter
event_types = [e['type'] for e in events]
type_counts = Counter(event_types)
```

**Q15: What is GIL (Global Interpreter Lock)?**

**Answer:**
GIL is a mutex that protects Python objects, preventing multiple threads from executing Python code simultaneously.

**Impact:**
- **CPU-bound tasks:** Threading doesn't help (GIL blocks)
- **I/O-bound tasks:** Threading helps (GIL released during I/O)

**Solutions:**
```python
# For CPU-bound: Use multiprocessing
from multiprocessing import Pool

def process_data(chunk):
    # CPU-intensive work
    return sum(chunk)

with Pool(processes=4) as pool:
    results = pool.map(process_data, data_chunks)

# For I/O-bound: Threading is fine
from concurrent.futures import ThreadPoolExecutor

def fetch_url(url):
    # I/O operation (GIL released)
    return requests.get(url).text

with ThreadPoolExecutor(max_workers=10) as executor:
    results = executor.map(fetch_url, urls)
```

---

### Level 2: Intermediate (20 Questions)

**Q16: Explain context managers and `with` statement**

**Answer:**
Context managers handle setup and cleanup automatically (RAII pattern).

**Built-in Examples:**
```python
# File handling
with open('data.txt', 'r') as f:
    data = f.read()
# File automatically closed, even if exception occurs

# Database connection
with psycopg2.connect(conn_string) as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT * FROM users")
# Connection and cursor automatically closed

# Lock management
from threading import Lock
lock = Lock()

with lock:
    # Critical section
    shared_resource += 1
# Lock automatically released
```

**Custom Context Manager:**
```python
# Method 1: Class-based
class Timer:
    def __enter__(self):
        self.start = time.time()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.end = time.time()
        print(f"Elapsed: {self.end - self.start:.4f}s")
        return False  # Don't suppress exceptions

with Timer():
    time.sleep(1)  # Elapsed: 1.0001s

# Method 2: Generator-based (@contextmanager)
from contextlib import contextmanager

@contextmanager
def database_transaction(conn):
    try:
        yield conn
        conn.commit()
    except Exception:
        conn.rollback()
        raise

with database_transaction(conn):
    conn.execute("INSERT INTO ...")
# Automatically commits or rolls back
```

**Data Engineering Use Case:**
```python
@contextmanager
def spark_session(app_name):
    from pyspark.sql import SparkSession
    spark = SparkSession.builder.appName(app_name).getOrCreate()
    try:
        yield spark
    finally:
        spark.stop()

with spark_session("ETL Job") as spark:
    df = spark.read.parquet("s3://data/")
    df.write.parquet("s3://output/")
# Spark automatically stopped
```

---

**Q17: What are generators? When to use them?**

**Answer:**
Generators are iterators that generate values on-the-fly using `yield`, saving memory.

**Regular Function vs Generator:**
```python
# Regular function: Creates entire list in memory
def get_numbers_list(n):
    result = []
    for i in range(n):
        result.append(i**2)
    return result

numbers = get_numbers_list(1000000)  # Uses ~38MB memory
# Must create entire list before using

# Generator: Creates values one at a time
def get_numbers_generator(n):
    for i in range(n):
        yield i**2

numbers = get_numbers_generator(1000000)  # Uses ~80 bytes!
# Values created as needed

for num in numbers:
    print(num)  # Each number generated on demand
```

**Benefits:**
1. **Memory efficient:** Don't store entire sequence
2. **Lazy evaluation:** Compute only when needed
3. **Infinite sequences:** Can represent infinite streams

**Data Engineering Examples:**
```python
# 1. Read large file line by line
def read_large_file(file_path):
    with open(file_path) as f:
        for line in f:
            yield line.strip()

for line in read_large_file('huge_log.txt'):
    process(line)  # Memory-efficient!

# 2. Batch processing
def batch_generator(items, batch_size):
    batch = []
    for item in items:
        batch.append(item)
        if len(batch) == batch_size:
            yield batch
            batch = []
    if batch:  # yield remaining
        yield batch

# Process 1M records in batches of 1000
for batch in batch_generator(all_records, 1000):
    process_batch(batch)

# 3. Streaming data pipeline
def transform_stream(input_stream):
    for record in input_stream:
        # Transform
        transformed = transform(record)
        # Filter
        if is_valid(transformed):
            yield transformed

# Chain generators
data = read_large_file('input.json')
parsed = (json.loads(line) for line in data)  # Generator expression
transformed = transform_stream(parsed)
for record in transformed:
    write_to_db(record)
```

**Generator Expression:**
```python
# Similar to list comprehension, but with ()
# List comprehension (entire list in memory)
squares_list = [x**2 for x in range(1000000)]

# Generator expression (lazy)
squares_gen = (x**2 for x in range(1000000))

# Use case: sum of large sequence
total = sum(x**2 for x in range(1000000))  # Memory-efficient!
```

---

**Q18: Explain `map()`, `filter()`, `reduce()`**

**Answer:**
Functional programming tools for transforming collections.

**map()**: Apply function to each element
```python
# Convert to uppercase
names = ['alice', 'bob', 'charlie']
upper_names = list(map(str.upper, names))
# ['ALICE', 'BOB', 'CHARLIE']

# Multiple iterables
nums1 = [1, 2, 3]
nums2 = [4, 5, 6]
sums = list(map(lambda x, y: x + y, nums1, nums2))
# [5, 7, 9]

# Data engineering: parse dates
date_strings = ['2026-01-01', '2026-01-02']
from datetime import datetime
dates = list(map(lambda s: datetime.strptime(s, '%Y-%m-%d'), date_strings))
```

**filter()**: Keep elements matching condition
```python
# Keep even numbers
numbers = [1, 2, 3, 4, 5, 6]
evens = list(filter(lambda x: x % 2 == 0, numbers))
# [2, 4, 6]

# Filter records
users = [
    {'name': 'Alice', 'age': 25},
    {'name': 'Bob', 'age': 17},
    {'name': 'Charlie', 'age': 30}
]
adults = list(filter(lambda u: u['age'] >= 18, users))
# [{'name': 'Alice', ...}, {'name': 'Charlie', ...}]
```

**reduce()**: Accumulate values into single result
```python
from functools import reduce

# Sum
numbers = [1, 2, 3, 4, 5]
total = reduce(lambda acc, x: acc + x, numbers)
# 15

# Find maximum
maximum = reduce(lambda a, b: a if a > b else b, numbers)
# 5

# Flatten nested list
nested = [[1, 2], [3, 4], [5, 6]]
flat = reduce(lambda acc, lst: acc + lst, nested, [])
# [1, 2, 3, 4, 5, 6]

# Data engineering: merge dicts
dicts = [{'a': 1}, {'b': 2}, {'c': 3}]
merged = reduce(lambda acc, d: {**acc, **d}, dicts, {})
# {'a': 1, 'b': 2, 'c': 3}
```

**Modern Alternatives (often clearer):**
```python
# Instead of map
[name.upper() for name in names]  # List comprehension

# Instead of filter
[x for x in numbers if x % 2 == 0]  # List comprehension with condition

# Instead of reduce for sum
sum(numbers)  # Built-in sum
max(numbers)  # Built-in max
```

---

**Q19-Q35: Intermediate Topics (condensed)**

Due to space, I'll provide condensed answers for remaining intermediate questions:

**Q19: Exception handling best practices**
```python
# Specific exceptions
try:
    result = int(user_input)
except ValueError as e:
    print(f"Invalid number: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
finally:
    cleanup()  # Always runs

# Context manager for resources
# Raise custom exceptions
class DataValidationError(Exception):
    pass

if not is_valid(data):
    raise DataValidationError("Invalid data format")
```

**Q20: `__init__` vs `__new__`**
- `__new__`: Creates instance (rarely overridden)
- `__init__`: Initializes instance (commonly used)

**Q21: `@staticmethod` vs `@classmethod`**
```python
class MyClass:
    @staticmethod
    def static_method():
        # No access to class or instance
        return "static"

    @classmethod
    def class_method(cls):
        # Access to class (cls)
        return cls.__name__
```

**Q22: What are metaclasses?** (Advanced - rarely asked)
Classes that create classes. Usually don't need them.

**Q23: Explain `__slots__`**
```python
class User:
    __slots__ = ['name', 'age']  # Restrict attributes, save memory
```

**Q24: What is monkey patching?**
```python
# Modify class/module at runtime (use cautiously!)
import math
math.pi = 3  # Don't do this in production!
```

**Q25: `@property` decorator**
```python
class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius

    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32

    @fahrenheit.setter
    def fahrenheit(self, value):
        self._celsius = (value - 32) * 5/9

temp = Temperature(0)
print(temp.fahrenheit)  # 32 (use like attribute!)
temp.fahrenheit = 100   # Setter called
```

**Q26-Q35: Quick answers**
- **Q26: Threading vs Multiprocessing:** Threading for I/O, Multiprocessing for CPU
- **Q27: `async`/`await`:** Asynchronous programming for concurrent I/O
- **Q28: `itertools`:** Tools for iterators (combinations, permutations, etc.)
- **Q29: Regular expressions:** Pattern matching with `re` module
- **Q30: Pickling:** Serialize Python objects
- **Q31: `__str__` vs `__repr__`:** Human-readable vs unambiguous representation
- **Q32: Duck typing:** "If it walks like a duck..." - no need for explicit types
- **Q33: Type hints:** PEP 484 type annotations (Python 3.5+)
- **Q34: Virtual environments:** Isolated Python environments
- **Q35: `requirements.txt`:** List of package dependencies

---

### Level 3: Advanced / Data Engineering Specific (15 Questions)

**Q36: How do you optimize Python code for large datasets?**

**Answer:**
Multiple strategies depending on bottleneck:

**1. Use appropriate data structures:**
```python
# Bad: List for membership testing
user_ids = [1, 2, 3, ...1000000]  # List
if user_id in user_ids:  # O(n) - slow!

# Good: Set for membership testing
user_ids = {1, 2, 3, ...1000000}  # Set
if user_id in user_ids:  # O(1) - fast!

# Bad: Concatenating strings in loop
result = ""
for item in items:
    result += str(item)  # Creates new string each time

# Good: Join
result = "".join(str(item) for item in items)
```

**2. Use pandas for tabular data:**
```python
# Much faster than Python loops for dataframes
import pandas as pd
df = pd.read_csv('large_file.csv')
df['new_col'] = df['col1'] * df['col2']  # Vectorized - fast!
```

**3. Use generators for large files:**
```python
# Process line by line (memory-efficient)
def process_large_file(filename):
    with open(filename) as f:
        for line in f:
            yield process(line)
```

**4. Use multiprocessing for CPU-bound:**
```python
from multiprocessing import Pool

def process_chunk(chunk):
    return [transform(item) for item in chunk]

with Pool(processes=8) as pool:
    results = pool.map(process_chunk, chunks)
```

**5. Use Cython/Numba for hot paths:**
```python
from numba import jit

@jit(nopython=True)
def compute_intensive(arr):
    # Compiled to machine code
    total = 0
    for i in arr:
        total += i ** 2
    return total
```

---

**Q37: Explain Python's memory management and garbage collection**

**Answer:**

**Reference Counting:**
- Python tracks how many references point to each object
- When count reaches 0, memory is freed immediately

**Example:**
```python
a = [1, 2, 3]  # ref count = 1
b = a          # ref count = 2
del a          # ref count = 1
del b          # ref count = 0 → freed
```

**Garbage Collector (for cycles):**
- Detects reference cycles
- Runs periodically or manually

```python
import gc

# Example: Circular reference
class Node:
    def __init__(self):
        self.ref = None

a = Node()
b = Node()
a.ref = b
b.ref = a  # Circular reference!
del a, del b  # Won't be freed by ref counting alone

gc.collect()  # Manually trigger GC to clean up cycles
```

**Memory Optimization Tips:**
```python
# 1. Use __slots__ to reduce memory
class Point:
    __slots__ = ['x', 'y']  # No __dict__, saves memory

# 2. Use generators instead of lists
# 3. Delete large objects when done
del large_dataframe
gc.collect()

# 4. Monitor memory
import sys
sys.getsizeof(my_object)  # Size in bytes
```

---

**Q38-Q50: Data Engineering Specific (condensed)**

**Q38: How do you read a 100GB file in Python?**
```python
# Never load entire file!
# Option 1: Line by line
with open('huge_file.txt') as f:
    for line in f:  # Iterator - memory efficient
        process(line)

# Option 2: Chunks
chunk_size = 1024 * 1024  # 1MB
with open('huge_file.txt') as f:
    while True:
        chunk = f.read(chunk_size)
        if not chunk:
            break
        process(chunk)

# Option 3: pandas chunks
for chunk in pd.read_csv('huge.csv', chunksize=10000):
    process(chunk)
```

**Q39: Parallel processing patterns**
```python
# Threading (I/O bound)
from concurrent.futures import ThreadPoolExecutor
with ThreadPoolExecutor(max_workers=10) as executor:
    results = executor.map(fetch_url, urls)

# Multiprocessing (CPU bound)
from multiprocessing import Pool
with Pool(processes=8) as pool:
    results = pool.map(process_data, chunks)
```

**Q40-Q50: Quick answers**
- **Q40:** `asyncio` for async I/O
- **Q41:** Connection pooling with `psycopg2.pool`
- **Q42:** Batch inserts for database performance
- **Q43:** `logging` module for production
- **Q44:** `unittest` and `pytest` for testing
- **Q45:** Type checking with `mypy`
- **Q46:** Profiling with `cProfile`
- **Q47:** Debugging with `pdb`
- **Q48:** Virtual environments with `venv`
- **Q49:** Package management with `pip`
- **Q50:** Code formatting with `black`

---

## SQL QUESTIONS (50 Questions)

### Level 1: Fundamentals (15 Questions)

**Q51: Difference between WHERE and HAVING?**

**Answer:**
- **WHERE:** Filters rows **before** grouping
- **HAVING:** Filters groups **after** aggregation

**Example:**
```sql
-- WHERE: Filter before GROUP BY
SELECT department, COUNT(*) as emp_count
FROM employees
WHERE salary > 50000  -- Filter individual rows
GROUP BY department;

-- HAVING: Filter after GROUP BY
SELECT department, AVG(salary) as avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000;  -- Filter aggregated groups

-- Both together
SELECT department, AVG(salary) as avg_salary
FROM employees
WHERE hire_date > '2020-01-01'  -- Individual row filter
GROUP BY department
HAVING AVG(salary) > 70000;     -- Group filter
```

**Memory:**
- WHERE = **W**hen (before grouping)
- HAVING = **H**as been grouped (after aggregation)

---

**Q52: Explain different types of JOINS with examples**

**Answer:**

**INNER JOIN:** Only matching rows from both tables
```sql
SELECT o.order_id, c.customer_name
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id;
-- Returns only orders with valid customers
```

**LEFT JOIN:** All from left, matching from right (NULL if no match)
```sql
SELECT c.customer_name, o.order_id
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
-- Returns ALL customers, even those without orders (order_id = NULL)
```

**RIGHT JOIN:** All from right, matching from left
```sql
SELECT c.customer_name, o.order_id
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id;
-- Returns ALL orders, even orphaned ones (customer_name = NULL)
```

**FULL OUTER JOIN:** All from both tables
```sql
SELECT c.customer_name, o.order_id
FROM customers c
FULL OUTER JOIN orders o ON c.customer_id = o.customer_id;
-- Returns all customers AND all orders, with NULLs where no match
```

**CROSS JOIN:** Cartesian product (every combination)
```sql
SELECT *
FROM colors CROSS JOIN sizes;
-- If 3 colors and 4 sizes → 12 rows (3 × 4)
```

**SELF JOIN:** Join table to itself
```sql
-- Find employees and their managers
SELECT e.name as employee, m.name as manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

**Visual:**
```
INNER:     [==]        Only overlap
LEFT:      [=====]     All left + overlap
RIGHT:     [=====]     All right + overlap
FULL:      [=======]   Everything
```

---

**Q53: What are window functions? Give examples.**

**Answer:**
Window functions perform calculations across rows related to current row, without grouping.

**Basic Syntax:**
```sql
function(...) OVER (
    [PARTITION BY ...]
    [ORDER BY ...]
    [ROWS/RANGE ...]
)
```

**ROW_NUMBER():** Assign unique sequential number
```sql
SELECT
    name,
    department,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) as overall_rank,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as dept_rank
FROM employees;

-- Result:
-- name    | dept  | salary | overall_rank | dept_rank
-- Alice   | Sales | 90000  | 1            | 1
-- Bob     | Eng   | 85000  | 2            | 1
-- Charlie | Sales | 75000  | 3            | 2
-- David   | Eng   | 70000  | 4            | 2
```

**RANK() and DENSE_RANK():**
```sql
-- RANK: Gaps in ranking for ties
-- DENSE_RANK: No gaps

SELECT
    name,
    score,
    RANK() OVER (ORDER BY score DESC) as rank,
    DENSE_RANK() OVER (ORDER BY score DESC) as dense_rank
FROM students;

-- score | rank | dense_rank
-- 95    | 1    | 1
-- 95    | 1    | 1  ← tied
-- 90    | 3    | 2  ← RANK skips 2, DENSE_RANK doesn't
-- 85    | 4    | 3
```

**LAG() and LEAD():** Access previous/next row
```sql
-- Calculate day-over-day change
SELECT
    date,
    revenue,
    LAG(revenue) OVER (ORDER BY date) as prev_day_revenue,
    revenue - LAG(revenue) OVER (ORDER BY date) as daily_change
FROM daily_sales;
```

**NTILE():** Divide into buckets
```sql
-- Divide customers into quartiles by purchase amount
SELECT
    customer_id,
    total_purchases,
    NTILE(4) OVER (ORDER BY total_purchases) as quartile
FROM customer_summary;
-- Quartile 1 = bottom 25%, Quartile 4 = top 25%
```

**Aggregate functions as window functions:**
```sql
-- Running total
SELECT
    date,
    revenue,
    SUM(revenue) OVER (ORDER BY date) as running_total,
    AVG(revenue) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as moving_avg_7day
FROM daily_sales;
```

---

**Q54: Explain UNION vs UNION ALL**

**Answer:**
- **UNION:** Combines results and removes duplicates (slower)
- **UNION ALL:** Combines results keeping duplicates (faster)

**Example:**
```sql
-- Table A: [1, 2, 3]
-- Table B: [2, 3, 4]

SELECT * FROM A
UNION
SELECT * FROM B;
-- Result: [1, 2, 3, 4]  ← duplicates removed

SELECT * FROM A
UNION ALL
SELECT * FROM B;
-- Result: [1, 2, 3, 2, 3, 4]  ← duplicates kept
```

**When to use:**
```sql
-- UNION: When you need distinct results
SELECT customer_id FROM orders_2024
UNION
SELECT customer_id FROM orders_2025;
-- Get unique customers who ordered in either year

-- UNION ALL: When you know no duplicates OR want them
SELECT customer_id FROM orders_2024
UNION ALL
SELECT customer_id FROM orders_2025;
-- Get all orders (allows counting total customer orders)
```

**Performance:** UNION ALL is faster (no de-duplication overhead)

---

**Q55-Q65: Fundamentals (condensed)**

**Q55: Primary Key vs Unique Key**
- Primary Key: NOT NULL + UNIQUE, one per table
- Unique Key: Can be NULL, multiple allowed

**Q56: DELETE vs TRUNCATE vs DROP**
```sql
DELETE FROM table WHERE condition;  -- Remove rows, can rollback, slow
TRUNCATE TABLE table;               -- Remove all rows, fast, can't rollback
DROP TABLE table;                   -- Delete table structure
```

**Q57: GROUP BY with multiple columns**
```sql
SELECT department, job_title, AVG(salary)
FROM employees
GROUP BY department, job_title;
-- Groups by each unique combination
```

**Q58: CASE statement**
```sql
SELECT
    name,
    salary,
    CASE
        WHEN salary > 100000 THEN 'High'
        WHEN salary > 70000 THEN 'Medium'
        ELSE 'Low'
    END as salary_band
FROM employees;
```

**Q59: COALESCE**
```sql
SELECT COALESCE(phone, email, 'No contact') as contact
FROM customers;
-- Returns first non-NULL value
```

**Q60: Subqueries**
```sql
-- Scalar subquery (returns single value)
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Correlated subquery
SELECT e1.name, e1.salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department = e1.department
);
```

**Q61: EXISTS vs IN**
```sql
-- EXISTS: Better for large outer table
SELECT * FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);

-- IN: Better for small list
SELECT * FROM customers
WHERE customer_id IN (SELECT customer_id FROM vip_customers);
```

**Q62: Date functions**
```sql
-- PostgreSQL/MySQL examples
SELECT
    CURRENT_DATE,
    CURRENT_TIMESTAMP,
    DATE_TRUNC('month', order_date),
    EXTRACT(YEAR FROM order_date),
    order_date + INTERVAL '7 days'
FROM orders;
```

**Q63: String functions**
```sql
SELECT
    UPPER(name),
    LOWER(email),
    CONCAT(first_name, ' ', last_name),
    LENGTH(description),
    SUBSTRING(code, 1, 3),
    REPLACE(text, 'old', 'new')
FROM users;
```

**Q64: Aggregate functions**
- COUNT, SUM, AVG, MIN, MAX
- COUNT(*) vs COUNT(column): NULL handling
- DISTINCT: COUNT(DISTINCT column)

**Q65: ORDER BY with NULL**
```sql
SELECT * FROM table
ORDER BY column NULLS FIRST;  -- NULLs at top
ORDER BY column NULLS LAST;   -- NULLs at bottom
```

---

### Level 2: Intermediate (20 Questions)

**Q66: Write a query to find the 2nd highest salary**

**Answer:**
Multiple approaches:

**Method 1: Subquery with LIMIT/OFFSET**
```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

**Method 2: MAX with subquery**
```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

**Method 3: DENSE_RANK (best for ties)**
```sql
WITH ranked AS (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as rank
    FROM employees
)
SELECT DISTINCT salary
FROM ranked
WHERE rank = 2;
```

**Method 4: Generalized Nth highest**
```sql
-- Get Nth highest (N=2)
WITH ranked AS (
    SELECT DISTINCT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) as rank
    FROM employees
)
SELECT salary
FROM ranked
WHERE rank = 2;
```

---

**Q67: Find duplicate records**

**Answer:**
```sql
-- Method 1: GROUP BY + HAVING
SELECT email, COUNT(*) as count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Method 2: Self-join
SELECT DISTINCT u1.*
FROM users u1
JOIN users u2 ON u1.email = u2.email AND u1.id <> u2.id;

-- Method 3: Window function
WITH duplicates AS (
    SELECT *,
           COUNT(*) OVER (PARTITION BY email) as dup_count
    FROM users
)
SELECT *
FROM duplicates
WHERE dup_count > 1;

-- Delete duplicates, keep one
DELETE FROM users
WHERE id NOT IN (
    SELECT MIN(id)
    FROM users
    GROUP BY email
);
```

---

**Q68: Consecutive numbers problem**

**Problem:** Find numbers that appear at least 3 times consecutively

**Answer:**
```sql
-- Table: Logs (id, num)
-- id | num
-- 1  | 1
-- 2  | 1
-- 3  | 1   ← These 3 are consecutive
-- 4  | 2
-- 5  | 1

WITH consecutive AS (
    SELECT
        num,
        id,
        LEAD(num, 1) OVER (ORDER BY id) as next1,
        LEAD(num, 2) OVER (ORDER BY id) as next2
    FROM Logs
)
SELECT DISTINCT num
FROM consecutive
WHERE num = next1 AND num = next2;
```

---

**Q69: Calculate running total**

**Answer:**
```sql
-- Method 1: Window function (modern, fast)
SELECT
    date,
    revenue,
    SUM(revenue) OVER (ORDER BY date) as running_total
FROM daily_sales
ORDER BY date;

-- Method 2: Correlated subquery (older databases)
SELECT
    s1.date,
    s1.revenue,
    (SELECT SUM(s2.revenue)
     FROM daily_sales s2
     WHERE s2.date <= s1.date) as running_total
FROM daily_sales s1
ORDER BY s1.date;
```

---

**Q70: Pivot table in SQL**

**Problem:** Convert rows to columns

**Answer:**
```sql
-- Input:
-- user_id | metric  | value
-- 1       | clicks  | 100
-- 1       | views   | 500
-- 2       | clicks  | 150

-- Output:
-- user_id | clicks | views
-- 1       | 100    | 500
-- 2       | 150    | NULL

-- Method 1: CASE statements
SELECT
    user_id,
    SUM(CASE WHEN metric = 'clicks' THEN value END) as clicks,
    SUM(CASE WHEN metric = 'views' THEN value END) as views
FROM user_metrics
GROUP BY user_id;

-- Method 2: PIVOT (SQL Server, Oracle)
SELECT *
FROM user_metrics
PIVOT (
    SUM(value)
    FOR metric IN ('clicks', 'views')
) AS pivoted;
```

---

**Q71-Q85: Intermediate SQL (condensed)**

**Q71: Self-join to find pairs**
```sql
-- Find all pairs of employees in same department
SELECT e1.name, e2.name
FROM employees e1
JOIN employees e2 ON e1.department = e2.department
WHERE e1.id < e2.id;  -- Avoid duplicates and self-pairs
```

**Q72: Date range overlaps**
```sql
-- Find overlapping bookings
SELECT b1.*, b2.*
FROM bookings b1
JOIN bookings b2 ON b1.room_id = b2.room_id
WHERE b1.id < b2.id
  AND b1.end_date >= b2.start_date
  AND b1.start_date <= b2.end_date;
```

**Q73: Gap and islands problem**
```sql
-- Find consecutive date ranges
WITH groups AS (
    SELECT
        date,
        date - ROW_NUMBER() OVER (ORDER BY date)::int as grp
    FROM attendance
)
SELECT
    MIN(date) as start_date,
    MAX(date) as end_date
FROM groups
GROUP BY grp;
```

**Q74: Median calculation**
```sql
-- PostgreSQL
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary)
FROM employees;

-- Window function approach
WITH ordered AS (
    SELECT salary,
           ROW_NUMBER() OVER (ORDER BY salary) as rn,
           COUNT(*) OVER () as total
    FROM employees
)
SELECT AVG(salary) as median
FROM ordered
WHERE rn IN (FLOOR((total + 1) / 2.0), CEIL((total + 1) / 2.0));
```

**Q75: Recursive CTE**
```sql
-- Organization hierarchy
WITH RECURSIVE emp_hierarchy AS (
    -- Anchor: Top-level employees
    SELECT employee_id, name, manager_id, 1 as level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive: Employees reporting to current level
    SELECT e.employee_id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN emp_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM emp_hierarchy;
```

**Q76: How do you create a date dimension table?**

**Answer:**
A date dimension table is a pre-populated calendar table used in star schema data warehouses for efficient date-based analysis.

**Why needed:**
- Fast date-based filtering (avoid date calculations in queries)
- Business calendar logic (fiscal quarters, holidays, weekends)
- Pre-calculated date attributes
- Consistent date handling across reports

**Implementation:**
```sql
CREATE TABLE dim_date (
    date_key INT PRIMARY KEY,           -- 20240503 (surrogate key)
    date DATE NOT NULL UNIQUE,          -- 2024-05-03
    day_of_week INT,                    -- 1-7 (1=Monday)
    day_name VARCHAR(10),               -- Monday, Tuesday, etc.
    day_of_month INT,                   -- 1-31
    day_of_year INT,                    -- 1-366
    week_of_year INT,                   -- 1-53
    month INT,                          -- 1-12
    month_name VARCHAR(10),             -- January, February, etc.
    quarter INT,                        -- 1-4
    year INT,                           -- 2024
    is_weekend BOOLEAN,                 -- TRUE/FALSE
    is_holiday BOOLEAN,                 -- TRUE/FALSE
    holiday_name VARCHAR(50),           -- "New Year's Day", NULL
    fiscal_year INT,                    -- 2024 (if fiscal year != calendar)
    fiscal_quarter INT,                 -- 1-4
    fiscal_period INT                   -- 1-12
);

-- Generate date dimension data (PostgreSQL)
INSERT INTO dim_date
SELECT
    TO_CHAR(date, 'YYYYMMDD')::INT as date_key,
    date,
    EXTRACT(ISODOW FROM date) as day_of_week,
    TO_CHAR(date, 'Day') as day_name,
    EXTRACT(DAY FROM date) as day_of_month,
    EXTRACT(DOY FROM date) as day_of_year,
    EXTRACT(WEEK FROM date) as week_of_year,
    EXTRACT(MONTH FROM date) as month,
    TO_CHAR(date, 'Month') as month_name,
    EXTRACT(QUARTER FROM date) as quarter,
    EXTRACT(YEAR FROM date) as year,
    EXTRACT(ISODOW FROM date) IN (6, 7) as is_weekend,
    FALSE as is_holiday,  -- Update separately
    NULL as holiday_name,
    -- Fiscal year (example: starts July 1)
    CASE
        WHEN EXTRACT(MONTH FROM date) >= 7 THEN EXTRACT(YEAR FROM date)
        ELSE EXTRACT(YEAR FROM date) - 1
    END as fiscal_year,
    CASE
        WHEN EXTRACT(MONTH FROM date) BETWEEN 7 AND 9 THEN 1
        WHEN EXTRACT(MONTH FROM date) BETWEEN 10 AND 12 THEN 2
        WHEN EXTRACT(MONTH FROM date) BETWEEN 1 AND 3 THEN 3
        ELSE 4
    END as fiscal_quarter,
    CASE
        WHEN EXTRACT(MONTH FROM date) >= 7 THEN EXTRACT(MONTH FROM date) - 6
        ELSE EXTRACT(MONTH FROM date) + 6
    END as fiscal_period
FROM generate_series(
    '2020-01-01'::DATE,
    '2030-12-31'::DATE,
    '1 day'::INTERVAL
) date;

-- Update holidays
UPDATE dim_date SET is_holiday = TRUE, holiday_name = 'New Year''s Day'
WHERE month = 1 AND day_of_month = 1;

UPDATE dim_date SET is_holiday = TRUE, holiday_name = 'Christmas'
WHERE month = 12 AND day_of_month = 25;

-- Create indexes
CREATE INDEX idx_date ON dim_date(date);
CREATE INDEX idx_year_month ON dim_date(year, month);
```

**Usage in queries:**
```sql
-- Join fact table with date dimension
SELECT
    d.year,
    d.quarter,
    d.month_name,
    SUM(f.amount) as total_revenue
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
WHERE d.year = 2024
  AND d.is_weekend = FALSE
  AND d.is_holiday = FALSE
GROUP BY d.year, d.quarter, d.month_name
ORDER BY d.year, d.quarter;
```

**Optum Use Case:**
```sql
-- Claims processed by fiscal quarter (fiscal year starts Oct 1)
SELECT
    d.fiscal_year,
    d.fiscal_quarter,
    COUNT(*) as claim_count,
    SUM(c.amount) as total_amount
FROM fact_claims c
JOIN dim_date d ON c.process_date_key = d.date_key
WHERE d.fiscal_year = 2024
GROUP BY d.fiscal_year, d.fiscal_quarter;
```

---

**Q77: Explain deduplication strategies in SQL.**

**Answer:**
Different approaches depending on data characteristics and requirements:

**Method 1: ROW_NUMBER() - Most flexible**
```sql
-- Delete duplicates, keep earliest record
WITH duplicates AS (
    SELECT
        id,
        ROW_NUMBER() OVER (
            PARTITION BY email
            ORDER BY created_at ASC  -- Keep earliest
        ) as rn
    FROM users
)
DELETE FROM users
WHERE id IN (
    SELECT id FROM duplicates WHERE rn > 1
);

-- Or create deduped table
CREATE TABLE users_deduped AS
SELECT *
FROM (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY email
            ORDER BY created_at DESC  -- Keep latest
        ) as rn
    FROM users
) t
WHERE rn = 1;
```

**Method 2: DISTINCT ON (PostgreSQL-specific)**
```sql
-- Keep first occurrence per email (by created_at)
SELECT DISTINCT ON (email)
    *
FROM users
ORDER BY email, created_at ASC;

-- More complex: Keep row with most complete data
SELECT DISTINCT ON (customer_id)
    customer_id,
    name,
    email,
    phone
FROM customer_records
ORDER BY
    customer_id,
    -- Prioritize records with non-null values
    (CASE WHEN phone IS NOT NULL THEN 0 ELSE 1 END),
    (CASE WHEN email IS NOT NULL THEN 0 ELSE 1 END),
    updated_at DESC;
```

**Method 3: GROUP BY + aggregation**
```sql
-- When you need to combine data from duplicates
SELECT
    email,
    MAX(name) as name,  -- Take non-null name
    MIN(created_at) as first_seen,
    MAX(last_login) as last_login,
    SUM(order_count) as total_orders
FROM users
GROUP BY email;
```

**Method 4: Self-join (for deletes)**
```sql
-- Delete duplicates, keep lowest ID
DELETE FROM users u1
USING users u2
WHERE u1.email = u2.email
  AND u1.id > u2.id;
```

**Method 5: EXISTS (for complex logic)**
```sql
-- Delete duplicates based on multiple columns
DELETE FROM orders o1
WHERE EXISTS (
    SELECT 1
    FROM orders o2
    WHERE o2.customer_id = o1.customer_id
      AND o2.product_id = o1.product_id
      AND o2.order_date = o1.order_date
      AND o2.id < o1.id  -- Keep lower ID
);
```

**Optum Claims Deduplication:**
```sql
-- Remove duplicate claims (same claim_id, different submission times)
WITH ranked_claims AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY claim_id
            ORDER BY
                -- Priority: final status > pending
                CASE WHEN status = 'FINAL' THEN 0 ELSE 1 END,
                -- Then latest submission
                submitted_at DESC,
                -- Tie-breaker: highest amount (most complete)
                amount DESC NULLS LAST
        ) as rn
    FROM claims_raw
)
INSERT INTO claims_deduped
SELECT
    claim_id,
    patient_id,
    provider_id,
    amount,
    status,
    submitted_at
FROM ranked_claims
WHERE rn = 1;

-- Result: 10M raw claims → 8M deduplicated (20% were duplicates)
```

---

**Q78: How do you handle NULLs in aggregations?**

**Answer:**
NULLs are ignored in aggregations (except COUNT(*)), which can cause issues:

**Problem:**
```sql
SELECT AVG(rating) FROM products;
-- If 5 products: ratings = [5, 4, NULL, 3, NULL]
-- AVG = (5+4+3)/3 = 4.0 (NULLs excluded)
-- But you might want (5+4+0+3+0)/5 = 2.4
```

**Solution 1: COALESCE to replace NULLs**
```sql
-- Treat NULL as 0
SELECT AVG(COALESCE(rating, 0)) as avg_rating
FROM products;

-- Treat NULL as default value
SELECT AVG(COALESCE(discount_pct, 10)) as avg_discount
FROM products;
```

**Solution 2: NULLIF to create NULLs**
```sql
-- Treat 0 as NULL (avoid divide by zero)
SELECT
    SUM(revenue) / NULLIF(SUM(orders), 0) as avg_order_value
FROM daily_stats;
```

**Solution 3: COUNT with CASE**
```sql
-- Count specific values
SELECT
    COUNT(*) as total_products,
    COUNT(rating) as products_with_rating,  -- Excludes NULLs
    COUNT(CASE WHEN rating >= 4 THEN 1 END) as high_rated,
    ROUND(100.0 * COUNT(rating) / COUNT(*), 2) as pct_rated
FROM products;
```

**Solution 4: FILTER clause (PostgreSQL 9.4+)**
```sql
SELECT
    COUNT(*) as total,
    COUNT(*) FILTER (WHERE rating IS NOT NULL) as has_rating,
    COUNT(*) FILTER (WHERE rating >= 4) as high_rated,
    AVG(rating) FILTER (WHERE rating IS NOT NULL) as avg_rating
FROM products;
```

**Solution 5: Handle in window functions**
```sql
-- Ignore NULLs in window aggregation
SELECT
    date,
    revenue,
    -- Moving average (ignores NULLs)
    AVG(revenue) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg_7day,
    -- Fill NULLs with previous value
    COALESCE(
        revenue,
        LAG(revenue) OVER (ORDER BY date)
    ) as revenue_filled
FROM daily_revenue;
```

**Optum Example:**
```sql
-- Patient satisfaction scores (some patients didn't respond)
SELECT
    provider_id,
    COUNT(*) as total_patients,
    COUNT(satisfaction_score) as responses,
    ROUND(100.0 * COUNT(satisfaction_score) / COUNT(*), 1) as response_rate_pct,
    -- Average score (only from responses)
    ROUND(AVG(satisfaction_score), 2) as avg_score,
    -- Average treating NULL as neutral (3)
    ROUND(AVG(COALESCE(satisfaction_score, 3)), 2) as avg_score_with_neutral,
    -- Breakdown
    COUNT(CASE WHEN satisfaction_score >= 4 THEN 1 END) as satisfied,
    COUNT(CASE WHEN satisfaction_score <= 2 THEN 1 END) as unsatisfied,
    COUNT(CASE WHEN satisfaction_score IS NULL THEN 1 END) as no_response
FROM patient_surveys
GROUP BY provider_id;
```

---

**Q79: Explain database normalization (1NF, 2NF, 3NF, BCNF).**

**Answer:**
Normalization = Organizing data to reduce redundancy and improve integrity.

**1NF (First Normal Form):**
- Atomic values (no arrays/lists in columns)
- Each column has single value
- Unique column names
- Order doesn't matter

```sql
-- ❌ Not 1NF: Multiple values in single column
CREATE TABLE orders_bad (
    order_id INT,
    customer_name VARCHAR(100),
    products VARCHAR(500)  -- "Apple, Banana, Orange"
);

-- ✅ 1NF: Atomic values
CREATE TABLE orders (
    order_id INT,
    customer_name VARCHAR(100)
);

CREATE TABLE order_items (
    order_id INT,
    product_name VARCHAR(100)
);
```

**2NF (Second Normal Form):**
- Must be in 1NF
- No partial dependencies (all non-key columns fully depend on primary key)

```sql
-- ❌ Not 2NF: customer_name depends only on customer_id, not full PK
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    customer_id INT,
    customer_name VARCHAR(100),  -- Partial dependency!
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- ✅ 2NF: Separate customer info
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT
);

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100)
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);
```

**3NF (Third Normal Form):**
- Must be in 2NF
- No transitive dependencies (non-key columns don't depend on other non-key columns)

```sql
-- ❌ Not 3NF: city depends on zip_code (transitive dependency)
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    zip_code VARCHAR(10),
    city VARCHAR(50)  -- city depends on zip_code!
);

-- ✅ 3NF: Separate location info
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    zip_code VARCHAR(10)
);

CREATE TABLE zip_codes (
    zip_code VARCHAR(10) PRIMARY KEY,
    city VARCHAR(50),
    state VARCHAR(2)
);
```

**BCNF (Boyce-Codd Normal Form):**
- Must be in 3NF
- Every determinant is a candidate key

```sql
-- Example: Student enrollments
-- ❌ Not BCNF: professor determines course (but professor isn't a key)
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    professor_id INT,
    PRIMARY KEY (student_id, course_id),
    -- Problem: professor_id → course_id (prof teaches one course)
    -- But professor_id is not a candidate key
);

-- ✅ BCNF: Separate course assignments
CREATE TABLE course_assignments (
    professor_id INT,
    course_id INT,
    PRIMARY KEY (professor_id)  -- Each prof teaches one course
);

CREATE TABLE enrollments (
    student_id INT,
    professor_id INT,
    PRIMARY KEY (student_id, professor_id)
);
```

**When to Denormalize:**
- Analytics/reporting (star schema)
- Read-heavy workloads
- Performance over storage

**Optum Example:**
```sql
-- Normalized (OLTP - transactional system)
CREATE TABLE claims (
    claim_id INT PRIMARY KEY,
    patient_id INT,
    provider_id INT,
    amount DECIMAL(10,2)
);

CREATE TABLE patients (
    patient_id INT PRIMARY KEY,
    member_id VARCHAR(20),
    name VARCHAR(100),
    dob DATE
);

CREATE TABLE providers (
    provider_id INT PRIMARY KEY,
    npi VARCHAR(10),
    name VARCHAR(100),
    specialty VARCHAR(50)
);

-- Denormalized (OLAP - analytics warehouse)
CREATE TABLE fact_claims (
    claim_id INT,
    claim_date DATE,
    amount DECIMAL(10,2),
    -- Patient info (denormalized)
    patient_id INT,
    patient_name VARCHAR(100),
    patient_age INT,
    -- Provider info (denormalized)
    provider_id INT,
    provider_name VARCHAR(100),
    provider_specialty VARCHAR(50),
    provider_network VARCHAR(50)
);
-- Trade-off: More storage, faster queries (no joins!)
```

---

**Q80: What are database indexes? When to use each type?**

**Answer:**
Indexes = Data structures that improve query performance at the cost of storage and write speed.

**B-tree Index (default, most common):**
- Balanced tree structure
- Good for: Equality, range queries, sorting
- Use for: Most columns in WHERE, JOIN, ORDER BY

```sql
-- Single column
CREATE INDEX idx_users_email ON users(email);

-- Composite index (column order matters!)
CREATE INDEX idx_orders_cust_date ON orders(customer_id, order_date);
-- Good for: WHERE customer_id = X AND order_date = Y
-- Also good for: WHERE customer_id = X
-- NOT used for: WHERE order_date = Y (first column not in WHERE)

-- Index with INCLUDE (PostgreSQL 11+)
CREATE INDEX idx_orders_cust_inc ON orders(customer_id)
INCLUDE (order_date, amount);
-- Benefits: Cover query (no table lookup needed)
```

**Hash Index:**
- Hash table
- Good for: Equality only (no range queries)
- Rarely used (B-tree usually better)

```sql
CREATE INDEX idx_users_id_hash ON users USING hash(user_id);
-- Only helps: WHERE user_id = 123
-- Doesn't help: WHERE user_id > 123
```

**GiST (Generalized Search Tree):**
- For geometric, full-text search
- Good for: Ranges, containment

```sql
-- Geometric queries
CREATE INDEX idx_locations_point ON locations USING gist(location);
-- Query: Find nearby locations
SELECT * FROM locations
WHERE location <-> point'(40.7,-74.0)' < 1000;  -- Within 1000 units
```

**GIN (Generalized Inverted Index):**
- For array, JSON, full-text
- Good for: Containment, text search

```sql
-- Array containment
CREATE INDEX idx_products_tags ON products USING gin(tags);
SELECT * FROM products WHERE tags @> ARRAY['electronics', 'sale'];

-- JSON queries
CREATE INDEX idx_users_metadata ON users USING gin(metadata);
SELECT * FROM users WHERE metadata @> '{"status": "active"}';

-- Full-text search
CREATE INDEX idx_articles_search ON articles USING gin(to_tsvector('english', content));
SELECT * FROM articles
WHERE to_tsvector('english', content) @@ to_tsquery('database & performance');
```

**Partial Index:**
- Index subset of rows
- Saves space, faster updates

```sql
-- Only index active orders
CREATE INDEX idx_active_orders ON orders(order_date)
WHERE status = 'active';

-- Only index recent data
CREATE INDEX idx_recent_logs ON logs(created_at)
WHERE created_at > CURRENT_DATE - INTERVAL '30 days';
```

**Expression Index:**
- Index on computed value

```sql
-- Index on lowercase email
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
SELECT * FROM users WHERE LOWER(email) = 'alice@example.com';

-- Index on extracted JSON field
CREATE INDEX idx_users_city ON users((metadata->>'city'));
```

**When NOT to index:**
- Small tables (<1000 rows)
- Columns with low cardinality (few distinct values, like boolean)
- Frequently updated columns
- Wide columns (long text)

**Optum Indexing Strategy:**
```sql
-- Claims table (200M rows)
CREATE TABLE claims (
    claim_id BIGINT PRIMARY KEY,
    patient_id INT,
    provider_id INT,
    claim_date DATE,
    process_date DATE,
    amount DECIMAL(10,2),
    status VARCHAR(20),
    diagnosis_codes TEXT[]
);

-- Indexes
CREATE INDEX idx_claims_patient ON claims(patient_id);  -- Patient lookup
CREATE INDEX idx_claims_provider ON claims(provider_id);  -- Provider reports
CREATE INDEX idx_claims_date ON claims(claim_date);  -- Date filtering
CREATE INDEX idx_claims_status_date ON claims(status, process_date)  -- Active claims
WHERE status IN ('PENDING', 'IN_REVIEW');  -- Partial index (10% of data)

CREATE INDEX idx_claims_amount ON claims(amount)
WHERE amount > 10000;  -- High-value claims

CREATE INDEX idx_claims_diagnosis ON claims USING gin(diagnosis_codes);  -- Array search

-- Covering index for common query
CREATE INDEX idx_claims_provider_cover ON claims(provider_id, claim_date)
INCLUDE (amount, status);
-- Covers: SELECT amount, status FROM claims
--         WHERE provider_id = X AND claim_date BETWEEN Y AND Z

-- Result:
-- Query time: 30 seconds → 0.5 seconds (60x faster)
-- Index size: 15GB total
-- Write performance: -10% (acceptable trade-off)
```

---

**Q81: Explain star schema vs snowflake schema.**

**Answer:**

**Star Schema:**
- Denormalized dimensions
- One fact table + multiple dimension tables
- Dimensions directly connect to fact (star shape)

```sql
-- Star Schema
CREATE TABLE fact_sales (
    sale_id BIGINT PRIMARY KEY,
    date_key INT,        -- FK to dim_date
    product_key INT,     -- FK to dim_product
    store_key INT,       -- FK to dim_store
    customer_key INT,    -- FK to dim_customer
    quantity INT,
    amount DECIMAL(10,2)
);

CREATE TABLE dim_product (
    product_key INT PRIMARY KEY,
    product_id VARCHAR(20),
    product_name VARCHAR(100),
    category VARCHAR(50),        -- Denormalized
    subcategory VARCHAR(50),     -- Denormalized
    brand VARCHAR(50)            -- Denormalized
);

CREATE TABLE dim_store (
    store_key INT PRIMARY KEY,
    store_id VARCHAR(20),
    store_name VARCHAR(100),
    city VARCHAR(50),            -- Denormalized
    state VARCHAR(2),            -- Denormalized
    region VARCHAR(20)           -- Denormalized
);

CREATE TABLE dim_customer (
    customer_key INT PRIMARY KEY,
    customer_id VARCHAR(20),
    name VARCHAR(100),
    city VARCHAR(50),            -- Denormalized
    state VARCHAR(2),            -- Denormalized
    segment VARCHAR(20)          -- Denormalized
);
```

**Snowflake Schema:**
- Normalized dimensions
- Dimensions split into sub-dimensions
- More complex joins

```sql
-- Snowflake Schema
CREATE TABLE fact_sales (
    sale_id BIGINT PRIMARY KEY,
    date_key INT,
    product_key INT,
    store_key INT,
    customer_key INT,
    quantity INT,
    amount DECIMAL(10,2)
);

CREATE TABLE dim_product (
    product_key INT PRIMARY KEY,
    product_id VARCHAR(20),
    product_name VARCHAR(100),
    category_key INT        -- FK to dim_category (normalized)
);

CREATE TABLE dim_category (
    category_key INT PRIMARY KEY,
    category_name VARCHAR(50),
    subcategory VARCHAR(50),
    brand_key INT          -- FK to dim_brand
);

CREATE TABLE dim_brand (
    brand_key INT PRIMARY KEY,
    brand_name VARCHAR(50)
);

CREATE TABLE dim_store (
    store_key INT PRIMARY KEY,
    store_id VARCHAR(20),
    store_name VARCHAR(100),
    location_key INT       -- FK to dim_location
);

CREATE TABLE dim_location (
    location_key INT PRIMARY KEY,
    city VARCHAR(50),
    state VARCHAR(2),
    region VARCHAR(20)
);
```

**Comparison:**

| Feature | Star Schema | Snowflake Schema |
|---------|-------------|------------------|
| **Joins** | Fewer (1-2 joins) | More (3-4 joins) |
| **Query Speed** | Faster | Slower |
| **Storage** | More (denormalized) | Less (normalized) |
| **ETL** | Simpler | More complex |
| **Updates** | Update anomalies possible | No anomalies |
| **Use Case** | Most data warehouses | Very large dimensions |

**Query Example:**

```sql
-- Star Schema (simple)
SELECT
    p.category,
    s.region,
    SUM(f.amount) as total_sales
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_store s ON f.store_key = s.store_key
WHERE f.date_key BETWEEN 20240101 AND 20241231
GROUP BY p.category, s.region;

-- Snowflake Schema (more joins)
SELECT
    c.category_name,
    l.region,
    SUM(f.amount) as total_sales
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_category c ON p.category_key = c.category_key
JOIN dim_store s ON f.store_key = s.store_key
JOIN dim_location l ON s.location_key = l.location_key
WHERE f.date_key BETWEEN 20240101 AND 20241231
GROUP BY c.category_name, l.region;
```

**Optum Choice: Star Schema**
```sql
-- Claims data warehouse (star schema)
CREATE TABLE fact_claims (
    claim_key BIGINT PRIMARY KEY,
    claim_id VARCHAR(20),
    process_date_key INT,
    patient_key INT,
    provider_key INT,
    diagnosis_key INT,
    amount DECIMAL(10,2),
    status VARCHAR(20)
);

CREATE TABLE dim_provider (
    provider_key INT PRIMARY KEY,
    provider_id VARCHAR(20),
    npi VARCHAR(10),
    name VARCHAR(100),
    specialty VARCHAR(50),      -- Denormalized
    subspecialty VARCHAR(50),   -- Denormalized
    network VARCHAR(50),        -- Denormalized
    city VARCHAR(50),           -- Denormalized
    state VARCHAR(2)            -- Denormalized
);

-- Why star over snowflake:
-- 1. Faster queries (analysts run 10,000+ queries/day)
-- 2. Simpler for BI tools (Tableau, Power BI)
-- 3. Storage cheap compared to analyst time
-- 4. Dimensions relatively small (10M providers vs 200M claims)

-- Query performance:
-- Star schema: 2 joins, 0.5 seconds
-- Snowflake would be: 5 joins, 2-3 seconds (not acceptable for interactive dashboards)
```

---

**Q82: Explain database partitioning strategies.**

**Answer:**

Partitioning = Splitting large table into smaller, manageable pieces.

**Benefits:**
- Faster queries (scan only relevant partitions)
- Easier maintenance (backup/restore individual partitions)
- Better performance (parallel query execution)
- Efficient data retention (drop old partitions)

**Range Partitioning:**
- Partition by value range (dates, IDs)
- Most common

```sql
-- PostgreSQL declarative partitioning
CREATE TABLE orders (
    order_id BIGINT,
    customer_id INT,
    order_date DATE,
    amount DECIMAL(10,2)
) PARTITION BY RANGE (order_date);

-- Create partitions
CREATE TABLE orders_2024_01 PARTITION OF orders
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE orders_2024_02 PARTITION OF orders
FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

CREATE TABLE orders_2024_03 PARTITION OF orders
FOR VALUES FROM ('2024-03-01') TO ('2024-04-01');

-- Query automatically uses partition pruning
SELECT * FROM orders
WHERE order_date BETWEEN '2024-02-01' AND '2024-02-28';
-- Only scans orders_2024_02 partition

-- Drop old data
DROP TABLE orders_2023_01;  -- Much faster than DELETE
```

**List Partitioning:**
- Partition by discrete values

```sql
CREATE TABLE sales (
    sale_id BIGINT,
    region VARCHAR(20),
    amount DECIMAL(10,2)
) PARTITION BY LIST (region);

CREATE TABLE sales_north PARTITION OF sales
FOR VALUES IN ('North', 'Northeast', 'Northwest');

CREATE TABLE sales_south PARTITION OF sales
FOR VALUES IN ('South', 'Southeast', 'Southwest');

CREATE TABLE sales_east PARTITION OF sales
FOR VALUES IN ('East');

CREATE TABLE sales_west PARTITION OF sales
FOR VALUES IN ('West');
```

**Hash Partitioning:**
- Distribute data evenly
- Good when no natural partition key

```sql
CREATE TABLE users (
    user_id BIGINT,
    name VARCHAR(100),
    email VARCHAR(100)
) PARTITION BY HASH (user_id);

CREATE TABLE users_0 PARTITION OF users
FOR VALUES WITH (MODULUS 4, REMAINDER 0);

CREATE TABLE users_1 PARTITION OF users
FOR VALUES WITH (MODULUS 4, REMAINDER 1);

CREATE TABLE users_2 PARTITION OF users
FOR VALUES WITH (MODULUS 4, REMAINDER 2);

CREATE TABLE users_3 PARTITION OF users
FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```

**Multi-level Partitioning:**
- Partition by multiple columns

```sql
CREATE TABLE logs (
    log_id BIGINT,
    log_date DATE,
    severity VARCHAR(20),
    message TEXT
) PARTITION BY RANGE (log_date);

CREATE TABLE logs_2024_05 PARTITION OF logs
FOR VALUES FROM ('2024-05-01') TO ('2024-06-01')
PARTITION BY LIST (severity);

CREATE TABLE logs_2024_05_error PARTITION OF logs_2024_05
FOR VALUES IN ('ERROR', 'CRITICAL');

CREATE TABLE logs_2024_05_info PARTITION OF logs_2024_05
FOR VALUES IN ('INFO', 'DEBUG');
```

**When to Partition:**
- ✅ Table > 100GB
- ✅ Queries filter by partition key
- ✅ Need to drop old data regularly
- ✅ Multi-tenant data (partition by tenant_id)
- ❌ Table < 10GB (overhead not worth it)
- ❌ No clear partition key

**Optum Claims Partitioning:**
```sql
-- 2TB claims table, partitioned by month
CREATE TABLE claims (
    claim_id BIGINT,
    patient_id INT,
    provider_id INT,
    claim_date DATE,
    process_date DATE,
    amount DECIMAL(10,2),
    status VARCHAR(20)
) PARTITION BY RANGE (claim_date);

-- Create partitions programmatically
DO $$
DECLARE
    start_date DATE := '2020-01-01';
    end_date DATE := '2025-12-31';
    current_date DATE;
    partition_name TEXT;
BEGIN
    current_date := start_date;
    WHILE current_date < end_date LOOP
        partition_name := 'claims_' || TO_CHAR(current_date, 'YYYY_MM');
        EXECUTE format(
            'CREATE TABLE %I PARTITION OF claims
             FOR VALUES FROM (%L) TO (%L)',
            partition_name,
            current_date,
            current_date + INTERVAL '1 month'
        );
        current_date := current_date + INTERVAL '1 month';
    END LOOP;
END $$;

-- Indexes on each partition
CREATE INDEX ON claims_2024_05(provider_id);
CREATE INDEX ON claims_2024_05(patient_id);

-- Query performance
-- Before partitioning: Full table scan (2TB, 5 minutes)
SELECT COUNT(*), SUM(amount)
FROM claims
WHERE claim_date BETWEEN '2024-05-01' AND '2024-05-31';

-- After partitioning: Partition scan (30GB, 3 seconds)
-- PostgreSQL automatically prunes to claims_2024_05 partition

-- Data retention
-- Drop partitions older than 7 years (compliance requirement)
DROP TABLE claims_2017_01;  -- 0.1 seconds vs hours with DELETE
DROP TABLE claims_2017_02;

-- Result:
-- Queries: 5 min → 3 sec (100x faster)
-- Data retention: Hours → seconds
-- Backup/restore: Full table → individual partitions (flexible)
```

---

**Q83: Explain window frame clauses (ROWS vs RANGE).**

**Answer:**

Window frames define which rows are included in window function calculation.

**Syntax:**
```sql
function() OVER (
    PARTITION BY column
    ORDER BY column
    frame_clause
)
```

**Frame Types:**

**ROWS:**
- Physical rows (count-based)
- Based on row position

```sql
-- Moving average of last 7 rows
SELECT
    date,
    revenue,
    AVG(revenue) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg_7_rows
FROM daily_revenue;

-- Example data:
-- Row 1: avg(row1)
-- Row 2: avg(row1, row2)
-- Row 7: avg(row1..row7)
-- Row 8: avg(row2..row8)  -- Sliding window
```

**RANGE:**
- Logical range (value-based)
- Based on column value

```sql
-- All rows within 7 days
SELECT
    date,
    revenue,
    SUM(revenue) OVER (
        ORDER BY date
        RANGE BETWEEN INTERVAL '6 days' PRECEDING AND CURRENT ROW
    ) as sum_last_7_days
FROM daily_revenue;

-- Example:
-- 2024-05-01: sum(all rows from 2024-04-25 to 2024-05-01)
-- 2024-05-02: sum(all rows from 2024-04-26 to 2024-05-02)
```

**Frame Bounds:**

```sql
-- UNBOUNDED PRECEDING: from start of partition
AVG(amount) OVER (ORDER BY date ROWS UNBOUNDED PRECEDING)

-- UNBOUNDED FOLLOWING: to end of partition
AVG(amount) OVER (ORDER BY date ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING)

-- N PRECEDING: N rows/range before current
AVG(amount) OVER (ORDER BY date ROWS 3 PRECEDING)

-- N FOLLOWING: N rows/range after current
AVG(amount) OVER (ORDER BY date ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)

-- CURRENT ROW: only current row
AVG(amount) OVER (ORDER BY date ROWS CURRENT ROW)  -- Just the current value
```

**Default Frame:**
- With ORDER BY: RANGE UNBOUNDED PRECEDING (cumulative)
- Without ORDER BY: All rows in partition

```sql
-- These are equivalent:
SUM(amount) OVER (ORDER BY date)
SUM(amount) OVER (ORDER BY date RANGE UNBOUNDED PRECEDING)

-- Example:
-- date       amount  running_sum
-- 2024-05-01   100      100
-- 2024-05-02   150      250
-- 2024-05-03   200      450  (cumulative)
```

**Practical Examples:**

**1. Moving Average (ROWS):**
```sql
SELECT
    date,
    revenue,
    -- 7-day moving average (exactly 7 rows)
    ROUND(AVG(revenue) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ), 2) as ma_7_day,
    -- Centered moving average (3 before, current, 3 after)
    ROUND(AVG(revenue) OVER (
        ORDER BY date
        ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING
    ), 2) as ma_centered
FROM daily_revenue;
```

**2. Time-based Window (RANGE):**
```sql
SELECT
    timestamp,
    clicks,
    -- All clicks within last hour
    SUM(clicks) OVER (
        ORDER BY timestamp
        RANGE BETWEEN INTERVAL '1 hour' PRECEDING AND CURRENT ROW
    ) as clicks_last_hour,
    -- All clicks in same day
    SUM(clicks) OVER (
        ORDER BY timestamp::DATE
        RANGE CURRENT ROW
    ) as clicks_same_day
FROM clickstream;
```

**3. Cumulative vs Sliding:**
```sql
SELECT
    month,
    revenue,
    -- Cumulative (running total)
    SUM(revenue) OVER (
        ORDER BY month
        ROWS UNBOUNDED PRECEDING
    ) as cumulative_revenue,
    -- Sliding (last 3 months)
    SUM(revenue) OVER (
        ORDER BY month
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) as trailing_3mo_revenue
FROM monthly_revenue;
```

**ROWS vs RANGE Difference:**

```sql
-- Sample data with ties:
-- date         amount
-- 2024-05-01   100
-- 2024-05-01   150  (same date!)
-- 2024-05-02   200

-- ROWS (count-based)
SELECT
    date,
    amount,
    SUM(amount) OVER (
        ORDER BY date
        ROWS BETWEEN 1 PRECEDING AND CURRENT ROW
    ) as sum_rows
FROM sales;
-- Results:
-- 2024-05-01, 100,  100  (just this row)
-- 2024-05-01, 150,  250  (previous row + this)
-- 2024-05-02, 200,  350  (previous row + this)

-- RANGE (value-based)
SELECT
    date,
    amount,
    SUM(amount) OVER (
        ORDER BY date
        RANGE BETWEEN 1 PRECEDING AND CURRENT ROW
    ) as sum_range
FROM sales;
-- Results:
-- 2024-05-01, 100,  250  (all rows with same date)
-- 2024-05-01, 150,  250  (all rows with same date)
-- 2024-05-02, 200,  450  (all from 2024-05-01 + this row)
```

**Optum Use Case:**
```sql
-- Patient medication adherence (7-day window)
SELECT
    patient_id,
    medication_date,
    took_medication,
    -- Count pills taken in last 7 days (ROWS)
    SUM(took_medication::INT) OVER (
        PARTITION BY patient_id
        ORDER BY medication_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as pills_taken_last_7_rows,
    -- Count pills taken in last 7 calendar days (RANGE)
    SUM(took_medication::INT) OVER (
        PARTITION BY patient_id
        ORDER BY medication_date
        RANGE BETWEEN INTERVAL '6 days' PRECEDING AND CURRENT ROW
    ) as pills_taken_last_7_days,
    -- Adherence %
    ROUND(100.0 * SUM(took_medication::INT) OVER (
        PARTITION BY patient_id
        ORDER BY medication_date
        RANGE BETWEEN INTERVAL '6 days' PRECEDING AND CURRENT ROW
    ) / 7, 1) as adherence_pct
FROM medication_log
WHERE medication_date >= CURRENT_DATE - 30;

-- RANGE is better here: If patient missed logging for 2 days,
-- ROWS would include 9 days of data, RANGE correctly uses 7 calendar days
```

---

**Q84: What are database constraints? Explain each type.**

**Answer:**

Constraints = Rules enforced by database to ensure data integrity.

**PRIMARY KEY:**
- Uniquely identifies each row
- Cannot be NULL
- Only one per table

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    email VARCHAR(100)
);

-- Composite primary key
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);
```

**FOREIGN KEY:**
- Ensures referential integrity
- Value must exist in referenced table
- Supports CASCADE operations

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE CASCADE      -- Delete orders when customer deleted
        ON UPDATE CASCADE      -- Update order.customer_id when customer.id changes
);

-- Other options:
-- ON DELETE SET NULL       -- Set to NULL when parent deleted
-- ON DELETE SET DEFAULT    -- Set to default value
-- ON DELETE RESTRICT       -- Prevent delete (default)
-- ON DELETE NO ACTION      -- Same as RESTRICT
```

**UNIQUE:**
- Ensures column values are unique
- Allows NULL (unlike PRIMARY KEY)
- Can have multiple UNIQUE constraints

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE,  -- Each email must be unique
    username VARCHAR(50) UNIQUE,
    phone VARCHAR(15)
);

-- Composite unique
CREATE TABLE bookings (
    booking_id INT PRIMARY KEY,
    room_id INT,
    booking_date DATE,
    UNIQUE (room_id, booking_date)  -- Room can't be booked twice same day
);
```

**CHECK:**
- Custom validation rules

```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2) CHECK (price >= 0),  -- Price can't be negative
    discount_pct INT CHECK (discount_pct BETWEEN 0 AND 100),
    stock INT CHECK (stock >= 0),
    category VARCHAR(50) CHECK (category IN ('Electronics', 'Clothing', 'Food'))
);

-- Table-level CHECK (multiple columns)
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    order_date DATE,
    ship_date DATE,
    CHECK (ship_date >= order_date)  -- Can't ship before order
);
```

**NOT NULL:**
- Column must have a value

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,  -- Required
    last_name VARCHAR(50) NOT NULL,   -- Required
    middle_name VARCHAR(50),           -- Optional
    email VARCHAR(100) NOT NULL UNIQUE,
    salary DECIMAL(10,2) NOT NULL CHECK (salary > 0)
);
```

**DEFAULT:**
- Provides default value when not specified

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'PENDING',
    priority INT DEFAULT 1,
    is_paid BOOLEAN DEFAULT FALSE
);

-- Usage
INSERT INTO orders (order_id, customer_id)
VALUES (1, 123);
-- Automatically sets: order_date=now, status='PENDING', priority=1, is_paid=FALSE
```

**EXCLUDE (PostgreSQL):**
- Prevent overlapping ranges

```sql
-- Prevent overlapping room bookings
CREATE TABLE bookings (
    room_id INT,
    booking_range DATERANGE,
    EXCLUDE USING gist (
        room_id WITH =,
        booking_range WITH &&  -- Overlaps operator
    )
);

-- This prevents:
INSERT INTO bookings VALUES (1, '[2024-05-01, 2024-05-05)');
INSERT INTO bookings VALUES (1, '[2024-05-03, 2024-05-07)');  -- ERROR! Overlaps
```

**Adding/Dropping Constraints:**

```sql
-- Add constraints to existing table
ALTER TABLE users
ADD CONSTRAINT unique_email UNIQUE (email);

ALTER TABLE orders
ADD CONSTRAINT fk_customer
FOREIGN KEY (customer_id) REFERENCES customers(customer_id);

ALTER TABLE products
ADD CONSTRAINT check_price CHECK (price >= 0);

-- Drop constraints
ALTER TABLE users DROP CONSTRAINT unique_email;
ALTER TABLE orders DROP CONSTRAINT fk_customer;

-- Temporarily disable constraints (PostgreSQL)
ALTER TABLE orders DISABLE TRIGGER ALL;
-- Bulk insert...
ALTER TABLE orders ENABLE TRIGGER ALL;
```

**Optum Healthcare Example:**
```sql
CREATE TABLE claims (
    claim_id VARCHAR(20) PRIMARY KEY,
    patient_id VARCHAR(20) NOT NULL,
    provider_id VARCHAR(10) NOT NULL,
    claim_date DATE NOT NULL,
    service_date DATE NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    status VARCHAR(20) DEFAULT 'PENDING',
    processed_date TIMESTAMP,

    -- Constraints
    CHECK (amount > 0),  -- Claims must have positive amount
    CHECK (service_date <= claim_date),  -- Service before claim
    CHECK (status IN ('PENDING', 'APPROVED', 'DENIED', 'IN_REVIEW')),
    CHECK (
        (status = 'PENDING' AND processed_date IS NULL) OR
        (status != 'PENDING' AND processed_date IS NOT NULL)
    ),  -- Processed claims must have processed_date

    -- Foreign keys
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id)
        ON DELETE RESTRICT,  -- Can't delete patient with claims
    FOREIGN KEY (provider_id) REFERENCES providers(provider_id)
        ON DELETE RESTRICT
);

-- Unique constraint: No duplicate claims for same service
CREATE UNIQUE INDEX idx_unique_claim
ON claims(patient_id, provider_id, service_date, amount)
WHERE status != 'DENIED';  -- Allow resubmitting denied claims

-- Example violations:
INSERT INTO claims VALUES ('C001', 'P123', 'D456', '2024-05-01', '2024-06-01', 100, 'PENDING', NULL);
-- ERROR: CHECK constraint failed (service_date > claim_date)

INSERT INTO claims VALUES ('C002', 'P999', 'D456', '2024-05-01', '2024-04-01', 100, 'PENDING', NULL);
-- ERROR: FOREIGN KEY constraint failed (patient P999 doesn't exist)

INSERT INTO claims VALUES ('C003', 'P123', 'D456', '2024-05-01', '2024-04-01', -50, 'PENDING', NULL);
-- ERROR: CHECK constraint failed (amount <= 0)
```

---

**Q85: How do you optimize database schema for analytics workloads?**

**Answer:**

Analytics workloads = Read-heavy, aggregate queries, historical data.

**Optimization Strategies:**

**1. Use Columnar Storage**
```sql
-- PostgreSQL with Citus columnar extension
CREATE TABLE fact_sales (
    sale_id BIGINT,
    date_key INT,
    product_key INT,
    amount DECIMAL(10,2)
) USING columnar;

-- Benefits:
-- - Only read needed columns (SELECT amount vs all columns)
-- - Better compression (similar values grouped)
-- - 10-100x faster for aggregate queries
```

**2. Denormalize (Star Schema)**
```sql
-- ❌ Normalized (OLTP): Many joins
SELECT p.name, c.category, SUM(s.amount)
FROM sales s
JOIN products p ON s.product_id = p.id
JOIN categories c ON p.category_id = c.id
GROUP BY p.name, c.category;

-- ✅ Denormalized (OLAP): No joins
CREATE TABLE fact_sales_denorm (
    sale_id BIGINT,
    product_name VARCHAR(100),  -- Denormalized
    category VARCHAR(50),        -- Denormalized
    amount DECIMAL(10,2)
);

SELECT product_name, category, SUM(amount)
FROM fact_sales_denorm
GROUP BY product_name, category;
```

**3. Pre-aggregate (Materialized Views)**
```sql
-- Expensive query run hourly:
SELECT
    product_id,
    DATE_TRUNC('day', sale_date) as day,
    SUM(amount) as daily_revenue,
    COUNT(*) as daily_sales
FROM sales
GROUP BY product_id, DATE_TRUNC('day', sale_date);

-- Pre-compute and store:
CREATE MATERIALIZED VIEW mv_daily_product_sales AS
SELECT
    product_id,
    DATE_TRUNC('day', sale_date) as day,
    SUM(amount) as daily_revenue,
    COUNT(*) as daily_sales
FROM sales
GROUP BY product_id, DATE_TRUNC('day', sale_date);

CREATE INDEX ON mv_daily_product_sales(product_id, day);

-- Refresh nightly
REFRESH MATERIALIZED VIEW mv_daily_product_sales;

-- Queries run against MV (instant results)
SELECT * FROM mv_daily_product_sales
WHERE product_id = 123;
```

**4. Partition by Time**
```sql
-- Partition large fact tables by date
CREATE TABLE fact_claims (
    claim_id BIGINT,
    claim_date DATE,
    amount DECIMAL(10,2)
) PARTITION BY RANGE (claim_date);

-- Queries scan only relevant partitions
SELECT SUM(amount)
FROM fact_claims
WHERE claim_date BETWEEN '2024-05-01' AND '2024-05-31';
-- Only scans May 2024 partition
```

**5. Bitmap Indexes (for low-cardinality columns)**
```sql
-- Good for columns with few distinct values
CREATE INDEX idx_claims_status ON claims(status);  -- 5 values: PENDING, APPROVED, etc.
CREATE INDEX idx_claims_region ON claims(region);  -- 4 values: North, South, East, West

-- Bitmap indexes excel at combining multiple conditions:
SELECT COUNT(*)
FROM claims
WHERE status = 'APPROVED'
  AND region = 'North';
-- Database combines bitmaps efficiently
```

**6. Covering Indexes**
```sql
-- Include all columns needed by query
CREATE INDEX idx_sales_product_cover ON sales(product_id, sale_date)
INCLUDE (amount, quantity);

-- Query never touches main table (index-only scan)
SELECT sale_date, amount, quantity
FROM sales
WHERE product_id = 123;
```

**7. Compression**
```sql
-- PostgreSQL: Enable compression
ALTER TABLE fact_sales SET (
    toast_compression = lz4  -- or zstd, pglz
);

-- Columnstore: Automatic compression (10x typical)
```

**8. Approximate Aggregates (for huge tables)**
```sql
-- Exact count (slow on billion-row table):
SELECT COUNT(*) FROM huge_table;  -- 30 seconds

-- Approximate count (fast):
SELECT reltuples::BIGINT as approx_count
FROM pg_class
WHERE relname = 'huge_table';  -- 0.001 seconds

-- HyperLogLog for DISTINCT (PostgreSQL extension)
SELECT hll_cardinality(hll_add_agg(hll_hash_text(user_id)))
FROM events;  -- Approximate COUNT(DISTINCT) with 2% error
```

**9. Remove Constraints (Analytics DB)**
```sql
-- OLTP: Full constraints
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT REFERENCES customers(id),
    amount DECIMAL(10,2) CHECK (amount > 0)
);

-- OLAP: Minimal constraints (faster loads, no FK overhead)
CREATE TABLE fact_orders (
    order_id BIGINT,  -- No PK!
    customer_id INT,  -- No FK!
    amount DECIMAL(10,2)  -- No CHECK!
);
-- Trust ETL to enforce integrity
```

**10. Cluster Tables**
```sql
-- Physically reorder rows for faster range scans
CLUSTER fact_sales USING idx_sales_date;
-- Now rows with similar dates stored together (better I/O)

-- Maintenance: Re-cluster periodically
CLUSTER fact_sales;
```

**Optum Analytics Warehouse:**
```sql
-- Before optimization:
-- - 2TB claims table
-- - Normalized schema (8 joins for basic report)
-- - Query time: 5 minutes

-- After optimization:
CREATE TABLE fact_claims_analytics (
    claim_key BIGINT,
    claim_date DATE,
    process_date DATE,
    -- Denormalized patient info
    patient_id VARCHAR(20),
    patient_age INT,
    patient_gender CHAR(1),
    -- Denormalized provider info
    provider_id VARCHAR(10),
    provider_name VARCHAR(100),
    provider_specialty VARCHAR(50),
    provider_network VARCHAR(50),
    -- Denormalized diagnosis
    diagnosis_code VARCHAR(10),
    diagnosis_description VARCHAR(200),
    -- Metrics
    amount DECIMAL(10,2),
    status VARCHAR(20)
) PARTITION BY RANGE (claim_date);  -- Partitioned by month

-- Columnar storage
ALTER TABLE fact_claims_analytics SET (columnar.compression = zstd);

-- Materialized views for common aggregates
CREATE MATERIALIZED VIEW mv_monthly_provider_summary AS
SELECT
    DATE_TRUNC('month', claim_date) as month,
    provider_id,
    provider_name,
    provider_specialty,
    COUNT(*) as claim_count,
    SUM(amount) as total_amount,
    AVG(amount) as avg_amount
FROM fact_claims_analytics
GROUP BY month, provider_id, provider_name, provider_specialty;

CREATE INDEX ON mv_monthly_provider_summary(provider_id, month);

-- Refresh nightly (incremental)
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_monthly_provider_summary;

-- Results:
-- Storage: 2TB → 200GB (90% compression with columnar)
-- Query time: 5 min → 2 sec (150x faster)
-- Joins: 8 → 0 (denormalized)
-- BI tool responsiveness: Batch → interactive
-- Cost: $10K/month compute → $2K/month (better performance, lower cost)
```

---

---

### Level 3: Advanced / Data Engineering (15 Questions)

**Q86: Optimize slow query - how to approach?**

**Answer:**
Systematic approach:

**1. Analyze execution plan**
```sql
EXPLAIN ANALYZE
SELECT ...
-- Look for: Sequential scans, high cost operations, large row counts
```

**2. Add appropriate indexes**
```sql
-- Index on filter columns
CREATE INDEX idx_orders_customer ON orders(customer_id);

-- Index on join columns
CREATE INDEX idx_order_items_order ON order_items(order_id);

-- Composite index for multiple columns
CREATE INDEX idx_orders_cust_date ON orders(customer_id, order_date);

-- Partial index for common filter
CREATE INDEX idx_active_orders ON orders(status)
WHERE status = 'active';
```

**3. Rewrite query**
```sql
-- Before: Subquery in SELECT (runs for each row)
SELECT
    c.name,
    (SELECT COUNT(*) FROM orders WHERE customer_id = c.id) as order_count
FROM customers c;

-- After: JOIN (runs once)
SELECT
    c.name,
    COUNT(o.id) as order_count
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name;
```

**4. Partition large tables**
```sql
-- Partition by date range
CREATE TABLE orders_2024_01 PARTITION OF orders
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');
```

**5. Use materialized views for complex queries**
```sql
CREATE MATERIALIZED VIEW customer_stats AS
SELECT
    customer_id,
    COUNT(*) as order_count,
    SUM(total) as lifetime_value
FROM orders
GROUP BY customer_id;

-- Refresh periodically
REFRESH MATERIALIZED VIEW customer_stats;
```

---

**Q87: How do you handle slowly changing dimensions (SCD)?**

**Answer:**
Different strategies depending on requirements:

**Type 1: Overwrite (no history)**
```sql
-- Just UPDATE
UPDATE customers
SET city = 'New York'
WHERE customer_id = 123;
-- Old value lost forever
```

**Type 2: Add new row (full history)**
```sql
-- Dimension table with version tracking
CREATE TABLE dim_customer (
    surrogate_key SERIAL PRIMARY KEY,
    customer_id INT,
    name VARCHAR(100),
    city VARCHAR(50),
    start_date DATE,
    end_date DATE,
    is_current BOOLEAN
);

-- When customer moves to new city:
-- 1. Close current record
UPDATE dim_customer
SET end_date = CURRENT_DATE, is_current = FALSE
WHERE customer_id = 123 AND is_current = TRUE;

-- 2. Insert new record
INSERT INTO dim_customer (customer_id, name, city, start_date, is_current)
VALUES (123, 'Alice', 'New York', CURRENT_DATE, TRUE);

-- Query point-in-time
SELECT * FROM dim_customer
WHERE customer_id = 123
  AND '2023-06-01' BETWEEN start_date AND COALESCE(end_date, '9999-12-31');
```

**Type 3: Add column (partial history)**
```sql
-- Track previous value in additional column
ALTER TABLE customers
ADD COLUMN previous_city VARCHAR(50);

UPDATE customers
SET previous_city = city, city = 'New York'
WHERE customer_id = 123;
```

---

**Q88: Write a query for sessionization (group events into sessions)**

**Problem:** Group user events into sessions (session = events within 30 minutes of each other)

**Answer:**
```sql
WITH session_starts AS (
    -- Identify session start: first event OR >30 min since last event
    SELECT
        user_id,
        event_time,
        CASE
            WHEN event_time - LAG(event_time) OVER (
                PARTITION BY user_id ORDER BY event_time
            ) > INTERVAL '30 minutes' OR LAG(event_time) OVER (
                PARTITION BY user_id ORDER BY event_time
            ) IS NULL
            THEN 1
            ELSE 0
        END as is_session_start
    FROM events
),
session_ids AS (
    -- Assign session ID
    SELECT
        user_id,
        event_time,
        SUM(is_session_start) OVER (
            PARTITION BY user_id
            ORDER BY event_time
        ) as session_id
    FROM session_starts
)
SELECT
    user_id,
    session_id,
    MIN(event_time) as session_start,
    MAX(event_time) as session_end,
    COUNT(*) as event_count
FROM session_ids
GROUP BY user_id, session_id;
```

---

**Q89: Calculate retention cohorts**

**Answer:**
```sql
-- Monthly cohort retention
WITH user_cohorts AS (
    -- Assign user to cohort (month of first order)
    SELECT
        user_id,
        DATE_TRUNC('month', MIN(order_date)) as cohort_month
    FROM orders
    GROUP BY user_id
),
cohort_activity AS (
    -- Track activity months after cohort
    SELECT
        uc.cohort_month,
        DATE_TRUNC('month', o.order_date) as activity_month,
        EXTRACT(MONTH FROM AGE(DATE_TRUNC('month', o.order_date), uc.cohort_month)) as months_since_cohort,
        COUNT(DISTINCT o.user_id) as active_users
    FROM user_cohorts uc
    JOIN orders o ON uc.user_id = o.user_id
    GROUP BY uc.cohort_month, activity_month
),
cohort_sizes AS (
    -- Cohort sizes
    SELECT
        cohort_month,
        COUNT(DISTINCT user_id) as cohort_size
    FROM user_cohorts
    GROUP BY cohort_month
)
SELECT
    ca.cohort_month,
    ca.months_since_cohort,
    ca.active_users,
    cs.cohort_size,
    ROUND(100.0 * ca.active_users / cs.cohort_size, 2) as retention_pct
FROM cohort_activity ca
JOIN cohort_sizes cs ON ca.cohort_month = cs.cohort_month
ORDER BY ca.cohort_month, ca.months_since_cohort;
```

---

**Q90: How do you perform funnel analysis in SQL?**

**Answer:**

Funnel analysis = Track user progression through sequential steps (view → cart → purchase).

**Method 1: User-level aggregation**
```sql
WITH funnel AS (
    SELECT
        user_id,
        MAX(CASE WHEN event = 'page_view' THEN 1 ELSE 0 END) as viewed,
        MAX(CASE WHEN event = 'add_to_cart' THEN 1 ELSE 0 END) as added,
        MAX(CASE WHEN event = 'purchase' THEN 1 ELSE 0 END) as purchased
    FROM events
    WHERE event_date BETWEEN '2024-05-01' AND '2024-05-31'
    GROUP BY user_id
)
SELECT
    SUM(viewed) as step1_views,
    SUM(added) as step2_cart,
    SUM(purchased) as step3_purchase,
    ROUND(100.0 * SUM(added) / NULLIF(SUM(viewed), 0), 2) as view_to_cart_pct,
    ROUND(100.0 * SUM(purchased) / NULLIF(SUM(added), 0), 2) as cart_to_purchase_pct,
    ROUND(100.0 * SUM(purchased) / NULLIF(SUM(viewed), 0), 2) as overall_conversion_pct
FROM funnel;
```

**Method 2: Sequential ordering**
```sql
-- Ensure steps happen in order
WITH ordered_events AS (
    SELECT
        user_id,
        event,
        event_time,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY event_time) as event_order
    FROM events
    WHERE event_date BETWEEN '2024-05-01' AND '2024-05-31'
),
funnel_steps AS (
    SELECT
        user_id,
        MIN(CASE WHEN event = 'page_view' THEN event_order END) as view_order,
        MIN(CASE WHEN event = 'add_to_cart' THEN event_order END) as cart_order,
        MIN(CASE WHEN event = 'purchase' THEN event_order END) as purchase_order
    FROM ordered_events
    GROUP BY user_id
)
SELECT
    COUNT(*) as total_users,
    COUNT(view_order) as viewed,
    COUNT(CASE WHEN cart_order > view_order THEN 1 END) as added_after_view,
    COUNT(CASE WHEN purchase_order > cart_order THEN 1 END) as purchased_after_cart,
    ROUND(100.0 * COUNT(CASE WHEN cart_order > view_order THEN 1 END) / COUNT(view_order), 2) as view_to_cart_pct,
    ROUND(100.0 * COUNT(CASE WHEN purchase_order > cart_order THEN 1 END) / COUNT(CASE WHEN cart_order > view_order THEN 1 END), 2) as cart_to_purchase_pct
FROM funnel_steps;
```

**Method 3: Time-windowed funnel**
```sql
-- Complete funnel within 24 hours
WITH first_view AS (
    SELECT
        user_id,
        MIN(event_time) as first_view_time
    FROM events
    WHERE event = 'page_view'
    GROUP BY user_id
),
funnel AS (
    SELECT
        fv.user_id,
        fv.first_view_time,
        MAX(CASE WHEN e.event = 'page_view' THEN 1 ELSE 0 END) as viewed,
        MAX(CASE WHEN e.event = 'add_to_cart'
                  AND e.event_time BETWEEN fv.first_view_time
                  AND fv.first_view_time + INTERVAL '24 hours' THEN 1 ELSE 0 END) as added,
        MAX(CASE WHEN e.event = 'purchase'
                  AND e.event_time BETWEEN fv.first_view_time
                  AND fv.first_view_time + INTERVAL '24 hours' THEN 1 ELSE 0 END) as purchased
    FROM first_view fv
    LEFT JOIN events e ON fv.user_id = e.user_id
    GROUP BY fv.user_id, fv.first_view_time
)
SELECT
    COUNT(*) as total_users,
    SUM(viewed) as step1_views,
    SUM(added) as step2_cart_within_24h,
    SUM(purchased) as step3_purchase_within_24h,
    ROUND(100.0 * SUM(added) / SUM(viewed), 2) as conversion_24h_pct
FROM funnel;
```

**Optum Patient Journey Funnel:**
```sql
-- Track patient journey: Appointment Scheduled → Checked In → Seen → Follow-up Scheduled
WITH patient_funnel AS (
    SELECT
        patient_id,
        MAX(CASE WHEN event_type = 'appointment_scheduled' THEN 1 ELSE 0 END) as scheduled,
        MAX(CASE WHEN event_type = 'checked_in' THEN 1 ELSE 0 END) as checked_in,
        MAX(CASE WHEN event_type = 'visit_completed' THEN 1 ELSE 0 END) as completed,
        MAX(CASE WHEN event_type = 'follow_up_scheduled' THEN 1 ELSE 0 END) as follow_up
    FROM patient_events
    WHERE event_date BETWEEN '2024-05-01' AND '2024-05-31'
    GROUP BY patient_id
)
SELECT
    SUM(scheduled) as step1_scheduled,
    SUM(checked_in) as step2_checked_in,
    SUM(completed) as step3_completed,
    SUM(follow_up) as step4_follow_up,
    -- Conversion rates
    ROUND(100.0 * SUM(checked_in) / SUM(scheduled), 2) as show_up_rate,
    ROUND(100.0 * SUM(completed) / SUM(checked_in), 2) as completion_rate,
    ROUND(100.0 * SUM(follow_up) / SUM(completed), 2) as follow_up_rate,
    ROUND(100.0 * SUM(follow_up) / SUM(scheduled), 2) as overall_retention_rate
FROM patient_funnel;

-- Result insights:
-- Show-up rate: 85% (15% no-shows)
-- Completion rate: 98%
-- Follow-up rate: 42% (opportunity to improve!)
```

---

**Q91: How do you write time-series analysis queries?**

**Answer:**

**Moving Averages:**
```sql
SELECT
    date,
    daily_revenue,
    -- Simple moving average (last 7 days)
    ROUND(AVG(daily_revenue) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ), 2) as sma_7_day,
    -- Weighted moving average (recent days weighted more)
    ROUND((
        1 * LAG(daily_revenue, 6) OVER (ORDER BY date) +
        2 * LAG(daily_revenue, 5) OVER (ORDER BY date) +
        3 * LAG(daily_revenue, 4) OVER (ORDER BY date) +
        4 * LAG(daily_revenue, 3) OVER (ORDER BY date) +
        5 * LAG(daily_revenue, 2) OVER (ORDER BY date) +
        6 * LAG(daily_revenue, 1) OVER (ORDER BY date) +
        7 * daily_revenue
    ) / 28.0, 2) as wma_7_day
FROM daily_sales;
```

**Exponential Moving Average:**
```sql
-- Approximate EMA using recursive CTE
WITH RECURSIVE ema AS (
    -- Base case: first row
    SELECT
        date,
        daily_revenue,
        daily_revenue as ema_value,
        1 as rn
    FROM daily_sales
    ORDER BY date
    LIMIT 1

    UNION ALL

    -- Recursive case: EMA = α * current + (1-α) * previous_ema
    SELECT
        ds.date,
        ds.daily_revenue,
        0.2 * ds.daily_revenue + 0.8 * e.ema_value as ema_value,  -- α = 0.2
        e.rn + 1
    FROM daily_sales ds
    JOIN ema e ON ds.date > e.date
    WHERE ds.date = (
        SELECT MIN(date) FROM daily_sales WHERE date > e.date
    )
)
SELECT * FROM ema;
```

**Trend Detection:**
```sql
-- Linear regression slope
WITH stats AS (
    SELECT
        date,
        revenue,
        ROW_NUMBER() OVER (ORDER BY date) as x,
        AVG(revenue) OVER () as avg_revenue,
        AVG(ROW_NUMBER() OVER (ORDER BY date)) OVER () as avg_x
    FROM monthly_revenue
)
SELECT
    -- Slope: Σ((x - x̄)(y - ȳ)) / Σ((x - x̄)²)
    SUM((x - avg_x) * (revenue - avg_revenue)) /
    NULLIF(SUM((x - avg_x) * (x - avg_x)), 0) as slope,
    -- Positive slope = upward trend, negative = downward
    CASE
        WHEN SUM((x - avg_x) * (revenue - avg_revenue)) > 0 THEN 'Upward Trend'
        WHEN SUM((x - avg_x) * (revenue - avg_revenue)) < 0 THEN 'Downward Trend'
        ELSE 'No Trend'
    END as trend_direction
FROM stats;
```

**Seasonality Detection:**
```sql
-- Month-over-month comparison
SELECT
    EXTRACT(MONTH FROM date) as month,
    EXTRACT(YEAR FROM date) as year,
    SUM(revenue) as monthly_revenue,
    -- Year-over-year comparison
    LAG(SUM(revenue), 12) OVER (ORDER BY date) as same_month_last_year,
    ROUND(100.0 * (SUM(revenue) - LAG(SUM(revenue), 12) OVER (ORDER BY date)) /
          NULLIF(LAG(SUM(revenue), 12) OVER (ORDER BY date), 0), 2) as yoy_growth_pct
FROM daily_revenue
GROUP BY EXTRACT(MONTH FROM date), EXTRACT(YEAR FROM date), DATE_TRUNC('month', date)
ORDER BY DATE_TRUNC('month', date);
```

**Anomaly Detection:**
```sql
-- Detect outliers using standard deviation
WITH stats AS (
    SELECT
        date,
        revenue,
        AVG(revenue) OVER (
            ORDER BY date
            ROWS BETWEEN 30 PRECEDING AND CURRENT ROW
        ) as rolling_avg,
        STDDEV(revenue) OVER (
            ORDER BY date
            ROWS BETWEEN 30 PRECEDING AND CURRENT ROW
        ) as rolling_stddev
    FROM daily_revenue
)
SELECT
    date,
    revenue,
    rolling_avg,
    rolling_stddev,
    CASE
        WHEN revenue > rolling_avg + 3 * rolling_stddev THEN 'High Outlier'
        WHEN revenue < rolling_avg - 3 * rolling_stddev THEN 'Low Outlier'
        ELSE 'Normal'
    END as anomaly_status
FROM stats
WHERE revenue > rolling_avg + 3 * rolling_stddev
   OR revenue < rolling_avg - 3 * rolling_stddev;
```

**Optum Claims Time-Series:**
```sql
-- Monthly claims analysis with trend and seasonality
SELECT
    DATE_TRUNC('month', claim_date) as month,
    COUNT(*) as claim_count,
    SUM(amount) as total_amount,
    -- 3-month moving average
    ROUND(AVG(COUNT(*)) OVER (
        ORDER BY DATE_TRUNC('month', claim_date)
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 2) as ma_3mo_count,
    -- Year-over-year growth
    ROUND(100.0 * (COUNT(*) - LAG(COUNT(*), 12) OVER (ORDER BY DATE_TRUNC('month', claim_date))) /
          NULLIF(LAG(COUNT(*), 12) OVER (ORDER BY DATE_TRUNC('month', claim_date)), 0), 2) as yoy_growth_pct,
    -- Seasonal index (current month vs 12-month average)
    ROUND(100.0 * COUNT(*) / AVG(COUNT(*)) OVER (
        ORDER BY DATE_TRUNC('month', claim_date)
        ROWS BETWEEN 11 PRECEDING AND CURRENT ROW
    ), 2) as seasonal_index
FROM claims
WHERE claim_date >= '2020-01-01'
GROUP BY DATE_TRUNC('month', claim_date)
ORDER BY month;
```

---

**Q92: How do you query graph data using SQL (shortest path, connected components)?**

**Answer:**

**Shortest Path (using Recursive CTE):**
```sql
-- Find shortest path between two nodes
CREATE TABLE graph_edges (
    from_node INT,
    to_node INT,
    weight INT
);

-- Dijkstra's algorithm approximation
WITH RECURSIVE paths AS (
    -- Start node
    SELECT
        from_node,
        to_node,
        weight as total_weight,
        ARRAY[from_node, to_node] as path,
        1 as depth
    FROM graph_edges
    WHERE from_node = 1  -- Starting node

    UNION ALL

    -- Extend path
    SELECT
        p.from_node,
        e.to_node,
        p.total_weight + e.weight,
        p.path || e.to_node,
        p.depth + 1
    FROM paths p
    JOIN graph_edges e ON p.to_node = e.from_node
    WHERE NOT (e.to_node = ANY(p.path))  -- Avoid cycles
      AND p.depth < 10  -- Limit depth
)
SELECT
    path,
    total_weight
FROM paths
WHERE to_node = 10  -- Target node
ORDER BY total_weight ASC
LIMIT 1;
```

**Connected Components:**
```sql
-- Find all connected components in graph
WITH RECURSIVE components AS (
    -- Start with all nodes
    SELECT
        node_id,
        node_id as component_id
    FROM (
        SELECT DISTINCT from_node as node_id FROM graph_edges
        UNION
        SELECT DISTINCT to_node as node_id FROM graph_edges
    ) nodes

    UNION

    -- Propagate component ID
    SELECT
        e.to_node as node_id,
        LEAST(c.component_id, c2.component_id) as component_id
    FROM components c
    JOIN graph_edges e ON c.node_id = e.from_node
    JOIN components c2 ON e.to_node = c2.node_id
    WHERE c.component_id != c2.component_id
)
SELECT
    component_id,
    COUNT(*) as component_size,
    ARRAY_AGG(node_id ORDER BY node_id) as nodes
FROM (
    SELECT DISTINCT ON (node_id) node_id, component_id
    FROM components
    ORDER BY node_id, component_id
) unique_components
GROUP BY component_id
ORDER BY component_size DESC;
```

**Optum Provider Network Analysis:**
```sql
-- Referral network: Which providers refer to each other
CREATE TABLE provider_referrals (
    referring_provider_id INT,
    referred_to_provider_id INT,
    referral_count INT
);

-- Find provider influence (who has most referrals incoming)
WITH RECURSIVE referral_network AS (
    -- Direct referrals
    SELECT
        referring_provider_id as source,
        referred_to_provider_id as target,
        referral_count,
        1 as degree,
        ARRAY[referring_provider_id, referred_to_provider_id] as path
    FROM provider_referrals

    UNION ALL

    -- Indirect referrals (2nd, 3rd degree)
    SELECT
        rn.source,
        pr.referred_to_provider_id,
        rn.referral_count + pr.referral_count,
        rn.degree + 1,
        rn.path || pr.referred_to_provider_id
    FROM referral_network rn
    JOIN provider_referrals pr ON rn.target = pr.referring_provider_id
    WHERE NOT (pr.referred_to_provider_id = ANY(rn.path))  -- No cycles
      AND rn.degree < 3  -- Limit to 3 degrees
)
SELECT
    target as provider_id,
    COUNT(DISTINCT source) as referring_providers,
    SUM(referral_count) as total_referrals,
    ROUND(AVG(degree), 2) as avg_degrees_of_separation
FROM referral_network
GROUP BY target
ORDER BY total_referrals DESC
LIMIT 20;
```

---

**Q93: How do you perform text analysis in SQL?**

**Answer:**

**Full-Text Search (PostgreSQL):**
```sql
-- Create full-text search index
CREATE INDEX idx_articles_fts ON articles
USING gin(to_tsvector('english', title || ' ' || content));

-- Search for documents containing keywords
SELECT
    article_id,
    title,
    ts_rank(to_tsvector('english', title || ' ' || content),
            to_tsquery('english', 'database & performance')) as relevance
FROM articles
WHERE to_tsvector('english', title || ' ' || content) @@
      to_tsquery('english', 'database & performance')
ORDER BY relevance DESC
LIMIT 10;

-- Boolean operators
-- & (AND): 'cat & dog'
-- | (OR): 'cat | dog'
-- ! (NOT): 'cat & !dog'
-- <-> (FOLLOWED BY): 'data <-> engineering'
```

**Pattern Matching with Regex:**
```sql
-- Extract email domains
SELECT
    email,
    SUBSTRING(email FROM '@(.*)$') as domain
FROM users
WHERE email ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$';

-- Find phone numbers
SELECT *
FROM contacts
WHERE phone ~ '^\(?[0-9]{3}\)?[-.\s]?[0-9]{3}[-.\s]?[0-9]{4}$';

-- Extract hashtags
SELECT
    tweet_id,
    REGEXP_MATCHES(tweet_text, '#\w+', 'g') as hashtags
FROM tweets;
```

**Fuzzy Matching (Levenshtein Distance):**
```sql
-- Requires fuzzystrmatch extension
CREATE EXTENSION IF NOT EXISTS fuzzystrmatch;

-- Find similar names
SELECT
    name,
    levenshtein('John Smith', name) as edit_distance,
    similarity('John Smith', name) as similarity_score
FROM customers
WHERE levenshtein('John Smith', name) < 3  -- Within 3 edits
ORDER BY edit_distance;

-- Soundex for phonetic matching
SELECT
    name,
    soundex(name)
FROM customers
WHERE soundex(name) = soundex('Smith');  -- Finds Smith, Smyth, etc.
```

**Text Analytics:**
```sql
-- Word frequency
SELECT
    word,
    COUNT(*) as frequency
FROM (
    SELECT unnest(string_to_array(lower(content), ' ')) as word
    FROM articles
) words
WHERE length(word) > 3  -- Exclude short words
GROUP BY word
ORDER BY frequency DESC
LIMIT 100;

-- N-grams (bigrams)
SELECT
    word1 || ' ' || word2 as bigram,
    COUNT(*) as frequency
FROM (
    SELECT
        word as word1,
        LEAD(word) OVER (PARTITION BY article_id ORDER BY position) as word2
    FROM (
        SELECT
            article_id,
            unnest(string_to_array(content, ' ')) as word,
            generate_series(1, array_length(string_to_array(content, ' '), 1)) as position
        FROM articles
    ) words_with_position
) bigrams
WHERE word2 IS NOT NULL
GROUP BY bigram
ORDER BY frequency DESC
LIMIT 50;
```

**Optum Clinical Notes Analysis:**
```sql
-- Full-text search in clinical notes
CREATE INDEX idx_notes_fts ON clinical_notes
USING gin(to_tsvector('english', note_text));

-- Find patients with specific conditions mentioned
SELECT
    patient_id,
    note_date,
    ts_headline('english', note_text,
                to_tsquery('english', 'diabetes & (type1 | type2)'),
                'StartSel=**, StopSel=**') as highlighted_text,
    ts_rank(to_tsvector('english', note_text),
            to_tsquery('english', 'diabetes & (type1 | type2)')) as relevance
FROM clinical_notes
WHERE to_tsvector('english', note_text) @@
      to_tsquery('english', 'diabetes & (type1 | type2)')
ORDER BY relevance DESC;

-- Extract medication mentions
SELECT
    note_id,
    REGEXP_MATCHES(note_text, '(metformin|insulin|glipizide)', 'gi') as medication
FROM clinical_notes
WHERE note_text ~* '(metformin|insulin|glipizide)';

-- Fuzzy match diagnosis codes (handle typos)
SELECT
    n.patient_id,
    n.note_id,
    d.icd10_code,
    d.description,
    levenshtein(lower(n.diagnosis_text), lower(d.description)) as edit_distance
FROM clinical_notes n
CROSS JOIN diagnosis_codes d
WHERE levenshtein(lower(n.diagnosis_text), lower(d.description)) < 5
ORDER BY n.patient_id, edit_distance;
```

---

**Q94: How do you work with JSON data in SQL?**

**Answer:**

**JSON Operators (PostgreSQL):**
```sql
-- -> : Get JSON object/array element
-- ->> : Get JSON object/array element as text
-- #> : Get JSON object at path
-- #>> : Get JSON object at path as text
-- @> : Contains
-- <@ : Contained by

CREATE TABLE user_profiles (
    user_id INT,
    profile_data JSONB  -- JSONB is faster than JSON
);

-- Insert JSON data
INSERT INTO user_profiles VALUES
(1, '{"name": "Alice", "age": 30, "address": {"city": "NYC", "zip": "10001"}, "tags": ["premium", "verified"]}'),
(2, '{"name": "Bob", "age": 25, "address": {"city": "LA", "zip": "90001"}, "tags": ["new"]}');
```

**Extracting JSON Fields:**
```sql
-- Get top-level field
SELECT
    user_id,
    profile_data->>'name' as name,  -- Text
    profile_data->'age' as age_json, -- JSON
    (profile_data->>'age')::INT as age_int  -- Convert to int
FROM user_profiles;

-- Get nested field
SELECT
    user_id,
    profile_data->'address'->>'city' as city,
    profile_data#>>'{address,city}' as city_alt  -- Alternative syntax
FROM user_profiles;

-- Get array element
SELECT
    user_id,
    profile_data->'tags'->0 as first_tag,  -- JSON
    profile_data->'tags'->>1 as second_tag  -- Text
FROM user_profiles;
```

**JSON Querying:**
```sql
-- Contains operator
SELECT * FROM user_profiles
WHERE profile_data @> '{"address": {"city": "NYC"}}';

-- Key exists
SELECT * FROM user_profiles
WHERE profile_data ? 'age';

-- Any key exists
SELECT * FROM user_profiles
WHERE profile_data ?| ARRAY['age', 'email'];

-- All keys exist
SELECT * FROM user_profiles
WHERE profile_data ?& ARRAY['name', 'age'];
```

**JSON Aggregation:**
```sql
-- Array to JSON
SELECT
    city,
    json_agg(name) as users
FROM (
    SELECT
        profile_data->>'name' as name,
        profile_data->'address'->>'city' as city
    FROM user_profiles
) t
GROUP BY city;

-- Object aggregation
SELECT
    json_object_agg(
        profile_data->>'name',
        profile_data->>'age'
    ) as name_age_map
FROM user_profiles;
```

**Expanding JSON:**
```sql
-- jsonb_each: Expand to key-value pairs
SELECT
    user_id,
    key,
    value
FROM user_profiles,
     jsonb_each(profile_data);

-- jsonb_array_elements: Expand array
SELECT
    user_id,
    jsonb_array_elements_text(profile_data->'tags') as tag
FROM user_profiles;
```

**JSON Index:**
```sql
-- GIN index on entire JSON
CREATE INDEX idx_profile_data ON user_profiles USING gin(profile_data);

-- Expression index on specific field
CREATE INDEX idx_profile_city ON user_profiles((profile_data->'address'->>'city'));
```

**Optum Patient Metadata:**
```sql
-- Store flexible patient attributes in JSONB
CREATE TABLE patient_metadata (
    patient_id INT,
    metadata JSONB
);

-- Sample data
INSERT INTO patient_metadata VALUES
(1, '{"allergies": ["penicillin", "shellfish"], "conditions": ["diabetes", "hypertension"], "last_visit": "2024-05-01", "insurance": {"provider": "UnitedHealth", "plan": "PPO", "member_id": "12345"}}');

-- Query patients with specific allergy
SELECT
    patient_id,
    metadata->>'last_visit' as last_visit
FROM patient_metadata
WHERE metadata->'allergies' ? 'penicillin';

-- Update nested JSON field
UPDATE patient_metadata
SET metadata = jsonb_set(
    metadata,
    '{insurance,plan}',
    '"HMO"'
)
WHERE patient_id = 1;

-- Add new element to array
UPDATE patient_metadata
SET metadata = jsonb_set(
    metadata,
    '{allergies}',
    (metadata->'allergies') || '["latex"]'::jsonb
)
WHERE patient_id = 1;

-- Extract insurance provider for all patients
SELECT
    patient_id,
    metadata#>>'{insurance,provider}' as insurance_provider,
    metadata#>>'{insurance,plan}' as plan_type
FROM patient_metadata;

-- Count patients by number of conditions
SELECT
    jsonb_array_length(metadata->'conditions') as condition_count,
    COUNT(*) as patient_count
FROM patient_metadata
GROUP BY jsonb_array_length(metadata->'conditions');
```

---

**Q95: What are transaction isolation levels? How do they differ?**

**Answer:**

Transaction isolation levels control how concurrent transactions interact.

**Four Isolation Levels (lowest to highest):**

**1. READ UNCOMMITTED (Dirty Reads Allowed):**
- Can read uncommitted changes from other transactions
- Rarely used (not even supported in PostgreSQL)

**2. READ COMMITTED (Default in PostgreSQL, SQL Server):**
- Only see committed data
- Each query sees latest committed data
- Can have non-repeatable reads

```sql
-- Transaction 1
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT balance FROM accounts WHERE id = 1;  -- Returns 1000

-- Transaction 2 (in parallel)
BEGIN;
UPDATE accounts SET balance = 1500 WHERE id = 1;
COMMIT;

-- Transaction 1 (continuing)
SELECT balance FROM accounts WHERE id = 1;  -- Returns 1500 (changed!)
COMMIT;
```

**3. REPEATABLE READ:**
- Sees snapshot of data at transaction start
- Same query returns same results within transaction
- Can have phantom reads (new rows)

```sql
-- Transaction 1
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT COUNT(*) FROM orders WHERE status = 'pending';  -- Returns 10

-- Transaction 2
BEGIN;
INSERT INTO orders (status) VALUES ('pending');
COMMIT;

-- Transaction 1
SELECT COUNT(*) FROM orders WHERE status = 'pending';  -- Still 10 (snapshot isolation)
-- But: SELECT * might show 11 rows (phantom read)
COMMIT;
```

**4. SERIALIZABLE (Strictest):**
- Complete isolation
- Transactions appear to run sequentially
- No dirty reads, non-repeatable reads, or phantom reads
- Can cause serialization errors (need retry logic)

```sql
BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT * FROM accounts WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 1;
COMMIT;  -- May fail with serialization error if conflict detected
```

**Comparison:**

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|-----------------|------------|---------------------|--------------|-------------|
| READ UNCOMMITTED | ✅ Possible | ✅ Possible | ✅ Possible | Fastest |
| READ COMMITTED | ❌ Prevented | ✅ Possible | ✅ Possible | Fast |
| REPEATABLE READ | ❌ Prevented | ❌ Prevented | ✅ Possible | Slower |
| SERIALIZABLE | ❌ Prevented | ❌ Prevented | ❌ Prevented | Slowest |

**Setting Isolation Level:**
```sql
-- Session level
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Transaction level
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- queries...
COMMIT;
```

**Optum Claims Processing:**
```sql
-- Use SERIALIZABLE for financial transactions (claim payments)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Check claim hasn't been paid
SELECT status FROM claims WHERE claim_id = 'C12345';

-- Process payment
UPDATE claims SET status = 'PAID', paid_amount = 1000 WHERE claim_id = 'C12345';
INSERT INTO payments (claim_id, amount) VALUES ('C12345', 1000);

COMMIT;  -- If another transaction processed same claim, this fails with serialization error

-- Handle serialization errors in application code:
-- try {
--     execute_transaction()
-- } catch (SerializationError) {
--     retry_with_backoff()
-- }

-- Use READ COMMITTED for read-heavy analytics (faster)
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT COUNT(*), AVG(amount) FROM claims WHERE status = 'PENDING';
COMMIT;
```

---

**Q96: How do you handle large JOINs efficiently?**

**Answer:**

**Strategies for large JOIN optimization:**

**1. Indexing:**
```sql
-- Index join columns
CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE INDEX idx_customers_id ON customers(id);

-- Now this join uses index lookup
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.id;
```

**2. Filter Before JOIN:**
```sql
-- ❌ Bad: Join then filter
SELECT *
FROM large_table1 t1
JOIN large_table2 t2 ON t1.id = t2.id
WHERE t1.date > '2024-01-01';

-- ✅ Good: Filter then join
SELECT *
FROM (SELECT * FROM large_table1 WHERE date > '2024-01-01') t1
JOIN large_table2 t2 ON t1.id = t2.id;
```

**3. Use EXISTS Instead of JOIN (when only checking existence):**
```sql
-- ❌ Slower: JOIN
SELECT DISTINCT c.*
FROM customers c
JOIN orders o ON c.id = o.customer_id;

-- ✅ Faster: EXISTS
SELECT c.*
FROM customers c
WHERE EXISTS (SELECT 1 FROM orders WHERE customer_id = c.id);
```

**4. Partition JOIN (divide and conquer):**
```sql
-- Join by partition key
SELECT *
FROM orders PARTITION (p_2024_01) o
JOIN order_items PARTITION (p_2024_01) oi ON o.order_id = oi.order_id;
```

**5. Bloom Filters (in distributed systems like Spark):**
```sql
-- Spark example (conceptual in SQL)
-- Small table creates bloom filter of join keys
-- Large table filters rows using bloom filter before shuffle
-- Reduces data transferred across network
```

**6. Join Order Optimization:**
```sql
-- ❌ Bad: Large table first
FROM large_table (1B rows)
JOIN medium_table (100M rows)
JOIN small_table (1K rows)

-- ✅ Good: Small table first (query optimizer should do this)
FROM small_table (1K rows)
JOIN medium_table (100M rows)
JOIN large_table (1B rows)
```

**7. Materialized Views for Complex Joins:**
```sql
-- Precompute expensive join
CREATE MATERIALIZED VIEW customer_order_summary AS
SELECT
    c.customer_id,
    c.name,
    COUNT(o.order_id) as order_count,
    SUM(o.amount) as total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name;

-- Refresh periodically
REFRESH MATERIALIZED VIEW customer_order_summary;

-- Query MV instead of joining
SELECT * FROM customer_order_summary WHERE total_spent > 10000;
```

**8. Partitioned JOIN (both tables partitioned by join key):**
```sql
-- Both tables partitioned by customer_id
CREATE TABLE orders_by_customer (
    customer_id INT,
    order_id INT,
    amount DECIMAL
) PARTITION BY HASH (customer_id);

CREATE TABLE customer_details (
    customer_id INT,
    name VARCHAR,
    email VARCHAR
) PARTITION BY HASH (customer_id);

-- JOIN reads matching partitions only (colocated join)
SELECT *
FROM orders_by_customer o
JOIN customer_details c ON o.customer_id = c.customer_id;
```

**Optum Claims JOIN Optimization:**
```sql
-- Problem: Join 200M claims with 10M patients and 5M providers
-- Before: 30 minutes

-- Solution 1: Index all join keys
CREATE INDEX idx_claims_patient ON claims(patient_id);
CREATE INDEX idx_claims_provider ON claims(provider_id);
CREATE INDEX idx_patients_id ON patients(patient_id);
CREATE INDEX idx_providers_id ON providers(provider_id);

-- Solution 2: Filter early
SELECT
    c.claim_id,
    p.patient_name,
    pr.provider_name,
    c.amount
FROM (
    SELECT * FROM claims
    WHERE claim_date >= '2024-01-01'  -- Reduce from 200M to 20M
) c
JOIN patients p ON c.patient_id = p.patient_id
JOIN providers pr ON c.provider_id = pr.provider_id;

-- Solution 3: Denormalize for common queries (star schema)
CREATE TABLE fact_claims_denorm AS
SELECT
    c.claim_id,
    c.claim_date,
    c.amount,
    p.patient_id,
    p.patient_name,  -- Denormalized
    pr.provider_id,
    pr.provider_name,  -- Denormalized
    pr.specialty  -- Denormalized
FROM claims c
JOIN patients p ON c.patient_id = p.patient_id
JOIN providers pr ON c.provider_id = pr.provider_id;

-- Now queries are fast (no joins!)
SELECT provider_name, SUM(amount)
FROM fact_claims_denorm
WHERE claim_date >= '2024-01-01'
GROUP BY provider_name;

-- Result: 30 min → 3 seconds (600x faster)
```

---

**Q97: How do you perform comprehensive data quality checks?**

**Answer:**

**Data Quality Dimensions:**
1. Completeness (no NULLs)
2. Accuracy (valid values)
3. Consistency (referential integrity)
4. Uniqueness (no duplicates)
5. Timeliness (fresh data)

**Comprehensive Quality Check Query:**
```sql
WITH quality_checks AS (
    SELECT
        -- Completeness
        COUNT(*) as total_rows,
        COUNT(CASE WHEN email IS NULL THEN 1 END) as null_email,
        COUNT(CASE WHEN phone IS NULL THEN 1 END) as null_phone,
        COUNT(CASE WHEN created_at IS NULL THEN 1 END) as null_created_at,

        -- Accuracy
        COUNT(CASE WHEN email !~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$'
                   THEN 1 END) as invalid_email,
        COUNT(CASE WHEN phone !~ '^\d{10}$' THEN 1 END) as invalid_phone,
        COUNT(CASE WHEN age < 0 OR age > 150 THEN 1 END) as invalid_age,
        COUNT(CASE WHEN amount < 0 THEN 1 END) as negative_amount,

        -- Timeliness
        COUNT(CASE WHEN created_at > CURRENT_DATE THEN 1 END) as future_dates,
        COUNT(CASE WHEN created_at < CURRENT_DATE - INTERVAL '10 years'
                   THEN 1 END) as very_old_records,

        -- Uniqueness
        COUNT(*) - COUNT(DISTINCT user_id) as duplicate_user_ids,
        COUNT(*) - COUNT(DISTINCT email) as duplicate_emails
    FROM users
)
SELECT
    total_rows,
    -- Completeness score
    ROUND(100.0 * (total_rows - null_email - null_phone - null_created_at) /
          (total_rows * 3), 2) as completeness_score_pct,
    -- Accuracy score
    ROUND(100.0 * (total_rows - invalid_email - invalid_phone - invalid_age - negative_amount) /
          (total_rows * 4), 2) as accuracy_score_pct,
    -- Detail counts
    null_email,
    null_phone,
    invalid_email,
    invalid_phone,
    invalid_age,
    negative_amount,
    future_dates,
    very_old_records,
    duplicate_user_ids,
    duplicate_emails,
    -- Overall quality score
    ROUND((
        100.0 * (total_rows - null_email - null_phone - null_created_at) / (total_rows * 3) +
        100.0 * (total_rows - invalid_email - invalid_phone - invalid_age - negative_amount) / (total_rows * 4)
    ) / 2, 2) as overall_quality_score_pct
FROM quality_checks;
```

**Referential Integrity Checks:**
```sql
-- Orphaned records (foreign key violations)
SELECT
    'Orphaned Orders' as check_name,
    COUNT(*) as violation_count
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL

UNION ALL

SELECT
    'Orphaned Order Items',
    COUNT(*)
FROM order_items oi
LEFT JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_id IS NULL;
```

**Statistical Outlier Detection:**
```sql
WITH stats AS (
    SELECT
        AVG(amount) as mean_amount,
        STDDEV(amount) as stddev_amount,
        PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY amount) as q1,
        PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY amount) as q3
    FROM transactions
)
SELECT
    COUNT(CASE WHEN amount > mean_amount + 3 * stddev_amount THEN 1 END) as high_outliers_3std,
    COUNT(CASE WHEN amount < mean_amount - 3 * stddev_amount THEN 1 END) as low_outliers_3std,
    COUNT(CASE WHEN amount > q3 + 1.5 * (q3 - q1) THEN 1 END) as high_outliers_iqr,
    COUNT(CASE WHEN amount < q1 - 1.5 * (q3 - q1) THEN 1 END) as low_outliers_iqr
FROM transactions, stats;
```

**Optum Claims Data Quality:**
```sql
-- Comprehensive claims data quality checks
WITH claim_quality AS (
    SELECT
        COUNT(*) as total_claims,

        -- Completeness
        COUNT(CASE WHEN claim_id IS NULL THEN 1 END) as missing_claim_id,
        COUNT(CASE WHEN patient_id IS NULL THEN 1 END) as missing_patient_id,
        COUNT(CASE WHEN provider_id IS NULL THEN 1 END) as missing_provider_id,
        COUNT(CASE WHEN amount IS NULL THEN 1 END) as missing_amount,

        -- Accuracy
        COUNT(CASE WHEN amount <= 0 THEN 1 END) as invalid_amount,
        COUNT(CASE WHEN service_date > claim_date THEN 1 END) as service_after_claim,
        COUNT(CASE WHEN claim_date > CURRENT_DATE THEN 1 END) as future_claims,
        COUNT(CASE WHEN patient_age < 0 OR patient_age > 120 THEN 1 END) as invalid_age,

        -- Referential Integrity
        (SELECT COUNT(*) FROM claims c
         LEFT JOIN patients p ON c.patient_id = p.patient_id
         WHERE p.patient_id IS NULL) as orphaned_patients,
        (SELECT COUNT(*) FROM claims c
         LEFT JOIN providers pr ON c.provider_id = pr.provider_id
         WHERE pr.provider_id IS NULL) as orphaned_providers,

        -- Uniqueness
        COUNT(*) - COUNT(DISTINCT claim_id) as duplicate_claim_ids,

        -- Business rules
        COUNT(CASE WHEN status = 'PAID' AND paid_amount IS NULL THEN 1 END) as paid_without_amount,
        COUNT(CASE WHEN status IN ('APPROVED', 'PAID') AND denial_reason IS NOT NULL
                   THEN 1 END) as approved_with_denial_reason
    FROM claims
)
SELECT
    total_claims,
    -- Issues
    missing_claim_id,
    missing_patient_id,
    missing_provider_id,
    missing_amount,
    invalid_amount,
    service_after_claim,
    future_claims,
    invalid_age,
    orphaned_patients,
    orphaned_providers,
    duplicate_claim_ids,
    paid_without_amount,
    approved_with_denial_reason,
    -- Overall quality score
    ROUND(100.0 * (
        total_claims -
        missing_claim_id - missing_patient_id - missing_provider_id - missing_amount -
        invalid_amount - service_after_claim - future_claims - invalid_age -
        orphaned_patients - orphaned_providers - duplicate_claim_ids -
        paid_without_amount - approved_with_denial_reason
    ) / total_claims, 2) as quality_score_pct,
    -- Alert level
    CASE
        WHEN (missing_claim_id + invalid_amount + orphaned_patients + orphaned_providers) > total_claims * 0.01
            THEN 'CRITICAL - >1% severe issues'
        WHEN (missing_claim_id + missing_patient_id + invalid_amount) > total_claims * 0.05
            THEN 'WARNING - >5% issues'
        ELSE 'OK'
    END as alert_level
FROM claim_quality;

-- Result: Run daily, alert if quality_score_pct < 95% or alert_level != 'OK'
```

---

**Q98-Q100: Already covered in detailed answers above (Q86-Q97)**

---

---

## Summary & Study Strategy

**Python (50 Q):**
- ✅ Fundamentals (Q1-Q15): List vs tuple, comprehensions, decorators
- ✅ Intermediate (Q16-Q35): Context managers, generators, map/filter/reduce
- ✅ Advanced (Q36-Q50): Optimization, memory, async, data engineering patterns

**SQL (50 Q):**
- ✅ Fundamentals (Q51-Q65): JOINs, GROUP BY, window functions
- ✅ Intermediate (Q66-Q85): Nth highest, duplicates, running totals, pivots
- ✅ Advanced (Q86-Q100): Optimization, SCD, sessionization, cohorts

**Study Plan:**
- **Week 1:** Python Q1-15, SQL Q51-65 (fundamentals)
- **Week 2:** Python Q16-35, SQL Q66-85 (intermediate)
- **Week 3:** Python Q36-50, SQL Q86-100 + practice problems

**Practice:**
- LeetCode: 50+ SQL problems
- HackerRank: Python challenges
- Write code daily, don't just read

**Interview Tip:**
Know fundamentals cold (Q1-65). Advanced questions (Q66-100) show depth but basics are tested more often.

Good luck! 🚀

---

**Q85: Explain Python decorators with examples.**

Functions that modify other functions. Used for logging, timing, access control.

```python
def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time() - start}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(2)
```

---

**Q86: What are context managers? Explain `with` statement.**

Manage resources (files, locks, connections) with setup/teardown.

```python
with open('file.txt') as f:
    data = f.read()
# File automatically closed

# Custom context manager
from contextlib import contextmanager

@contextmanager
def database_connection():
    conn = get_connection()
    try:
        yield conn
    finally:
        conn.close()
```

---

**Q87: Explain SQL window functions.**

Perform calculations across rows related to current row.

```sql
-- Running total
SELECT 
    date,
    amount,
    SUM(amount) OVER (ORDER BY date) as running_total
FROM claims;

-- Rank by amount per state
SELECT 
    state,
    claim_id,
    amount,
    RANK() OVER (PARTITION BY state ORDER BY amount DESC) as rank
FROM claims;
```

**Common window functions:** ROW_NUMBER, RANK, DENSE_RANK, LAG, LEAD, NTILE.

---

**Q88: How to optimize slow SQL queries?**

1. Add indexes on WHERE/JOIN columns
2. Use EXPLAIN to analyze query plan
3. Avoid SELECT *, fetch only needed columns
4. Use EXISTS instead of IN for subqueries
5. Partition large tables
6. Update statistics

```sql
-- Before
SELECT * FROM claims WHERE member_id = '123';

-- After (with index)
CREATE INDEX idx_member_id ON claims(member_id);
SELECT claim_id, amount FROM claims WHERE member_id = '123';
```

---

**Q89: Explain Python generators and `yield`.**

Generate values lazily (memory efficient).

```python
def read_large_file(file_path):
    with open(file_path) as f:
        for line in f:
            yield line.strip()  # Yields one line at a time

# Memory efficient (doesn't load entire file)
for line in read_large_file('huge.csv'):
    process(line)
```

---

**Q90: What are SQL CTEs (Common Table Expressions)?**

Named temporary result sets. Improve readability.

```sql
WITH high_value_claims AS (
    SELECT * FROM claims WHERE amount > 10000
),
fraud_scored AS (
    SELECT *, fraud_score(claim_id) as score
    FROM high_value_claims
)
SELECT * FROM fraud_scored WHERE score > 0.8;
```

---

**Q91: Explain Python `*args` and `**kwargs`.**

Variable-length arguments.

```python
def process(*args):  # Tuple of positional args
    for arg in args:
        print(arg)

def configure(**kwargs):  # Dict of keyword args
    for key, value in kwargs.items():
        print(f"{key} = {value}")

process(1, 2, 3)  # args = (1, 2, 3)
configure(host='localhost', port=5432)  # kwargs = {'host': 'localhost', 'port': 5432}
```

---

**Q92: How to handle NULL values in SQL?**

```sql
-- Check for NULL
SELECT * FROM claims WHERE diagnosis_code IS NULL;

-- Replace NULL
SELECT COALESCE(diagnosis_code, 'UNKNOWN') FROM claims;

-- Null-safe comparison
SELECT * FROM claims WHERE IFNULL(amount, 0) > 100;

-- Aggregate functions ignore NULLs
SELECT AVG(amount) FROM claims;  -- Excludes NULLs
```

---

**Q93: Explain Python list comprehensions vs generator expressions.**

```python
# List comprehension (creates entire list in memory)
squares = [x**2 for x in range(1000000)]  # Uses ~40MB

# Generator expression (lazy evaluation)
squares = (x**2 for x in range(1000000))  # Uses minimal memory

# Use generator for large datasets
total = sum(x**2 for x in range(1000000))  # Memory efficient
```

---

**Q94: What are SQL indexes? Types and trade-offs?**

Speed up queries by creating lookup structures.

**Types:**
- **B-tree:** Default, good for range queries
- **Hash:** Fast exact match, no range queries
- **Bitmap:** Low-cardinality columns (gender, status)
- **Composite:** Multiple columns

```sql
CREATE INDEX idx_member_date ON claims(member_id, claim_date);
```

**Trade-offs:**
- ✅ Faster reads
- ❌ Slower writes (index must be updated)
- ❌ Storage overhead

---

**Q95: Explain Python `asyncio` for concurrent I/O.**

Async/await for non-blocking I/O.

```python
import asyncio
import aiohttp

async def fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

async def main():
    # Run 10 requests concurrently
    urls = [f"https://api.example.com/data/{i}" for i in range(10)]
    results = await asyncio.gather(*[fetch(url) for url in urls])

asyncio.run(main())
```

---

**Q96: How to find duplicate rows in SQL?**

```sql
-- Find duplicates
SELECT member_id, claim_date, COUNT(*) as count
FROM claims
GROUP BY member_id, claim_date
HAVING COUNT(*) > 1;

-- Delete duplicates (keep first)
WITH ranked AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY member_id, claim_date ORDER BY id) as rn
    FROM claims
)
DELETE FROM claims WHERE id IN (SELECT id FROM ranked WHERE rn > 1);
```

---

**Q97: Explain Python `functools` module.**

Higher-order functions and function tools.

```python
from functools import lru_cache, partial, reduce

# Memoization (cache results)
@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Partial application
from operator import add
add_10 = partial(add, 10)
print(add_10(5))  # 15

# Reduce
from functools import reduce
total = reduce(lambda x, y: x + y, [1, 2, 3, 4])  # 10
```

---

**Q98: What are SQL execution plans? How to read EXPLAIN?**

Shows how database executes query (scan type, join method, cost).

```sql
EXPLAIN SELECT * FROM claims c
JOIN members m ON c.member_id = m.id
WHERE c.amount > 1000;

-- Look for:
-- 1. Seq Scan (slow) vs Index Scan (fast)
-- 2. Join method (Hash, Merge, Nested Loop)
-- 3. Estimated rows vs actual
-- 4. High cost operations
```

**Optimize by:** adding indexes, rewriting query, updating statistics.

---

**Q99: Explain Python data classes (`@dataclass`).**

Simplify class creation with automatic __init__, __repr__, etc.

```python
from dataclasses import dataclass

@dataclass
class Claim:
    claim_id: str
    member_id: str
    amount: float
    date: str

claim = Claim('C123', 'M456', 1500.00, '2024-05-01')
print(claim)  # Claim(claim_id='C123', member_id='M456', amount=1500.0, date='2024-05-01')
```

---

**Q100: Tell me about a complex Python/SQL project you built at Optum.**

**Claims data validation pipeline:**
- Python ETL processing 10M+ claims daily
- SQL queries on 500GB PostgreSQL database
- Used Python multiprocessing for parallel validation
- Complex SQL with window functions for fraud detection scoring
- Implemented connection pooling (SQLAlchemy)
- Data quality checks with Great Expectations
- Reduced processing time from 8 hours to 2 hours
- 99.9% data quality score

**Key SQL optimization:**
- Added composite indexes on (member_id, claim_date)
- Partitioned claims table by month (improved query 10x)
- Used materialized views for reporting

**Key Python patterns:**
- Generators for memory-efficient processing
- Context managers for database connections
- Asyncio for concurrent API calls
- Decorators for logging/monitoring

---


**Q93:** Python type hints? Improve code clarity: `def process(data: List[dict]) -> pd.DataFrame:`. **Q94:** SQL query optimization checklist? Add indexes, use EXPLAIN, avoid SELECT *, filter early, use EXISTS not IN, partition tables. **Q95:** Python error handling? Try/except, raise custom exceptions, finally for cleanup. **Q96:** SQL transaction isolation levels? Read Uncommitted, Read Committed, Repeatable Read, Serializable. Higher isolation = more consistency, less concurrency. **Q97:** Python unit testing? pytest, unittest, mocking with unittest.mock. Test edge cases, exceptions. **Q98:** SQL performance tools? EXPLAIN, query analyzer, DMVs (SQL Server), pg_stat_statements (Postgres). **Q99:** Python logging best practices? Use logging module, structured logs (JSON), log levels (DEBUG/INFO/WARNING/ERROR), centralized logging. **Q100:** SQL vs NoSQL? SQL: ACID, structured, relations. NoSQL: flexible schema, horizontal scaling, eventually consistent. Use SQL for transactions, NoSQL for scale/flexibility.

