# Java - Core Java & Data Engineering

**Master Java fundamentals, collections, multithreading, and data engineering tools**

---

## 📚 What's in This Folder

### **1. 100-QUESTIONS.md**
Comprehensive Q&A covering:
- Core Java fundamentals (OOP, inheritance, polymorphism)
- Collections Framework (List, Set, Map, Queue)
- Multithreading and Concurrency
- Java 8+ features (Streams, Lambda, Optional)
- Exception handling and best practices
- JDBC and database connectivity
- Design patterns
- Spring Framework basics
- JVM internals and garbage collection
- Data engineering with Java (Spark, Kafka)

### **2. CHEATSHEET.md**
Quick reference with:
- One-line definitions
- Common patterns and idioms
- Collection operations
- Stream API examples
- Multithreading patterns
- Interview talking points

---

## 🎯 What You'll Learn

### **Core Java**
- **OOP Principles**: Encapsulation, Inheritance, Polymorphism, Abstraction
- **Classes & Objects**: Constructors, methods, access modifiers
- **Interfaces & Abstract Classes**: When to use each
- **Exception Handling**: try-catch, custom exceptions, best practices
- **Generics**: Type-safe collections and methods

### **Collections Framework**
- **List**: ArrayList, LinkedList, Vector
- **Set**: HashSet, TreeSet, LinkedHashSet
- **Map**: HashMap, TreeMap, LinkedHashMap, ConcurrentHashMap
- **Queue**: PriorityQueue, ArrayDeque, LinkedList
- **Performance**: Time complexity of operations

### **Java 8+ Features**
- **Lambda Expressions**: Functional interfaces, method references
- **Stream API**: map, filter, reduce, collect
- **Optional**: Handling null safely
- **Date/Time API**: LocalDate, LocalDateTime, ZonedDateTime
- **Default Methods**: Interface evolution

### **Multithreading**
- **Thread Creation**: Thread class, Runnable interface
- **Synchronization**: synchronized, locks, volatile
- **Concurrent Collections**: ConcurrentHashMap, CopyOnWriteArrayList
- **Executors**: ThreadPoolExecutor, ScheduledExecutorService
- **CompletableFuture**: Async programming

### **JDBC & Database**
- **Connection Management**: DriverManager, DataSource
- **Statement Types**: Statement, PreparedStatement, CallableStatement
- **Result Processing**: ResultSet navigation
- **Transaction Management**: commit, rollback
- **Connection Pooling**: HikariCP, Apache DBCP

### **Advanced Topics**
- **JVM Internals**: Heap, Stack, Metaspace
- **Garbage Collection**: Types of GC, tuning
- **Design Patterns**: Singleton, Factory, Builder, Observer
- **Spring Framework**: Dependency Injection, Spring Boot
- **Testing**: JUnit, Mockito

---

## 💼 Why Java Matters for Data Engineers

### **Industry Adoption**
- **#1 language** for enterprise data systems
- **Hadoop ecosystem**: Spark, Kafka, Flink all support Java
- **Spring Batch**: Enterprise batch processing
- **Stable & performant**: JVM optimization for long-running jobs

### **Key Use Cases**
- **Apache Spark**: Process large-scale data in Java
- **Apache Kafka**: Build streaming data pipelines
- **Spring Batch**: Complex ETL workflows
- **JDBC**: Database connectivity for all pipelines
- **Microservices**: RESTful APIs for data serving

### **Interview Focus**
1. Collections and their use cases
2. Multithreading and concurrency
3. Java 8 Stream API
4. JDBC and connection management
5. Exception handling best practices

---

## 🚀 Quick Start Guide

### **1. Review Cheatsheet (20 minutes)**
Review `CHEATSHEET.md` for quick overview of syntax and patterns.

### **2. Study 100 Questions (6-8 hours)**
- **Q1-Q20:** Core Java (OOP, classes, interfaces)
- **Q21-Q40:** Collections Framework
- **Q41-Q60:** Multithreading and concurrency
- **Q61-Q80:** Java 8+ features (Streams, Lambda, Optional)
- **Q81-Q100:** JDBC, design patterns, Spring basics

### **3. Hands-On Practice**

**Setup:**
```bash
# Install Java (if not already)
brew install openjdk@17  # macOS
# or download from https://adoptium.net/

# Verify installation
java -version
javac -version

# Setup IDE (IntelliJ IDEA Community - free)
brew install --cask intellij-idea-ce
```

**Practice Projects:**
```java
// 1. Collections practice
public class CollectionsPractice {
    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        names.add("Alice");
        names.add("Bob");
        
        // Stream operations
        names.stream()
            .filter(name -> name.startsWith("A"))
            .forEach(System.out::println);
    }
}

// 2. Multithreading practice
public class ThreadPractice {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(4);
        
        for (int i = 0; i < 10; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println("Task " + taskId + " executed by " + 
                    Thread.currentThread().getName());
            });
        }
        
        executor.shutdown();
    }
}

// 3. JDBC practice
public class JDBCPractice {
    public static void main(String[] args) {
        String url = "jdbc:postgresql://localhost:5432/mydb";
        String sql = "SELECT * FROM users WHERE age > ?";
        
        try (Connection conn = DriverManager.getConnection(url, "user", "pass");
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setInt(1, 18);
            ResultSet rs = pstmt.executeQuery();
            
            while (rs.next()) {
                System.out.println(rs.getString("name"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

---

## 📖 Topic Coverage

### **Collections Framework Hierarchy**

```
Collection (Interface)
├── List (Interface)
│   ├── ArrayList (Class) - Fast random access, slow insert/delete
│   ├── LinkedList (Class) - Fast insert/delete, slow random access
│   └── Vector (Class) - Synchronized ArrayList (legacy)
│
├── Set (Interface)
│   ├── HashSet (Class) - No order, O(1) operations
│   ├── LinkedHashSet (Class) - Insertion order maintained
│   └── TreeSet (Class) - Sorted order, O(log n) operations
│
└── Queue (Interface)
    ├── PriorityQueue (Class) - Heap-based priority queue
    ├── ArrayDeque (Class) - Resizable array deque
    └── LinkedList (Class) - Also implements Queue

Map (Interface) - Not part of Collection
├── HashMap (Class) - No order, O(1) operations
├── LinkedHashMap (Class) - Insertion/access order
├── TreeMap (Class) - Sorted by keys, O(log n) operations
└── ConcurrentHashMap (Class) - Thread-safe, no locking
```

---

### **Java 8 Stream API**

**Common Operations:**
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Filter: Get even numbers
List<Integer> evens = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

// Map: Square each number
List<Integer> squares = numbers.stream()
    .map(n -> n * n)
    .collect(Collectors.toList());

// Reduce: Sum all numbers
int sum = numbers.stream()
    .reduce(0, Integer::sum);

// Complex pipeline
double average = numbers.stream()
    .filter(n -> n > 5)
    .mapToInt(Integer::intValue)
    .average()
    .orElse(0.0);

// Grouping
Map<Boolean, List<Integer>> partitioned = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));
```

---

### **Multithreading Patterns**

**1. ExecutorService:**
```java
ExecutorService executor = Executors.newFixedThreadPool(4);

// Submit tasks
Future<Integer> future = executor.submit(() -> {
    // Computation
    return 42;
});

// Get result
Integer result = future.get(); // Blocks until complete

executor.shutdown();
```

**2. CompletableFuture:**
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // Async computation
    return "Hello";
}).thenApply(s -> s + " World")
  .thenAccept(System.out::println);
```

**3. Synchronized Access:**
```java
public class Counter {
    private int count = 0;
    
    // Method synchronization
    public synchronized void increment() {
        count++;
    }
    
    // Block synchronization
    public void decrement() {
        synchronized(this) {
            count--;
        }
    }
}
```

---

### **JDBC Best Practices**

**1. Use try-with-resources:**
```java
try (Connection conn = DriverManager.getConnection(url);
     PreparedStatement pstmt = conn.prepareStatement(sql);
     ResultSet rs = pstmt.executeQuery()) {
    
    while (rs.next()) {
        // Process results
    }
} // Auto-closes resources
```

**2. Use PreparedStatement:**
```java
// BAD: SQL Injection risk
String sql = "SELECT * FROM users WHERE name = '" + userName + "'";

// GOOD: Safe from injection
String sql = "SELECT * FROM users WHERE name = ?";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setString(1, userName);
```

**3. Batch Operations:**
```java
PreparedStatement pstmt = conn.prepareStatement(
    "INSERT INTO users (name, age) VALUES (?, ?)"
);

for (User user : users) {
    pstmt.setString(1, user.getName());
    pstmt.setInt(2, user.getAge());
    pstmt.addBatch();
}

pstmt.executeBatch(); // Execute all at once
```

---

## 🎓 Interview Preparation

### **Week 1: Core Java**
- Study Q1-Q30
- Focus on OOP principles
- Practice inheritance and polymorphism
- Understand exception handling

### **Week 2: Collections**
- Study Q31-Q50
- Master List, Set, Map
- Understand time complexity
- Practice Stream API

### **Week 3: Multithreading**
- Study Q51-Q70
- Thread creation and synchronization
- Executors and thread pools
- Concurrent collections

### **Week 4: Advanced Topics**
- Study Q71-Q100
- JDBC and database connectivity
- Design patterns
- Spring Framework basics
- Mock interviews

---

## 💡 Common Interview Questions

### **Conceptual**

**Q: "ArrayList vs LinkedList?"**
- **ArrayList**: Dynamic array, fast random access O(1), slow insert/delete O(n)
  - Use for: Frequent reads, rare modifications
- **LinkedList**: Doubly-linked list, slow random access O(n), fast insert/delete O(1)
  - Use for: Frequent insert/delete at ends, queue/deque

**Q: "HashMap vs TreeMap vs LinkedHashMap?"**
- **HashMap**: No order, O(1) operations, allows one null key
- **TreeMap**: Sorted by keys, O(log n) operations, no null keys
- **LinkedHashMap**: Insertion order, O(1) operations

**Q: "== vs equals()?"**
- **==**: Compares references (memory addresses)
- **equals()**: Compares content (if overridden properly)

```java
String s1 = new String("hello");
String s2 = new String("hello");

s1 == s2;        // false (different objects)
s1.equals(s2);   // true (same content)
```

**Q: "When to use synchronized?"**
Use when multiple threads access shared mutable data:
```java
// Shared counter needs synchronization
public synchronized void increment() {
    count++; // count is shared, mutable
}
```

---

### **Coding Questions**

**Q: "Remove duplicates from ArrayList"**
```java
// Solution 1: Using Set
List<Integer> list = Arrays.asList(1, 2, 2, 3, 3, 4);
List<Integer> unique = new ArrayList<>(new HashSet<>(list));

// Solution 2: Using Stream
List<Integer> unique = list.stream()
    .distinct()
    .collect(Collectors.toList());
```

**Q: "Find second highest number"**
```java
List<Integer> numbers = Arrays.asList(5, 2, 8, 2, 9, 1);

Integer secondHighest = numbers.stream()
    .distinct()
    .sorted(Comparator.reverseOrder())
    .skip(1)
    .findFirst()
    .orElse(null);
```

**Q: "Group employees by department"**
```java
List<Employee> employees = getEmployees();

Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));
```

---

## 🔗 Resources

### **Official Documentation**
- [Oracle Java Documentation](https://docs.oracle.com/en/java/)
- [Java SE API](https://docs.oracle.com/en/java/javase/17/docs/api/)
- [Java Tutorials](https://docs.oracle.com/javase/tutorial/)

### **Books**
- **"Effective Java"** by Joshua Bloch (Best practices)
- **"Java Concurrency in Practice"** by Brian Goetz
- **"Head First Java"** by Kathy Sierra (Beginner-friendly)

### **Practice**
- [LeetCode](https://leetcode.com/) - Java coding problems
- [HackerRank](https://www.hackerrank.com/domains/java) - Java challenges
- [CodingBat](https://codingbat.com/java) - Java practice problems

---

## 🎯 Key Takeaways

### **Must Know**
✅ Collections (List, Set, Map) and when to use each
✅ Multithreading basics (synchronized, ExecutorService)
✅ Java 8 Stream API (map, filter, reduce)
✅ Exception handling (try-catch-finally, custom exceptions)
✅ JDBC (PreparedStatement, connection management)

### **Should Know**
✅ Generics and type safety
✅ Optional for null handling
✅ CompletableFuture for async programming
✅ Design patterns (Singleton, Factory, Builder)
✅ JVM memory management (Heap, Stack, GC)

### **Nice to Know**
✅ Spring Framework (DI, Spring Boot)
✅ Functional interfaces and method references
✅ Reflection API
✅ Serialization
✅ Advanced GC tuning

### **Red Flags**
❌ Not using generics (raw types)
❌ Ignoring thread safety in concurrent code
❌ SQL injection vulnerabilities (not using PreparedStatement)
❌ Not closing resources (forgetting try-with-resources)
❌ Using == for String comparison

---

## 📝 Data Engineering with Java

### **Apache Spark (Java API)**
```java
SparkSession spark = SparkSession.builder()
    .appName("DataProcessing")
    .master("local[*]")
    .getOrCreate();

Dataset<Row> df = spark.read()
    .format("csv")
    .option("header", "true")
    .load("data.csv");

// Transformations
Dataset<Row> filtered = df.filter("age > 25");
Dataset<Row> grouped = df.groupBy("city").count();

// Save
filtered.write()
    .format("parquet")
    .mode("overwrite")
    .save("output/");
```

### **Apache Kafka (Java Client)**
```java
// Producer
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);

ProducerRecord<String, String> record = 
    new ProducerRecord<>("my-topic", "key", "value");
producer.send(record);

// Consumer
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("group.id", "my-group");
props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("my-topic"));

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        System.out.println(record.value());
    }
}
```

---

**Master Java for data engineering success! ☕**

*For detailed answers and examples, see 100-QUESTIONS.md*
*For quick review before interviews, see CHEATSHEET.md*
