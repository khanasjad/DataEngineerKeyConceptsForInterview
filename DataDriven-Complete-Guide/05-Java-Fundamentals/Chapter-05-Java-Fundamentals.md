# Chapter 5: Java Fundamentals for Data Engineering

**Duration:** 5-6 days | **Difficulty:** Beginner to Intermediate
**Problems to Solve:** 15-20 easy Java problems
**Prerequisites:** Basic Java syntax knowledge

---

## 📚 Table of Contents

1. [Introduction](#introduction)
2. [Collections Framework Deep Dive](#collections-framework)
3. [Streams API (Java 8+)](#streams-api)
4. [String Processing](#string-processing)
5. [File I/O](#file-io)
6. [Lambda Expressions and Functional Interfaces](#lambda-expressions)
7. [Exception Handling](#exception-handling)
8. [Date and Time API (Java 8+)](#datetime-api)
9. [Common Data Processing Patterns](#data-processing-patterns)
10. [Practice Problems](#practice-problems)
11. [Solutions](#solutions)
12. [Summary](#summary)

---

## 1. Introduction {#introduction}

### Why Java for Data Engineering?

Java is a **powerhouse** in data engineering:
- **Big Data:** Apache Spark, Hadoop, Kafka (all written in Java/Scala)
- **Stream Processing:** Apache Flink, Apache Storm
- **Enterprise ETL:** Talend, Informatica
- **Performance:** JVM optimization, multithreading
- **Ecosystem:** Massive library support

### What You'll Learn

Master Java fundamentals for data engineering:
- ✅ Collections Framework (ArrayList, HashMap, HashSet, TreeMap)
- ✅ Streams API for data transformation
- ✅ Lambda expressions and functional programming
- ✅ File I/O (CSV, JSON processing)
- ✅ String manipulation and regex
- ✅ Modern Java features (Java 8+)

### Java vs Python for Data Engineering

| Aspect | Java | Python |
|--------|------|--------|
| **Performance** | Faster (compiled) | Slower (interpreted) |
| **Type Safety** | Static typing (compile-time errors) | Dynamic typing |
| **Verbosity** | More verbose | Concise |
| **Use Case** | Large-scale systems, Spark jobs | Scripting, data analysis |
| **Learning Curve** | Steeper | Gentler |

**Best practice:** Use Java for production data pipelines, Python for prototyping.

---

## 2. Collections Framework Deep Dive {#collections-framework}

### Collection Hierarchy

```
Collection (interface)
├── List (interface)
│   ├── ArrayList (class)
│   ├── LinkedList (class)
│   └── Vector (class)
├── Set (interface)
│   ├── HashSet (class)
│   ├── LinkedHashSet (class)
│   └── TreeSet (class)
└── Queue (interface)
    ├── PriorityQueue (class)
    └── Deque (interface)
        └── ArrayDeque (class)

Map (interface - separate hierarchy)
├── HashMap (class)
├── LinkedHashMap (class)
└── TreeMap (class)
```

---

### List: Ordered Collections

#### ArrayList - Dynamic Array

**When to use:** Fast random access, frequent reads

```java
import java.util.*;

public class ArrayListExample {
    public static void main(String[] args) {
        // Creation
        List<String> names = new ArrayList<>();

        // Adding elements
        names.add("Alice");
        names.add("Bob");
        names.add("Charlie");

        // Access by index
        String first = names.get(0);  // "Alice"

        // Size
        int size = names.size();  // 3

        // Iteration
        for (String name : names) {
            System.out.println(name);
        }

        // Modern iteration (Java 8+)
        names.forEach(System.out::println);

        // Modification
        names.set(1, "Robert");  // Replace Bob with Robert
        names.remove(0);  // Remove first element
        names.remove("Charlie");  // Remove by value

        // Checking
        boolean hasAlice = names.contains("Alice");
        int index = names.indexOf("Robert");  // -1 if not found

        // Sorting
        Collections.sort(names);  // Ascending
        Collections.sort(names, Collections.reverseOrder());  // Descending

        // Sublist
        List<String> subList = names.subList(0, 2);  // [0, 2)

        // Convert to array
        String[] array = names.toArray(new String[0]);
    }
}
```

**Time Complexity:**
- Access: O(1)
- Search: O(n)
- Insert/Delete at end: O(1) amortized
- Insert/Delete at middle: O(n)

---

#### LinkedList - Doubly-Linked List

**When to use:** Frequent insertions/deletions, especially at beginning/end

```java
import java.util.*;

public class LinkedListExample {
    public static void main(String[] args) {
        LinkedList<Integer> list = new LinkedList<>();

        // Add to front/back
        list.addFirst(1);
        list.addLast(3);
        list.add(1, 2);  // Insert at index 1

        // Access first/last
        int first = list.getFirst();  // 1
        int last = list.getLast();    // 3

        // Remove first/last
        list.removeFirst();
        list.removeLast();

        // Use as Queue (FIFO)
        list.offer(1);  // Add to end
        list.offer(2);
        int head = list.poll();  // Remove from front, returns 1

        // Use as Stack (LIFO)
        list.push(1);  // Add to front
        list.push(2);
        int top = list.pop();  // Remove from front, returns 2
    }
}
```

**Time Complexity:**
- Access: O(n)
- Insert/Delete at ends: O(1)
- Insert/Delete at middle: O(n) to find + O(1) to insert

---

### Set: Unique Collections

#### HashSet - Unordered Unique Elements

**When to use:** Fast membership testing, no duplicates needed

```java
import java.util.*;

public class HashSetExample {
    public static void main(String[] args) {
        Set<String> users = new HashSet<>();

        // Adding elements (duplicates ignored)
        users.add("Alice");
        users.add("Bob");
        users.add("Alice");  // Ignored
        System.out.println(users.size());  // 2

        // Checking membership (O(1) average)
        boolean hasAlice = users.contains("Alice");  // true

        // Iteration (order not guaranteed)
        for (String user : users) {
            System.out.println(user);
        }

        // Set operations
        Set<Integer> set1 = new HashSet<>(Arrays.asList(1, 2, 3, 4));
        Set<Integer> set2 = new HashSet<>(Arrays.asList(3, 4, 5, 6));

        // Union
        Set<Integer> union = new HashSet<>(set1);
        union.addAll(set2);  // {1, 2, 3, 4, 5, 6}

        // Intersection
        Set<Integer> intersection = new HashSet<>(set1);
        intersection.retainAll(set2);  // {3, 4}

        // Difference (set1 - set2)
        Set<Integer> difference = new HashSet<>(set1);
        difference.removeAll(set2);  // {1, 2}
    }
}
```

**Example: Find unique users**
```java
public class UniqueUsers {
    public static void main(String[] args) {
        Set<Integer> users2023 = new HashSet<>(Arrays.asList(1, 2, 3, 4, 5));
        Set<Integer> users2024 = new HashSet<>(Arrays.asList(4, 5, 6, 7, 8));

        // New users (in 2024, not in 2023)
        Set<Integer> newUsers = new HashSet<>(users2024);
        newUsers.removeAll(users2023);  // {6, 7, 8}

        // Returning users (in both)
        Set<Integer> returning = new HashSet<>(users2023);
        returning.retainAll(users2024);  // {4, 5}

        // Churned users (in 2023, not in 2024)
        Set<Integer> churned = new HashSet<>(users2023);
        churned.removeAll(users2024);  // {1, 2, 3}

        System.out.println("New: " + newUsers);
        System.out.println("Returning: " + returning);
        System.out.println("Churned: " + churned);
    }
}
```

---

#### TreeSet - Sorted Unique Elements

**When to use:** Need sorted unique elements, range queries

```java
import java.util.*;

public class TreeSetExample {
    public static void main(String[] args) {
        TreeSet<Integer> numbers = new TreeSet<>();

        // Adding (automatically sorted)
        numbers.add(5);
        numbers.add(1);
        numbers.add(3);
        System.out.println(numbers);  // [1, 3, 5]

        // Range operations
        TreeSet<Integer> scores = new TreeSet<>(
            Arrays.asList(50, 60, 70, 80, 90, 100)
        );

        SortedSet<Integer> highScores = scores.tailSet(80);  // >= 80: [80, 90, 100]
        SortedSet<Integer> midScores = scores.subSet(60, 90);  // [60, 90): [60, 70, 80]

        // Navigation
        int first = scores.first();  // 50
        int last = scores.last();    // 100
        int lower = scores.lower(80);   // Strictly less than 80: 70
        int higher = scores.higher(80); // Strictly greater than 80: 90
        int floor = scores.floor(75);   // <= 75: 70
        int ceiling = scores.ceiling(75); // >= 75: 80
    }
}
```

**Time Complexity:** O(log n) for add, remove, contains

---

### Map: Key-Value Pairs

#### HashMap - Fast Lookup

**When to use:** Fast key-based access, counting, grouping

```java
import java.util.*;

public class HashMapExample {
    public static void main(String[] args) {
        Map<String, Integer> wordCount = new HashMap<>();

        // Put
        wordCount.put("apple", 3);
        wordCount.put("banana", 2);

        // Get
        Integer count = wordCount.get("apple");  // 3
        Integer missing = wordCount.get("grape");  // null

        // Get with default
        int appleCount = wordCount.getOrDefault("apple", 0);  // 3
        int grapeCount = wordCount.getOrDefault("grape", 0);  // 0

        // Check key
        boolean hasApple = wordCount.containsKey("apple");  // true
        boolean hasGrape = wordCount.containsKey("grape");  // false

        // Update
        wordCount.put("apple", 4);  // Replaces existing value

        // Compute if absent
        wordCount.putIfAbsent("cherry", 1);

        // Iteration
        for (Map.Entry<String, Integer> entry : wordCount.entrySet()) {
            System.out.println(entry.getKey() + ": " + entry.getValue());
        }

        // Java 8+ forEach
        wordCount.forEach((key, value) ->
            System.out.println(key + ": " + value)
        );

        // Keys and values
        Set<String> keys = wordCount.keySet();
        Collection<Integer> values = wordCount.values();
    }
}
```

**Counting Pattern:**
```java
public class WordCounter {
    public static Map<String, Integer> countWords(String[] words) {
        Map<String, Integer> counts = new HashMap<>();

        for (String word : words) {
            // Traditional way
            if (counts.containsKey(word)) {
                counts.put(word, counts.get(word) + 1);
            } else {
                counts.put(word, 1);
            }

            // Better: getOrDefault
            counts.put(word, counts.getOrDefault(word, 0) + 1);

            // Java 8+: merge
            counts.merge(word, 1, Integer::sum);
        }

        return counts;
    }

    public static void main(String[] args) {
        String[] words = {"apple", "banana", "apple", "cherry", "banana", "apple"};
        Map<String, Integer> counts = countWords(words);
        System.out.println(counts);  // {apple=3, banana=2, cherry=1}
    }
}
```

**Grouping Pattern:**
```java
public class Grouping {
    static class Transaction {
        String userId;
        double amount;

        Transaction(String userId, double amount) {
            this.userId = userId;
            this.amount = amount;
        }
    }

    public static Map<String, List<Double>> groupByUser(List<Transaction> transactions) {
        Map<String, List<Double>> grouped = new HashMap<>();

        for (Transaction txn : transactions) {
            grouped.putIfAbsent(txn.userId, new ArrayList<>());
            grouped.get(txn.userId).add(txn.amount);
        }

        return grouped;
    }

    public static void main(String[] args) {
        List<Transaction> transactions = Arrays.asList(
            new Transaction("Alice", 100),
            new Transaction("Bob", 200),
            new Transaction("Alice", 150)
        );

        Map<String, List<Double>> grouped = groupByUser(transactions);
        System.out.println(grouped);
        // {Alice=[100.0, 150.0], Bob=[200.0]}
    }
}
```

---

#### TreeMap - Sorted Map

**When to use:** Need sorted keys, range queries on keys

```java
import java.util.*;

public class TreeMapExample {
    public static void main(String[] args) {
        TreeMap<Integer, String> map = new TreeMap<>();

        // Put (automatically sorted by key)
        map.put(3, "three");
        map.put(1, "one");
        map.put(2, "two");

        System.out.println(map);  // {1=one, 2=two, 3=three}

        // Navigation
        Map.Entry<Integer, String> first = map.firstEntry();  // 1=one
        Map.Entry<Integer, String> last = map.lastEntry();    // 3=three

        Integer firstKey = map.firstKey();  // 1
        Integer lastKey = map.lastKey();    // 3

        // Range views
        SortedMap<Integer, String> subMap = map.subMap(1, 3);  // [1, 3): {1=one, 2=two}
        SortedMap<Integer, String> headMap = map.headMap(2);   // < 2: {1=one}
        SortedMap<Integer, String> tailMap = map.tailMap(2);   // >= 2: {2=two, 3=three}
    }
}
```

---

### Queue and Deque

#### PriorityQueue - Heap

**When to use:** Need min/max element efficiently

```java
import java.util.*;

public class PriorityQueueExample {
    public static void main(String[] args) {
        // Min heap (default)
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        minHeap.offer(5);
        minHeap.offer(1);
        minHeap.offer(3);

        System.out.println(minHeap.poll());  // 1 (smallest)
        System.out.println(minHeap.poll());  // 3
        System.out.println(minHeap.poll());  // 5

        // Max heap
        PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
        maxHeap.offer(5);
        maxHeap.offer(1);
        maxHeap.offer(3);

        System.out.println(maxHeap.poll());  // 5 (largest)

        // Custom comparator
        PriorityQueue<String> pq = new PriorityQueue<>((a, b) -> a.length() - b.length());
        pq.offer("apple");
        pq.offer("pie");
        pq.offer("banana");

        System.out.println(pq.poll());  // "pie" (shortest)
    }
}
```

**Example: Top K frequent elements**
```java
public class TopKFrequent {
    public static List<String> topKFrequent(String[] words, int k) {
        // Count frequencies
        Map<String, Integer> counts = new HashMap<>();
        for (String word : words) {
            counts.merge(word, 1, Integer::sum);
        }

        // Min heap of size k
        PriorityQueue<Map.Entry<String, Integer>> heap = new PriorityQueue<>(
            (a, b) -> a.getValue() - b.getValue()
        );

        for (Map.Entry<String, Integer> entry : counts.entrySet()) {
            heap.offer(entry);
            if (heap.size() > k) {
                heap.poll();  // Remove smallest
            }
        }

        // Extract results
        List<String> result = new ArrayList<>();
        while (!heap.isEmpty()) {
            result.add(0, heap.poll().getKey());  // Add to front (reverse order)
        }

        return result;
    }

    public static void main(String[] args) {
        String[] words = {"apple", "banana", "apple", "cherry", "banana", "apple"};
        List<String> top2 = topKFrequent(words, 2);
        System.out.println(top2);  // [apple, banana]
    }
}
```

---

#### ArrayDeque - Double-Ended Queue

**When to use:** Need efficient add/remove from both ends

```java
import java.util.*;

public class ArrayDequeExample {
    public static void main(String[] args) {
        Deque<Integer> deque = new ArrayDeque<>();

        // Add to both ends
        deque.addFirst(1);  // [1]
        deque.addLast(3);   // [1, 3]
        deque.addFirst(0);  // [0, 1, 3]

        // Remove from both ends
        int first = deque.removeFirst();  // 0
        int last = deque.removeLast();    // 3

        // Peek (don't remove)
        int peekFirst = deque.peekFirst();
        int peekLast = deque.peekLast();

        // Use as Stack
        deque.push(1);
        deque.push(2);
        int top = deque.pop();  // 2

        // Use as Queue
        deque.offer(1);
        deque.offer(2);
        int head = deque.poll();  // 1
    }
}
```

**Example: Sliding window maximum**
```java
public class SlidingWindowMax {
    public static int[] maxSlidingWindow(int[] nums, int k) {
        if (nums == null || nums.length == 0) return new int[0];

        int[] result = new int[nums.length - k + 1];
        Deque<Integer> deque = new ArrayDeque<>();  // Store indices

        for (int i = 0; i < nums.length; i++) {
            // Remove elements outside window
            while (!deque.isEmpty() && deque.peekFirst() <= i - k) {
                deque.pollFirst();
            }

            // Remove smaller elements (they'll never be max)
            while (!deque.isEmpty() && nums[deque.peekLast()] < nums[i]) {
                deque.pollLast();
            }

            deque.offerLast(i);

            // Add to result after first window complete
            if (i >= k - 1) {
                result[i - k + 1] = nums[deque.peekFirst()];
            }
        }

        return result;
    }

    public static void main(String[] args) {
        int[] nums = {1, 3, -1, -3, 5, 3, 6, 7};
        int[] result = maxSlidingWindow(nums, 3);
        System.out.println(Arrays.toString(result));  // [3, 3, 5, 5, 6, 7]
    }
}
```

---

## 3. Streams API (Java 8+) {#streams-api}

### Introduction to Streams

**What is a Stream?**
A sequence of elements supporting sequential and parallel operations.

**Key concepts:**
- **Lazy evaluation:** Operations only execute when needed
- **Functional style:** Chain operations with lambdas
- **Immutable:** Original collection unchanged
- **One-time use:** Stream consumed after terminal operation

### Creating Streams

```java
import java.util.stream.*;
import java.util.*;

public class StreamCreation {
    public static void main(String[] args) {
        // From collection
        List<String> list = Arrays.asList("a", "b", "c");
        Stream<String> stream1 = list.stream();

        // From array
        String[] array = {"a", "b", "c"};
        Stream<String> stream2 = Arrays.stream(array);

        // From values
        Stream<String> stream3 = Stream.of("a", "b", "c");

        // Infinite streams
        Stream<Integer> infiniteStream = Stream.iterate(0, n -> n + 1);
        Stream<Double> randomStream = Stream.generate(Math::random);

        // Range
        IntStream range = IntStream.range(0, 10);  // 0 to 9
        IntStream rangeClosed = IntStream.rangeClosed(0, 10);  // 0 to 10

        // Empty stream
        Stream<String> emptyStream = Stream.empty();
    }
}
```

---

### Intermediate Operations (Lazy)

#### filter() - Keep elements matching predicate

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Keep even numbers
List<Integer> evens = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
// [2, 4, 6, 8, 10]
```

---

#### map() - Transform each element

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// Convert to uppercase
List<String> upperNames = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
// [ALICE, BOB, CHARLIE]

// Get lengths
List<Integer> lengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList());
// [5, 3, 7]
```

---

#### flatMap() - Flatten nested structures

```java
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4),
    Arrays.asList(5, 6)
);

// Flatten to single list
List<Integer> flat = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
// [1, 2, 3, 4, 5, 6]
```

---

#### distinct() - Remove duplicates

```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3, 4);

List<Integer> unique = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
// [1, 2, 3, 4]
```

---

#### sorted() - Sort elements

```java
List<Integer> numbers = Arrays.asList(3, 1, 4, 1, 5, 9, 2);

// Natural order
List<Integer> sorted = numbers.stream()
    .sorted()
    .collect(Collectors.toList());
// [1, 1, 2, 3, 4, 5, 9]

// Custom comparator
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
List<String> byLength = names.stream()
    .sorted((a, b) -> a.length() - b.length())
    .collect(Collectors.toList());
// [Bob, Alice, Charlie]
```

---

#### limit() and skip()

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// First 5
List<Integer> first5 = numbers.stream()
    .limit(5)
    .collect(Collectors.toList());
// [1, 2, 3, 4, 5]

// Skip first 5, take next 3
List<Integer> middle = numbers.stream()
    .skip(5)
    .limit(3)
    .collect(Collectors.toList());
// [6, 7, 8]
```

---

### Terminal Operations (Eager)

#### collect() - Accumulate to collection

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// To List
List<String> list = names.stream()
    .collect(Collectors.toList());

// To Set
Set<String> set = names.stream()
    .collect(Collectors.toSet());

// To specific collection
ArrayList<String> arrayList = names.stream()
    .collect(Collectors.toCollection(ArrayList::new));
```

---

#### forEach() - Perform action on each element

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

names.stream()
    .forEach(System.out::println);

// With index (use IntStream)
IntStream.range(0, names.size())
    .forEach(i -> System.out.println(i + ": " + names.get(i)));
```

---

#### reduce() - Aggregate to single value

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Sum
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);
// Or: .reduce(0, Integer::sum);

// Product
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);

// Max
Optional<Integer> max = numbers.stream()
    .reduce(Integer::max);

// String concatenation
List<String> words = Arrays.asList("Hello", "World", "Java");
String sentence = words.stream()
    .reduce("", (a, b) -> a + " " + b).trim();
// "Hello World Java"
```

---

#### count(), min(), max()

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

long count = numbers.stream().count();  // 5

Optional<Integer> min = numbers.stream().min(Integer::compareTo);
Optional<Integer> max = numbers.stream().max(Integer::compareTo);
```

---

#### anyMatch(), allMatch(), noneMatch()

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0);  // true

boolean allPositive = numbers.stream()
    .allMatch(n -> n > 0);  // true

boolean noneNegative = numbers.stream()
    .noneMatch(n -> n < 0);  // true
```

---

#### findFirst(), findAny()

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

Optional<Integer> first = numbers.stream()
    .filter(n -> n > 3)
    .findFirst();  // Optional[4]

Optional<Integer> any = numbers.stream()
    .filter(n -> n > 3)
    .findAny();  // Optional[4] (or 5, non-deterministic in parallel)
```

---

### Advanced Collectors

#### Grouping

```java
class Transaction {
    String userId;
    double amount;

    Transaction(String userId, double amount) {
        this.userId = userId;
        this.amount = amount;
    }

    String getUserId() { return userId; }
    double getAmount() { return amount; }
}

List<Transaction> transactions = Arrays.asList(
    new Transaction("Alice", 100),
    new Transaction("Bob", 200),
    new Transaction("Alice", 150),
    new Transaction("Bob", 50)
);

// Group by user
Map<String, List<Transaction>> byUser = transactions.stream()
    .collect(Collectors.groupingBy(Transaction::getUserId));

// Group and count
Map<String, Long> countByUser = transactions.stream()
    .collect(Collectors.groupingBy(
        Transaction::getUserId,
        Collectors.counting()
    ));

// Group and sum amounts
Map<String, Double> sumByUser = transactions.stream()
    .collect(Collectors.groupingBy(
        Transaction::getUserId,
        Collectors.summingDouble(Transaction::getAmount)
    ));
```

---

#### Partitioning

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Partition into even and odd
Map<Boolean, List<Integer>> partitioned = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));

List<Integer> evens = partitioned.get(true);   // [2, 4, 6, 8, 10]
List<Integer> odds = partitioned.get(false);   // [1, 3, 5, 7, 9]
```

---

#### Joining

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

String joined = names.stream()
    .collect(Collectors.joining());  // "AliceBobCharlie"

String withComma = names.stream()
    .collect(Collectors.joining(", "));  // "Alice, Bob, Charlie"

String withPrefixSuffix = names.stream()
    .collect(Collectors.joining(", ", "[", "]"));  // "[Alice, Bob, Charlie]"
```

---

### Practical Examples

**Example 1: Count word frequency**
```java
public class WordFrequency {
    public static Map<String, Long> countWords(String text) {
        return Arrays.stream(text.toLowerCase().split("\\s+"))
            .collect(Collectors.groupingBy(
                word -> word,
                Collectors.counting()
            ));
    }

    public static void main(String[] args) {
        String text = "hello world hello java world";
        Map<String, Long> freq = countWords(text);
        System.out.println(freq);  // {hello=2, world=2, java=1}
    }
}
```

**Example 2: Filter and transform data**
```java
class User {
    String name;
    int age;
    String city;

    User(String name, int age, String city) {
        this.name = name;
        this.age = age;
        this.city = city;
    }

    // Getters
    String getName() { return name; }
    int getAge() { return age; }
    String getCity() { return city; }
}

public class UserAnalysis {
    public static void main(String[] args) {
        List<User> users = Arrays.asList(
            new User("Alice", 25, "NYC"),
            new User("Bob", 35, "LA"),
            new User("Charlie", 30, "NYC"),
            new User("David", 40, "LA")
        );

        // Users in NYC, sorted by age
        List<String> nycUsers = users.stream()
            .filter(u -> u.getCity().equals("NYC"))
            .sorted((a, b) -> a.getAge() - b.getAge())
            .map(User::getName)
            .collect(Collectors.toList());

        System.out.println(nycUsers);  // [Alice, Charlie]

        // Average age by city
        Map<String, Double> avgAgeByCity = users.stream()
            .collect(Collectors.groupingBy(
                User::getCity,
                Collectors.averagingInt(User::getAge)
            ));

        System.out.println(avgAgeByCity);  // {NYC=27.5, LA=37.5}
    }
}
```

---

## 4. String Processing {#string-processing}

### Basic String Operations

```java
public class StringBasics {
    public static void main(String[] args) {
        String text = "Hello, World!";

        // Length
        int length = text.length();  // 13

        // Character access
        char first = text.charAt(0);  // 'H'
        char last = text.charAt(text.length() - 1);  // '!'

        // Substring
        String sub1 = text.substring(0, 5);  // "Hello"
        String sub2 = text.substring(7);     // "World!"

        // Case conversion
        String upper = text.toUpperCase();  // "HELLO, WORLD!"
        String lower = text.toLowerCase();  // "hello, world!"

        // Checking
        boolean startsWith = text.startsWith("Hello");  // true
        boolean endsWith = text.endsWith("!");          // true
        boolean contains = text.contains("World");      // true

        // Whitespace
        String spaces = "  hello  ";
        String trimmed = spaces.trim();  // "hello"

        // Replace
        String replaced = text.replace("World", "Java");  // "Hello, Java!"
        String removed = text.replace(",", "");           // "Hello World!"

        // Split
        String csv = "apple,banana,cherry";
        String[] fruits = csv.split(",");  // ["apple", "banana", "cherry"]

        // Join (Java 8+)
        String joined = String.join(", ", fruits);  // "apple, banana, cherry"

        // Comparison
        boolean equals = text.equals("Hello, World!");  // true
        boolean equalsIgnoreCase = text.equalsIgnoreCase("hello, world!");  // true
        int compareTo = text.compareTo("Hello");  // Positive (comes after)
    }
}
```

---

### StringBuilder - Efficient String Building

**When to use:** Building strings in loops (avoid String concatenation)

```java
public class StringBuilderExample {
    public static void main(String[] args) {
        // Bad: String concatenation in loop (O(n²))
        String result = "";
        for (int i = 0; i < 1000; i++) {
            result += i;  // Creates new String each time!
        }

        // Good: StringBuilder (O(n))
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 1000; i++) {
            sb.append(i);
        }
        String result2 = sb.toString();

        // StringBuilder methods
        StringBuilder builder = new StringBuilder();
        builder.append("Hello");
        builder.append(" ");
        builder.append("World");

        builder.insert(6, "Beautiful ");  // "Hello Beautiful World"
        builder.delete(6, 16);             // "Hello World"
        builder.reverse();                 // "dlroW olleH"

        String str = builder.toString();
    }
}
```

---

### Regular Expressions

```java
import java.util.regex.*;

public class RegexExample {
    public static void main(String[] args) {
        String text = "My email is alice@example.com";

        // Find match
        Pattern pattern = Pattern.compile("\\w+@\\w+\\.\\w+");
        Matcher matcher = pattern.matcher(text);

        if (matcher.find()) {
            System.out.println("Email: " + matcher.group());  // alice@example.com
        }

        // Find all matches
        text = "Emails: alice@ex.com, bob@ex.com";
        matcher = pattern.matcher(text);
        while (matcher.find()) {
            System.out.println(matcher.group());
        }

        // Replace
        text = "Price: $100.50";
        String cleaned = text.replaceAll("[$,]", "");  // "Price: 100.50"

        // Split
        text = "apple, banana; cherry";
        String[] fruits = text.split("[,;]\\s*");  // ["apple", "banana", "cherry"]

        // Extract groups
        text = "Date: 2024-01-15";
        pattern = Pattern.compile("(\\d{4})-(\\d{2})-(\\d{2})");
        matcher = pattern.matcher(text);

        if (matcher.find()) {
            String year = matcher.group(1);   // "2024"
            String month = matcher.group(2);  // "01"
            String day = matcher.group(3);    // "15"
        }

        // Validation
        String email = "user@example.com";
        boolean isValid = email.matches("^[\\w._%+-]+@[\\w.-]+\\.[A-Za-z]{2,}$");
    }
}
```

**Common regex patterns:**
```java
// Email
"^[\\w._%+-]+@[\\w.-]+\\.[A-Za-z]{2,}$"

// Phone (###-###-####)
"^\\d{3}-\\d{3}-\\d{4}$"

// URL
"https?://[^\\s]+"

// IP address
"^(\\d{1,3}\\.){3}\\d{1,3}$"

// Date (YYYY-MM-DD)
"^\\d{4}-\\d{2}-\\d{2}$"
```

---

### String Formatting

```java
public class StringFormatting {
    public static void main(String[] args) {
        String name = "Alice";
        int age = 30;
        double price = 1234.567;

        // String.format (like printf)
        String formatted = String.format("Name: %s, Age: %d", name, age);

        // Numbers
        String decimal = String.format("%.2f", price);  // "1234.57"
        String withComma = String.format("%,.2f", price);  // "1,234.57"

        // Padding
        String padded = String.format("%5d", 42);  // "   42"
        String zeroPadded = String.format("%05d", 42);  // "00042"

        // Left align
        String leftAlign = String.format("%-10s", "hello");  // "hello     "

        // System.out.printf (prints directly)
        System.out.printf("Name: %s, Age: %d%n", name, age);
    }
}
```

---

## 5. File I/O {#file-io}

### Reading Files

```java
import java.io.*;
import java.nio.file.*;
import java.util.*;

public class FileReading {
    // Java 7+: Files.readAllLines
    public static List<String> readAllLines(String filename) throws IOException {
        return Files.readAllLines(Paths.get(filename));
    }

    // Read entire file as string
    public static String readAsString(String filename) throws IOException {
        return new String(Files.readAllBytes(Paths.get(filename)));
    }

    // Line by line (memory efficient for large files)
    public static void readLineByLine(String filename) throws IOException {
        try (BufferedReader reader = new BufferedReader(new FileReader(filename))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        }
    }

    // Java 8+: Stream API
    public static void readWithStream(String filename) throws IOException {
        try (Stream<String> lines = Files.lines(Paths.get(filename))) {
            lines.forEach(System.out::println);
        }
    }

    public static void main(String[] args) {
        try {
            List<String> lines = readAllLines("data.txt");
            System.out.println("Read " + lines.size() + " lines");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

### Writing Files

```java
import java.io.*;
import java.nio.file.*;
import java.util.*;

public class FileWriting {
    // Write lines
    public static void writeLines(String filename, List<String> lines) throws IOException {
        Files.write(Paths.get(filename), lines);
    }

    // Append to file
    public static void appendLines(String filename, List<String> lines) throws IOException {
        Files.write(
            Paths.get(filename),
            lines,
            StandardOpenOption.CREATE,
            StandardOpenOption.APPEND
        );
    }

    // BufferedWriter (for large files)
    public static void writeWithBuffer(String filename, List<String> lines) throws IOException {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(filename))) {
            for (String line : lines) {
                writer.write(line);
                writer.newLine();
            }
        }
    }

    public static void main(String[] args) {
        try {
            List<String> lines = Arrays.asList("Line 1", "Line 2", "Line 3");
            writeLines("output.txt", lines);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

### CSV Processing

```java
import java.io.*;
import java.util.*;

public class CSVProcessor {
    static class User {
        int id;
        String name;
        int age;

        User(int id, String name, int age) {
            this.id = id;
            this.name = name;
            this.age = age;
        }

        String toCSV() {
            return id + "," + name + "," + age;
        }
    }

    // Read CSV
    public static List<User> readCSV(String filename) throws IOException {
        List<User> users = new ArrayList<>();

        try (BufferedReader reader = new BufferedReader(new FileReader(filename))) {
            String line = reader.readLine();  // Skip header

            while ((line = reader.readLine()) != null) {
                String[] parts = line.split(",");
                int id = Integer.parseInt(parts[0]);
                String name = parts[1];
                int age = Integer.parseInt(parts[2]);
                users.add(new User(id, name, age));
            }
        }

        return users;
    }

    // Write CSV
    public static void writeCSV(String filename, List<User> users) throws IOException {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(filename))) {
            // Header
            writer.write("id,name,age");
            writer.newLine();

            // Data
            for (User user : users) {
                writer.write(user.toCSV());
                writer.newLine();
            }
        }
    }

    public static void main(String[] args) {
        try {
            List<User> users = Arrays.asList(
                new User(1, "Alice", 30),
                new User(2, "Bob", 25)
            );

            writeCSV("users.csv", users);
            List<User> readUsers = readCSV("users.csv");
            System.out.println("Read " + readUsers.size() + " users");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

### JSON Processing (with Gson)

**Add dependency:**
```xml
<!-- Maven -->
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.10.1</version>
</dependency>
```

```java
import com.google.gson.*;
import java.io.*;
import java.util.*;

public class JSONProcessor {
    static class User {
        int id;
        String name;
        int age;

        User(int id, String name, int age) {
            this.id = id;
            this.name = name;
            this.age = age;
        }
    }

    public static void main(String[] args) {
        Gson gson = new Gson();

        // Object to JSON string
        User user = new User(1, "Alice", 30);
        String json = gson.toJson(user);
        System.out.println(json);  // {"id":1,"name":"Alice","age":30}

        // JSON string to object
        String jsonStr = "{\"id\":1,\"name\":\"Alice\",\"age\":30}";
        User parsed = gson.fromJson(jsonStr, User.class);

        // List to JSON
        List<User> users = Arrays.asList(
            new User(1, "Alice", 30),
            new User(2, "Bob", 25)
        );
        String jsonList = gson.toJson(users);

        // JSON to list
        User[] usersArray = gson.fromJson(jsonList, User[].class);

        // Pretty print
        Gson prettyGson = new GsonBuilder().setPrettyPrinting().create();
        String prettyJson = prettyGson.toJson(users);
        System.out.println(prettyJson);

        // Write to file
        try (Writer writer = new FileWriter("users.json")) {
            gson.toJson(users, writer);
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Read from file
        try (Reader reader = new FileReader("users.json")) {
            User[] fromFile = gson.fromJson(reader, User[].class);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

## 6. Lambda Expressions and Functional Interfaces {#lambda-expressions}

### Lambda Syntax

```java
// No parameters
() -> System.out.println("Hello")

// One parameter (parentheses optional)
x -> x * x
(x) -> x * x

// Multiple parameters
(x, y) -> x + y

// Multiple statements (need braces and return)
(x, y) -> {
    int sum = x + y;
    return sum;
}
```

---

### Common Functional Interfaces

```java
import java.util.function.*;

public class FunctionalInterfaces {
    public static void main(String[] args) {
        // Predicate<T> - boolean test(T t)
        Predicate<Integer> isEven = n -> n % 2 == 0;
        System.out.println(isEven.test(4));  // true

        // Function<T, R> - R apply(T t)
        Function<String, Integer> length = s -> s.length();
        System.out.println(length.apply("hello"));  // 5

        // Consumer<T> - void accept(T t)
        Consumer<String> print = s -> System.out.println(s);
        print.accept("Hello");

        // Supplier<T> - T get()
        Supplier<Double> random = () -> Math.random();
        System.out.println(random.get());

        // BiFunction<T, U, R> - R apply(T t, U u)
        BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
        System.out.println(add.apply(2, 3));  // 5

        // UnaryOperator<T> - T apply(T t) (special Function)
        UnaryOperator<Integer> square = x -> x * x;
        System.out.println(square.apply(5));  // 25

        // BinaryOperator<T> - T apply(T t1, T t2) (special BiFunction)
        BinaryOperator<Integer> multiply = (a, b) -> a * b;
        System.out.println(multiply.apply(3, 4));  // 12
    }
}
```

---

### Method References

```java
import java.util.*;

public class MethodReferences {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

        // Static method reference
        names.forEach(System.out::println);
        // Equivalent to: names.forEach(s -> System.out.println(s));

        // Instance method reference
        names.stream()
            .map(String::toUpperCase)
            .forEach(System.out::println);
        // Equivalent to: .map(s -> s.toUpperCase())

        // Constructor reference
        List<Integer> lengths = names.stream()
            .map(String::length)
            .collect(ArrayList::new, ArrayList::add, ArrayList::addAll);
    }

    // Custom method reference
    public static int customCompare(String a, String b) {
        return a.length() - b.length();
    }

    public static void sortExample() {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

        // Method reference
        names.sort(MethodReferences::customCompare);
    }
}
```

---

## 7. Exception Handling {#exception-handling}

### Try-Catch Basics

```java
public class ExceptionHandling {
    public static void main(String[] args) {
        // Basic try-catch
        try {
            int result = 10 / 0;
        } catch (ArithmeticException e) {
            System.out.println("Cannot divide by zero");
        }

        // Multiple catch blocks
        try {
            String text = null;
            int length = text.length();  // NullPointerException
            int number = Integer.parseInt("abc");  // NumberFormatException
        } catch (NullPointerException e) {
            System.out.println("Null pointer: " + e.getMessage());
        } catch (NumberFormatException e) {
            System.out.println("Invalid number format");
        } catch (Exception e) {
            System.out.println("General exception: " + e);
        }

        // Multi-catch (Java 7+)
        try {
            // Code that might throw IOException or SQLException
        } catch (IOException | SQLException e) {
            System.out.println("IO or SQL exception: " + e);
        }
    }
}
```

---

### Finally and Try-with-Resources

```java
import java.io.*;

public class FinallyExample {
    // Traditional try-finally
    public static void readFileTraditional(String filename) {
        BufferedReader reader = null;
        try {
            reader = new BufferedReader(new FileReader(filename));
            String line = reader.readLine();
            System.out.println(line);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // Always executed (cleanup)
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    // Try-with-resources (Java 7+) - Better!
    public static void readFileModern(String filename) {
        try (BufferedReader reader = new BufferedReader(new FileReader(filename))) {
            String line = reader.readLine();
            System.out.println(line);
        } catch (IOException e) {
            e.printStackTrace();
        }
        // reader.close() automatically called
    }

    // Multiple resources
    public static void copyFile(String src, String dest) {
        try (BufferedReader reader = new BufferedReader(new FileReader(src));
             BufferedWriter writer = new BufferedWriter(new FileWriter(dest))) {

            String line;
            while ((line = reader.readLine()) != null) {
                writer.write(line);
                writer.newLine();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

### Checked vs Unchecked Exceptions

```java
import java.io.*;

public class ExceptionTypes {
    // Checked exception (must handle or declare)
    public static void checkedExample() throws IOException {
        FileReader reader = new FileReader("file.txt");  // Checked
        // Must either catch or declare throws
    }

    // Unchecked exception (runtime exception)
    public static void uncheckedExample() {
        int result = 10 / 0;  // ArithmeticException (unchecked)
        // No need to catch or declare
    }

    // Custom exception
    static class InvalidUserException extends Exception {
        public InvalidUserException(String message) {
            super(message);
        }
    }

    public static void validateUser(String username) throws InvalidUserException {
        if (username == null || username.isEmpty()) {
            throw new InvalidUserException("Username cannot be empty");
        }
    }
}
```

---

## 8. Date and Time API (Java 8+) {#datetime-api}

### LocalDate, LocalTime, LocalDateTime

```java
import java.time.*;
import java.time.format.*;

public class DateTimeExample {
    public static void main(String[] args) {
        // Current date and time
        LocalDate today = LocalDate.now();
        LocalTime now = LocalTime.now();
        LocalDateTime dateTime = LocalDateTime.now();

        System.out.println("Today: " + today);  // 2024-01-15
        System.out.println("Now: " + now);      // 10:30:45.123
        System.out.println("DateTime: " + dateTime);

        // Create specific date
        LocalDate date = LocalDate.of(2024, 1, 15);
        LocalDate date2 = LocalDate.of(2024, Month.JANUARY, 15);

        // Create from string
        LocalDate parsed = LocalDate.parse("2024-01-15");

        // Extract components
        int year = date.getYear();
        Month month = date.getMonth();
        int monthValue = date.getMonthValue();
        int day = date.getDayOfMonth();
        DayOfWeek dayOfWeek = date.getDayOfWeek();

        // Date arithmetic
        LocalDate tomorrow = today.plusDays(1);
        LocalDate nextWeek = today.plusWeeks(1);
        LocalDate nextMonth = today.plusMonths(1);
        LocalDate nextYear = today.plusYears(1);

        LocalDate yesterday = today.minusDays(1);

        // Comparison
        boolean isBefore = date.isBefore(today);
        boolean isAfter = date.isAfter(today);
        boolean isEqual = date.equals(today);

        // Period (date-based)
        Period period = Period.between(date, today);
        int days = period.getDays();
        int months = period.getMonths();
        int years = period.getYears();
    }
}
```

---

### Duration and Period

```java
import java.time.*;

public class DurationExample {
    public static void main(String[] args) {
        // Period (date-based)
        LocalDate start = LocalDate.of(2024, 1, 1);
        LocalDate end = LocalDate.of(2024, 12, 31);

        Period period = Period.between(start, end);
        System.out.println("Days: " + period.getDays());
        System.out.println("Months: " + period.getMonths());

        // Duration (time-based)
        LocalDateTime startTime = LocalDateTime.of(2024, 1, 1, 10, 0);
        LocalDateTime endTime = LocalDateTime.of(2024, 1, 1, 15, 30);

        Duration duration = Duration.between(startTime, endTime);
        System.out.println("Hours: " + duration.toHours());
        System.out.println("Minutes: " + duration.toMinutes());
        System.out.println("Seconds: " + duration.getSeconds());

        // Create duration
        Duration twoHours = Duration.ofHours(2);
        Duration thirtyMinutes = Duration.ofMinutes(30);
    }
}
```

---

### Formatting and Parsing

```java
import java.time.*;
import java.time.format.*;

public class DateFormatting {
    public static void main(String[] args) {
        LocalDateTime dateTime = LocalDateTime.now();

        // Built-in formatters
        String iso = dateTime.format(DateTimeFormatter.ISO_DATE_TIME);

        // Custom format
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        String formatted = dateTime.format(formatter);
        System.out.println(formatted);  // 2024-01-15 10:30:45

        // Parse
        LocalDateTime parsed = LocalDateTime.parse("2024-01-15 10:30:45", formatter);

        // Common patterns
        DateTimeFormatter fmt1 = DateTimeFormatter.ofPattern("MM/dd/yyyy");
        DateTimeFormatter fmt2 = DateTimeFormatter.ofPattern("dd-MMM-yyyy");
        DateTimeFormatter fmt3 = DateTimeFormatter.ofPattern("EEEE, MMMM dd, yyyy");
    }
}
```

---

## 9. Common Data Processing Patterns {#data-processing-patterns}

### Pattern 1: Frequency Counting

```java
import java.util.*;

public class FrequencyCounter {
    // Count word frequency
    public static Map<String, Integer> countWords(String[] words) {
        Map<String, Integer> counts = new HashMap<>();

        for (String word : words) {
            counts.merge(word, 1, Integer::sum);
        }

        return counts;
    }

    // With Streams
    public static Map<String, Long> countWordsStream(String[] words) {
        return Arrays.stream(words)
            .collect(Collectors.groupingBy(
                word -> word,
                Collectors.counting()
            ));
    }

    public static void main(String[] args) {
        String[] words = {"apple", "banana", "apple", "cherry", "banana", "apple"};

        Map<String, Integer> counts = countWords(words);
        System.out.println(counts);  // {apple=3, banana=2, cherry=1}

        // Find most frequent
        String mostFrequent = counts.entrySet().stream()
            .max(Map.Entry.comparingByValue())
            .map(Map.Entry::getKey)
            .orElse(null);

        System.out.println("Most frequent: " + mostFrequent);  // apple
    }
}
```

---

### Pattern 2: Grouping and Aggregation

```java
import java.util.*;
import java.util.stream.*;

public class GroupingExample {
    static class Transaction {
        String userId;
        double amount;
        String date;

        Transaction(String userId, double amount, String date) {
            this.userId = userId;
            this.amount = amount;
            this.date = date;
        }

        String getUserId() { return userId; }
        double getAmount() { return amount; }
        String getDate() { return date; }
    }

    public static void main(String[] args) {
        List<Transaction> transactions = Arrays.asList(
            new Transaction("Alice", 100, "2024-01-01"),
            new Transaction("Bob", 200, "2024-01-01"),
            new Transaction("Alice", 150, "2024-01-02"),
            new Transaction("Bob", 50, "2024-01-02")
        );

        // Group by user
        Map<String, List<Transaction>> byUser = transactions.stream()
            .collect(Collectors.groupingBy(Transaction::getUserId));

        // Sum by user
        Map<String, Double> sumByUser = transactions.stream()
            .collect(Collectors.groupingBy(
                Transaction::getUserId,
                Collectors.summingDouble(Transaction::getAmount)
            ));

        System.out.println(sumByUser);  // {Alice=250.0, Bob=250.0}

        // Group by date, then by user
        Map<String, Map<String, Double>> byDateThenUser = transactions.stream()
            .collect(Collectors.groupingBy(
                Transaction::getDate,
                Collectors.groupingBy(
                    Transaction::getUserId,
                    Collectors.summingDouble(Transaction::getAmount)
                )
            ));

        System.out.println(byDateThenUser);
        // {2024-01-01={Alice=100.0, Bob=200.0}, 2024-01-02={Alice=150.0, Bob=50.0}}
    }
}
```

---

### Pattern 3: Finding Top N

```java
import java.util.*;
import java.util.stream.*;

public class TopNExample {
    // Find top N by value
    public static List<Map.Entry<String, Integer>> topN(
        Map<String, Integer> map, int n
    ) {
        return map.entrySet().stream()
            .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
            .limit(n)
            .collect(Collectors.toList());
    }

    public static void main(String[] args) {
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Alice", 95);
        scores.put("Bob", 87);
        scores.put("Charlie", 92);
        scores.put("David", 88);
        scores.put("Eve", 90);

        List<Map.Entry<String, Integer>> top3 = topN(scores, 3);

        System.out.println("Top 3:");
        top3.forEach(e ->
            System.out.println(e.getKey() + ": " + e.getValue())
        );
        // Alice: 95
        // Charlie: 92
        // Eve: 90
    }
}
```

---

## 10. Practice Problems {#practice-problems}

### Problem 1: "Activity Time Ledger" (Easy)

Match activity start/end events and calculate durations.

**Input:**
```java
List<Event> events = Arrays.asList(
    new Event("start", "coding", 10),
    new Event("end", "coding", 15),
    new Event("start", "meeting", 16),
    new Event("end", "meeting", 17)
);
```

**Task:** Return map of activity → duration

---

### Problem 2: "Letters in the Noise" (Easy)

Find most frequent character in string (ignore spaces).

---

### Problem 3: "Caesar Shift Check" (Easy)

Check if string2 is a Caesar shift of string1.

---

## 11. Solutions {#solutions}

### Solution 1: "Activity Time Ledger"

```java
import java.util.*;

public class ActivityLedger {
    static class Event {
        String type;
        String activity;
        int time;

        Event(String type, String activity, int time) {
            this.type = type;
            this.activity = activity;
            this.time = time;
        }
    }

    public static Map<String, Integer> calculateDurations(List<Event> events) {
        Map<String, Map<String, Integer>> activityTimes = new HashMap<>();

        for (Event event : events) {
            activityTimes.putIfAbsent(event.activity, new HashMap<>());
            activityTimes.get(event.activity).put(event.type, event.time);
        }

        Map<String, Integer> durations = new HashMap<>();
        for (Map.Entry<String, Map<String, Integer>> entry : activityTimes.entrySet()) {
            String activity = entry.getKey();
            Map<String, Integer> times = entry.getValue();

            if (times.containsKey("start") && times.containsKey("end")) {
                int duration = times.get("end") - times.get("start");
                durations.put(activity, duration);
            }
        }

        return durations;
    }

    public static void main(String[] args) {
        List<Event> events = Arrays.asList(
            new Event("start", "coding", 10),
            new Event("end", "coding", 15),
            new Event("start", "meeting", 16),
            new Event("end", "meeting", 17)
        );

        Map<String, Integer> durations = calculateDurations(events);
        System.out.println(durations);  // {coding=5, meeting=1}
    }
}
```

---

### Solution 2: "Letters in the Noise"

```java
import java.util.*;

public class MostFrequentChar {
    public static char findMostFrequent(String text) {
        Map<Character, Integer> counts = new HashMap<>();

        for (char c : text.toCharArray()) {
            if (c != ' ') {
                counts.merge(c, 1, Integer::sum);
            }
        }

        return counts.entrySet().stream()
            .max(Map.Entry.comparingByValue())
            .map(Map.Entry::getKey)
            .orElse('\0');
    }

    public static void main(String[] args) {
        String text = "hello world";
        char result = findMostFrequent(text);
        System.out.println("Most frequent: " + result);  // 'l' (appears 3 times)
    }
}
```

---

### Solution 3: "Caesar Shift Check"

```java
public class CaesarShift {
    public static boolean isCaesarShift(String s1, String s2) {
        if (s1.length() != s2.length()) return false;
        if (s1.isEmpty()) return true;

        // Calculate shift from first character
        int shift = (s2.charAt(0) - s1.charAt(0) + 26) % 26;

        // Check if all characters have same shift
        for (int i = 0; i < s1.length(); i++) {
            int expected = (s1.charAt(i) - 'a' + shift) % 26 + 'a';
            if (s2.charAt(i) != expected) {
                return false;
            }
        }

        return true;
    }

    public static void main(String[] args) {
        System.out.println(isCaesarShift("abc", "bcd"));  // true
        System.out.println(isCaesarShift("abc", "xyz"));  // false
        System.out.println(isCaesarShift("xyz", "abc"));  // true (shift by 3)
    }
}
```

---

## 12. Summary {#summary}

### Key Concepts Mastered

✅ **Collections:** ArrayList, HashMap, HashSet, TreeMap, PriorityQueue, ArrayDeque
✅ **Streams API:** filter, map, reduce, collect, grouping
✅ **Lambda Expressions:** Functional interfaces, method references
✅ **Strings:** StringBuilder, regex, formatting
✅ **File I/O:** BufferedReader/Writer, CSV, JSON
✅ **Date/Time:** LocalDate, Duration, formatting
✅ **Patterns:** Counting, grouping, top N

### Java vs Python Summary

| Feature | Java | Python |
|---------|------|--------|
| **Syntax** | Verbose, static types | Concise, dynamic types |
| **Collections** | ArrayList, HashMap | list, dict |
| **Streams** | .stream().filter() | list comprehensions |
| **File I/O** | BufferedReader | with open() |
| **Performance** | Faster | Slower |
| **Use Case** | Production pipelines | Scripts, analysis |

### Self-Assessment

- [ ] Comfortable with Collections Framework?
- [ ] Can write Stream pipelines?
- [ ] Understand lambda expressions?
- [ ] Can process CSV/JSON files?
- [ ] Know when to use which collection?

### Next Steps

**Practice on DataDriven.io:**
- 20-25 easy Java problems
- Focus on: collections, streams, string processing
- Average time: 20-25 min per problem
- Success rate: 80%+

**Then:** Chapter 6 (Python Data Processing) or start coding!

---

**Congratulations on mastering Java fundamentals for data engineering!** 🎉

---
