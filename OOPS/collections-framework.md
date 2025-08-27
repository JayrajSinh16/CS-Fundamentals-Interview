# 📚 Collections Framework - Lists, Sets, Maps & Iteration

Java Collections Framework provides a unified architecture for storing and manipulating groups of objects. It includes interfaces, implementations, and algorithms for common data structures, enabling efficient and type-safe data handling in Java applications.

## 🎯 Key Concepts

- **Collection Hierarchy**: Interfaces and their implementations
- **List**: Ordered collections allowing duplicates
- **Set**: Collections with unique elements
- **Map**: Key-value pair collections
- **Iteration**: Different ways to traverse collections
- **Algorithms**: Utility methods for sorting, searching
- **Generics**: Type safety with parameterized types

## 🌍 Real-World Analogy

**Library Management System**:
- **List** = Book catalog (ordered, can have duplicate titles)
- **Set** = Unique ISBN numbers (no duplicates allowed)
- **Map** = Book ID to Book details mapping
- **Iterator** = Librarian walking through shelves systematically
- **Algorithms** = Sorting books by author, searching by title

## 📋 Collections Framework Architecture

```
Collections Hierarchy:
┌─────────────────────────────────────────────────────────────────┐
│                    Collection Interface Tree                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                    Collection<E>                                │
│                         │                                       │
│         ┌───────────────┼───────────────┐                       │
│         │               │               │                       │
│      List<E>         Set<E>          Queue<E>                   │
│         │               │               │                       │
│    ┌────┴────┐     ┌────┴────┐     ┌────┴────┐                 │
│    │         │     │         │     │         │                 │
│ ArrayList  Vector  HashSet TreeSet LinkedList PriorityQueue     │
│ LinkedList Stack  LinkedHashSet   ArrayDeque                   │
│                   EnumSet                                       │
│                                                                 │
│                        Map<K,V> (separate hierarchy)           │
│                           │                                     │
│              ┌────────────┼────────────┐                       │
│              │            │            │                       │
│          HashMap     TreeMap     LinkedHashMap                 │
│          Hashtable   EnumMap     WeakHashMap                   │
│          Properties  IdentityHashMap                           │
└─────────────────────────────────────────────────────────────────┘

Common Implementations Comparison:
┌─────────────────────────────────────────────────────────────────┐
│  Type     │ Implementation │ Access │ Insert │ Delete │ Memory  │
│  ────────────────────────────────────────────────────────────── │
│  List     │ ArrayList      │ O(1)   │ O(1)*  │ O(n)   │ Dynamic │
│           │ LinkedList     │ O(n)   │ O(1)   │ O(1)   │ Node    │
│           │ Vector         │ O(1)   │ O(1)*  │ O(n)   │ Sync    │
│  ────────────────────────────────────────────────────────────── │
│  Set      │ HashSet        │ O(1)   │ O(1)   │ O(1)   │ Hash    │
│           │ TreeSet        │ O(log n)│O(log n)│O(log n)│ Tree    │
│           │ LinkedHashSet  │ O(1)   │ O(1)   │ O(1)   │ Order   │
│  ────────────────────────────────────────────────────────────── │
│  Map      │ HashMap        │ O(1)   │ O(1)   │ O(1)   │ Hash    │
│           │ TreeMap        │ O(log n)│O(log n)│O(log n)│ Sorted  │
│           │ LinkedHashMap  │ O(1)   │ O(1)   │ O(1)   │ Order   │
│                                                                 │
│  * Amortized time complexity                                    │
└─────────────────────────────────────────────────────────────────┘
```

## 💻 Practical Examples

### Example 1: List Implementations

```java
// ListImplementations.java
import java.util.*;

public class ListImplementations {
    
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
            return age == student.age && 
                   Double.compare(student.gpa, gpa) == 0 && 
                   Objects.equals(name, student.name);
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(name, age, gpa);
        }
    }
    
    public static void demonstrateListImplementations() {
        System.out.println("=== List Implementations ===");
        
        // ArrayList - Dynamic array
        System.out.println("\n--- ArrayList Example ---");
        List<Student> arrayList = new ArrayList<>();
        
        // Adding elements
        arrayList.add(new Student("Alice", 20, 3.8));
        arrayList.add(new Student("Bob", 22, 3.5));
        arrayList.add(new Student("Charlie", 21, 3.9));
        arrayList.add(1, new Student("David", 19, 3.7)); // Insert at index
        
        System.out.println("ArrayList contents:");
        for (int i = 0; i < arrayList.size(); i++) {
            System.out.printf("Index %d: %s%n", i, arrayList.get(i));
        }
        
        // LinkedList - Doubly linked list
        System.out.println("\n--- LinkedList Example ---");
        LinkedList<Student> linkedList = new LinkedList<>();
        
        // Adding elements
        linkedList.addFirst(new Student("Eve", 23, 3.6));
        linkedList.addLast(new Student("Frank", 20, 3.4));
        linkedList.add(new Student("Grace", 22, 4.0));
        
        System.out.println("LinkedList contents:");
        System.out.println("First: " + linkedList.getFirst());
        System.out.println("Last: " + linkedList.getLast());
        
        // List operations
        System.out.println("\n--- List Operations ---");
        List<String> fruits = Arrays.asList("Apple", "Banana", "Cherry", "Apple");
        
        // Searching
        System.out.println("Index of Apple: " + fruits.indexOf("Apple"));
        System.out.println("Last index of Apple: " + fruits.lastIndexOf("Apple"));
        System.out.println("Contains Banana: " + fruits.contains("Banana"));
        
        // Sublist
        List<String> subFruits = fruits.subList(1, 3);
        System.out.println("Sublist (1-3): " + subFruits);
        
        // List iteration methods
        System.out.println("\n--- Iteration Methods ---");
        
        // Traditional for loop
        System.out.println("Traditional for loop:");
        for (int i = 0; i < fruits.size(); i++) {
            System.out.printf("  [%d] %s%n", i, fruits.get(i));
        }
        
        // Enhanced for loop
        System.out.println("Enhanced for loop:");
        for (String fruit : fruits) {
            System.out.println("  " + fruit);
        }
        
        // Iterator
        System.out.println("Iterator:");
        Iterator<String> iterator = fruits.iterator();
        while (iterator.hasNext()) {
            System.out.println("  " + iterator.next());
        }
        
        // Stream API
        System.out.println("Stream API:");
        fruits.stream()
              .filter(fruit -> fruit.startsWith("A"))
              .forEach(fruit -> System.out.println("  " + fruit));
    }
    
    public static void main(String[] args) {
        demonstrateListImplementations();
    }
}
```

### Example 2: Set Implementations

```java
// SetImplementations.java
import java.util.*;

public class SetImplementations {
    
    static class Book implements Comparable<Book> {
        private String title;
        private String author;
        private int year;
        private String isbn;
        
        public Book(String title, String author, int year, String isbn) {
            this.title = title;
            this.author = author;
            this.year = year;
            this.isbn = isbn;
        }
        
        @Override
        public int compareTo(Book other) {
            // Primary sort by author, secondary by title
            int authorComparison = this.author.compareTo(other.author);
            return authorComparison != 0 ? authorComparison : this.title.compareTo(other.title);
        }
        
        @Override
        public boolean equals(Object obj) {
            if (this == obj) return true;
            if (obj == null || getClass() != obj.getClass()) return false;
            Book book = (Book) obj;
            return Objects.equals(isbn, book.isbn); // Books equal if same ISBN
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(isbn);
        }
        
        @Override
        public String toString() {
            return String.format("'%s' by %s (%d) [%s]", title, author, year, isbn);
        }
        
        // Getters
        public String getTitle() { return title; }
        public String getAuthor() { return author; }
        public int getYear() { return year; }
        public String getIsbn() { return isbn; }
    }
    
    public static void demonstrateSetImplementations() {
        System.out.println("=== Set Implementations ===");
        
        // HashSet - No ordering, O(1) operations
        System.out.println("\n--- HashSet Example ---");
        Set<Book> hashSet = new HashSet<>();
        
        hashSet.add(new Book("1984", "George Orwell", 1949, "978-0-452-28423-4"));
        hashSet.add(new Book("To Kill a Mockingbird", "Harper Lee", 1960, "978-0-06-112008-4"));
        hashSet.add(new Book("The Great Gatsby", "F. Scott Fitzgerald", 1925, "978-0-7432-7356-5"));
        hashSet.add(new Book("1984", "George Orwell", 1949, "978-0-452-28423-4")); // Duplicate
        
        System.out.println("HashSet size: " + hashSet.size());
        System.out.println("HashSet contents (no guaranteed order):");
        hashSet.forEach(book -> System.out.println("  " + book));
        
        // TreeSet - Natural ordering, O(log n) operations
        System.out.println("\n--- TreeSet Example ---");
        Set<Book> treeSet = new TreeSet<>();
        
        treeSet.add(new Book("Pride and Prejudice", "Jane Austen", 1813, "978-0-14-143951-8"));
        treeSet.add(new Book("Animal Farm", "George Orwell", 1945, "978-0-452-28424-1"));
        treeSet.add(new Book("Emma", "Jane Austen", 1815, "978-0-14-143955-6"));
        treeSet.add(new Book("Brave New World", "Aldous Huxley", 1932, "978-0-06-085052-4"));
        
        System.out.println("TreeSet contents (sorted by author, then title):");
        treeSet.forEach(book -> System.out.println("  " + book));
        
        // LinkedHashSet - Insertion order, O(1) operations
        System.out.println("\n--- LinkedHashSet Example ---");
        Set<String> linkedHashSet = new LinkedHashSet<>();
        
        linkedHashSet.add("First");
        linkedHashSet.add("Second");
        linkedHashSet.add("Third");
        linkedHashSet.add("First"); // Duplicate, won't be added
        
        System.out.println("LinkedHashSet (maintains insertion order):");
        linkedHashSet.forEach(item -> System.out.println("  " + item));
        
        // Set operations
        System.out.println("\n--- Set Operations ---");
        Set<String> set1 = new HashSet<>(Arrays.asList("A", "B", "C", "D"));
        Set<String> set2 = new HashSet<>(Arrays.asList("C", "D", "E", "F"));
        
        // Union
        Set<String> union = new HashSet<>(set1);
        union.addAll(set2);
        System.out.println("Union: " + union);
        
        // Intersection
        Set<String> intersection = new HashSet<>(set1);
        intersection.retainAll(set2);
        System.out.println("Intersection: " + intersection);
        
        // Difference
        Set<String> difference = new HashSet<>(set1);
        difference.removeAll(set2);
        System.out.println("Difference (set1 - set2): " + difference);
        
        // Set comparison
        System.out.println("\n--- Set Comparison ---");
        Set<Integer> numbers1 = Set.of(1, 2, 3, 4, 5);
        Set<Integer> numbers2 = Set.of(1, 3, 5);
        
        System.out.println("numbers1 contains all of numbers2: " + numbers1.containsAll(numbers2));
        System.out.println("numbers2 is subset of numbers1: " + numbers1.containsAll(numbers2));
        System.out.println("Sets are disjoint: " + Collections.disjoint(numbers1, numbers2));
    }
    
    public static void main(String[] args) {
        demonstrateSetImplementations();
    }
}
```

### Example 3: Map Implementations

```java
// MapImplementations.java
import java.util.*;

public class MapImplementations {
    
    static class Employee {
        private int id;
        private String name;
        private String department;
        private double salary;
        
        public Employee(int id, String name, String department, double salary) {
            this.id = id;
            this.name = name;
            this.department = department;
            this.salary = salary;
        }
        
        @Override
        public String toString() {
            return String.format("Employee{id=%d, name='%s', dept='%s', salary=%.0f}", 
                    id, name, department, salary);
        }
        
        // Getters
        public int getId() { return id; }
        public String getName() { return name; }
        public String getDepartment() { return department; }
        public double getSalary() { return salary; }
    }
    
    public static void demonstrateMapImplementations() {
        System.out.println("=== Map Implementations ===");
        
        // HashMap - No ordering, O(1) operations
        System.out.println("\n--- HashMap Example ---");
        Map<Integer, Employee> employeeMap = new HashMap<>();
        
        employeeMap.put(101, new Employee(101, "Alice Johnson", "Engineering", 75000));
        employeeMap.put(102, new Employee(102, "Bob Smith", "Marketing", 65000));
        employeeMap.put(103, new Employee(103, "Carol Davis", "Engineering", 80000));
        employeeMap.put(104, new Employee(104, "David Wilson", "HR", 60000));
        
        System.out.println("HashMap contents:");
        employeeMap.forEach((id, employee) -> 
            System.out.printf("  ID %d: %s%n", id, employee));
        
        // TreeMap - Natural key ordering, O(log n) operations
        System.out.println("\n--- TreeMap Example ---");
        Map<String, List<Employee>> departmentMap = new TreeMap<>();
        
        // Group employees by department
        for (Employee emp : employeeMap.values()) {
            departmentMap.computeIfAbsent(emp.getDepartment(), k -> new ArrayList<>()).add(emp);
        }
        
        System.out.println("TreeMap contents (sorted by department):");
        departmentMap.forEach((dept, employees) -> {
            System.out.printf("  %s Department:%n", dept);
            employees.forEach(emp -> System.out.printf("    %s%n", emp));
        });
        
        // LinkedHashMap - Insertion/access order
        System.out.println("\n--- LinkedHashMap Example ---");
        Map<String, Integer> accessLog = new LinkedHashMap<>(16, 0.75f, true); // Access order
        
        accessLog.put("page1.html", 1);
        accessLog.put("page2.html", 1);
        accessLog.put("page3.html", 1);
        
        // Access some pages (changes order in access-order LinkedHashMap)
        accessLog.get("page1.html");
        accessLog.get("page3.html");
        
        System.out.println("LinkedHashMap (access order):");
        accessLog.forEach((page, count) -> 
            System.out.printf("  %s: %d%n", page, count));
        
        // Map operations
        System.out.println("\n--- Map Operations ---");
        Map<String, Integer> wordCount = new HashMap<>();
        String text = "the quick brown fox jumps over the lazy dog the fox";
        String[] words = text.split("\\s+");
        
        // Count word occurrences
        for (String word : words) {
            wordCount.merge(word, 1, Integer::sum);
        }
        
        System.out.println("Word count:");
        wordCount.entrySet().stream()
                .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
                .forEach(entry -> 
                    System.out.printf("  '%s': %d%n", entry.getKey(), entry.getValue()));
        
        // Map iteration methods
        System.out.println("\n--- Map Iteration Methods ---");
        
        // Key set iteration
        System.out.println("Keys:");
        for (String key : wordCount.keySet()) {
            System.out.printf("  %s%n", key);
        }
        
        // Values iteration
        System.out.println("Values:");
        for (Integer value : wordCount.values()) {
            System.out.printf("  %d%n", value);
        }
        
        // Entry set iteration
        System.out.println("Entries:");
        for (Map.Entry<String, Integer> entry : wordCount.entrySet()) {
            System.out.printf("  %s -> %d%n", entry.getKey(), entry.getValue());
        }
        
        // Stream operations on maps
        System.out.println("\n--- Stream Operations ---");
        
        // Filter and transform
        wordCount.entrySet().stream()
                .filter(entry -> entry.getValue() > 1)
                .map(entry -> entry.getKey().toUpperCase())
                .sorted()
                .forEach(word -> System.out.printf("  Frequent: %s%n", word));
    }
    
    public static void main(String[] args) {
        demonstrateMapImplementations();
    }
}
```

## 🔥 Interview Questions & Answers

### Q1: What's the difference between ArrayList and LinkedList?
**Answer**: 
- **ArrayList**: Dynamic array, O(1) random access, O(n) insertion/deletion in middle
- **LinkedList**: Doubly-linked list, O(n) access, O(1) insertion/deletion at known position
- **Use ArrayList**: When you need frequent random access to elements
- **Use LinkedList**: When you need frequent insertion/deletion at beginning/middle

### Q2: When would you use TreeSet over HashSet?
**Answer**: 
- **TreeSet**: When you need sorted order, implements NavigableSet, O(log n) operations
- **HashSet**: When you need fast operations and don't care about order, O(1) operations
- **TreeSet benefits**: Sorted iteration, range queries, first/last element access
- **HashSet benefits**: Faster operations, no Comparable requirement

### Q3: How does HashMap handle collisions?
**Answer**: 
- **Chaining**: Multiple entries in same bucket stored in linked list/tree
- **Load factor**: Threshold (0.75) for resizing to maintain performance
- **Tree conversion**: Chains convert to balanced trees when length > 8
- **Hashing**: Uses object's hashCode() and equals() methods

### Q4: What's the difference between HashMap and TreeMap?
**Answer**: 
- **HashMap**: Hash table, O(1) operations, no ordering
- **TreeMap**: Red-black tree, O(log n) operations, sorted by keys
- **Use HashMap**: Fast access, no ordering requirement
- **Use TreeMap**: Need sorted keys, range operations, NavigableMap features

### Q5: How do you make a collection thread-safe?
**Answer**: 
- **Collections.synchronized*()**: Wrapper methods for thread safety
- **ConcurrentHashMap**: Thread-safe map with better performance
- **CopyOnWriteArrayList**: Thread-safe list for read-heavy scenarios
- **Explicit synchronization**: Use synchronized blocks when needed

---
[← Previous: Multiple Inheritance](./multiple-inheritance.md) | [Next: Multithreading Basics →](./multithreading-basics.md)
