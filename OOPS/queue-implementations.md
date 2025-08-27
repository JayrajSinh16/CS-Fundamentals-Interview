# Queue Implementations - FIFO Collections

## Navigation
- [← Previous: Map Implementations](map-implementations.md)
- [🏠 Main Menu](README.md)
- [→ Next: Iteration Mechanisms](iteration-mechanisms.md)

---

## Table of Contents
1. [Queue Interface Overview](#queue-interface-overview)
2. [PriorityQueue](#priorityqueue)
3. [ArrayDeque](#arraydeque)
4. [LinkedList as Queue](#linkedlist-as-queue)
5. [Performance Comparison](#performance-comparison)
6. [When to Use Which](#when-to-use-which)
7. [Interview Questions](#interview-questions)

---

## Queue Interface Overview

### What is Queue Interface?
The Queue interface extends Collection and represents a collection designed for holding elements prior to processing. Queues typically order elements in a FIFO (first-in-first-out) manner.

### Real-world Analogy 🎫
Think of Queue implementations like different types of **waiting systems**:
- **PriorityQueue**: Hospital emergency room - high priority patients go first
- **ArrayDeque**: Fast-moving supermarket checkout - efficient both ways
- **LinkedList**: Traditional queue line - can add/remove from both ends but optimized for ends

### Key Queue Interface Methods
```java
import java.util.*;

public class QueueInterfaceDemo {
    public static void main(String[] args) {
        Queue<String> queue = new LinkedList<>();
        
        // Adding elements (rear of queue)
        queue.offer("First");    // Preferred - returns boolean
        queue.add("Second");     // Throws exception if fails
        queue.offer("Third");
        
        System.out.println("Queue: " + queue);
        
        // Examining head element
        String head1 = queue.peek();    // Returns null if empty
        String head2 = queue.element(); // Throws exception if empty
        
        System.out.println("Head (peek): " + head1);
        System.out.println("Head (element): " + head2);
        
        // Removing elements (front of queue)
        String removed1 = queue.poll();   // Returns null if empty
        String removed2 = queue.remove(); // Throws exception if empty
        
        System.out.println("Removed (poll): " + removed1);
        System.out.println("Removed (remove): " + removed2);
        System.out.println("Remaining: " + queue);
        
        // Queue properties
        System.out.println("Size: " + queue.size());
        System.out.println("Is empty: " + queue.isEmpty());
    }
}
```

---

## PriorityQueue

### What is PriorityQueue?
PriorityQueue is a heap-based priority queue implementation. Elements are ordered according to their natural ordering or by a Comparator provided at queue construction time.

### Internal Structure
```java
// Simplified internal structure of PriorityQueue
public class PriorityQueue<E> {
    private Object[] queue;  // Binary heap array
    private int size;        // Number of elements
    private Comparator<? super E> comparator;
    
    // Add element - maintains heap property
    public boolean offer(E element) {
        if (element == null) throw new NullPointerException();
        
        int i = size;
        if (i >= queue.length) {
            grow(i + 1); // Resize if needed
        }
        
        size = i + 1;
        if (i == 0) {
            queue[0] = element;
        } else {
            siftUp(i, element); // Maintain heap order
        }
        return true;
    }
    
    // Remove head - maintains heap property
    public E poll() {
        if (size == 0) return null;
        
        int s = --size;
        E result = (E) queue[0];
        E x = (E) queue[s];
        queue[s] = null;
        
        if (s != 0) {
            siftDown(0, x); // Restore heap order
        }
        
        return result;
    }
}
```

### PriorityQueue Features and Examples

#### 1. Natural Ordering (Min-Heap)
```java
import java.util.*;

public class PriorityQueueBasics {
    public static void main(String[] args) {
        // Natural ordering - min heap
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        
        // Add elements in random order
        minHeap.offer(15);
        minHeap.offer(10);
        minHeap.offer(20);
        minHeap.offer(8);
        minHeap.offer(25);
        
        System.out.println("PriorityQueue (min-heap): " + minHeap);
        
        // Remove elements - always gets smallest
        System.out.println("Removing elements in priority order:");
        while (!minHeap.isEmpty()) {
            System.out.println("Removed: " + minHeap.poll());
        }
        
        // String priority queue
        PriorityQueue<String> stringQueue = new PriorityQueue<>();
        stringQueue.addAll(Arrays.asList("Zebra", "Apple", "Banana", "Cherry"));
        
        System.out.println("\nString queue: " + stringQueue);
        System.out.println("Removing strings in alphabetical order:");
        while (!stringQueue.isEmpty()) {
            System.out.println("Removed: " + stringQueue.poll());
        }
    }
}
```

#### 2. Custom Ordering with Comparator
```java
import java.util.*;

public class PriorityQueueCustomOrder {
    static class Task {
        private String name;
        private int priority; // 1 = highest, 5 = lowest
        
        public Task(String name, int priority) {
            this.name = name;
            this.priority = priority;
        }
        
        public String getName() { return name; }
        public int getPriority() { return priority; }
        
        @Override
        public String toString() {
            return name + "(P" + priority + ")";
        }
    }
    
    public static void main(String[] args) {
        // Priority queue for tasks - higher priority first
        PriorityQueue<Task> taskQueue = new PriorityQueue<>(
            Comparator.comparing(Task::getPriority)
        );
        
        taskQueue.offer(new Task("Email Client", 3));
        taskQueue.offer(new Task("Security Patch", 1));
        taskQueue.offer(new Task("Documentation", 5));
        taskQueue.offer(new Task("Bug Fix", 2));
        taskQueue.offer(new Task("Feature Request", 4));
        
        System.out.println("Task queue: " + taskQueue);
        
        System.out.println("\nProcessing tasks by priority:");
        while (!taskQueue.isEmpty()) {
            Task task = taskQueue.poll();
            System.out.println("Processing: " + task);
        }
        
        // Max heap for numbers
        PriorityQueue<Integer> maxHeap = new PriorityQueue<>(
            Collections.reverseOrder()
        );
        
        maxHeap.addAll(Arrays.asList(15, 10, 20, 8, 25));
        System.out.println("\nMax heap: " + maxHeap);
        
        System.out.println("Removing elements (largest first):");
        while (!maxHeap.isEmpty()) {
            System.out.println("Removed: " + maxHeap.poll());
        }
    }
}
```

#### 3. Advanced Priority Queue Applications
```java
import java.util.*;

public class PriorityQueueApplications {
    // Dijkstra's algorithm simulation
    static class Edge {
        int destination;
        int weight;
        
        Edge(int destination, int weight) {
            this.destination = destination;
            this.weight = weight;
        }
    }
    
    static class Node {
        int vertex;
        int distance;
        
        Node(int vertex, int distance) {
            this.vertex = vertex;
            this.distance = distance;
        }
        
        @Override
        public String toString() {
            return "Node(" + vertex + ", dist=" + distance + ")";
        }
    }
    
    public static void demonstrateDijkstra() {
        System.out.println("=== Dijkstra's Algorithm Simulation ===");
        
        // Priority queue for shortest path
        PriorityQueue<Node> pq = new PriorityQueue<>(
            Comparator.comparing(node -> node.distance)
        );
        
        // Simulate finding shortest paths
        pq.offer(new Node(0, 0));   // Start node
        pq.offer(new Node(1, 5));
        pq.offer(new Node(2, 3));
        pq.offer(new Node(3, 8));
        pq.offer(new Node(2, 2));   // Better path to node 2
        
        System.out.println("Processing nodes by shortest distance:");
        Set<Integer> visited = new HashSet<>();
        
        while (!pq.isEmpty()) {
            Node current = pq.poll();
            if (visited.contains(current.vertex)) {
                continue; // Already processed
            }
            
            visited.add(current.vertex);
            System.out.println("Visited: " + current);
        }
    }
    
    // Top K elements problem
    public static void demonstrateTopK() {
        System.out.println("\n=== Top K Elements ===");
        
        int[] numbers = {7, 10, 4, 3, 20, 15, 8, 2};
        int k = 3;
        
        // Use min heap of size k to find top k elements
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        
        for (int num : numbers) {
            minHeap.offer(num);
            if (minHeap.size() > k) {
                minHeap.poll(); // Remove smallest
            }
        }
        
        System.out.println("Array: " + Arrays.toString(numbers));
        System.out.println("Top " + k + " elements: " + minHeap);
    }
    
    // Meeting room scheduler
    static class Meeting {
        String name;
        int startTime;
        int endTime;
        int priority;
        
        Meeting(String name, int startTime, int endTime, int priority) {
            this.name = name;
            this.startTime = startTime;
            this.endTime = endTime;
            this.priority = priority;
        }
        
        @Override
        public String toString() {
            return name + "(" + startTime + "-" + endTime + ", P" + priority + ")";
        }
    }
    
    public static void demonstrateMeetingScheduler() {
        System.out.println("\n=== Meeting Scheduler ===");
        
        PriorityQueue<Meeting> meetings = new PriorityQueue<>(
            Comparator.comparing((Meeting m) -> m.priority)
                     .thenComparing(m -> m.startTime)
        );
        
        meetings.offer(new Meeting("Team Standup", 9, 10, 2));
        meetings.offer(new Meeting("CEO Review", 10, 11, 1));
        meetings.offer(new Meeting("Code Review", 11, 12, 3));
        meetings.offer(new Meeting("Client Call", 10, 11, 1));
        meetings.offer(new Meeting("Lunch", 12, 13, 4));
        
        System.out.println("Scheduling meetings by priority:");
        while (!meetings.isEmpty()) {
            System.out.println("Next: " + meetings.poll());
        }
    }
    
    public static void main(String[] args) {
        demonstrateDijkstra();
        demonstrateTopK();
        demonstrateMeetingScheduler();
    }
}
```

---

## ArrayDeque

### What is ArrayDeque?
ArrayDeque (Array Double Ended Queue) is a resizable array implementation of the Deque interface. It allows insertion and removal at both ends and is more efficient than LinkedList for most operations.

### Features and Examples

#### 1. Basic Deque Operations
```java
import java.util.*;

public class ArrayDequeBasics {
    public static void main(String[] args) {
        ArrayDeque<String> deque = new ArrayDeque<>();
        
        // Add to both ends
        deque.addFirst("Middle");
        deque.addLast("End");
        deque.addFirst("Start");
        
        System.out.println("Deque: " + deque);
        
        // Peek at both ends
        System.out.println("First: " + deque.peekFirst());
        System.out.println("Last: " + deque.peekLast());
        
        // Remove from both ends
        String first = deque.removeFirst();
        String last = deque.removeLast();
        
        System.out.println("Removed first: " + first);
        System.out.println("Removed last: " + last);
        System.out.println("Remaining: " + deque);
        
        // Use as stack (LIFO)
        ArrayDeque<Integer> stack = new ArrayDeque<>();
        stack.push(1);
        stack.push(2);
        stack.push(3);
        
        System.out.println("\nStack: " + stack);
        System.out.println("Pop: " + stack.pop());
        System.out.println("Peek: " + stack.peek());
        System.out.println("Stack after pop: " + stack);
        
        // Use as queue (FIFO)
        ArrayDeque<String> queue = new ArrayDeque<>();
        queue.offer("First");
        queue.offer("Second");
        queue.offer("Third");
        
        System.out.println("\nQueue: " + queue);
        System.out.println("Poll: " + queue.poll());
        System.out.println("Queue after poll: " + queue);
    }
}
```

#### 2. ArrayDeque vs LinkedList Performance
```java
import java.util.*;

public class ArrayDequeVsLinkedList {
    private static final int SIZE = 100000;
    
    public static void main(String[] args) {
        System.out.println("ArrayDeque vs LinkedList Performance\n");
        
        testStackOperations();
        testQueueOperations();
        testRandomAccess();
    }
    
    private static void testStackOperations() {
        System.out.println("=== Stack Operations (Push/Pop) ===");
        
        // ArrayDeque as stack
        long start = System.nanoTime();
        Deque<Integer> arrayStack = new ArrayDeque<>();
        for (int i = 0; i < SIZE; i++) {
            arrayStack.push(i);
        }
        for (int i = 0; i < SIZE; i++) {
            arrayStack.pop();
        }
        long arrayTime = System.nanoTime() - start;
        
        // LinkedList as stack
        start = System.nanoTime();
        Deque<Integer> linkedStack = new LinkedList<>();
        for (int i = 0; i < SIZE; i++) {
            linkedStack.push(i);
        }
        for (int i = 0; i < SIZE; i++) {
            linkedStack.pop();
        }
        long linkedTime = System.nanoTime() - start;
        
        System.out.printf("ArrayDeque: %.2f ms%n", arrayTime / 1_000_000.0);
        System.out.printf("LinkedList: %.2f ms%n", linkedTime / 1_000_000.0);
        System.out.printf("ArrayDeque is %.1fx faster%n%n", (double)linkedTime / arrayTime);
    }
    
    private static void testQueueOperations() {
        System.out.println("=== Queue Operations (Offer/Poll) ===");
        
        // ArrayDeque as queue
        long start = System.nanoTime();
        Deque<Integer> arrayQueue = new ArrayDeque<>();
        for (int i = 0; i < SIZE; i++) {
            arrayQueue.offer(i);
        }
        for (int i = 0; i < SIZE; i++) {
            arrayQueue.poll();
        }
        long arrayTime = System.nanoTime() - start;
        
        // LinkedList as queue
        start = System.nanoTime();
        Deque<Integer> linkedQueue = new LinkedList<>();
        for (int i = 0; i < SIZE; i++) {
            linkedQueue.offer(i);
        }
        for (int i = 0; i < SIZE; i++) {
            linkedQueue.poll();
        }
        long linkedTime = System.nanoTime() - start;
        
        System.out.printf("ArrayDeque: %.2f ms%n", arrayTime / 1_000_000.0);
        System.out.printf("LinkedList: %.2f ms%n", linkedTime / 1_000_000.0);
        System.out.printf("ArrayDeque is %.1fx faster%n%n", (double)linkedTime / arrayTime);
    }
    
    private static void testRandomAccess() {
        System.out.println("=== Memory Usage Comparison ===");
        
        Runtime runtime = Runtime.getRuntime();
        
        // ArrayDeque memory
        runtime.gc();
        long beforeArray = runtime.totalMemory() - runtime.freeMemory();
        ArrayDeque<Integer> arrayDeque = new ArrayDeque<>();
        for (int i = 0; i < SIZE; i++) {
            arrayDeque.add(i);
        }
        long afterArray = runtime.totalMemory() - runtime.freeMemory();
        
        // LinkedList memory
        runtime.gc();
        long beforeLinked = runtime.totalMemory() - runtime.freeMemory();
        LinkedList<Integer> linkedList = new LinkedList<>();
        for (int i = 0; i < SIZE; i++) {
            linkedList.add(i);
        }
        long afterLinked = runtime.totalMemory() - runtime.freeMemory();
        
        long arrayMemory = afterArray - beforeArray;
        long linkedMemory = afterLinked - beforeLinked;
        
        System.out.printf("ArrayDeque: %d KB%n", arrayMemory / 1024);
        System.out.printf("LinkedList: %d KB%n", linkedMemory / 1024);
        System.out.printf("LinkedList uses %.1fx more memory%n", (double)linkedMemory / arrayMemory);
    }
}
```

#### 3. Practical Applications
```java
import java.util.*;

public class ArrayDequeApplications {
    // Palindrome checker
    public static boolean isPalindrome(String str) {
        ArrayDeque<Character> deque = new ArrayDeque<>();
        
        // Add all characters to deque
        for (char c : str.toLowerCase().toCharArray()) {
            if (Character.isLetterOrDigit(c)) {
                deque.add(c);
            }
        }
        
        // Check if palindrome by comparing from both ends
        while (deque.size() > 1) {
            if (!deque.removeFirst().equals(deque.removeLast())) {
                return false;
            }
        }
        
        return true;
    }
    
    // Sliding window maximum
    public static int[] slidingWindowMaximum(int[] arr, int k) {
        ArrayDeque<Integer> deque = new ArrayDeque<>(); // Store indices
        int[] result = new int[arr.length - k + 1];
        
        for (int i = 0; i < arr.length; i++) {
            // Remove elements outside window
            while (!deque.isEmpty() && deque.peekFirst() <= i - k) {
                deque.pollFirst();
            }
            
            // Remove smaller elements from rear
            while (!deque.isEmpty() && arr[deque.peekLast()] <= arr[i]) {
                deque.pollLast();
            }
            
            deque.offerLast(i);
            
            // Window is complete
            if (i >= k - 1) {
                result[i - k + 1] = arr[deque.peekFirst()];
            }
        }
        
        return result;
    }
    
    // Browser history simulation
    static class BrowserHistory {
        private ArrayDeque<String> history = new ArrayDeque<>();
        private ArrayDeque<String> future = new ArrayDeque<>();
        private String current;
        
        public BrowserHistory(String homepage) {
            this.current = homepage;
        }
        
        public void visit(String url) {
            if (current != null) {
                history.push(current);
            }
            current = url;
            future.clear(); // Clear forward history
        }
        
        public String back() {
            if (!history.isEmpty()) {
                future.push(current);
                current = history.pop();
            }
            return current;
        }
        
        public String forward() {
            if (!future.isEmpty()) {
                history.push(current);
                current = future.pop();
            }
            return current;
        }
        
        public String getCurrentPage() {
            return current;
        }
    }
    
    public static void main(String[] args) {
        // Test palindrome checker
        System.out.println("=== Palindrome Checker ===");
        String[] testStrings = {"racecar", "A man a plan a canal Panama", "hello", "Madam"};
        
        for (String test : testStrings) {
            System.out.println("\"" + test + "\" is palindrome: " + isPalindrome(test));
        }
        
        // Test sliding window maximum
        System.out.println("\n=== Sliding Window Maximum ===");
        int[] arr = {1, 3, -1, -3, 5, 3, 6, 7};
        int k = 3;
        int[] maxInWindows = slidingWindowMaximum(arr, k);
        
        System.out.println("Array: " + Arrays.toString(arr));
        System.out.println("Window size: " + k);
        System.out.println("Max in each window: " + Arrays.toString(maxInWindows));
        
        // Test browser history
        System.out.println("\n=== Browser History ===");
        BrowserHistory browser = new BrowserHistory("google.com");
        
        System.out.println("Current: " + browser.getCurrentPage());
        
        browser.visit("youtube.com");
        System.out.println("After visiting youtube: " + browser.getCurrentPage());
        
        browser.visit("github.com");
        System.out.println("After visiting github: " + browser.getCurrentPage());
        
        System.out.println("Back: " + browser.back());
        System.out.println("Back: " + browser.back());
        
        System.out.println("Forward: " + browser.forward());
        
        browser.visit("stackoverflow.com");
        System.out.println("After visiting stackoverflow: " + browser.getCurrentPage());
        
        System.out.println("Back: " + browser.back());
    }
}
```

---

## LinkedList as Queue

### LinkedList Queue Operations
```java
import java.util.*;

public class LinkedListAsQueue {
    public static void main(String[] args) {
        // LinkedList implements both List and Deque
        LinkedList<String> queue = new LinkedList<>();
        
        // Queue operations (FIFO)
        queue.offer("First");     // Add to rear
        queue.offer("Second");
        queue.offer("Third");
        
        System.out.println("Queue: " + queue);
        
        // Remove from front
        System.out.println("Poll: " + queue.poll());
        System.out.println("After poll: " + queue);
        
        // Deque operations
        queue.addFirst("New First");
        queue.addLast("New Last");
        
        System.out.println("After deque operations: " + queue);
        
        // List operations (random access)
        System.out.println("Element at index 1: " + queue.get(1));
        queue.set(1, "Modified");
        System.out.println("After modification: " + queue);
        
        // Stack operations
        LinkedList<Integer> stack = new LinkedList<>();
        stack.push(1);
        stack.push(2);
        stack.push(3);
        
        System.out.println("\nStack: " + stack);
        System.out.println("Pop: " + stack.pop());
        System.out.println("Stack after pop: " + stack);
        
        // Demonstrate why LinkedList is good for queue/stack
        demonstrateLinkedListAdvantages();
    }
    
    private static void demonstrateLinkedListAdvantages() {
        System.out.println("\n=== LinkedList Advantages for Queue/Stack ===");
        
        LinkedList<String> list = new LinkedList<>();
        
        // Efficient operations at both ends
        long start = System.nanoTime();
        for (int i = 0; i < 50000; i++) {
            list.addFirst("Start-" + i);  // O(1)
            list.addLast("End-" + i);     // O(1)
        }
        for (int i = 0; i < 50000; i++) {
            list.removeFirst();           // O(1)
            list.removeLast();            // O(1)
        }
        long time = System.nanoTime() - start;
        
        System.out.println("LinkedList add/remove at ends: " + (time / 1_000_000) + "ms");
        
        // Compare with ArrayList for insertions at beginning
        ArrayList<String> arrayList = new ArrayList<>();
        
        start = System.nanoTime();
        for (int i = 0; i < 10000; i++) { // Smaller number due to O(n) complexity
            arrayList.add(0, "Start-" + i); // O(n) - shift all elements
        }
        time = System.nanoTime() - start;
        
        System.out.println("ArrayList add at beginning: " + (time / 1_000_000) + "ms");
        System.out.println("LinkedList is much faster for insertions at beginning!");
    }
}
```

---

## Performance Comparison

### Time Complexity Comparison
| Operation | PriorityQueue | ArrayDeque | LinkedList |
|-----------|---------------|------------|------------|
| **offer/add** | O(log n) | O(1) amortized | O(1) |
| **poll/remove** | O(log n) | O(1) | O(1) |
| **peek** | O(1) | O(1) | O(1) |
| **add at beginning** | N/A | O(1) amortized | O(1) |
| **add at end** | N/A | O(1) amortized | O(1) |
| **Random access** | N/A | N/A | O(n) |

### Space Complexity
- **PriorityQueue**: O(n) - Array-based heap
- **ArrayDeque**: O(n) - Circular array
- **LinkedList**: O(n) - Each node has extra pointer overhead

---

## When to Use Which

### Decision Matrix

#### Use PriorityQueue When:
- **Priority-based processing** is required
- **Heap operations** are needed
- **Always need min/max element**
- **Order matters more than insertion order**

#### Use ArrayDeque When:
- **Stack or queue operations** are primary
- **Performance is critical**
- **Memory efficiency** is important
- **No random access** is needed

#### Use LinkedList When:
- **Both List and Queue** operations are needed
- **Frequent insertions/deletions** at arbitrary positions
- **Implementing other data structures**
- **Size varies dramatically**

### Practical Use Cases
```java
import java.util.*;

public class QueueUseCases {
    public static void main(String[] args) {
        demonstrateTaskScheduling();   // PriorityQueue
        demonstrateBreadthFirstSearch(); // ArrayDeque
        demonstrateUndoRedo();         // LinkedList
    }
    
    // Task scheduling with priorities
    private static void demonstrateTaskScheduling() {
        System.out.println("=== Task Scheduling (PriorityQueue) ===");
        
        PriorityQueue<Task> scheduler = new PriorityQueue<>(
            Comparator.comparing(Task::getPriority)
                     .thenComparing(Task::getDeadline)
        );
        
        scheduler.offer(new Task("Code Review", 2, "2024-01-15"));
        scheduler.offer(new Task("Bug Fix", 1, "2024-01-10"));
        scheduler.offer(new Task("Documentation", 3, "2024-01-20"));
        scheduler.offer(new Task("Security Patch", 1, "2024-01-12"));
        
        System.out.println("Processing tasks by priority and deadline:");
        while (!scheduler.isEmpty()) {
            System.out.println("Next: " + scheduler.poll());
        }
        System.out.println();
    }
    
    // Breadth-first search simulation
    private static void demonstrateBreadthFirstSearch() {
        System.out.println("=== BFS Simulation (ArrayDeque) ===");
        
        // Simulate BFS on a tree/graph
        ArrayDeque<String> bfsQueue = new ArrayDeque<>();
        Set<String> visited = new HashSet<>();
        
        bfsQueue.offer("Root");
        
        while (!bfsQueue.isEmpty()) {
            String current = bfsQueue.poll();
            if (visited.contains(current)) continue;
            
            visited.add(current);
            System.out.println("Visiting: " + current);
            
            // Add children based on current node
            switch (current) {
                case "Root":
                    bfsQueue.offer("LeftChild");
                    bfsQueue.offer("RightChild");
                    break;
                case "LeftChild":
                    bfsQueue.offer("LeftLeft");
                    bfsQueue.offer("LeftRight");
                    break;
                case "RightChild":
                    bfsQueue.offer("RightLeft");
                    bfsQueue.offer("RightRight");
                    break;
            }
        }
        System.out.println();
    }
    
    // Undo/Redo functionality
    private static void demonstrateUndoRedo() {
        System.out.println("=== Undo/Redo (LinkedList) ===");
        
        UndoRedoSystem system = new UndoRedoSystem();
        
        system.execute("Create file");
        system.execute("Write content");
        system.execute("Save file");
        
        System.out.println("Current state: " + system.getCurrentState());
        
        system.undo();
        System.out.println("After undo: " + system.getCurrentState());
        
        system.undo();
        System.out.println("After undo: " + system.getCurrentState());
        
        system.redo();
        System.out.println("After redo: " + system.getCurrentState());
        
        system.execute("Add new content");
        System.out.println("After new action: " + system.getCurrentState());
    }
    
    static class Task {
        private String name;
        private int priority;
        private String deadline;
        
        public Task(String name, int priority, String deadline) {
            this.name = name;
            this.priority = priority;
            this.deadline = deadline;
        }
        
        public String getName() { return name; }
        public int getPriority() { return priority; }
        public String getDeadline() { return deadline; }
        
        @Override
        public String toString() {
            return name + " (P" + priority + ", " + deadline + ")";
        }
    }
    
    static class UndoRedoSystem {
        private LinkedList<String> undoStack = new LinkedList<>();
        private LinkedList<String> redoStack = new LinkedList<>();
        private String currentState = "Initial state";
        
        public void execute(String action) {
            undoStack.push(currentState);
            currentState = action;
            redoStack.clear(); // Clear redo stack on new action
        }
        
        public void undo() {
            if (!undoStack.isEmpty()) {
                redoStack.push(currentState);
                currentState = undoStack.pop();
            }
        }
        
        public void redo() {
            if (!redoStack.isEmpty()) {
                undoStack.push(currentState);
                currentState = redoStack.pop();
            }
        }
        
        public String getCurrentState() {
            return currentState;
        }
    }
}
```

---

## Interview Questions

### Q1: What is the difference between Queue and Deque interfaces?
**Answer:**
- **Queue**: Single-ended, FIFO operations (offer, poll, peek)
- **Deque**: Double-ended, operations at both ends (addFirst, addLast, removeFirst, removeLast)

```java
// Queue - FIFO only
Queue<String> queue = new LinkedList<>();
queue.offer("first");   // Add to rear
queue.poll();           // Remove from front

// Deque - Both ends
Deque<String> deque = new ArrayDeque<>();
deque.addFirst("front");  // Add to front
deque.addLast("back");    // Add to back
deque.removeFirst();      // Remove from front
deque.removeLast();       // Remove from back
```

### Q2: How does PriorityQueue maintain order?
**Answer:**
PriorityQueue uses a **binary heap** data structure:

```java
// Internal representation as array:
// For element at index i:
// - Left child at index 2*i + 1
// - Right child at index 2*i + 2
// - Parent at index (i-1)/2

// Min-heap property: parent <= children
// When adding: "bubble up" to maintain heap
// When removing: "bubble down" to maintain heap
```

### Q3: Why is ArrayDeque preferred over LinkedList for stack/queue operations?
**Answer:**
**Performance and Memory advantages:**

| Aspect | ArrayDeque | LinkedList |
|--------|------------|------------|
| **Memory** | Lower overhead | Higher (node pointers) |
| **Cache** | Better locality | Scattered in memory |
| **Operations** | Faster push/pop | More allocations |
| **Iteration** | Faster | Slower |

### Q4: Can PriorityQueue contain duplicate elements?
**Answer:**
Yes, PriorityQueue allows duplicates:

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.addAll(Arrays.asList(3, 1, 4, 1, 5)); // Two 1's allowed

System.out.println(pq); // [1, 1, 4, 3, 5] (heap order, not sorted)

// Removal gets elements in priority order
while (!pq.isEmpty()) {
    System.out.print(pq.poll() + " "); // 1 1 3 4 5
}
```

### Q5: How do you implement a thread-safe queue?
**Answer:**
Several approaches:

```java
// 1. Synchronized wrapper
Queue<String> syncQueue = Collections.synchronizedList(new LinkedList<>());

// 2. Concurrent collections
Queue<String> concurrentQueue = new ConcurrentLinkedQueue<>();
BlockingQueue<String> blockingQueue = new LinkedBlockingQueue<>();

// 3. Manual synchronization
public class SynchronizedQueue<T> {
    private final Queue<T> queue = new LinkedList<>();
    
    public synchronized void offer(T item) { queue.offer(item); }
    public synchronized T poll() { return queue.poll(); }
    public synchronized T peek() { return queue.peek(); }
    public synchronized int size() { return queue.size(); }
}
```

---

## Navigation
- [← Previous: Map Implementations](map-implementations.md)
- [🏠 Main Menu](README.md)
- [→ Next: Iteration Mechanisms](iteration-mechanisms.md)

---

*This guide covers comprehensive Queue implementations for Java interview preparation. Understanding when to use PriorityQueue, ArrayDeque, or LinkedList for different scenarios is crucial for optimal performance.*
