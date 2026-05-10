# Java Cheatsheet - Quick Reference

**One-line definitions and quick examples for Java concepts**

---

## Core Java Fundamentals

- **Class**: Blueprint for creating objects (template)
- **Object**: Instance of a class (actual entity)
- **Constructor**: Special method to initialize objects
- **this**: Reference to current object
- **static**: Belongs to class, not instance
- **final**: Cannot be changed (constant, immutable)
- **abstract**: Incomplete, must be implemented by subclass
- **interface**: Contract defining behavior (all methods abstract by default)

---

## OOP Principles

- **Encapsulation**: Data hiding using private fields + public getters/setters
- **Inheritance**: Child class inherits parent's properties (`extends`)
- **Polymorphism**: One interface, multiple implementations (method overriding)
- **Abstraction**: Hiding implementation details, showing only functionality

---

## Collections Framework

### List (Ordered, allows duplicates)
```java
List<String> list = new ArrayList<>();          // Fast read O(1), slow write O(n)
List<String> list = new LinkedList<>();         // Fast write O(1), slow read O(n)
List<String> list = new Vector<>();             // Synchronized ArrayList (legacy)
```

### Set (Unordered, no duplicates)
```java
Set<String> set = new HashSet<>();              // No order, O(1) operations
Set<String> set = new LinkedHashSet<>();        // Insertion order maintained
Set<String> set = new TreeSet<>();              // Sorted, O(log n) operations
```

### Map (Key-value pairs)
```java
Map<K, V> map = new HashMap<>();                // No order, O(1), allows null
Map<K, V> map = new LinkedHashMap<>();          // Insertion order
Map<K, V> map = new TreeMap<>();                // Sorted by keys, O(log n)
Map<K, V> map = new ConcurrentHashMap<>();      // Thread-safe, no locking
```

### Queue (FIFO operations)
```java
Queue<String> queue = new LinkedList<>();       // Standard queue
Queue<String> queue = new PriorityQueue<>();    // Heap-based priority queue
Deque<String> deque = new ArrayDeque<>();       // Double-ended queue
```

---

## String Operations

```java
String s = "Hello";
s.length()                  // 5
s.charAt(0)                 // 'H'
s.substring(1, 4)           // "ell"
s.indexOf("lo")             // 3
s.toLowerCase()             // "hello"
s.toUpperCase()             // "HELLO"
s.replace("l", "L")         // "HeLLo"
s.split(",")                // String[]
s.trim()                    // Remove leading/trailing whitespace
s.equals("hello")           // false (content comparison)
s.equalsIgnoreCase("hello") // true
s.startsWith("He")          // true
s.endsWith("lo")            // true
s.contains("ell")           // true
```

---

## Java 8+ Stream API

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Filter
numbers.stream().filter(n -> n % 2 == 0).collect(Collectors.toList());

// Map
numbers.stream().map(n -> n * 2).collect(Collectors.toList());

// Reduce
numbers.stream().reduce(0, Integer::sum);

// Find
numbers.stream().findFirst().orElse(0);
numbers.stream().findAny().orElse(0);

// Match
numbers.stream().anyMatch(n -> n > 3);      // true if any element > 3
numbers.stream().allMatch(n -> n > 0);      // true if all elements > 0
numbers.stream().noneMatch(n -> n < 0);     // true if no element < 0

// Count
numbers.stream().count();

// Sort
numbers.stream().sorted().collect(Collectors.toList());
numbers.stream().sorted(Comparator.reverseOrder()).collect(Collectors.toList());

// Distinct
numbers.stream().distinct().collect(Collectors.toList());

// Skip & Limit
numbers.stream().skip(2).limit(2).collect(Collectors.toList());

// FlatMap (flatten nested collections)
List<List<Integer>> nested = Arrays.asList(Arrays.asList(1, 2), Arrays.asList(3, 4));
nested.stream().flatMap(List::stream).collect(Collectors.toList());

// Grouping
Map<Boolean, List<Integer>> partitioned = 
    numbers.stream().collect(Collectors.partitioningBy(n -> n % 2 == 0));

// Collectors
numbers.stream().collect(Collectors.toList());
numbers.stream().collect(Collectors.toSet());
numbers.stream().collect(Collectors.joining(", "));
numbers.stream().collect(Collectors.summingInt(Integer::intValue));
numbers.stream().collect(Collectors.averagingInt(Integer::intValue));
```

---

## Lambda Expressions

```java
// Runnable
Runnable r = () -> System.out.println("Hello");

// Comparator
Comparator<String> comp = (s1, s2) -> s1.compareTo(s2);

// Functional Interface
Function<Integer, Integer> square = x -> x * x;
Predicate<Integer> isEven = x -> x % 2 == 0;
Consumer<String> print = s -> System.out.println(s);
Supplier<Double> random = () -> Math.random();

// Method Reference
list.forEach(System.out::println);                // Instance method
list.sort(String::compareTo);                     // Instance method
list.stream().map(Integer::parseInt);             // Static method
list.stream().map(String::new);                   // Constructor
```

---

## Optional (Null Handling)

```java
Optional<String> optional = Optional.of("value");
Optional<String> empty = Optional.empty();
Optional<String> nullable = Optional.ofNullable(value);

// Check presence
optional.isPresent();       // true if value exists
optional.isEmpty();         // true if value is absent

// Get value
optional.get();             // Returns value or throws NoSuchElementException
optional.orElse("default"); // Returns value or default
optional.orElseGet(() -> "computed default");
optional.orElseThrow(() -> new Exception("No value"));

// Transform
optional.map(String::toUpperCase);
optional.flatMap(s -> Optional.of(s.length()));
optional.filter(s -> s.length() > 5);

// Action if present
optional.ifPresent(System.out::println);
optional.ifPresentOrElse(
    System.out::println,
    () -> System.out.println("Empty")
);
```

---

## Exception Handling

```java
// Try-catch
try {
    // Risky code
} catch (IOException e) {
    // Handle IOException
} catch (Exception e) {
    // Handle all other exceptions
} finally {
    // Always executes (cleanup)
}

// Try-with-resources (auto-close)
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line = br.readLine();
} catch (IOException e) {
    e.printStackTrace();
}

// Throw exception
throw new IllegalArgumentException("Invalid argument");

// Custom exception
public class CustomException extends Exception {
    public CustomException(String message) {
        super(message);
    }
}
```

---

## Multithreading

### Thread Creation
```java
// Method 1: Extend Thread
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread running");
    }
}
new MyThread().start();

// Method 2: Implement Runnable
Runnable task = () -> System.out.println("Task running");
new Thread(task).start();

// Method 3: ExecutorService
ExecutorService executor = Executors.newFixedThreadPool(4);
executor.submit(() -> System.out.println("Task"));
executor.shutdown();
```

### Synchronization
```java
// Synchronized method
public synchronized void increment() {
    count++;
}

// Synchronized block
synchronized(lock) {
    count++;
}

// Volatile (visibility guarantee, no atomicity)
private volatile boolean flag = false;

// ReentrantLock
private final Lock lock = new ReentrantLock();
lock.lock();
try {
    // Critical section
} finally {
    lock.unlock();
}
```

### Concurrent Collections
```java
ConcurrentHashMap<K, V> map = new ConcurrentHashMap<>();
CopyOnWriteArrayList<T> list = new CopyOnWriteArrayList<>();
BlockingQueue<T> queue = new LinkedBlockingQueue<>();
```

### CompletableFuture
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return "Result";
});

future.thenApply(s -> s.toUpperCase())
      .thenAccept(System.out::println)
      .exceptionally(ex -> {
          System.err.println("Error: " + ex.getMessage());
          return null;
      });
```

---

## JDBC

```java
// Load driver (automatic in modern JDBC)
Class.forName("org.postgresql.Driver");

// Connect
Connection conn = DriverManager.getConnection(
    "jdbc:postgresql://localhost:5432/mydb",
    "user",
    "password"
);

// Query (read)
String sql = "SELECT * FROM users WHERE age > ?";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setInt(1, 18);
ResultSet rs = pstmt.executeQuery();

while (rs.next()) {
    String name = rs.getString("name");
    int age = rs.getInt("age");
}

// Insert/Update/Delete
String sql = "INSERT INTO users (name, age) VALUES (?, ?)";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setString(1, "Alice");
pstmt.setInt(2, 25);
int rowsAffected = pstmt.executeUpdate();

// Batch operations
for (User user : users) {
    pstmt.setString(1, user.getName());
    pstmt.setInt(2, user.getAge());
    pstmt.addBatch();
}
pstmt.executeBatch();

// Transaction
conn.setAutoCommit(false);
try {
    // Multiple operations
    pstmt1.executeUpdate();
    pstmt2.executeUpdate();
    conn.commit();
} catch (SQLException e) {
    conn.rollback();
    throw e;
}

// Always close resources (or use try-with-resources)
rs.close();
pstmt.close();
conn.close();
```

---

## File I/O

```java
// Read file (Java 11+)
String content = Files.readString(Path.of("file.txt"));
List<String> lines = Files.readAllLines(Path.of("file.txt"));

// Write file
Files.writeString(Path.of("file.txt"), "content");
Files.write(Path.of("file.txt"), lines);

// BufferedReader (traditional)
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
}

// BufferedWriter
try (BufferedWriter bw = new BufferedWriter(new FileWriter("file.txt"))) {
    bw.write("Hello World");
    bw.newLine();
}
```

---

## Date & Time (Java 8+)

```java
// LocalDate (date without time)
LocalDate today = LocalDate.now();
LocalDate date = LocalDate.of(2024, 1, 15);
LocalDate tomorrow = today.plusDays(1);
LocalDate lastWeek = today.minusWeeks(1);

// LocalDateTime (date + time)
LocalDateTime now = LocalDateTime.now();
LocalDateTime dt = LocalDateTime.of(2024, 1, 15, 14, 30);

// ZonedDateTime (date + time + timezone)
ZonedDateTime zdt = ZonedDateTime.now(ZoneId.of("America/New_York"));

// Formatting
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String formatted = now.format(formatter);
LocalDateTime parsed = LocalDateTime.parse("2024-01-15 14:30:00", formatter);

// Period (date-based)
Period period = Period.between(LocalDate.of(2024, 1, 1), LocalDate.now());
int days = period.getDays();

// Duration (time-based)
Duration duration = Duration.between(
    LocalDateTime.of(2024, 1, 1, 10, 0),
    LocalDateTime.now()
);
long hours = duration.toHours();
```

---

## Common Design Patterns

### Singleton
```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

### Builder
```java
public class User {
    private final String name;
    private final int age;
    
    private User(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
    }
    
    public static class Builder {
        private String name;
        private int age;
        
        public Builder name(String name) {
            this.name = name;
            return this;
        }
        
        public Builder age(int age) {
            this.age = age;
            return this;
        }
        
        public User build() {
            return new User(this);
        }
    }
}

// Usage
User user = new User.Builder()
    .name("Alice")
    .age(25)
    .build();
```

### Factory
```java
public interface Shape {
    void draw();
}

public class ShapeFactory {
    public static Shape createShape(String type) {
        switch (type) {
            case "circle": return new Circle();
            case "square": return new Square();
            default: throw new IllegalArgumentException("Unknown shape");
        }
    }
}
```

---

## Common Idioms

### Check null safely
```java
// BAD
if (obj != null && obj.getValue() != null) { ... }

// GOOD (Optional)
Optional.ofNullable(obj)
    .map(Object::getValue)
    .ifPresent(value -> { ... });
```

### Compare objects
```java
// Override equals() and hashCode()
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    User user = (User) o;
    return age == user.age && Objects.equals(name, user.name);
}

@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```

### Immutable class
```java
public final class ImmutableUser {
    private final String name;
    private final int age;
    
    public ImmutableUser(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
}
```

---

## JVM Memory

- **Heap**: Objects, instance variables (shared across threads)
- **Stack**: Local variables, method calls (per thread)
- **Metaspace**: Class metadata (Java 8+, replaces PermGen)
- **Code Cache**: JIT-compiled native code

---

## Garbage Collection Types

- **Serial GC**: Single-threaded, small apps
- **Parallel GC**: Multi-threaded, throughput-focused
- **CMS (Concurrent Mark Sweep)**: Low pause time (deprecated)
- **G1 GC**: Balanced, default in Java 9+ (recommended)
- **ZGC**: Ultra-low latency (<10ms pauses)

---

## Time Complexity

### ArrayList
- **get(index)**: O(1)
- **add(element)**: O(1) amortized
- **add(index, element)**: O(n)
- **remove(index)**: O(n)
- **contains(element)**: O(n)

### LinkedList
- **get(index)**: O(n)
- **add(element)**: O(1)
- **add(index, element)**: O(n)
- **remove(index)**: O(n)
- **addFirst/addLast**: O(1)

### HashMap
- **get(key)**: O(1) average
- **put(key, value)**: O(1) average
- **remove(key)**: O(1) average
- **containsKey**: O(1) average

### TreeMap
- **get(key)**: O(log n)
- **put(key, value)**: O(log n)
- **remove(key)**: O(log n)

---

## Interview Quick Tips

✅ **Always use generics** (`List<String>` not `List`)
✅ **Use StringBuilder** for string concatenation in loops
✅ **Close resources** with try-with-resources
✅ **Use PreparedStatement** to prevent SQL injection
✅ **Override equals() and hashCode()** together
✅ **Prefer composition** over inheritance
✅ **Make fields private**, provide getters/setters
✅ **Use interface types** for variables (`List list = new ArrayList()`)
✅ **Handle exceptions** appropriately (don't swallow)
✅ **Make classes immutable** when possible

❌ **Don't compare String with ==** (use equals())
❌ **Don't catch Exception** (too broad, catch specific)
❌ **Don't ignore thread safety** in concurrent code
❌ **Don't forget to close** JDBC resources
❌ **Don't use raw types** (`List` without generics)

---

**Master these concepts for Java interviews! ☕**
