# Map Implementations - Key-Value Pair Collections

## Navigation
- [← Previous: Set Implementations](set-implementations.md)
- [🏠 Main Menu](README.md)
- [→ Next: Queue Implementations](queue-implementations.md)

---

## Table of Contents
1. [Map Interface Overview](#map-interface-overview)
2. [HashMap](#hashmap)
3. [LinkedHashMap](#linkedhashmap)
4. [TreeMap](#treemap)
5. [Performance Comparison](#performance-comparison)
6. [When to Use Which](#when-to-use-which)
7. [Interview Questions](#interview-questions)

---

## Map Interface Overview

### What is Map Interface?
The Map interface represents a mapping between keys and values. Each key can map to at most one value, and duplicate keys are not allowed.

### Real-world Analogy 🗂️
Think of Map implementations like different types of **filing systems**:
- **HashMap**: File cabinet with quick access codes - fast retrieval but no order
- **LinkedHashMap**: Chronological filing system - maintains order of filing
- **TreeMap**: Alphabetical filing system - always keeps files sorted by name

### Key Map Interface Methods
```java
import java.util.*;

public class MapInterfaceDemo {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        
        // Adding key-value pairs
        map.put("Alice", 25);
        map.put("Bob", 30);
        map.put("Charlie", 35);
        
        // Accessing values
        Integer age = map.get("Alice");
        Integer defaultAge = map.getOrDefault("David", 0);
        
        // Checking properties
        boolean hasKey = map.containsKey("Bob");
        boolean hasValue = map.containsValue(30);
        int size = map.size();
        boolean isEmpty = map.isEmpty();
        
        // Removing entries
        Integer removed = map.remove("Charlie");
        
        // Iterating through map
        for (Map.Entry<String, Integer> entry : map.entrySet()) {
            System.out.println(entry.getKey() + " -> " + entry.getValue());
        }
        
        // Key and value sets
        Set<String> keys = map.keySet();
        Collection<Integer> values = map.values();
        
        System.out.println("Keys: " + keys);
        System.out.println("Values: " + values);
    }
}
```

---

## HashMap

### What is HashMap?
HashMap is a hash table based implementation of Map interface. It provides constant-time performance for basic operations and allows one null key and multiple null values.

### Internal Structure
```java
// Simplified internal structure of HashMap
public class HashMap<K, V> {
    static final int DEFAULT_INITIAL_CAPACITY = 16;
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    
    Node<K, V>[] table;  // Array of buckets
    int size;            // Number of key-value pairs
    int threshold;       // Resize threshold
    
    // Node structure for chaining
    static class Node<K, V> {
        final int hash;
        final K key;
        V value;
        Node<K, V> next;
        
        Node(int hash, K key, V value, Node<K, V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }
    
    // Put operation
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    
    // Hash function
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
}
```

### HashMap Features and Examples

#### 1. Basic Operations
```java
import java.util.*;

public class HashMapBasics {
    public static void main(String[] args) {
        HashMap<String, String> capitals = new HashMap<>();
        
        // Adding key-value pairs
        capitals.put("USA", "Washington DC");
        capitals.put("India", "New Delhi");
        capitals.put("Japan", "Tokyo");
        capitals.put("France", "Paris");
        
        System.out.println("Capitals: " + capitals);
        
        // Accessing values
        String usCapital = capitals.get("USA");
        String unknownCapital = capitals.getOrDefault("Brazil", "Unknown");
        
        System.out.println("US Capital: " + usCapital);
        System.out.println("Brazil Capital: " + unknownCapital);
        
        // Updating values
        capitals.put("USA", "Washington D.C."); // Updates existing value
        System.out.println("Updated USA capital: " + capitals.get("USA"));
        
        // Checking existence
        if (capitals.containsKey("India")) {
            System.out.println("India's capital: " + capitals.get("India"));
        }
        
        // Removing entries
        String removed = capitals.remove("Japan");
        System.out.println("Removed: " + removed);
        System.out.println("After removal: " + capitals);
    }
}
```

#### 2. Advanced HashMap Operations
```java
import java.util.*;

public class HashMapAdvanced {
    public static void main(String[] args) {
        HashMap<String, List<String>> studentGrades = new HashMap<>();
        
        // Initialize with empty lists
        studentGrades.put("Alice", new ArrayList<>());
        studentGrades.put("Bob", new ArrayList<>());
        
        // Add grades
        studentGrades.get("Alice").add("A");
        studentGrades.get("Alice").add("B+");
        studentGrades.get("Bob").add("A-");
        studentGrades.get("Bob").add("B");
        
        // Using computeIfAbsent for easier initialization
        studentGrades.computeIfAbsent("Charlie", k -> new ArrayList<>()).add("A+");
        studentGrades.computeIfAbsent("Charlie", k -> new ArrayList<>()).add("A");
        
        System.out.println("Student Grades: " + studentGrades);
        
        // Merge operation
        HashMap<String, Integer> scores1 = new HashMap<>();
        scores1.put("Alice", 85);
        scores1.put("Bob", 90);
        
        HashMap<String, Integer> scores2 = new HashMap<>();
        scores2.put("Bob", 95);
        scores2.put("Charlie", 88);
        
        // Merge maps - take maximum score
        scores2.forEach((key, value) -> 
            scores1.merge(key, value, Integer::max)
        );
        
        System.out.println("Merged scores: " + scores1);
        
        // Replace operations
        scores1.replace("Alice", 85, 90); // Replace if current value matches
        scores1.replaceAll((k, v) -> v + 5); // Add 5 to all scores
        
        System.out.println("After replacement: " + scores1);
    }
}
```

#### 3. Custom Objects as Keys
```java
import java.util.*;

public class HashMapCustomKeys {
    static class Student {
        private String name;
        private int id;
        
        public Student(String name, int id) {
            this.name = name;
            this.id = id;
        }
        
        public String getName() { return name; }
        public int getId() { return id; }
        
        @Override
        public boolean equals(Object obj) {
            if (this == obj) return true;
            if (obj == null || getClass() != obj.getClass()) return false;
            Student student = (Student) obj;
            return id == student.id && Objects.equals(name, student.name);
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(name, id);
        }
        
        @Override
        public String toString() {
            return name + "(" + id + ")";
        }
    }
    
    public static void main(String[] args) {
        HashMap<Student, String> studentMajors = new HashMap<>();
        
        Student alice = new Student("Alice", 101);
        Student bob = new Student("Bob", 102);
        Student charlie = new Student("Charlie", 103);
        
        studentMajors.put(alice, "Computer Science");
        studentMajors.put(bob, "Mathematics");
        studentMajors.put(charlie, "Physics");
        
        // Creating same student again
        Student aliceCopy = new Student("Alice", 101);
        String aliceMajor = studentMajors.get(aliceCopy); // Works due to proper equals/hashCode
        
        System.out.println("Alice's major: " + aliceMajor);
        System.out.println("All student majors: " + studentMajors);
        
        // Demonstrate importance of hashCode consistency
        demonstrateHashCodeImportance();
    }
    
    private static void demonstrateHashCodeImportance() {
        System.out.println("\n=== HashCode Importance Demo ===");
        
        // Bad example: mutable key
        class MutableKey {
            private String value;
            
            public MutableKey(String value) { this.value = value; }
            public String getValue() { return value; }
            public void setValue(String value) { this.value = value; }
            
            @Override
            public boolean equals(Object obj) {
                if (this == obj) return true;
                if (obj == null || getClass() != obj.getClass()) return false;
                MutableKey that = (MutableKey) obj;
                return Objects.equals(value, that.value);
            }
            
            @Override
            public int hashCode() {
                return Objects.hash(value);
            }
        }
        
        HashMap<MutableKey, String> badMap = new HashMap<>();
        MutableKey key = new MutableKey("original");
        badMap.put(key, "some value");
        
        System.out.println("Before mutation - found: " + badMap.get(key));
        
        key.setValue("modified"); // BAD: changing key after insertion
        System.out.println("After mutation - found: " + badMap.get(key)); // May return null!
        
        System.out.println("Lesson: Never modify keys after insertion in HashMap!");
    }
}
```

---

## LinkedHashMap

### What is LinkedHashMap?
LinkedHashMap extends HashMap and maintains a doubly-linked list of entries in either insertion order or access order. It provides predictable iteration order.

### Features and Examples

#### 1. Insertion Order Maintenance
```java
import java.util.*;

public class LinkedHashMapBasics {
    public static void main(String[] args) {
        // LinkedHashMap maintains insertion order
        LinkedHashMap<String, Integer> linkedMap = new LinkedHashMap<>();
        linkedMap.put("Third", 3);
        linkedMap.put("First", 1);
        linkedMap.put("Second", 2);
        linkedMap.put("Fourth", 4);
        
        System.out.println("LinkedHashMap (insertion order): " + linkedMap);
        
        // Compare with HashMap (no guaranteed order)
        HashMap<String, Integer> hashMap = new HashMap<>();
        hashMap.put("Third", 3);
        hashMap.put("First", 1);
        hashMap.put("Second", 2);
        hashMap.put("Fourth", 4);
        
        System.out.println("HashMap (no guaranteed order): " + hashMap);
        
        // Iteration order is predictable
        System.out.println("LinkedHashMap iteration:");
        for (Map.Entry<String, Integer> entry : linkedMap.entrySet()) {
            System.out.println(entry.getKey() + " -> " + entry.getValue());
        }
    }
}
```

#### 2. Access Order LinkedHashMap (LRU Cache)
```java
import java.util.*;

public class LRUCacheWithLinkedHashMap {
    static class LRUCache<K, V> extends LinkedHashMap<K, V> {
        private final int capacity;
        
        public LRUCache(int capacity) {
            // accessOrder = true for LRU behavior
            super(capacity + 1, 1.0f, true);
            this.capacity = capacity;
        }
        
        @Override
        protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
            return size() > capacity;
        }
        
        public void display() {
            System.out.println("Cache (LRU to MRU): " + this);
        }
    }
    
    public static void main(String[] args) {
        LRUCache<String, String> cache = new LRUCache<>(3);
        
        // Add entries
        cache.put("A", "Apple");
        cache.put("B", "Banana");
        cache.put("C", "Cherry");
        cache.display();
        
        // Access A (moves it to end)
        cache.get("A");
        cache.display();
        
        // Add D (should evict B - least recently used)
        cache.put("D", "Date");
        cache.display();
        
        // Access C (moves it to end)
        cache.get("C");
        cache.display();
        
        // Add E (should evict A)
        cache.put("E", "Elderberry");
        cache.display();
    }
}
```

#### 3. Insertion vs Access Order
```java
import java.util.*;

public class LinkedHashMapOrderComparison {
    public static void main(String[] args) {
        // Insertion order (default)
        LinkedHashMap<String, String> insertionOrder = new LinkedHashMap<>();
        
        // Access order
        LinkedHashMap<String, String> accessOrder = new LinkedHashMap<>(16, 0.75f, true);
        
        // Add same data to both
        String[] keys = {"A", "B", "C", "D"};
        String[] values = {"Apple", "Banana", "Cherry", "Date"};
        
        for (int i = 0; i < keys.length; i++) {
            insertionOrder.put(keys[i], values[i]);
            accessOrder.put(keys[i], values[i]);
        }
        
        System.out.println("Initial state:");
        System.out.println("Insertion order: " + insertionOrder);
        System.out.println("Access order: " + accessOrder);
        
        // Access some elements
        insertionOrder.get("B");
        insertionOrder.get("D");
        
        accessOrder.get("B");
        accessOrder.get("D");
        
        System.out.println("\nAfter accessing B and D:");
        System.out.println("Insertion order: " + insertionOrder); // No change
        System.out.println("Access order: " + accessOrder);       // B and D moved to end
    }
}
```

---

## TreeMap

### What is TreeMap?
TreeMap is a Red-Black tree based NavigableMap implementation. The map is sorted according to the natural ordering of its keys or by a Comparator provided at map creation time.

### Features and Examples

#### 1. Natural Sorting
```java
import java.util.*;

public class TreeMapBasics {
    public static void main(String[] args) {
        TreeMap<String, Integer> treeMap = new TreeMap<>();
        
        // Add entries in random order
        treeMap.put("Charlie", 30);
        treeMap.put("Alice", 25);
        treeMap.put("Bob", 35);
        treeMap.put("David", 28);
        
        System.out.println("TreeMap (sorted by key): " + treeMap);
        
        // TreeMap specific methods
        System.out.println("First key: " + treeMap.firstKey());
        System.out.println("Last key: " + treeMap.lastKey());
        System.out.println("First entry: " + treeMap.firstEntry());
        System.out.println("Last entry: " + treeMap.lastEntry());
        
        // Navigation methods
        System.out.println("Lower key than 'Charlie': " + treeMap.lowerKey("Charlie"));
        System.out.println("Floor key of 'Charlie': " + treeMap.floorKey("Charlie"));
        System.out.println("Ceiling key of 'Charlie': " + treeMap.ceilingKey("Charlie"));
        System.out.println("Higher key than 'Charlie': " + treeMap.higherKey("Charlie"));
        
        // Range operations
        System.out.println("Keys before 'Charlie': " + treeMap.headMap("Charlie"));
        System.out.println("Keys from 'Charlie': " + treeMap.tailMap("Charlie"));
        System.out.println("Keys from 'Bob' to 'David': " + treeMap.subMap("Bob", "David"));
    }
}
```

#### 2. Custom Sorting with Comparator
```java
import java.util.*;

public class TreeMapCustomSorting {
    static class Employee {
        private String name;
        private int salary;
        private String department;
        
        public Employee(String name, int salary, String department) {
            this.name = name;
            this.salary = salary;
            this.department = department;
        }
        
        public String getName() { return name; }
        public int getSalary() { return salary; }
        public String getDepartment() { return department; }
        
        @Override
        public String toString() {
            return name + "(" + salary + ", " + department + ")";
        }
    }
    
    public static void main(String[] args) {
        // Sort employees by salary (descending), then by name
        TreeMap<Employee, String> employeeRoles = new TreeMap<>(
            Comparator.comparing((Employee e) -> e.getSalary(), Comparator.reverseOrder())
                     .thenComparing(Employee::getName)
        );
        
        employeeRoles.put(new Employee("Alice", 75000, "Engineering"), "Senior Developer");
        employeeRoles.put(new Employee("Bob", 80000, "Engineering"), "Lead Developer");
        employeeRoles.put(new Employee("Charlie", 75000, "Marketing"), "Manager");
        employeeRoles.put(new Employee("David", 70000, "Engineering"), "Developer");
        
        System.out.println("Employees sorted by salary (desc), then name:");
        for (Map.Entry<Employee, String> entry : employeeRoles.entrySet()) {
            System.out.println(entry.getKey() + " -> " + entry.getValue());
        }
        
        // Find employees with salary >= 75000
        Employee threshold = new Employee("", 75000, "");
        NavigableMap<Employee, String> highEarners = employeeRoles.tailMap(threshold, true);
        
        System.out.println("\nHigh earners (>= 75000):");
        highEarners.forEach((emp, role) -> System.out.println(emp + " -> " + role));
    }
}
```

#### 3. Range Operations and Navigation
```java
import java.util.*;

public class TreeMapRangeOperations {
    public static void main(String[] args) {
        TreeMap<Integer, String> scores = new TreeMap<>();
        
        // Add test scores
        scores.put(95, "Excellent");
        scores.put(85, "Good");
        scores.put(75, "Average");
        scores.put(65, "Below Average");
        scores.put(55, "Poor");
        scores.put(90, "Very Good");
        scores.put(80, "Above Average");
        
        System.out.println("All scores: " + scores);
        
        // Range queries
        System.out.println("Scores >= 80: " + scores.tailMap(80));
        System.out.println("Scores < 80: " + scores.headMap(80));
        System.out.println("Scores 70-90: " + scores.subMap(70, 90));
        
        // Poll operations (remove and return)
        Map.Entry<Integer, String> lowest = scores.pollFirstEntry();
        Map.Entry<Integer, String> highest = scores.pollLastEntry();
        
        System.out.println("Removed lowest: " + lowest);
        System.out.println("Removed highest: " + highest);
        System.out.println("Remaining scores: " + scores);
        
        // Reverse navigation
        System.out.println("Reverse order iteration:");
        NavigableMap<Integer, String> descendingMap = scores.descendingMap();
        descendingMap.forEach((score, grade) -> 
            System.out.println(score + " -> " + grade));
    }
}
```

---

## Performance Comparison

### Time Complexity Comparison
| Operation | HashMap | LinkedHashMap | TreeMap |
|-----------|---------|---------------|---------|
| **Get** | O(1) | O(1) | O(log n) |
| **Put** | O(1) | O(1) | O(log n) |
| **Remove** | O(1) | O(1) | O(log n) |
| **ContainsKey** | O(1) | O(1) | O(log n) |
| **Iteration** | O(n) | O(n) | O(n) |
| **First/Last** | N/A | O(1) | O(log n) |
| **Range Operations** | N/A | N/A | O(log n) |

### Performance Benchmark
```java
import java.util.*;

public class MapPerformanceBenchmark {
    private static final int SIZE = 100000;
    
    public static void main(String[] args) {
        System.out.println("Map Performance Benchmark with " + SIZE + " elements\n");
        
        testPutOperations();
        testGetOperations();
        testIterationPerformance();
    }
    
    private static void testPutOperations() {
        System.out.println("=== Put Operations Test ===");
        
        // HashMap
        long start = System.nanoTime();
        HashMap<Integer, String> hashMap = new HashMap<>();
        for (int i = 0; i < SIZE; i++) {
            hashMap.put(i, "Value" + i);
        }
        long hashMapTime = System.nanoTime() - start;
        
        // LinkedHashMap
        start = System.nanoTime();
        LinkedHashMap<Integer, String> linkedHashMap = new LinkedHashMap<>();
        for (int i = 0; i < SIZE; i++) {
            linkedHashMap.put(i, "Value" + i);
        }
        long linkedHashMapTime = System.nanoTime() - start;
        
        // TreeMap
        start = System.nanoTime();
        TreeMap<Integer, String> treeMap = new TreeMap<>();
        for (int i = 0; i < SIZE; i++) {
            treeMap.put(i, "Value" + i);
        }
        long treeMapTime = System.nanoTime() - start;
        
        System.out.printf("HashMap: %.2f ms%n", hashMapTime / 1_000_000.0);
        System.out.printf("LinkedHashMap: %.2f ms%n", linkedHashMapTime / 1_000_000.0);
        System.out.printf("TreeMap: %.2f ms%n", treeMapTime / 1_000_000.0);
        System.out.println();
    }
    
    private static void testGetOperations() {
        System.out.println("=== Get Operations Test ===");
        
        // Prepare maps
        HashMap<Integer, String> hashMap = new HashMap<>();
        LinkedHashMap<Integer, String> linkedHashMap = new LinkedHashMap<>();
        TreeMap<Integer, String> treeMap = new TreeMap<>();
        
        for (int i = 0; i < SIZE; i++) {
            hashMap.put(i, "Value" + i);
            linkedHashMap.put(i, "Value" + i);
            treeMap.put(i, "Value" + i);
        }
        
        Random random = new Random();
        int accessCount = SIZE / 10;
        
        // HashMap get
        long start = System.nanoTime();
        for (int i = 0; i < accessCount; i++) {
            hashMap.get(random.nextInt(SIZE));
        }
        long hashMapTime = System.nanoTime() - start;
        
        // LinkedHashMap get
        start = System.nanoTime();
        for (int i = 0; i < accessCount; i++) {
            linkedHashMap.get(random.nextInt(SIZE));
        }
        long linkedHashMapTime = System.nanoTime() - start;
        
        // TreeMap get
        start = System.nanoTime();
        for (int i = 0; i < accessCount; i++) {
            treeMap.get(random.nextInt(SIZE));
        }
        long treeMapTime = System.nanoTime() - start;
        
        System.out.printf("HashMap: %.2f ms%n", hashMapTime / 1_000_000.0);
        System.out.printf("LinkedHashMap: %.2f ms%n", linkedHashMapTime / 1_000_000.0);
        System.out.printf("TreeMap: %.2f ms%n", treeMapTime / 1_000_000.0);
        System.out.println();
    }
    
    private static void testIterationPerformance() {
        System.out.println("=== Iteration Performance Test ===");
        
        // Prepare maps with smaller size for iteration test
        int iterSize = SIZE / 10;
        HashMap<Integer, String> hashMap = new HashMap<>();
        LinkedHashMap<Integer, String> linkedHashMap = new LinkedHashMap<>();
        TreeMap<Integer, String> treeMap = new TreeMap<>();
        
        for (int i = 0; i < iterSize; i++) {
            hashMap.put(i, "Value" + i);
            linkedHashMap.put(i, "Value" + i);
            treeMap.put(i, "Value" + i);
        }
        
        // HashMap iteration
        long start = System.nanoTime();
        for (Map.Entry<Integer, String> entry : hashMap.entrySet()) {
            // Just iterate, don't print
        }
        long hashMapTime = System.nanoTime() - start;
        
        // LinkedHashMap iteration
        start = System.nanoTime();
        for (Map.Entry<Integer, String> entry : linkedHashMap.entrySet()) {
            // Just iterate, don't print
        }
        long linkedHashMapTime = System.nanoTime() - start;
        
        // TreeMap iteration
        start = System.nanoTime();
        for (Map.Entry<Integer, String> entry : treeMap.entrySet()) {
            // Just iterate, don't print
        }
        long treeMapTime = System.nanoTime() - start;
        
        System.out.printf("HashMap: %.2f ms%n", hashMapTime / 1_000_000.0);
        System.out.printf("LinkedHashMap: %.2f ms%n", linkedHashMapTime / 1_000_000.0);
        System.out.printf("TreeMap: %.2f ms%n", treeMapTime / 1_000_000.0);
    }
}
```

---

## When to Use Which

### Decision Matrix

#### Use HashMap When:
- **Fast access** is the primary requirement
- **Order doesn't matter**
- **Memory efficiency** is important
- **General-purpose** mapping is needed

#### Use LinkedHashMap When:
- **Insertion/access order** matters
- **Predictable iteration** is required
- **Fast access** is still needed
- **Cache implementations** are being built

#### Use TreeMap When:
- **Sorted order** is required
- **Range operations** are needed
- **Navigation methods** are used frequently
- **Custom sorting** logic is required

### Practical Examples
```java
import java.util.*;

public class MapUseCases {
    public static void main(String[] args) {
        demonstrateWordFrequency();
        demonstratePriceRangeQuery();
        demonstrateOrderedConfiguration();
    }
    
    // Use case 1: Word frequency counting (HashMap)
    private static void demonstrateWordFrequency() {
        System.out.println("=== Word Frequency (HashMap) ===");
        
        String text = "the quick brown fox jumps over the lazy dog the fox is quick";
        String[] words = text.split(" ");
        
        Map<String, Integer> wordCount = new HashMap<>();
        
        for (String word : words) {
            wordCount.merge(word, 1, Integer::sum);
        }
        
        System.out.println("Word frequencies: " + wordCount);
        
        // Find most frequent word
        String mostFrequent = wordCount.entrySet().stream()
            .max(Map.Entry.comparingByValue())
            .map(Map.Entry::getKey)
            .orElse("");
        
        System.out.println("Most frequent word: " + mostFrequent);
        System.out.println();
    }
    
    // Use case 2: Price range queries (TreeMap)
    private static void demonstratePriceRangeQuery() {
        System.out.println("=== Price Range Queries (TreeMap) ===");
        
        TreeMap<Double, String> products = new TreeMap<>();
        products.put(19.99, "Book");
        products.put(299.99, "Tablet");
        products.put(899.99, "Laptop");
        products.put(49.99, "Headphones");
        products.put(129.99, "Smartwatch");
        products.put(599.99, "Smartphone");
        
        System.out.println("All products: " + products);
        
        // Products under $100
        SortedMap<Double, String> affordable = products.headMap(100.0);
        System.out.println("Products under $100: " + affordable);
        
        // Products between $100-$500
        SortedMap<Double, String> midRange = products.subMap(100.0, 500.0);
        System.out.println("Products $100-$500: " + midRange);
        
        // Most expensive product
        Map.Entry<Double, String> mostExpensive = products.lastEntry();
        System.out.println("Most expensive: " + mostExpensive);
        System.out.println();
    }
    
    // Use case 3: Configuration with insertion order (LinkedHashMap)
    private static void demonstrateOrderedConfiguration() {
        System.out.println("=== Ordered Configuration (LinkedHashMap) ===");
        
        LinkedHashMap<String, String> config = new LinkedHashMap<>();
        
        // Maintain order of configuration loading
        config.put("database.url", "jdbc:mysql://localhost:3306/app");
        config.put("database.username", "admin");
        config.put("database.password", "secret");
        config.put("server.port", "8080");
        config.put("app.name", "MyApplication");
        config.put("app.version", "1.0.0");
        
        System.out.println("Configuration (in loading order):");
        config.forEach((key, value) -> {
            // Mask passwords in output
            String displayValue = key.contains("password") ? "****" : value;
            System.out.println(key + " = " + displayValue);
        });
        
        // Update configuration while maintaining order
        config.put("app.version", "1.0.1");
        config.put("app.debug", "false"); // New entry goes to end
        
        System.out.println("\nAfter updates:");
        config.forEach((key, value) -> {
            String displayValue = key.contains("password") ? "****" : value;
            System.out.println(key + " = " + displayValue);
        });
    }
}
```

---

## Interview Questions

### Q1: What is the difference between HashMap, LinkedHashMap, and TreeMap?

**Answer:**

| Aspect | HashMap | LinkedHashMap | TreeMap |
|--------|---------|---------------|---------|
| **Ordering** | No guarantee | Insertion/Access order | Natural/Custom sorted |
| **Performance** | O(1) for basic ops | O(1) for basic ops | O(log n) for basic ops |
| **Null keys** | One null key allowed | One null key allowed | No null keys |
| **Null values** | Multiple null values | Multiple null values | Multiple null values |
| **Interface** | Map | Map | NavigableMap |
| **Internal** | Hash table | Hash table + Linked list | Red-Black Tree |

### Q2: How does HashMap handle collisions?
**Answer:**
HashMap handles collisions using **chaining** and **tree conversion**:

```java
// In Java 8+:
// 1. Initially uses linked list for collisions
// 2. When bucket size >= 8, converts to red-black tree
// 3. When tree size <= 6, converts back to linked list

// Simplified collision handling:
public V put(K key, V value) {
    int hash = hash(key);
    int index = hash & (table.length - 1);
    
    Node<K,V> node = table[index];
    if (node == null) {
        // No collision - create new node
        table[index] = new Node<>(hash, key, value, null);
    } else {
        // Collision - add to chain or tree
        // ... collision resolution logic
    }
}
```

### Q3: What is the significance of load factor in HashMap?
**Answer:**
Load factor determines when HashMap should resize:

```java
// Default load factor = 0.75
// Resize when: size > capacity * load_factor

HashMap<String, String> map = new HashMap<>(16); // Initial capacity 16
// Will resize when size > 16 * 0.75 = 12

// Trade-offs:
// High load factor (> 0.75): Less memory, more collisions
// Low load factor (< 0.75): More memory, fewer collisions
```

### Q4: Can you use mutable objects as keys in HashMap?
**Answer:**
**Not recommended** because it can break the map:

```java
class MutableKey {
    private int value;
    
    public MutableKey(int value) { this.value = value; }
    public void setValue(int value) { this.value = value; }
    
    @Override
    public int hashCode() { return value; }
    @Override
    public boolean equals(Object obj) {
        return obj instanceof MutableKey && ((MutableKey)obj).value == this.value;
    }
}

HashMap<MutableKey, String> map = new HashMap<>();
MutableKey key = new MutableKey(1);
map.put(key, "value");

System.out.println(map.get(key)); // "value"

key.setValue(2); // DANGEROUS: mutating key
System.out.println(map.get(key)); // null (key lost!)
```

### Q5: How do you iterate through a Map efficiently?
**Answer:**
Different iteration approaches:

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);
map.put("C", 3);

// 1. EntrySet (most efficient for key-value pairs)
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " -> " + entry.getValue());
}

// 2. KeySet (when only keys needed)
for (String key : map.keySet()) {
    System.out.println(key + " -> " + map.get(key)); // Less efficient
}

// 3. Values (when only values needed)
for (Integer value : map.values()) {
    System.out.println(value);
}

// 4. Java 8 forEach (modern approach)
map.forEach((key, value) -> 
    System.out.println(key + " -> " + value));

// 5. Streams (for complex processing)
map.entrySet().stream()
   .filter(entry -> entry.getValue() > 1)
   .forEach(entry -> System.out.println(entry));
```

---

## Navigation
- [← Previous: Set Implementations](set-implementations.md)
- [🏠 Main Menu](README.md)
- [→ Next: Queue Implementations](queue-implementations.md)

---

*This guide covers comprehensive Map implementations for Java interview preparation. Understanding the trade-offs between different Map implementations is essential for choosing the right data structure for specific use cases.*
