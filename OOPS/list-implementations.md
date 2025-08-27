# List Implementations - Ordered Collections

## Navigation
- [🏠 Main Menu](README.md)
- [→ Next: Set Implementations](set-implementations.md)

---

## Table of Contents
1. [List Interface Overview](#list-interface-overview)
2. [ArrayList](#arraylist)
3. [LinkedList](#linkedlist)
4. [Vector](#vector)
5. [Performance Comparison](#performance-comparison)
6. [When to Use Which](#when-to-use-which)
7. [Interview Questions](#interview-questions)

---

## List Interface Overview

### What is List Interface?
The List interface extends Collection and represents an ordered collection (sequence) that allows duplicate elements. Lists provide positional access and insertion of elements.

### Real-world Analogy 📋
Think of List implementations like different types of **notebooks**:
- **ArrayList**: Spiral notebook - fast random access to any page, but inserting pages in middle is difficult
- **LinkedList**: Ring binder - easy to insert/remove pages anywhere, but finding specific page takes time
- **Vector**: Old-fashioned bound book - same as spiral notebook but with protective cover (synchronized)

### Key List Interface Methods
```java
import java.util.*;

public class ListInterfaceDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        
        // Adding elements
        list.add("First");           // Add at end
        list.add(0, "Zero");         // Add at specific index
        list.addAll(Arrays.asList("Second", "Third"));
        
        // Accessing elements
        String first = list.get(0);          // Get by index
        int index = list.indexOf("Second");  // Find index
        
        // Modifying elements
        list.set(1, "Modified");     // Replace at index
        
        // Removing elements
        list.remove(0);              // Remove by index
        list.remove("Third");        // Remove by object
        
        // Checking properties
        boolean isEmpty = list.isEmpty();
        int size = list.size();
        boolean contains = list.contains("Second");
        
        System.out.println("Final list: " + list);
        System.out.println("Size: " + size);
    }
}
```

---

## ArrayList

### What is ArrayList?
ArrayList is a resizable array implementation of List interface. It provides dynamic arrays that can grow and shrink as needed.

### Internal Structure
```java
// Simplified internal structure of ArrayList
public class ArrayList<E> {
    private static final int DEFAULT_CAPACITY = 10;
    private Object[] elementData;  // Internal array
    private int size;              // Number of elements
    
    // Constructor creates array with default capacity
    public ArrayList() {
        this.elementData = new Object[DEFAULT_CAPACITY];
    }
    
    // Add element - resize if needed
    public boolean add(E element) {
        ensureCapacity(size + 1);
        elementData[size++] = element;
        return true;
    }
    
    // Get element by index - O(1) time
    public E get(int index) {
        return (E) elementData[index];
    }
    
    // Resize array when needed
    private void ensureCapacity(int minCapacity) {
        if (minCapacity > elementData.length) {
            int newCapacity = elementData.length * 3/2 + 1;
            elementData = Arrays.copyOf(elementData, newCapacity);
        }
    }
}
```

### ArrayList Features and Examples

#### 1. Basic Operations
```java
import java.util.*;

public class ArrayListBasics {
    public static void main(String[] args) {
        // Different ways to create ArrayList
        ArrayList<Integer> numbers = new ArrayList<>();
        ArrayList<String> names = new ArrayList<>(20); // Initial capacity
        ArrayList<String> copy = new ArrayList<>(Arrays.asList("A", "B", "C"));
        
        // Adding elements
        numbers.add(10);
        numbers.add(20);
        numbers.add(1, 15); // Insert at index 1
        System.out.println("Numbers: " + numbers); // [10, 15, 20]
        
        // Accessing elements
        int first = numbers.get(0);
        int last = numbers.get(numbers.size() - 1);
        System.out.println("First: " + first + ", Last: " + last);
        
        // Modifying elements
        numbers.set(1, 25); // Replace element at index 1
        System.out.println("After modification: " + numbers);
        
        // Finding elements
        int index = numbers.indexOf(20);
        boolean contains = numbers.contains(25);
        System.out.println("Index of 20: " + index + ", Contains 25: " + contains);
        
        // Removing elements
        numbers.remove(0);           // Remove by index
        numbers.remove(Integer.valueOf(25)); // Remove by object
        System.out.println("After removal: " + numbers);
    }
}
```

#### 2. Bulk Operations
```java
import java.util.*;

public class ArrayListBulkOps {
    public static void main(String[] args) {
        ArrayList<String> fruits = new ArrayList<>();
        fruits.addAll(Arrays.asList("Apple", "Banana", "Orange"));
        
        ArrayList<String> moreFruits = new ArrayList<>();
        moreFruits.addAll(Arrays.asList("Mango", "Grapes"));
        
        // Add all elements from another collection
        fruits.addAll(moreFruits);
        System.out.println("All fruits: " + fruits);
        
        // Add at specific index
        fruits.addAll(2, Arrays.asList("Pineapple", "Kiwi"));
        System.out.println("After insertion: " + fruits);
        
        // Remove all elements from collection
        fruits.removeAll(Arrays.asList("Banana", "Grapes"));
        System.out.println("After removal: " + fruits);
        
        // Retain only specified elements
        fruits.retainAll(Arrays.asList("Apple", "Mango", "Kiwi"));
        System.out.println("After retain: " + fruits);
        
        // Clear all elements
        moreFruits.clear();
        System.out.println("Cleared list: " + moreFruits);
    }
}
```

#### 3. ArrayList with Custom Objects
```java
import java.util.*;

public class ArrayListCustomObjects {
    static class Student {
        private String name;
        private int age;
        private double gpa;
        
        public Student(String name, int age, double gpa) {
            this.name = name;
            this.age = age;
            this.gpa = gpa;
        }
        
        // Getters
        public String getName() { return name; }
        public int getAge() { return age; }
        public double getGpa() { return gpa; }
        
        @Override
        public String toString() {
            return String.format("Student{name='%s', age=%d, gpa=%.1f}", name, age, gpa);
        }
        
        @Override
        public boolean equals(Object obj) {
            if (this == obj) return true;
            if (obj == null || getClass() != obj.getClass()) return false;
            Student student = (Student) obj;
            return Objects.equals(name, student.name);
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(name);
        }
    }
    
    public static void main(String[] args) {
        ArrayList<Student> students = new ArrayList<>();
        
        // Adding students
        students.add(new Student("Alice", 20, 3.8));
        students.add(new Student("Bob", 21, 3.5));
        students.add(new Student("Charlie", 19, 3.9));
        
        // Searching for student
        Student searchStudent = new Student("Bob", 0, 0.0); // Only name matters for equals
        int index = students.indexOf(searchStudent);
        System.out.println("Bob found at index: " + index);
        
        // Sorting students by GPA
        students.sort((s1, s2) -> Double.compare(s2.getGpa(), s1.getGpa())); // Descending
        System.out.println("Students sorted by GPA:");
        students.forEach(System.out::println);
        
        // Finding students with GPA > 3.6
        System.out.println("\nStudents with GPA > 3.6:");
        students.stream()
                .filter(s -> s.getGpa() > 3.6)
                .forEach(System.out::println);
    }
}
```

---

## LinkedList

### What is LinkedList?
LinkedList is a doubly-linked list implementation of List and Deque interfaces. Each element (node) contains data and references to both next and previous elements.

### Internal Structure
```java
// Simplified internal structure of LinkedList
public class LinkedList<E> {
    private Node<E> first;  // Reference to first node
    private Node<E> last;   // Reference to last node
    private int size;       // Number of elements
    
    // Node class
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;
        
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
    
    // Add element at end - O(1) time
    public boolean add(E element) {
        linkLast(element);
        return true;
    }
    
    // Get element by index - O(n) time
    public E get(int index) {
        return node(index).item;
    }
    
    // Find node at index
    Node<E> node(int index) {
        if (index < (size >> 1)) {
            // Search from beginning
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            // Search from end
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
}
```

### LinkedList Features and Examples

#### 1. Basic Operations
```java
import java.util.*;

public class LinkedListBasics {
    public static void main(String[] args) {
        LinkedList<String> list = new LinkedList<>();
        
        // Adding elements
        list.add("First");
        list.add("Second");
        list.addFirst("Zero");    // Add at beginning
        list.addLast("Third");    // Add at end
        list.add(2, "OneAndHalf"); // Add at index
        
        System.out.println("List: " + list);
        
        // Accessing elements
        String first = list.getFirst();
        String last = list.getLast();
        String middle = list.get(2);
        
        System.out.println("First: " + first + ", Last: " + last + ", Middle: " + middle);
        
        // Removing elements
        String removedFirst = list.removeFirst();
        String removedLast = list.removeLast();
        String removedAtIndex = list.remove(1);
        
        System.out.println("Removed - First: " + removedFirst + 
                         ", Last: " + removedLast + ", At index 1: " + removedAtIndex);
        System.out.println("Final list: " + list);
    }
}
```

#### 2. Queue and Stack Operations
```java
import java.util.*;

public class LinkedListAsQueueStack {
    public static void main(String[] args) {
        LinkedList<Integer> queue = new LinkedList<>();
        LinkedList<String> stack = new LinkedList<>();
        
        // Using as Queue (FIFO)
        System.out.println("=== Queue Operations ===");
        queue.offer(1);  // Add to rear
        queue.offer(2);
        queue.offer(3);
        
        System.out.println("Queue: " + queue);
        System.out.println("Poll (remove from front): " + queue.poll());
        System.out.println("Peek (view front): " + queue.peek());
        System.out.println("Queue after poll: " + queue);
        
        // Using as Stack (LIFO)
        System.out.println("\n=== Stack Operations ===");
        stack.push("A");  // Add to top
        stack.push("B");
        stack.push("C");
        
        System.out.println("Stack: " + stack);
        System.out.println("Pop (remove from top): " + stack.pop());
        System.out.println("Peek (view top): " + stack.peek());
        System.out.println("Stack after pop: " + stack);
        
        // Deque operations (double-ended queue)
        System.out.println("\n=== Deque Operations ===");
        LinkedList<Character> deque = new LinkedList<>();
        
        deque.addFirst('X');
        deque.addLast('Y');
        deque.addFirst('W');
        deque.addLast('Z');
        
        System.out.println("Deque: " + deque);
        System.out.println("Remove first: " + deque.removeFirst());
        System.out.println("Remove last: " + deque.removeLast());
        System.out.println("Final deque: " + deque);
    }
}
```

#### 3. Performance Demonstration
```java
import java.util.*;

public class LinkedListPerformance {
    public static void main(String[] args) {
        final int SIZE = 100000;
        
        // Insertion at beginning
        System.out.println("=== Insertion at Beginning ===");
        
        long start = System.currentTimeMillis();
        ArrayList<Integer> arrayList = new ArrayList<>();
        for (int i = 0; i < SIZE; i++) {
            arrayList.add(0, i); // Insert at beginning
        }
        long arrayListTime = System.currentTimeMillis() - start;
        
        start = System.currentTimeMillis();
        LinkedList<Integer> linkedList = new LinkedList<>();
        for (int i = 0; i < SIZE; i++) {
            linkedList.addFirst(i); // Insert at beginning
        }
        long linkedListTime = System.currentTimeMillis() - start;
        
        System.out.println("ArrayList time: " + arrayListTime + "ms");
        System.out.println("LinkedList time: " + linkedListTime + "ms");
        
        // Random access
        System.out.println("\n=== Random Access ===");
        Random random = new Random();
        
        start = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
            int index = random.nextInt(arrayList.size());
            arrayList.get(index);
        }
        arrayListTime = System.currentTimeMillis() - start;
        
        start = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
            int index = random.nextInt(linkedList.size());
            linkedList.get(index);
        }
        linkedListTime = System.currentTimeMillis() - start;
        
        System.out.println("ArrayList random access time: " + arrayListTime + "ms");
        System.out.println("LinkedList random access time: " + linkedListTime + "ms");
    }
}
```

---

## Vector

### What is Vector?
Vector is a legacy class that implements a dynamic array similar to ArrayList, but with synchronized methods making it thread-safe.

### Vector vs ArrayList
```java
import java.util.*;

public class VectorVsArrayList {
    public static void main(String[] args) {
        // Vector - synchronized (thread-safe)
        Vector<String> vector = new Vector<>();
        vector.add("Vector1");
        vector.add("Vector2");
        
        // ArrayList - not synchronized
        ArrayList<String> arrayList = new ArrayList<>();
        arrayList.add("Array1");
        arrayList.add("Array2");
        
        // Vector specific methods
        vector.addElement("Vector3");           // Similar to add()
        vector.insertElementAt("Vector0", 0);   // Similar to add(index, element)
        
        System.out.println("Vector: " + vector);
        System.out.println("Vector capacity: " + vector.capacity());
        System.out.println("Vector size: " + vector.size());
        
        // Enumeration (legacy way to iterate)
        System.out.println("\nUsing Enumeration:");
        Enumeration<String> enumeration = vector.elements();
        while (enumeration.hasMoreElements()) {
            System.out.println(enumeration.nextElement());
        }
        
        // Growth comparison
        demonstrateGrowth();
    }
    
    private static void demonstrateGrowth() {
        System.out.println("\n=== Growth Comparison ===");
        
        // Vector doubles in size when full
        Vector<Integer> vector = new Vector<>(2);
        System.out.println("Initial vector capacity: " + vector.capacity());
        
        for (int i = 0; i < 5; i++) {
            vector.add(i);
            System.out.println("After adding " + i + ", capacity: " + vector.capacity());
        }
        
        // ArrayList grows by 50%
        ArrayList<Integer> arrayList = new ArrayList<>(2);
        System.out.println("\nArrayList doesn't expose capacity, but grows by ~50%");
    }
}
```

### Thread Safety Comparison
```java
import java.util.*;
import java.util.concurrent.*;

public class ThreadSafetyDemo {
    private static final int THREAD_COUNT = 10;
    private static final int OPERATIONS_PER_THREAD = 1000;
    
    public static void main(String[] args) throws InterruptedException {
        // Test Vector (thread-safe)
        testCollection("Vector", new Vector<>());
        
        // Test ArrayList (not thread-safe)
        testCollection("ArrayList", new ArrayList<>());
        
        // Test synchronized ArrayList
        testCollection("Synchronized ArrayList", 
                      Collections.synchronizedList(new ArrayList<>()));
    }
    
    private static void testCollection(String name, List<Integer> list) 
            throws InterruptedException {
        
        ExecutorService executor = Executors.newFixedThreadPool(THREAD_COUNT);
        CountDownLatch latch = new CountDownLatch(THREAD_COUNT);
        
        long startTime = System.currentTimeMillis();
        
        for (int i = 0; i < THREAD_COUNT; i++) {
            final int threadId = i;
            executor.submit(() -> {
                try {
                    for (int j = 0; j < OPERATIONS_PER_THREAD; j++) {
                        list.add(threadId * OPERATIONS_PER_THREAD + j);
                    }
                } finally {
                    latch.countDown();
                }
            });
        }
        
        latch.await();
        executor.shutdown();
        
        long endTime = System.currentTimeMillis();
        
        System.out.println(name + ":");
        System.out.println("  Expected size: " + (THREAD_COUNT * OPERATIONS_PER_THREAD));
        System.out.println("  Actual size: " + list.size());
        System.out.println("  Time taken: " + (endTime - startTime) + "ms");
        System.out.println("  Data integrity: " + 
                          (list.size() == THREAD_COUNT * OPERATIONS_PER_THREAD ? "PASSED" : "FAILED"));
        System.out.println();
    }
}
```

---

## Performance Comparison

### Time Complexity Comparison
| Operation | ArrayList | LinkedList | Vector |
|-----------|-----------|------------|--------|
| **Access (get/set)** | O(1) | O(n) | O(1) |
| **Insert at end** | O(1) amortized | O(1) | O(1) amortized |
| **Insert at beginning** | O(n) | O(1) | O(n) |
| **Insert at middle** | O(n) | O(n) | O(n) |
| **Remove from end** | O(1) | O(1) | O(1) |
| **Remove from beginning** | O(n) | O(1) | O(n) |
| **Search** | O(n) | O(n) | O(n) |

### Memory Usage Comparison
```java
import java.util.*;

public class MemoryComparison {
    public static void main(String[] args) {
        final int SIZE = 1000000;
        
        // Memory usage demonstration
        Runtime runtime = Runtime.getRuntime();
        
        // ArrayList memory usage
        runtime.gc();
        long beforeArrayList = runtime.totalMemory() - runtime.freeMemory();
        
        ArrayList<Integer> arrayList = new ArrayList<>();
        for (int i = 0; i < SIZE; i++) {
            arrayList.add(i);
        }
        
        long afterArrayList = runtime.totalMemory() - runtime.freeMemory();
        long arrayListMemory = afterArrayList - beforeArrayList;
        
        // LinkedList memory usage
        runtime.gc();
        long beforeLinkedList = runtime.totalMemory() - runtime.freeMemory();
        
        LinkedList<Integer> linkedList = new LinkedList<>();
        for (int i = 0; i < SIZE; i++) {
            linkedList.add(i);
        }
        
        long afterLinkedList = runtime.totalMemory() - runtime.freeMemory();
        long linkedListMemory = afterLinkedList - beforeLinkedList;
        
        System.out.println("Memory Usage Comparison for " + SIZE + " integers:");
        System.out.println("ArrayList: " + (arrayListMemory / 1024 / 1024) + " MB");
        System.out.println("LinkedList: " + (linkedListMemory / 1024 / 1024) + " MB");
        System.out.println("LinkedList uses ~" + (linkedListMemory / arrayListMemory) + "x more memory");
        
        // Explain why
        System.out.println("\nReason:");
        System.out.println("- ArrayList: Only stores the actual data in contiguous array");
        System.out.println("- LinkedList: Each node stores data + 2 pointers (next/prev) + object overhead");
    }
}
```

### Performance Benchmark
```java
import java.util.*;

public class ListPerformanceBenchmark {
    private static final int SIZE = 100000;
    
    public static void main(String[] args) {
        System.out.println("Performance Benchmark with " + SIZE + " elements\n");
        
        // Test different operations
        testSequentialAdd();
        testRandomAccess();
        testInsertionAtBeginning();
        testRemovalFromMiddle();
    }
    
    private static void testSequentialAdd() {
        System.out.println("=== Sequential Add Test ===");
        
        long start = System.nanoTime();
        ArrayList<Integer> arrayList = new ArrayList<>();
        for (int i = 0; i < SIZE; i++) {
            arrayList.add(i);
        }
        long arrayListTime = System.nanoTime() - start;
        
        start = System.nanoTime();
        LinkedList<Integer> linkedList = new LinkedList<>();
        for (int i = 0; i < SIZE; i++) {
            linkedList.add(i);
        }
        long linkedListTime = System.nanoTime() - start;
        
        start = System.nanoTime();
        Vector<Integer> vector = new Vector<>();
        for (int i = 0; i < SIZE; i++) {
            vector.add(i);
        }
        long vectorTime = System.nanoTime() - start;
        
        System.out.printf("ArrayList: %.2f ms%n", arrayListTime / 1_000_000.0);
        System.out.printf("LinkedList: %.2f ms%n", linkedListTime / 1_000_000.0);
        System.out.printf("Vector: %.2f ms%n", vectorTime / 1_000_000.0);
        System.out.println();
    }
    
    private static void testRandomAccess() {
        System.out.println("=== Random Access Test ===");
        
        // Prepare lists
        ArrayList<Integer> arrayList = new ArrayList<>();
        LinkedList<Integer> linkedList = new LinkedList<>();
        Vector<Integer> vector = new Vector<>();
        
        for (int i = 0; i < SIZE; i++) {
            arrayList.add(i);
            linkedList.add(i);
            vector.add(i);
        }
        
        Random random = new Random();
        int accessCount = SIZE / 10;
        
        // ArrayList random access
        long start = System.nanoTime();
        for (int i = 0; i < accessCount; i++) {
            arrayList.get(random.nextInt(SIZE));
        }
        long arrayListTime = System.nanoTime() - start;
        
        // LinkedList random access
        start = System.nanoTime();
        for (int i = 0; i < accessCount; i++) {
            linkedList.get(random.nextInt(SIZE));
        }
        long linkedListTime = System.nanoTime() - start;
        
        // Vector random access
        start = System.nanoTime();
        for (int i = 0; i < accessCount; i++) {
            vector.get(random.nextInt(SIZE));
        }
        long vectorTime = System.nanoTime() - start;
        
        System.out.printf("ArrayList: %.2f ms%n", arrayListTime / 1_000_000.0);
        System.out.printf("LinkedList: %.2f ms%n", linkedListTime / 1_000_000.0);
        System.out.printf("Vector: %.2f ms%n", vectorTime / 1_000_000.0);
        System.out.println();
    }
    
    private static void testInsertionAtBeginning() {
        System.out.println("=== Insertion at Beginning Test ===");
        
        int insertCount = SIZE / 100; // Smaller number due to O(n) complexity
        
        // ArrayList
        long start = System.nanoTime();
        ArrayList<Integer> arrayList = new ArrayList<>();
        for (int i = 0; i < insertCount; i++) {
            arrayList.add(0, i);
        }
        long arrayListTime = System.nanoTime() - start;
        
        // LinkedList
        start = System.nanoTime();
        LinkedList<Integer> linkedList = new LinkedList<>();
        for (int i = 0; i < insertCount; i++) {
            linkedList.addFirst(i);
        }
        long linkedListTime = System.nanoTime() - start;
        
        System.out.printf("ArrayList: %.2f ms%n", arrayListTime / 1_000_000.0);
        System.out.printf("LinkedList: %.2f ms%n", linkedListTime / 1_000_000.0);
        System.out.println();
    }
    
    private static void testRemovalFromMiddle() {
        System.out.println("=== Removal from Middle Test ===");
        
        // Prepare lists
        ArrayList<Integer> arrayList = new ArrayList<>();
        LinkedList<Integer> linkedList = new LinkedList<>();
        
        for (int i = 0; i < SIZE; i++) {
            arrayList.add(i);
            linkedList.add(i);
        }
        
        int removeCount = SIZE / 100;
        
        // ArrayList removal
        long start = System.nanoTime();
        for (int i = 0; i < removeCount; i++) {
            arrayList.remove(arrayList.size() / 2);
        }
        long arrayListTime = System.nanoTime() - start;
        
        // LinkedList removal
        start = System.nanoTime();
        for (int i = 0; i < removeCount; i++) {
            linkedList.remove(linkedList.size() / 2);
        }
        long linkedListTime = System.nanoTime() - start;
        
        System.out.printf("ArrayList: %.2f ms%n", arrayListTime / 1_000_000.0);
        System.out.printf("LinkedList: %.2f ms%n", linkedListTime / 1_000_000.0);
        System.out.println();
    }
}
```

---

## When to Use Which

### Decision Matrix

#### Use ArrayList When:
- **Frequent random access** to elements by index
- **More reads than writes** in your application
- **Memory efficiency** is important
- **Cache performance** matters (spatial locality)
- **Simple iteration** through all elements

#### Use LinkedList When:
- **Frequent insertions/deletions** at beginning or middle
- **Queue/Stack operations** are primary use case
- **Size varies dramatically** during runtime
- **Memory is not a primary concern**
- **Implementing other data structures** (stacks, queues)

#### Use Vector When:
- **Legacy code compatibility** is required
- **Built-in thread safety** is needed (though Collections.synchronizedList() is preferred)
- **Simple thread-safe operations** without complex synchronization

### Practical Examples

#### 1. Data Processing Pipeline
```java
import java.util.*;

public class DataProcessingExample {
    // Use ArrayList for batch processing with random access
    public static void processDataBatch(List<String> data) {
        ArrayList<String> processedData = new ArrayList<>(data.size());
        
        // Efficient random access and iteration
        for (int i = 0; i < data.size(); i++) {
            String processed = data.get(i).toUpperCase().trim();
            processedData.add(processed);
        }
        
        // Efficient bulk operations
        processedData.removeAll(Arrays.asList("", null));
        
        System.out.println("Processed " + processedData.size() + " items");
    }
    
    // Use LinkedList for streaming data with frequent insertions
    public static void processStreamingData() {
        LinkedList<String> buffer = new LinkedList<>();
        
        // Simulate streaming data - frequent additions/removals
        for (int i = 0; i < 100; i++) {
            buffer.addLast("Data-" + i);
            
            // Process in batches of 10
            if (buffer.size() >= 10) {
                // Remove from front efficiently
                while (buffer.size() > 5) {
                    String data = buffer.removeFirst();
                    System.out.println("Processing: " + data);
                }
            }
        }
    }
    
    public static void main(String[] args) {
        List<String> batchData = Arrays.asList("apple", "banana", "cherry", "date");
        processDataBatch(batchData);
        
        System.out.println("\n--- Streaming Processing ---");
        processStreamingData();
    }
}
```

#### 2. Cache Implementation
```java
import java.util.*;

public class CacheImplementation {
    // LRU Cache using LinkedList for O(1) move operations
    static class LRUCache<K, V> {
        private final int capacity;
        private final LinkedHashMap<K, V> cache;
        
        public LRUCache(int capacity) {
            this.capacity = capacity;
            this.cache = new LinkedHashMap<K, V>(capacity, 0.75f, true) {
                @Override
                protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                    return size() > capacity;
                }
            };
        }
        
        public V get(K key) {
            return cache.get(key); // Moves to end automatically
        }
        
        public void put(K key, V value) {
            cache.put(key, value);
        }
        
        public void display() {
            System.out.println("Cache contents: " + cache);
        }
    }
    
    // Simple cache using ArrayList for direct access
    static class SimpleCache<T> {
        private final ArrayList<T> cache;
        private final int maxSize;
        
        public SimpleCache(int maxSize) {
            this.maxSize = maxSize;
            this.cache = new ArrayList<>(maxSize);
        }
        
        public void add(T item) {
            if (cache.size() >= maxSize) {
                cache.remove(0); // Remove oldest (least efficient for ArrayList)
            }
            cache.add(item);
        }
        
        public T get(int index) {
            return cache.get(index);
        }
        
        public void display() {
            System.out.println("Simple cache: " + cache);
        }
    }
    
    public static void main(String[] args) {
        // LRU Cache demo
        System.out.println("=== LRU Cache Demo ===");
        LRUCache<String, String> lruCache = new LRUCache<>(3);
        
        lruCache.put("A", "Apple");
        lruCache.put("B", "Banana");
        lruCache.put("C", "Cherry");
        lruCache.display();
        
        lruCache.get("A"); // Move A to end (most recently used)
        lruCache.put("D", "Date"); // Should evict B
        lruCache.display();
        
        // Simple cache demo
        System.out.println("\n=== Simple Cache Demo ===");
        SimpleCache<String> simpleCache = new SimpleCache<>(3);
        
        simpleCache.add("First");
        simpleCache.add("Second");
        simpleCache.add("Third");
        simpleCache.display();
        
        simpleCache.add("Fourth"); // Should remove "First"
        simpleCache.display();
    }
}
```

---

## Interview Questions

### Q1: What is the difference between ArrayList and LinkedList?
**Answer:**

| Aspect | ArrayList | LinkedList |
|--------|-----------|------------|
| **Internal Structure** | Dynamic array | Doubly linked list |
| **Memory** | Contiguous memory allocation | Scattered memory with node overhead |
| **Access Time** | O(1) random access | O(n) sequential access |
| **Insertion/Deletion** | O(n) at beginning/middle, O(1) at end | O(1) if position known, O(n) to find position |
| **Memory Overhead** | Low (only array resize) | High (each node has prev/next pointers) |
| **Cache Performance** | Better (spatial locality) | Worse (scattered memory) |

**Code Example:**
```java
// ArrayList - good for random access
List<String> arrayList = new ArrayList<>();
arrayList.add("A"); arrayList.add("B"); arrayList.add("C");
String element = arrayList.get(1); // O(1) - very fast

// LinkedList - good for frequent insertions
List<String> linkedList = new LinkedList<>();
linkedList.add("A"); linkedList.add("B"); linkedList.add("C");
linkedList.add(0, "Start"); // O(1) - very fast at beginning
```

### Q2: When would you use Vector instead of ArrayList?
**Answer:**
Vector should be used when:
1. **Legacy compatibility** is required
2. **Built-in thread safety** is needed
3. **Simple synchronized operations** without complex locking

However, **modern alternatives are preferred**:
```java
// Instead of Vector, use:
List<String> synchronizedList = Collections.synchronizedList(new ArrayList<>());

// Or for better performance in concurrent scenarios:
List<String> concurrentList = new CopyOnWriteArrayList<>();
```

### Q3: How does ArrayList handle capacity management?
**Answer:**
ArrayList manages capacity through:

1. **Initial Capacity**: Default 10, or specified in constructor
2. **Growth Strategy**: Increases by ~50% when full
3. **Trimming**: Can manually trim with `trimToSize()`

```java
ArrayList<Integer> list = new ArrayList<>(5); // Initial capacity 5

// When 6th element added:
// Old capacity: 5
// New capacity: 5 + (5/2) = 7 (approximately 1.5x growth)

// Manual optimization
list.trimToSize(); // Reduces capacity to exact size
```

### Q4: How do you make ArrayList thread-safe?
**Answer:**
Multiple approaches:

1. **Collections.synchronizedList()**:
```java
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
// Need external synchronization for iteration
synchronized(syncList) {
    for(String item : syncList) {
        System.out.println(item);
    }
}
```

2. **CopyOnWriteArrayList**:
```java
List<String> cowList = new CopyOnWriteArrayList<>();
// Thread-safe for all operations, but expensive writes
```

3. **Manual synchronization**:
```java
public class ThreadSafeList<T> {
    private final List<T> list = new ArrayList<>();
    
    public synchronized void add(T item) { list.add(item); }
    public synchronized T get(int index) { return list.get(index); }
    // ... other synchronized methods
}
```

### Q5: What happens when you modify a list during iteration?
**Answer:**
Modifying a list during iteration can cause **ConcurrentModificationException**:

```java
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C", "D"));

// This will throw ConcurrentModificationException
try {
    for (String item : list) {
        if (item.equals("B")) {
            list.remove(item); // Modification during iteration
        }
    }
} catch (ConcurrentModificationException e) {
    System.out.println("Exception: " + e.getMessage());
}

// Safe approaches:
// 1. Use Iterator.remove()
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    String item = iterator.next();
    if (item.equals("B")) {
        iterator.remove(); // Safe removal
    }
}

// 2. Use removeIf() (Java 8+)
list.removeIf(item -> item.equals("C"));

// 3. Collect indices and remove in reverse order
List<Integer> toRemove = new ArrayList<>();
for (int i = 0; i < list.size(); i++) {
    if (list.get(i).equals("D")) {
        toRemove.add(i);
    }
}
for (int i = toRemove.size() - 1; i >= 0; i--) {
    list.remove((int)toRemove.get(i));
}
```

### Q6: How do you convert between different List implementations?
**Answer:**
```java
// Original ArrayList
List<String> arrayList = new ArrayList<>(Arrays.asList("A", "B", "C"));

// Convert to LinkedList
List<String> linkedList = new LinkedList<>(arrayList);

// Convert to Vector
Vector<String> vector = new Vector<>(arrayList);

// Convert to Array
String[] array = arrayList.toArray(new String[0]);

// Convert back to List
List<String> fromArray = Arrays.asList(array);
List<String> mutableFromArray = new ArrayList<>(Arrays.asList(array));

// Using streams (Java 8+)
List<String> upperCaseList = arrayList.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toCollection(LinkedList::new));
```

### Q7: What is the difference between capacity and size in ArrayList?
**Answer:**
- **Size**: Number of elements currently in the list
- **Capacity**: Total number of elements the internal array can hold

```java
ArrayList<String> list = new ArrayList<>(20); // Initial capacity 20
System.out.println("Initial size: " + list.size());     // 0
// Capacity is internal, not directly accessible

list.add("A");
list.add("B");
System.out.println("Size after adding 2 elements: " + list.size()); // 2
// Capacity is still 20

// When capacity is exceeded, ArrayList automatically grows
for (int i = 0; i < 25; i++) {
    list.add("Item" + i);
}
// Internal capacity automatically increased (likely to 30: 20 * 1.5)

// Optimize memory usage
list.trimToSize(); // Reduces capacity to match size exactly
```

---

## Navigation
- [🏠 Main Menu](README.md)
- [→ Next: Set Implementations](set-implementations.md)

---

*This guide covers comprehensive List implementations for Java interview preparation. Understanding when to use each implementation and their performance characteristics is crucial for technical interviews.*
