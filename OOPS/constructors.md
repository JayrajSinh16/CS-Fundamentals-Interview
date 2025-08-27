# 🏗️ Constructors in Java - Object Initialization

Constructors are special methods responsible for initializing objects when they're created. Understanding constructors is fundamental to object-oriented programming and frequently tested in technical interviews.

## 🎯 Key Concepts

- **Object Initialization**: Constructors set up initial state of objects
- **Constructor Types**: Default, parameterized, copy constructors
- **Constructor Chaining**: Calling one constructor from another
- **Constructor Overloading**: Multiple constructors with different parameters
- **Inheritance and Constructors**: How constructors work with inheritance

## 🌍 Real-World Analogy

**House Construction Analogy**:
- **Constructor** = Blueprint execution process when building a house
- **Default Constructor** = Standard house with basic features
- **Parameterized Constructor** = Custom house with specific requirements
- **Constructor Chaining** = Using existing foundation for different house styles
- **Copy Constructor** = Building identical house based on existing one
- **Constructor Overloading** = Different building packages (basic, premium, luxury)

## 🏗️ Constructor Types and Lifecycle

```
Object Creation Process:
┌─────────────────────────────────────────────────────────────┐
│                    new ClassName()                         │
├─────────────────────────────────────────────────────────────┤
│ 1. Memory Allocation                                        │
│    • Allocate memory in heap for object                    │
│    • Initialize instance variables to default values       │
├─────────────────────────────────────────────────────────────┤
│ 2. Constructor Execution                                    │
│    • Call appropriate constructor based on parameters      │
│    • Execute constructor chaining (this/super calls)       │
│    • Initialize instance variables                         │
│    • Execute constructor body                              │
├─────────────────────────────────────────────────────────────┤
│ 3. Object Reference Return                                  │
│    • Return reference to newly created object              │
│    • Object is now ready for use                          │
└─────────────────────────────────────────────────────────────┘

Constructor Resolution Order:
1. super() or this() call (if present)
2. Instance variable initializers
3. Constructor body execution
```

## 💻 Practical Examples

### Example 1: Basic Constructor Types

```java
// ConstructorBasicsDemo.java - Understanding different constructor types
public class ConstructorBasicsDemo {
    
    // Person class demonstrating different constructor types
    static class Person {
        private String name;
        private int age;
        private String email;
        private String address;
        
        // Default constructor (no parameters)
        public Person() {
            this.name = "Unknown";
            this.age = 0;
            this.email = "not-provided@example.com";
            this.address = "Not specified";
            System.out.println("Default constructor called");
        }
        
        // Parameterized constructor (name only)
        public Person(String name) {
            this(); // Call default constructor first
            this.name = name;
            System.out.println("Single parameter constructor called");
        }
        
        // Parameterized constructor (name and age)
        public Person(String name, int age) {
            this(name); // Call single parameter constructor
            this.age = age;
            System.out.println("Two parameter constructor called");
        }
        
        // Parameterized constructor (all fields)
        public Person(String name, int age, String email, String address) {
            this.name = name;
            this.age = age;
            this.email = email;
            this.address = address;
            System.out.println("Full parameter constructor called");
        }
        
        // Copy constructor (not built-in, manually implemented)
        public Person(Person other) {
            if (other != null) {
                this.name = other.name;
                this.age = other.age;
                this.email = other.email;
                this.address = other.address;
                System.out.println("Copy constructor called");
            } else {
                throw new IllegalArgumentException("Cannot copy from null object");
            }
        }
        
        // Getters for demonstration
        public String getName() { return name; }
        public int getAge() { return age; }
        public String getEmail() { return email; }
        public String getAddress() { return address; }
        
        @Override
        public String toString() {
            return String.format("Person{name='%s', age=%d, email='%s', address='%s'}", 
                    name, age, email, address);
        }
        
        @Override
        public boolean equals(Object obj) {
            if (this == obj) return true;
            if (obj == null || getClass() != obj.getClass()) return false;
            
            Person person = (Person) obj;
            return age == person.age &&
                   Objects.equals(name, person.name) &&
                   Objects.equals(email, person.email) &&
                   Objects.equals(address, person.address);
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(name, age, email, address);
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Constructor Types Demonstration ===\n");
        
        // Default constructor
        System.out.println("--- Default Constructor ---");
        Person person1 = new Person();
        System.out.println("Result: " + person1);
        System.out.println();
        
        // Single parameter constructor
        System.out.println("--- Single Parameter Constructor ---");
        Person person2 = new Person("Alice");
        System.out.println("Result: " + person2);
        System.out.println();
        
        // Two parameter constructor
        System.out.println("--- Two Parameter Constructor ---");
        Person person3 = new Person("Bob", 30);
        System.out.println("Result: " + person3);
        System.out.println();
        
        // Full parameter constructor
        System.out.println("--- Full Parameter Constructor ---");
        Person person4 = new Person("Charlie", 25, "charlie@email.com", "123 Main St");
        System.out.println("Result: " + person4);
        System.out.println();
        
        // Copy constructor
        System.out.println("--- Copy Constructor ---");
        Person person5 = new Person(person4);
        System.out.println("Original: " + person4);
        System.out.println("Copy: " + person5);
        System.out.println("Are equal: " + person4.equals(person5));
        System.out.println("Same reference: " + (person4 == person5));
        System.out.println();
    }
}

/*
Constructor Characteristics:
1. Same name as class
2. No return type (not even void)
3. Called automatically when object is created
4. Can be overloaded
5. Cannot be inherited
6. Cannot be abstract, static, or final
*/
```

### Example 2: Constructor Chaining and this/super Keywords

```java
// ConstructorChainingDemo.java - Demonstrating constructor chaining
public class ConstructorChainingDemo {
    
    // Base class demonstrating constructor chaining
    static class Vehicle {
        protected String brand;
        protected String model;
        protected int year;
        protected double price;
        
        // Default constructor
        public Vehicle() {
            this("Unknown Brand", "Unknown Model", 2000, 0.0);
            System.out.println("Vehicle default constructor");
        }
        
        // Constructor with brand only
        public Vehicle(String brand) {
            this(brand, "Unknown Model", 2000, 0.0);
            System.out.println("Vehicle single parameter constructor");
        }
        
        // Constructor with brand and model
        public Vehicle(String brand, String model) {
            this(brand, model, 2000, 0.0);
            System.out.println("Vehicle two parameter constructor");
        }
        
        // Master constructor (all parameters)
        public Vehicle(String brand, String model, int year, double price) {
            this.brand = brand;
            this.model = model;
            this.year = year;
            this.price = price;
            System.out.println("Vehicle master constructor");
            validateData();
        }
        
        private void validateData() {
            if (year < 1900 || year > 2030) {
                throw new IllegalArgumentException("Invalid year: " + year);
            }
            if (price < 0) {
                throw new IllegalArgumentException("Price cannot be negative: " + price);
            }
        }
        
        @Override
        public String toString() {
            return String.format("Vehicle{brand='%s', model='%s', year=%d, price=%.2f}", 
                    brand, model, year, price);
        }
    }
    
    // Derived class demonstrating super() calls
    static class Car extends Vehicle {
        private int doors;
        private String fuelType;
        private boolean isAutomatic;
        
        // Default constructor
        public Car() {
            super(); // Call parent default constructor
            this.doors = 4;
            this.fuelType = "Gasoline";
            this.isAutomatic = true;
            System.out.println("Car default constructor");
        }
        
        // Constructor with basic info
        public Car(String brand, String model) {
            super(brand, model); // Call parent constructor
            this.doors = 4;
            this.fuelType = "Gasoline";
            this.isAutomatic = true;
            System.out.println("Car basic constructor");
        }
        
        // Constructor with vehicle info
        public Car(String brand, String model, int year, double price) {
            super(brand, model, year, price); // Call parent constructor
            this.doors = 4;
            this.fuelType = "Gasoline";
            this.isAutomatic = true;
            System.out.println("Car vehicle info constructor");
        }
        
        // Full constructor
        public Car(String brand, String model, int year, double price, 
                   int doors, String fuelType, boolean isAutomatic) {
            super(brand, model, year, price); // Must be first statement
            this.doors = doors;
            this.fuelType = fuelType;
            this.isAutomatic = isAutomatic;
            System.out.println("Car full constructor");
            validateCarData();
        }
        
        private void validateCarData() {
            if (doors < 2 || doors > 6) {
                throw new IllegalArgumentException("Invalid number of doors: " + doors);
            }
        }
        
        // Getters
        public int getDoors() { return doors; }
        public String getFuelType() { return fuelType; }
        public boolean isAutomatic() { return isAutomatic; }
        
        @Override
        public String toString() {
            return String.format("Car{%s, doors=%d, fuel='%s', automatic=%b}", 
                    super.toString().replace("Vehicle{", "").replace("}", ""),
                    doors, fuelType, isAutomatic);
        }
    }
    
    // Advanced example with builder pattern integration
    static class SportsCar extends Car {
        private int horsepower;
        private double topSpeed;
        private boolean hasTurbo;
        
        // Private constructor for builder pattern
        private SportsCar(Builder builder) {
            super(builder.brand, builder.model, builder.year, builder.price,
                  builder.doors, builder.fuelType, builder.isAutomatic);
            this.horsepower = builder.horsepower;
            this.topSpeed = builder.topSpeed;
            this.hasTurbo = builder.hasTurbo;
            System.out.println("SportsCar builder constructor");
        }
        
        // Traditional constructor
        public SportsCar(String brand, String model, int year, double price,
                        int horsepower, double topSpeed, boolean hasTurbo) {
            super(brand, model, year, price, 2, "Premium", true);
            this.horsepower = horsepower;
            this.topSpeed = topSpeed;
            this.hasTurbo = hasTurbo;
            System.out.println("SportsCar traditional constructor");
        }
        
        // Builder pattern
        public static class Builder {
            // Required parameters
            private String brand;
            private String model;
            
            // Optional parameters with defaults
            private int year = 2023;
            private double price = 50000.0;
            private int doors = 2;
            private String fuelType = "Premium";
            private boolean isAutomatic = true;
            private int horsepower = 300;
            private double topSpeed = 150.0;
            private boolean hasTurbo = false;
            
            public Builder(String brand, String model) {
                this.brand = brand;
                this.model = model;
            }
            
            public Builder year(int year) { this.year = year; return this; }
            public Builder price(double price) { this.price = price; return this; }
            public Builder doors(int doors) { this.doors = doors; return this; }
            public Builder fuelType(String fuelType) { this.fuelType = fuelType; return this; }
            public Builder automatic(boolean isAutomatic) { this.isAutomatic = isAutomatic; return this; }
            public Builder horsepower(int horsepower) { this.horsepower = horsepower; return this; }
            public Builder topSpeed(double topSpeed) { this.topSpeed = topSpeed; return this; }
            public Builder turbo(boolean hasTurbo) { this.hasTurbo = hasTurbo; return this; }
            
            public SportsCar build() {
                return new SportsCar(this);
            }
        }
        
        @Override
        public String toString() {
            return String.format("SportsCar{%s, hp=%d, topSpeed=%.1f, turbo=%b}", 
                    super.toString().replace("Car{", "").replace("}", ""),
                    horsepower, topSpeed, hasTurbo);
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Constructor Chaining Demonstration ===\n");
        
        // Vehicle constructor chaining
        System.out.println("--- Vehicle Constructor Chaining ---");
        Vehicle vehicle1 = new Vehicle();
        System.out.println("Result: " + vehicle1);
        System.out.println();
        
        System.out.println("--- Vehicle with Brand ---");
        Vehicle vehicle2 = new Vehicle("Toyota");
        System.out.println("Result: " + vehicle2);
        System.out.println();
        
        // Car inheritance and super() calls
        System.out.println("--- Car Inheritance Chain ---");
        Car car1 = new Car();
        System.out.println("Result: " + car1);
        System.out.println();
        
        System.out.println("--- Car with Basic Info ---");
        Car car2 = new Car("Honda", "Civic");
        System.out.println("Result: " + car2);
        System.out.println();
        
        System.out.println("--- Car with Full Info ---");
        Car car3 = new Car("BMW", "M3", 2023, 65000.0, 4, "Premium", true);
        System.out.println("Result: " + car3);
        System.out.println();
        
        // SportsCar with builder pattern
        System.out.println("--- SportsCar with Builder Pattern ---");
        SportsCar sportsCar1 = new SportsCar.Builder("Ferrari", "F8")
                .year(2023)
                .price(280000.0)
                .horsepower(710)
                .topSpeed(211.0)
                .turbo(true)
                .build();
        System.out.println("Result: " + sportsCar1);
        System.out.println();
        
        System.out.println("--- SportsCar Traditional Constructor ---");
        SportsCar sportsCar2 = new SportsCar("Lamborghini", "Huracan", 
                2023, 250000.0, 630, 202.0, false);
        System.out.println("Result: " + sportsCar2);
    }
}

/*
Constructor Chaining Rules:
1. this() or super() must be first statement
2. Cannot call both this() and super() in same constructor
3. Constructor chaining prevents code duplication
4. super() is implicit if not specified
5. Builder pattern can simplify complex construction
*/
```

### Example 3: Constructor Best Practices and Common Issues

```java
// ConstructorBestPracticesDemo.java - Best practices and common pitfalls
import java.util.*;

public class ConstructorBestPracticesDemo {
    
    // Good example: Defensive copying and validation
    static class BankAccount {
        private final String accountNumber;
        private final String ownerName;
        private volatile double balance;
        private final List<String> transactionHistory;
        private final Date createdDate;
        
        // Private constructor for internal use
        private BankAccount() {
            this.accountNumber = generateAccountNumber();
            this.ownerName = "Unknown";
            this.balance = 0.0;
            this.transactionHistory = new ArrayList<>();
            this.createdDate = new Date();
        }
        
        // Public constructor with validation
        public BankAccount(String ownerName, double initialBalance) {
            // Validation first
            validateOwnerName(ownerName);
            validateBalance(initialBalance);
            
            // Initialize fields
            this.accountNumber = generateAccountNumber();
            this.ownerName = ownerName.trim();
            this.balance = initialBalance;
            this.transactionHistory = new ArrayList<>();
            this.createdDate = new Date();
            
            // Log initial transaction
            if (initialBalance > 0) {
                transactionHistory.add(String.format("Initial deposit: $%.2f", initialBalance));
            }
        }
        
        // Constructor with transaction history (defensive copying)
        public BankAccount(String ownerName, double initialBalance, List<String> existingHistory) {
            this(ownerName, initialBalance); // Chain to main constructor
            
            // Defensive copying - don't expose internal structure
            if (existingHistory != null) {
                this.transactionHistory.addAll(existingHistory);
            }
        }
        
        // Copy constructor with modifications
        public BankAccount(BankAccount other, String newOwnerName) {
            if (other == null) {
                throw new IllegalArgumentException("Cannot copy from null account");
            }
            
            validateOwnerName(newOwnerName);
            
            this.accountNumber = generateAccountNumber(); // New account number
            this.ownerName = newOwnerName;
            this.balance = 0.0; // New account starts with zero balance
            this.transactionHistory = new ArrayList<>();
            this.createdDate = new Date();
            
            // Copy some metadata but not balance for security
            transactionHistory.add("Account created as copy of " + other.accountNumber);
        }
        
        // Validation methods
        private void validateOwnerName(String name) {
            if (name == null || name.trim().isEmpty()) {
                throw new IllegalArgumentException("Owner name cannot be null or empty");
            }
            if (name.trim().length() < 2) {
                throw new IllegalArgumentException("Owner name must be at least 2 characters");
            }
        }
        
        private void validateBalance(double balance) {
            if (balance < 0) {
                throw new IllegalArgumentException("Initial balance cannot be negative");
            }
            if (balance > 1_000_000) {
                throw new IllegalArgumentException("Initial balance cannot exceed $1,000,000");
            }
        }
        
        private String generateAccountNumber() {
            return "ACC" + System.currentTimeMillis() + (int)(Math.random() * 1000);
        }
        
        // Getters with defensive copying where needed
        public String getAccountNumber() { return accountNumber; }
        public String getOwnerName() { return ownerName; }
        public double getBalance() { return balance; }
        
        public List<String> getTransactionHistory() {
            return new ArrayList<>(transactionHistory); // Defensive copy
        }
        
        public Date getCreatedDate() {
            return new Date(createdDate.getTime()); // Defensive copy
        }
        
        @Override
        public String toString() {
            return String.format("BankAccount{account='%s', owner='%s', balance=$%.2f, created=%s}", 
                    accountNumber, ownerName, balance, createdDate);
        }
    }
    
    // Bad example: Common constructor mistakes
    static class BadExample {
        private String name;
        private List<String> items;
        private Date timestamp;
        
        // ❌ No validation
        public BadExample(String name, List<String> items) {
            this.name = name; // Could be null
            this.items = items; // Direct assignment - no defensive copying
            this.timestamp = new Date(); // Mutable object
        }
        
        // ❌ Exposing mutable internals
        public List<String> getItems() {
            return items; // Caller can modify internal state
        }
        
        public Date getTimestamp() {
            return timestamp; // Caller can modify internal state
        }
    }
    
    // Good example: Immutable class with builder
    static final class ImmutablePerson {
        private final String firstName;
        private final String lastName;
        private final int age;
        private final List<String> hobbies;
        private final Map<String, String> properties;
        
        // Private constructor - only accessible via builder
        private ImmutablePerson(Builder builder) {
            this.firstName = builder.firstName;
            this.lastName = builder.lastName;
            this.age = builder.age;
            this.hobbies = Collections.unmodifiableList(new ArrayList<>(builder.hobbies));
            this.properties = Collections.unmodifiableMap(new HashMap<>(builder.properties));
        }
        
        // Builder pattern for complex construction
        public static class Builder {
            private String firstName;
            private String lastName;
            private int age;
            private List<String> hobbies = new ArrayList<>();
            private Map<String, String> properties = new HashMap<>();
            
            public Builder firstName(String firstName) {
                this.firstName = Objects.requireNonNull(firstName, "First name cannot be null");
                return this;
            }
            
            public Builder lastName(String lastName) {
                this.lastName = Objects.requireNonNull(lastName, "Last name cannot be null");
                return this;
            }
            
            public Builder age(int age) {
                if (age < 0 || age > 150) {
                    throw new IllegalArgumentException("Age must be between 0 and 150");
                }
                this.age = age;
                return this;
            }
            
            public Builder addHobby(String hobby) {
                if (hobby != null && !hobby.trim().isEmpty()) {
                    this.hobbies.add(hobby.trim());
                }
                return this;
            }
            
            public Builder addProperty(String key, String value) {
                if (key != null && value != null) {
                    this.properties.put(key, value);
                }
                return this;
            }
            
            public ImmutablePerson build() {
                // Final validation
                if (firstName == null || lastName == null) {
                    throw new IllegalStateException("First name and last name are required");
                }
                return new ImmutablePerson(this);
            }
        }
        
        // Getters (no setters for immutable class)
        public String getFirstName() { return firstName; }
        public String getLastName() { return lastName; }
        public int getAge() { return age; }
        public List<String> getHobbies() { return hobbies; } // Already unmodifiable
        public Map<String, String> getProperties() { return properties; } // Already unmodifiable
        
        @Override
        public String toString() {
            return String.format("ImmutablePerson{name='%s %s', age=%d, hobbies=%s, properties=%s}", 
                    firstName, lastName, age, hobbies, properties);
        }
        
        @Override
        public boolean equals(Object obj) {
            if (this == obj) return true;
            if (obj == null || getClass() != obj.getClass()) return false;
            
            ImmutablePerson that = (ImmutablePerson) obj;
            return age == that.age &&
                   Objects.equals(firstName, that.firstName) &&
                   Objects.equals(lastName, that.lastName) &&
                   Objects.equals(hobbies, that.hobbies) &&
                   Objects.equals(properties, that.properties);
        }
        
        @Override
        public int hashCode() {
            return Objects.hash(firstName, lastName, age, hobbies, properties);
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Constructor Best Practices Demonstration ===\n");
        
        demonstrateGoodConstructors();
        demonstrateBadConstructors();
        demonstrateImmutableConstruction();
        demonstrateConstructorPerformance();
    }
    
    private static void demonstrateGoodConstructors() {
        System.out.println("--- Good Constructor Practices ---");
        
        try {
            // Valid construction
            BankAccount account1 = new BankAccount("John Doe", 1000.0);
            System.out.println("✅ " + account1);
            
            // Constructor with defensive copying
            List<String> history = Arrays.asList("Previous transaction 1", "Previous transaction 2");
            BankAccount account2 = new BankAccount("Jane Smith", 500.0, history);
            System.out.println("✅ " + account2);
            System.out.println("   Transaction history: " + account2.getTransactionHistory());
            
            // Copy constructor
            BankAccount account3 = new BankAccount(account1, "Bob Johnson");
            System.out.println("✅ " + account3);
            
        } catch (Exception e) {
            System.out.println("❌ " + e.getMessage());
        }
        
        System.out.println();
    }
    
    private static void demonstrateBadConstructors() {
        System.out.println("--- Constructor Validation ---");
        
        // Test various invalid inputs
        String[] invalidNames = {null, "", "  ", "A"};
        double[] invalidBalances = {-100.0, -0.01, 1_000_001.0};
        
        for (String name : invalidNames) {
            try {
                new BankAccount(name, 100.0);
                System.out.println("❌ Should have failed for name: " + name);
            } catch (IllegalArgumentException e) {
                System.out.println("✅ Correctly rejected name '" + name + "': " + e.getMessage());
            }
        }
        
        for (double balance : invalidBalances) {
            try {
                new BankAccount("Valid Name", balance);
                System.out.println("❌ Should have failed for balance: " + balance);
            } catch (IllegalArgumentException e) {
                System.out.println("✅ Correctly rejected balance " + balance + ": " + e.getMessage());
            }
        }
        
        System.out.println();
    }
    
    private static void demonstrateImmutableConstruction() {
        System.out.println("--- Immutable Object Construction ---");
        
        ImmutablePerson person = new ImmutablePerson.Builder()
                .firstName("Alice")
                .lastName("Johnson")
                .age(28)
                .addHobby("Reading")
                .addHobby("Swimming")
                .addProperty("occupation", "Software Engineer")
                .addProperty("city", "San Francisco")
                .build();
        
        System.out.println("✅ Immutable person created: " + person);
        
        // Demonstrate immutability
        List<String> hobbies = person.getHobbies();
        try {
            hobbies.add("New Hobby"); // This should fail
            System.out.println("❌ Should not be able to modify hobbies");
        } catch (UnsupportedOperationException e) {
            System.out.println("✅ Hobbies list is truly immutable");
        }
        
        System.out.println();
    }
    
    private static void demonstrateConstructorPerformance() {
        System.out.println("--- Constructor Performance Considerations ---");
        
        int iterations = 10000;
        
        // Time simple constructor
        long start = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            new BankAccount("User" + i, 100.0);
        }
        long simpleTime = System.nanoTime() - start;
        
        // Time complex constructor with validation and copying
        List<String> largeHistory = new ArrayList<>();
        for (int i = 0; i < 100; i++) {
            largeHistory.add("Transaction " + i);
        }
        
        start = System.nanoTime();
        for (int i = 0; i < iterations; i++) {
            new BankAccount("User" + i, 100.0, largeHistory);
        }
        long complexTime = System.nanoTime() - start;
        
        System.out.printf("Simple constructor: %.2f ms (%d iterations)\n", 
                simpleTime / 1_000_000.0, iterations);
        System.out.printf("Complex constructor: %.2f ms (%d iterations)\n", 
                complexTime / 1_000_000.0, iterations);
        System.out.printf("Performance impact: %.1fx slower\n", 
                (double) complexTime / simpleTime);
        
        System.out.println("\n=== Constructor Best Practices Summary ===");
        System.out.println("✅ Validate all parameters");
        System.out.println("✅ Use defensive copying for mutable objects");
        System.out.println("✅ Initialize all fields");
        System.out.println("✅ Use constructor chaining to avoid duplication");
        System.out.println("✅ Consider builder pattern for complex objects");
        System.out.println("✅ Make immutable objects when possible");
        System.out.println("❌ Don't expose mutable internals");
        System.out.println("❌ Don't allow partially constructed objects");
        System.out.println("❌ Don't ignore validation for performance");
    }
}

/*
Constructor Best Practices:
1. Validate all parameters early
2. Use defensive copying for mutable parameters
3. Initialize all fields to valid states
4. Use constructor chaining to reduce duplication
5. Consider builder pattern for complex construction
6. Make objects immutable when possible
7. Handle exceptions appropriately
8. Document constructor contracts
*/
```

## 📊 Constructor Types Comparison

| Constructor Type | Parameters | Use Case | Example |
|------------------|------------|----------|---------|
| **Default** | None | Basic initialization | `Person()` |
| **Parameterized** | One or more | Custom initialization | `Person(String name)` |
| **Copy** | Same class object | Object duplication | `Person(Person other)` |
| **Private** | Any | Builder pattern, Singleton | `new Builder().build()` |

## 🔄 Constructor Execution Order

### Inheritance Chain
```java
class Parent {
    Parent() { System.out.println("Parent constructor"); }
}

class Child extends Parent {
    Child() { 
        super(); // Implicit call
        System.out.println("Child constructor"); 
    }
}

// Output:
// Parent constructor
// Child constructor
```

### With Instance Initializers
```java
class Example {
    private int value = initValue(); // 1. Instance initializer
    
    { System.out.println("Instance block"); } // 2. Instance block
    
    public Example() { 
        System.out.println("Constructor"); // 3. Constructor body
    }
    
    private int initValue() {
        System.out.println("Field initializer");
        return 42;
    }
}
```

## ⚠️ Common Constructor Mistakes

### 1. No parameter validation
```java
// ❌ No validation
public Person(String name, int age) {
    this.name = name; // Could be null
    this.age = age;   // Could be negative
}

// ✅ Proper validation
public Person(String name, int age) {
    this.name = Objects.requireNonNull(name, "Name cannot be null");
    if (age < 0 || age > 150) {
        throw new IllegalArgumentException("Invalid age: " + age);
    }
    this.age = age;
}
```

### 2. Exposing mutable internals
```java
// ❌ Direct assignment of mutable objects
public Person(List<String> hobbies) {
    this.hobbies = hobbies; // Caller can modify internal state
}

// ✅ Defensive copying
public Person(List<String> hobbies) {
    this.hobbies = hobbies != null ? new ArrayList<>(hobbies) : new ArrayList<>();
}
```

### 3. Constructor overloading confusion
```java
// ❌ Ambiguous constructors
public Person(String name) { /* ... */ }
public Person(String email) { /* ... */ } // Compiler error!

// ✅ Clear differentiation
public Person(String name) { /* ... */ }
public static Person withEmail(String email) { /* ... */ } // Factory method
```

## 🔥 Interview Questions & Answers

### Q1: What is the difference between a constructor and a method?
**Answer**: 
- **Constructor**: Special method for object initialization, same name as class, no return type, called automatically with `new`
- **Method**: Regular function, any name, has return type, called explicitly on objects
- **Purpose**: Constructor sets initial state, methods perform operations
- **Inheritance**: Constructors not inherited, methods are inherited

### Q2: Can constructors be overloaded? How does constructor chaining work?
**Answer**: Yes, constructors can be overloaded with different parameters:
- **Overloading**: Multiple constructors with different parameter lists
- **this()**: Calls another constructor in same class (must be first statement)
- **super()**: Calls parent class constructor (must be first statement)
- **Chaining**: Link constructors to avoid code duplication and ensure proper initialization

### Q3: What happens if you don't provide a constructor in a class?
**Answer**: Java provides a default no-argument constructor automatically:
- **Implicit**: Added by compiler if no constructors exist
- **Behavior**: Calls super() and initializes fields to default values
- **Lost**: Disappears if you define any constructor explicitly
- **Access**: Same visibility as the class (public class gets public constructor)

### Q4: Can constructors be private? What are the use cases?
**Answer**: Yes, private constructors are used for:
- **Singleton Pattern**: Prevent external instantiation
- **Builder Pattern**: Force use of builder for construction
- **Utility Classes**: Prevent instantiation of static-only classes
- **Factory Methods**: Control object creation through static methods

### Q5: How do you handle exceptions in constructors?
**Answer**: 
- **Validation**: Check parameters and throw IllegalArgumentException for invalid input
- **Resource Cleanup**: Use try-finally or try-with-resources if needed
- **Partial Construction**: Avoid leaving objects in invalid state
- **Checked Exceptions**: Can throw checked exceptions from constructors
- **Best Practice**: Fail fast with clear error messages

---
[← Back to Main Guide](./README.md) | [← Previous: String Handling](./string-handling.md) | [Next: Static Keyword →](./static-keyword.md)
