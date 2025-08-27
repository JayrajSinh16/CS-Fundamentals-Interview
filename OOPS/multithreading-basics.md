# 🧵 Multithreading Basics - Concurrency and Parallel Programming

Multithreading enables concurrent execution of multiple threads within a single program, allowing better resource utilization and improved performance. Java provides comprehensive support for multithreading through the Thread class, Runnable interface, and java.util.concurrent package.

## 🎯 Key Concepts

- **Thread**: Lightweight subprocess with independent execution path
- **Concurrency**: Multiple threads making progress simultaneously
- **Parallelism**: Multiple threads executing simultaneously on different cores
- **Synchronization**: Coordinating thread access to shared resources
- **Thread Safety**: Ensuring correct behavior in multithreaded environment
- **Deadlock**: Circular dependency preventing thread progress

## 🌍 Real-World Analogy

**Restaurant Kitchen**:
- **Thread** = Chef working on different dishes
- **Concurrency** = Multiple chefs working on orders simultaneously
- **Synchronization** = Taking turns using shared equipment (oven, stove)
- **Deadlock** = Two chefs waiting for each other's ingredients
- **Thread Pool** = Fixed number of chefs handling all orders

## 📋 Thread Creation and Management

```java
// ThreadBasics.java
public class ThreadBasics {
    
    // Method 1: Extending Thread class
    static class WorkerThread extends Thread {
        private String workerName;
        private int taskCount;
        
        public WorkerThread(String workerName, int taskCount) {
            this.workerName = workerName;
            this.taskCount = taskCount;
        }
        
        @Override
        public void run() {
            for (int i = 1; i <= taskCount; i++) {
                System.out.printf("%s executing task %d%n", workerName, i);
                try {
                    Thread.sleep(1000); // Simulate work
                } catch (InterruptedException e) {
                    System.out.printf("%s interrupted%n", workerName);
                    return;
                }
            }
            System.out.printf("%s completed all tasks%n", workerName);
        }
    }
    
    // Method 2: Implementing Runnable interface
    static class TaskRunner implements Runnable {
        private String taskName;
        private int iterations;
        
        public TaskRunner(String taskName, int iterations) {
            this.taskName = taskName;
            this.iterations = iterations;
        }
        
        @Override
        public void run() {
            for (int i = 1; i <= iterations; i++) {
                System.out.printf("Task %s - iteration %d [Thread: %s]%n", 
                        taskName, i, Thread.currentThread().getName());
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return;
                }
            }
        }
    }
    
    public static void demonstrateThreadCreation() {
        System.out.println("=== Thread Creation Methods ===");
        
        // Using Thread class
        WorkerThread worker1 = new WorkerThread("Worker-1", 3);
        WorkerThread worker2 = new WorkerThread("Worker-2", 3);
        
        worker1.start();
        worker2.start();
        
        // Using Runnable interface
        Thread task1 = new Thread(new TaskRunner("DataProcessing", 3));
        Thread task2 = new Thread(new TaskRunner("FileUpload", 3));
        
        task1.start();
        task2.start();
        
        // Lambda expression (Java 8+)
        Thread lambdaThread = new Thread(() -> {
            for (int i = 1; i <= 3; i++) {
                System.out.printf("Lambda task %d%n", i);
                try {
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return;
                }
            }
        });
        lambdaThread.start();
    }
    
    public static void main(String[] args) {
        demonstrateThreadCreation();
    }
}
```

## 🔒 Synchronization and Thread Safety

```java
// ThreadSynchronization.java
import java.util.concurrent.locks.*;

public class ThreadSynchronization {
    
    // Shared resource without synchronization (thread-unsafe)
    static class UnsafeCounter {
        private int count = 0;
        
        public void increment() {
            count++; // Not atomic operation
        }
        
        public int getCount() {
            return count;
        }
    }
    
    // Synchronized methods
    static class SafeCounter {
        private int count = 0;
        
        public synchronized void increment() {
            count++;
        }
        
        public synchronized int getCount() {
            return count;
        }
    }
    
    // Using explicit locks
    static class LockCounter {
        private int count = 0;
        private final ReentrantLock lock = new ReentrantLock();
        
        public void increment() {
            lock.lock();
            try {
                count++;
            } finally {
                lock.unlock();
            }
        }
        
        public int getCount() {
            lock.lock();
            try {
                return count;
            } finally {
                lock.unlock();
            }
        }
    }
    
    // Producer-Consumer pattern
    static class Buffer {
        private final Queue<Integer> queue = new LinkedList<>();
        private final int capacity;
        private final Object lock = new Object();
        
        public Buffer(int capacity) {
            this.capacity = capacity;
        }
        
        public void produce(int item) throws InterruptedException {
            synchronized (lock) {
                while (queue.size() >= capacity) {
                    lock.wait(); // Wait for space
                }
                queue.offer(item);
                System.out.printf("Produced: %d [Queue size: %d]%n", item, queue.size());
                lock.notifyAll(); // Notify consumers
            }
        }
        
        public int consume() throws InterruptedException {
            synchronized (lock) {
                while (queue.isEmpty()) {
                    lock.wait(); // Wait for items
                }
                int item = queue.poll();
                System.out.printf("Consumed: %d [Queue size: %d]%n", item, queue.size());
                lock.notifyAll(); // Notify producers
                return item;
            }
        }
    }
    
    public static void demonstrateSynchronization() {
        System.out.println("=== Thread Synchronization ===");
        
        // Test unsafe vs safe counters
        UnsafeCounter unsafeCounter = new UnsafeCounter();
        SafeCounter safeCounter = new SafeCounter();
        
        // Create multiple threads incrementing counters
        Runnable incrementTask = () -> {
            for (int i = 0; i < 1000; i++) {
                unsafeCounter.increment();
                safeCounter.increment();
            }
        };
        
        Thread[] threads = new Thread[5];
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(incrementTask);
            threads[i].start();
        }
        
        // Wait for all threads to complete
        for (Thread thread : threads) {
            try {
                thread.join();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        
        System.out.printf("Unsafe counter result: %d (expected: 5000)%n", unsafeCounter.getCount());
        System.out.printf("Safe counter result: %d (expected: 5000)%n", safeCounter.getCount());
        
        // Producer-Consumer example
        Buffer buffer = new Buffer(5);
        
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    buffer.produce(i);
                    Thread.sleep(100);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    buffer.consume();
                    Thread.sleep(150);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        producer.start();
        consumer.start();
    }
    
    public static void main(String[] args) {
        demonstrateSynchronization();
    }
}
```

## 🔥 Interview Questions & Answers

### Q1: What's the difference between extending Thread and implementing Runnable?
**Answer**: 
- **Extending Thread**: Direct thread creation but limits inheritance (single inheritance)
- **Implementing Runnable**: More flexible, allows extending other classes, better design
- **Best practice**: Use Runnable interface for better code organization and reusability

### Q2: How do you prevent deadlock in multithreaded applications?
**Answer**: 
- **Lock ordering**: Always acquire locks in same order
- **Timeout**: Use tryLock() with timeout
- **Avoid nested locks**: Minimize lock scope
- **Lock-free algorithms**: Use atomic operations when possible

### Q3: What's the difference between synchronized and volatile?
**Answer**: 
- **synchronized**: Provides mutual exclusion and visibility guarantees
- **volatile**: Provides visibility guarantee only, no atomicity
- **Use synchronized**: For compound operations requiring atomicity
- **Use volatile**: For simple visibility requirements (flags, status)

### Q4: Explain the Producer-Consumer problem and its solution.
**Answer**: 
- **Problem**: Synchronizing producers adding items and consumers removing items
- **Solution**: Use wait()/notify() or BlockingQueue for coordination
- **Key concepts**: Buffer capacity, thread communication, preventing race conditions

### Q5: What are thread pools and why are they useful?
**Answer**: 
- **Thread pool**: Pre-created threads that execute tasks
- **Benefits**: Reduced thread creation overhead, controlled concurrency, resource management
- **ExecutorService**: Java's thread pool implementation
- **Types**: FixedThreadPool, CachedThreadPool, ScheduledThreadPool

---
[← Previous: Collections Framework](./collections-framework.md) | [Next: Java 8 Features →](./java8-features.md)
