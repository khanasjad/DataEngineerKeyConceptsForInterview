# Java - 100 Interview Questions

**Comprehensive Q&A for Data Engineering Interviews**

---

## Core Java Fundamentals (Q1-Q20)

### Q1: What is the difference between JDK, JRE, and JVM?

**Answer:**
- **JVM (Java Virtual Machine)**: Runtime environment that executes Java bytecode. Platform-dependent.
- **JRE (Java Runtime Environment)**: JVM + libraries needed to run Java applications. No compiler.
- **JDK (Java Development Kit)**: JRE + development tools (compiler, debugger). Needed for development.

**Analogy:** JVM is the engine, JRE is the car, JDK is the car factory.

---

### Q2: Explain the four pillars of OOP in Java

**Answer:**

**1. Encapsulation**: Data hiding using private fields + public getters/setters
```java
public class Account {
    private double balance;  // Hidden

    public double getBalance() { return balance; }
    public void deposit(double amount) { balance += amount; }
}
```

**2. Inheritance**: Child class inherits parent properties
```java
class Animal { void eat() {} }
class Dog extends Animal { void bark() {} }
```

**3. Polymorphism**: One interface, multiple implementations
```java
Animal a = new Dog();  // Runtime polymorphism
a.eat();  // Calls Dog's version if overridden
```

**4. Abstraction**: Hiding implementation, showing only functionality
```java
abstract class Shape {
    abstract double area();  // No implementation
}
```

---

### Q3: What is the difference between abstract class and interface?

**Answer:**

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| Methods | Can have both abstract and concrete | All abstract (before Java 8) |
| Variables | Any type | public static final only |
| Inheritance | Single (extends one) | Multiple (implements many) |
| Constructor | Can have | Cannot have |
| Access Modifiers | Any | public only (before Java 9) |
| When to use | "is-a" relationship | "can-do" relationship |

**Example:**
```java
// Abstract class: "Dog IS-A Animal"
abstract class Animal {
    String name;
    abstract void makeSound();
    void sleep() { System.out.println("Sleeping..."); }
}

// Interface: "Dog CAN-DO Swim"
interface Swimmable {
    void swim();
}

class Dog extends Animal implements Swimmable {
    void makeSound() { System.out.println("Bark"); }
    public void swim() { System.out.println("Swimming"); }
}
```

---

### Q4: What is the difference between == and equals()?

**Answer:**
- **==**: Compares references (memory addresses)
- **equals()**: Compares content (if properly overridden)

```java
String s1 = new String("hello");
String s2 = new String("hello");

s1 == s2;        // false (different objects)
s1.equals(s2);   // true (same content)

// String pool exception
String s3 = "hello";
String s4 = "hello";
s3 == s4;        // true (same reference in pool)
```

**For custom classes:**
```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    User user = (User) o;
    return age == user.age && Objects.equals(name, user.name);
}
```

---

### Q5: What is the purpose of hashCode() and why must it be overridden with equals()?

**Answer:**
**hashCode()** returns an integer used by hash-based collections (HashMap, HashSet).

**Contract:** If two objects are equal (equals() returns true), they MUST have the same hashCode.

```java
@Override
public int hashCode() {
    return Objects.hash(name, age);
}

// Why it matters:
User u1 = new User("Alice", 25);
User u2 = new User("Alice", 25);

// Without proper hashCode:
Set<User> set = new HashSet<>();
set.add(u1);
set.add(u2);  // Would add duplicate! (different hashCode)

// With proper hashCode:
// u1.hashCode() == u2.hashCode(), so HashSet recognizes duplicate
```

**Rule:** Always override both equals() and hashCode() together.

---

### Q6: What is the difference between String, StringBuilder, and StringBuffer?

**Answer:**

| Feature | String | StringBuilder | StringBuffer |
|---------|--------|---------------|--------------|
| Mutability | Immutable | Mutable | Mutable |
| Thread-safe | Yes | No | Yes (synchronized) |
| Performance | Slow (creates new object) | Fast | Slower than StringBuilder |
| When to use | Fixed strings | Single-threaded concat | Multi-threaded concat |

```java
// String (creates 3 objects!)
String s = "Hello";
s = s + " World";  // New object created
s = s + "!";       // Another new object

// StringBuilder (modifies same object)
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");  // Same object
sb.append("!");       // Same object
String result = sb.toString();

// Rule: Use StringBuilder in loops
for (int i = 0; i < 1000; i++) {
    s += i;  // BAD: Creates 1000 objects
    sb.append(i);  // GOOD: Modifies 1 object
}
```

---

### Q7: What is the difference between final, finally, and finalize()?

**Answer:**

**1. final**: Keyword for constants
```java
final int MAX = 100;  // Cannot change
final class Math {}  // Cannot extend
final void process() {}  // Cannot override
```

**2. finally**: Block that always executes (cleanup)
```java
try {
    // Risky code
} catch (Exception e) {
    // Handle error
} finally {
    // ALWAYS runs (close resources)
    connection.close();
}
```

**3. finalize()**: Method called by GC before object destruction (deprecated)
```java
@Override
protected void finalize() throws Throwable {
    // Cleanup before GC
    // DON'T USE: Use try-with-resources instead
}
```

---

### Q8: What is the difference between checked and unchecked exceptions?

**Answer:**

**Checked Exceptions**: Must be caught or declared (compile-time)
```java
// IOException, SQLException, FileNotFoundException
public void readFile() throws IOException {  // Must declare
    FileReader fr = new FileReader("file.txt");
}

// Or catch
try {
    FileReader fr = new FileReader("file.txt");
} catch (IOException e) {
    e.printStackTrace();
}
```

**Unchecked Exceptions**: Don't need to be caught (runtime)
```java
// NullPointerException, ArrayIndexOutOfBoundsException, IllegalArgumentException
int[] arr = {1, 2, 3};
int x = arr[10];  // ArrayIndexOutOfBoundsException (no need to catch)
```

**Rule:** Use checked for recoverable errors, unchecked for programming errors.

---

### Q9: What is the difference between throw and throws?

**Answer:**

**throw**: Explicitly throw an exception
```java
public void withdraw(double amount) {
    if (amount > balance) {
        throw new IllegalArgumentException("Insufficient funds");
    }
}
```

**throws**: Declare that method might throw exception
```java
public void readFile() throws IOException {
    // Method might throw IOException
}
```

**Together:**
```java
public void processFile(String path) throws IOException {
    if (path == null) {
        throw new IllegalArgumentException("Path cannot be null");
    }
    FileReader fr = new FileReader(path);  // might throw IOException
}
```

---

### Q10: What is try-with-resources?

**Answer:**
Automatically closes resources that implement AutoCloseable.

**Without try-with-resources:**
```java
BufferedReader br = null;
try {
    br = new BufferedReader(new FileReader("file.txt"));
    String line = br.readLine();
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (br != null) {
        try {
            br.close();  // Manual close
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**With try-with-resources:**
```java
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line = br.readLine();
} catch (IOException e) {
    e.printStackTrace();
}
// br.close() called automatically!
```

**Multiple resources:**
```java
try (Connection conn = DriverManager.getConnection(url);
     PreparedStatement pstmt = conn.prepareStatement(sql);
     ResultSet rs = pstmt.executeQuery()) {

    while (rs.next()) {
        // Process
    }
} // All closed automatically in reverse order
```

---

### Q11: What is the difference between static and instance variables/methods?

**Answer:**

**Static (Class-level)**: Belongs to class, shared by all instances
```java
class Counter {
    static int count = 0;  // Shared

    static void increment() {
        count++;
    }
}

Counter.increment();  // Called on class
System.out.println(Counter.count);  // Access via class
```

**Instance (Object-level)**: Belongs to individual object
```java
class Person {
    String name;  // Each person has own name

    void greet() {
        System.out.println("Hi, I'm " + name);
    }
}

Person p1 = new Person();
p1.name = "Alice";
p1.greet();  // Called on object
```

**Key differences:**
- Static: Loaded when class loads, exists before any object
- Instance: Created when object is created
- Static methods cannot access instance variables
- Instance methods can access both static and instance variables

---

### Q12: What is method overloading vs method overriding?

**Answer:**

**Overloading (Compile-time polymorphism)**: Same name, different parameters
```java
class Calculator {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }
    int add(int a, int b, int c) { return a + b + c; }
}
```

**Overriding (Runtime polymorphism)**: Same signature in child class
```java
class Animal {
    void makeSound() {
        System.out.println("Some sound");
    }
}

class Dog extends Animal {
    @Override
    void makeSound() {
        System.out.println("Bark");
    }
}

Animal a = new Dog();
a.makeSound();  // Prints "Bark" (runtime decision)
```

---

### Q13: What is the purpose of the this keyword?

**Answer:**
Refers to current object instance.

**Uses:**

**1. Distinguish instance variable from parameter:**
```java
class Person {
    String name;

    Person(String name) {
        this.name = name;  // this.name is instance variable
    }
}
```

**2. Call another constructor:**
```java
class Person {
    String name;
    int age;

    Person(String name) {
        this(name, 0);  // Calls other constructor
    }

    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

**3. Return current object:**
```java
class Builder {
    Builder setName(String name) {
        this.name = name;
        return this;  // Method chaining
    }

    Builder setAge(int age) {
        this.age = age;
        return this;
    }
}

// Usage: new Builder().setName("Alice").setAge(25);
```

---

### Q14: What is the purpose of the super keyword?

**Answer:**
Refers to parent class.

**Uses:**

**1. Access parent's variable:**
```java
class Animal {
    String type = "Animal";
}

class Dog extends Animal {
    String type = "Dog";

    void printTypes() {
        System.out.println(type);        // Dog
        System.out.println(super.type);  // Animal
    }
}
```

**2. Call parent's method:**
```java
class Animal {
    void eat() {
        System.out.println("Eating...");
    }
}

class Dog extends Animal {
    @Override
    void eat() {
        super.eat();  // Call parent's eat()
        System.out.println("...dog food");
    }
}
```

**3. Call parent's constructor:**
```java
class Animal {
    Animal(String type) {
        System.out.println("Animal: " + type);
    }
}

class Dog extends Animal {
    Dog() {
        super("Mammal");  // MUST be first line
    }
}
```

---

### Q15: What is the difference between composition and inheritance?

**Answer:**

**Inheritance (is-a)**: Child inherits from parent
```java
class Vehicle {
    void start() {}
}

class Car extends Vehicle {  // Car IS-A Vehicle
    // Inherits start()
}
```

**Composition (has-a)**: Object contains another object
```java
class Engine {
    void start() {}
}

class Car {
    private Engine engine;  // Car HAS-A Engine

    Car() {
        this.engine = new Engine();
    }

    void start() {
        engine.start();
    }
}
```

**When to use:**
- **Inheritance**: True "is-a" relationship, need polymorphism
- **Composition**: "has-a" relationship, more flexible (preferred)

**Why prefer composition:**
```java
// Problem with inheritance: Can only extend one class
class Car extends Vehicle {}  // Can't also extend Machine

// Composition: Can have multiple
class Car {
    private Engine engine;
    private Transmission transmission;
    private GPS gps;
}
```

---

### Q16: What are access modifiers and when to use each?

**Answer:**

| Modifier | Class | Package | Subclass | World |
|----------|-------|---------|----------|-------|
| public | Yes | Yes | Yes | Yes |
| protected | Yes | Yes | Yes | No |
| default (no modifier) | Yes | Yes | No | No |
| private | Yes | No | No | No |

**Examples:**
```java
public class User {
    public String username;       // Accessible everywhere
    protected String email;       // Accessible in subclasses
    String phone;                 // Package-private (default)
    private String password;      // Only within this class

    public User(String username) {  // Constructor usually public
        this.username = username;
    }

    private void validatePassword() {  // Helper method, private
        // Internal logic
    }
}
```

**Best practices:**
- Make fields private, provide public getters/setters
- Make methods public only if needed externally
- Use protected for methods meant to be overridden

---

### Q17: What is a constructor and its rules?

**Answer:**
Special method to initialize objects.

**Rules:**
1. Same name as class
2. No return type (not even void)
3. Called automatically when object is created
4. Can be overloaded

```java
class Person {
    String name;
    int age;

    // Default constructor (no-arg)
    Person() {
        this.name = "Unknown";
        this.age = 0;
    }

    // Parameterized constructor
    Person(String name) {
        this.name = name;
        this.age = 0;
    }

    // Fully parameterized
    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

// Usage:
Person p1 = new Person();              // Calls default
Person p2 = new Person("Alice");       // Calls parameterized
Person p3 = new Person("Bob", 25);     // Calls fully parameterized
```

**Constructor chaining:**
```java
Person(String name) {
    this(name, 0);  // Calls Person(String, int)
}
```

---

### Q18: What is the difference between shallow copy and deep copy?

**Answer:**

**Shallow Copy**: Copies object but references remain shared
```java
class Person {
    String name;
    Address address;
}

Person p1 = new Person();
p1.name = "Alice";
p1.address = new Address("NYC");

// Shallow copy (clone() default behavior)
Person p2 = (Person) p1.clone();
p2.address.city = "LA";

System.out.println(p1.address.city);  // "LA" (shared reference!)
```

**Deep Copy**: Copies object and all referenced objects
```java
class Person implements Cloneable {
    String name;
    Address address;

    @Override
    public Object clone() throws CloneNotSupportedException {
        Person cloned = (Person) super.clone();
        cloned.address = (Address) address.clone();  // Deep copy
        return cloned;
    }
}

Person p2 = (Person) p1.clone();
p2.address.city = "LA";

System.out.println(p1.address.city);  // "NYC" (separate objects)
```

---

### Q19: What is the difference between pass-by-value and pass-by-reference? What does Java use?

**Answer:**
**Java is ALWAYS pass-by-value**, but the value can be a reference.

**Primitives (pass-by-value):**
```java
void modify(int x) {
    x = 100;  // Changes local copy
}

int a = 10;
modify(a);
System.out.println(a);  // 10 (unchanged)
```

**Objects (pass-by-value of reference):**
```java
void modify(Person p) {
    p.name = "Bob";  // Changes object (reference points to same object)
    p = new Person();  // Changes local reference only
    p.name = "Charlie";
}

Person person = new Person();
person.name = "Alice";
modify(person);
System.out.println(person.name);  // "Bob" (not "Charlie")
```

**Key point:** Reference itself is passed by value, not by reference.

---

### Q20: What are wrapper classes and autoboxing?

**Answer:**
**Wrapper classes**: Object representation of primitives

| Primitive | Wrapper |
|-----------|---------|
| int | Integer |
| double | Double |
| boolean | Boolean |
| char | Character |

**Why needed:**
- Collections only store objects (not primitives)
- Need null capability
- Provide utility methods

**Autoboxing (primitive → wrapper):**
```java
Integer i = 10;  // Autoboxing: int → Integer
List<Integer> list = new ArrayList<>();
list.add(5);  // Autoboxing
```

**Unboxing (wrapper → primitive):**
```java
Integer i = 100;
int x = i;  // Unboxing: Integer → int
```

**Caution:**
```java
Integer a = 100;
Integer b = 100;
a == b;  // true (cached from -128 to 127)

Integer c = 1000;
Integer d = 1000;
c == d;  // false (different objects)
c.equals(d);  // true (correct comparison)
```

---

## Collections Framework (Q21-Q40)

### Q21: What is the Collections Framework hierarchy?

**Answer:**
```
Collection (Interface)
├── List (Interface)
│   ├── ArrayList
│   ├── LinkedList
│   └── Vector
├── Set (Interface)
│   ├── HashSet
│   ├── LinkedHashSet
│   └── TreeSet (SortedSet)
└── Queue (Interface)
    ├── PriorityQueue
    ├── ArrayDeque
    └── LinkedList

Map (Interface) - Not part of Collection
├── HashMap
├── LinkedHashMap
├── TreeMap (SortedMap)
├── Hashtable
└── ConcurrentHashMap
```

---

### Q22: What is the difference between ArrayList and LinkedList?

**Answer:**

| Feature | ArrayList | LinkedList |
|---------|-----------|------------|
| Structure | Dynamic array | Doubly-linked list |
| Access | O(1) | O(n) |
| Insert/Delete (middle) | O(n) | O(1) |
| Insert/Delete (ends) | O(1) amortized | O(1) |
| Memory | Compact | More (node overhead) |

**When to use:**
```java
// ArrayList: Frequent reads, rare modifications
List<String> list = new ArrayList<>();
list.add("A");
list.get(0);  // Fast O(1)

// LinkedList: Frequent insert/delete at ends, queue/deque
List<String> list = new LinkedList<>();
list.addFirst("A");  // O(1)
list.removeLast();   // O(1)
```

---

### Q23: What is the difference between HashSet, LinkedHashSet, and TreeSet?

**Answer:**

| Feature | HashSet | LinkedHashSet | TreeSet |
|---------|---------|---------------|---------|
| Order | None | Insertion order | Sorted |
| Performance | O(1) | O(1) | O(log n) |
| Allows null | Yes (one) | Yes (one) | No |
| Implementation | HashMap | LinkedHashMap | TreeMap (Red-Black tree) |

**Examples:**
```java
// HashSet: No order
Set<Integer> set = new HashSet<>(Arrays.asList(3, 1, 2));
System.out.println(set);  // [1, 2, 3] or any order

// LinkedHashSet: Insertion order
Set<Integer> set = new LinkedHashSet<>(Arrays.asList(3, 1, 2));
System.out.println(set);  // [3, 1, 2]

// TreeSet: Sorted
Set<Integer> set = new TreeSet<>(Arrays.asList(3, 1, 2));
System.out.println(set);  // [1, 2, 3]
```

---

### Q24: What is the difference between HashMap, LinkedHashMap, and TreeMap?

**Answer:**

| Feature | HashMap | LinkedHashMap | TreeMap |
|---------|---------|---------------|---------|
| Order | None | Insertion/Access | Sorted by keys |
| Performance | O(1) | O(1) | O(log n) |
| Allows null key | Yes (one) | Yes (one) | No |
| Thread-safe | No | No | No |

**Examples:**
```java
// HashMap: No order
Map<String, Integer> map = new HashMap<>();
map.put("C", 3);
map.put("A", 1);
map.put("B", 2);
System.out.println(map.keySet());  // [A, B, C] or any order

// LinkedHashMap: Insertion order
Map<String, Integer> map = new LinkedHashMap<>();
map.put("C", 3);
map.put("A", 1);
System.out.println(map.keySet());  // [C, A]

// TreeMap: Sorted
Map<String, Integer> map = new TreeMap<>();
map.put("C", 3);
map.put("A", 1);
System.out.println(map.keySet());  // [A, C]
```

---

### Q25: How does HashMap work internally?

**Answer:**
HashMap uses **array + linked list/tree** structure.

**Internal structure:**
1. Array of buckets (default 16)
2. Each bucket stores linked list (or tree if >8 elements)
3. Hash function determines bucket index

**Process:**
```java
Map<String, Integer> map = new HashMap<>();
map.put("Alice", 25);

// 1. Calculate hash
int hash = "Alice".hashCode();  // e.g., 63165345

// 2. Calculate bucket index
int index = hash & (capacity - 1);  // e.g., 1

// 3. Store in bucket[1]
// If collision, add to linked list
```

**Collision handling:**
```
Bucket 0: null
Bucket 1: ["Alice"→25] → ["Bob"→30]  // Linked list (collision)
Bucket 2: ["Charlie"→35]
Bucket 3: null
...
```

**Key points:**
- Load factor: 0.75 (resize when 75% full)
- Resizing: Doubles capacity, rehashes all keys
- Java 8+: Tree if bucket size >8 (O(log n) instead of O(n))

---

### Q26: What is the difference between HashMap and ConcurrentHashMap?

**Answer:**

| Feature | HashMap | ConcurrentHashMap |
|---------|---------|-------------------|
| Thread-safe | No | Yes |
| Null key/value | Yes | No |
| Performance | Fast | Slower (but concurrent) |
| Locking | None | Segment locking (Java 7), CAS (Java 8+) |

**HashMap (not thread-safe):**
```java
Map<String, Integer> map = new HashMap<>();

// Problem: Race condition
Thread 1: map.put("A", 1);
Thread 2: map.put("B", 2);
// Can corrupt internal structure!
```

**ConcurrentHashMap (thread-safe):**
```java
Map<String, Integer> map = new ConcurrentHashMap<>();

// Safe: No locking for reads, fine-grained locking for writes
Thread 1: map.put("A", 1);
Thread 2: map.get("B");  // No blocking
```

**When to use:**
- Single-threaded: HashMap
- Multi-threaded: ConcurrentHashMap

---

### Q27: What is the difference between fail-fast and fail-safe iterators?

**Answer:**

**Fail-fast**: Throws ConcurrentModificationException if collection modified during iteration
```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4));

for (Integer num : list) {
    if (num == 2) {
        list.remove(num);  // ConcurrentModificationException!
    }
}

// Fix: Use Iterator.remove()
Iterator<Integer> it = list.iterator();
while (it.hasNext()) {
    Integer num = it.next();
    if (num == 2) {
        it.remove();  // Safe
    }
}
```

**Fail-safe**: Works on copy, no exception
```java
List<Integer> list = new CopyOnWriteArrayList<>(Arrays.asList(1, 2, 3, 4));

for (Integer num : list) {
    if (num == 2) {
        list.remove(num);  // No exception (iterates on copy)
    }
}
```

**Collections:**
- **Fail-fast**: ArrayList, HashMap, HashSet
- **Fail-safe**: CopyOnWriteArrayList, ConcurrentHashMap

---

### Q28: What is the difference between Comparable and Comparator?

**Answer:**

**Comparable**: Natural ordering (inside class)
```java
class Person implements Comparable<Person> {
    String name;
    int age;

    @Override
    public int compareTo(Person other) {
        return this.age - other.age;  // Sort by age
    }
}

List<Person> people = Arrays.asList(
    new Person("Alice", 25),
    new Person("Bob", 20)
);
Collections.sort(people);  // Uses compareTo()
```

**Comparator**: Custom ordering (outside class)
```java
class NameComparator implements Comparator<Person> {
    @Override
    public int compare(Person p1, Person p2) {
        return p1.name.compareTo(p2.name);  // Sort by name
    }
}

Collections.sort(people, new NameComparator());

// Lambda version (Java 8+)
Collections.sort(people, (p1, p2) -> p1.name.compareTo(p2.name));
Collections.sort(people, Comparator.comparing(Person::getName));
```

**When to use:**
- **Comparable**: One natural ordering
- **Comparator**: Multiple orderings or can't modify class

---

### Q29: What is the difference between Iterator and ListIterator?

**Answer:**

| Feature | Iterator | ListIterator |
|---------|----------|--------------|
| Direction | Forward only | Bidirectional |
| Works on | All collections | List only |
| Methods | hasNext(), next(), remove() | + hasPrevious(), previous(), add(), set() |

**Iterator:**
```java
List<String> list = Arrays.asList("A", "B", "C");
Iterator<String> it = list.iterator();

while (it.hasNext()) {
    System.out.println(it.next());
}
```

**ListIterator:**
```java
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
ListIterator<String> it = list.listIterator();

// Forward
while (it.hasNext()) {
    System.out.println(it.next());
}

// Backward
while (it.hasPrevious()) {
    System.out.println(it.previous());
}

// Modify during iteration
while (it.hasNext()) {
    String s = it.next();
    it.set(s.toLowerCase());  // Modify
}
```

---

### Q30: What is the difference between Queue and Deque?

**Answer:**

**Queue**: FIFO (First-In-First-Out)
```java
Queue<String> queue = new LinkedList<>();
queue.offer("A");  // Add to end
queue.offer("B");
queue.poll();      // Remove from front → "A"
queue.peek();      // View front → "B"
```

**Deque**: Double-ended queue (both ends)
```java
Deque<String> deque = new ArrayDeque<>();
deque.offerFirst("A");  // Add to front
deque.offerLast("B");   // Add to end
deque.pollFirst();      // Remove from front → "A"
deque.pollLast();       // Remove from end → "B"
```

**Use cases:**
- **Queue**: Task queue, BFS
- **Deque**: Undo/redo, sliding window

---

### Q31: What is PriorityQueue?

**Answer:**
Heap-based queue where elements are ordered by priority (min-heap by default).

**Default (min-heap):**
```java
Queue<Integer> pq = new PriorityQueue<>();
pq.offer(5);
pq.offer(1);
pq.offer(3);

System.out.println(pq.poll());  // 1 (smallest)
System.out.println(pq.poll());  // 3
System.out.println(pq.poll());  // 5
```

**Max-heap:**
```java
Queue<Integer> pq = new PriorityQueue<>(Comparator.reverseOrder());
pq.offer(5);
pq.offer(1);
pq.offer(3);

System.out.println(pq.poll());  // 5 (largest)
```

**Custom objects:**
```java
class Task implements Comparable<Task> {
    String name;
    int priority;

    @Override
    public int compareTo(Task other) {
        return this.priority - other.priority;  // Lower number = higher priority
    }
}

Queue<Task> tasks = new PriorityQueue<>();
tasks.offer(new Task("Low", 3));
tasks.offer(new Task("High", 1));
tasks.poll();  // Returns "High" task
```

**Time complexity:**
- offer(): O(log n)
- poll(): O(log n)
- peek(): O(1)

---

### Q32: What are the differences between Vector and ArrayList?

**Answer:**

| Feature | Vector | ArrayList |
|---------|--------|-----------|
| Synchronized | Yes (thread-safe) | No |
| Performance | Slower | Faster |
| Growth | 100% (doubles) | 50% (1.5x) |
| Legacy | Yes (Java 1.0) | No (Java 1.2) |

**Vector (legacy, avoid):**
```java
Vector<String> vector = new Vector<>();
vector.add("A");  // Synchronized method (overhead)
```

**ArrayList (preferred):**
```java
List<String> list = new ArrayList<>();
list.add("A");  // Not synchronized (faster)

// If need thread-safety:
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
```

**Recommendation**: Use ArrayList + synchronization wrapper if needed.

---

### Q33: How do you make a collection thread-safe?

**Answer:**

**Option 1: Collections.synchronized*():**
```java
List<String> list = Collections.synchronizedList(new ArrayList<>());
Set<String> set = Collections.synchronizedSet(new HashSet<>());
Map<String, Integer> map = Collections.synchronizedMap(new HashMap<>());

// IMPORTANT: Must synchronize when iterating
synchronized (list) {
    for (String s : list) {
        System.out.println(s);
    }
}
```

**Option 2: Concurrent collections (better):**
```java
List<String> list = new CopyOnWriteArrayList<>();  // Reads don't block
Set<String> set = ConcurrentHashMap.newKeySet();   // Concurrent
Map<String, Integer> map = new ConcurrentHashMap<>();  // Best performance
```

**Option 3: Manual synchronization:**
```java
class SynchronizedList {
    private List<String> list = new ArrayList<>();

    public synchronized void add(String s) {
        list.add(s);
    }

    public synchronized String get(int index) {
        return list.get(index);
    }
}
```

---

### Q34: What is the difference between Arrays.asList() and new ArrayList()?

**Answer:**

**Arrays.asList() (fixed-size):**
```java
List<String> list = Arrays.asList("A", "B", "C");
list.set(0, "Z");  // OK (modify)
list.add("D");     // UnsupportedOperationException! (can't resize)
list.remove(0);    // UnsupportedOperationException! (can't resize)

// Backed by array: changes affect original
String[] arr = {"A", "B"};
List<String> list = Arrays.asList(arr);
list.set(0, "Z");
System.out.println(arr[0]);  // "Z" (affected!)
```

**new ArrayList() (resizable):**
```java
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
list.add("D");     // OK
list.remove(0);    // OK
```

---

### Q35: What is the time complexity of common collection operations?

**Answer:**

**ArrayList:**
- get(index): O(1)
- add(element): O(1) amortized
- add(index, element): O(n)
- remove(index): O(n)
- contains(element): O(n)

**LinkedList:**
- get(index): O(n)
- add(element): O(1)
- addFirst/addLast: O(1)
- remove(index): O(n)
- contains(element): O(n)

**HashMap:**
- get(key): O(1) average
- put(key, value): O(1) average
- remove(key): O(1) average
- containsKey: O(1) average

**TreeMap:**
- get(key): O(log n)
- put(key, value): O(log n)
- remove(key): O(log n)

**HashSet:**
- add(element): O(1) average
- contains(element): O(1) average
- remove(element): O(1) average

**TreeSet:**
- add(element): O(log n)
- contains(element): O(log n)
- remove(element): O(log n)

---

### Q36: How do you sort a List?

**Answer:**

**Option 1: Collections.sort() (modifies original):**
```java
List<Integer> list = Arrays.asList(3, 1, 4, 2);
Collections.sort(list);  // [1, 2, 3, 4]

// Reverse order
Collections.sort(list, Collections.reverseOrder());
```

**Option 2: List.sort() (Java 8+):**
```java
list.sort(Comparator.naturalOrder());
list.sort(Comparator.reverseOrder());
```

**Custom sorting:**
```java
List<Person> people = getPeople();

// By age
people.sort(Comparator.comparing(Person::getAge));

// By name
people.sort(Comparator.comparing(Person::getName));

// Multiple fields
people.sort(Comparator.comparing(Person::getAge)
                      .thenComparing(Person::getName));

// Reverse
people.sort(Comparator.comparing(Person::getAge).reversed());
```

---

### Q37: How do you remove duplicates from a List?

**Answer:**

**Option 1: Using Set:**
```java
List<Integer> list = Arrays.asList(1, 2, 2, 3, 3, 4);
List<Integer> unique = new ArrayList<>(new HashSet<>(list));
// Order not preserved
```

**Option 2: Using LinkedHashSet (preserve order):**
```java
List<Integer> unique = new ArrayList<>(new LinkedHashSet<>(list));
```

**Option 3: Using Stream (Java 8+):**
```java
List<Integer> unique = list.stream()
    .distinct()
    .collect(Collectors.toList());
```

---

### Q38: How do you find common elements between two Lists?

**Answer:**

**Option 1: retainAll():**
```java
List<Integer> list1 = Arrays.asList(1, 2, 3, 4);
List<Integer> list2 = Arrays.asList(3, 4, 5, 6);

list1.retainAll(list2);  // list1 now [3, 4]
```

**Option 2: Stream:**
```java
List<Integer> common = list1.stream()
    .filter(list2::contains)
    .collect(Collectors.toList());
```

**Option 3: Using Set (faster for large lists):**
```java
Set<Integer> set2 = new HashSet<>(list2);
List<Integer> common = list1.stream()
    .filter(set2::contains)
    .collect(Collectors.toList());
```

---

### Q39: What is the Collections utility class?

**Answer:**
Utility methods for collections.

**Common methods:**
```java
// Sort
Collections.sort(list);
Collections.sort(list, Comparator.reverseOrder());

// Search (must be sorted)
int index = Collections.binarySearch(list, 5);

// Shuffle
Collections.shuffle(list);

// Reverse
Collections.reverse(list);

// Min/Max
int min = Collections.min(list);
int max = Collections.max(list);

// Frequency
int count = Collections.frequency(list, 3);  // Count 3's

// Fill
Collections.fill(list, 0);  // Set all to 0

// Replace
Collections.replaceAll(list, 1, 10);  // Replace all 1's with 10

// Unmodifiable
List<String> immutable = Collections.unmodifiableList(list);

// Singleton
Set<String> single = Collections.singleton("A");  // Immutable set with 1 element

// Empty
List<String> empty = Collections.emptyList();
```

---

### Q40: What is the difference between Collection and Collections?

**Answer:**

**Collection**: Interface (root of collection hierarchy)
```java
Collection<String> collection = new ArrayList<>();
collection.add("A");
collection.remove("A");
collection.size();
```

**Collections**: Utility class with static methods
```java
Collections.sort(list);
Collections.reverse(list);
Collections.shuffle(list);
List<String> immutable = Collections.unmodifiableList(list);
```

---

## Multithreading and Concurrency (Q41-Q60)

### Q41: What is the difference between process and thread?

**Answer:**

**Process**: Independent program with own memory space
- Heavy-weight
- Inter-process communication is expensive
- Example: Chrome tab is a process

**Thread**: Lightweight unit within a process, shares memory
- Light-weight
- Threads share memory, communication is fast
- Example: Multiple threads in one Java application

**In Java:**
```java
// Creating threads (multiple tasks in one process)
Thread t1 = new Thread(() -> System.out.println("Thread 1"));
Thread t2 = new Thread(() -> System.out.println("Thread 2"));
t1.start();
t2.start();
```

---

### Q42: What are the ways to create a thread in Java?

**Answer:**

**Method 1: Extend Thread class**
```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread running");
    }
}

MyThread t = new MyThread();
t.start();
```

**Method 2: Implement Runnable**
```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable running");
    }
}

Thread t = new Thread(new MyRunnable());
t.start();

// Lambda version (Java 8+)
Thread t = new Thread(() -> System.out.println("Lambda thread"));
t.start();
```

**Method 3: ExecutorService (recommended)**
```java
ExecutorService executor = Executors.newFixedThreadPool(4);
executor.submit(() -> System.out.println("Task"));
executor.shutdown();
```

**Method 4: Callable (returns result)**
```java
ExecutorService executor = Executors.newFixedThreadPool(2);
Future<Integer> future = executor.submit(() -> {
    return 42;
});
Integer result = future.get();  // Blocks until complete
```

---

### Q43: What is the difference between start() and run()?

**Answer:**

**start()**: Creates new thread, calls run() in that thread
```java
Thread t = new Thread(() -> System.out.println("In new thread"));
t.start();  // Creates new thread, runs run() in that thread
```

**run()**: Executes in current thread (no new thread created)
```java
Thread t = new Thread(() -> System.out.println("In same thread"));
t.run();  // Executes in current thread (main thread)
// No parallelism!
```

**Key difference:**
```java
// start() - parallel execution
new Thread(() -> task1()).start();
new Thread(() -> task2()).start();
// task1 and task2 run concurrently

// run() - sequential execution
new Thread(() -> task1()).run();
new Thread(() -> task2()).run();
// task1 completes, then task2 starts (no parallelism)
```

---

### Q44: What is synchronization and why is it needed?

**Answer:**
Prevents race conditions when multiple threads access shared data.

**Problem (race condition):**
```java
class Counter {
    private int count = 0;

    public void increment() {
        count++;  // Not atomic! (read, increment, write)
    }
}

// 2 threads call increment() 1000 times each
// Expected: 2000
// Actual: ~1800 (lost updates!)
```

**Solution (synchronized):**
```java
class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;  // Only one thread at a time
    }
}

// Now: 2000 (correct)
```

**How it works:**
- Only one thread can execute synchronized method at a time
- Other threads wait (blocked)

---

### Q45: What are the types of synchronization?

**Answer:**

**1. Method synchronization:**
```java
public synchronized void method() {
    // Synchronized on this object
}
```

**2. Block synchronization:**
```java
public void method() {
    synchronized (this) {
        // Only this block synchronized
    }
}
```

**3. Static synchronization:**
```java
public static synchronized void method() {
    // Synchronized on Class object
}
```

**4. Custom lock object:**
```java
private final Object lock = new Object();

public void method() {
    synchronized (lock) {
        // Synchronized on custom lock
    }
}
```

**When to use block over method:**
```java
public void process() {
    // Non-critical code (parallel)
    doSomething();

    synchronized (this) {
        // Only critical section synchronized
        sharedData++;
    }

    // More non-critical code (parallel)
    doSomethingElse();
}
```

---

### Q46: What is the difference between synchronized and volatile?

**Answer:**

**synchronized**: Mutual exclusion + visibility
```java
class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;  // Atomic operation
    }
}
```

**volatile**: Visibility only (no atomicity)
```java
class Flag {
    private volatile boolean flag = false;

    public void setFlag() {
        flag = true;  // All threads see immediately
    }

    public void check() {
        if (flag) {
            // React to flag change
        }
    }
}
```

**Key differences:**

| Feature | synchronized | volatile |
|---------|--------------|----------|
| Atomicity | Yes | No |
| Visibility | Yes | Yes |
| Locking | Yes (blocks) | No (non-blocking) |
| Use case | Compound operations | Simple flags/status |

**volatile limitation:**
```java
private volatile int count = 0;

public void increment() {
    count++;  // NOT thread-safe (read-modify-write)
}

// Need synchronized for this
```

---

### Q47: What is deadlock and how to prevent it?

**Answer:**
**Deadlock**: Two threads wait for each other's locks forever.

**Example:**
```java
Object lock1 = new Object();
Object lock2 = new Object();

// Thread 1
synchronized (lock1) {
    synchronized (lock2) {
        // Do work
    }
}

// Thread 2
synchronized (lock2) {
    synchronized (lock1) {  // Deadlock!
        // Do work
    }
}
```

**Prevention strategies:**

**1. Same lock order:**
```java
// Both threads acquire in same order
// Thread 1 and Thread 2
synchronized (lock1) {
    synchronized (lock2) {
        // Do work
    }
}
```

**2. Use tryLock() with timeout:**
```java
Lock lock1 = new ReentrantLock();
Lock lock2 = new ReentrantLock();

if (lock1.tryLock(100, TimeUnit.MILLISECONDS)) {
    try {
        if (lock2.tryLock(100, TimeUnit.MILLISECONDS)) {
            try {
                // Do work
            } finally {
                lock2.unlock();
            }
        }
    } finally {
        lock1.unlock();
    }
}
```

**3. Avoid nested locks:**
```java
// Don't hold multiple locks
synchronized (lock1) {
    // Work with resource 1
}
synchronized (lock2) {
    // Work with resource 2
}
```

---

### Q48: What is the difference between wait() and sleep()?

**Answer:**

**wait()**: Releases lock, waits for notify()
```java
synchronized (obj) {
    obj.wait();  // Releases lock, waits
}
// Another thread calls obj.notify()
```

**sleep()**: Holds lock, waits for time
```java
synchronized (obj) {
    Thread.sleep(1000);  // Holds lock for 1 second
}
```

**Key differences:**

| Feature | wait() | sleep() |
|---------|--------|---------|
| Package | Object class | Thread class |
| Lock | Releases | Holds |
| Wake up | notify()/notifyAll() | Time expires |
| Must be synchronized | Yes | No |

**Example:**
```java
class Producer {
    synchronized void produce() {
        // Produce item
        notify();  // Wake up consumer
    }
}

class Consumer {
    synchronized void consume() {
        wait();  // Wait for item, releases lock
        // Consume item
    }
}
```

---

### Q49: What is the difference between notify() and notifyAll()?

**Answer:**

**notify()**: Wakes up ONE waiting thread
```java
synchronized (obj) {
    obj.notify();  // Random one thread woken up
}
```

**notifyAll()**: Wakes up ALL waiting threads
```java
synchronized (obj) {
    obj.notifyAll();  // All threads compete for lock
}
```

**When to use:**
- **notify()**: Only one thread should process (e.g., one consumer)
- **notifyAll()**: Multiple threads might need to check condition (safer)

**Example:**
```java
class BoundedBuffer {
    private Queue<Integer> queue = new LinkedList<>();
    private int capacity = 10;

    public synchronized void put(int value) throws InterruptedException {
        while (queue.size() == capacity) {
            wait();  // Buffer full
        }
        queue.add(value);
        notifyAll();  // Wake up all consumers
    }

    public synchronized int take() throws InterruptedException {
        while (queue.isEmpty()) {
            wait();  // Buffer empty
        }
        int value = queue.poll();
        notifyAll();  // Wake up all producers
        return value;
    }
}
```

**Why notifyAll() is safer:**
- notify() might wake wrong thread (e.g., another producer instead of consumer)
- notifyAll() ensures correct thread wakes up

---

### Q50: What is ExecutorService?

**Answer:**
Thread pool manager for executing tasks.

**Types:**

**1. FixedThreadPool (fixed number of threads):**
```java
ExecutorService executor = Executors.newFixedThreadPool(4);

for (int i = 0; i < 100; i++) {
    final int taskId = i;
    executor.submit(() -> {
        System.out.println("Task " + taskId);
    });
}

executor.shutdown();  // No new tasks, finish existing
executor.awaitTermination(1, TimeUnit.MINUTES);  // Wait for completion
```

**2. CachedThreadPool (creates threads as needed):**
```java
ExecutorService executor = Executors.newCachedThreadPool();
// Creates new thread if all busy, reuses idle threads
```

**3. SingleThreadExecutor (one thread, sequential):**
```java
ExecutorService executor = Executors.newSingleThreadExecutor();
// Executes tasks one at a time
```

**4. ScheduledThreadPool (delayed/periodic tasks):**
```java
ScheduledExecutorService executor = Executors.newScheduledThreadPool(2);

// Execute once after delay
executor.schedule(() -> System.out.println("Task"), 5, TimeUnit.SECONDS);

// Execute periodically
executor.scheduleAtFixedRate(() -> System.out.println("Periodic"), 0, 1, TimeUnit.SECONDS);
```

**Benefits:**
- Reuses threads (no creation overhead)
- Limits concurrent threads (prevents resource exhaustion)
- Task queue management

---

### Q51: What is the difference between submit() and execute()?

**Answer:**

**execute()**: Fire and forget (no return value)
```java
ExecutorService executor = Executors.newFixedThreadPool(2);
executor.execute(() -> {
    System.out.println("Task");
});
// Can't get result or check status
```

**submit()**: Returns Future (can get result/status)
```java
Future<Integer> future = executor.submit(() -> {
    return 42;
});

Integer result = future.get();  // Blocks until complete
boolean done = future.isDone();
boolean cancelled = future.isCancelled();
```

**Use cases:**
- **execute()**: Fire-and-forget tasks (logging, notifications)
- **submit()**: Need result or status (computation, data processing)

---

### Q52: What is Future and CompletableFuture?

**Answer:**

**Future (Java 5)**: Represents async computation result
```java
ExecutorService executor = Executors.newFixedThreadPool(2);

Future<Integer> future = executor.submit(() -> {
    Thread.sleep(1000);
    return 42;
});

// Blocking operations
Integer result = future.get();  // Waits up to 1 second
Integer result = future.get(1, TimeUnit.SECONDS);  // Timeout
boolean done = future.isDone();
future.cancel(true);  // Cancel task
```

**CompletableFuture (Java 8)**: Non-blocking async operations
```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    return 42;
});

// Non-blocking chaining
future.thenApply(result -> result * 2)
      .thenAccept(result -> System.out.println(result))
      .exceptionally(ex -> {
          System.err.println("Error: " + ex);
          return null;
      });

// Combine multiple futures
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 10);
CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(() -> 20);

CompletableFuture<Integer> combined = future1.thenCombine(future2, (a, b) -> a + b);
System.out.println(combined.get());  // 30
```

**Key advantages of CompletableFuture:**
- Non-blocking
- Composable (chaining)
- Exception handling
- Combine multiple futures

---

### Q53: What is ReentrantLock?

**Answer:**
More flexible alternative to synchronized.

**Basic usage:**
```java
private final Lock lock = new ReentrantLock();

public void method() {
    lock.lock();
    try {
        // Critical section
    } finally {
        lock.unlock();  // MUST unlock in finally
    }
}
```

**Advantages over synchronized:**

**1. tryLock() (non-blocking):**
```java
if (lock.tryLock()) {
    try {
        // Got lock
    } finally {
        lock.unlock();
    }
} else {
    // Couldn't get lock, do something else
}
```

**2. tryLock() with timeout:**
```java
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        // Got lock within 1 second
    } finally {
        lock.unlock();
    }
} else {
    // Timeout
}
```

**3. Interruptible locking:**
```java
try {
    lock.lockInterruptibly();
    // Critical section
} catch (InterruptedException e) {
    // Thread interrupted while waiting
} finally {
    lock.unlock();
}
```

**4. Condition variables:**
```java
private final Lock lock = new ReentrantLock();
private final Condition condition = lock.newCondition();

// Thread 1
lock.lock();
try {
    while (!ready) {
        condition.await();  // Like wait()
    }
} finally {
    lock.unlock();
}

// Thread 2
lock.lock();
try {
    ready = true;
    condition.signal();  // Like notify()
} finally {
    lock.unlock();
}
```

---

### Q54: What is ReadWriteLock?

**Answer:**
Separate locks for read and write operations.

**Advantage**: Multiple readers can access simultaneously, writer has exclusive access.

```java
private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
private final Lock readLock = rwLock.readLock();
private final Lock writeLock = rwLock.writeLock();

// Multiple threads can read simultaneously
public String read() {
    readLock.lock();
    try {
        return data;  // Many readers allowed
    } finally {
        readLock.unlock();
    }
}

// Only one writer at a time, no readers allowed
public void write(String newData) {
    writeLock.lock();
    try {
        data = newData;  // Exclusive access
    } finally {
        writeLock.unlock();
    }
}
```

**Use case**: Read-heavy workloads (cache, configuration)

---

### Q55: What are thread-safe collections in Java?

**Answer:**

**Legacy (avoid):**
- Vector, Hashtable (slow, entire collection locked)

**Collections.synchronized***:**
```java
List<String> list = Collections.synchronizedList(new ArrayList<>());
Map<String, Integer> map = Collections.synchronizedMap(new HashMap<>());

// Must manually synchronize iteration
synchronized (list) {
    for (String s : list) {
        System.out.println(s);
    }
}
```

**Concurrent collections (recommended):**

**1. ConcurrentHashMap:**
```java
Map<String, Integer> map = new ConcurrentHashMap<>();
map.put("A", 1);  // Thread-safe, high performance
```

**2. CopyOnWriteArrayList:**
```java
List<String> list = new CopyOnWriteArrayList<>();
// Reads don't block, writes copy entire list
// Good for read-heavy workloads
```

**3. CopyOnWriteArraySet:**
```java
Set<String> set = new CopyOnWriteArraySet<>();
```

**4. BlockingQueue:**
```java
BlockingQueue<String> queue = new LinkedBlockingQueue<>();
queue.put("A");  // Blocks if full
String item = queue.take();  // Blocks if empty
```

**5. ConcurrentLinkedQueue:**
```java
Queue<String> queue = new ConcurrentLinkedQueue<>();
// Non-blocking, lock-free
```

---

### Q56: What is the difference between Synchronized collection and Concurrent collection?

**Answer:**

| Feature | Synchronized | Concurrent |
|---------|--------------|------------|
| Locking | Entire collection | Segment/fine-grained |
| Performance | Low (entire lock) | High (partial lock) |
| Iteration | Manual sync needed | No manual sync |
| Null values | Allowed | Not allowed (most) |

**Synchronized:**
```java
List<String> list = Collections.synchronizedList(new ArrayList<>());

// Problem: Iteration not thread-safe
for (String s : list) {  // ConcurrentModificationException!
    System.out.println(s);
}

// Fix: Manual synchronization
synchronized (list) {
    for (String s : list) {
        System.out.println(s);
    }
}
```

**Concurrent:**
```java
List<String> list = new CopyOnWriteArrayList<>();

// Safe: No manual synchronization needed
for (String s : list) {
    System.out.println(s);
}

// Another thread can modify during iteration
list.add("New");  // No exception
```

---

### Q57: What is the difference between CountDownLatch, CyclicBarrier, and Semaphore?

**Answer:**

**CountDownLatch**: Waits for N events before proceeding (one-time)
```java
CountDownLatch latch = new CountDownLatch(3);

// 3 worker threads
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        doWork();
        latch.countDown();  // Signal completion
    }).start();
}

// Main thread waits
latch.await();  // Blocks until count reaches 0
System.out.println("All workers done");
```

**CyclicBarrier**: Waits for N threads to reach barrier (reusable)
```java
CyclicBarrier barrier = new CyclicBarrier(3, () -> {
    System.out.println("All reached barrier");
});

// 3 threads
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        doPhase1();
        barrier.await();  // Wait for others
        doPhase2();
        barrier.await();  // Reusable!
    }).start();
}
```

**Semaphore**: Limits concurrent access to resource
```java
Semaphore semaphore = new Semaphore(3);  // 3 permits

// 10 threads, only 3 can execute at once
for (int i = 0; i < 10; i++) {
    new Thread(() -> {
        semaphore.acquire();  // Get permit (blocks if none available)
        try {
            useResource();  // Only 3 threads here at once
        } finally {
            semaphore.release();  // Release permit
        }
    }).start();
}
```

**Use cases:**
- **CountDownLatch**: Wait for initialization tasks
- **CyclicBarrier**: Multi-phase parallel algorithms
- **Semaphore**: Connection pool, rate limiting

---

### Q58: What is ThreadLocal?

**Answer:**
Each thread has its own copy of variable.

**Use case**: Store thread-specific data (user session, transaction context)

```java
private static ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);

public void increment() {
    int value = threadLocal.get();
    threadLocal.set(value + 1);
}

// Thread 1
threadLocal.set(10);
System.out.println(threadLocal.get());  // 10

// Thread 2
threadLocal.set(20);
System.out.println(threadLocal.get());  // 20 (independent)
```

**Real-world example (transaction context):**
```java
public class TransactionContext {
    private static ThreadLocal<String> transactionId = new ThreadLocal<>();

    public static void setTransactionId(String id) {
        transactionId.set(id);
    }

    public static String getTransactionId() {
        return transactionId.get();
    }

    public static void clear() {
        transactionId.remove();  // Important: prevent memory leak
    }
}

// In request handler
TransactionContext.setTransactionId(UUID.randomUUID().toString());
try {
    processRequest();  // All methods can access transaction ID
} finally {
    TransactionContext.clear();  // Cleanup
}
```

**Warning**: Always remove() in finally to prevent memory leaks (especially in thread pools).

---

### Q59: What is the Fork/Join framework?

**Answer:**
Divide-and-conquer parallel processing.

**Use case**: Recursively split task into subtasks.

```java
class SumTask extends RecursiveTask<Long> {
    private final int[] array;
    private final int start, end;
    private static final int THRESHOLD = 1000;

    SumTask(int[] array, int start, int end) {
        this.array = array;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        int length = end - start;

        // Base case: small enough, compute directly
        if (length <= THRESHOLD) {
            long sum = 0;
            for (int i = start; i < end; i++) {
                sum += array[i];
            }
            return sum;
        }

        // Recursive case: split in half
        int mid = start + length / 2;
        SumTask leftTask = new SumTask(array, start, mid);
        SumTask rightTask = new SumTask(array, mid, end);

        leftTask.fork();  // Execute in parallel
        long rightResult = rightTask.compute();
        long leftResult = leftTask.join();  // Wait for result

        return leftResult + rightResult;
    }
}

// Usage
int[] array = new int[10_000_000];
ForkJoinPool pool = new ForkJoinPool();
long sum = pool.invoke(new SumTask(array, 0, array.length));
```

**When to use**: Large arrays, tree processing, parallel algorithms

---

### Q60: What are common thread lifecycle states?

**Answer:**

**States:**
1. **NEW**: Thread created but not started
2. **RUNNABLE**: Running or ready to run
3. **BLOCKED**: Waiting for monitor lock
4. **WAITING**: Waiting indefinitely (wait(), join())
5. **TIMED_WAITING**: Waiting for specified time (sleep(), wait(timeout))
6. **TERMINATED**: Finished execution

```java
Thread t = new Thread(() -> {
    System.out.println("Running");
});

System.out.println(t.getState());  // NEW

t.start();
System.out.println(t.getState());  // RUNNABLE

t.join();
System.out.println(t.getState());  // TERMINATED
```

**Transitions:**
```
NEW → RUNNABLE → TERMINATED

RUNNABLE ↔ BLOCKED (waiting for lock)
RUNNABLE ↔ WAITING (wait(), join())
RUNNABLE ↔ TIMED_WAITING (sleep(), wait(timeout))
```

---

## Java 8+ Features (Q61-Q80)

### Q61: What is a lambda expression?

**Answer:**
Concise way to represent anonymous function.

**Syntax:**
```java
(parameters) -> expression
(parameters) -> { statements; }
```

**Examples:**
```java
// No parameters
Runnable r = () -> System.out.println("Hello");

// One parameter (parentheses optional)
Consumer<String> c = s -> System.out.println(s);

// Multiple parameters
Comparator<Integer> comp = (a, b) -> a.compareTo(b);

// Multiple statements
Function<Integer, Integer> f = x -> {
    int result = x * x;
    return result;
};
```

**Before lambda:**
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return s1.compareTo(s2);
    }
});
```

**With lambda:**
```java
Collections.sort(names, (s1, s2) -> s1.compareTo(s2));
// Or method reference
Collections.sort(names, String::compareTo);
```

---

### Q62: What is a functional interface?

**Answer:**
Interface with exactly one abstract method (SAM - Single Abstract Method).

**Built-in functional interfaces:**
```java
// Function<T, R>: Takes T, returns R
Function<String, Integer> length = s -> s.length();
Integer len = length.apply("Hello");  // 5

// Predicate<T>: Takes T, returns boolean
Predicate<Integer> isEven = n -> n % 2 == 0;
boolean result = isEven.test(4);  // true

// Consumer<T>: Takes T, returns nothing
Consumer<String> print = s -> System.out.println(s);
print.accept("Hello");

// Supplier<T>: Takes nothing, returns T
Supplier<Double> random = () -> Math.random();
Double value = random.get();

// BiFunction<T, U, R>: Takes T and U, returns R
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
Integer sum = add.apply(5, 3);  // 8
```

**Custom functional interface:**
```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);

    // Can have default and static methods
    default int square(int n) {
        return n * n;
    }
}

Calculator add = (a, b) -> a + b;
Calculator multiply = (a, b) -> a * b;

System.out.println(add.calculate(5, 3));  // 8
System.out.println(multiply.calculate(5, 3));  // 15
```

---

### Q63: What are method references?

**Answer:**
Shorthand for lambda expressions that only call a method.

**Types:**

**1. Static method reference:**
```java
// Lambda
Function<String, Integer> f = s -> Integer.parseInt(s);

// Method reference
Function<String, Integer> f = Integer::parseInt;
```

**2. Instance method on object:**
```java
String str = "Hello";

// Lambda
Supplier<String> s = () -> str.toUpperCase();

// Method reference
Supplier<String> s = str::toUpperCase;
```

**3. Instance method on class:**
```java
// Lambda
Function<String, String> f = s -> s.toUpperCase();

// Method reference
Function<String, String> f = String::toUpperCase;
```

**4. Constructor reference:**
```java
// Lambda
Supplier<List<String>> s = () -> new ArrayList<>();

// Method reference
Supplier<List<String>> s = ArrayList::new;

// With parameter
Function<Integer, List<String>> f = ArrayList::new;
List<String> list = f.apply(10);  // Initial capacity 10
```

**Examples:**
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// Lambda
names.forEach(s -> System.out.println(s));

// Method reference
names.forEach(System.out::println);

// Lambda
names.sort((s1, s2) -> s1.compareTo(s2));

// Method reference
names.sort(String::compareTo);
```

---

### Q64: What is the Stream API?

**Answer:**
Functional-style operations on collections.

**Basic operations:**
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Filter: Keep elements matching condition
List<Integer> evens = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());  // [2, 4, 6, 8, 10]

// Map: Transform elements
List<Integer> squares = numbers.stream()
    .map(n -> n * n)
    .collect(Collectors.toList());  // [1, 4, 9, 16, ...]

// Reduce: Aggregate to single value
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);  // 55

// Count
long count = numbers.stream()
    .filter(n -> n > 5)
    .count();  // 5
```

**Intermediate vs Terminal operations:**

**Intermediate (lazy, returns stream):**
- filter(), map(), sorted(), distinct(), limit(), skip()

**Terminal (eager, returns result):**
- collect(), forEach(), reduce(), count(), findFirst(), anyMatch()

**Chaining:**
```java
double average = numbers.stream()
    .filter(n -> n > 5)      // Intermediate
    .map(n -> n * 2)         // Intermediate
    .mapToInt(Integer::intValue)  // Intermediate
    .average()               // Terminal
    .orElse(0.0);
```

---

### Q65: What are common Stream operations?

**Answer:**

**filter()**: Keep matching elements
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
List<String> filtered = names.stream()
    .filter(name -> name.startsWith("A"))
    .collect(Collectors.toList());  // ["Alice"]
```

**map()**: Transform elements
```java
List<String> upperCase = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());  // ["ALICE", "BOB", ...]
```

**flatMap()**: Flatten nested structures
```java
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4)
);
List<Integer> flat = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());  // [1, 2, 3, 4]
```

**sorted()**: Sort elements
```java
List<String> sorted = names.stream()
    .sorted()
    .collect(Collectors.toList());  // ["Alice", "Bob", "Charlie", "David"]

// Custom comparator
List<String> sortedByLength = names.stream()
    .sorted(Comparator.comparing(String::length))
    .collect(Collectors.toList());
```

**distinct()**: Remove duplicates
```java
List<Integer> unique = Arrays.asList(1, 2, 2, 3, 3, 4).stream()
    .distinct()
    .collect(Collectors.toList());  // [1, 2, 3, 4]
```

**limit() / skip()**: Pagination
```java
List<Integer> page = numbers.stream()
    .skip(5)   // Skip first 5
    .limit(3)  // Take next 3
    .collect(Collectors.toList());  // [6, 7, 8]
```

**anyMatch() / allMatch() / noneMatch()**:
```java
boolean hasEven = numbers.stream().anyMatch(n -> n % 2 == 0);   // true
boolean allPositive = numbers.stream().allMatch(n -> n > 0);     // true
boolean noneNegative = numbers.stream().noneMatch(n -> n < 0);   // true
```

**findFirst() / findAny()**:
```java
Optional<Integer> first = numbers.stream()
    .filter(n -> n > 5)
    .findFirst();  // Optional[6]
```

---

### Q66: What is Optional?

**Answer:**
Container for value that may or may not be present (avoid NullPointerException).

**Creating Optional:**
```java
Optional<String> optional = Optional.of("value");           // NPE if null
Optional<String> empty = Optional.empty();                  // Empty
Optional<String> nullable = Optional.ofNullable(getValue()); // Safe
```

**Checking presence:**
```java
if (optional.isPresent()) {
    String value = optional.get();
}

// Java 11+
if (optional.isEmpty()) {
    // Handle empty
}
```

**Getting value:**
```java
String value = optional.get();  // Throws NoSuchElementException if empty

String value = optional.orElse("default");  // Return default if empty

String value = optional.orElseGet(() -> computeDefault());  // Lazy default

String value = optional.orElseThrow(() -> new Exception("No value"));
```

**Transforming:**
```java
Optional<String> upper = optional.map(String::toUpperCase);

Optional<Integer> length = optional
    .filter(s -> s.length() > 5)
    .map(String::length);
```

**ifPresent():**
```java
optional.ifPresent(value -> System.out.println(value));

// Java 9+
optional.ifPresentOrElse(
    value -> System.out.println(value),
    () -> System.out.println("Empty")
);
```

**Real-world example:**
```java
// BAD: Verbose null checks
User user = getUserById(id);
if (user != null) {
    Address address = user.getAddress();
    if (address != null) {
        String city = address.getCity();
        if (city != null) {
            System.out.println(city);
        }
    }
}

// GOOD: Optional chaining
Optional<User> user = getUserById(id);
user.flatMap(User::getAddress)
    .map(Address::getCity)
    .ifPresent(System.out::println);
```

---

### Q67: What are Collectors?

**Answer:**
Terminal operations that collect stream elements.

**toList() / toSet():**
```java
List<String> list = stream.collect(Collectors.toList());
Set<String> set = stream.collect(Collectors.toSet());
```

**toMap():**
```java
Map<Integer, String> map = people.stream()
    .collect(Collectors.toMap(
        Person::getId,      // Key
        Person::getName     // Value
    ));
```

**joining():**
```java
String joined = stream.collect(Collectors.joining(", "));
// "Alice, Bob, Charlie"

String csv = stream.collect(Collectors.joining(", ", "[", "]"));
// "[Alice, Bob, Charlie]"
```

**groupingBy():**
```java
Map<String, List<Person>> byCity = people.stream()
    .collect(Collectors.groupingBy(Person::getCity));

// {
//   "NYC": [Person(...), Person(...)],
//   "LA": [Person(...)]
// }
```

**partitioningBy():**
```java
Map<Boolean, List<Integer>> partitioned = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));

// {
//   true: [2, 4, 6, 8],   // Even
//   false: [1, 3, 5, 7, 9] // Odd
// }
```

**counting() / summingInt() / averagingInt():**
```java
long count = stream.collect(Collectors.counting());

int sum = stream.collect(Collectors.summingInt(Person::getAge));

double avg = stream.collect(Collectors.averagingInt(Person::getAge));
```

**Complex example:**
```java
Map<String, Long> cityCount = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.counting()
    ));
// {"NYC": 5, "LA": 3}
```

---

### Q68: What is the difference between map() and flatMap()?

**Answer:**

**map()**: One-to-one transformation
```java
List<String> words = Arrays.asList("hello", "world");

List<Integer> lengths = words.stream()
    .map(String::length)  // "hello" -> 5, "world" -> 5
    .collect(Collectors.toList());  // [5, 5]
```

**flatMap()**: One-to-many transformation (flattens)
```java
List<String> words = Arrays.asList("hello", "world");

List<String> letters = words.stream()
    .map(word -> word.split(""))  // ["h","e","l","l","o"], ["w","o","r","l","d"]
    .flatMap(Arrays::stream)      // Flatten to single stream
    .collect(Collectors.toList());  // [h, e, l, l, o, w, o, r, l, d]
```

**When to use:**
- **map()**: Transform each element
- **flatMap()**: Transform to stream, then flatten

**Real-world example:**
```java
List<Order> orders = getOrders();

// Get all items from all orders
List<Item> allItems = orders.stream()
    .flatMap(order -> order.getItems().stream())
    .collect(Collectors.toList());
```

---

### Q69: What are default and static methods in interfaces (Java 8)?

**Answer:**

**Default methods**: Provide implementation in interface
```java
interface Vehicle {
    // Abstract method
    void start();

    // Default method
    default void stop() {
        System.out.println("Vehicle stopped");
    }
}

class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car started");
    }

    // Can override default method (optional)
    @Override
    public void stop() {
        System.out.println("Car stopped");
    }
}
```

**Static methods**: Utility methods in interface
```java
interface MathUtils {
    static int add(int a, int b) {
        return a + b;
    }

    static int multiply(int a, int b) {
        return a * b;
    }
}

// Call via interface name
int result = MathUtils.add(5, 3);
```

**Why needed:**
- Backward compatibility (add methods without breaking implementations)
- Code reuse (common implementations)

**Diamond problem:**
```java
interface A {
    default void method() { System.out.println("A"); }
}

interface B {
    default void method() { System.out.println("B"); }
}

class C implements A, B {
    @Override
    public void method() {
        A.super.method();  // Explicitly choose A's implementation
    }
}
```

---

### Q70: What is the new Date/Time API (Java 8)?

**Answer:**

**LocalDate (date without time):**
```java
LocalDate today = LocalDate.now();              // 2024-05-10
LocalDate date = LocalDate.of(2024, 5, 10);
LocalDate tomorrow = today.plusDays(1);
LocalDate lastWeek = today.minusWeeks(1);

int year = date.getYear();
Month month = date.getMonth();
int day = date.getDayOfMonth();
```

**LocalTime (time without date):**
```java
LocalTime now = LocalTime.now();                // 14:30:15
LocalTime time = LocalTime.of(14, 30);
LocalTime later = now.plusHours(2);
```

**LocalDateTime (date + time):**
```java
LocalDateTime now = LocalDateTime.now();
LocalDateTime dt = LocalDateTime.of(2024, 5, 10, 14, 30);
LocalDateTime nextHour = now.plusHours(1);
```

**ZonedDateTime (date + time + timezone):**
```java
ZonedDateTime zdt = ZonedDateTime.now(ZoneId.of("America/New_York"));
ZonedDateTime utc = ZonedDateTime.now(ZoneId.of("UTC"));
```

**Formatting:**
```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

String formatted = now.format(formatter);  // "2024-05-10 14:30:15"

LocalDateTime parsed = LocalDateTime.parse("2024-05-10 14:30:15", formatter);
```

**Period (date-based duration):**
```java
Period period = Period.between(
    LocalDate.of(2024, 1, 1),
    LocalDate.now()
);
int days = period.getDays();
int months = period.getMonths();
```

**Duration (time-based duration):**
```java
Duration duration = Duration.between(
    LocalDateTime.of(2024, 1, 1, 10, 0),
    LocalDateTime.now()
);
long hours = duration.toHours();
long minutes = duration.toMinutes();
```

**Why better than old Date/Calendar:**
- Immutable (thread-safe)
- Clear API
- Timezone support
- No deprecated methods

---

### Q71: What is the difference between parallelStream() and stream()?

**Answer:**

**stream()**: Sequential processing
```java
List<Integer> numbers = IntStream.rangeClosed(1, 1000000)
    .boxed()
    .collect(Collectors.toList());

long sum = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();  // Processes sequentially (one thread)
```

**parallelStream()**: Parallel processing
```java
long sum = numbers.parallelStream()
    .mapToInt(Integer::intValue)
    .sum();  // Processes in parallel (multiple threads)
```

**When to use:**
- **stream()**: Small datasets, order matters, stateful operations
- **parallelStream()**: Large datasets, CPU-intensive operations, independent elements

**Caution:**
```java
// BAD: Not thread-safe
List<Integer> result = new ArrayList<>();
numbers.parallelStream()
    .forEach(n -> result.add(n * 2));  // Race condition!

// GOOD: Use collect()
List<Integer> result = numbers.parallelStream()
    .map(n -> n * 2)
    .collect(Collectors.toList());  // Thread-safe
```

---

### Q72: What are the primitive specialized streams (IntStream, LongStream, DoubleStream)?

**Answer:**
Avoid boxing/unboxing overhead for primitives.

**IntStream:**
```java
// Create
IntStream.range(1, 10)        // 1 to 9
IntStream.rangeClosed(1, 10)  // 1 to 10
IntStream.of(1, 2, 3, 4, 5)

// Operations
int sum = IntStream.range(1, 11).sum();           // 55
OptionalDouble avg = IntStream.range(1, 11).average();  // 5.5
OptionalInt max = IntStream.of(1, 5, 3).max();    // 5

// Convert to Stream<Integer>
Stream<Integer> boxed = IntStream.range(1, 10).boxed();
```

**LongStream:**
```java
LongStream.range(1L, 1000000L)
    .parallel()
    .sum();
```

**DoubleStream:**
```java
double avg = DoubleStream.of(1.5, 2.5, 3.5)
    .average()
    .orElse(0.0);
```

**Convert Stream to IntStream:**
```java
List<String> words = Arrays.asList("a", "bb", "ccc");

int totalLength = words.stream()
    .mapToInt(String::length)  // Stream<String> -> IntStream
    .sum();  // 6
```

---

### Q73: What is the forEach() method difference between Stream and Collection?

**Answer:**

**Collection.forEach()**: Iteration order defined by collection
```java
List<String> list = Arrays.asList("A", "B", "C");
list.forEach(System.out::println);  // A, B, C (order preserved)
```

**Stream.forEach()**: No guaranteed order (especially parallel streams)
```java
list.stream()
    .forEach(System.out::println);  // A, B, C (usually)

list.parallelStream()
    .forEach(System.out::println);  // B, A, C (random order!)
```

**forEachOrdered()**: Preserves order even in parallel streams
```java
list.parallelStream()
    .forEachOrdered(System.out::println);  // A, B, C (order preserved)
```

---

### Q74: What is the peek() method used for?

**Answer:**
Debug intermediate stream operations (doesn't change stream).

**Example:**
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

List<Integer> result = numbers.stream()
    .peek(n -> System.out.println("Before filter: " + n))
    .filter(n -> n % 2 == 0)
    .peek(n -> System.out.println("After filter: " + n))
    .map(n -> n * 2)
    .peek(n -> System.out.println("After map: " + n))
    .collect(Collectors.toList());

// Output:
// Before filter: 1
// Before filter: 2
// After filter: 2
// After map: 4
// ...
```

**Use cases:**
- Debugging
- Logging
- Side effects (though not recommended)

---

### Q75: What are the differences between intermediate and terminal operations?

**Answer:**

**Intermediate operations (lazy, return stream):**
- filter(), map(), flatMap(), sorted(), distinct(), limit(), skip(), peek()
- Not executed until terminal operation called

```java
Stream<Integer> stream = numbers.stream()
    .filter(n -> {
        System.out.println("Filtering: " + n);
        return n > 5;
    });  // Nothing printed yet!

stream.collect(Collectors.toList());  // NOW prints filtering messages
```

**Terminal operations (eager, return result):**
- collect(), forEach(), reduce(), count(), findFirst(), findAny(), anyMatch(), allMatch(), noneMatch(), min(), max()

**Chaining:**
```java
// Intermediate → Intermediate → ... → Terminal
List<Integer> result = numbers.stream()  // Stream
    .filter(n -> n > 5)                  // Stream (intermediate)
    .map(n -> n * 2)                     // Stream (intermediate)
    .sorted()                            // Stream (intermediate)
    .collect(Collectors.toList());       // List (terminal)
```

---

### Q76: What is reduce() and how does it work?

**Answer:**
Combine stream elements to single result.

**Syntax:**
```java
T reduce(T identity, BinaryOperator<T> accumulator)
```

**Sum:**
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);  // 15

// Or method reference
int sum = numbers.stream()
    .reduce(0, Integer::sum);
```

**Product:**
```java
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);  // 120
```

**Max:**
```java
Optional<Integer> max = numbers.stream()
    .reduce((a, b) -> a > b ? a : b);

// Or
Optional<Integer> max = numbers.stream()
    .reduce(Integer::max);
```

**Concatenate strings:**
```java
List<String> words = Arrays.asList("Hello", "World");

String joined = words.stream()
    .reduce("", (a, b) -> a + " " + b);  // " Hello World"

// Better: use joining()
String joined = words.stream()
    .collect(Collectors.joining(" "));  // "Hello World"
```

**How it works:**
```java
// reduce(0, (a, b) -> a + b) on [1, 2, 3, 4]
// Step 1: result = 0 + 1 = 1
// Step 2: result = 1 + 2 = 3
// Step 3: result = 3 + 3 = 6
// Step 4: result = 6 + 4 = 10
```

---

### Q77: What is the difference between findFirst() and findAny()?

**Answer:**

**findFirst()**: Returns first element (deterministic)
```java
Optional<Integer> first = numbers.stream()
    .filter(n -> n > 5)
    .findFirst();  // Always returns same element
```

**findAny()**: Returns any element (non-deterministic in parallel)
```java
Optional<Integer> any = numbers.parallelStream()
    .filter(n -> n > 5)
    .findAny();  // May return different element each run
```

**When to use:**
- **findFirst()**: Order matters
- **findAny()**: Order doesn't matter, better performance in parallel streams

---

### Q78: What is the difference between sorted() and Comparator?

**Answer:**

**sorted() (natural order):**
```java
List<Integer> sorted = numbers.stream()
    .sorted()  // Natural order (1, 2, 3, ...)
    .collect(Collectors.toList());
```

**sorted(Comparator) (custom order):**
```java
// Reverse order
List<Integer> sorted = numbers.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());

// Custom comparator
List<Person> sorted = people.stream()
    .sorted(Comparator.comparing(Person::getAge))
    .collect(Collectors.toList());

// Multiple fields
List<Person> sorted = people.stream()
    .sorted(Comparator.comparing(Person::getAge)
                      .thenComparing(Person::getName))
    .collect(Collectors.toList());

// Null-safe
List<Person> sorted = people.stream()
    .sorted(Comparator.comparing(Person::getAge,
                                  Comparator.nullsLast(Comparator.naturalOrder())))
    .collect(Collectors.toList());
```

---

### Q79: What is the try-with-resources improvement in Java 9?

**Answer:**
Can use effectively final variables.

**Java 7-8:**
```java
BufferedReader br = new BufferedReader(new FileReader("file.txt"));
try (BufferedReader br2 = br) {  // Need to re-declare
    String line = br2.readLine();
}
```

**Java 9+:**
```java
BufferedReader br = new BufferedReader(new FileReader("file.txt"));
try (br) {  // Can use directly (if effectively final)
    String line = br.readLine();
}
```

---

### Q80: What are var (local variable type inference) in Java 10?

**Answer:**
Compiler infers type from initializer.

**Examples:**
```java
// Before Java 10
List<String> list = new ArrayList<>();
Map<Integer, String> map = new HashMap<>();

// Java 10+ (var)
var list = new ArrayList<String>();  // Type: ArrayList<String>
var map = new HashMap<Integer, String>();  // Type: HashMap<Integer, String>
var name = "Alice";  // Type: String
var age = 25;  // Type: int
```

**Limitations:**
```java
// Can't use without initializer
var x;  // ERROR

// Can't use with null
var name = null;  // ERROR

// Can't use for fields
class Person {
    var name = "Alice";  // ERROR (only for local variables)
}

// Can't use for method parameters
void method(var name) {}  // ERROR
```

**When to use:**
- Complex generic types (readability)
- Anonymous classes

**When NOT to use:**
- When type not obvious from right-hand side

---

## JDBC, Design Patterns, and Spring (Q81-Q100)

### Q81: What is JDBC and its components?

**Answer:**
Java Database Connectivity - API to connect to databases.

**Components:**

**1. DriverManager**: Manages database drivers
```java
Connection conn = DriverManager.getConnection(
    "jdbc:postgresql://localhost:5432/mydb",
    "username",
    "password"
);
```

**2. Connection**: Database connection
```java
conn.setAutoCommit(false);  // Manual transaction
conn.commit();
conn.rollback();
conn.close();
```

**3. Statement**: Execute SQL (avoid, use PreparedStatement)
```java
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT * FROM users");
```

**4. PreparedStatement**: Precompiled SQL (prevents SQL injection)
```java
String sql = "SELECT * FROM users WHERE age > ?";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setInt(1, 18);
ResultSet rs = pstmt.executeQuery();
```

**5. CallableStatement**: Execute stored procedures
```java
CallableStatement cstmt = conn.prepareCall("{call getUserById(?)}");
cstmt.setInt(1, 123);
ResultSet rs = cstmt.executeQuery();
```

**6. ResultSet**: Query results
```java
while (rs.next()) {
    int id = rs.getInt("id");
    String name = rs.getString("name");
    int age = rs.getInt("age");
}
```

---

### Q82: What is the difference between Statement, PreparedStatement, and CallableStatement?

**Answer:**

**Statement (avoid)**: Simple SQL execution
```java
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT * FROM users WHERE id = " + userId);
// SQL Injection risk!
```

**PreparedStatement (recommended)**: Precompiled SQL
```java
PreparedStatement pstmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
pstmt.setInt(1, userId);  // Safe from SQL injection
ResultSet rs = pstmt.executeQuery();

// Benefits:
// - Prevents SQL injection
// - Better performance (precompiled)
// - Handles data type conversion
```

**CallableStatement**: Stored procedures
```java
CallableStatement cstmt = conn.prepareCall("{call getUserById(?)}");
cstmt.setInt(1, userId);
ResultSet rs = cstmt.executeQuery();
```

**Comparison:**

| Feature | Statement | PreparedStatement | CallableStatement |
|---------|-----------|-------------------|-------------------|
| SQL Injection | Vulnerable | Safe | Safe |
| Performance | Slow | Fast (precompiled) | Fast |
| Use case | Simple queries | Parameterized queries | Stored procedures |

---

### Q83: How do you handle transactions in JDBC?

**Answer:**

**Default (auto-commit):**
```java
Connection conn = DriverManager.getConnection(url);
PreparedStatement pstmt = conn.prepareStatement("INSERT INTO users VALUES (?, ?)");
pstmt.setInt(1, 1);
pstmt.setString(2, "Alice");
pstmt.executeUpdate();  // Automatically committed
```

**Manual transaction:**
```java
Connection conn = DriverManager.getConnection(url);
conn.setAutoCommit(false);  // Start transaction

try {
    // Operation 1
    PreparedStatement pstmt1 = conn.prepareStatement("INSERT INTO accounts VALUES (?, ?)");
    pstmt1.setInt(1, 1);
    pstmt1.setDouble(2, 1000.0);
    pstmt1.executeUpdate();

    // Operation 2
    PreparedStatement pstmt2 = conn.prepareStatement("UPDATE balance SET amount = ? WHERE id = ?");
    pstmt2.setDouble(1, 500.0);
    pstmt2.setInt(2, 1);
    pstmt2.executeUpdate();

    // Both succeed, commit
    conn.commit();

} catch (SQLException e) {
    // Error, rollback
    conn.rollback();
    throw e;

} finally {
    conn.setAutoCommit(true);  // Restore
}
```

**Savepoints:**
```java
conn.setAutoCommit(false);

Savepoint sp1 = conn.setSavepoint("savepoint1");

try {
    // Some operations
    pstmt1.executeUpdate();

    Savepoint sp2 = conn.setSavepoint("savepoint2");

    // More operations
    pstmt2.executeUpdate();  // Error!

} catch (SQLException e) {
    conn.rollback(sp2);  // Rollback to sp2, sp1 operations preserved
}

conn.commit();
```

---

### Q84: What are JDBC best practices?

**Answer:**

**1. Use try-with-resources:**
```java
String sql = "SELECT * FROM users";

try (Connection conn = DriverManager.getConnection(url);
     PreparedStatement pstmt = conn.prepareStatement(sql);
     ResultSet rs = pstmt.executeQuery()) {

    while (rs.next()) {
        System.out.println(rs.getString("name"));
    }
} // Auto-closes all resources
```

**2. Use PreparedStatement (not Statement):**
```java
// BAD
String sql = "SELECT * FROM users WHERE name = '" + userName + "'";

// GOOD
String sql = "SELECT * FROM users WHERE name = ?";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setString(1, userName);
```

**3. Use batch operations for bulk inserts:**
```java
PreparedStatement pstmt = conn.prepareStatement("INSERT INTO users VALUES (?, ?)");

for (User user : users) {
    pstmt.setInt(1, user.getId());
    pstmt.setString(2, user.getName());
    pstmt.addBatch();
}

pstmt.executeBatch();  // Execute all at once
```

**4. Use connection pooling (HikariCP):**
```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:postgresql://localhost:5432/mydb");
config.setUsername("user");
config.setPassword("password");
config.setMaximumPoolSize(10);

HikariDataSource dataSource = new HikariDataSource(config);

// Get connection from pool
try (Connection conn = dataSource.getConnection()) {
    // Use connection
}
```

**5. Handle exceptions properly:**
```java
try (Connection conn = getConnection()) {
    // Database operations
} catch (SQLException e) {
    log.error("Database error", e);
    // Don't swallow exception
    throw new DataAccessException("Failed to fetch users", e);
}
```

---

### Q85: What is connection pooling and why use it?

**Answer:**
Reuse database connections instead of creating new ones.

**Without pooling (slow):**
```java
for (int i = 0; i < 1000; i++) {
    Connection conn = DriverManager.getConnection(url);  // Expensive!
    // Execute query
    conn.close();
}
```

**With pooling (fast):**
```java
// Create pool once
HikariDataSource pool = new HikariDataSource(config);

for (int i = 0; i < 1000; i++) {
    Connection conn = pool.getConnection();  // Reuses existing connection
    // Execute query
    conn.close();  // Returns to pool (doesn't actually close)
}
```

**Benefits:**
- **Performance**: Avoid connection overhead (10-100ms per connection)
- **Resource management**: Limit concurrent connections
- **Connection reuse**: Maintain warm connections

**Popular pools:**
- **HikariCP** (recommended, fastest)
- Apache DBCP
- C3P0

---

### Q86: Explain the Singleton design pattern

**Answer:**
Ensures only one instance of class exists.

**Eager initialization:**
```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}  // Private constructor

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

**Lazy initialization:**
```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

**Double-checked locking (best):**
```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**Enum (simplest, thread-safe):**
```java
public enum Singleton {
    INSTANCE;

    public void doSomething() {
        // Business logic
    }
}

// Usage
Singleton.INSTANCE.doSomething();
```

**Use cases**: Configuration manager, logger, database connection pool

---

### Q87: Explain the Factory design pattern

**Answer:**
Creates objects without exposing creation logic.

**Example:**
```java
// Product interface
interface Shape {
    void draw();
}

// Concrete products
class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("Drawing Circle");
    }
}

class Square implements Shape {
    @Override
    public void draw() {
        System.out.println("Drawing Square");
    }
}

// Factory
class ShapeFactory {
    public static Shape createShape(String type) {
        switch (type.toLowerCase()) {
            case "circle":
                return new Circle();
            case "square":
                return new Square();
            default:
                throw new IllegalArgumentException("Unknown shape: " + type);
        }
    }
}

// Usage
Shape shape = ShapeFactory.createShape("circle");
shape.draw();
```

**Benefits:**
- Loose coupling (client doesn't know concrete classes)
- Single place for object creation logic
- Easy to add new types

---

### Q88: Explain the Builder design pattern

**Answer:**
Constructs complex objects step-by-step.

**Example:**
```java
public class User {
    private final String firstName;  // Required
    private final String lastName;   // Required
    private final int age;           // Optional
    private final String phone;      // Optional
    private final String address;    // Optional

    private User(Builder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
        this.phone = builder.phone;
        this.address = builder.address;
    }

    public static class Builder {
        private final String firstName;
        private final String lastName;
        private int age = 0;
        private String phone = "";
        private String address = "";

        public Builder(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }

        public Builder age(int age) {
            this.age = age;
            return this;
        }

        public Builder phone(String phone) {
            this.phone = phone;
            return this;
        }

        public Builder address(String address) {
            this.address = address;
            return this;
        }

        public User build() {
            return new User(this);
        }
    }
}

// Usage
User user = new User.Builder("John", "Doe")
    .age(30)
    .phone("555-1234")
    .address("123 Main St")
    .build();
```

**Benefits:**
- Readable (named parameters)
- Immutable objects
- Flexible (optional parameters)

**Use cases**: Complex objects, many optional parameters, immutable objects

---

### Q89: Explain the Observer design pattern

**Answer:**
Notifies multiple objects when state changes.

**Example:**
```java
// Observer interface
interface Observer {
    void update(String message);
}

// Subject
class NewsAgency {
    private List<Observer> observers = new ArrayList<>();
    private String news;

    public void addObserver(Observer observer) {
        observers.add(observer);
    }

    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }

    public void setNews(String news) {
        this.news = news;
        notifyObservers();
    }

    private void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(news);
        }
    }
}

// Concrete observers
class NewsChannel implements Observer {
    private String name;

    public NewsChannel(String name) {
        this.name = name;
    }

    @Override
    public void update(String news) {
        System.out.println(name + " received news: " + news);
    }
}

// Usage
NewsAgency agency = new NewsAgency();
NewsChannel cnn = new NewsChannel("CNN");
NewsChannel bbc = new NewsChannel("BBC");

agency.addObserver(cnn);
agency.addObserver(bbc);

agency.setNews("Breaking news!");
// Output:
// CNN received news: Breaking news!
// BBC received news: Breaking news!
```

**Use cases**: Event handling, publish-subscribe, MVC pattern

---

### Q90: What is dependency injection?

**Answer:**
Providing dependencies from outside instead of creating them inside.

**Without DI (tight coupling):**
```java
class UserService {
    private UserRepository repository = new UserRepository();  // Tight coupling

    public User getUser(int id) {
        return repository.findById(id);
    }
}
```

**With DI (loose coupling):**
```java
class UserService {
    private final UserRepository repository;

    // Constructor injection
    public UserService(UserRepository repository) {
        this.repository = repository;
    }

    public User getUser(int id) {
        return repository.findById(id);
    }
}

// Usage
UserRepository repository = new UserRepository();
UserService service = new UserService(repository);
```

**Benefits:**
- Testability (can inject mock dependencies)
- Loose coupling
- Flexibility (swap implementations)

**DI in Spring:**
```java
@Service
class UserService {
    private final UserRepository repository;

    @Autowired  // Spring injects dependency
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

---

### Q91: What is the difference between @Component, @Service, @Repository, and @Controller in Spring?

**Answer:**

All are specializations of @Component with semantic meaning.

**@Component**: Generic bean
```java
@Component
public class MyComponent {
    // Generic business logic
}
```

**@Service**: Business logic layer
```java
@Service
public class UserService {
    public User getUser(int id) {
        // Business logic
    }
}
```

**@Repository**: Data access layer
```java
@Repository
public class UserRepository {
    public User findById(int id) {
        // Database access
    }
}
```

**@Controller**: Web layer (MVC)
```java
@Controller
public class UserController {
    @GetMapping("/users/{id}")
    public String getUser(@PathVariable int id, Model model) {
        // Handle web request
        return "user";
    }
}
```

**@RestController**: REST API (@Controller + @ResponseBody)
```java
@RestController
@RequestMapping("/api/users")
public class UserRestController {
    @GetMapping("/{id}")
    public User getUser(@PathVariable int id) {
        return userService.getUser(id);  // Returns JSON
    }
}
```

**Technically identical**, but convey different roles in architecture.

---

### Q92: What is Spring Boot and its advantages?

**Answer:**
Opinionated framework for creating production-ready Spring applications with minimal configuration.

**Traditional Spring (verbose):**
```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="password"/>
</bean>
```

**Spring Boot (simple):**
```properties
# application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
```

**Minimal Spring Boot app:**
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

@RestController
class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello World";
    }
}
```

**Advantages:**
- **Auto-configuration**: Automatically configures beans based on classpath
- **Embedded server**: Tomcat/Jetty embedded (no WAR deployment)
- **Starter dependencies**: spring-boot-starter-web includes everything needed
- **Production-ready**: Actuator for monitoring, health checks
- **Less boilerplate**: Minimal configuration

---

### Q93: What is Apache Spark and how to use it with Java?

**Answer:**
Distributed data processing framework.

**Setup:**
```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.12</artifactId>
    <version>3.5.0</version>
</dependency>
```

**Basic example:**
```java
SparkConf conf = new SparkConf()
    .setAppName("DataProcessing")
    .setMaster("local[*]");

JavaSparkContext sc = new JavaSparkContext(conf);

// Read data
JavaRDD<String> lines = sc.textFile("data.txt");

// Transform
JavaRDD<String> words = lines.flatMap(line -> Arrays.asList(line.split(" ")).iterator());

// Count
Map<String, Long> wordCounts = words.countByValue();

// Save
JavaPairRDD<String, Long> pairs = words.mapToPair(word -> new Tuple2<>(word, 1L));
JavaPairRDD<String, Long> counts = pairs.reduceByKey((a, b) -> a + b);
counts.saveAsTextFile("output");

sc.close();
```

**DataFrame API (recommended):**
```java
SparkSession spark = SparkSession.builder()
    .appName("DataProcessing")
    .master("local[*]")
    .getOrCreate();

// Read CSV
Dataset<Row> df = spark.read()
    .option("header", "true")
    .csv("data.csv");

// Transform
Dataset<Row> filtered = df.filter("age > 25");
Dataset<Row> grouped = df.groupBy("city").count();

// Save
filtered.write()
    .mode("overwrite")
    .parquet("output.parquet");

spark.close();
```

---

### Q94: What is Apache Kafka and how to use it with Java?

**Answer:**
Distributed event streaming platform.

**Producer:**
```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);

ProducerRecord<String, String> record = new ProducerRecord<>(
    "my-topic",      // Topic
    "key1",          // Key
    "Hello Kafka"    // Value
);

producer.send(record, (metadata, exception) -> {
    if (exception == null) {
        System.out.println("Sent to partition " + metadata.partition());
    } else {
        exception.printStackTrace();
    }
});

producer.close();
```

**Consumer:**
```java
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
        System.out.printf("Key: %s, Value: %s, Partition: %d, Offset: %d%n",
            record.key(), record.value(), record.partition(), record.offset());
    }
}
```

---

### Q95: How do you read/write JSON in Java?

**Answer:**

**Using Jackson:**
```java
// Object to JSON
ObjectMapper mapper = new ObjectMapper();

User user = new User("Alice", 25);
String json = mapper.writeValueAsString(user);
// {"name":"Alice","age":25}

// JSON to Object
String json = "{\"name\":\"Alice\",\"age\":25}";
User user = mapper.readValue(json, User.class);

// JSON file to Object
User user = mapper.readValue(new File("user.json"), User.class);

// List of objects
List<User> users = mapper.readValue(json, new TypeReference<List<User>>() {});
```

**Using Gson:**
```java
Gson gson = new Gson();

// Object to JSON
User user = new User("Alice", 25);
String json = gson.toJson(user);

// JSON to Object
User user = gson.fromJson(json, User.class);

// List
List<User> users = gson.fromJson(json, new TypeToken<List<User>>(){}.getType());
```

---

### Q96: How do you read/write CSV in Java?

**Answer:**

**Using OpenCSV:**
```java
// Read CSV
try (CSVReader reader = new CSVReader(new FileReader("data.csv"))) {
    List<String[]> rows = reader.readAll();
    for (String[] row : rows) {
        System.out.println(Arrays.toString(row));
    }
}

// Write CSV
try (CSVWriter writer = new CSVWriter(new FileWriter("output.csv"))) {
    String[] header = {"Name", "Age", "City"};
    writer.writeNext(header);

    String[] row1 = {"Alice", "25", "NYC"};
    writer.writeNext(row1);
}
```

**Using Apache Commons CSV:**
```java
// Read
try (Reader in = new FileReader("data.csv");
     CSVParser parser = CSVFormat.DEFAULT.withHeader().parse(in)) {

    for (CSVRecord record : parser) {
        String name = record.get("Name");
        String age = record.get("Age");
    }
}

// Write
try (Writer out = new FileWriter("output.csv");
     CSVPrinter printer = CSVFormat.DEFAULT.withHeader("Name", "Age").print(out)) {

    printer.printRecord("Alice", 25);
    printer.printRecord("Bob", 30);
}
```

---

### Q97: What is JVM memory management?

**Answer:**

**Memory areas:**

**1. Heap**: Objects, instance variables (shared)
- Young Generation (Eden + Survivor)
- Old Generation (Tenured)

**2. Stack**: Local variables, method calls (per thread)

**3. Metaspace**: Class metadata (Java 8+, replaces PermGen)

**4. Code Cache**: JIT-compiled native code

**Garbage Collection:**

**Minor GC**: Cleans Young Generation
```
Eden (new objects) → Survivor 0 → Survivor 1 → Old Generation
```

**Major GC**: Cleans Old Generation (slower)

**Full GC**: Cleans entire heap (stop-the-world)

**Example:**
```java
// Object created in Eden
User user = new User();  // Heap: Young Gen (Eden)

// Method call
void process() {
    int x = 10;  // Stack
    String s = new String("hello");  // s on stack, "hello" on heap
}
```

**Tuning:**
```bash
java -Xms512m -Xmx2g -XX:+UseG1GC MyApp
# -Xms: Initial heap
# -Xmx: Max heap
# -XX:+UseG1GC: Use G1 garbage collector
```

---

### Q98: What are the types of garbage collectors in Java?

**Answer:**

**1. Serial GC**: Single-threaded, small apps
```bash
-XX:+UseSerialGC
```

**2. Parallel GC**: Multi-threaded, throughput-focused
```bash
-XX:+UseParallelGC
```

**3. CMS (Concurrent Mark Sweep)**: Low pause time (deprecated)
```bash
-XX:+UseConcMarkSweepGC
```

**4. G1 GC (Garbage First)**: Balanced, default in Java 9+
```bash
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200  # Target pause time
```

**5. ZGC**: Ultra-low latency (<10ms pauses)
```bash
-XX:+UseZGC
```

**6. Shenandoah**: Low latency
```bash
-XX:+UseShenandoahGC
```

**Comparison:**

| GC | Pause Time | Throughput | Use Case |
|----|------------|------------|----------|
| Serial | High | Low | Small apps |
| Parallel | High | High | Batch processing |
| G1 | Medium | Medium | General-purpose |
| ZGC | Very Low | Medium | Low-latency apps |

---

### Q99: How do you troubleshoot memory issues in Java?

**Answer:**

**1. Heap dump:**
```bash
# Generate heap dump
jmap -dump:format=b,file=heap.hprof <pid>

# Or automatically on OutOfMemoryError
java -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp MyApp
```

**Analyze with tools:**
- Eclipse Memory Analyzer (MAT)
- VisualVM
- JProfiler

**2. GC logs:**
```bash
java -Xlog:gc*:file=gc.log MyApp
```

**3. Thread dump:**
```bash
jstack <pid> > threads.txt
```

**4. JVM metrics:**
```bash
jstat -gc <pid> 1000  # GC stats every 1 second
jstat -gcutil <pid>   # GC utilization
```

**Common issues:**
- **OutOfMemoryError: Java heap space**: Increase heap (-Xmx) or fix memory leak
- **OutOfMemoryError: Metaspace**: Increase metaspace (-XX:MaxMetaspaceSize)
- **High GC time**: Tune GC or optimize code
- **Memory leak**: Find with heap dump

---

### Q100: What are best practices for Java in data engineering?

**Answer:**

**1. Use appropriate collections:**
```java
// Large data: Use streams, don't load all in memory
Stream<String> lines = Files.lines(Paths.get("large-file.txt"));
lines.filter(line -> line.contains("error"))
     .forEach(System.out::println);
```

**2. Close resources:**
```java
try (Connection conn = getConnection();
     PreparedStatement pstmt = conn.prepareStatement(sql)) {
    // Auto-closes
}
```

**3. Use batch operations:**
```java
PreparedStatement pstmt = conn.prepareStatement("INSERT ...");
for (Record r : records) {
    pstmt.setString(1, r.value);
    pstmt.addBatch();

    if (++count % 1000 == 0) {
        pstmt.executeBatch();  // Batch every 1000
    }
}
pstmt.executeBatch();  // Final batch
```

**4. Parallelize when possible:**
```java
List<File> files = getFiles();

files.parallelStream()
    .map(file -> processFile(file))
    .collect(Collectors.toList());
```

**5. Handle errors gracefully:**
```java
try {
    processData();
} catch (Exception e) {
    log.error("Failed to process", e);
    sendAlert(e);
    // Don't swallow exceptions
    throw new DataProcessingException("Processing failed", e);
}
```

**6. Monitor performance:**
```java
long start = System.currentTimeMillis();
processData();
long duration = System.currentTimeMillis() - start;
log.info("Processing took {} ms", duration);
```

**7. Use appropriate data structures:**
- Frequent lookups: HashMap/HashSet
- Ordered data: TreeMap/TreeSet
- FIFO queue: LinkedList/ArrayDeque
- Priority queue: PriorityQueue

**8. Optimize for Spark/Kafka:**
```java
// Spark: Use DataFrame API (not RDD)
Dataset<Row> df = spark.read().parquet("data.parquet");

// Kafka: Batch sends
producer.send(record);  // Async, batched automatically
```

---

## Summary

You now have comprehensive Java knowledge covering:
- **Core Java** (OOP, classes, exceptions)
- **Collections** (List, Set, Map, performance)
- **Multithreading** (threads, synchronization, ExecutorService, concurrent collections)
- **Java 8+** (Streams, Lambda, Optional, Date/Time API)
- **JDBC** (database connectivity, transactions, best practices)
- **Design Patterns** (Singleton, Factory, Builder, Observer)
- **Spring** (Dependency Injection, Spring Boot basics)
- **Data Engineering** (Spark, Kafka, performance tuning)

**Ready for Java interviews! Good luck! ☕**
