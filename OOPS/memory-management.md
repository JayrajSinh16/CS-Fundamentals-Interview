# 🧠 Memory Management in Java - Stack vs Heap

Understanding Java memory management is crucial for writing efficient applications and succeeding in technical interviews. Java's automatic memory management through the JVM provides both convenience and complexity that every developer must understand.

## 🎯 Key Concepts

- **Stack Memory**: Stores method calls, local variables, and partial results
- **Heap Memory**: Stores objects, instance variables, and arrays
- **Method Area**: Stores class-level information, static variables, and constants
- **Memory Allocation**: How and where different data types are stored
- **Garbage Collection**: Automatic cleanup of unused objects

## 🌍 Real-World Analogy

**Restaurant Kitchen Analogy**:
- **Stack** = Kitchen counter (temporary workspace, organized, limited space)
- **Heap** = Storage room (long-term storage, larger space, needs periodic cleaning)
- **Method Area** = Recipe book (shared knowledge, permanent reference)
- **Garbage Collector** = Cleaning staff (removes unused items automatically)

## 🏗️ JVM Memory Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        JVM Memory                           │
├─────────────────────────────────────────────────────────────┤
│  Method Area (Metaspace in Java 8+)                        │
│  • Class definitions, method bytecode                       │
│  • Static variables, constants                              │
│  • Runtime constant pool                                    │
├─────────────────────────────────────────────────────────────┤
│  Heap Memory                                                │
│  ┌─────────────────┬─────────────────┬─────────────────┐   │
│  │  Young Gen      │  Old Gen        │  Permanent Gen  │   │
│  │  • Eden         │  • Tenured      │  (Java 7-)      │   │
│  │  • Survivor S0  │  • Long-lived   │  • Class Meta   │   │
│  │  │  Survivor S1  │    objects      │                 │   │
│  └─────────────────┴─────────────────┴─────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│  Stack Memory (Per Thread)                                 │
│  • Method frames                                            │
│  • Local variables                                          │
│  • Method parameters                                        │
│  • Return addresses                                         │
├─────────────────────────────────────────────────────────────┤
│  PC (Program Counter) Register                              │
│  Native Method Stack                                        │
└─────────────────────────────────────────────────────────────┘
```

## 💻 Practical Examples

### Example 1: Stack vs Heap Demonstration

```java
// MemoryDemo.java - Understanding Stack and Heap allocation
public class MemoryDemo {
    
    // Static variables (Method Area)
    private static int staticCounter = 0;
    private static final String CONSTANT_STRING = "I'm in Method Area";
    
    // Instance variables (Heap)
    private int instanceVar;
    private String instanceString;
    private int[] instanceArray;
    
    public MemoryDemo(int value) {
        // Constructor parameters and local variables (Stack)
        this.instanceVar = value;  // 'this' reference on Stack, object on Heap
        this.instanceString = "Instance: " + value;  // String object on Heap
        this.instanceArray = new int[5];  // Array object on Heap, reference on Stack
        
        staticCounter++;  // Static variable in Method Area
    }
    
    public void demonstrateMemoryAllocation() {
        // Local variables (Stack) - primitive values stored directly
        int localInt = 42;
        double localDouble = 3.14;
        boolean localBoolean = true;
        
        // Object references (Stack) - objects themselves (Heap)
        String localString = "Local string object";  // String in Heap
        StringBuilder localBuilder = new StringBuilder("Builder");  // Object in Heap
        
        // Array (Stack reference, Heap object)
        int[] localArray = {1, 2, 3, 4, 5};
        
        // Method call creates new stack frame
        int result = calculateSum(localArray);
        
        System.out.println("=== Stack Variables (Method Frame) ===");
        System.out.println("Local int: " + localInt);
        System.out.println("Local double: " + localDouble);
        System.out.println("Local boolean: " + localBoolean);
        System.out.println("Calculation result: " + result);
        
        System.out.println("\n=== Heap Objects ===");
        System.out.println("Instance variable: " + instanceVar);
        System.out.println("Instance string: " + instanceString);
        System.out.println("Local string: " + localString);
        System.out.println("String builder: " + localBuilder.toString());
        
        System.out.println("\n=== Method Area ===");
        System.out.println("Static counter: " + staticCounter);
        System.out.println("Constant string: " + CONSTANT_STRING);
        
        // Demonstrate array storage
        System.out.println("\n=== Array Storage ===");
        System.out.print("Local array (Heap): ");
        for (int value : localArray) {
            System.out.print(value + " ");
        }
        System.out.println();
        
        System.out.print("Instance array (Heap): ");
        for (int i = 0; i < instanceArray.length; i++) {
            instanceArray[i] = i * i;
            System.out.print(instanceArray[i] + " ");
        }
        System.out.println();
    }
    
    // Method creates new stack frame
    private int calculateSum(int[] array) {
        // Method parameters and local variables in new stack frame
        int sum = 0;
        for (int value : array) {
            sum += value;  // Local variable 'sum' in stack frame
        }
        return sum;  // Return value passed back, stack frame destroyed
    }
    
    public static void main(String[] args) {
        System.out.println("=== Java Memory Management Demonstration ===\n");
        
        // main method stack frame contains these local variables
        MemoryDemo demo1 = new MemoryDemo(10);  // Reference in Stack, object in Heap
        MemoryDemo demo2 = new MemoryDemo(20);  // Reference in Stack, object in Heap
        
        System.out.println("Demo 1:");
        demo1.demonstrateMemoryAllocation();
        
        System.out.println("\n" + "=".repeat(60) + "\n");
        
        System.out.println("Demo 2:");
        demo2.demonstrateMemoryAllocation();
        
        // Memory usage information
        showMemoryUsage();
        
        // Demonstrate memory cleanup
        demo1 = null;  // Remove reference, object eligible for GC
        demo2 = null;  // Remove reference, object eligible for GC
        
        System.gc();  // Suggest garbage collection
        System.out.println("\nAfter suggesting garbage collection:");
        showMemoryUsage();
    }
    
    private static void showMemoryUsage() {
        Runtime runtime = Runtime.getRuntime();
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        long maxMemory = runtime.maxMemory();
        
        System.out.println("\n=== JVM Memory Usage ===");
        System.out.printf("Max Heap Size: %d MB%n", maxMemory / (1024 * 1024));
        System.out.printf("Total Heap: %d MB%n", totalMemory / (1024 * 1024));
        System.out.printf("Used Heap: %d MB%n", usedMemory / (1024 * 1024));
        System.out.printf("Free Heap: %d MB%n", freeMemory / (1024 * 1024));
        System.out.printf("Heap Utilization: %.2f%%%n", 
                (usedMemory * 100.0) / totalMemory);
    }
}

/*
Memory Allocation Summary:
- Primitives in methods: Stack
- Object references in methods: Stack
- Objects themselves: Heap  
- Static variables: Method Area
- Instance variables: Heap (part of object)
- Method bytecode: Method Area
*/
```

### Example 2: Stack Frame Visualization

```java
// StackFrameDemo.java - Visualizing method call stack
public class StackFrameDemo {
    
    private int objectField = 100;  // Stored in Heap as part of object
    
    public void method1() {
        // Stack Frame for method1
        int local1 = 10;
        String str1 = "Method1";
        
        System.out.println("=== Method1 Stack Frame ===");
        System.out.println("Local variable local1: " + local1);
        System.out.println("String reference str1: " + str1);
        System.out.println("Object field: " + objectField);
        
        method2(local1 + 5);  // Pass parameter, create new frame
    }
    
    public void method2(int param) {
        // Stack Frame for method2 (on top of method1 frame)
        int local2 = param * 2;
        char[] charArray = {'A', 'B', 'C'};
        
        System.out.println("\n=== Method2 Stack Frame ===");
        System.out.println("Parameter param: " + param);
        System.out.println("Local variable local2: " + local2);
        System.out.println("Char array reference: " + charArray);
        System.out.println("Object field: " + objectField);
        
        method3(local2);  // Another frame on top
    }
    
    public void method3(int value) {
        // Stack Frame for method3 (on top of method2 frame)
        double result = Math.sqrt(value);
        Object tempObject = new Object();
        
        System.out.println("\n=== Method3 Stack Frame ===");
        System.out.println("Parameter value: " + value);
        System.out.println("Local variable result: " + result);
        System.out.println("Temp object reference: " + tempObject);
        System.out.println("Object field: " + objectField);
        
        // When method3 returns, its stack frame is destroyed
        System.out.println("\nMethod3 returning - frame will be destroyed");
    }
    
    public static void main(String[] args) {
        // Main method stack frame
        StackFrameDemo demo = new StackFrameDemo();  // Reference in Stack
        
        System.out.println("=== Main Method Stack Frame ===");
        System.out.println("Demo object reference: " + demo);
        
        demo.method1();  // Creates method1 frame
        
        System.out.println("\nBack in main - all other frames destroyed");
        System.out.println("Demo object still exists: " + demo);
    }
}

/*
Stack Frame Lifecycle:
1. main() frame created
2. method1() frame created on top
3. method2() frame created on top
4. method3() frame created on top
5. method3() frame destroyed (returns)
6. method2() frame destroyed (returns)
7. method1() frame destroyed (returns)
8. Back to main() frame
*/
```

### Example 3: Memory Leaks and Prevention

```java
// MemoryLeakDemo.java - Common memory issues and solutions
import java.util.*;

public class MemoryLeakDemo {
    
    // Potential memory leak: static collection growing indefinitely
    private static List<String> staticList = new ArrayList<>();
    
    // Instance variables
    private List<LargeObject> objectList;
    private Map<String, String> cache;
    
    public MemoryLeakDemo() {
        this.objectList = new ArrayList<>();
        this.cache = new HashMap<>();
    }
    
    // Example 1: Static collection memory leak
    public void demonstrateStaticCollectionLeak() {
        System.out.println("=== Static Collection Memory Leak ===");
        
        // ❌ Memory leak: adding to static collection without removal
        for (int i = 0; i < 10000; i++) {
            staticList.add("Item " + i);  // Never removed, grows indefinitely
        }
        
        System.out.println("Static list size: " + staticList.size());
        System.out.println("⚠️ Static collections can cause memory leaks!");
        
        // ✅ Solution: Clear when appropriate or use weak references
        if (staticList.size() > 5000) {
            staticList.clear();
            System.out.println("✅ Cleared static list to prevent memory leak");
        }
    }
    
    // Example 2: Listener/Observer memory leak
    public static class EventSource {
        private List<EventListener> listeners = new ArrayList<>();
        
        public void addListener(EventListener listener) {
            listeners.add(listener);
        }
        
        // ❌ Missing remove method can cause memory leaks
        public void removeListener(EventListener listener) {
            listeners.remove(listener);
        }
        
        public void notifyListeners(String event) {
            for (EventListener listener : listeners) {
                listener.onEvent(event);
            }
        }
    }
    
    public interface EventListener {
        void onEvent(String event);
    }
    
    // Example 3: Large object management
    public static class LargeObject {
        private byte[] data = new byte[1024 * 1024];  // 1MB object
        private String id;
        
        public LargeObject(String id) {
            this.id = id;
        }
        
        @Override
        public String toString() {
            return "LargeObject{id='" + id + "', size=1MB}";
        }
    }
    
    public void demonstrateLargeObjectManagement() {
        System.out.println("\n=== Large Object Management ===");
        
        // Create several large objects
        for (int i = 0; i < 10; i++) {
            LargeObject obj = new LargeObject("Object-" + i);
            objectList.add(obj);
        }
        
        System.out.println("Created " + objectList.size() + " large objects");
        showMemoryUsage("After creating large objects");
        
        // ✅ Good practice: Remove references when no longer needed
        objectList.clear();
        System.gc();  // Suggest garbage collection
        
        showMemoryUsage("After clearing large objects");
    }
    
    // Example 4: Cache management
    public void demonstrateCacheManagement() {
        System.out.println("\n=== Cache Management ===");
        
        // Fill cache
        for (int i = 0; i < 1000; i++) {
            cache.put("key" + i, "value" + i + "_with_some_long_content_that_takes_memory");
        }
        
        System.out.println("Cache size: " + cache.size());
        showMemoryUsage("After filling cache");
        
        // ✅ Implement cache size limits
        if (cache.size() > 500) {
            // Remove oldest entries (simple LRU simulation)
            Iterator<String> iterator = cache.keySet().iterator();
            int toRemove = cache.size() - 500;
            while (iterator.hasNext() && toRemove > 0) {
                iterator.next();
                iterator.remove();
                toRemove--;
            }
        }
        
        System.out.println("Cache size after cleanup: " + cache.size());
        showMemoryUsage("After cache cleanup");
    }
    
    // Example 5: Proper resource management
    public void demonstrateResourceManagement() {
        System.out.println("\n=== Resource Management ===");
        
        // ❌ Poor resource management
        // FileInputStream fis = new FileInputStream("file.txt");
        // // File handle not closed - resource leak!
        
        // ✅ Proper resource management with try-with-resources
        try {
            // Simulated resource that implements AutoCloseable
            AutoCloseableResource resource = new AutoCloseableResource("Database Connection");
            System.out.println("Using resource: " + resource);
            // Resource automatically closed when try block exits
        } catch (Exception e) {
            System.out.println("Error handling resource: " + e.getMessage());
        }
    }
    
    // Simulated auto-closeable resource
    public static class AutoCloseableResource implements AutoCloseable {
        private String name;
        
        public AutoCloseableResource(String name) {
            this.name = name;
            System.out.println("✅ Resource opened: " + name);
        }
        
        @Override
        public void close() {
            System.out.println("✅ Resource closed: " + name);
        }
        
        @Override
        public String toString() {
            return name;
        }
    }
    
    private static void showMemoryUsage(String context) {
        Runtime runtime = Runtime.getRuntime();
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        
        System.out.printf("%s - Used: %d MB, Free: %d MB%n", 
                context, usedMemory / (1024 * 1024), freeMemory / (1024 * 1024));
    }
    
    public static void main(String[] args) {
        System.out.println("=== Memory Leak Prevention Demonstration ===");
        
        MemoryLeakDemo demo = new MemoryLeakDemo();
        
        demo.demonstrateStaticCollectionLeak();
        demo.demonstrateLargeObjectManagement();
        demo.demonstrateCacheManagement();
        demo.demonstrateResourceManagement();
        
        System.out.println("\n=== Memory Management Best Practices ===");
        System.out.println("✅ Clear collections when no longer needed");
        System.out.println("✅ Remove listeners/observers properly");
        System.out.println("✅ Use try-with-resources for AutoCloseable objects");
        System.out.println("✅ Implement cache size limits");
        System.out.println("✅ Set object references to null when done");
        System.out.println("✅ Be careful with static collections");
    }
}

/*
Common Memory Leak Causes:
1. Static collections that grow indefinitely
2. Listeners/observers not removed
3. Resources not properly closed
4. Cache without size limits
5. Circular references (less common in Java)
*/
```

### Example 4: Memory Monitoring and Tuning

```java
// MemoryMonitor.java - Monitor and analyze memory usage
import java.lang.management.*;
import java.util.*;

public class MemoryMonitor {
    
    private static final int OBJECT_COUNT = 100000;
    
    public static void main(String[] args) {
        MemoryMonitor monitor = new MemoryMonitor();
        
        System.out.println("=== Memory Monitoring and Analysis ===\n");
        
        monitor.analyzeInitialMemory();
        monitor.simulateMemoryUsage();
        monitor.demonstrateGarbageCollection();
        monitor.showDetailedMemoryInfo();
        monitor.demonstrateMemoryTuning();
    }
    
    public void analyzeInitialMemory() {
        System.out.println("=== Initial Memory State ===");
        displayMemoryInfo();
        displayGCInfo();
    }
    
    public void simulateMemoryUsage() {
        System.out.println("\n=== Simulating Memory Usage ===");
        
        List<String> strings = new ArrayList<>();
        List<int[]> arrays = new ArrayList<>();
        
        System.out.println("Creating " + OBJECT_COUNT + " objects...");
        
        long startTime = System.currentTimeMillis();
        
        // Create many objects to observe memory allocation
        for (int i = 0; i < OBJECT_COUNT; i++) {
            strings.add("String object number " + i + " with some content");
            arrays.add(new int[]{i, i * 2, i * 3, i * 4, i * 5});
            
            // Periodic memory check
            if (i % 20000 == 0 && i > 0) {
                System.out.printf("Created %d objects - ", i);
                displayMemoryInfo();
            }
        }
        
        long endTime = System.currentTimeMillis();
        System.out.printf("Object creation completed in %d ms%n", endTime - startTime);
        
        System.out.println("\nAfter creating all objects:");
        displayMemoryInfo();
        
        // Clear references to make objects eligible for GC
        strings.clear();
        arrays.clear();
        
        System.out.println("\nAfter clearing references:");
        displayMemoryInfo();
    }
    
    public void demonstrateGarbageCollection() {
        System.out.println("\n=== Garbage Collection Demonstration ===");
        
        long gcCountBefore = getTotalGCCount();
        long gcTimeBefore = getTotalGCTime();
        
        System.out.println("GC stats before collection:");
        System.out.println("GC Count: " + gcCountBefore);
        System.out.println("GC Time: " + gcTimeBefore + " ms");
        
        // Force garbage collection
        System.out.println("\nForcing garbage collection...");
        long startTime = System.currentTimeMillis();
        System.gc();
        long endTime = System.currentTimeMillis();
        
        // Wait a bit for GC to complete
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        long gcCountAfter = getTotalGCCount();
        long gcTimeAfter = getTotalGCTime();
        
        System.out.println("\nGC stats after collection:");
        System.out.println("GC Count: " + gcCountAfter + " (+" + (gcCountAfter - gcCountBefore) + ")");
        System.out.println("GC Time: " + gcTimeAfter + " ms (+" + (gcTimeAfter - gcTimeBefore) + " ms)");
        System.out.println("Manual GC request time: " + (endTime - startTime) + " ms");
        
        displayMemoryInfo();
    }
    
    public void showDetailedMemoryInfo() {
        System.out.println("\n=== Detailed Memory Information ===");
        
        MemoryMXBean memoryBean = ManagementFactory.getMemoryMXBean();
        
        // Heap memory usage
        MemoryUsage heapUsage = memoryBean.getHeapMemoryUsage();
        System.out.println("Heap Memory Usage:");
        printMemoryUsage(heapUsage);
        
        // Non-heap memory usage (Method Area, etc.)
        MemoryUsage nonHeapUsage = memoryBean.getNonHeapMemoryUsage();
        System.out.println("\nNon-Heap Memory Usage:");
        printMemoryUsage(nonHeapUsage);
        
        // Memory pool information
        System.out.println("\nMemory Pools:");
        List<MemoryPoolMXBean> memoryPools = ManagementFactory.getMemoryPoolMXBeans();
        for (MemoryPoolMXBean pool : memoryPools) {
            System.out.printf("Pool: %s (Type: %s)%n", pool.getName(), pool.getType());
            if (pool.getUsage() != null) {
                printMemoryUsage(pool.getUsage());
            }
            System.out.println();
        }
    }
    
    public void demonstrateMemoryTuning() {
        System.out.println("\n=== Memory Tuning Information ===");
        
        Runtime runtime = Runtime.getRuntime();
        long maxMemory = runtime.maxMemory();
        long totalMemory = runtime.totalMemory();
        
        System.out.println("JVM Memory Configuration:");
        System.out.printf("Max Heap Size (-Xmx): %d MB%n", maxMemory / (1024 * 1024));
        System.out.printf("Initial Heap Size (-Xms): %d MB%n", totalMemory / (1024 * 1024));
        System.out.printf("Available Processors: %d%n", runtime.availableProcessors());
        
        System.out.println("\nMemory Tuning Parameters:");
        System.out.println("java -Xms512m -Xmx2g -XX:NewRatio=3 MyApp");
        System.out.println("  -Xms512m: Initial heap size 512MB");
        System.out.println("  -Xmx2g: Maximum heap size 2GB");
        System.out.println("  -XX:NewRatio=3: Old:Young generation ratio 3:1");
        
        System.out.println("\nGarbage Collector Options:");
        System.out.println("  -XX:+UseG1GC: Use G1 garbage collector");
        System.out.println("  -XX:+UseParallelGC: Use parallel garbage collector");
        System.out.println("  -XX:+UseConcMarkSweepGC: Use CMS garbage collector");
        
        System.out.println("\nMonitoring Options:");
        System.out.println("  -XX:+PrintGC: Print garbage collection info");
        System.out.println("  -XX:+PrintGCDetails: Detailed GC information");
        System.out.println("  -Xloggc:gc.log: Log GC to file");
    }
    
    private void displayMemoryInfo() {
        Runtime runtime = Runtime.getRuntime();
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        long maxMemory = runtime.maxMemory();
        
        System.out.printf("Memory: Used=%dMB, Free=%dMB, Total=%dMB, Max=%dMB%n",
                usedMemory / (1024 * 1024),
                freeMemory / (1024 * 1024),
                totalMemory / (1024 * 1024),
                maxMemory / (1024 * 1024));
    }
    
    private void displayGCInfo() {
        List<GarbageCollectorMXBean> gcBeans = ManagementFactory.getGarbageCollectorMXBeans();
        System.out.println("Garbage Collectors:");
        for (GarbageCollectorMXBean gcBean : gcBeans) {
            System.out.printf("  %s: Collections=%d, Time=%dms%n",
                    gcBean.getName(), gcBean.getCollectionCount(), gcBean.getCollectionTime());
        }
    }
    
    private long getTotalGCCount() {
        List<GarbageCollectorMXBean> gcBeans = ManagementFactory.getGarbageCollectorMXBeans();
        long total = 0;
        for (GarbageCollectorMXBean gcBean : gcBeans) {
            total += gcBean.getCollectionCount();
        }
        return total;
    }
    
    private long getTotalGCTime() {
        List<GarbageCollectorMXBean> gcBeans = ManagementFactory.getGarbageCollectorMXBeans();
        long total = 0;
        for (GarbageCollectorMXBean gcBean : gcBeans) {
            total += gcBean.getCollectionTime();
        }
        return total;
    }
    
    private void printMemoryUsage(MemoryUsage usage) {
        System.out.printf("  Used: %d MB%n", usage.getUsed() / (1024 * 1024));
        System.out.printf("  Committed: %d MB%n", usage.getCommitted() / (1024 * 1024));
        System.out.printf("  Max: %s%n", 
                usage.getMax() == -1 ? "undefined" : (usage.getMax() / (1024 * 1024)) + " MB");
    }
}

/*
Memory Monitoring Tools:
1. jstat: JVM statistics
2. jmap: Memory map
3. jhat: Heap analysis tool
4. VisualVM: Visual profiling
5. JConsole: JMX monitoring
6. Eclipse MAT: Memory analyzer
*/
```

## 📊 Memory Comparison Table

| Memory Area | Purpose | Storage | Scope | Garbage Collected |
|-------------|---------|---------|-------|-------------------|
| **Stack** | Method execution | Local variables, method parameters | Thread-local | No (automatic cleanup) |
| **Heap** | Object storage | Objects, instance variables, arrays | JVM-wide | Yes |
| **Method Area** | Class information | Class metadata, static variables | JVM-wide | Limited (classes can be unloaded) |
| **PC Register** | Execution tracking | Current instruction pointer | Thread-local | No |
| **Native Method Stack** | Native calls | Native method execution | Thread-local | No |

## ⚠️ Common Memory Issues

### 1. Stack Overflow
```java
// ❌ Infinite recursion causes stack overflow
public void infiniteRecursion(int n) {
    System.out.println(n);
    infiniteRecursion(n + 1);  // Never stops, stack frames accumulate
}

// ✅ Proper recursion with base case
public int factorial(int n) {
    if (n <= 1) return 1;  // Base case prevents infinite recursion
    return n * factorial(n - 1);
}
```

### 2. Memory Leaks
```java
// ❌ Memory leak example
private static List<Object> cache = new ArrayList<>();
public void addToCache(Object obj) {
    cache.add(obj);  // Never removed, grows indefinitely
}

// ✅ Proper cache management
private static Map<String, Object> cache = new LinkedHashMap<String, Object>(100, 0.75f, true) {
    @Override
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > 100;  // Limit cache size
    }
};
```

### 3. OutOfMemoryError
```java
// ❌ Can cause OutOfMemoryError
List<byte[]> bigObjects = new ArrayList<>();
while (true) {
    bigObjects.add(new byte[1024 * 1024]);  // Creates 1MB objects indefinitely
}

// ✅ Monitor memory usage
Runtime runtime = Runtime.getRuntime();
if (runtime.freeMemory() < runtime.totalMemory() * 0.1) {
    // Take action when memory is low
    cleanupResources();
}
```

## 🔥 Interview Questions & Answers

### Q1: Explain the difference between Stack and Heap memory in Java.
**Answer**: 
- **Stack**: Thread-specific memory for method calls, local variables, and parameters. Fast access, automatically managed, limited size
- **Heap**: Shared memory for objects and instance variables. Larger space, garbage collected, slower access than stack
- **Key Difference**: Stack stores method execution context, Heap stores object data

### Q2: What happens to local variables when a method finishes execution?
**Answer**: Local variables are stored in the method's stack frame. When the method finishes:
1. **Stack frame is destroyed** - all local variables are immediately deallocated
2. **Primitive values are lost** - they were stored directly in the stack
3. **Object references are lost** - but objects remain in heap until garbage collected
4. **Return value is passed** to calling method before frame destruction

### Q3: How does Java prevent memory leaks compared to C/C++?
**Answer**: Java prevents memory leaks through:
1. **Automatic Garbage Collection** - unused objects are automatically removed
2. **No manual memory management** - no malloc/free operations
3. **Reference tracking** - GC can determine when objects are unreachable
4. **However**: Logic leaks still possible (static collections, listeners, unclosed resources)

### Q4: What is the difference between shallow and deep copying in terms of memory?
**Answer**: 
- **Shallow Copy**: Copies object references, both copies point to same heap objects
- **Deep Copy**: Creates new objects in heap, completely independent copies
- **Memory Impact**: Shallow uses less memory but changes affect both copies, deep uses more memory but provides isolation

### Q5: How can you monitor and optimize Java memory usage?
**Answer**: 
1. **Monitoring**: Use JConsole, VisualVM, jstat, or profiling tools
2. **JVM Parameters**: Tune -Xms, -Xmx, -XX:NewRatio for heap sizing
3. **Code Optimization**: Clear collections, remove listeners, use try-with-resources
4. **Garbage Collector**: Choose appropriate GC algorithm (G1, Parallel, CMS)
5. **Memory Analysis**: Use heap dumps and memory analyzers to find leaks

---
[← Back to Main Guide](./README.md) | [← Previous: JDK vs JRE vs JVM](./jdk-jre-jvm.md) | [Next: Garbage Collection →](./garbage-collection.md)
