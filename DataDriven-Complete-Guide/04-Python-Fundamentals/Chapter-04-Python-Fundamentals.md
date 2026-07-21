# Chapter 4: Python Fundamentals for Data Engineering

**Duration:** 4-5 days | **Difficulty:** Beginner to Intermediate
**Problems to Solve:** 15-20 easy Python problems
**Prerequisites:** Basic Python knowledge

---

## 📚 Table of Contents

1. [Introduction](#introduction)
2. [Data Structures Deep Dive](#data-structures)
3. [Collections Module](#collections-module)
4. [String Processing](#string-processing)
5. [List and Dictionary Comprehensions](#comprehensions)
6. [Lambda Functions and Functional Programming](#lambda-functions)
7. [File I/O](#file-io)
8. [Error Handling](#error-handling)
9. [Practice Problems](#practice-problems)
10. [Solutions](#solutions)
11. [Summary](#summary)

---

## 1. Introduction {#introduction}

### Why Python for Data Engineering?

Python is the **Swiss Army knife** of data engineering:
- **Data processing:** pandas, numpy
- **Orchestration:** Apache Airflow
- **Streaming:** PySpark, Apache Flink
- **APIs:** Flask, FastAPI
- **Automation:** Scripting, ETL

### What You'll Learn

Master Python fundamentals that appear in 40% of DataDriven problems:
- ✅ Advanced data structures (Counter, defaultdict, deque)
- ✅ String manipulation and regex
- ✅ Pythonic patterns (comprehensions, lambdas)
- ✅ File processing (CSV, JSON)
- ✅ Error handling

---

## 2. Data Structures Deep Dive {#data-structures}

### Lists: Ordered, Mutable Collections

**Creation:**
```python
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True]
empty = []
from_range = list(range(10))  # [0, 1, 2, ..., 9]
```

**Common Operations:**
```python
numbers = [1, 2, 3, 4, 5]

# Access
numbers[0]        # 1 (first element)
numbers[-1]       # 5 (last element)
numbers[1:3]      # [2, 3] (slicing: start included, end excluded)
numbers[:3]       # [1, 2, 3] (first 3)
numbers[2:]       # [3, 4, 5] (from index 2)
numbers[-3:]      # [3, 4, 5] (last 3)

# Modification
numbers.append(6)           # [1, 2, 3, 4, 5, 6]
numbers.insert(0, 0)        # [0, 1, 2, 3, 4, 5, 6]
numbers.extend([7, 8])      # [0, 1, 2, 3, 4, 5, 6, 7, 8]
numbers.remove(0)           # Remove first occurrence of 0
numbers.pop()               # Remove and return last element
numbers.pop(0)              # Remove and return element at index 0

# Query
len(numbers)      # Length
min(numbers)      # Minimum
max(numbers)      # Maximum
sum(numbers)      # Sum
numbers.count(3)  # Count occurrences of 3
numbers.index(3)  # Index of first occurrence of 3

# Sorting
numbers.sort()                # In-place sort (ascending)
numbers.sort(reverse=True)    # In-place sort (descending)
sorted_nums = sorted(numbers) # Returns new sorted list

# Checking membership
3 in numbers      # True
10 in numbers     # False
```

**List as Stack (LIFO):**
```python
stack = []
stack.append(1)  # Push
stack.append(2)
stack.append(3)
top = stack.pop()  # Pop: returns 3
```

---

### Tuples: Ordered, Immutable Collections

**Why use tuples?**
- Immutable (can't modify)
- Faster than lists
- Can be dictionary keys
- Return multiple values from functions

```python
# Creation
point = (10, 20)
person = ("Alice", 30, "Engineer")
single = (1,)  # Single element tuple (note comma!)

# Access (same as lists)
point[0]       # 10
person[-1]     # "Engineer"

# Unpacking
x, y = point
name, age, job = person

# Multiple return values
def get_stats(numbers):
    return min(numbers), max(numbers), sum(numbers)

min_val, max_val, total = get_stats([1, 2, 3, 4, 5])
```

---

### Sets: Unordered, Unique Collections

**Use when:**
- Need unique elements
- Fast membership testing (O(1))
- Set operations (union, intersection)

```python
# Creation
numbers = {1, 2, 3, 4, 5}
from_list = set([1, 2, 2, 3, 3, 3])  # {1, 2, 3}
empty = set()  # NOT {}! That's a dict

# Operations
numbers.add(6)        # Add element
numbers.update([7, 8, 9])  # Add multiple
numbers.remove(1)     # Remove (raises error if not found)
numbers.discard(1)    # Remove (no error if not found)

# Set operations
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

a | b          # Union: {1, 2, 3, 4, 5, 6}
a & b          # Intersection: {3, 4}
a - b          # Difference (in a, not in b): {1, 2}
a ^ b          # Symmetric difference: {1, 2, 5, 6}

# Membership testing (FAST!)
3 in numbers   # True (O(1))
```

**Example: Find unique users**
```python
users_2023 = {1, 2, 3, 4, 5}
users_2024 = {4, 5, 6, 7, 8}

new_users = users_2024 - users_2023     # {6, 7, 8}
returning = users_2023 & users_2024     # {4, 5}
churned = users_2023 - users_2024       # {1, 2, 3}
```

---

### Dictionaries: Key-Value Pairs

**Use for:**
- Lookup tables
- Counting occurrences
- Grouping data
- Caching results

```python
# Creation
user = {
    "name": "Alice",
    "age": 30,
    "email": "alice@example.com"
}

# Access
user["name"]              # "Alice"
user.get("phone")         # None (no error)
user.get("phone", "N/A")  # "N/A" (default value)

# Modification
user["age"] = 31          # Update
user["phone"] = "555-1234"  # Add new key

# Deletion
del user["phone"]         # Remove key-value pair
user.pop("age")           # Remove and return value

# Query
len(user)                 # Number of keys
"name" in user            # True
user.keys()               # dict_keys(['name', 'age', 'email'])
user.values()             # dict_values(['Alice', 30, ...])
user.items()              # dict_items([('name', 'Alice'), ...])

# Iteration
for key in user:
    print(key, user[key])

for key, value in user.items():
    print(f"{key}: {value}")

# Merging (Python 3.9+)
user1 = {"name": "Alice", "age": 30}
user2 = {"age": 31, "city": "NYC"}
merged = user1 | user2  # {"name": "Alice", "age": 31, "city": "NYC"}
```

**Example: Count occurrences**
```python
words = ["apple", "banana", "apple", "cherry", "banana", "apple"]

# Manual counting
counts = {}
for word in words:
    if word in counts:
        counts[word] += 1
    else:
        counts[word] = 1

print(counts)  # {"apple": 3, "banana": 2, "cherry": 1}

# Better: Use get()
counts = {}
for word in words:
    counts[word] = counts.get(word, 0) + 1

# Even better: Use Counter (next section!)
```

---

## 3. Collections Module {#collections-module}

### Counter: Counting Made Easy

**The most useful tool for data engineering!**

```python
from collections import Counter

# Count occurrences
words = ["apple", "banana", "apple", "cherry", "banana", "apple"]
counts = Counter(words)
print(counts)  # Counter({'apple': 3, 'banana': 2, 'cherry': 1})

# Access counts
counts["apple"]     # 3
counts["grape"]     # 0 (default for missing keys)

# Most common
counts.most_common(2)  # [('apple', 3), ('banana', 2)]

# Operations
more_words = ["apple", "grape", "grape"]
counts.update(more_words)  # Add more counts

# Arithmetic
a = Counter(['a', 'b', 'c'])
b = Counter(['b', 'c', 'd'])
a + b  # Counter({'b': 2, 'c': 2, 'a': 1, 'd': 1})
a - b  # Counter({'a': 1})
```

**Example: DataDriven "Letters in the Noise"**
```python
def most_frequent_char(text):
    """Find most frequent character (excluding spaces)"""
    counts = Counter(char for char in text if char != ' ')
    if not counts:
        return None
    return counts.most_common(1)[0][0]

print(most_frequent_char("hello world"))  # 'l' (appears 3 times)
```

---

### defaultdict: Never KeyError Again

**Problem with dict:**
```python
# This crashes!
groups = {}
groups['a'].append(1)  # KeyError: 'a'

# Need to check first
if 'a' not in groups:
    groups['a'] = []
groups['a'].append(1)
```

**Solution with defaultdict:**
```python
from collections import defaultdict

# Automatically creates empty list for new keys
groups = defaultdict(list)
groups['a'].append(1)  # Works!
groups['a'].append(2)
groups['b'].append(3)
print(dict(groups))  # {'a': [1, 2], 'b': [3]}
```

**Common patterns:**

```python
# Group by key
from collections import defaultdict

transactions = [
    {'user': 'Alice', 'amount': 100},
    {'user': 'Bob', 'amount': 200},
    {'user': 'Alice', 'amount': 150},
]

# Group amounts by user
user_amounts = defaultdict(list)
for txn in transactions:
    user_amounts[txn['user']].append(txn['amount'])

print(dict(user_amounts))
# {'Alice': [100, 150], 'Bob': [200]}

# Count by category
category_counts = defaultdict(int)
for item in items:
    category_counts[item['category']] += 1

# Nested groups
nested = defaultdict(lambda: defaultdict(list))
nested['user1']['2024-01'].append(100)
```

---

### deque: Double-Ended Queue

**Efficient for:**
- Adding/removing from both ends (O(1))
- Sliding windows
- Breadth-first search
- Undo/redo functionality

```python
from collections import deque

# Creation
dq = deque([1, 2, 3])

# Add to both ends
dq.append(4)       # Right: deque([1, 2, 3, 4])
dq.appendleft(0)   # Left: deque([0, 1, 2, 3, 4])

# Remove from both ends
dq.pop()           # Remove from right: returns 4
dq.popleft()       # Remove from left: returns 0

# Rotation
dq = deque([1, 2, 3, 4, 5])
dq.rotate(2)       # deque([4, 5, 1, 2, 3])
dq.rotate(-2)      # deque([1, 2, 3, 4, 5])

# Max length (circular buffer)
recent = deque(maxlen=3)
recent.extend([1, 2, 3])  # deque([1, 2, 3])
recent.append(4)          # deque([2, 3, 4]) - 1 was removed
```

**Example: Sliding window maximum**
```python
from collections import deque

def sliding_window_max(arr, k):
    """Find maximum in each k-sized window"""
    result = []
    dq = deque()  # Store indices

    for i in range(len(arr)):
        # Remove elements outside window
        while dq and dq[0] <= i - k:
            dq.popleft()

        # Remove smaller elements (they'll never be max)
        while dq and arr[dq[-1]] < arr[i]:
            dq.pop()

        dq.append(i)

        # Add to result after first window complete
        if i >= k - 1:
            result.append(arr[dq[0]])

    return result

arr = [1, 3, -1, -3, 5, 3, 6, 7]
print(sliding_window_max(arr, 3))  # [3, 3, 5, 5, 6, 7]
```

---

### OrderedDict: Preserve Insertion Order

**Note:** Regular dicts preserve order in Python 3.7+, but OrderedDict is still useful for:
- Explicit ordering guarantee
- Moving elements to end
- Equality testing considers order

```python
from collections import OrderedDict

od = OrderedDict()
od['a'] = 1
od['b'] = 2
od['c'] = 3

# Move to end
od.move_to_end('a')  # OrderedDict([('b', 2), ('c', 3), ('a', 1)])

# Pop last
od.popitem(last=True)   # Remove from end
od.popitem(last=False)  # Remove from beginning
```

---

## 4. String Processing {#string-processing}

### Basic String Operations

```python
text = "Hello, World!"

# Case conversion
text.upper()       # "HELLO, WORLD!"
text.lower()       # "hello, world!"
text.capitalize()  # "Hello, world!"
text.title()       # "Hello, World!"

# Checking
text.startswith("Hello")  # True
text.endswith("!")        # True
"World" in text           # True
text.isalpha()            # False (has comma and space)
text.isdigit()            # False
text.isalnum()            # False

# Whitespace
text = "  hello  "
text.strip()    # "hello"
text.lstrip()   # "hello  "
text.rstrip()   # "  hello"

# Splitting and joining
text = "apple,banana,cherry"
fruits = text.split(",")     # ["apple", "banana", "cherry"]
joined = ", ".join(fruits)   # "apple, banana, cherry"

# Replace
text = "Hello World"
text.replace("World", "Python")  # "Hello Python"
text.replace("l", "L", 1)        # "HeLlo World" (replace first only)

# Find
text.find("World")   # 6 (index of first occurrence)
text.find("xyz")     # -1 (not found)
text.index("World")  # 6 (raises ValueError if not found)
```

---

### String Formatting

**Three methods:**

```python
name = "Alice"
age = 30

# 1. f-strings (Python 3.6+, BEST)
message = f"Hello, {name}! You are {age} years old."

# Expressions in f-strings
message = f"Next year you'll be {age + 1}"
message = f"Name in caps: {name.upper()}"

# Formatting numbers
price = 1234.567
message = f"Price: ${price:,.2f}"  # "Price: $1,234.57"

# Padding
num = 42
f"{num:05d}"  # "00042" (pad with zeros, total width 5)

# 2. str.format()
message = "Hello, {}! You are {} years old.".format(name, age)
message = "Hello, {name}! You are {age} years old.".format(name=name, age=age)

# 3. % formatting (old style)
message = "Hello, %s! You are %d years old." % (name, age)
```

---

### Regular Expressions (regex)

```python
import re

# Basic matching
text = "My email is alice@example.com"

# Find match
match = re.search(r'\w+@\w+\.\w+', text)
if match:
    print(match.group())  # "alice@example.com"

# Find all matches
text = "Emails: alice@ex.com, bob@ex.com"
emails = re.findall(r'\w+@\w+\.\w+', text)
print(emails)  # ['alice@ex.com', 'bob@ex.com']

# Split
text = "apple, banana; cherry"
fruits = re.split(r'[,;]\s*', text)
print(fruits)  # ['apple', 'banana', 'cherry']

# Replace
text = "Price: $100.50"
cleaned = re.sub(r'[$,]', '', text)
print(cleaned)  # "Price: 100.50"

# Capture groups
text = "Date: 2024-01-15"
match = re.search(r'(\d{4})-(\d{2})-(\d{2})', text)
year, month, day = match.groups()
print(f"Year: {year}, Month: {month}, Day: {day}")
```

**Common regex patterns:**
```python
# Patterns
r'\d'       # Digit
r'\w'       # Word character (letter, digit, underscore)
r'\s'       # Whitespace
r'.'        # Any character except newline
r'^'        # Start of string
r'$'        # End of string
r'*'        # 0 or more
r'+'        # 1 or more
r'?'        # 0 or 1
r'{3}'      # Exactly 3
r'{2,5}'    # 2 to 5

# Email validation (basic)
r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'

# Phone number (###-###-####)
r'^\d{3}-\d{3}-\d{4}$'

# URL
r'https?://[^\s]+'
```

**Example: Extract numbers from string**
```python
def extract_numbers(text):
    """Extract all numbers from text"""
    return [int(x) for x in re.findall(r'\d+', text)]

text = "I have 3 apples and 5 oranges, total 8 fruits"
print(extract_numbers(text))  # [3, 5, 8]
```

---

## 5. List and Dictionary Comprehensions {#comprehensions}

### List Comprehensions

**Pattern:** `[expression for item in iterable if condition]`

**Basic examples:**
```python
# Square numbers
squares = [x**2 for x in range(10)]
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# Filter even numbers
evens = [x for x in range(10) if x % 2 == 0]
# [0, 2, 4, 6, 8]

# Transform strings
names = ["alice", "bob", "charlie"]
upper_names = [name.upper() for name in names]
# ["ALICE", "BOB", "CHARLIE"]

# With condition
names = ["Alice", "Bob", "Charlie", "David"]
short_names = [name for name in names if len(name) <= 5]
# ["Alice", "Bob", "David"]
```

**Nested comprehensions:**
```python
# Flatten 2D list
matrix = [[1, 2], [3, 4], [5, 6]]
flat = [item for row in matrix for item in row]
# [1, 2, 3, 4, 5, 6]

# Cartesian product
colors = ["red", "blue"]
sizes = ["S", "M", "L"]
combinations = [(color, size) for color in colors for size in sizes]
# [('red', 'S'), ('red', 'M'), ('red', 'L'), ('blue', 'S'), ...]
```

**vs traditional loop:**
```python
# Traditional
squares = []
for x in range(10):
    squares.append(x**2)

# Comprehension (more Pythonic)
squares = [x**2 for x in range(10)]
```

---

### Dictionary Comprehensions

**Pattern:** `{key_expr: value_expr for item in iterable if condition}`

```python
# Square numbers as dict
squares = {x: x**2 for x in range(5)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# Swap keys and values
original = {'a': 1, 'b': 2, 'c': 3}
swapped = {v: k for k, v in original.items()}
# {1: 'a', 2: 'b', 3: 'c'}

# Filter
prices = {'apple': 1.20, 'banana': 0.50, 'cherry': 2.50}
expensive = {k: v for k, v in prices.items() if v > 1.00}
# {'apple': 1.20, 'cherry': 2.50}

# From two lists
keys = ['a', 'b', 'c']
values = [1, 2, 3]
d = {k: v for k, v in zip(keys, values)}
# {'a': 1, 'b': 2, 'c': 3}
```

---

### Set Comprehensions

```python
# Unique lengths
names = ["Alice", "Bob", "Charlie", "Al"]
lengths = {len(name) for name in names}
# {2, 3, 5, 7}
```

---

### When NOT to Use Comprehensions

**Avoid when:**
- Logic is too complex (use regular loop)
- Need multiple statements per iteration
- Readability suffers

**Bad (too complex):**
```python
result = [x if x > 0 else -x if x < 0 else 0 for x in numbers if x != 5]
```

**Good (use regular loop):**
```python
result = []
for x in numbers:
    if x != 5:
        if x > 0:
            result.append(x)
        elif x < 0:
            result.append(-x)
        else:
            result.append(0)
```

---

## 6. Lambda Functions and Functional Programming {#lambda-functions}

### Lambda Functions (Anonymous Functions)

**Syntax:** `lambda arguments: expression`

```python
# Regular function
def square(x):
    return x**2

# Lambda equivalent
square = lambda x: x**2

square(5)  # 25
```

**Common use: Short functions as arguments**

```python
# Sort by custom key
users = [
    {'name': 'Alice', 'age': 30},
    {'name': 'Bob', 'age': 25},
    {'name': 'Charlie', 'age': 35}
]

# Sort by age
sorted_users = sorted(users, key=lambda u: u['age'])

# Sort by name length
sorted_users = sorted(users, key=lambda u: len(u['name']))

# Sort by multiple keys
sorted_users = sorted(users, key=lambda u: (u['age'], u['name']))
```

---

### map(): Transform Each Element

**Pattern:** `map(function, iterable)`

```python
numbers = [1, 2, 3, 4, 5]

# Square each number
squared = list(map(lambda x: x**2, numbers))
# [1, 4, 9, 16, 25]

# Convert to strings
strings = list(map(str, numbers))
# ['1', '2', '3', '4', '5']

# Multiple iterables
a = [1, 2, 3]
b = [10, 20, 30]
sums = list(map(lambda x, y: x + y, a, b))
# [11, 22, 33]
```

**vs list comprehension:**
```python
# map
squared = list(map(lambda x: x**2, numbers))

# comprehension (more Pythonic)
squared = [x**2 for x in numbers]
```

---

### filter(): Keep Elements Matching Condition

**Pattern:** `filter(function, iterable)`

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Keep even numbers
evens = list(filter(lambda x: x % 2 == 0, numbers))
# [2, 4, 6, 8, 10]

# Keep numbers > 5
large = list(filter(lambda x: x > 5, numbers))
# [6, 7, 8, 9, 10]
```

**vs list comprehension:**
```python
# filter
evens = list(filter(lambda x: x % 2 == 0, numbers))

# comprehension (more Pythonic)
evens = [x for x in numbers if x % 2 == 0]
```

---

### reduce(): Aggregate to Single Value

```python
from functools import reduce

numbers = [1, 2, 3, 4, 5]

# Sum
total = reduce(lambda acc, x: acc + x, numbers)
# 15 (1+2+3+4+5)

# Product
product = reduce(lambda acc, x: acc * x, numbers)
# 120 (1*2*3*4*5)

# Max
maximum = reduce(lambda acc, x: max(acc, x), numbers)
# 5
```

**Note:** Use built-in functions when available:
```python
# Better than reduce
sum(numbers)
max(numbers)
```

---

## 7. File I/O {#file-io}

### Reading Files

```python
# Read entire file
with open('data.txt', 'r') as f:
    content = f.read()
    print(content)

# Read line by line (memory efficient)
with open('data.txt', 'r') as f:
    for line in f:
        print(line.strip())  # Remove \n

# Read all lines into list
with open('data.txt', 'r') as f:
    lines = f.readlines()  # ['line1\n', 'line2\n', ...]
```

**Why `with` statement?**
- Automatically closes file (even if error occurs)
- Better than manual `f.open()` / `f.close()`

---

### Writing Files

```python
# Write (overwrites existing file)
with open('output.txt', 'w') as f:
    f.write('Hello, World!\n')
    f.write('Second line\n')

# Append (adds to existing file)
with open('output.txt', 'a') as f:
    f.write('Appended line\n')

# Write multiple lines
lines = ['line1\n', 'line2\n', 'line3\n']
with open('output.txt', 'w') as f:
    f.writelines(lines)
```

---

### CSV Files

```python
import csv

# Read CSV
with open('data.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row['user_id'], row['amount'])

# Write CSV
data = [
    {'user_id': 1, 'name': 'Alice', 'amount': 100},
    {'user_id': 2, 'name': 'Bob', 'amount': 200}
]

with open('output.csv', 'w', newline='') as f:
    fieldnames = ['user_id', 'name', 'amount']
    writer = csv.DictWriter(f, fieldnames=fieldnames)

    writer.writeheader()
    writer.writerows(data)
```

---

### JSON Files

```python
import json

# Read JSON
with open('data.json', 'r') as f:
    data = json.load(f)
    print(data['users'][0]['name'])

# Write JSON
data = {
    'users': [
        {'name': 'Alice', 'age': 30},
        {'name': 'Bob', 'age': 25}
    ]
}

with open('output.json', 'w') as f:
    json.dump(data, f, indent=2)

# JSON string <-> Python object
json_string = json.dumps(data, indent=2)
parsed_data = json.loads(json_string)
```

---

## 8. Error Handling {#error-handling}

### Try-Except Basics

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero!")
    result = None

try:
    number = int("abc")
except ValueError:
    print("Invalid number format")
    number = 0
```

---

### Multiple Exceptions

```python
try:
    file = open('data.txt', 'r')
    data = json.load(file)
    value = data['key']
except FileNotFoundError:
    print("File not found")
except json.JSONDecodeError:
    print("Invalid JSON")
except KeyError:
    print("Key not found in JSON")
```

---

### Finally and Else

```python
try:
    file = open('data.txt', 'r')
    content = file.read()
except FileNotFoundError:
    print("File not found")
else:
    # Runs if no exception
    print("File read successfully")
finally:
    # Always runs (cleanup)
    if 'file' in locals():
        file.close()
```

---

## 9. Practice Problems {#practice-problems}

### Problem 1: "Activity Time Ledger" (Easy)

**Scenario:** Match activity start/end times

**Input:**
```python
events = [
    {'type': 'start', 'activity': 'coding', 'time': 10},
    {'type': 'end', 'activity': 'coding', 'time': 15},
    {'type': 'start', 'activity': 'meeting', 'time': 16},
    {'type': 'end', 'activity': 'meeting', 'time': 17},
]
```

**Task:** Calculate duration for each activity

**Hints:**
- Group by activity
- Track start/end times
- Calculate duration

---

### Problem 2: "Caesar Shift Check" (Easy)

**Task:** Check if string is a valid Caesar cipher shift

**Example:**
```python
is_caesar_shift("abc", "bcd")  # True (shift by 1)
is_caesar_shift("abc", "xyz")  # False
```

---

### Problem 3: "Letters in the Noise" (Easy)

**Task:** Find most frequent character in string (ignore spaces)

---

## 10. Solutions {#solutions}

### Solution 1: "Activity Time Ledger"

```python
from collections import defaultdict

def calculate_durations(events):
    """Calculate duration for each activity"""
    # Group events by activity
    activity_times = defaultdict(dict)

    for event in events:
        activity = event['activity']
        time = event['time']
        event_type = event['type']

        activity_times[activity][event_type] = time

    # Calculate durations
    durations = {}
    for activity, times in activity_times.items():
        if 'start' in times and 'end' in times:
            durations[activity] = times['end'] - times['start']

    return durations

# Test
events = [
    {'type': 'start', 'activity': 'coding', 'time': 10},
    {'type': 'end', 'activity': 'coding', 'time': 15},
    {'type': 'start', 'activity': 'meeting', 'time': 16},
    {'type': 'end', 'activity': 'meeting', 'time': 17},
]

print(calculate_durations(events))
# {'coding': 5, 'meeting': 1}
```

---

### Solution 2: "Caesar Shift Check"

```python
def is_caesar_shift(s1, s2):
    """Check if s2 is a Caesar shift of s1"""
    if len(s1) != len(s2):
        return False

    if not s1:
        return True

    # Calculate shift from first character
    shift = (ord(s2[0]) - ord(s1[0])) % 26

    # Check if all characters have same shift
    for c1, c2 in zip(s1, s2):
        expected = chr((ord(c1) - ord('a') + shift) % 26 + ord('a'))
        if c2 != expected:
            return False

    return True

# Test
print(is_caesar_shift("abc", "bcd"))  # True
print(is_caesar_shift("abc", "xyz"))  # False
print(is_caesar_shift("xyz", "abc"))  # True (shift by 3)
```

---

### Solution 3: "Letters in the Noise"

```python
from collections import Counter

def most_frequent_char(text):
    """Find most frequent character (excluding spaces)"""
    # Count characters (exclude spaces)
    counts = Counter(char for char in text.lower() if char != ' ')

    if not counts:
        return None

    # Get most common
    return counts.most_common(1)[0][0]

# Test
print(most_frequent_char("hello world"))  # 'l' (appears 3 times)
print(most_frequent_char("aabbcc"))       # 'a' (or 'b' or 'c', all tied)
```

---

## 11. Summary {#summary}

### Key Concepts Mastered

✅ **Data Structures:** Lists, tuples, sets, dicts
✅ **Collections:** Counter, defaultdict, deque, OrderedDict
✅ **Strings:** Manipulation, formatting, regex
✅ **Comprehensions:** List, dict, set comprehensions
✅ **Functional:** Lambda, map, filter, reduce
✅ **File I/O:** Reading/writing text, CSV, JSON
✅ **Error Handling:** Try-except-finally

### Self-Assessment

- [ ] Can use Counter for frequency counting?
- [ ] Comfortable with defaultdict for grouping?
- [ ] Know when to use list vs tuple vs set?
- [ ] Can write list/dict comprehensions?
- [ ] Understand regex basics?
- [ ] Can parse CSV and JSON files?

### Next Steps

**Chapter 5: Python Data Processing**

Topics:
- Pandas operations
- Session detection algorithms
- Grouping and aggregation patterns
- DataDriven medium Python problems

**Practice goals:**
- [ ] 20-25 easy Python problems
- [ ] Success rate: 85%+
- [ ] Average time: 15-20 min

**Congratulations on Python fundamentals!** 🎉

---
