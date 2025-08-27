# 🗑️ Garbage Collection in Java - Automatic Memory Management

Garbage Collection (GC) is one of Java's most powerful features, automatically managing memory by removing unused objects. Understanding GC is crucial for writing efficient applications and optimizing performance in production environments.

## 🎯 Key Concepts

- **Automatic Memory Management**: No manual memory allocation/deallocation
- **Reachability**: Objects are collected when no references exist
- **Generational Hypothesis**: Most objects die young
- **GC Algorithms**: Different strategies for different use cases
- **Performance Impact**: GC pauses affect application responsiveness

## 🌍 Real-World Analogy

**Office Cleaning Service Analogy**:
- **Garbage Collector** = Professional cleaning service
- **Heap Memory** = Office building with multiple floors
- **Young Objects** = Daily trash (papers, coffee cups) - cleaned frequently
- **Old Objects** = Furniture and equipment - cleaned less frequently
- **GC Pause** = Brief moments when office is closed for deep cleaning
- **Memory Pressure** = Office getting too cluttered, needs immediate cleaning

## 🏗️ Generational Garbage Collection

```
┌─────────────────────────────────────────────────────────────┐
│                        Java Heap Memory                    │
├─────────────────────────────────────────────────────────────┤
│  Young Generation (Frequently Collected)                   │
│  ┌─────────────┬─────────────┬─────────────┐              │
│  │    Eden     │ Survivor S0 │ Survivor S1 │              │
│  │             │             │             │              │
│  │ New objects │ Survived 1  │ Survived 2  │              │
│  │ created     │ GC cycle    │ GC cycles   │              │
│  │ here        │             │             │              │
│  └─────────────┴─────────────┴─────────────┘              │
│                       ↓                                     │
│              Objects that survive multiple                  │
│              young GC cycles are promoted                   │
│                       ↓                                     │
├─────────────────────────────────────────────────────────────┤
│  Old Generation (Tenured - Less Frequently Collected)      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Long-lived Objects                     │   │
│  │           Promoted from Young Gen                   │   │
│  │          • Service instances                        │   │
│  │          • Configuration objects                    │   │
│  │          • Large data structures                    │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│  Permanent Generation (Java 7-) / Metaspace (Java 8+)     │
│  • Class metadata                                          │
│  • Method bytecode                                         │
│  • Constant pool                                           │
└─────────────────────────────────────────────────────────────┘
```

## 💻 Practical Examples

### Example 1: Understanding Object Lifecycle and GC

```java
// GCLifecycleDemo.java - Demonstrates object lifecycle and garbage collection
import java.lang.ref.WeakReference;
import java.util.*;

public class GCLifecycleDemo {
    
    private static List<Object> roots = new ArrayList<>();
    private static int objectCounter = 0;
    
    // Class to demonstrate object lifecycle
    static class TrackableObject {
        private final int id;
        private final byte[] data;
        private static int totalCreated = 0;
        
        public TrackableObject(int size) {
            this.id = ++totalCreated;
            this.data = new byte[size];
            objectCounter++;
            System.out.printf("Created object %d (size: %d bytes)%n", id, size);
        }
        
        @Override
        protected void finalize() throws Throwable {
            // Note: finalize() is deprecated in Java 9+, use Cleaner API instead
            objectCounter--;
            System.out.printf("Object %d finalized (GC collected)%n", id);
            super.finalize();
        }
        
        public int getId() { return id; }
        
        public static int getTotalCreated() { return totalCreated; }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Garbage Collection Lifecycle Demonstration ===\n");
        
        GCLifecycleDemo demo = new GCLifecycleDemo();
        demo.demonstrateObjectLifecycle();
        demo.demonstrateReachability();
        demo.demonstrateGenerationalGC();
        demo.demonstrateWeakReferences();
    }
    
    public void demonstrateObjectLifecycle() {
        System.out.println("=== Object Lifecycle Demonstration ===");
        
        showMemoryStats("Initial state");
        
        // Create objects in local scope
        {
            TrackableObject obj1 = new TrackableObject(1024);     // Will be GC'd
            TrackableObject obj2 = new TrackableObject(2048);     // Will be GC'd
            TrackableObject obj3 = new TrackableObject(4096);     // Will be kept alive
            
            roots.add(obj3);  // Keep reference to obj3
            
            System.out.println("Objects created, obj3 added to roots");
            showMemoryStats("After object creation");
            
        } // obj1 and obj2 go out of scope, eligible for GC
        
        System.out.println("\nLocal scope ended, forcing GC...");
        System.gc();
        sleep(100);  // Give GC time to run
        
        showMemoryStats("After forced GC");
        System.out.println("Note: obj3 should still be alive (in roots list)\n");
    }
    
    public void demonstrateReachability() {
        System.out.println("=== Object Reachability Demonstration ===");
        
        // Create a chain of references
        List<TrackableObject> chain = new ArrayList<>();
        
        for (int i = 0; i < 5; i++) {
            chain.add(new TrackableObject(1024));
        }
        
        System.out.println("Created chain of " + chain.size() + " objects");
        showMemoryStats("After creating chain");
        
        // Remove reference to chain - all objects become unreachable
        chain = null;
        
        System.out.println("Removed reference to chain, forcing GC...");
        System.gc();
        sleep(100);
        
        showMemoryStats("After removing chain reference");
        System.out.println();
    }
    
    public void demonstrateGenerationalGC() {
        System.out.println("=== Generational GC Demonstration ===");
        
        List<TrackableObject> survivors = new ArrayList<>();
        
        // Simulate different object generations
        System.out.println("Creating young objects (most will die quickly):");
        
        for (int generation = 1; generation <= 3; generation++) {
            System.out.printf("\n--- Generation %d ---\n", generation);
            
            // Create many short-lived objects
            for (int i = 0; i < 10; i++) {
                TrackableObject temp = new TrackableObject(512);
                
                // Only some objects survive to next generation
                if (i % 3 == 0) {
                    survivors.add(temp);
                    System.out.printf("Object %d promoted to survivors\n", temp.getId());
                }
                // Others go out of scope immediately (young generation candidates)
            }
            
            // Minor GC simulation
            System.out.println("Simulating minor GC...");
            System.gc();
            sleep(50);
            
            showMemoryStats("After generation " + generation);
        }
        
        System.out.println("\nLong-lived objects in survivors list: " + survivors.size());
        
        // Clear survivors (simulate major GC)
        survivors.clear();
        System.out.println("Cleared survivors, forcing major GC...");
        System.gc();
        sleep(100);
        
        showMemoryStats("After major GC simulation");
        System.out.println();
    }
    
    public void demonstrateWeakReferences() {
        System.out.println("=== Weak References Demonstration ===");
        
        TrackableObject strongRef = new TrackableObject(2048);
        WeakReference<TrackableObject> weakRef = new WeakReference<>(strongRef);
        
        System.out.println("Created strong and weak references to same object");
        System.out.println("Strong ref exists: " + (strongRef != null));
        System.out.println("Weak ref target exists: " + (weakRef.get() != null));
        
        // Remove strong reference
        strongRef = null;
        
        System.out.println("\nRemoved strong reference, forcing GC...");
        System.gc();
        sleep(100);
        
        System.out.println("Strong ref exists: " + (strongRef != null));
        System.out.println("Weak ref target exists: " + (weakRef.get() != null));
        System.out.println("Weak references don't prevent garbage collection!\n");
    }
    
    private void showMemoryStats(String context) {
        Runtime runtime = Runtime.getRuntime();
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        
        System.out.printf("%s: Used=%dKB, Free=%dKB, Objects=%d\n",
                context,
                usedMemory / 1024,
                freeMemory / 1024,
                objectCounter);
    }
    
    private void sleep(int ms) {
        try {
            Thread.sleep(ms);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}

/*
Object Lifecycle States:
1. Created - new object allocated in heap
2. Strongly Reachable - accessible via references
3. Softly Reachable - only soft references remain
4. Weakly Reachable - only weak references remain
5. Phantom Reachable - only phantom references remain
6. Unreachable - no references, eligible for GC
7. Finalized - finalize() called (if exists)
8. Reclaimed - memory freed
*/
```

### Example 2: Different Garbage Collection Algorithms

```java
// GCAlgorithmDemo.java - Demonstrates different GC characteristics
import java.lang.management.*;
import java.util.*;
import java.util.concurrent.*;

public class GCAlgorithmDemo {
    
    private static final int ALLOCATION_SIZE = 1024 * 1024; // 1MB
    private static final int ITERATIONS = 100;
    
    public static void main(String[] args) {
        System.out.println("=== Garbage Collection Algorithms Demonstration ===\n");
        
        GCAlgorithmDemo demo = new GCAlgorithmDemo();
        demo.showCurrentGCInfo();
        demo.simulateWorkload();
        demo.demonstrateGCTuning();
    }
    
    public void showCurrentGCInfo() {
        System.out.println("=== Current GC Configuration ===");
        
        List<GarbageCollectorMXBean> gcBeans = ManagementFactory.getGarbageCollectorMXBeans();
        
        for (GarbageCollectorMXBean gcBean : gcBeans) {
            System.out.printf("GC Name: %s\n", gcBean.getName());
            System.out.printf("Collection Count: %d\n", gcBean.getCollectionCount());
            System.out.printf("Collection Time: %d ms\n", gcBean.getCollectionTime());
            System.out.printf("Memory Pools: %s\n", Arrays.toString(gcBean.getMemoryPoolNames()));
            System.out.println();
        }
        
        // Show memory pools
        System.out.println("Memory Pools:");
        List<MemoryPoolMXBean> memoryPools = ManagementFactory.getMemoryPoolMXBeans();
        for (MemoryPoolMXBean pool : memoryPools) {
            MemoryUsage usage = pool.getUsage();
            if (usage != null) {
                System.out.printf("Pool: %s, Type: %s, Used: %d MB\n",
                        pool.getName(),
                        pool.getType(),
                        usage.getUsed() / (1024 * 1024));
            }
        }
        System.out.println();
    }
    
    public void simulateWorkload() {
        System.out.println("=== Simulating Different Workload Patterns ===");
        
        // Pattern 1: Many short-lived objects (Young generation stress)
        simulateYoungGenWorkload();
        
        // Pattern 2: Long-lived objects (Old generation stress)
        simulateOldGenWorkload();
        
        // Pattern 3: Mixed workload
        simulateMixedWorkload();
    }
    
    private void simulateYoungGenWorkload() {
        System.out.println("--- Young Generation Workload ---");
        
        long startGCCount = getTotalGCCount();
        long startGCTime = getTotalGCTime();
        long startTime = System.currentTimeMillis();
        
        // Create many short-lived objects
        for (int i = 0; i < ITERATIONS; i++) {
            List<byte[]> tempObjects = new ArrayList<>();
            
            // Create objects that will quickly become garbage
            for (int j = 0; j < 10; j++) {
                tempObjects.add(new byte[ALLOCATION_SIZE]);
            }
            
            // Objects go out of scope and become eligible for GC
            // This simulates typical web request processing
        }
        
        long endTime = System.currentTimeMillis();
        long endGCCount = getTotalGCCount();
        long endGCTime = getTotalGCTime();
        
        System.out.printf("Execution time: %d ms\n", endTime - startTime);
        System.out.printf("GC collections: %d\n", endGCCount - startGCCount);
        System.out.printf("GC time: %d ms\n", endGCTime - startGCTime);
        System.out.println("Characteristics: High allocation rate, short object lifetime\n");
    }
    
    private void simulateOldGenWorkload() {
        System.out.println("--- Old Generation Workload ---");
        
        long startGCCount = getTotalGCCount();
        long startGCTime = getTotalGCTime();
        long startTime = System.currentTimeMillis();
        
        // Create long-lived objects
        List<byte[]> longLivedObjects = new ArrayList<>();
        
        for (int i = 0; i < ITERATIONS / 2; i++) {
            // Objects that will survive multiple GC cycles
            longLivedObjects.add(new byte[ALLOCATION_SIZE]);
            
            // Simulate some work
            if (i % 10 == 0) {
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
        }
        
        long endTime = System.currentTimeMillis();
        long endGCCount = getTotalGCCount();
        long endGCTime = getTotalGCTime();
        
        System.out.printf("Execution time: %d ms\n", endTime - startTime);
        System.out.printf("GC collections: %d\n", endGCCount - startGCCount);
        System.out.printf("GC time: %d ms\n", endGCTime - startGCTime);
        System.out.printf("Long-lived objects created: %d\n", longLivedObjects.size());
        System.out.println("Characteristics: Low allocation rate, long object lifetime\n");
        
        // Clean up to prevent memory issues
        longLivedObjects.clear();
    }
    
    private void simulateMixedWorkload() {
        System.out.println("--- Mixed Workload ---");
        
        long startGCCount = getTotalGCCount();
        long startGCTime = getTotalGCTime();
        long startTime = System.currentTimeMillis();
        
        List<byte[]> survivors = new ArrayList<>();
        
        for (int i = 0; i < ITERATIONS; i++) {
            // Create short-lived objects
            for (int j = 0; j < 5; j++) {
                byte[] temp = new byte[ALLOCATION_SIZE / 2];
                // Most become garbage immediately
            }
            
            // Occasionally create long-lived objects
            if (i % 10 == 0) {
                survivors.add(new byte[ALLOCATION_SIZE]);
            }
            
            // Occasionally remove old objects
            if (i % 20 == 0 && !survivors.isEmpty()) {
                survivors.remove(0);
            }
        }
        
        long endTime = System.currentTimeMillis();
        long endGCCount = getTotalGCCount();
        long endGCTime = getTotalGCTime();
        
        System.out.printf("Execution time: %d ms\n", endTime - startTime);
        System.out.printf("GC collections: %d\n", endGCCount - startGCCount);
        System.out.printf("GC time: %d ms\n", endGCTime - startGCTime);
        System.out.printf("Surviving objects: %d\n", survivors.size());
        System.out.println("Characteristics: Mixed allocation patterns, realistic workload\n");
        
        survivors.clear();
    }
    
    public void demonstrateGCTuning() {
        System.out.println("=== GC Tuning Information ===");
        
        Runtime runtime = Runtime.getRuntime();
        
        System.out.println("Current JVM Memory Settings:");
        System.out.printf("Max Heap: %d MB\n", runtime.maxMemory() / (1024 * 1024));
        System.out.printf("Total Heap: %d MB\n", runtime.totalMemory() / (1024 * 1024));
        System.out.printf("Free Heap: %d MB\n", runtime.freeMemory() / (1024 * 1024));
        
        System.out.println("\nCommon GC Algorithms and Use Cases:");
        
        System.out.println("\n1. Serial GC (-XX:+UseSerialGC):");
        System.out.println("   • Single-threaded");
        System.out.println("   • Best for: Small applications, single-core machines");
        System.out.println("   • Pause time: Short for small heaps");
        
        System.out.println("\n2. Parallel GC (-XX:+UseParallelGC) [Default in Java 8]:");
        System.out.println("   • Multi-threaded for both young and old generation");
        System.out.println("   • Best for: Throughput-focused applications");
        System.out.println("   • Pause time: Can be longer but good throughput");
        
        System.out.println("\n3. G1 GC (-XX:+UseG1GC) [Default in Java 9+]:");
        System.out.println("   • Low-latency collector");
        System.out.println("   • Best for: Large heaps (>4GB), latency-sensitive apps");
        System.out.println("   • Pause time: Predictable, configurable target");
        
        System.out.println("\n4. CMS GC (-XX:+UseConcMarkSweepGC) [Deprecated]:");
        System.out.println("   • Concurrent collector for old generation");
        System.out.println("   • Best for: Low-latency requirements");
        System.out.println("   • Pause time: Low but can fragment memory");
        
        System.out.println("\n5. ZGC (-XX:+UseZGC) [Java 11+]:");
        System.out.println("   • Ultra-low latency collector");
        System.out.println("   • Best for: Very large heaps, strict latency requirements");
        System.out.println("   • Pause time: <10ms regardless of heap size");
        
        System.out.println("\nGC Tuning Parameters:");
        System.out.println("  -Xms<size>: Initial heap size");
        System.out.println("  -Xmx<size>: Maximum heap size");
        System.out.println("  -XX:NewRatio=<ratio>: Old/Young generation ratio");
        System.out.println("  -XX:MaxGCPauseMillis=<ms>: Target pause time (G1)");
        System.out.println("  -XX:GCTimeRatio=<ratio>: Throughput goal");
        
        System.out.println("\nMonitoring Options:");
        System.out.println("  -XX:+PrintGC: Basic GC information");
        System.out.println("  -XX:+PrintGCDetails: Detailed GC information");
        System.out.println("  -XX:+PrintGCTimeStamps: Include timestamps");
        System.out.println("  -Xloggc:<file>: Log GC to file");
    }
    
    private long getTotalGCCount() {
        List<GarbageCollectorMXBean> gcBeans = ManagementFactory.getGarbageCollectorMXBeans();
        return gcBeans.stream().mapToLong(GarbageCollectorMXBean::getCollectionCount).sum();
    }
    
    private long getTotalGCTime() {
        List<GarbageCollectorMXBean> gcBeans = ManagementFactory.getGarbageCollectorMXBeans();
        return gcBeans.stream().mapToLong(GarbageCollectorMXBean::getCollectionTime).sum();
    }
}

/*
GC Algorithm Characteristics:

Serial GC:
- Single-threaded, stop-the-world
- Good for small applications

Parallel GC:
- Multi-threaded, stop-the-world
- Good throughput for batch jobs

G1 GC:
- Low-latency, incremental
- Predictable pause times

CMS GC (deprecated):
- Concurrent old generation collection
- Can cause fragmentation

ZGC/Shenandoah:
- Ultra-low latency
- Suitable for very large heaps
*/
```

### Example 3: Memory Leak Detection and GC Analysis

```java
// MemoryLeakGCDemo.java - Demonstrates memory leaks and their GC impact
import java.lang.management.*;
import java.util.*;
import java.util.concurrent.*;

public class MemoryLeakGCDemo {
    
    // Potential memory leak: static collection
    private static Map<String, Object> globalCache = new HashMap<>();
    private static List<EventListener> listeners = new ArrayList<>();
    
    // Interface for listener pattern
    interface EventListener {
        void onEvent(String event);
    }
    
    // Class that holds references and can cause leaks
    static class LeakyClass implements EventListener {
        private final String id;
        private final byte[] data;
        private final List<Object> references;
        
        public LeakyClass(String id) {
            this.id = id;
            this.data = new byte[1024 * 100]; // 100KB per object
            this.references = new ArrayList<>();
        }
        
        @Override
        public void onEvent(String event) {
            // Store event data (potential leak)
            references.add(new EventData(event, new Date()));
        }
        
        public String getId() { return id; }
        public int getReferenceCount() { return references.size(); }
    }
    
    static class EventData {
        private final String event;
        private final Date timestamp;
        private final byte[] payload = new byte[1024]; // 1KB per event
        
        public EventData(String event, Date timestamp) {
            this.event = event;
            this.timestamp = timestamp;
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Memory Leak and GC Impact Demonstration ===\n");
        
        MemoryLeakGCDemo demo = new MemoryLeakGCDemo();
        
        demo.showInitialState();
        demo.simulateMemoryLeak();
        demo.demonstrateGCPressure();
        demo.showCleanupImpact();
    }
    
    public void showInitialState() {
        System.out.println("=== Initial Memory State ===");
        showMemoryAndGCStats("Initial");
    }
    
    public void simulateMemoryLeak() {
        System.out.println("\n=== Simulating Memory Leaks ===");
        
        // Leak 1: Growing static collection
        System.out.println("Creating objects and adding to static cache...");
        for (int i = 0; i < 1000; i++) {
            LeakyClass obj = new LeakyClass("Object-" + i);
            globalCache.put("key-" + i, obj);
            
            // Add as listener (another potential leak)
            listeners.add(obj);
            
            if (i % 200 == 0) {
                showMemoryAndGCStats("After " + (i + 1) + " objects");
            }
        }
        
        // Leak 2: Listener pattern leak
        System.out.println("\nTriggering events (causes listener memory growth)...");
        for (int i = 0; i < 5000; i++) {
            triggerEvent("Event-" + i);
            
            if (i % 1000 == 0) {
                showMemoryAndGCStats("After " + (i + 1) + " events");
            }
        }
        
        System.out.println("Memory leak simulation complete");
        showMemoryAndGCStats("After leak simulation");
    }
    
    public void demonstrateGCPressure() {
        System.out.println("\n=== Demonstrating GC Pressure ===");
        
        long startGCCount = getTotalGCCount();
        long startGCTime = getTotalGCTime();
        
        // Create memory pressure to force frequent GC
        System.out.println("Creating memory pressure...");
        
        List<byte[]> temporaryObjects = new ArrayList<>();
        
        for (int i = 0; i < 2000; i++) {
            // Create large temporary objects
            temporaryObjects.add(new byte[1024 * 1024]); // 1MB each
            
            // Periodically clear some to simulate realistic usage
            if (i % 100 == 0 && temporaryObjects.size() > 50) {
                for (int j = 0; j < 25; j++) {
                    temporaryObjects.remove(0);
                }
            }
            
            if (i % 500 == 0) {
                long currentGCCount = getTotalGCCount();
                long currentGCTime = getTotalGCTime();
                
                System.out.printf("Iteration %d: GC Count: %d (+%d), GC Time: %d ms (+%d ms)\n",
                        i,
                        currentGCCount,
                        currentGCCount - startGCCount,
                        currentGCTime,
                        currentGCTime - startGCTime);
                
                showMemoryUsage();
            }
        }
        
        temporaryObjects.clear();
        
        long endGCCount = getTotalGCCount();
        long endGCTime = getTotalGCTime();
        
        System.out.printf("\nGC Pressure Results:\n");
        System.out.printf("Total GC Collections: %d\n", endGCCount - startGCCount);
        System.out.printf("Total GC Time: %d ms\n", endGCTime - startGCTime);
        System.out.printf("Average GC Time: %.2f ms\n", 
                (double)(endGCTime - startGCTime) / Math.max(1, endGCCount - startGCCount));
    }
    
    public void showCleanupImpact() {
        System.out.println("\n=== Cleanup Impact on GC ===");
        
        showMemoryAndGCStats("Before cleanup");
        
        long cleanupStartTime = System.currentTimeMillis();
        long startGCCount = getTotalGCCount();
        long startGCTime = getTotalGCTime();
        
        // Cleanup memory leaks
        System.out.println("Cleaning up memory leaks...");
        
        // Clear static collections
        globalCache.clear();
        listeners.clear();
        
        // Force garbage collection
        System.out.println("Forcing garbage collection...");
        System.gc();
        
        // Wait for GC to complete
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        long cleanupEndTime = System.currentTimeMillis();
        long endGCCount = getTotalGCCount();
        long endGCTime = getTotalGCTime();
        
        System.out.printf("Cleanup completed in %d ms\n", cleanupEndTime - cleanupStartTime);
        System.out.printf("GC collections during cleanup: %d\n", endGCCount - startGCCount);
        System.out.printf("GC time during cleanup: %d ms\n", endGCTime - startGCTime);
        
        showMemoryAndGCStats("After cleanup");
        
        // Demonstrate improved GC performance
        demonstrateImprovedPerformance();
    }
    
    private void demonstrateImprovedPerformance() {
        System.out.println("\n=== Performance After Cleanup ===");
        
        long startTime = System.currentTimeMillis();
        long startGCCount = getTotalGCCount();
        long startGCTime = getTotalGCTime();
        
        // Perform same operations as before
        List<Object> tempObjects = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            tempObjects.add(new byte[1024 * 100]); // 100KB objects
        }
        
        tempObjects.clear();
        
        long endTime = System.currentTimeMillis();
        long endGCCount = getTotalGCCount();
        long endGCTime = getTotalGCTime();
        
        System.out.printf("Operation time: %d ms\n", endTime - startTime);
        System.out.printf("GC collections: %d\n", endGCCount - startGCCount);
        System.out.printf("GC time: %d ms\n", endGCTime - startGCTime);
        System.out.println("Performance should be better with clean memory!");
    }
    
    private void triggerEvent(String event) {
        for (EventListener listener : listeners) {
            listener.onEvent(event);
        }
    }
    
    private void showMemoryAndGCStats(String context) {
        System.out.printf("\n--- %s ---\n", context);
        showMemoryUsage();
        showGCStats();
    }
    
    private void showMemoryUsage() {
        Runtime runtime = Runtime.getRuntime();
        long maxMemory = runtime.maxMemory();
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        
        System.out.printf("Memory: Used=%dMB, Free=%dMB, Total=%dMB, Max=%dMB\n",
                usedMemory / (1024 * 1024),
                freeMemory / (1024 * 1024),
                totalMemory / (1024 * 1024),
                maxMemory / (1024 * 1024));
        
        double usagePercent = (usedMemory * 100.0) / maxMemory;
        System.out.printf("Heap utilization: %.1f%%\n", usagePercent);
    }
    
    private void showGCStats() {
        List<GarbageCollectorMXBean> gcBeans = ManagementFactory.getGarbageCollectorMXBeans();
        
        for (GarbageCollectorMXBean gcBean : gcBeans) {
            System.out.printf("GC %s: Collections=%d, Time=%dms\n",
                    gcBean.getName(),
                    gcBean.getCollectionCount(),
                    gcBean.getCollectionTime());
        }
    }
    
    private long getTotalGCCount() {
        List<GarbageCollectorMXBean> gcBeans = ManagementFactory.getGarbageCollectorMXBeans();
        return gcBeans.stream().mapToLong(GarbageCollectorMXBean::getCollectionCount).sum();
    }
    
    private long getTotalGCTime() {
        List<GarbageCollectorMXBean> gcBeans = ManagementFactory.getGarbageCollectorMXBeans();
        return gcBeans.stream().mapToLong(GarbageCollectorMXBean::getCollectionTime).sum();
    }
}

/*
Memory Leak Impact on GC:
1. Increased GC frequency
2. Longer GC pause times
3. Reduced available memory
4. Potential OutOfMemoryError
5. Degraded application performance

Prevention Strategies:
1. Clear collections when done
2. Remove listeners properly
3. Use weak references when appropriate
4. Monitor memory usage
5. Profile applications regularly
*/
```

## 📊 GC Algorithm Comparison

| Algorithm | Type | Pause Time | Throughput | Best Use Case |
|-----------|------|------------|------------|---------------|
| **Serial GC** | Stop-the-world | Short (small heaps) | Good | Single-core, small apps |
| **Parallel GC** | Stop-the-world | Medium | High | Batch processing, throughput |
| **G1 GC** | Incremental | Predictable | Good | Large heaps, low latency |
| **CMS GC** | Concurrent | Low | Medium | Low latency (deprecated) |
| **ZGC** | Concurrent | Ultra-low (<10ms) | Good | Very large heaps |

## 🎯 GC Tuning Guidelines

### 1. Heap Sizing
```bash
# Initial and maximum heap size
-Xms2g -Xmx4g

# Young generation sizing
-XX:NewRatio=3           # Old:Young = 3:1
-XX:MaxNewSize=1g        # Maximum young generation size
```

### 2. GC Selection
```bash
# G1 GC (recommended for most applications)
-XX:+UseG1GC -XX:MaxGCPauseMillis=200

# Parallel GC (for throughput)
-XX:+UseParallelGC -XX:GCTimeRatio=99

# ZGC (for ultra-low latency)
-XX:+UseZGC
```

### 3. Monitoring
```bash
# GC logging
-XX:+PrintGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps
-Xloggc:gc.log

# JFR (Java Flight Recorder)
-XX:+FlightRecorder -XX:StartFlightRecording=duration=60s,filename=gc.jfr
```

## ⚠️ Common GC Issues

### 1. Frequent Minor GC
```java
// ❌ High allocation rate
for (int i = 0; i < 1000000; i++) {
    String temp = "String " + i;  // Creates many temporary objects
}

// ✅ Reduce allocations
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000000; i++) {
    sb.append("String ").append(i);  // Reuse StringBuilder
}
```

### 2. Long GC Pauses
```java
// ❌ Large objects causing major GC
List<byte[]> largeObjects = new ArrayList<>();
largeObjects.add(new byte[100 * 1024 * 1024]);  // 100MB object

// ✅ Break down large objects
List<List<byte[]>> chunkedObjects = new ArrayList<>();
for (int i = 0; i < 100; i++) {
    chunkedObjects.add(Arrays.asList(new byte[1024 * 1024]));  // 1MB chunks
}
```

### 3. Memory Leaks
```java
// ❌ Static collection growing indefinitely
private static List<Object> cache = new ArrayList<>();

// ✅ Bounded cache with cleanup
private static final int MAX_SIZE = 1000;
private static Map<String, Object> cache = new LinkedHashMap<String, Object>(MAX_SIZE, 0.75f, true) {
    @Override
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > MAX_SIZE;
    }
};
```

## 🔥 Interview Questions & Answers

### Q1: What is garbage collection and how does it work in Java?
**Answer**: Garbage collection automatically manages memory by removing objects that are no longer reachable. Process:
1. **Mark**: Identify reachable objects starting from GC roots
2. **Sweep**: Remove unreachable objects
3. **Compact**: Defragment memory to reduce holes
4. **Generational**: Separate young (frequently collected) and old (less frequently collected) objects

### Q2: Explain the difference between minor and major garbage collection.
**Answer**: 
- **Minor GC**: Cleans young generation (Eden + Survivor spaces), fast and frequent
- **Major GC**: Cleans old generation, slower and less frequent  
- **Full GC**: Cleans entire heap (young + old + metaspace), slowest but most thorough
- **Generational hypothesis**: Most objects die young, so minor GC is more efficient

### Q3: What are GC roots and why are they important?
**Answer**: GC roots are starting points for reachability analysis:
- **Static variables**: Class-level references
- **Local variables**: In active method frames
- **JNI references**: Native code references
- **Thread objects**: Active threads
Objects reachable from GC roots are kept alive; unreachable objects are collected.

### Q4: How can you tune garbage collection performance?
**Answer**: 
1. **Choose appropriate GC**: G1 for low latency, Parallel for throughput
2. **Size heap properly**: -Xms and -Xmx settings
3. **Tune generations**: -XX:NewRatio for young/old ratio
4. **Set pause targets**: -XX:MaxGCPauseMillis for G1
5. **Monitor and profile**: Use GC logs, profilers to identify bottlenecks

### Q5: What causes OutOfMemoryError and how can you prevent it?
**Answer**: Causes:
- **Heap space**: Objects cannot be allocated
- **Metaspace**: Too many classes loaded
- **Memory leaks**: Objects not properly dereferenced

Prevention:
- **Increase heap size**: -Xmx parameter
- **Fix memory leaks**: Clear collections, remove listeners
- **Optimize object usage**: Reuse objects, reduce allocations
- **Monitor memory**: Use profiling tools regularly

---
[← Back to Main Guide](./README.md) | [← Previous: Memory Management](./memory-management.md) | [Next: String Handling →](./string-handling.md)
