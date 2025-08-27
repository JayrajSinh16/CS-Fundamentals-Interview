# 🔌 Interfaces in Java - Contracts and Multiple Inheritance

Interfaces in Java define contracts that classes must follow, providing a way to achieve multiple inheritance of type and establish common behavior across unrelated classes. They represent pure abstraction and are fundamental to object-oriented design principles.

## 🎯 Key Concepts

- **Interface**: A reference type that defines a contract of methods
- **Contract**: Set of method signatures that implementing classes must provide
- **Multiple Inheritance**: Classes can implement multiple interfaces
- **Abstract Methods**: Methods without implementation (default behavior)
- **Default Methods**: Methods with implementation in interfaces (Java 8+)
- **Static Methods**: Utility methods that belong to the interface (Java 8+)
- **Constants**: All variables in interfaces are public static final

## 🌍 Real-World Analogy

**Electronic Device Standards Analogy**:
- **Interface** = USB standard specification (defines connection contract)
- **Implementing Classes** = Devices that support USB (phones, laptops, printers)
- **Multiple Inheritance** = Device supporting multiple standards (USB, Bluetooth, WiFi)
- **Default Methods** = Standard behaviors all USB devices share
- **Abstract Methods** = Required operations each device must implement differently
- **Constants** = Standard specifications (voltage, data transfer rates)

## 📋 Interface Structure and Rules

```
Interface Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                     Interface Components                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  public interface InterfaceName {                              │
│                                                                 │
│    // Constants (implicitly public static final)               │
│    int CONSTANT_VALUE = 100;                                   │
│    String DEFAULT_NAME = "DefaultName";                        │
│                                                                 │
│    // Abstract methods (implicitly public abstract)            │
│    void abstractMethod();                                       │
│    String getInformation();                                     │
│    boolean performAction(String parameter);                    │
│                                                                 │
│    // Default methods (Java 8+, with implementation)           │
│    default void defaultMethod() {                              │
│        System.out.println("Default behavior");                │
│    }                                                            │
│                                                                 │
│    // Static methods (Java 8+, utility methods)                │
│    static void staticUtilityMethod() {                         │
│        System.out.println("Static utility");                  │
│    }                                                            │
│                                                                 │
│    // Private methods (Java 9+, helper methods)                │
│    private void helperMethod() {                               │
│        System.out.println("Private helper");                  │
│    }                                                            │
│  }                                                              │
└─────────────────────────────────────────────────────────────────┘

Interface Evolution (Java Versions):
┌─────────────────────────────────────────────────────────────────┐
│                    Interface Features by Version               │
├─────────────────────────────────────────────────────────────────┤
│  Java 1.0-7:                                                   │
│  • Abstract methods only                                       │
│  • Public static final constants                              │
│  • Multiple inheritance of type                               │
│                                                                 │
│  Java 8:                                                       │
│  • Default methods (with implementation)                      │
│  • Static methods                                             │
│  • Functional interfaces (@FunctionalInterface)               │
│                                                                 │
│  Java 9:                                                       │
│  • Private methods (helper methods)                           │
│  • Private static methods                                     │
│                                                                 │
│  Java 17+:                                                     │
│  • Sealed interfaces (restricted inheritance)                 │
│  • Pattern matching enhancements                              │
└─────────────────────────────────────────────────────────────────┘

Interface Implementation Rules:
┌─────────────────────────────────────────────────────────────────┐
│                     Implementation Constraints                 │
├─────────────────────────────────────────────────────────────────┤
│  ✅ Must implement:                                             │
│  • All abstract methods                                        │
│  • Maintain method signatures exactly                         │
│  • Use public access modifier for methods                     │
│                                                                 │
│  🔄 Can override:                                              │
│  • Default methods (optional)                                 │
│  • Provide more specific implementation                       │
│                                                                 │
│  ❌ Cannot:                                                     │
│  • Override static methods                                    │
│  • Override private methods                                   │
│  • Add instance variables                                     │
│  • Reduce method visibility                                   │
│                                                                 │
│  🌟 Multiple inheritance handling:                             │
│  • Diamond problem resolution through default methods         │
│  • Explicit override required for conflicts                   │
│  • Interface method wins over class method                    │
└─────────────────────────────────────────────────────────────────┘
```

## 💻 Practical Examples

### Example 1: Basic Interface Implementation

```java
// BasicInterfaceExample.java - Fundamental interface concepts
import java.util.*;

public class BasicInterfaceExample {
    
    // Basic interface with different method types
    interface Drawable {
        // Constants (implicitly public static final)
        String DEFAULT_COLOR = "BLACK";
        int MAX_COORDINATES = 1000;
        
        // Abstract methods (implicitly public abstract)
        void draw();
        void setColor(String color);
        void setPosition(int x, int y);
        
        // Default method (Java 8+)
        default void reset() {
            setColor(DEFAULT_COLOR);
            setPosition(0, 0);
            System.out.println("Shape reset to default state");
        }
        
        // Static method (Java 8+)
        static void printDrawingInfo() {
            System.out.println("Drawing with max coordinates: " + MAX_COORDINATES);
        }
        
        // Default method for area calculation
        default double getArea() {
            return 0.0; // Default implementation
        }
    }
    
    // Another interface for resizable objects
    interface Resizable {
        void resize(double factor);
        
        default void doubleSize() {
            resize(2.0);
        }
        
        default void halveSize() {
            resize(0.5);
        }
    }
    
    // Interface for objects that can be rotated
    interface Rotatable {
        void rotate(double angle);
        double getRotation();
        
        default void rotateRight() {
            rotate(90);
        }
        
        default void rotateLeft() {
            rotate(-90);
        }
        
        default void resetRotation() {
            rotate(-getRotation());
        }
    }
    
    // Concrete class implementing multiple interfaces
    static class Circle implements Drawable, Resizable, Rotatable {
        private String color;
        private int x, y;
        private double radius;
        private double rotation;
        
        public Circle(double radius) {
            this.radius = radius;
            this.color = DEFAULT_COLOR;
            this.x = 0;
            this.y = 0;
            this.rotation = 0;
        }
        
        // Implementing Drawable interface
        @Override
        public void draw() {
            System.out.printf("Drawing %s circle at (%d, %d) with radius %.2f, rotation %.1f°%n", 
                    color, x, y, radius, rotation);
        }
        
        @Override
        public void setColor(String color) {
            this.color = color;
            System.out.printf("Circle color set to %s%n", color);
        }
        
        @Override
        public void setPosition(int x, int y) {
            this.x = x;
            this.y = y;
            System.out.printf("Circle position set to (%d, %d)%n", x, y);
        }
        
        @Override
        public double getArea() {
            return Math.PI * radius * radius;
        }
        
        // Implementing Resizable interface
        @Override
        public void resize(double factor) {
            radius *= factor;
            System.out.printf("Circle resized by factor %.2f, new radius: %.2f%n", factor, radius);
        }
        
        // Implementing Rotatable interface
        @Override
        public void rotate(double angle) {
            rotation = (rotation + angle) % 360;
            if (rotation < 0) rotation += 360;
            System.out.printf("Circle rotated by %.1f°, current rotation: %.1f°%n", angle, rotation);
        }
        
        @Override
        public double getRotation() {
            return rotation;
        }
        
        // Additional methods
        public double getRadius() { return radius; }
        public String getColor() { return color; }
        public int getX() { return x; }
        public int getY() { return y; }
    }
    
    static class Rectangle implements Drawable, Resizable, Rotatable {
        private String color;
        private int x, y;
        private double width, height;
        private double rotation;
        
        public Rectangle(double width, double height) {
            this.width = width;
            this.height = height;
            this.color = DEFAULT_COLOR;
            this.x = 0;
            this.y = 0;
            this.rotation = 0;
        }
        
        @Override
        public void draw() {
            System.out.printf("Drawing %s rectangle at (%d, %d) with dimensions %.2fx%.2f, rotation %.1f°%n", 
                    color, x, y, width, height, rotation);
        }
        
        @Override
        public void setColor(String color) {
            this.color = color;
            System.out.printf("Rectangle color set to %s%n", color);
        }
        
        @Override
        public void setPosition(int x, int y) {
            this.x = x;
            this.y = y;
            System.out.printf("Rectangle position set to (%d, %d)%n", x, y);
        }
        
        @Override
        public double getArea() {
            return width * height;
        }
        
        @Override
        public void resize(double factor) {
            width *= factor;
            height *= factor;
            System.out.printf("Rectangle resized by factor %.2f, new dimensions: %.2fx%.2f%n", 
                    factor, width, height);
        }
        
        @Override
        public void rotate(double angle) {
            rotation = (rotation + angle) % 360;
            if (rotation < 0) rotation += 360;
            System.out.printf("Rectangle rotated by %.1f°, current rotation: %.1f°%n", angle, rotation);
        }
        
        @Override
        public double getRotation() {
            return rotation;
        }
        
        public double getWidth() { return width; }
        public double getHeight() { return height; }
    }
    
    static class Triangle implements Drawable {
        private String color;
        private int x, y;
        private double base, height;
        
        public Triangle(double base, double height) {
            this.base = base;
            this.height = height;
            this.color = DEFAULT_COLOR;
            this.x = 0;
            this.y = 0;
        }
        
        @Override
        public void draw() {
            System.out.printf("Drawing %s triangle at (%d, %d) with base %.2f and height %.2f%n", 
                    color, x, y, base, height);
        }
        
        @Override
        public void setColor(String color) {
            this.color = color;
            System.out.printf("Triangle color set to %s%n", color);
        }
        
        @Override
        public void setPosition(int x, int y) {
            this.x = x;
            this.y = y;
            System.out.printf("Triangle position set to (%d, %d)%n", x, y);
        }
        
        @Override
        public double getArea() {
            return 0.5 * base * height;
        }
        
        public double getBase() { return base; }
        public double getHeight() { return height; }
    }
    
    // Demonstration methods
    public static void demonstrateBasicInterfaces() {
        System.out.println("=== Basic Interface Implementation ===");
        
        // Create different shapes
        Circle circle = new Circle(5.0);
        Rectangle rectangle = new Rectangle(10.0, 6.0);
        Triangle triangle = new Triangle(8.0, 4.0);
        
        // Store in Drawable array (interface polymorphism)
        Drawable[] shapes = {circle, rectangle, triangle};
        
        System.out.println("\n--- Testing Drawable interface ---");
        for (Drawable shape : shapes) {
            shape.setColor("RED");
            shape.setPosition(10, 20);
            shape.draw();
            System.out.printf("Area: %.2f%n", shape.getArea());
            shape.reset(); // Using default method
            System.out.println();
        }
        
        // Test multiple inheritance
        System.out.println("--- Testing Multiple Interface Implementation ---");
        
        // Circle and Rectangle implement multiple interfaces
        Resizable[] resizableShapes = {circle, rectangle};
        Rotatable[] rotatableShapes = {circle, rectangle};
        
        System.out.println("Testing Resizable interface:");
        for (Resizable shape : resizableShapes) {
            shape.doubleSize(); // Using default method
            shape.halveSize();  // Using default method
        }
        
        System.out.println("\nTesting Rotatable interface:");
        for (Rotatable shape : rotatableShapes) {
            shape.rotateRight();  // Using default method
            shape.rotateLeft();   // Using default method
        }
        
        // Test static method
        System.out.println("\n--- Testing Static Interface Method ---");
        Drawable.printDrawingInfo();
    }
    
    public static void main(String[] args) {
        demonstrateBasicInterfaces();
        
        System.out.println("\n=== Interface Benefits ===");
        System.out.println("✅ Multiple inheritance of type");
        System.out.println("✅ Contract enforcement across unrelated classes");
        System.out.println("✅ Polymorphic behavior");
        System.out.println("✅ Default method implementation sharing");
        System.out.println("✅ Constants definition");
    }
}

/*
Interface Key Points:

1. Purpose:
   - Define contracts for classes to implement
   - Enable multiple inheritance of type
   - Establish common behavior across unrelated classes
   - Support polymorphism and loose coupling

2. Method Types:
   - Abstract methods: Must be implemented by classes
   - Default methods: Provide shared implementation
   - Static methods: Utility methods belonging to interface
   - Private methods: Helper methods for default/static methods

3. Constants:
   - All variables are implicitly public static final
   - Used for defining contract-related constants
   - Accessible via interface name

4. Implementation Rules:
   - Classes must implement all abstract methods
   - Can override default methods
   - Cannot override static methods
   - Must maintain exact method signatures

5. Multiple Inheritance:
   - Classes can implement multiple interfaces
   - Diamond problem resolved through default methods
   - Explicit override required for method conflicts
*/
```

### Example 2: Advanced Interface Patterns

```java
// AdvancedInterfacePatterns.java - Complex interface usage patterns
import java.util.*;
import java.util.function.*;

public class AdvancedInterfacePatterns {
    
    // Functional interface (single abstract method)
    @FunctionalInterface
    interface DataValidator<T> {
        boolean validate(T data);
        
        // Default methods for combining validators
        default DataValidator<T> and(DataValidator<T> other) {
            return data -> this.validate(data) && other.validate(data);
        }
        
        default DataValidator<T> or(DataValidator<T> other) {
            return data -> this.validate(data) || other.validate(data);
        }
        
        default DataValidator<T> negate() {
            return data -> !this.validate(data);
        }
        
        // Static factory methods
        static <T> DataValidator<T> alwaysTrue() {
            return data -> true;
        }
        
        static <T> DataValidator<T> alwaysFalse() {
            return data -> false;
        }
    }
    
    // Interface with generic methods and bounds
    interface Repository<T, ID> {
        Optional<T> findById(ID id);
        List<T> findAll();
        T save(T entity);
        void deleteById(ID id);
        boolean existsById(ID id);
        
        // Default pagination method
        default List<T> findAll(int page, int size) {
            List<T> all = findAll();
            int start = page * size;
            int end = Math.min(start + size, all.size());
            return start < all.size() ? all.subList(start, end) : new ArrayList<>();
        }
        
        // Default method with stream operations
        default long count() {
            return findAll().size();
        }
        
        // Static utility method
        static <T, ID> Repository<T, ID> empty() {
            return new Repository<T, ID>() {
                @Override
                public Optional<T> findById(ID id) { return Optional.empty(); }
                @Override
                public List<T> findAll() { return new ArrayList<>(); }
                @Override
                public T save(T entity) { return entity; }
                @Override
                public void deleteById(ID id) {}
                @Override
                public boolean existsById(ID id) { return false; }
            };
        }
    }
    
    // Interface for event handling
    interface EventListener<T> {
        void onEvent(T event);
        
        default boolean canHandle(Class<?> eventType) {
            return true;
        }
        
        default int getPriority() {
            return 0; // Default priority
        }
    }
    
    // Interface for caching strategies
    interface CacheStrategy<K, V> {
        void put(K key, V value);
        Optional<V> get(K key);
        void remove(K key);
        void clear();
        
        default boolean containsKey(K key) {
            return get(key).isPresent();
        }
        
        default int size() {
            return 0; // Default implementation
        }
        
        // Default method using template pattern
        default V computeIfAbsent(K key, Function<K, V> valueSupplier) {
            Optional<V> existing = get(key);
            if (existing.isPresent()) {
                return existing.get();
            }
            
            V value = valueSupplier.apply(key);
            put(key, value);
            return value;
        }
    }
    
    // User entity for demonstration
    static class User {
        private final Long id;
        private final String username;
        private final String email;
        private final int age;
        
        public User(Long id, String username, String email, int age) {
            this.id = id;
            this.username = username;
            this.email = email;
            this.age = age;
        }
        
        // Getters
        public Long getId() { return id; }
        public String getUsername() { return username; }
        public String getEmail() { return email; }
        public int getAge() { return age; }
        
        @Override
        public String toString() {
            return String.format("User{id=%d, username='%s', email='%s', age=%d}", 
                    id, username, email, age);
        }
        
        @Override
        public boolean equals(Object obj) {
            if (this == obj) return true;
            if (obj == null || getClass() != obj.getClass()) return false;
            User user = (User) obj;
            return Objects.equals(id, user.id);
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(id);
        }
    }
    
    // In-memory repository implementation
    static class InMemoryUserRepository implements Repository<User, Long> {
        private final Map<Long, User> users = new HashMap<>();
        private Long nextId = 1L;
        
        @Override
        public Optional<User> findById(Long id) {
            return Optional.ofNullable(users.get(id));
        }
        
        @Override
        public List<User> findAll() {
            return new ArrayList<>(users.values());
        }
        
        @Override
        public User save(User user) {
            if (user.getId() == null) {
                // Create new user with generated ID
                User newUser = new User(nextId++, user.getUsername(), user.getEmail(), user.getAge());
                users.put(newUser.getId(), newUser);
                return newUser;
            } else {
                // Update existing user
                users.put(user.getId(), user);
                return user;
            }
        }
        
        @Override
        public void deleteById(Long id) {
            users.remove(id);
        }
        
        @Override
        public boolean existsById(Long id) {
            return users.containsKey(id);
        }
        
        @Override
        public long count() {
            return users.size();
        }
    }
    
    // LRU Cache implementation
    static class LRUCache<K, V> implements CacheStrategy<K, V> {
        private final int capacity;
        private final LinkedHashMap<K, V> cache;
        
        public LRUCache(int capacity) {
            this.capacity = capacity;
            this.cache = new LinkedHashMap<K, V>(capacity, 0.75f, true) {
                @Override
                protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                    return size() > LRUCache.this.capacity;
                }
            };
        }
        
        @Override
        public void put(K key, V value) {
            cache.put(key, value);
        }
        
        @Override
        public Optional<V> get(K key) {
            return Optional.ofNullable(cache.get(key));
        }
        
        @Override
        public void remove(K key) {
            cache.remove(key);
        }
        
        @Override
        public void clear() {
            cache.clear();
        }
        
        @Override
        public int size() {
            return cache.size();
        }
    }
    
    // Time-based cache implementation
    static class TTLCache<K, V> implements CacheStrategy<K, V> {
        private final Map<K, CacheEntry<V>> cache = new ConcurrentHashMap<>();
        private final long ttlMillis;
        
        public TTLCache(long ttlMillis) {
            this.ttlMillis = ttlMillis;
        }
        
        @Override
        public void put(K key, V value) {
            cache.put(key, new CacheEntry<>(value, System.currentTimeMillis() + ttlMillis));
        }
        
        @Override
        public Optional<V> get(K key) {
            CacheEntry<V> entry = cache.get(key);
            if (entry == null) {
                return Optional.empty();
            }
            
            if (System.currentTimeMillis() > entry.expirationTime) {
                cache.remove(key);
                return Optional.empty();
            }
            
            return Optional.of(entry.value);
        }
        
        @Override
        public void remove(K key) {
            cache.remove(key);
        }
        
        @Override
        public void clear() {
            cache.clear();
        }
        
        @Override
        public int size() {
            // Clean expired entries
            cleanExpired();
            return cache.size();
        }
        
        private void cleanExpired() {
            long now = System.currentTimeMillis();
            cache.entrySet().removeIf(entry -> now > entry.getValue().expirationTime);
        }
        
        private static class CacheEntry<V> {
            final V value;
            final long expirationTime;
            
            CacheEntry(V value, long expirationTime) {
                this.value = value;
                this.expirationTime = expirationTime;
            }
        }
    }
    
    // Event system using interfaces
    static class EventSystem<T> {
        private final List<EventListener<T>> listeners = new ArrayList<>();
        
        public void subscribe(EventListener<T> listener) {
            listeners.add(listener);
            listeners.sort((a, b) -> Integer.compare(b.getPriority(), a.getPriority()));
        }
        
        public void unsubscribe(EventListener<T> listener) {
            listeners.remove(listener);
        }
        
        public void publish(T event) {
            for (EventListener<T> listener : listeners) {
                if (listener.canHandle(event.getClass())) {
                    try {
                        listener.onEvent(event);
                    } catch (Exception e) {
                        System.err.printf("Error in event listener: %s%n", e.getMessage());
                    }
                }
            }
        }
    }
    
    // Event types
    static class UserCreatedEvent {
        private final User user;
        private final long timestamp;
        
        public UserCreatedEvent(User user) {
            this.user = user;
            this.timestamp = System.currentTimeMillis();
        }
        
        public User getUser() { return user; }
        public long getTimestamp() { return timestamp; }
    }
    
    static class UserDeletedEvent {
        private final Long userId;
        private final long timestamp;
        
        public UserDeletedEvent(Long userId) {
            this.userId = userId;
            this.timestamp = System.currentTimeMillis();
        }
        
        public Long getUserId() { return userId; }
        public long getTimestamp() { return timestamp; }
    }
    
    // Demonstration
    public static void demonstrateAdvancedPatterns() {
        System.out.println("=== Advanced Interface Patterns ===");
        
        // 1. Functional interfaces and validators
        System.out.println("\n--- Functional Interface Pattern ---");
        
        DataValidator<String> lengthValidator = s -> s != null && s.length() >= 3;
        DataValidator<String> emailValidator = s -> s != null && s.contains("@");
        DataValidator<String> combinedValidator = lengthValidator.and(emailValidator);
        
        String[] testEmails = {"john@example.com", "ab", "test@test", null};
        for (String email : testEmails) {
            System.out.printf("Email '%s' validation: %s%n", email, combinedValidator.validate(email));
        }
        
        // 2. Repository pattern with generics
        System.out.println("\n--- Repository Pattern ---");
        
        Repository<User, Long> userRepo = new InMemoryUserRepository();
        
        // Add some users
        User user1 = userRepo.save(new User(null, "john_doe", "john@example.com", 25));
        User user2 = userRepo.save(new User(null, "jane_smith", "jane@example.com", 30));
        User user3 = userRepo.save(new User(null, "bob_wilson", "bob@example.com", 35));
        
        System.out.printf("Total users: %d%n", userRepo.count());
        System.out.printf("All users: %s%n", userRepo.findAll());
        System.out.printf("User with ID 2: %s%n", userRepo.findById(2L));
        System.out.printf("Paginated (page 0, size 2): %s%n", userRepo.findAll(0, 2));
        
        // 3. Cache strategy pattern
        System.out.println("\n--- Cache Strategy Pattern ---");
        
        CacheStrategy<String, String> lruCache = new LRUCache<>(3);
        CacheStrategy<String, String> ttlCache = new TTLCache<>(2000); // 2 seconds TTL
        
        // Test LRU cache
        System.out.println("Testing LRU Cache:");
        lruCache.put("key1", "value1");
        lruCache.put("key2", "value2");
        lruCache.put("key3", "value3");
        lruCache.put("key4", "value4"); // Should evict key1
        
        System.out.printf("key1 (should be empty): %s%n", lruCache.get("key1"));
        System.out.printf("key2: %s%n", lruCache.get("key2"));
        
        // Test computeIfAbsent default method
        String computed = lruCache.computeIfAbsent("key5", k -> "computed_" + k);
        System.out.printf("Computed value: %s%n", computed);
        
        // 4. Event system
        System.out.println("\n--- Event System Pattern ---");
        
        EventSystem<Object> eventSystem = new EventSystem<>();
        
        // Subscribe listeners
        eventSystem.subscribe(new EventListener<Object>() {
            @Override
            public void onEvent(Object event) {
                if (event instanceof UserCreatedEvent) {
                    UserCreatedEvent userEvent = (UserCreatedEvent) event;
                    System.out.printf("User created: %s%n", userEvent.getUser().getUsername());
                }
            }
            
            @Override
            public boolean canHandle(Class<?> eventType) {
                return UserCreatedEvent.class.isAssignableFrom(eventType);
            }
            
            @Override
            public int getPriority() {
                return 10; // High priority
            }
        });
        
        eventSystem.subscribe(event -> {
            if (event instanceof UserDeletedEvent) {
                UserDeletedEvent deleteEvent = (UserDeletedEvent) event;
                System.out.printf("User deleted: %d%n", deleteEvent.getUserId());
            }
        });
        
        // Publish events
        eventSystem.publish(new UserCreatedEvent(user1));
        eventSystem.publish(new UserDeletedEvent(2L));
    }
    
    public static void main(String[] args) {
        demonstrateAdvancedPatterns();
        
        System.out.println("\n=== Advanced Interface Benefits ===");
        System.out.println("✅ Functional programming support");
        System.out.println("✅ Generic type safety");
        System.out.println("✅ Strategy pattern implementation");
        System.out.println("✅ Event-driven architecture");
        System.out.println("✅ Default method code sharing");
    }
}

/*
Advanced Interface Patterns:

1. Functional Interfaces:
   - Single abstract method (SAM)
   - Lambda expression support
   - Method chaining with default methods
   - Composable validation logic

2. Generic Interfaces:
   - Type safety across implementations
   - Bounded type parameters
   - Wildcard types for flexibility
   - Generic method definitions

3. Strategy Pattern:
   - Interchangeable algorithms
   - Runtime strategy selection
   - Default method shared behavior
   - Policy-based configurations

4. Event System:
   - Loose coupling between publishers and subscribers
   - Priority-based event handling
   - Type-safe event dispatching
   - Error isolation between listeners

5. Repository Pattern:
   - Data access abstraction
   - CRUD operations standardization
   - Pagination support through default methods
   - Multiple implementation strategies
*/
```

### Example 3: Interface Conflicts and Resolution

```java
// InterfaceConflicts.java - Handling diamond problem and method conflicts
import java.util.*;

public class InterfaceConflicts {
    
    // Interfaces with conflicting default methods
    interface Flyable {
        void takeOff();
        void land();
        
        default void move() {
            System.out.println("Moving through the air");
        }
        
        default String getMovementType() {
            return "AERIAL";
        }
        
        default int getSpeed() {
            return 100; // km/h
        }
    }
    
    interface Swimmable {
        void dive();
        void surface();
        
        default void move() {
            System.out.println("Moving through water");
        }
        
        default String getMovementType() {
            return "AQUATIC";
        }
        
        default int getSpeed() {
            return 50; // km/h
        }
    }
    
    interface Walkable {
        void step();
        
        default void move() {
            System.out.println("Moving on land");
        }
        
        default String getMovementType() {
            return "TERRESTRIAL";
        }
        
        default int getSpeed() {
            return 30; // km/h
        }
    }
    
    // Class implementing multiple interfaces with conflicts
    static class Duck implements Flyable, Swimmable, Walkable {
        private String currentMode = "LAND";
        
        // Must override conflicting default methods
        @Override
        public void move() {
            switch (currentMode) {
                case "AIR":
                    Flyable.super.move(); // Explicitly call specific interface method
                    break;
                case "WATER":
                    Swimmable.super.move();
                    break;
                case "LAND":
                    Walkable.super.move();
                    break;
                default:
                    System.out.println("Duck is resting");
            }
        }
        
        @Override
        public String getMovementType() {
            return "AMPHIBIOUS_" + currentMode;
        }
        
        @Override
        public int getSpeed() {
            // Combine speeds based on current mode
            switch (currentMode) {
                case "AIR":
                    return Flyable.super.getSpeed();
                case "WATER":
                    return Swimmable.super.getSpeed();
                case "LAND":
                    return Walkable.super.getSpeed();
                default:
                    return 0;
            }
        }
        
        // Implement abstract methods from interfaces
        @Override
        public void takeOff() {
            currentMode = "AIR";
            System.out.println("Duck takes off into the air");
        }
        
        @Override
        public void land() {
            currentMode = "LAND";
            System.out.println("Duck lands on the ground");
        }
        
        @Override
        public void dive() {
            currentMode = "WATER";
            System.out.println("Duck dives into water");
        }
        
        @Override
        public void surface() {
            currentMode = "LAND";
            System.out.println("Duck surfaces from water");
        }
        
        @Override
        public void step() {
            System.out.println("Duck takes a step");
        }
        
        public void setCurrentMode(String mode) {
            this.currentMode = mode;
            System.out.printf("Duck mode changed to: %s%n", mode);
        }
        
        public String getCurrentMode() {
            return currentMode;
        }
    }
    
    // Interface inheritance hierarchy
    interface Vehicle {
        void start();
        void stop();
        
        default String getType() {
            return "GENERIC_VEHICLE";
        }
        
        default void performMaintenance() {
            System.out.println("Performing basic vehicle maintenance");
        }
    }
    
    interface LandVehicle extends Vehicle {
        void accelerate();
        void brake();
        
        @Override
        default String getType() {
            return "LAND_VEHICLE";
        }
        
        default void checkTires() {
            System.out.println("Checking tire pressure and condition");
        }
        
        @Override
        default void performMaintenance() {
            Vehicle.super.performMaintenance();
            checkTires();
        }
    }
    
    interface WaterVehicle extends Vehicle {
        void sail();
        void anchor();
        
        @Override
        default String getType() {
            return "WATER_VEHICLE";
        }
        
        default void checkHull() {
            System.out.println("Inspecting hull for damage");
        }
        
        @Override
        default void performMaintenance() {
            Vehicle.super.performMaintenance();
            checkHull();
        }
    }
    
    // Amphibious vehicle implementing multiple interface hierarchies
    static class AmphibiousVehicle implements LandVehicle, WaterVehicle {
        private boolean inWater = false;
        private boolean engineRunning = false;
        
        @Override
        public void start() {
            engineRunning = true;
            System.out.println("Amphibious vehicle engine started");
        }
        
        @Override
        public void stop() {
            engineRunning = false;
            System.out.println("Amphibious vehicle engine stopped");
        }
        
        // Resolve conflict between LandVehicle and WaterVehicle
        @Override
        public String getType() {
            return inWater ? "AMPHIBIOUS_WATER_MODE" : "AMPHIBIOUS_LAND_MODE";
        }
        
        @Override
        public void performMaintenance() {
            // Call both maintenance routines
            System.out.println("Starting amphibious vehicle maintenance");
            LandVehicle.super.performMaintenance();
            WaterVehicle.super.performMaintenance();
            System.out.println("Checking water seals and transition mechanisms");
        }
        
        @Override
        public void accelerate() {
            if (!inWater) {
                System.out.println("Accelerating on land");
            } else {
                System.out.println("Cannot accelerate in water mode");
            }
        }
        
        @Override
        public void brake() {
            if (!inWater) {
                System.out.println("Braking on land");
            } else {
                System.out.println("Reducing speed in water");
            }
        }
        
        @Override
        public void sail() {
            if (inWater) {
                System.out.println("Sailing through water");
            } else {
                System.out.println("Cannot sail on land");
            }
        }
        
        @Override
        public void anchor() {
            if (inWater) {
                System.out.println("Dropping anchor");
            } else {
                System.out.println("Cannot anchor on land");
            }
        }
        
        public void enterWater() {
            inWater = true;
            System.out.println("Amphibious vehicle entering water");
        }
        
        public void exitWater() {
            inWater = false;
            System.out.println("Amphibious vehicle exiting water");
        }
        
        public boolean isInWater() {
            return inWater;
        }
    }
    
    // Multiple inheritance with functional interfaces
    @FunctionalInterface
    interface Processor<T> {
        T process(T input);
        
        default Processor<T> andThen(Processor<T> next) {
            return input -> next.process(this.process(input));
        }
        
        default Processor<T> compose(Processor<T> before) {
            return input -> this.process(before.process(input));
        }
        
        static <T> Processor<T> identity() {
            return input -> input;
        }
    }
    
    @FunctionalInterface
    interface Validator<T> {
        boolean isValid(T input);
        
        default Validator<T> and(Validator<T> other) {
            return input -> this.isValid(input) && other.isValid(input);
        }
        
        default Validator<T> or(Validator<T> other) {
            return input -> this.isValid(input) || other.isValid(input);
        }
        
        default Validator<T> negate() {
            return input -> !this.isValid(input);
        }
    }
    
    // Class implementing multiple functional interfaces
    static class DataPipeline<T> implements Processor<T>, Validator<T> {
        private final List<Processor<T>> processors = new ArrayList<>();
        private final List<Validator<T>> validators = new ArrayList<>();
        
        public DataPipeline<T> addProcessor(Processor<T> processor) {
            processors.add(processor);
            return this;
        }
        
        public DataPipeline<T> addValidator(Validator<T> validator) {
            validators.add(validator);
            return this;
        }
        
        @Override
        public T process(T input) {
            T result = input;
            for (Processor<T> processor : processors) {
                result = processor.process(result);
            }
            return result;
        }
        
        @Override
        public boolean isValid(T input) {
            return validators.stream().allMatch(validator -> validator.isValid(input));
        }
        
        public T processWithValidation(T input) {
            if (!isValid(input)) {
                throw new IllegalArgumentException("Input failed validation");
            }
            return process(input);
        }
    }
    
    // Demonstration methods
    public static void demonstrateConflictResolution() {
        System.out.println("=== Interface Conflict Resolution ===");
        
        // Duck with multiple movement capabilities
        Duck duck = new Duck();
        
        System.out.println("\n--- Duck Movement Modes ---");
        System.out.printf("Initial mode: %s, type: %s, speed: %d%n", 
                duck.getCurrentMode(), duck.getMovementType(), duck.getSpeed());
        duck.move();
        
        duck.takeOff();
        System.out.printf("After takeoff - type: %s, speed: %d%n", 
                duck.getMovementType(), duck.getSpeed());
        duck.move();
        
        duck.dive();
        System.out.printf("After diving - type: %s, speed: %d%n", 
                duck.getMovementType(), duck.getSpeed());
        duck.move();
        
        duck.land();
        System.out.printf("After landing - type: %s, speed: %d%n", 
                duck.getMovementType(), duck.getSpeed());
        duck.move();
        
        // Amphibious vehicle
        System.out.println("\n--- Amphibious Vehicle ---");
        AmphibiousVehicle vehicle = new AmphibiousVehicle();
        
        vehicle.start();
        System.out.printf("Vehicle type: %s%n", vehicle.getType());
        vehicle.accelerate();
        
        vehicle.enterWater();
        System.out.printf("Vehicle type: %s%n", vehicle.getType());
        vehicle.sail();
        vehicle.brake(); // Should behave differently in water
        
        vehicle.exitWater();
        vehicle.performMaintenance(); // Calls multiple maintenance routines
        
        // Functional interface composition
        System.out.println("\n--- Functional Interface Composition ---");
        DataPipeline<String> pipeline = new DataPipeline<>();
        
        // Add processors
        pipeline.addProcessor(String::trim)
                .addProcessor(String::toLowerCase)
                .addProcessor(s -> s.replace(" ", "_"));
        
        // Add validators
        pipeline.addValidator(s -> s != null)
                .addValidator(s -> !s.isEmpty())
                .addValidator(s -> s.length() >= 3);
        
        String input = "  Hello World  ";
        System.out.printf("Input: '%s'%n", input);
        System.out.printf("Valid: %s%n", pipeline.isValid(input));
        System.out.printf("Processed: '%s'%n", pipeline.processWithValidation(input));
        
        // Test validation failure
        try {
            pipeline.processWithValidation("AB");
        } catch (IllegalArgumentException e) {
            System.out.printf("Validation failed for 'AB': %s%n", e.getMessage());
        }
    }
    
    public static void main(String[] args) {
        demonstrateConflictResolution();
        
        System.out.println("\n=== Interface Conflict Resolution Strategies ===");
        System.out.println("🔧 Resolution Techniques:");
        System.out.println("  ✅ Explicit interface method calls (Interface.super.method())");
        System.out.println("  ✅ Override conflicting default methods");
        System.out.println("  ✅ Provide custom implementation combining behaviors");
        System.out.println("  ✅ Use composition for complex conflict scenarios");
        
        System.out.println("\n🎯 Best Practices:");
        System.out.println("  ✅ Design interfaces to minimize conflicts");
        System.out.println("  ✅ Use descriptive method names");
        System.out.println("  ✅ Document conflict resolution strategies");
        System.out.println("  ✅ Consider interface segregation principle");
    }
}

/*
Interface Conflict Resolution:

1. Diamond Problem:
   - Multiple interfaces with same default method
   - Compilation error if not resolved
   - Explicit override required in implementing class
   - Use Interface.super.method() to call specific implementation

2. Method Signature Conflicts:
   - Same method signature from multiple interfaces
   - Single implementation satisfies all interfaces
   - Return type must be compatible across interfaces
   - Exception handling must follow inheritance rules

3. Inheritance Hierarchy Conflicts:
   - Subinterface overrides parent interface default method
   - Most specific interface method wins
   - Interface methods override class methods
   - Explicit super calls for parent behavior

4. Functional Interface Composition:
   - Multiple functional interfaces can be combined
   - Use default methods for composition
   - Lambda expressions for clean implementation
   - Method references for existing behavior

5. Resolution Strategies:
   - Override with custom logic
   - Delegate to specific interface
   - Combine behaviors intelligently
   - Use composition instead of inheritance
*/
```

## 📊 Interface Evolution and Best Practices

### Interface Design Principles

```
Interface Design Guidelines:
┌─────────────────────────────────────────────────────────────────┐
│                    SOLID Principles for Interfaces             │
├─────────────────────────────────────────────────────────────────┤
│  🎯 Single Responsibility Principle (SRP):                      │
│     • Each interface should have one reason to change          │
│     • Focused, cohesive method groups                          │
│     • Avoid "fat" interfaces with many unrelated methods       │
│                                                                 │
│  🔓 Open/Closed Principle (OCP):                               │
│     • Interfaces are open for extension via default methods    │
│     • Closed for modification once published                   │
│     • Use default methods to add new functionality             │
│                                                                 │
│  🔄 Liskov Substitution Principle (LSP):                       │
│     • Implementations must be substitutable                    │
│     • Maintain behavioral contracts                           │
│     • Consistent method semantics across implementations       │
│                                                                 │
│  🧩 Interface Segregation Principle (ISP):                     │
│     • Many specific interfaces better than one general         │
│     • Clients shouldn't depend on unused methods              │
│     • Compose interfaces for complex requirements             │
│                                                                 │
│  🔌 Dependency Inversion Principle (DIP):                      │
│     • Depend on abstractions, not concretions                 │
│     • Use interfaces to define contracts                       │
│     • Enable loose coupling and testability                   │
└─────────────────────────────────────────────────────────────────┘

Interface Naming Conventions:
┌─────────────────────────────────────────────────────────────────┐
│                      Naming Best Practices                     │
├─────────────────────────────────────────────────────────────────┤
│  📝 Behavioral Interfaces (ability/capability):                │
│     • Comparable, Serializable, Cloneable                     │
│     • Readable, Writable, Drawable                            │
│     • -able, -ible suffixes for capabilities                  │
│                                                                 │
│  🏭 Service Interfaces (what they do):                         │
│     • Repository, Service, Manager                             │
│     • Processor, Handler, Controller                          │
│     • Clear action-oriented names                             │
│                                                                 │
│  📋 Contract Interfaces (data access):                         │
│     • DAO (Data Access Object)                                │
│     • API, Gateway, Provider                                  │
│     • Descriptive of their purpose                            │
│                                                                 │
│  ⚙️ Functional Interfaces:                                      │
│     • Function, Consumer, Supplier                            │
│     • Predicate, Operator, Comparator                         │
│     • Based on operation they represent                       │
└─────────────────────────────────────────────────────────────────┘
```

## ⚠️ Common Interface Mistakes

### 1. Creating interfaces that are too broad
```java
// ❌ Bad - "fat" interface
interface UserManager {
    void createUser(User user);
    void deleteUser(Long id);
    void sendEmail(String email);
    void generateReport();
    void backupData();
}

// ✅ Good - segregated interfaces
interface UserRepository {
    void createUser(User user);
    void deleteUser(Long id);
}

interface EmailService {
    void sendEmail(String email);
}

interface ReportGenerator {
    void generateReport();
}
```

### 2. Misusing default methods
```java
// ❌ Bad - default method with external dependencies
interface BadService {
    default void sendNotification() {
        EmailService.getInstance().send(); // Tight coupling
    }
}

// ✅ Good - pure default methods
interface GoodService {
    void sendNotification(EmailService emailService);
    
    default boolean isNotificationEnabled() {
        return true; // Pure logic, no external dependencies
    }
}
```

### 3. Not handling interface evolution properly
```java
// ❌ Bad - breaking existing implementations
interface PaymentProcessor {
    void processPayment(Payment payment);
    // Adding new method breaks existing implementations
    void processRefund(Refund refund); 
}

// ✅ Good - using default methods for evolution
interface PaymentProcessor {
    void processPayment(Payment payment);
    
    default void processRefund(Refund refund) {
        throw new UnsupportedOperationException("Refund not supported");
    }
}
```

## 🔥 Interview Questions & Answers

### Q1: What's the difference between abstract classes and interfaces in modern Java?
**Answer**: 
- **Abstract classes**: Can have state (instance variables), constructors, concrete methods, single inheritance
- **Interfaces**: No state (only constants), no constructors, default/static methods (Java 8+), multiple inheritance
- **Java 8+ changes**: Interfaces can have default and static methods, reducing differences
- **Use case**: Abstract classes for shared implementation and state, interfaces for contracts and multiple inheritance

### Q2: How do default methods solve the interface evolution problem?
**Answer**: 
- **Problem**: Adding methods to interfaces breaks existing implementations
- **Solution**: Default methods provide implementation in interface
- **Backward compatibility**: Existing classes continue to work without modification
- **Optional override**: Implementing classes can override default methods if needed
- **Benefits**: API evolution without breaking changes, shared behavior across implementations

### Q3: How does Java resolve conflicts when multiple interfaces have the same default method?
**Answer**: 
- **Diamond problem**: Multiple interfaces with same default method signature
- **Compilation error**: Class must explicitly override the conflicting method
- **Resolution**: Use `Interface.super.method()` to call specific interface implementation
- **Custom implementation**: Provide own logic combining or choosing between interfaces
- **Priority**: More specific interface (subinterface) wins over parent interface

### Q4: What are functional interfaces and why are they important?
**Answer**: 
- **Definition**: Interface with exactly one abstract method (SAM - Single Abstract Method)
- **Lambda support**: Can be implemented with lambda expressions or method references
- **`@FunctionalInterface`**: Optional annotation for compiler checking
- **Built-in examples**: `Function<T,R>`, `Predicate<T>`, `Consumer<T>`, `Supplier<T>`
- **Benefits**: Functional programming support, cleaner code, stream API integration

### Q5: When should you use interfaces vs abstract classes in your design?
**Answer**: 
- **Use interfaces when**: Need multiple inheritance, defining contracts, loose coupling, unrelated classes need same behavior
- **Use abstract classes when**: Need shared state/implementation, strong IS-A relationship, template method pattern, constructor initialization
- **Modern approach**: Often use both together - interfaces for contracts, abstract classes for shared implementation
- **Design pattern**: Composition over inheritance, depend on abstractions

---
[← Back to Main Guide](./README.md) | [← Previous: Abstract Classes](./abstract-classes.md) | [Next: Multiple Inheritance →](./multiple-inheritance.md)
