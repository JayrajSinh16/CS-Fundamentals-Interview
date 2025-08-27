# Set Implementations - Unique Element Collections

## Navigation
- [← Previous: List Implementations](list-implementations.md)
- [🏠 Main Menu](README.md)
- [→ Next: Map Implementations](map-implementations.md)

---

## Table of Contents
1. [Set Interface Overview](#set-interface-overview)
2. [HashSet](#hashset)
3. [LinkedHashSet](#linkedhashset)
4. [TreeSet](#treeset)
5. [Performance Comparison](#performance-comparison)
6. [When to Use Which](#when-to-use-which)
7. [Interview Questions](#interview-questions)

---

## Set Interface Overview

### What is Set Interface?
The Set interface extends Collection and represents a collection that contains no duplicate elements. It models the mathematical set abstraction.

### Real-world Analogy 🏷️
Think of Set implementations like different types of **membership systems**:
- **HashSet**: Club membership cards in a box - fast to check membership, but no order
- **LinkedHashSet**: VIP list with entry order - maintains order of arrival
- **TreeSet**: Honor roll sorted by name - always keeps members sorted

### Key Set Interface Methods
```java
import java.util.*;

public class SetInterfaceDemo {
    public static void main(String[] args) {
        Set<String> set = new HashSet<>();
        
        // Adding elements
        set.add("Apple");
        set.add("Banana");
        set.add("Apple");    // Duplicate - won't be added
        
        // Checking properties
        boolean contains = set.contains("Apple");
        int size = set.size();
        boolean isEmpty = set.isEmpty();
        
        // Removing elements
        boolean removed = set.remove("Banana");
        
        // Bulk operations
        Set<String> fruits = Set.of("Cherry", "Date", "Apple");
        set.addAll(fruits);        // Union
        set.retainAll(fruits);     // Intersection
        set.removeAll(fruits);     // Difference
        
        System.out.println("Final set: " + set);
        System.out.println("Size: " + set.size());
    }
}
```

---

## HashSet

### What is HashSet?
HashSet is a hash table based implementation of Set interface. It uses hashing to store elements and provides constant time performance for basic operations.

### Internal Structure
```java
// Simplified internal structure of HashSet
public class HashSet<E> {
    private HashMap<E, Object> map;  // Uses HashMap internally
    private static final Object PRESENT = new Object();
    
    public HashSet() {
        map = new HashMap<>();
    }
    
    public boolean add(E element) {
        return map.put(element, PRESENT) == null;
    }
    
    public boolean contains(Object element) {
        return map.containsKey(element);
    }
    
    public boolean remove(Object element) {
        return map.remove(element) == PRESENT;
    }
}
```

### HashSet Features and Examples

#### 1. Basic Operations
```java
import java.util.*;

public class HashSetBasics {
    public static void main(String[] args) {
        HashSet<String> colors = new HashSet<>();
        
        // Adding elements
        colors.add("Red");
        colors.add("Green");
        colors.add("Blue");
        colors.add("Red");    // Duplicate - ignored
        
        System.out.println("Colors: " + colors);
        System.out.println("Size: " + colors.size()); // 3, not 4
        
        // Checking membership
        if (colors.contains("Red")) {
            System.out.println("Red is present");
        }
        
        // Removing elements
        colors.remove("Green");
        System.out.println("After removing Green: " + colors);
        
        // Iteration (order not guaranteed)
        System.out.println("Iterating through colors:");
        for (String color : colors) {
            System.out.println("- " + color);
        }
    }
}
```

#### 2. Custom Objects in HashSet
```java
import java.util.*;

public class HashSetCustomObjects {
    static class Person {
        private String name;
        private int age;
        
        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }
        
        public String getName() { return name; }
        public int getAge() { return age; }
        
        @Override
        public boolean equals(Object obj) {
            if (this == obj) return true;
            if (obj == null || getClass() != obj.getClass()) return false;
            Person person = (Person) obj;
            return age == person.age && Objects.equals(name, person.name);
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(name, age);
        }
        
        @Override
        public String toString() {
            return name + "(" + age + ")";
        }
    }
    
    public static void main(String[] args) {
        HashSet<Person> people = new HashSet<>();
        
        people.add(new Person("Alice", 25));
        people.add(new Person("Bob", 30));
        people.add(new Person("Alice", 25)); // Duplicate based on equals()
        
        System.out.println("People: " + people);
        System.out.println("Size: " + people.size()); // 2, not 3
        
        // Searching
        Person searchPerson = new Person("Alice", 25);
        boolean found = people.contains(searchPerson);
        System.out.println("Alice(25) found: " + found);
    }
}
```

#### 3. Set Operations
```java
import java.util.*;

public class SetOperations {
    public static void main(String[] args) {
        Set<Integer> set1 = new HashSet<>(Arrays.asList(1, 2, 3, 4, 5));
        Set<Integer> set2 = new HashSet<>(Arrays.asList(4, 5, 6, 7, 8));
        
        // Union (all elements from both sets)
        Set<Integer> union = new HashSet<>(set1);
        union.addAll(set2);
        System.out.println("Union: " + union);
        
        // Intersection (common elements)
        Set<Integer> intersection = new HashSet<>(set1);
        intersection.retainAll(set2);
        System.out.println("Intersection: " + intersection);
        
        // Difference (elements in set1 but not in set2)
        Set<Integer> difference = new HashSet<>(set1);
        difference.removeAll(set2);
        System.out.println("Difference (set1 - set2): " + difference);
        
        // Symmetric difference (elements in either set but not in both)
        Set<Integer> symmetricDiff = new HashSet<>(set1);
        symmetricDiff.addAll(set2);
        Set<Integer> temp = new HashSet<>(set1);
        temp.retainAll(set2);
        symmetricDiff.removeAll(temp);
        System.out.println("Symmetric Difference: " + symmetricDiff);
        
        // Check subset
        Set<Integer> subset = new HashSet<>(Arrays.asList(1, 2, 3));
        boolean isSubset = set1.containsAll(subset);
        System.out.println("Is {1,2,3} subset of set1: " + isSubset);
    }
}
```

---

## LinkedHashSet

### What is LinkedHashSet?
LinkedHashSet extends HashSet and maintains a doubly-linked list of entries in insertion order. It combines the benefits of HashSet (fast access) with predictable iteration order.

### Features and Examples

#### 1. Insertion Order Maintenance
```java
import java.util.*;

public class LinkedHashSetBasics {
    public static void main(String[] args) {
        // LinkedHashSet maintains insertion order
        LinkedHashSet<String> linkedSet = new LinkedHashSet<>();
        linkedSet.add("First");
        linkedSet.add("Second");
        linkedSet.add("Third");
        linkedSet.add("First"); // Duplicate - ignored but doesn't change order
        
        System.out.println("LinkedHashSet (insertion order): " + linkedSet);
        
        // Compare with HashSet (no guaranteed order)
        HashSet<String> hashSet = new HashSet<>();
        hashSet.add("First");
        hashSet.add("Second");
        hashSet.add("Third");
        
        System.out.println("HashSet (no guaranteed order): " + hashSet);
        
        // Iteration order is predictable in LinkedHashSet
        System.out.println("LinkedHashSet iteration:");
        for (String item : linkedSet) {
            System.out.println("- " + item);
        }
    }
}
```

#### 2. LRU Cache Implementation
```java
import java.util.*;

public class LRUCacheWithLinkedHashSet {
    static class LRUCache<T> {
        private final int maxSize;
        private final LinkedHashSet<T> cache;
        
        public LRUCache(int maxSize) {
            this.maxSize = maxSize;
            this.cache = new LinkedHashSet<>();
        }
        
        public void access(T item) {
            // If already present, remove and re-add to make it most recent
            if (cache.contains(item)) {
                cache.remove(item);
            }
            // If cache is full, remove least recently used (first element)
            else if (cache.size() >= maxSize) {
                T lru = cache.iterator().next();
                cache.remove(lru);
                System.out.println("Evicted: " + lru);
            }
            
            cache.add(item);
        }
        
        public void display() {
            System.out.println("Cache (LRU to MRU): " + cache);
        }
    }
    
    public static void main(String[] args) {
        LRUCache<String> cache = new LRUCache<>(3);
        
        cache.access("A");
        cache.display();
        
        cache.access("B");
        cache.display();
        
        cache.access("C");
        cache.display();
        
        cache.access("A"); // Move A to end (most recent)
        cache.display();
        
        cache.access("D"); // Should evict B
        cache.display();
    }
}
```

---

## TreeSet

### What is TreeSet?
TreeSet is a NavigableSet implementation based on a TreeMap. Elements are sorted using their natural ordering or by a Comparator provided at set creation time.

### Features and Examples

#### 1. Natural Sorting
```java
import java.util.*;

public class TreeSetBasics {
    public static void main(String[] args) {
        TreeSet<Integer> numbers = new TreeSet<>();
        
        // Add numbers in random order
        numbers.add(5);
        numbers.add(2);
        numbers.add(8);
        numbers.add(1);
        numbers.add(9);
        numbers.add(2); // Duplicate - ignored
        
        System.out.println("TreeSet (sorted): " + numbers);
        
        // TreeSet specific methods
        System.out.println("First (smallest): " + numbers.first());
        System.out.println("Last (largest): " + numbers.last());
        
        // Navigation methods
        System.out.println("Lower than 5: " + numbers.lower(5));
        System.out.println("Floor of 5: " + numbers.floor(5));
        System.out.println("Ceiling of 5: " + numbers.ceiling(5));
        System.out.println("Higher than 5: " + numbers.higher(5));
        
        // Range operations
        System.out.println("Elements < 5: " + numbers.headSet(5));
        System.out.println("Elements >= 5: " + numbers.tailSet(5));
        System.out.println("Elements [2, 8): " + numbers.subSet(2, 8));
    }
}
```

#### 2. Custom Sorting with Comparator
```java
import java.util.*;

public class TreeSetCustomSorting {
    static class Student {
        private String name;
        private double gpa;
        
        public Student(String name, double gpa) {
            this.name = name;
            this.gpa = gpa;
        }
        
        public String getName() { return name; }
        public double getGpa() { return gpa; }
        
        @Override
        public String toString() {
            return name + "(" + gpa + ")";
        }
    }
    
    public static void main(String[] args) {
        // Sort by GPA (descending)
        TreeSet<Student> studentsByGPA = new TreeSet<>(
            (s1, s2) -> Double.compare(s2.getGpa(), s1.getGpa())
        );
        
        studentsByGPA.add(new Student("Alice", 3.8));
        studentsByGPA.add(new Student("Bob", 3.5));
        studentsByGPA.add(new Student("Charlie", 3.9));
        studentsByGPA.add(new Student("David", 3.7));
        
        System.out.println("Students by GPA (descending):");
        for (Student student : studentsByGPA) {
            System.out.println(student);
        }
        
        // Sort by name length, then alphabetically
        TreeSet<Student> studentsByName = new TreeSet<>(
            Comparator.comparing((Student s) -> s.getName().length())
                     .thenComparing(Student::getName)
        );
        
        studentsByName.addAll(studentsByGPA);
        System.out.println("\nStudents by name length, then alphabetically:");
        for (Student student : studentsByName) {
            System.out.println(student);
        }
    }
}
```

#### 3. String TreeSet Operations
```java
import java.util.*;

public class TreeSetStringOperations {
    public static void main(String[] args) {
        TreeSet<String> words = new TreeSet<>();
        
        // Add words
        String[] wordArray = {"apple", "banana", "cherry", "date", "elderberry"};
        words.addAll(Arrays.asList(wordArray));
        
        System.out.println("All words: " + words);
        
        // Find words starting with specific letter
        String startLetter = "c";
        String nextLetter = "d";
        
        SortedSet<String> wordsStartingWithC = words.subSet(startLetter, nextLetter);
        System.out.println("Words starting with 'c': " + wordsStartingWithC);
        
        // Find words before 'd'
        SortedSet<String> wordsBeforeD = words.headSet("d");
        System.out.println("Words before 'd': " + wordsBeforeD);
        
        // Find words from 'd' onwards
        SortedSet<String> wordsFromD = words.tailSet("d");
        System.out.println("Words from 'd' onwards: " + wordsFromD);
        
        // Reverse iteration
        System.out.println("Reverse order:");
        Iterator<String> reverseIter = words.descendingIterator();
        while (reverseIter.hasNext()) {
            System.out.println("- " + reverseIter.next());
        }
    }
}
```

---

## Performance Comparison

### Time Complexity Comparison
| Operation | HashSet | LinkedHashSet | TreeSet |
|-----------|---------|---------------|---------|
| **Add** | O(1) | O(1) | O(log n) |
| **Remove** | O(1) | O(1) | O(log n) |
| **Contains** | O(1) | O(1) | O(log n) |
| **Iteration** | O(n) | O(n) | O(n) |
| **First/Last** | N/A | O(1) | O(log n) |
| **Navigation** | N/A | N/A | O(log n) |

### Memory Usage Comparison
```java
import java.util.*;

public class SetMemoryComparison {
    public static void main(String[] args) {
        final int SIZE = 100000;
        
        Runtime runtime = Runtime.getRuntime();
        
        // HashSet memory test
        runtime.gc();
        long beforeHashSet = runtime.totalMemory() - runtime.freeMemory();
        
        HashSet<Integer> hashSet = new HashSet<>();
        for (int i = 0; i < SIZE; i++) {
            hashSet.add(i);
        }
        
        long afterHashSet = runtime.totalMemory() - runtime.freeMemory();
        long hashSetMemory = afterHashSet - beforeHashSet;
        
        // LinkedHashSet memory test
        runtime.gc();
        long beforeLinkedHashSet = runtime.totalMemory() - runtime.freeMemory();
        
        LinkedHashSet<Integer> linkedHashSet = new LinkedHashSet<>();
        for (int i = 0; i < SIZE; i++) {
            linkedHashSet.add(i);
        }
        
        long afterLinkedHashSet = runtime.totalMemory() - runtime.freeMemory();
        long linkedHashSetMemory = afterLinkedHashSet - beforeLinkedHashSet;
        
        // TreeSet memory test
        runtime.gc();
        long beforeTreeSet = runtime.totalMemory() - runtime.freeMemory();
        
        TreeSet<Integer> treeSet = new TreeSet<>();
        for (int i = 0; i < SIZE; i++) {
            treeSet.add(i);
        }
        
        long afterTreeSet = runtime.totalMemory() - runtime.freeMemory();
        long treeSetMemory = afterTreeSet - beforeTreeSet;
        
        System.out.println("Memory Usage for " + SIZE + " integers:");
        System.out.println("HashSet: " + (hashSetMemory / 1024) + " KB");
        System.out.println("LinkedHashSet: " + (linkedHashSetMemory / 1024) + " KB");
        System.out.println("TreeSet: " + (treeSetMemory / 1024) + " KB");
    }
}
```

---

## When to Use Which

### Decision Matrix

#### Use HashSet When:
- **Fast lookup/insertion** is the primary requirement
- **Order doesn't matter**
- **Memory efficiency** is important
- **Simple duplicate elimination** is needed

#### Use LinkedHashSet When:
- **Insertion order matters**
- **Predictable iteration** is required
- **Fast lookup** is still needed
- **Cache-like behavior** is desired

#### Use TreeSet When:
- **Sorted order** is required
- **Range operations** are needed
- **Navigation methods** (first, last, ceiling, floor) are used
- **Custom sorting** logic is required

### Practical Examples
```java
import java.util.*;

public class SetUseCases {
    public static void main(String[] args) {
        // Use case 1: Remove duplicates (HashSet)
        List<String> listWithDuplicates = Arrays.asList("A", "B", "A", "C", "B", "D");
        Set<String> uniqueElements = new HashSet<>(listWithDuplicates);
        System.out.println("Original: " + listWithDuplicates);
        System.out.println("Unique (HashSet): " + uniqueElements);
        
        // Use case 2: Maintain insertion order (LinkedHashSet)
        Set<String> orderedUnique = new LinkedHashSet<>(listWithDuplicates);
        System.out.println("Unique with order (LinkedHashSet): " + orderedUnique);
        
        // Use case 3: Sorted unique elements (TreeSet)
        Set<String> sortedUnique = new TreeSet<>(listWithDuplicates);
        System.out.println("Sorted unique (TreeSet): " + sortedUnique);
        
        // Use case 4: Finding students with top grades
        demonstrateTopStudents();
    }
    
    private static void demonstrateTopStudents() {
        System.out.println("\n=== Top Students Example ===");
        
        // TreeSet automatically sorts students by GPA
        TreeSet<Student> topStudents = new TreeSet<>(
            (s1, s2) -> Double.compare(s2.getGpa(), s1.getGpa())
        );
        
        topStudents.add(new Student("Alice", 3.8));
        topStudents.add(new Student("Bob", 3.5));
        topStudents.add(new Student("Charlie", 3.9));
        topStudents.add(new Student("David", 3.7));
        topStudents.add(new Student("Eve", 3.9)); // Same GPA as Charlie
        
        System.out.println("Top 3 students:");
        topStudents.stream().limit(3).forEach(System.out::println);
        
        // Find students with GPA >= 3.7
        Student threshold = new Student("", 3.7);
        NavigableSet<Student> qualifiedStudents = topStudents.tailSet(threshold, true);
        System.out.println("\nStudents with GPA >= 3.7:");
        qualifiedStudents.forEach(System.out::println);
    }
    
    static class Student {
        private String name;
        private double gpa;
        
        public Student(String name, double gpa) {
            this.name = name;
            this.gpa = gpa;
        }
        
        public String getName() { return name; }
        public double getGpa() { return gpa; }
        
        @Override
        public String toString() {
            return name + "(" + gpa + ")";
        }
    }
}
```

---

## Interview Questions

### Q1: What is the difference between HashSet, LinkedHashSet, and TreeSet?

**Answer:**

| Aspect | HashSet | LinkedHashSet | TreeSet |
|--------|---------|---------------|---------|
| **Ordering** | No guarantee | Insertion order | Natural/Custom sorted order |
| **Performance** | O(1) for basic ops | O(1) for basic ops | O(log n) for basic ops |
| **Null values** | One null allowed | One null allowed | No nulls allowed |
| **Interface** | Set | Set | NavigableSet |
| **Internal** | HashMap | HashMap + LinkedList | Red-Black Tree |

### Q2: How does HashSet ensure uniqueness?
**Answer:**
HashSet uses `hashCode()` and `equals()` methods:

```java
public boolean add(E element) {
    // Uses hashCode() to find bucket
    // Uses equals() to check if element already exists
    return map.put(element, PRESENT) == null;
}
```

For custom objects, both methods must be properly implemented.

### Q3: Can you store null values in different Set implementations?
**Answer:**
```java
// HashSet and LinkedHashSet - one null allowed
Set<String> hashSet = new HashSet<>();
hashSet.add(null);  // OK
hashSet.add(null);  // Still OK, no duplicates

// TreeSet - no nulls allowed
Set<String> treeSet = new TreeSet<>();
// treeSet.add(null);  // Throws NullPointerException
```

### Q4: How do you convert between different Set implementations?
**Answer:**
```java
Set<String> hashSet = new HashSet<>(Arrays.asList("C", "A", "B"));
System.out.println("HashSet: " + hashSet);

// Convert to LinkedHashSet (maintains insertion order)
Set<String> linkedHashSet = new LinkedHashSet<>(hashSet);
System.out.println("LinkedHashSet: " + linkedHashSet);

// Convert to TreeSet (sorted order)
Set<String> treeSet = new TreeSet<>(hashSet);
System.out.println("TreeSet: " + treeSet);
```

### Q5: How do you implement set operations like union, intersection?
**Answer:**
```java
Set<Integer> set1 = new HashSet<>(Arrays.asList(1, 2, 3, 4));
Set<Integer> set2 = new HashSet<>(Arrays.asList(3, 4, 5, 6));

// Union
Set<Integer> union = new HashSet<>(set1);
union.addAll(set2);

// Intersection
Set<Integer> intersection = new HashSet<>(set1);
intersection.retainAll(set2);

// Difference
Set<Integer> difference = new HashSet<>(set1);
difference.removeAll(set2);

// Using Streams (Java 8+)
Set<Integer> streamUnion = Stream.concat(set1.stream(), set2.stream())
                                .collect(Collectors.toSet());
```

---

## Navigation
- [← Previous: List Implementations](list-implementations.md)
- [🏠 Main Menu](README.md)
- [→ Next: Map Implementations](map-implementations.md)

---

*This guide covers comprehensive Set implementations for Java interview preparation. Understanding the trade-offs between different Set implementations is crucial for choosing the right data structure.*
