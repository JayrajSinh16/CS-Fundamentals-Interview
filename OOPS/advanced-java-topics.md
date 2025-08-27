# Advanced Java Topics - Enterprise-Level Concepts

## Navigation
- [← Previous: Java 8 Features](java8-features.md)
- [🏠 Main Menu](README.md)

---

## Table of Contents
1. [Generics](#generics)
2. [Reflection API](#reflection-api)
3. [Annotations](#annotations)
4. [Design Patterns in Java](#design-patterns-in-java)
5. [Memory Management](#memory-management)
6. [Concurrency Utilities](#concurrency-utilities)
7. [Interview Questions](#interview-questions)

---

## Generics

### What are Generics?
Generics enable types (classes and interfaces) to be parameters when defining classes, interfaces, and methods. They provide type safety at compile time and eliminate the need for casting.

### Real-world Analogy 📦
Think of generics like **labeled storage containers**:
- **Without generics**: Mixed box - you put anything in, but need to check what you get out
- **With generics**: Labeled box - "Books Only" container ensures you only store and retrieve books
- **Benefit**: No surprises, type safety guaranteed

### Basic Generic Classes

#### 1. Generic Class Implementation
```java
// Generic class with single type parameter
public class Box<T> {
    private T item;
    
    public void setItem(T item) {
        this.item = item;
    }
    
    public T getItem() {
        return item;
    }
    
    public boolean isEmpty() {
        return item == null;
    }
}

// Generic class with multiple type parameters
public class Pair<T, U> {
    private T first;
    private U second;
    
    public Pair(T first, U second) {
        this.first = first;
        this.second = second;
    }
    
    public T getFirst() { return first; }
    public U getSecond() { return second; }
    
    @Override
    public String toString() {
        return "(" + first + ", " + second + ")";
    }
}

// Usage example
public class GenericsExample {
    public static void main(String[] args) {
        // Type-safe containers
        Box<String> stringBox = new Box<>();
        stringBox.setItem("Hello Generics");
        String value = stringBox.getItem(); // No casting needed
        
        Box<Integer> intBox = new Box<>();
        intBox.setItem(42);
        Integer number = intBox.getItem();
        
        // Multiple type parameters
        Pair<String, Integer> nameAge = new Pair<>("Alice", 30);
        Pair<Double, Boolean> scorePass = new Pair<>(85.5, true);
        
        System.out.println("Name-Age: " + nameAge);
        System.out.println("Score-Pass: " + scorePass);
    }
}
```

#### 2. Bounded Type Parameters
```java
// Upper bounded - T must extend Number
public class NumberBox<T extends Number> {
    private T number;
    
    public void setNumber(T number) {
        this.number = number;
    }
    
    public double getDoubleValue() {
        return number.doubleValue(); // Can call Number methods
    }
    
    public boolean isPositive() {
        return number.doubleValue() > 0;
    }
}

// Multiple bounds - T must implement both Comparable and Serializable
public class SortableBox<T extends Comparable<T> & Serializable> {
    private T item;
    
    public void setItem(T item) {
        this.item = item;
    }
    
    public int compareTo(SortableBox<T> other) {
        return this.item.compareTo(other.item);
    }
}

// Usage
public class BoundedGenericsExample {
    public static void main(String[] args) {
        NumberBox<Integer> intBox = new NumberBox<>();
        intBox.setNumber(42);
        System.out.println("Double value: " + intBox.getDoubleValue());
        System.out.println("Is positive: " + intBox.isPositive());
        
        NumberBox<Double> doubleBox = new NumberBox<>();
        doubleBox.setNumber(-15.7);
        System.out.println("Is positive: " + doubleBox.isPositive());
        
        // This would cause compile error:
        // NumberBox<String> stringBox = new NumberBox<>(); // String doesn't extend Number
    }
}
```

#### 3. Wildcards
```java
import java.util.*;

public class WildcardsExample {
    // Upper bounded wildcard - read only
    public static double sumNumbers(List<? extends Number> numbers) {
        double sum = 0.0;
        for (Number num : numbers) {
            sum += num.doubleValue();
        }
        return sum;
    }
    
    // Lower bounded wildcard - write only
    public static void addNumbers(List<? super Integer> numbers) {
        numbers.add(1);
        numbers.add(2);
        numbers.add(3);
    }
    
    // Unbounded wildcard
    public static void printList(List<?> list) {
        for (Object item : list) {
            System.out.println(item);
        }
    }
    
    public static void main(String[] args) {
        // Upper bounded example
        List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
        List<Double> doubles = Arrays.asList(1.1, 2.2, 3.3);
        
        System.out.println("Sum of integers: " + sumNumbers(integers));
        System.out.println("Sum of doubles: " + sumNumbers(doubles));
        
        // Lower bounded example
        List<Number> numbers = new ArrayList<>();
        addNumbers(numbers);
        System.out.println("Numbers: " + numbers);
        
        // Unbounded example
        List<String> strings = Arrays.asList("Hello", "World");
        printList(strings);
        printList(integers);
    }
}
```

---

## Reflection API

### What is Reflection?
Reflection is a feature that allows a program to examine and modify its own structure and behavior at runtime. It provides the ability to inspect classes, interfaces, fields, and methods without knowing their compile-time names.

### Real-world Analogy 🔍
Think of reflection like a **security scanner at an airport**:
- **Normal code**: You know what's in your bag (predefined access to class members)
- **Reflection**: X-ray machine that can see inside any bag (runtime inspection of any class)
- **Power**: Can examine and manipulate contents dynamically
- **Caution**: Powerful but should be used carefully for security reasons

### Basic Reflection Operations

#### 1. Class Information
```java
import java.lang.reflect.*;
import java.util.*;

public class ReflectionBasics {
    static class Person {
        private String name;
        protected int age;
        public String email;
        
        public Person() {}
        
        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }
        
        public void introduce() {
            System.out.println("Hi, I'm " + name);
        }
        
        private void secretMethod() {
            System.out.println("This is a secret!");
        }
    }
    
    public static void main(String[] args) throws Exception {
        Class<Person> clazz = Person.class;
        
        // Alternative ways to get Class object
        // Class<?> clazz2 = Class.forName("ReflectionBasics$Person");
        // Class<?> clazz3 = new Person().getClass();
        
        // Basic class information
        System.out.println("Class name: " + clazz.getSimpleName());
        System.out.println("Package: " + clazz.getPackage());
        System.out.println("Superclass: " + clazz.getSuperclass().getSimpleName());
        
        // Constructors
        System.out.println("\n--- Constructors ---");
        Constructor<?>[] constructors = clazz.getConstructors();
        for (Constructor<?> constructor : constructors) {
            System.out.println("Constructor: " + constructor);
            System.out.println("Parameters: " + constructor.getParameterCount());
        }
        
        // Fields
        System.out.println("\n--- Fields ---");
        Field[] fields = clazz.getDeclaredFields(); // Including private
        for (Field field : fields) {
            System.out.println("Field: " + field.getName() + 
                             " (Type: " + field.getType().getSimpleName() + 
                             ", Modifiers: " + Modifier.toString(field.getModifiers()) + ")");
        }
        
        // Methods
        System.out.println("\n--- Methods ---");
        Method[] methods = clazz.getDeclaredMethods(); // Including private
        for (Method method : methods) {
            System.out.println("Method: " + method.getName() + 
                             " (Return: " + method.getReturnType().getSimpleName() + 
                             ", Modifiers: " + Modifier.toString(method.getModifiers()) + ")");
        }
    }
}
```

#### 2. Dynamic Object Creation and Method Invocation
```java
import java.lang.reflect.*;

public class DynamicReflection {
    static class Calculator {
        public int add(int a, int b) {
            return a + b;
        }
        
        public int multiply(int a, int b) {
            return a * b;
        }
        
        private int secretCalculation(int x) {
            return x * x + 10;
        }
    }
    
    public static void main(String[] args) throws Exception {
        Class<Calculator> clazz = Calculator.class;
        
        // Create instance dynamically
        Constructor<Calculator> constructor = clazz.getConstructor();
        Calculator calc = constructor.newInstance();
        
        // Invoke public method
        Method addMethod = clazz.getMethod("add", int.class, int.class);
        Object result = addMethod.invoke(calc, 5, 3);
        System.out.println("5 + 3 = " + result);
        
        // Invoke private method
        Method secretMethod = clazz.getDeclaredMethod("secretCalculation", int.class);
        secretMethod.setAccessible(true); // Bypass access control
        Object secretResult = secretMethod.invoke(calc, 7);
        System.out.println("Secret calculation(7) = " + secretResult);
        
        // Dynamic method selection
        String methodName = "multiply";
        Method dynamicMethod = clazz.getMethod(methodName, int.class, int.class);
        Object dynamicResult = dynamicMethod.invoke(calc, 4, 6);
        System.out.println("4 * 6 = " + dynamicResult);
    }
}
```

#### 3. Field Access and Modification
```java
import java.lang.reflect.*;

public class FieldReflection {
    static class Person {
        private String name = "Unknown";
        protected int age = 0;
        public String status = "Active";
    }
    
    public static void main(String[] args) throws Exception {
        Person person = new Person();
        Class<Person> clazz = Person.class;
        
        // Access and modify private field
        Field nameField = clazz.getDeclaredField("name");
        nameField.setAccessible(true);
        
        System.out.println("Original name: " + nameField.get(person));
        nameField.set(person, "John Doe");
        System.out.println("Modified name: " + nameField.get(person));
        
        // Access protected field
        Field ageField = clazz.getDeclaredField("age");
        ageField.setAccessible(true);
        ageField.set(person, 30);
        System.out.println("Age: " + ageField.get(person));
        
        // Access public field (no setAccessible needed)
        Field statusField = clazz.getField("status");
        System.out.println("Status: " + statusField.get(person));
        
        // Field information
        System.out.println("\n--- Field Details ---");
        for (Field field : clazz.getDeclaredFields()) {
            System.out.println("Field: " + field.getName());
            System.out.println("  Type: " + field.getType());
            System.out.println("  Modifiers: " + Modifier.toString(field.getModifiers()));
            System.out.println("  Accessible: " + field.isAccessible());
        }
    }
}
```

---

## Annotations

### What are Annotations?
Annotations provide metadata about the program that doesn't directly affect program execution but can be used by tools and frameworks for various purposes like code generation, validation, or configuration.

### Real-world Analogy 🏷️
Think of annotations like **labels on products**:
- **Product**: Your code (methods, classes, fields)
- **Labels**: Annotations (@Override, @Deprecated, etc.)
- **Information**: Tell tools and frameworks how to handle the code
- **Processing**: Scanners (tools) read labels and take appropriate actions

### Built-in Annotations

#### 1. Common Built-in Annotations
```java
import java.util.*;

public class BuiltInAnnotations {
    // @Override - ensures method is actually overriding
    @Override
    public String toString() {
        return "BuiltInAnnotations instance";
    }
    
    // @Deprecated - marks as obsolete
    @Deprecated
    public void oldMethod() {
        System.out.println("This method is deprecated");
    }
    
    // @SuppressWarnings - suppresses compiler warnings
    @SuppressWarnings("unchecked")
    public List<String> getUnsafeList() {
        List rawList = new ArrayList(); // Raw type warning suppressed
        rawList.add("item");
        return rawList;
    }
    
    // @SafeVarargs - suppresses varargs warnings
    @SafeVarargs
    public final <T> void printItems(T... items) {
        for (T item : items) {
            System.out.println(item);
        }
    }
    
    // @FunctionalInterface - ensures interface has exactly one abstract method
    @FunctionalInterface
    interface Calculator {
        int calculate(int a, int b);
        
        // Can have default methods
        default void printResult(int a, int b) {
            System.out.println("Result: " + calculate(a, b));
        }
    }
    
    public static void main(String[] args) {
        BuiltInAnnotations example = new BuiltInAnnotations();
        
        System.out.println(example.toString());
        example.oldMethod(); // Compiler shows deprecation warning
        
        List<String> list = example.getUnsafeList();
        System.out.println("List: " + list);
        
        example.printItems("A", "B", "C");
        example.printItems(1, 2, 3, 4, 5);
        
        // Using functional interface
        Calculator adder = (a, b) -> a + b;
        adder.printResult(5, 3);
    }
}
```

#### 2. Custom Annotations
```java
import java.lang.annotation.*;
import java.lang.reflect.*;

// Annotation for marking test methods
@Retention(RetentionPolicy.RUNTIME) // Available at runtime
@Target(ElementType.METHOD) // Can only be applied to methods
public @interface Test {
    String description() default "";
    int timeout() default 0;
    Class<? extends Exception> expected() default None.class;
    
    // Helper class for default expected exception
    class None extends Exception {}
}

// Annotation for configuration
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.FIELD})
public @interface Config {
    String value();
    String[] tags() default {};
    Priority priority() default Priority.MEDIUM;
    
    enum Priority {
        LOW, MEDIUM, HIGH, CRITICAL
    }
}

// Class using custom annotations
@Config(value = "UserService", tags = {"service", "user"}, priority = Config.Priority.HIGH)
public class UserService {
    
    @Config("database.url")
    private String databaseUrl;
    
    @Test(description = "Test user creation", timeout = 5000)
    public void testCreateUser() {
        System.out.println("Creating user...");
        // Test implementation
    }
    
    @Test(description = "Test invalid user", expected = IllegalArgumentException.class)
    public void testInvalidUser() {
        System.out.println("Testing invalid user...");
        // Would throw IllegalArgumentException
    }
    
    @Test
    public void testDefaultAnnotation() {
        System.out.println("Default test method");
    }
}

// Annotation processor
public class AnnotationProcessor {
    public static void processTestAnnotations(Class<?> clazz) {
        System.out.println("\n--- Processing Test Annotations for " + clazz.getSimpleName() + " ---");
        
        Method[] methods = clazz.getDeclaredMethods();
        for (Method method : methods) {
            if (method.isAnnotationPresent(Test.class)) {
                Test test = method.getAnnotation(Test.class);
                
                System.out.println("Test method: " + method.getName());
                System.out.println("  Description: " + 
                    (test.description().isEmpty() ? "No description" : test.description()));
                System.out.println("  Timeout: " + 
                    (test.timeout() == 0 ? "No timeout" : test.timeout() + "ms"));
                System.out.println("  Expected exception: " + 
                    (test.expected() == Test.None.class ? "None" : test.expected().getSimpleName()));
            }
        }
    }
    
    public static void processConfigAnnotations(Class<?> clazz) {
        System.out.println("\n--- Processing Config Annotations for " + clazz.getSimpleName() + " ---");
        
        // Class-level annotation
        if (clazz.isAnnotationPresent(Config.class)) {
            Config config = clazz.getAnnotation(Config.class);
            System.out.println("Class config:");
            System.out.println("  Value: " + config.value());
            System.out.println("  Tags: " + Arrays.toString(config.tags()));
            System.out.println("  Priority: " + config.priority());
        }
        
        // Field-level annotations
        Field[] fields = clazz.getDeclaredFields();
        for (Field field : fields) {
            if (field.isAnnotationPresent(Config.class)) {
                Config config = field.getAnnotation(Config.class);
                System.out.println("Field config for " + field.getName() + ":");
                System.out.println("  Value: " + config.value());
            }
        }
    }
    
    public static void main(String[] args) {
        processTestAnnotations(UserService.class);
        processConfigAnnotations(UserService.class);
    }
}
```

---

## Design Patterns in Java

### Singleton Pattern
```java
// Thread-safe singleton with enum (best practice)
public enum DatabaseConnection {
    INSTANCE;
    
    private String connectionString;
    
    DatabaseConnection() {
        // Initialize connection
        this.connectionString = "jdbc:mysql://localhost:3306/mydb";
    }
    
    public void connect() {
        System.out.println("Connecting to: " + connectionString);
    }
    
    public void executeQuery(String query) {
        System.out.println("Executing: " + query);
    }
}

// Alternative: Lazy initialization with double-checked locking
public class ConfigManager {
    private static volatile ConfigManager instance;
    private Properties config;
    
    private ConfigManager() {
        config = new Properties();
        // Load configuration
    }
    
    public static ConfigManager getInstance() {
        if (instance == null) {
            synchronized (ConfigManager.class) {
                if (instance == null) {
                    instance = new ConfigManager();
                }
            }
        }
        return instance;
    }
    
    public String getProperty(String key) {
        return config.getProperty(key);
    }
}
```

### Factory Pattern
```java
// Product interface
interface Vehicle {
    void start();
    void stop();
    String getType();
}

// Concrete products
class Car implements Vehicle {
    @Override
    public void start() { System.out.println("Car started"); }
    
    @Override
    public void stop() { System.out.println("Car stopped"); }
    
    @Override
    public String getType() { return "Car"; }
}

class Motorcycle implements Vehicle {
    @Override
    public void start() { System.out.println("Motorcycle started"); }
    
    @Override
    public void stop() { System.out.println("Motorcycle stopped"); }
    
    @Override
    public String getType() { return "Motorcycle"; }
}

class Truck implements Vehicle {
    @Override
    public void start() { System.out.println("Truck started"); }
    
    @Override
    public void stop() { System.out.println("Truck stopped"); }
    
    @Override
    public String getType() { return "Truck"; }
}

// Factory
public class VehicleFactory {
    public enum VehicleType {
        CAR, MOTORCYCLE, TRUCK
    }
    
    public static Vehicle createVehicle(VehicleType type) {
        switch (type) {
            case CAR:
                return new Car();
            case MOTORCYCLE:
                return new Motorcycle();
            case TRUCK:
                return new Truck();
            default:
                throw new IllegalArgumentException("Unknown vehicle type: " + type);
        }
    }
    
    public static void main(String[] args) {
        Vehicle car = VehicleFactory.createVehicle(VehicleType.CAR);
        Vehicle motorcycle = VehicleFactory.createVehicle(VehicleType.MOTORCYCLE);
        
        car.start();
        motorcycle.start();
        
        System.out.println("Created: " + car.getType() + " and " + motorcycle.getType());
    }
}
```

### Observer Pattern
```java
import java.util.*;

// Observer interface
interface NewsObserver {
    void update(String news);
}

// Subject interface
interface NewsPublisher {
    void addObserver(NewsObserver observer);
    void removeObserver(NewsObserver observer);
    void notifyObservers(String news);
}

// Concrete subject
class NewsAgency implements NewsPublisher {
    private List<NewsObserver> observers = new ArrayList<>();
    private String latestNews;
    
    @Override
    public void addObserver(NewsObserver observer) {
        observers.add(observer);
    }
    
    @Override
    public void removeObserver(NewsObserver observer) {
        observers.remove(observer);
    }
    
    @Override
    public void notifyObservers(String news) {
        for (NewsObserver observer : observers) {
            observer.update(news);
        }
    }
    
    public void publishNews(String news) {
        this.latestNews = news;
        System.out.println("Publishing news: " + news);
        notifyObservers(news);
    }
}

// Concrete observers
class TVChannel implements NewsObserver {
    private String name;
    
    public TVChannel(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String news) {
        System.out.println(name + " received news: " + news);
    }
}

class Newspaper implements NewsObserver {
    private String name;
    
    public Newspaper(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String news) {
        System.out.println(name + " will print: " + news);
    }
}

// Usage
public class ObserverPatternExample {
    public static void main(String[] args) {
        NewsAgency agency = new NewsAgency();
        
        TVChannel cnn = new TVChannel("CNN");
        TVChannel bbc = new TVChannel("BBC");
        Newspaper times = new Newspaper("The Times");
        
        agency.addObserver(cnn);
        agency.addObserver(bbc);
        agency.addObserver(times);
        
        agency.publishNews("Breaking: Important event happened!");
        
        System.out.println("\nRemoving BBC from observers...");
        agency.removeObserver(bbc);
        
        agency.publishNews("Update: More details available.");
    }
}
```

---

## Memory Management

### Heap vs Stack Memory
```java
public class MemoryExample {
    // Class variable - stored in Method Area
    static int classVariable = 100;
    
    // Instance variable - stored in Heap
    int instanceVariable = 200;
    
    public void demonstrateMemory() {
        // Local variables - stored in Stack
        int localInt = 10;
        String localString = "Hello"; // Reference in Stack, object in Heap
        
        // Object creation - stored in Heap
        StringBuilder sb = new StringBuilder("World");
        
        // Method call - new frame added to Stack
        processData(localInt, localString);
        
        // Arrays - stored in Heap
        int[] numbers = new int[5];
        Object[] objects = new Object[3];
        
        System.out.println("Local variables and objects created");
    }
    
    private void processData(int value, String text) {
        // Parameters and local variables in this method's stack frame
        int result = value * 2;
        String processed = text.toUpperCase();
        
        System.out.println("Processed: " + processed + ", Result: " + result);
    }
    
    public static void main(String[] args) {
        // main method stack frame
        MemoryExample example = new MemoryExample(); // Object in Heap, reference in Stack
        example.demonstrateMemory();
    }
}
```

### Garbage Collection Awareness
```java
import java.lang.ref.*;
import java.util.*;

public class GarbageCollectionExample {
    private static List<Object> memoryLeakList = new ArrayList<>();
    
    static class LargeObject {
        private byte[] data = new byte[1024 * 1024]; // 1MB
        private String name;
        
        public LargeObject(String name) {
            this.name = name;
        }
        
        @Override
        protected void finalize() throws Throwable {
            System.out.println("Finalizing: " + name);
            super.finalize();
        }
    }
    
    public static void demonstrateGC() {
        System.out.println("=== Garbage Collection Demo ===");
        
        // Creating objects that will become eligible for GC
        for (int i = 0; i < 5; i++) {
            LargeObject obj = new LargeObject("Object-" + i);
            // obj goes out of scope and becomes eligible for GC
        }
        
        // Suggest garbage collection
        System.gc();
        
        try {
            Thread.sleep(1000); // Give GC time to run
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    public static void demonstrateMemoryLeak() {
        System.out.println("\n=== Memory Leak Simulation ===");
        
        // Simulating memory leak - objects never removed from list
        for (int i = 0; i < 3; i++) {
            LargeObject obj = new LargeObject("Leaked-" + i);
            memoryLeakList.add(obj); // Prevents GC
        }
        
        System.gc(); // These objects won't be collected
        
        // To fix the leak, clear the list
        // memoryLeakList.clear();
    }
    
    public static void demonstrateWeakReferences() {
        System.out.println("\n=== Weak References Demo ===");
        
        // Strong reference
        LargeObject strongRef = new LargeObject("StrongRef");
        
        // Weak reference
        WeakReference<LargeObject> weakRef = new WeakReference<>(strongRef);
        
        System.out.println("Before GC - Weak ref object: " + weakRef.get());
        
        // Remove strong reference
        strongRef = null;
        
        // Suggest GC
        System.gc();
        
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        System.out.println("After GC - Weak ref object: " + weakRef.get());
    }
    
    public static void main(String[] args) {
        demonstrateGC();
        demonstrateMemoryLeak();
        demonstrateWeakReferences();
        
        // Monitor memory usage
        Runtime runtime = Runtime.getRuntime();
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        
        System.out.println("\n=== Memory Usage ===");
        System.out.println("Total memory: " + (totalMemory / 1024 / 1024) + " MB");
        System.out.println("Used memory: " + (usedMemory / 1024 / 1024) + " MB");
        System.out.println("Free memory: " + (freeMemory / 1024 / 1024) + " MB");
    }
}
```

---

## Concurrency Utilities

### Executor Framework
```java
import java.util.concurrent.*;
import java.util.*;

public class ExecutorExample {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        // Fixed thread pool
        ExecutorService fixedPool = Executors.newFixedThreadPool(3);
        
        // Cached thread pool
        ExecutorService cachedPool = Executors.newCachedThreadPool();
        
        // Single thread executor
        ExecutorService singleExecutor = Executors.newSingleThreadExecutor();
        
        // Scheduled executor
        ScheduledExecutorService scheduledExecutor = Executors.newScheduledThreadPool(2);
        
        // Submit tasks to fixed pool
        System.out.println("=== Fixed Thread Pool Demo ===");
        List<Future<String>> futures = new ArrayList<>();
        
        for (int i = 1; i <= 5; i++) {
            final int taskId = i;
            Future<String> future = fixedPool.submit(() -> {
                Thread.sleep(1000);
                return "Task " + taskId + " completed by " + Thread.currentThread().getName();
            });
            futures.add(future);
        }
        
        // Get results
        for (Future<String> future : futures) {
            System.out.println(future.get());
        }
        
        // Scheduled tasks
        System.out.println("\n=== Scheduled Tasks Demo ===");
        
        // Schedule one-time task
        ScheduledFuture<String> scheduledTask = scheduledExecutor.schedule(() -> {
            return "Scheduled task executed after 2 seconds";
        }, 2, TimeUnit.SECONDS);
        
        System.out.println(scheduledTask.get());
        
        // Schedule repeating task
        ScheduledFuture<?> repeatingTask = scheduledExecutor.scheduleAtFixedRate(() -> {
            System.out.println("Repeating task: " + new Date());
        }, 0, 1, TimeUnit.SECONDS);
        
        // Let it run for 3 seconds then cancel
        Thread.sleep(3000);
        repeatingTask.cancel(true);
        
        // Shutdown executors
        fixedPool.shutdown();
        cachedPool.shutdown();
        singleExecutor.shutdown();
        scheduledExecutor.shutdown();
        
        // Wait for termination
        if (!fixedPool.awaitTermination(5, TimeUnit.SECONDS)) {
            fixedPool.shutdownNow();
        }
    }
}
```

### CompletableFuture
```java
import java.util.concurrent.*;
import java.util.function.*;

public class CompletableFutureExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // Simple async computation
        CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            return "Hello";
        });
        
        CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1500);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            return "World";
        });
        
        // Combine futures
        CompletableFuture<String> combined = future1.thenCombine(future2, (s1, s2) -> s1 + " " + s2);
        System.out.println("Combined result: " + combined.get());
        
        // Chain operations
        CompletableFuture<String> chained = CompletableFuture
            .supplyAsync(() -> "processing")
            .thenApply(String::toUpperCase)
            .thenApply(s -> s + "...")
            .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + " DONE"));
        
        System.out.println("Chained result: " + chained.get());
        
        // Handle exceptions
        CompletableFuture<String> withException = CompletableFuture
            .supplyAsync(() -> {
                if (Math.random() > 0.5) {
                    throw new RuntimeException("Random error");
                }
                return "Success";
            })
            .handle((result, throwable) -> {
                if (throwable != null) {
                    return "Error handled: " + throwable.getMessage();
                }
                return result;
            });
        
        System.out.println("Exception handled: " + withException.get());
        
        // All of - wait for all to complete
        CompletableFuture<Void> allOf = CompletableFuture.allOf(
            CompletableFuture.runAsync(() -> System.out.println("Task 1")),
            CompletableFuture.runAsync(() -> System.out.println("Task 2")),
            CompletableFuture.runAsync(() -> System.out.println("Task 3"))
        );
        
        allOf.get(); // Wait for all tasks to complete
        System.out.println("All tasks completed");
        
        // Any of - complete when first one finishes
        CompletableFuture<Object> anyOf = CompletableFuture.anyOf(
            CompletableFuture.supplyAsync(() -> {
                try { Thread.sleep(2000); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
                return "Slow task";
            }),
            CompletableFuture.supplyAsync(() -> {
                try { Thread.sleep(500); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
                return "Fast task";
            })
        );
        
        System.out.println("First completed: " + anyOf.get());
    }
}
```

---

## Interview Questions

### Q1: What is the difference between generics and raw types?
**Answer:**
- **Raw types**: Using generic classes without type parameters (legacy compatibility)
- **Generics**: Type-safe, compile-time checking, no casting needed

```java
// Raw type - not recommended
List rawList = new ArrayList();
rawList.add("String");
rawList.add(123); // Can add any type
String item = (String) rawList.get(0); // Casting required, potential ClassCastException

// Generic type - recommended
List<String> genericList = new ArrayList<>();
genericList.add("String");
// genericList.add(123); // Compile error
String item2 = genericList.get(0); // No casting needed
```

### Q2: When would you use reflection and what are its disadvantages?
**Answer:**
**Use cases:**
- Frameworks (Spring, Hibernate) for dependency injection
- Testing frameworks for accessing private methods
- Serialization/deserialization libraries
- Plugin architectures

**Disadvantages:**
- **Performance overhead**: Slower than direct access
- **Security concerns**: Can break encapsulation
- **Code fragility**: Breaks with refactoring
- **Compile-time safety**: No compile-time checking

### Q3: Explain the different types of references in Java.
**Answer:**
```java
// Strong reference - prevents GC
Object obj = new Object();

// Weak reference - allows GC if no strong references
WeakReference<Object> weakRef = new WeakReference<>(obj);

// Soft reference - GC'd when memory is low
SoftReference<Object> softRef = new SoftReference<>(obj);

// Phantom reference - for cleanup actions after GC
PhantomReference<Object> phantomRef = new PhantomReference<>(obj, referenceQueue);
```

### Q4: What are the benefits of using annotations?
**Answer:**
1. **Metadata**: Provide information about code without affecting execution
2. **Code generation**: Tools can generate code based on annotations
3. **Configuration**: Replace XML configuration (Spring, JPA)
4. **Validation**: Runtime validation of constraints
5. **Documentation**: Self-documenting code

### Q5: Explain the difference between Executor and ExecutorService.
**Answer:**
- **Executor**: Simple interface with only `execute(Runnable)` method
- **ExecutorService**: Extends Executor with additional methods:
  - `submit()` - returns Future
  - `shutdown()` - graceful shutdown
  - `shutdownNow()` - forceful shutdown
  - `awaitTermination()` - wait for termination

```java
// Executor - basic execution
Executor executor = Executors.newSingleThreadExecutor();
executor.execute(() -> System.out.println("Task executed"));

// ExecutorService - more control
ExecutorService service = Executors.newFixedThreadPool(3);
Future<String> future = service.submit(() -> "Task result");
String result = future.get();
service.shutdown();
```

---

## Navigation
- [← Previous: Java 8 Features](java8-features.md)
- [🏠 Main Menu](README.md)

---

*This guide covers advanced Java topics essential for senior-level interviews. Master these concepts to demonstrate deep understanding of Java's enterprise-level capabilities and modern programming practices.*
