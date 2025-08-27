# Iteration Mechanisms - Collection Traversal

## Navigation
- [← Previous: Queue Implementations](queue-implementations.md)
- [🏠 Main Menu](README.md)

---

## Table of Contents
1. [Iteration Overview](#iteration-overview)
2. [Iterator Interface](#iterator-interface)
3. [ListIterator Interface](#listiterator-interface)
4. [Enhanced For-Loop](#enhanced-for-loop)
5. [Stream API Iteration](#stream-api-iteration)
6. [Performance Comparison](#performance-comparison)
7. [Interview Questions](#interview-questions)

---

## Iteration Overview

### What is Iteration?
Iteration is the process of traversing through elements of a collection one by one. Java provides multiple mechanisms to iterate over collections, each with its own advantages and use cases.

### Real-world Analogy 📖
Think of iteration mechanisms like different ways to **read a book**:
- **Iterator**: Reading page by page with a bookmark - you can pause, resume, and remove pages
- **Enhanced for-loop**: Speed reading - fast and simple, but you can't modify the book
- **ListIterator**: Advanced bookmark - can move forward/backward and insert pages
- **Stream**: Digital reader with advanced features - can filter, transform, and process content

### Evolution of Iteration in Java
```java
import java.util.*;

public class IterationEvolution {
    public static void main(String[] args) {
        List<String> fruits = Arrays.asList("Apple", "Banana", "Cherry", "Date");
        
        // 1. Traditional for loop (Java 1.0+)
        System.out.println("1. Traditional for loop:");
        for (int i = 0; i < fruits.size(); i++) {
            System.out.println(fruits.get(i));
        }
        
        // 2. Iterator (Java 1.2+)
        System.out.println("\n2. Iterator:");
        Iterator<String> iter = fruits.iterator();
        while (iter.hasNext()) {
            System.out.println(iter.next());
        }
        
        // 3. Enhanced for loop (Java 1.5+)
        System.out.println("\n3. Enhanced for loop:");
        for (String fruit : fruits) {
            System.out.println(fruit);
        }
        
        // 4. forEach with lambda (Java 8+)
        System.out.println("\n4. forEach with lambda:");
        fruits.forEach(System.out::println);
        
        // 5. Stream API (Java 8+)
        System.out.println("\n5. Stream API:");
        fruits.stream()
              .filter(fruit -> fruit.length() > 5)
              .forEach(System.out::println);
    }
}
```

---

## Iterator Interface

### What is Iterator?
Iterator is an interface that provides methods to traverse collections. It's the most basic and universal way to iterate through any collection in Java.

### Iterator Methods
```java
public interface Iterator<E> {
    boolean hasNext();    // Returns true if more elements exist
    E next();            // Returns next element
    void remove();       // Removes last returned element (optional)
}
```

### Basic Iterator Usage
```java
import java.util.*;

public class IteratorBasics {
    public static void main(String[] args) {
        List<String> languages = new ArrayList<>(
            Arrays.asList("Java", "Python", "JavaScript", "C++", "Go")
        );
        
        System.out.println("Original list: " + languages);
        
        // Basic iteration
        System.out.println("\n=== Basic Iteration ===");
        Iterator<String> iter = languages.iterator();
        while (iter.hasNext()) {
            String language = iter.next();
            System.out.println("Language: " + language);
        }
        
        // Safe removal during iteration
        System.out.println("\n=== Safe Removal ===");
        iter = languages.iterator();
        while (iter.hasNext()) {
            String language = iter.next();
            if (language.equals("JavaScript")) {
                iter.remove(); // Safe removal
                System.out.println("Removed: " + language);
            }
        }
        
        System.out.println("After removal: " + languages);
        
        // Iterator for different collection types
        demonstrateIteratorWithDifferentCollections();
    }
    
    private static void demonstrateIteratorWithDifferentCollections() {
        System.out.println("\n=== Iterator with Different Collections ===");
        
        // Set iteration
        Set<Integer> numbers = new HashSet<>(Arrays.asList(1, 2, 3, 4, 5));
        System.out.println("Set iteration:");
        Iterator<Integer> setIter = numbers.iterator();
        while (setIter.hasNext()) {
            System.out.print(setIter.next() + " ");
        }
        System.out.println();
        
        // Map iteration (through entrySet)
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Alice", 95);
        scores.put("Bob", 87);
        scores.put("Charlie", 92);
        
        System.out.println("Map iteration:");
        Iterator<Map.Entry<String, Integer>> mapIter = scores.entrySet().iterator();
        while (mapIter.hasNext()) {
            Map.Entry<String, Integer> entry = mapIter.next();
            System.out.println(entry.getKey() + " -> " + entry.getValue());
        }
        
        // Queue iteration
        Queue<String> queue = new LinkedList<>(Arrays.asList("First", "Second", "Third"));
        System.out.println("Queue iteration:");
        Iterator<String> queueIter = queue.iterator();
        while (queueIter.hasNext()) {
            System.out.print(queueIter.next() + " ");
        }
        System.out.println();
    }
}
```

### Advanced Iterator Patterns
```java
import java.util.*;

public class IteratorAdvanced {
    // Custom iterator for filtering
    static class FilteredIterator<T> implements Iterator<T> {
        private Iterator<T> iterator;
        private Predicate<T> predicate;
        private T nextElement;
        private boolean hasNextElement;
        
        public FilteredIterator(Iterator<T> iterator, Predicate<T> predicate) {
            this.iterator = iterator;
            this.predicate = predicate;
            findNext();
        }
        
        private void findNext() {
            hasNextElement = false;
            while (iterator.hasNext()) {
                nextElement = iterator.next();
                if (predicate.test(nextElement)) {
                    hasNextElement = true;
                    break;
                }
            }
        }
        
        @Override
        public boolean hasNext() {
            return hasNextElement;
        }
        
        @Override
        public T next() {
            if (!hasNextElement) {
                throw new NoSuchElementException();
            }
            T result = nextElement;
            findNext();
            return result;
        }
    }
    
    // Custom iterable class
    static class NumberRange implements Iterable<Integer> {
        private int start;
        private int end;
        private int step;
        
        public NumberRange(int start, int end, int step) {
            this.start = start;
            this.end = end;
            this.step = step;
        }
        
        @Override
        public Iterator<Integer> iterator() {
            return new Iterator<Integer>() {
                private int current = start;
                
                @Override
                public boolean hasNext() {
                    return step > 0 ? current < end : current > end;
                }
                
                @Override
                public Integer next() {
                    if (!hasNext()) {
                        throw new NoSuchElementException();
                    }
                    int result = current;
                    current += step;
                    return result;
                }
            };
        }
    }
    
    public static void main(String[] args) {
        // Test filtered iterator
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        System.out.println("=== Filtered Iterator (Even Numbers) ===");
        FilteredIterator<Integer> evenIter = new FilteredIterator<>(
            numbers.iterator(), 
            n -> n % 2 == 0
        );
        
        while (evenIter.hasNext()) {
            System.out.print(evenIter.next() + " ");
        }
        System.out.println();
        
        // Test custom iterable
        System.out.println("\n=== Custom Number Range ===");
        NumberRange range = new NumberRange(0, 20, 3);
        
        for (Integer num : range) {
            System.out.print(num + " ");
        }
        System.out.println();
        
        // Reverse iteration with custom range
        System.out.println("Reverse range:");
        NumberRange reverseRange = new NumberRange(20, 0, -3);
        for (Integer num : reverseRange) {
            System.out.print(num + " ");
        }
        System.out.println();
    }
    
    @FunctionalInterface
    interface Predicate<T> {
        boolean test(T t);
    }
}
```

### Iterator Safety and Best Practices
```java
import java.util.*;

public class IteratorSafety {
    public static void main(String[] args) {
        demonstrateConcurrentModification();
        demonstrateFailFastBehavior();
        demonstrateSafeModification();
    }
    
    private static void demonstrateConcurrentModification() {
        System.out.println("=== Concurrent Modification Exception ===");
        
        List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C", "D", "E"));
        
        try {
            // This will throw ConcurrentModificationException
            for (String item : list) {
                if (item.equals("C")) {
                    list.remove(item); // Modifying collection during iteration
                }
            }
        } catch (ConcurrentModificationException e) {
            System.out.println("Exception caught: " + e.getClass().getSimpleName());
        }
        
        System.out.println("List after exception: " + list);
    }
    
    private static void demonstrateFailFastBehavior() {
        System.out.println("\n=== Fail-Fast Behavior ===");
        
        List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
        Iterator<Integer> iter = numbers.iterator();
        
        numbers.add(6); // Modify collection after creating iterator
        
        try {
            while (iter.hasNext()) {
                System.out.println(iter.next()); // Will fail after first element
            }
        } catch (ConcurrentModificationException e) {
            System.out.println("Fail-fast detected: " + e.getClass().getSimpleName());
        }
    }
    
    private static void demonstrateSafeModification() {
        System.out.println("\n=== Safe Modification Patterns ===");
        
        List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C", "D", "E"));
        
        // 1. Using Iterator.remove()
        System.out.println("Before removal: " + list);
        Iterator<String> iter = list.iterator();
        while (iter.hasNext()) {
            String item = iter.next();
            if (item.equals("C")) {
                iter.remove(); // Safe removal
            }
        }
        System.out.println("After iterator removal: " + list);
        
        // 2. Using removeIf (Java 8+)
        list = new ArrayList<>(Arrays.asList("A", "B", "C", "D", "E"));
        list.removeIf(item -> item.equals("D"));
        System.out.println("After removeIf: " + list);
        
        // 3. Collecting elements to remove
        list = new ArrayList<>(Arrays.asList("A", "B", "C", "D", "E"));
        List<String> toRemove = new ArrayList<>();
        for (String item : list) {
            if (item.equals("B") || item.equals("E")) {
                toRemove.add(item);
            }
        }
        list.removeAll(toRemove);
        System.out.println("After collecting and removing: " + list);
        
        // 4. Reverse iteration for index-based removal
        list = new ArrayList<>(Arrays.asList("A", "B", "C", "D", "E"));
        for (int i = list.size() - 1; i >= 0; i--) {
            if (list.get(i).equals("A")) {
                list.remove(i);
            }
        }
        System.out.println("After reverse iteration removal: " + list);
    }
}
```

---

## ListIterator Interface

### What is ListIterator?
ListIterator extends Iterator and provides bidirectional iteration for List collections. It allows traversing in both directions and supports modification operations.

### ListIterator Methods
```java
public interface ListIterator<E> extends Iterator<E> {
    // Iterator methods
    boolean hasNext();
    E next();
    void remove();
    
    // Additional ListIterator methods
    boolean hasPrevious();     // Returns true if previous elements exist
    E previous();              // Returns previous element
    int nextIndex();           // Returns index of next element
    int previousIndex();       // Returns index of previous element
    void set(E e);            // Replaces last returned element
    void add(E e);            // Inserts element before next element
}
```

### ListIterator Basic Usage
```java
import java.util.*;

public class ListIteratorBasics {
    public static void main(String[] args) {
        List<String> cities = new ArrayList<>(
            Arrays.asList("New York", "London", "Tokyo", "Paris", "Sydney")
        );
        
        System.out.println("Original list: " + cities);
        
        // Forward iteration
        System.out.println("\n=== Forward Iteration ===");
        ListIterator<String> listIter = cities.listIterator();
        while (listIter.hasNext()) {
            int index = listIter.nextIndex();
            String city = listIter.next();
            System.out.println("Index " + index + ": " + city);
        }
        
        // Backward iteration
        System.out.println("\n=== Backward Iteration ===");
        while (listIter.hasPrevious()) {
            int index = listIter.previousIndex();
            String city = listIter.previous();
            System.out.println("Index " + index + ": " + city);
        }
        
        // Start from specific position
        System.out.println("\n=== Start from Middle ===");
        listIter = cities.listIterator(2); // Start from index 2
        System.out.println("Starting from index 2:");
        while (listIter.hasNext()) {
            System.out.println("- " + listIter.next());
        }
        
        demonstrateModificationOperations();
    }
    
    private static void demonstrateModificationOperations() {
        System.out.println("\n=== Modification Operations ===");
        
        List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
        System.out.println("Original: " + numbers);
        
        ListIterator<Integer> iter = numbers.listIterator();
        
        // Add elements
        iter.add(0); // Add at beginning
        System.out.println("After adding 0 at start: " + numbers);
        
        // Navigate and modify
        iter.next(); // Skip 0
        iter.next(); // Skip 1
        iter.set(999); // Replace 1 with 999
        System.out.println("After replacing 1 with 999: " + numbers);
        
        // Add element in middle
        iter.add(100); // Add after current position
        System.out.println("After adding 100: " + numbers);
        
        // Remove element
        iter.previous(); // Go back to 100
        iter.remove();   // Remove 100
        System.out.println("After removing 100: " + numbers);
    }
}
```

### Advanced ListIterator Applications
```java
import java.util.*;

public class ListIteratorAdvanced {
    // Reverse a list using ListIterator
    public static <T> void reverseList(List<T> list) {
        ListIterator<T> iter = list.listIterator();
        List<T> temp = new ArrayList<>();
        
        // Collect all elements
        while (iter.hasNext()) {
            temp.add(iter.next());
        }
        
        // Go back to beginning
        while (iter.hasPrevious()) {
            iter.previous();
            iter.remove();
        }
        
        // Add elements in reverse order
        for (int i = temp.size() - 1; i >= 0; i--) {
            iter.add(temp.get(i));
        }
    }
    
    // Insert elements at specific positions
    public static <T> void insertAtMultiplePositions(List<T> list, T element, int... positions) {
        Arrays.sort(positions); // Sort positions
        
        ListIterator<T> iter = list.listIterator();
        int currentPos = 0;
        int posIndex = 0;
        
        while (iter.hasNext() && posIndex < positions.length) {
            if (currentPos == positions[posIndex]) {
                iter.add(element);
                posIndex++;
                // Don't increment currentPos for the added element
            } else {
                iter.next();
                currentPos++;
            }
        }
        
        // Handle positions at the end
        while (posIndex < positions.length) {
            iter.add(element);
            posIndex++;
        }
    }
    
    // Find and replace all occurrences
    public static <T> int findAndReplace(List<T> list, T target, T replacement) {
        ListIterator<T> iter = list.listIterator();
        int count = 0;
        
        while (iter.hasNext()) {
            T element = iter.next();
            if (Objects.equals(element, target)) {
                iter.set(replacement);
                count++;
            }
        }
        
        return count;
    }
    
    // Palindrome checker for lists
    public static <T> boolean isPalindrome(List<T> list) {
        if (list.size() <= 1) return true;
        
        ListIterator<T> forward = list.listIterator();
        ListIterator<T> backward = list.listIterator(list.size());
        
        while (forward.nextIndex() < backward.previousIndex()) {
            if (!Objects.equals(forward.next(), backward.previous())) {
                return false;
            }
        }
        
        return true;
    }
    
    public static void main(String[] args) {
        // Test reverse
        System.out.println("=== Reverse List ===");
        List<String> fruits = new ArrayList<>(Arrays.asList("Apple", "Banana", "Cherry"));
        System.out.println("Before reverse: " + fruits);
        reverseList(fruits);
        System.out.println("After reverse: " + fruits);
        
        // Test multiple insertions
        System.out.println("\n=== Multiple Insertions ===");
        List<String> letters = new ArrayList<>(Arrays.asList("A", "B", "C", "D"));
        System.out.println("Before insertion: " + letters);
        insertAtMultiplePositions(letters, "X", 1, 3, 5);
        System.out.println("After inserting X at positions 1,3,5: " + letters);
        
        // Test find and replace
        System.out.println("\n=== Find and Replace ===");
        List<String> words = new ArrayList<>(Arrays.asList("cat", "dog", "cat", "bird", "cat"));
        System.out.println("Before replace: " + words);
        int replaced = findAndReplace(words, "cat", "feline");
        System.out.println("After replacing 'cat' with 'feline': " + words);
        System.out.println("Replaced " + replaced + " occurrences");
        
        // Test palindrome
        System.out.println("\n=== Palindrome Check ===");
        List<Integer> palindrome = Arrays.asList(1, 2, 3, 2, 1);
        List<Integer> notPalindrome = Arrays.asList(1, 2, 3, 4, 5);
        
        System.out.println(palindrome + " is palindrome: " + isPalindrome(palindrome));
        System.out.println(notPalindrome + " is palindrome: " + isPalindrome(notPalindrome));
    }
}
```

---

## Enhanced For-Loop

### What is Enhanced For-Loop?
The enhanced for-loop (for-each loop) provides a simplified way to iterate over collections and arrays. It's more readable and less error-prone than traditional loops.

### Basic Enhanced For-Loop
```java
import java.util.*;

public class EnhancedForLoopBasics {
    public static void main(String[] args) {
        // Array iteration
        System.out.println("=== Array Iteration ===");
        int[] numbers = {1, 2, 3, 4, 5};
        
        // Traditional for loop
        System.out.println("Traditional for loop:");
        for (int i = 0; i < numbers.length; i++) {
            System.out.print(numbers[i] + " ");
        }
        System.out.println();
        
        // Enhanced for loop
        System.out.println("Enhanced for loop:");
        for (int num : numbers) {
            System.out.print(num + " ");
        }
        System.out.println();
        
        // Collection iteration
        System.out.println("\n=== Collection Iteration ===");
        List<String> colors = Arrays.asList("Red", "Green", "Blue", "Yellow");
        
        for (String color : colors) {
            System.out.println("Color: " + color);
        }
        
        // Map iteration
        System.out.println("\n=== Map Iteration ===");
        Map<String, Integer> ages = new HashMap<>();
        ages.put("Alice", 25);
        ages.put("Bob", 30);
        ages.put("Charlie", 35);
        
        // Iterate over entries
        for (Map.Entry<String, Integer> entry : ages.entrySet()) {
            System.out.println(entry.getKey() + " is " + entry.getValue() + " years old");
        }
        
        // Iterate over keys
        System.out.println("Names:");
        for (String name : ages.keySet()) {
            System.out.println("- " + name);
        }
        
        // Iterate over values
        System.out.println("Ages:");
        for (Integer age : ages.values()) {
            System.out.println("- " + age);
        }
        
        demonstrateNestedStructures();
    }
    
    private static void demonstrateNestedStructures() {
        System.out.println("\n=== Nested Structures ===");
        
        // 2D array
        int[][] matrix = {
            {1, 2, 3},
            {4, 5, 6},
            {7, 8, 9}
        };
        
        System.out.println("2D Array:");
        for (int[] row : matrix) {
            for (int value : row) {
                System.out.print(value + " ");
            }
            System.out.println();
        }
        
        // List of lists
        List<List<String>> departments = Arrays.asList(
            Arrays.asList("Alice", "Bob", "Charlie"),
            Arrays.asList("David", "Eve"),
            Arrays.asList("Frank", "Grace", "Henry", "Ivy")
        );
        
        System.out.println("Departments:");
        int deptNum = 1;
        for (List<String> department : departments) {
            System.out.println("Department " + deptNum + ":");
            for (String employee : department) {
                System.out.println("  - " + employee);
            }
            deptNum++;
        }
    }
}
```

### Enhanced For-Loop Limitations and Alternatives
```java
import java.util.*;

public class EnhancedForLoopLimitations {
    public static void main(String[] args) {
        List<String> items = new ArrayList<>(Arrays.asList("A", "B", "C", "D", "E"));
        
        demonstrateReadOnlyNature();
        demonstrateIndexAccess();
        demonstrateModificationLimitations();
        demonstratePerformanceConsiderations();
    }
    
    private static void demonstrateReadOnlyNature() {
        System.out.println("=== Read-Only Nature ===");
        
        List<StringBuilder> builders = Arrays.asList(
            new StringBuilder("Hello"),
            new StringBuilder("World"),
            new StringBuilder("Java")
        );
        
        // Can modify object contents (not the reference)
        for (StringBuilder sb : builders) {
            sb.append("!"); // This works - modifying object state
            // sb = new StringBuilder("New"); // This won't affect the collection
        }
        
        System.out.println("After modification: " + builders);
    }
    
    private static void demonstrateIndexAccess() {
        System.out.println("\n=== Index Access Limitation ===");
        
        List<String> items = Arrays.asList("A", "B", "C", "D", "E");
        
        // Enhanced for-loop - no index access
        System.out.println("Enhanced for-loop (no indices):");
        for (String item : items) {
            System.out.println(item);
        }
        
        // Traditional for-loop - with index access
        System.out.println("Traditional for-loop (with indices):");
        for (int i = 0; i < items.size(); i++) {
            System.out.println("Index " + i + ": " + items.get(i));
        }
        
        // Solution: Use iterator with manual counting
        System.out.println("Iterator with manual counting:");
        int index = 0;
        for (String item : items) {
            System.out.println("Index " + index + ": " + item);
            index++;
        }
    }
    
    private static void demonstrateModificationLimitations() {
        System.out.println("\n=== Modification Limitations ===");
        
        List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C", "D"));
        
        // This will cause ConcurrentModificationException
        try {
            for (String item : list) {
                if (item.equals("B")) {
                    // list.remove(item); // Uncommenting this will throw exception
                }
            }
        } catch (ConcurrentModificationException e) {
            System.out.println("Exception: " + e.getClass().getSimpleName());
        }
        
        // Solutions:
        // 1. Use Iterator
        System.out.println("Solution 1: Using Iterator");
        Iterator<String> iter = list.iterator();
        while (iter.hasNext()) {
            String item = iter.next();
            if (item.equals("B")) {
                iter.remove();
            }
        }
        System.out.println("After iterator removal: " + list);
        
        // 2. Use removeIf
        list = new ArrayList<>(Arrays.asList("A", "B", "C", "D"));
        System.out.println("Solution 2: Using removeIf");
        list.removeIf(item -> item.equals("C"));
        System.out.println("After removeIf: " + list);
        
        // 3. Collect and remove
        list = new ArrayList<>(Arrays.asList("A", "B", "C", "D"));
        System.out.println("Solution 3: Collect and remove");
        List<String> toRemove = new ArrayList<>();
        for (String item : list) {
            if (item.equals("D")) {
                toRemove.add(item);
            }
        }
        list.removeAll(toRemove);
        System.out.println("After collect and remove: " + list);
    }
    
    private static void demonstratePerformanceConsiderations() {
        System.out.println("\n=== Performance Considerations ===");
        
        // ArrayList - enhanced for-loop is efficient
        List<Integer> arrayList = new ArrayList<>();
        for (int i = 0; i < 10000; i++) {
            arrayList.add(i);
        }
        
        long start = System.nanoTime();
        int sum1 = 0;
        for (int num : arrayList) {
            sum1 += num;
        }
        long enhancedTime = System.nanoTime() - start;
        
        start = System.nanoTime();
        int sum2 = 0;
        for (int i = 0; i < arrayList.size(); i++) {
            sum2 += arrayList.get(i);
        }
        long traditionalTime = System.nanoTime() - start;
        
        System.out.printf("ArrayList - Enhanced: %.2f ms, Traditional: %.2f ms%n", 
                         enhancedTime / 1_000_000.0, traditionalTime / 1_000_000.0);
        
        // LinkedList - enhanced for-loop is much more efficient
        List<Integer> linkedList = new LinkedList<>();
        for (int i = 0; i < 5000; i++) { // Smaller size due to O(n²) complexity
            linkedList.add(i);
        }
        
        start = System.nanoTime();
        sum1 = 0;
        for (int num : linkedList) {
            sum1 += num;
        }
        enhancedTime = System.nanoTime() - start;
        
        start = System.nanoTime();
        sum2 = 0;
        for (int i = 0; i < linkedList.size(); i++) {
            sum2 += linkedList.get(i); // O(n) for each access!
        }
        traditionalTime = System.nanoTime() - start;
        
        System.out.printf("LinkedList - Enhanced: %.2f ms, Traditional: %.2f ms%n", 
                         enhancedTime / 1_000_000.0, traditionalTime / 1_000_000.0);
        System.out.printf("Enhanced is %.1fx faster for LinkedList%n", 
                         (double)traditionalTime / enhancedTime);
    }
}
```

---

## Stream API Iteration

### Stream-based Iteration
```java
import java.util.*;
import java.util.stream.*;

public class StreamIteration {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");
        
        demonstrateBasicStreamOperations(words);
        demonstrateAdvancedStreamOperations();
        demonstrateParallelStreams();
    }
    
    private static void demonstrateBasicStreamOperations(List<String> words) {
        System.out.println("=== Basic Stream Operations ===");
        
        // Simple iteration
        System.out.println("All words:");
        words.stream().forEach(System.out::println);
        
        // Filtering and iteration
        System.out.println("\nWords longer than 5 characters:");
        words.stream()
             .filter(word -> word.length() > 5)
             .forEach(System.out::println);
        
        // Mapping and iteration
        System.out.println("\nUppercase words:");
        words.stream()
             .map(String::toUpperCase)
             .forEach(System.out::println);
        
        // Chaining operations
        System.out.println("\nFirst 3 words, uppercase, starting with vowels:");
        words.stream()
             .filter(word -> "aeiouAEIOU".indexOf(word.charAt(0)) >= 0)
             .map(String::toUpperCase)
             .limit(3)
             .forEach(System.out::println);
    }
    
    private static void demonstrateAdvancedStreamOperations() {
        System.out.println("\n=== Advanced Stream Operations ===");
        
        List<Person> people = Arrays.asList(
            new Person("Alice", 25, "Engineer"),
            new Person("Bob", 30, "Manager"),
            new Person("Charlie", 35, "Engineer"),
            new Person("Diana", 28, "Designer"),
            new Person("Eve", 32, "Manager")
        );
        
        // Group by profession
        System.out.println("People grouped by profession:");
        people.stream()
              .collect(Collectors.groupingBy(Person::getProfession))
              .forEach((profession, personList) -> {
                  System.out.println(profession + ": " + 
                      personList.stream()
                               .map(Person::getName)
                               .collect(Collectors.joining(", ")));
              });
        
        // Find specific elements
        System.out.println("\nFirst engineer:");
        people.stream()
              .filter(p -> p.getProfession().equals("Engineer"))
              .findFirst()
              .ifPresent(System.out::println);
        
        // Statistics
        System.out.println("\nAge statistics:");
        IntSummaryStatistics ageStats = people.stream()
                                             .mapToInt(Person::getAge)
                                             .summaryStatistics();
        System.out.println("Average age: " + ageStats.getAverage());
        System.out.println("Min age: " + ageStats.getMin());
        System.out.println("Max age: " + ageStats.getMax());
        
        // Custom collector
        System.out.println("\nNames with ages:");
        String nameAgeList = people.stream()
                                  .map(p -> p.getName() + "(" + p.getAge() + ")")
                                  .collect(Collectors.joining(", "));
        System.out.println(nameAgeList);
    }
    
    private static void demonstrateParallelStreams() {
        System.out.println("\n=== Parallel Streams ===");
        
        List<Integer> numbers = IntStream.rangeClosed(1, 1000000)
                                        .boxed()
                                        .collect(Collectors.toList());
        
        // Sequential stream
        long start = System.nanoTime();
        long sequentialSum = numbers.stream()
                                   .mapToLong(Integer::longValue)
                                   .sum();
        long sequentialTime = System.nanoTime() - start;
        
        // Parallel stream
        start = System.nanoTime();
        long parallelSum = numbers.parallelStream()
                                 .mapToLong(Integer::longValue)
                                 .sum();
        long parallelTime = System.nanoTime() - start;
        
        System.out.println("Sequential sum: " + sequentialSum + 
                          " (Time: " + (sequentialTime / 1_000_000) + "ms)");
        System.out.println("Parallel sum: " + parallelSum + 
                          " (Time: " + (parallelTime / 1_000_000) + "ms)");
        System.out.printf("Parallel is %.1fx faster%n", 
                         (double)sequentialTime / parallelTime);
        
        // Demonstrate when parallel might not be beneficial
        List<String> shortList = Arrays.asList("A", "B", "C", "D", "E");
        
        start = System.nanoTime();
        shortList.stream().forEach(s -> {
            // Simulate some work
            try { Thread.sleep(1); } catch (InterruptedException e) {}
        });
        long shortSequentialTime = System.nanoTime() - start;
        
        start = System.nanoTime();
        shortList.parallelStream().forEach(s -> {
            // Simulate some work
            try { Thread.sleep(1); } catch (InterruptedException e) {}
        });
        long shortParallelTime = System.nanoTime() - start;
        
        System.out.println("\nFor small collections:");
        System.out.println("Sequential: " + (shortSequentialTime / 1_000_000) + "ms");
        System.out.println("Parallel: " + (shortParallelTime / 1_000_000) + "ms");
        System.out.println("Parallel overhead can make it slower for small collections");
    }
    
    static class Person {
        private String name;
        private int age;
        private String profession;
        
        public Person(String name, int age, String profession) {
            this.name = name;
            this.age = age;
            this.profession = profession;
        }
        
        public String getName() { return name; }
        public int getAge() { return age; }
        public String getProfession() { return profession; }
        
        @Override
        public String toString() {
            return name + "(" + age + ", " + profession + ")";
        }
    }
}
```

---

## Performance Comparison

### Iteration Performance Benchmark
```java
import java.util.*;

public class IterationPerformanceBenchmark {
    private static final int SIZE = 1000000;
    private static final int ITERATIONS = 5;
    
    public static void main(String[] args) {
        // Prepare data
        List<Integer> arrayList = new ArrayList<>();
        List<Integer> linkedList = new LinkedList<>();
        
        for (int i = 0; i < SIZE; i++) {
            arrayList.add(i);
            linkedList.add(i);
        }
        
        System.out.println("Performance Benchmark with " + SIZE + " elements");
        System.out.println("Average of " + ITERATIONS + " iterations\n");
        
        benchmarkArrayList(arrayList);
        benchmarkLinkedList(linkedList);
        benchmarkDifferentCollections();
    }
    
    private static void benchmarkArrayList(List<Integer> list) {
        System.out.println("=== ArrayList Benchmark ===");
        
        // Traditional for loop
        long totalTime = 0;
        for (int i = 0; i < ITERATIONS; i++) {
            long start = System.nanoTime();
            long sum = 0;
            for (int j = 0; j < list.size(); j++) {
                sum += list.get(j);
            }
            totalTime += System.nanoTime() - start;
        }
        System.out.printf("Traditional for loop: %.2f ms%n", totalTime / ITERATIONS / 1_000_000.0);
        
        // Enhanced for loop
        totalTime = 0;
        for (int i = 0; i < ITERATIONS; i++) {
            long start = System.nanoTime();
            long sum = 0;
            for (int num : list) {
                sum += num;
            }
            totalTime += System.nanoTime() - start;
        }
        System.out.printf("Enhanced for loop: %.2f ms%n", totalTime / ITERATIONS / 1_000_000.0);
        
        // Iterator
        totalTime = 0;
        for (int i = 0; i < ITERATIONS; i++) {
            long start = System.nanoTime();
            long sum = 0;
            Iterator<Integer> iter = list.iterator();
            while (iter.hasNext()) {
                sum += iter.next();
            }
            totalTime += System.nanoTime() - start;
        }
        System.out.printf("Iterator: %.2f ms%n", totalTime / ITERATIONS / 1_000_000.0);
        
        // Stream
        totalTime = 0;
        for (int i = 0; i < ITERATIONS; i++) {
            long start = System.nanoTime();
            long sum = list.stream().mapToLong(Integer::longValue).sum();
            totalTime += System.nanoTime() - start;
        }
        System.out.printf("Stream: %.2f ms%n", totalTime / ITERATIONS / 1_000_000.0);
        
        // Parallel Stream
        totalTime = 0;
        for (int i = 0; i < ITERATIONS; i++) {
            long start = System.nanoTime();
            long sum = list.parallelStream().mapToLong(Integer::longValue).sum();
            totalTime += System.nanoTime() - start;
        }
        System.out.printf("Parallel Stream: %.2f ms%n%n", totalTime / ITERATIONS / 1_000_000.0);
    }
    
    private static void benchmarkLinkedList(List<Integer> list) {
        System.out.println("=== LinkedList Benchmark ===");
        
        // Enhanced for loop (efficient for LinkedList)
        long totalTime = 0;
        for (int i = 0; i < ITERATIONS; i++) {
            long start = System.nanoTime();
            long sum = 0;
            for (int num : list) {
                sum += num;
            }
            totalTime += System.nanoTime() - start;
        }
        System.out.printf("Enhanced for loop: %.2f ms%n", totalTime / ITERATIONS / 1_000_000.0);
        
        // Iterator (same performance as enhanced for-loop)
        totalTime = 0;
        for (int i = 0; i < ITERATIONS; i++) {
            long start = System.nanoTime();
            long sum = 0;
            Iterator<Integer> iter = list.iterator();
            while (iter.hasNext()) {
                sum += iter.next();
            }
            totalTime += System.nanoTime() - start;
        }
        System.out.printf("Iterator: %.2f ms%n", totalTime / ITERATIONS / 1_000_000.0);
        
        // Traditional for loop (very slow for LinkedList)
        int smallerSize = Math.min(SIZE, 10000); // Use smaller size to avoid timeout
        List<Integer> smallList = new LinkedList<>(list.subList(0, smallerSize));
        
        long start = System.nanoTime();
        long sum = 0;
        for (int j = 0; j < smallList.size(); j++) {
            sum += smallList.get(j); // O(n) for each access!
        }
        long time = System.nanoTime() - start;
        System.out.printf("Traditional for loop (%d elements): %.2f ms%n", 
                         smallerSize, time / 1_000_000.0);
        System.out.printf("Estimated time for full list: %.2f seconds%n%n", 
                         time / 1_000_000.0 * SIZE / smallerSize / 1000.0);
    }
    
    private static void benchmarkDifferentCollections() {
        System.out.println("=== Different Collections Comparison ===");
        
        // Prepare different collection types
        List<Integer> arrayList = new ArrayList<>();
        Set<Integer> hashSet = new HashSet<>();
        Set<Integer> treeSet = new TreeSet<>();
        
        for (int i = 0; i < SIZE / 10; i++) { // Smaller size for TreeSet
            arrayList.add(i);
            hashSet.add(i);
            treeSet.add(i);
        }
        
        // ArrayList
        long start = System.nanoTime();
        long sum = 0;
        for (int num : arrayList) {
            sum += num;
        }
        long arrayListTime = System.nanoTime() - start;
        
        // HashSet
        start = System.nanoTime();
        sum = 0;
        for (int num : hashSet) {
            sum += num;
        }
        long hashSetTime = System.nanoTime() - start;
        
        // TreeSet
        start = System.nanoTime();
        sum = 0;
        for (int num : treeSet) {
            sum += num;
        }
        long treeSetTime = System.nanoTime() - start;
        
        System.out.printf("ArrayList iteration: %.2f ms%n", arrayListTime / 1_000_000.0);
        System.out.printf("HashSet iteration: %.2f ms%n", hashSetTime / 1_000_000.0);
        System.out.printf("TreeSet iteration: %.2f ms%n", treeSetTime / 1_000_000.0);
    }
}
```

---

## Interview Questions

### Q1: What are the different ways to iterate over a collection in Java?

**Answer:**
```java
List<String> list = Arrays.asList("A", "B", "C");

// 1. Traditional for loop (index-based)
for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}

// 2. Enhanced for loop (for-each)
for (String item : list) {
    System.out.println(item);
}

// 3. Iterator
Iterator<String> iter = list.iterator();
while (iter.hasNext()) {
    System.out.println(iter.next());
}

// 4. ListIterator (for Lists only)
ListIterator<String> listIter = list.listIterator();
while (listIter.hasNext()) {
    System.out.println(listIter.next());
}

// 5. forEach method (Java 8+)
list.forEach(System.out::println);

// 6. Stream API (Java 8+)
list.stream().forEach(System.out::println);
```

### Q2: When would you use Iterator instead of enhanced for-loop?
**Answer:**
Use Iterator when you need to:

1. **Remove elements safely during iteration**
2. **More control over iteration process**
3. **Iterate over multiple collections simultaneously**

```java
// Safe removal
Iterator<String> iter = list.iterator();
while (iter.hasNext()) {
    String item = iter.next();
    if (item.equals("unwanted")) {
        iter.remove(); // Safe removal
    }
}

// Enhanced for-loop cannot do this:
// for (String item : list) {
//     list.remove(item); // ConcurrentModificationException!
// }
```

### Q3: What is the difference between Iterator and ListIterator?
**Answer:**

| Feature | Iterator | ListIterator |
|---------|----------|--------------|
| **Direction** | Forward only | Both directions |
| **Modification** | remove() only | add(), remove(), set() |
| **Index access** | No | nextIndex(), previousIndex() |
| **Collections** | All collections | Lists only |
| **Starting position** | Beginning only | Any position |

### Q4: What is fail-fast behavior in iterators?
**Answer:**
Fail-fast iterators detect concurrent modification and throw `ConcurrentModificationException`:

```java
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
Iterator<String> iter = list.iterator();

list.add("D"); // Modify collection after creating iterator

iter.next(); // Throws ConcurrentModificationException
```

**Solutions:**
- Use Iterator.remove() for safe removal
- Use ConcurrentHashMap/CopyOnWriteArrayList for concurrent access
- Use synchronized collections with proper synchronization

### Q5: Which iteration method is most efficient for different collection types?
**Answer:**

| Collection | Most Efficient | Reason |
|------------|----------------|---------|
| **ArrayList** | Enhanced for-loop or traditional for-loop | Direct array access |
| **LinkedList** | Enhanced for-loop or Iterator | Avoids O(n) get(index) calls |
| **HashSet** | Enhanced for-loop or Iterator | Only options available |
| **TreeSet** | Enhanced for-loop or Iterator | In-order traversal |
| **HashMap** | Enhanced for-loop over entrySet() | Single pass through entries |

```java
// Efficient LinkedList iteration
for (String item : linkedList) { // O(n)
    process(item);
}

// Inefficient LinkedList iteration
for (int i = 0; i < linkedList.size(); i++) { // O(n²)
    process(linkedList.get(i)); // Each get() is O(n)
}
```

---

## Navigation
- [← Previous: Queue Implementations](queue-implementations.md)
- [🏠 Main Menu](README.md)

---

*This comprehensive guide covers all iteration mechanisms in Java. Understanding when and how to use each iteration method is crucial for writing efficient and maintainable code, especially in technical interviews where performance considerations matter.*
