# Java 8 Features - Modern Java Programming

## Navigation
- [← Previous: Multithreading Basics](multithreading-basics.md)
- [🏠 Main Menu](README.md)
- [→ Next: Advanced Java Topics](advanced-java-topics.md)

---

## Table of Contents
1. [Lambda Expressions](#lambda-expressions)
2. [Functional Interfaces](#functional-interfaces)
3. [Stream API](#stream-api)
4. [Method References](#method-references)
5. [Optional Class](#optional-class)
6. [Default Methods in Interfaces](#default-methods-in-interfaces)
7. [Interview Questions](#interview-questions)

---

## Lambda Expressions

### What are Lambda Expressions?
Lambda expressions are anonymous functions that can be used to create delegates or expression tree types. They provide a clear and concise way to represent one method interface using an expression.

### Real-world Analogy 🏪
Think of lambda expressions like **self-service kiosks**:
- **Traditional method**: Go to a cashier, explain what you want, wait for them to process
- **Lambda approach**: Use a touch screen, directly input your requirements, get instant results
- **Benefit**: Faster, more direct, less overhead

### Basic Syntax
```java
// Traditional anonymous class
Runnable r1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello World");
    }
};

// Lambda expression
Runnable r2 = () -> System.out.println("Hello World");
```

### Practical Examples

#### 1. Basic Lambda Expressions
```java
import java.util.*;
import java.util.function.*;

public class LambdaBasics {
    public static void main(String[] args) {
        // No parameters
        Runnable task = () -> System.out.println("Task executed");
        
        // One parameter
        Consumer<String> printer = message -> System.out.println(message);
        
        // Multiple parameters
        BinaryOperator<Integer> adder = (a, b) -> a + b;
        
        // Multi-line lambda
        Function<String, String> processor = (input) -> {
            String processed = input.trim().toUpperCase();
            return "Processed: " + processed;
        };
        
        // Usage
        task.run();
        printer.accept("Hello Lambda!");
        System.out.println("Sum: " + adder.apply(5, 3));
        System.out.println(processor.apply("  hello world  "));
    }
}
```

#### 2. Lambda with Collections
```java
import java.util.*;
import java.util.stream.*;

public class LambdaCollections {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");
        
        // Traditional iteration
        System.out.println("Traditional way:");
        for (String name : names) {
            System.out.println(name);
        }
        
        // Lambda forEach
        System.out.println("\nLambda way:");
        names.forEach(name -> System.out.println(name));
        
        // Method reference (even cleaner)
        System.out.println("\nMethod reference:");
        names.forEach(System.out::println);
        
        // Filtering and processing
        System.out.println("\nFiltered names (length > 3):");
        names.stream()
             .filter(name -> name.length() > 3)
             .forEach(System.out::println);
    }
}
```

---

## Functional Interfaces

### What are Functional Interfaces?
A functional interface is an interface that contains exactly one abstract method. They can have multiple default or static methods, but only one abstract method.

### Built-in Functional Interfaces

#### 1. Predicate<T> - Test a condition
```java
import java.util.function.Predicate;
import java.util.*;

public class PredicateExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // Test for even numbers
        Predicate<Integer> isEven = n -> n % 2 == 0;
        
        // Test for numbers greater than 5
        Predicate<Integer> greaterThanFive = n -> n > 5;
        
        // Combine predicates
        Predicate<Integer> evenAndGreaterThanFive = isEven.and(greaterThanFive);
        
        System.out.println("Even numbers:");
        numbers.stream()
               .filter(isEven)
               .forEach(System.out::println);
        
        System.out.println("\nEven numbers greater than 5:");
        numbers.stream()
               .filter(evenAndGreaterThanFive)
               .forEach(System.out::println);
    }
}
```

#### 2. Function<T, R> - Transform input to output
```java
import java.util.function.Function;
import java.util.*;

public class FunctionExample {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("apple", "banana", "cherry");
        
        // Function to get string length
        Function<String, Integer> getLength = str -> str.length();
        
        // Function to convert to uppercase
        Function<String, String> toUpperCase = str -> str.toUpperCase();
        
        // Compose functions
        Function<String, String> lengthAndUpper = 
            toUpperCase.compose(String::valueOf).compose(getLength);
        
        System.out.println("Word lengths:");
        words.stream()
             .map(getLength)
             .forEach(System.out::println);
        
        System.out.println("\nUppercase words:");
        words.stream()
             .map(toUpperCase)
             .forEach(System.out::println);
    }
}
```

#### 3. Consumer<T> - Accept input, return nothing
```java
import java.util.function.Consumer;
import java.util.*;

public class ConsumerExample {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
        
        // Simple consumer
        Consumer<String> printer = System.out::println;
        
        // Consumer with logging
        Consumer<String> logger = name -> 
            System.out.println("Processing: " + name);
        
        // Chain consumers
        Consumer<String> combined = logger.andThen(printer);
        
        System.out.println("Using combined consumer:");
        names.forEach(combined);
    }
}
```

#### 4. Supplier<T> - Provide a value
```java
import java.util.function.Supplier;
import java.time.LocalDateTime;
import java.util.Random;

public class SupplierExample {
    public static void main(String[] args) {
        // Supplier for current time
        Supplier<LocalDateTime> timeSupplier = LocalDateTime::now;
        
        // Supplier for random numbers
        Supplier<Integer> randomSupplier = () -> new Random().nextInt(100);
        
        // Supplier for messages
        Supplier<String> messageSupplier = () -> "Hello from supplier!";
        
        System.out.println("Current time: " + timeSupplier.get());
        System.out.println("Random number: " + randomSupplier.get());
        System.out.println("Message: " + messageSupplier.get());
        
        // Using supplier for lazy evaluation
        System.out.println("\nGenerating 5 random numbers:");
        for (int i = 0; i < 5; i++) {
            System.out.println(randomSupplier.get());
        }
    }
}
```

---

## Stream API

### What is Stream API?
Stream API allows functional-style operations on collections of objects. A stream is a sequence of objects that supports various methods which can be pipelined to produce the desired result.

### Real-world Analogy 🏭
Think of Stream API like an **assembly line in a factory**:
- **Raw materials**: Your original collection
- **Stations**: Each operation (filter, map, sort)
- **Final product**: The result after all transformations
- **Efficiency**: Operations are optimized and can run in parallel

### Basic Stream Operations

#### 1. Creating Streams
```java
import java.util.*;
import java.util.stream.*;

public class StreamCreation {
    public static void main(String[] args) {
        // From collections
        List<String> list = Arrays.asList("a", "b", "c");
        Stream<String> streamFromList = list.stream();
        
        // From arrays
        String[] array = {"x", "y", "z"};
        Stream<String> streamFromArray = Arrays.stream(array);
        
        // From values
        Stream<String> streamFromValues = Stream.of("p", "q", "r");
        
        // Generate streams
        Stream<Integer> infiniteStream = Stream.iterate(0, n -> n + 2);
        Stream<Double> randomStream = Stream.generate(Math::random);
        
        // Range streams
        IntStream rangeStream = IntStream.range(1, 10);
        IntStream rangeClosedStream = IntStream.rangeClosed(1, 10);
        
        // Print first 5 even numbers
        infiniteStream.limit(5).forEach(System.out::println);
    }
}
```

#### 2. Intermediate Operations (Lazy)
```java
import java.util.*;
import java.util.stream.*;

public class IntermediateOperations {
    static class Person {
        private String name;
        private int age;
        private String city;
        
        public Person(String name, int age, String city) {
            this.name = name;
            this.age = age;
            this.city = city;
        }
        
        // Getters
        public String getName() { return name; }
        public int getAge() { return age; }
        public String getCity() { return city; }
        
        @Override
        public String toString() {
            return name + "(" + age + ", " + city + ")";
        }
    }
    
    public static void main(String[] args) {
        List<Person> people = Arrays.asList(
            new Person("Alice", 30, "New York"),
            new Person("Bob", 25, "London"),
            new Person("Charlie", 35, "New York"),
            new Person("David", 28, "London"),
            new Person("Eve", 32, "Paris")
        );
        
        // Filter - select elements
        System.out.println("People over 30:");
        people.stream()
               .filter(person -> person.getAge() > 30)
               .forEach(System.out::println);
        
        // Map - transform elements
        System.out.println("\nNames in uppercase:");
        people.stream()
               .map(Person::getName)
               .map(String::toUpperCase)
               .forEach(System.out::println);
        
        // Distinct - remove duplicates
        System.out.println("\nUnique cities:");
        people.stream()
               .map(Person::getCity)
               .distinct()
               .forEach(System.out::println);
        
        // Sorted - sort elements
        System.out.println("\nPeople sorted by age:");
        people.stream()
               .sorted(Comparator.comparing(Person::getAge))
               .forEach(System.out::println);
        
        // Limit and Skip
        System.out.println("\nFirst 2 people after skipping 1:");
        people.stream()
               .skip(1)
               .limit(2)
               .forEach(System.out::println);
    }
}
```

#### 3. Terminal Operations (Eager)
```java
import java.util.*;
import java.util.stream.*;

public class TerminalOperations {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // forEach - perform action on each element
        System.out.println("All numbers:");
        numbers.stream().forEach(System.out::print);
        System.out.println();
        
        // collect - collect to collection
        List<Integer> evenNumbers = numbers.stream()
            .filter(n -> n % 2 == 0)
            .collect(Collectors.toList());
        System.out.println("Even numbers: " + evenNumbers);
        
        // reduce - combine elements
        Optional<Integer> sum = numbers.stream()
            .reduce((a, b) -> a + b);
        System.out.println("Sum: " + sum.orElse(0));
        
        // count - count elements
        long count = numbers.stream()
            .filter(n -> n > 5)
            .count();
        System.out.println("Numbers > 5: " + count);
        
        // anyMatch, allMatch, noneMatch
        boolean anyEven = numbers.stream().anyMatch(n -> n % 2 == 0);
        boolean allPositive = numbers.stream().allMatch(n -> n > 0);
        boolean noneNegative = numbers.stream().noneMatch(n -> n < 0);
        
        System.out.println("Any even: " + anyEven);
        System.out.println("All positive: " + allPositive);
        System.out.println("None negative: " + noneNegative);
        
        // findFirst, findAny
        Optional<Integer> firstEven = numbers.stream()
            .filter(n -> n % 2 == 0)
            .findFirst();
        System.out.println("First even: " + firstEven.orElse(-1));
        
        // min, max
        Optional<Integer> min = numbers.stream().min(Integer::compareTo);
        Optional<Integer> max = numbers.stream().max(Integer::compareTo);
        System.out.println("Min: " + min.orElse(-1));
        System.out.println("Max: " + max.orElse(-1));
    }
}
```

---

## Method References

### What are Method References?
Method references provide a way to refer to methods without executing them. They are used to make lambda expressions even more readable and concise.

### Types of Method References

#### 1. Static Method References
```java
import java.util.*;
import java.util.function.*;

public class StaticMethodReferences {
    public static void main(String[] args) {
        List<String> numbers = Arrays.asList("1", "2", "3", "4", "5");
        
        // Lambda expression
        numbers.stream()
               .map(s -> Integer.parseInt(s))
               .forEach(System.out::println);
        
        // Method reference
        numbers.stream()
               .map(Integer::parseInt)
               .forEach(System.out::println);
        
        // More examples
        List<Double> doubles = Arrays.asList(1.5, 2.7, 3.9);
        
        // Using Math.ceil
        doubles.stream()
               .map(Math::ceil)
               .forEach(System.out::println);
    }
}
```

#### 2. Instance Method References
```java
import java.util.*;

public class InstanceMethodReferences {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("apple", "banana", "cherry");
        
        // Reference to instance method of particular object
        words.forEach(System.out::println);
        
        // Reference to instance method of arbitrary object
        words.stream()
             .map(String::toUpperCase)  // Same as s -> s.toUpperCase()
             .forEach(System.out::println);
        
        // Sorting example
        words.sort(String::compareToIgnoreCase);
        System.out.println("Sorted: " + words);
    }
}
```

#### 3. Constructor References
```java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;

public class ConstructorReferences {
    static class Person {
        private String name;
        
        public Person(String name) {
            this.name = name;
        }
        
        public String getName() {
            return name;
        }
        
        @Override
        public String toString() {
            return "Person{name='" + name + "'}";
        }
    }
    
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
        
        // Lambda expression
        List<Person> people1 = names.stream()
            .map(name -> new Person(name))
            .collect(Collectors.toList());
        
        // Constructor reference
        List<Person> people2 = names.stream()
            .map(Person::new)
            .collect(Collectors.toList());
        
        people2.forEach(System.out::println);
        
        // Array constructor reference
        String[] array = names.stream()
            .toArray(String[]::new);
        
        System.out.println("Array: " + Arrays.toString(array));
    }
}
```

---

## Optional Class

### What is Optional?
Optional is a container class to express optional values instead of null references. It helps avoid NullPointerException and makes the code more readable.

### Real-world Analogy 📦
Think of Optional like a **package delivery**:
- **Empty Optional**: Empty box - you know there's supposed to be something, but it's not there
- **Present Optional**: Box with contents - you have what you expected
- **Checking**: You always check if the box has contents before using them
- **Safety**: No surprises, no "where's my package?" moments

### Basic Optional Operations

#### 1. Creating Optional
```java
import java.util.Optional;

public class OptionalCreation {
    public static void main(String[] args) {
        // Empty optional
        Optional<String> empty = Optional.empty();
        
        // Optional with value
        Optional<String> nonEmpty = Optional.of("Hello");
        
        // Optional that might be null
        String nullableValue = null;
        Optional<String> nullable = Optional.ofNullable(nullableValue);
        
        System.out.println("Empty: " + empty);
        System.out.println("Non-empty: " + nonEmpty);
        System.out.println("Nullable: " + nullable);
        
        // Checking if present
        System.out.println("Empty present: " + empty.isPresent());
        System.out.println("Non-empty present: " + nonEmpty.isPresent());
        
        // Java 11+ isEmpty method
        System.out.println("Empty is empty: " + empty.isEmpty());
    }
}
```

#### 2. Working with Optional Values
```java
import java.util.Optional;

public class OptionalOperations {
    public static void main(String[] args) {
        Optional<String> name = Optional.of("John");
        Optional<String> emptyName = Optional.empty();
        
        // Get value if present, otherwise throw exception
        try {
            System.out.println("Name: " + name.get());
            // System.out.println("Empty name: " + emptyName.get()); // Throws exception
        } catch (Exception e) {
            System.out.println("Exception: " + e.getMessage());
        }
        
        // Get value with default
        System.out.println("Name or default: " + name.orElse("Unknown"));
        System.out.println("Empty name or default: " + emptyName.orElse("Unknown"));
        
        // Get value with supplier
        System.out.println("Name or supplier: " + 
            name.orElseGet(() -> "Generated Name"));
        
        // Throw custom exception if empty
        try {
            String result = emptyName.orElseThrow(
                () -> new IllegalArgumentException("Name is required"));
        } catch (IllegalArgumentException e) {
            System.out.println("Custom exception: " + e.getMessage());
        }
        
        // If present, do something
        name.ifPresent(n -> System.out.println("Hello, " + n));
        emptyName.ifPresent(n -> System.out.println("Hello, " + n)); // Nothing happens
        
        // Java 9+ ifPresentOrElse
        name.ifPresentOrElse(
            n -> System.out.println("Found: " + n),
            () -> System.out.println("Not found")
        );
    }
}
```

#### 3. Optional Transformations
```java
import java.util.Optional;

public class OptionalTransformations {
    static class Person {
        private String name;
        private Optional<Address> address;
        
        public Person(String name, Address address) {
            this.name = name;
            this.address = Optional.ofNullable(address);
        }
        
        public String getName() { return name; }
        public Optional<Address> getAddress() { return address; }
    }
    
    static class Address {
        private String street;
        private String city;
        
        public Address(String street, String city) {
            this.street = street;
            this.city = city;
        }
        
        public String getStreet() { return street; }
        public String getCity() { return city; }
    }
    
    public static void main(String[] args) {
        Person personWithAddress = new Person("John", 
            new Address("123 Main St", "New York"));
        Person personWithoutAddress = new Person("Jane", null);
        
        // Map - transform if present
        Optional<String> johnCity = Optional.of(personWithAddress)
            .map(Person::getAddress)
            .map(addr -> addr.map(Address::getCity))
            .orElse(Optional.empty());
        
        // FlatMap - avoid nested Optionals
        Optional<String> johnCityFlat = Optional.of(personWithAddress)
            .map(Person::getAddress)
            .flatMap(addr -> addr.map(Address::getCity));
        
        // Filter - keep only if condition matches
        Optional<Person> youngPerson = Optional.of(personWithAddress)
            .filter(p -> p.getName().length() < 10);
        
        System.out.println("John's city (map): " + johnCity);
        System.out.println("John's city (flatMap): " + johnCityFlat);
        System.out.println("Young person: " + youngPerson.map(Person::getName));
        
        // Chaining operations
        String result = Optional.of(personWithAddress)
            .map(Person::getAddress)
            .flatMap(addr -> addr.map(Address::getCity))
            .map(String::toUpperCase)
            .orElse("NO CITY");
        
        System.out.println("Final result: " + result);
    }
}
```

---

## Default Methods in Interfaces

### What are Default Methods?
Default methods allow you to add new methods to interfaces without breaking existing implementations. They provide a default implementation that implementing classes can use or override.

### Evolution Example
```java
// Before Java 8 - would break existing implementations
interface Vehicle {
    void start();
    void stop();
    // Adding this would break all existing implementations
    // void honk();
}

// After Java 8 - backward compatible
interface VehicleWithDefault {
    void start();
    void stop();
    
    // Default method - won't break existing implementations
    default void honk() {
        System.out.println("Beep beep!");
    }
    
    // Static method in interface
    static void displayManufacturer() {
        System.out.println("Generic Vehicle Manufacturer");
    }
}

class Car implements VehicleWithDefault {
    @Override
    public void start() {
        System.out.println("Car started");
    }
    
    @Override
    public void stop() {
        System.out.println("Car stopped");
    }
    
    // Can choose to override default method
    @Override
    public void honk() {
        System.out.println("Car horn: BEEP BEEP!");
    }
}

class Bicycle implements VehicleWithDefault {
    @Override
    public void start() {
        System.out.println("Started pedaling");
    }
    
    @Override
    public void stop() {
        System.out.println("Stopped pedaling");
    }
    
    // Uses default honk() method
}
```

### Multiple Inheritance with Default Methods
```java
interface Flyable {
    default void move() {
        System.out.println("Flying through the air");
    }
}

interface Swimmable {
    default void move() {
        System.out.println("Swimming through water");
    }
}

// Must resolve conflict explicitly
class Duck implements Flyable, Swimmable {
    @Override
    public void move() {
        // Choose which default to use
        Flyable.super.move();
        // Or provide own implementation
        // System.out.println("Duck waddles, swims, and flies");
    }
}

public class DefaultMethodExample {
    public static void main(String[] args) {
        Car car = new Car();
        Bicycle bike = new Bicycle();
        Duck duck = new Duck();
        
        car.start();
        car.honk();
        car.stop();
        
        bike.start();
        bike.honk(); // Uses default implementation
        bike.stop();
        
        duck.move(); // Resolved conflict
        
        VehicleWithDefault.displayManufacturer(); // Static method
    }
}
```

---

## Interview Questions

### Q1: What are the advantages of lambda expressions over anonymous classes?
**Answer:**
1. **Conciseness**: Less boilerplate code
2. **Readability**: More expressive and easier to understand
3. **Performance**: No extra class file generation, better performance
4. **Functional programming**: Enables functional programming paradigms
5. **Type inference**: Compiler can infer types automatically

```java
// Anonymous class - verbose
Comparator<String> comp1 = new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return s1.length() - s2.length();
    }
};

// Lambda - concise
Comparator<String> comp2 = (s1, s2) -> s1.length() - s2.length();

// Method reference - even more concise
Comparator<String> comp3 = Comparator.comparing(String::length);
```

### Q2: Explain the difference between map() and flatMap() in streams.
**Answer:**
- **map()**: Transforms each element to another form (1-to-1 mapping)
- **flatMap()**: Transforms each element to a stream and flattens the result (1-to-many mapping)

```java
List<String> sentences = Arrays.asList("Hello world", "Java streams");

// map() - transforms to stream of arrays
Stream<String[]> mapped = sentences.stream()
    .map(sentence -> sentence.split(" "));

// flatMap() - transforms and flattens to stream of words
Stream<String> flatMapped = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")));

flatMapped.forEach(System.out::println); // Prints: Hello, world, Java, streams
```

### Q3: When should you use Optional instead of null?
**Answer:**
Use Optional when:
1. **Method return types** that might not have a value
2. **Optional fields** in classes
3. **Method parameters** that are optional
4. **Avoiding NullPointerException**

```java
// Bad - prone to NPE
public String findUserName(int id) {
    User user = database.findUser(id);
    return user.getName(); // Might throw NPE
}

// Good - explicit about possible absence
public Optional<String> findUserName(int id) {
    return database.findUser(id)
                  .map(User::getName);
}

// Usage
Optional<String> name = findUserName(123);
name.ifPresent(System.out::println);
String displayName = name.orElse("Unknown User");
```

### Q4: What is the purpose of functional interfaces in Java 8?
**Answer:**
Functional interfaces enable:
1. **Lambda expressions**: Target type for lambdas
2. **Method references**: Cleaner syntax
3. **Higher-order functions**: Functions that accept/return functions
4. **Functional programming**: Immutable, side-effect-free programming

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
    
    // Can have default methods
    default void printResult(int a, int b) {
        System.out.println("Result: " + calculate(a, b));
    }
    
    // Can have static methods
    static Calculator getAdder() {
        return (a, b) -> a + b;
    }
}

// Usage
Calculator adder = (a, b) -> a + b;
Calculator multiplier = (a, b) -> a * b;

adder.printResult(5, 3); // Result: 8
```

### Q5: Explain the Stream API processing model.
**Answer:**
Stream processing follows these principles:
1. **Lazy evaluation**: Intermediate operations are not executed until terminal operation
2. **Pipeline**: Operations are chained together
3. **Short-circuiting**: Some operations can terminate early
4. **Parallel processing**: Can be parallelized automatically

```java
List<String> words = Arrays.asList("apple", "banana", "cherry", "date");

// This creates a pipeline but doesn't execute until collect()
Stream<String> pipeline = words.stream()
    .filter(w -> {
        System.out.println("Filtering: " + w);
        return w.length() > 4;
    })
    .map(w -> {
        System.out.println("Mapping: " + w);
        return w.toUpperCase();
    });

System.out.println("Pipeline created, now collecting...");
List<String> result = pipeline.collect(Collectors.toList());
System.out.println("Result: " + result);

// Output shows lazy evaluation:
// Pipeline created, now collecting...
// Filtering: apple
// Mapping: apple
// Filtering: banana
// Mapping: banana
// Filtering: cherry
// Mapping: cherry
// Filtering: date
// Result: [APPLE, BANANA, CHERRY]
```

---

## Navigation
- [← Previous: Multithreading Basics](multithreading-basics.md)
- [🏠 Main Menu](README.md)
- [→ Next: Advanced Java Topics](advanced-java-topics.md)

---

*This guide covers essential Java 8 features with practical examples and interview preparation. Each concept builds upon previous knowledge to create a comprehensive understanding of modern Java programming.*
