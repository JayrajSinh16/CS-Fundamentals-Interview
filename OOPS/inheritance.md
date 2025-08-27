# 🏗️ Inheritance - The Second Pillar of OOP

Inheritance is a mechanism where a new class (child/subclass) acquires properties and behaviors of an existing class (parent/superclass). It's like a family tree where children inherit traits from their parents while being able to add their own unique characteristics.

## 🎯 Key Concepts

- **IS-A Relationship**: Child class "is a" type of parent class
- **Code Reusability**: Inherit existing functionality without rewriting
- **Method Overriding**: Child can provide specific implementation of parent method
- **Super Keyword**: Access parent class members
- **Constructor Chaining**: Parent constructor called before child constructor

## 🌍 Real-World Analogy

**Animal Kingdom**: A `Dog` inherits basic characteristics from `Animal` (eating, sleeping, breathing) but adds its own specific behaviors (barking, wagging tail). All dogs are animals, but not all animals are dogs.

## 📚 Types of Inheritance in Java

### 1. Single Inheritance
```
   Animal
     ↓
    Dog
```

### 2. Multilevel Inheritance
```
   Animal
     ↓
   Mammal
     ↓
    Dog
```

### 3. Hierarchical Inheritance
```
    Animal
   ↙  ↓  ↘
 Dog Cat Bird
```

**Note**: Java doesn't support multiple inheritance of classes (diamond problem) but supports it through interfaces.

## 💻 Practical Examples

### Example 1: Basic Inheritance - Vehicle Hierarchy

```java
// Parent/Base/Super class
class Vehicle {
    // Protected - accessible by subclasses
    protected String brand;
    protected String model;
    protected int year;
    protected boolean isRunning;
    
    // Constructor
    public Vehicle(String brand, String model, int year) {
        this.brand = brand;
        this.model = model;
        this.year = year;
        this.isRunning = false;
        System.out.println("Vehicle constructor called");
    }
    
    // Methods that can be inherited
    public void start() {
        if (!isRunning) {
            isRunning = true;
            System.out.println(brand + " " + model + " started");
        }
    }
    
    public void stop() {
        if (isRunning) {
            isRunning = false;
            System.out.println(brand + " " + model + " stopped");
        }
    }
    
    public void displayInfo() {
        System.out.printf("Vehicle: %s %s (%d) - %s%n", 
                brand, model, year, isRunning ? "Running" : "Stopped");
    }
    
    // Method that can be overridden
    public void honk() {
        System.out.println("Generic vehicle sound");
    }
    
    // Getters
    public String getBrand() { return brand; }
    public String getModel() { return model; }
    public int getYear() { return year; }
    public boolean isRunning() { return isRunning; }
}

// Child/Derived/Sub class
class Car extends Vehicle {
    private int numberOfDoors;
    private String fuelType;
    
    // Constructor - must call parent constructor
    public Car(String brand, String model, int year, int numberOfDoors, String fuelType) {
        super(brand, model, year); // Call parent constructor
        this.numberOfDoors = numberOfDoors;
        this.fuelType = fuelType;
        System.out.println("Car constructor called");
    }
    
    // Car-specific methods
    public void openTrunk() {
        System.out.println("Car trunk opened");
    }
    
    public void turnOnAC() {
        if (isRunning) {
            System.out.println("Air conditioning turned on");
        } else {
            System.out.println("Start the car first!");
        }
    }
    
    // Override parent method
    @Override
    public void honk() {
        System.out.println("Car horn: Beep beep!");
    }
    
    // Override to add car-specific information
    @Override
    public void displayInfo() {
        super.displayInfo(); // Call parent method
        System.out.printf("Car Details: %d doors, %s fuel%n", numberOfDoors, fuelType);
    }
    
    // Getters for car-specific fields
    public int getNumberOfDoors() { return numberOfDoors; }
    public String getFuelType() { return fuelType; }
}

// Another child class
class Motorcycle extends Vehicle {
    private boolean hasSidecar;
    private int engineCC;
    
    public Motorcycle(String brand, String model, int year, int engineCC, boolean hasSidecar) {
        super(brand, model, year);
        this.engineCC = engineCC;
        this.hasSidecar = hasSidecar;
        System.out.println("Motorcycle constructor called");
    }
    
    public void wheelie() {
        if (isRunning) {
            System.out.println("Performing wheelie!");
        } else {
            System.out.println("Start the motorcycle first!");
        }
    }
    
    @Override
    public void honk() {
        System.out.println("Motorcycle horn: Beep beep beep!");
    }
    
    @Override
    public void displayInfo() {
        super.displayInfo();
        System.out.printf("Motorcycle Details: %dCC engine, %s%n", 
                engineCC, hasSidecar ? "with sidecar" : "no sidecar");
    }
}

// Usage
public class InheritanceDemo {
    public static void main(String[] args) {
        System.out.println("=== Creating Car ===");
        Car car = new Car("Toyota", "Camry", 2023, 4, "Gasoline");
        car.start();           // Inherited method
        car.honk();           // Overridden method
        car.openTrunk();      // Car-specific method
        car.turnOnAC();       // Car-specific method
        car.displayInfo();    // Overridden method
        
        System.out.println("\n=== Creating Motorcycle ===");
        Motorcycle bike = new Motorcycle("Harley", "Davidson", 2022, 1200, false);
        bike.start();         // Inherited method
        bike.honk();         // Overridden method
        bike.wheelie();      // Motorcycle-specific method
        bike.displayInfo(); // Overridden method
        
        System.out.println("\n=== Polymorphism Example ===");
        Vehicle[] vehicles = {car, bike};
        for (Vehicle v : vehicles) {
            v.honk(); // Calls appropriate overridden method
        }
    }
}
```

### Example 2: Multilevel Inheritance - Employee Hierarchy

```java
// Base class
class Person {
    protected String name;
    protected int age;
    protected String address;
    
    public Person(String name, int age, String address) {
        this.name = name;
        this.age = age;
        this.address = address;
    }
    
    public void eat() {
        System.out.println(name + " is eating");
    }
    
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
    
    public void displayBasicInfo() {
        System.out.printf("Name: %s, Age: %d, Address: %s%n", name, age, address);
    }
    
    // Getters
    public String getName() { return name; }
    public int getAge() { return age; }
    public String getAddress() { return address; }
}

// First level inheritance
class Employee extends Person {
    protected String employeeId;
    protected double salary;
    protected String department;
    
    public Employee(String name, int age, String address, String employeeId, 
                   double salary, String department) {
        super(name, age, address); // Call Person constructor
        this.employeeId = employeeId;
        this.salary = salary;
        this.department = department;
    }
    
    public void work() {
        System.out.println(name + " is working in " + department + " department");
    }
    
    public void attendMeeting() {
        System.out.println(name + " is attending a meeting");
    }
    
    public double calculateMonthlyPay() {
        return salary / 12;
    }
    
    @Override
    public void displayBasicInfo() {
        super.displayBasicInfo(); // Call parent method
        System.out.printf("Employee ID: %s, Department: %s, Salary: $%.2f%n", 
                employeeId, department, salary);
    }
    
    // Getters
    public String getEmployeeId() { return employeeId; }
    public double getSalary() { return salary; }
    public String getDepartment() { return department; }
}

// Second level inheritance
class Manager extends Employee {
    private int teamSize;
    private double bonus;
    private String[] teamMembers;
    
    public Manager(String name, int age, String address, String employeeId,
                  double salary, String department, int teamSize, double bonus) {
        super(name, age, address, employeeId, salary, department);
        this.teamSize = teamSize;
        this.bonus = bonus;
        this.teamMembers = new String[teamSize];
    }
    
    public void conductMeeting() {
        System.out.println("Manager " + name + " is conducting team meeting");
    }
    
    public void reviewPerformance() {
        System.out.println("Manager " + name + " is reviewing team performance");
    }
    
    public void addTeamMember(String memberName, int index) {
        if (index >= 0 && index < teamSize) {
            teamMembers[index] = memberName;
            System.out.println(memberName + " added to team");
        }
    }
    
    // Override to include bonus
    @Override
    public double calculateMonthlyPay() {
        return (salary + bonus) / 12;
    }
    
    @Override
    public void work() {
        super.work(); // Call Employee's work method
        System.out.println("Additionally, managing a team of " + teamSize + " members");
    }
    
    @Override
    public void displayBasicInfo() {
        super.displayBasicInfo(); // Call Employee's displayBasicInfo
        System.out.printf("Manager Details: Team Size: %d, Bonus: $%.2f%n", 
                teamSize, bonus);
    }
    
    // Getters
    public int getTeamSize() { return teamSize; }
    public double getBonus() { return bonus; }
}

// Usage
public class MultilevelInheritanceDemo {
    public static void main(String[] args) {
        System.out.println("=== Creating Manager ===");
        Manager manager = new Manager("Alice Johnson", 35, "123 Main St", 
                                    "MGR001", 80000, "Engineering", 5, 10000);
        
        // Can access methods from all levels
        manager.eat();              // From Person
        manager.work();             // From Employee (overridden)
        manager.conductMeeting();   // From Manager
        manager.displayBasicInfo(); // Overridden method
        
        System.out.println("Monthly Pay: $" + manager.calculateMonthlyPay());
        
        manager.addTeamMember("Bob Smith", 0);
        manager.addTeamMember("Carol Davis", 1);
    }
}
```

### Example 3: Method Overriding and Super Keyword

```java
class Animal {
    protected String species;
    protected int age;
    
    public Animal(String species, int age) {
        this.species = species;
        this.age = age;
    }
    
    // Method to be overridden
    public void makeSound() {
        System.out.println("Animal makes a generic sound");
    }
    
    public void move() {
        System.out.println("Animal moves");
    }
    
    public void displayInfo() {
        System.out.printf("Species: %s, Age: %d years%n", species, age);
    }
    
    // Final method - cannot be overridden
    public final void breathe() {
        System.out.println("Animal breathes");
    }
}

class Dog extends Animal {
    private String breed;
    
    public Dog(String breed, int age) {
        super("Canine", age); // Call parent constructor with species
        this.breed = breed;
    }
    
    // Override parent method
    @Override
    public void makeSound() {
        System.out.println("Dog barks: Woof woof!");
    }
    
    @Override
    public void move() {
        System.out.println("Dog runs on four legs");
    }
    
    // Override and extend parent method
    @Override
    public void displayInfo() {
        super.displayInfo(); // Call parent method first
        System.out.println("Breed: " + breed);
    }
    
    // Dog-specific method
    public void wagTail() {
        System.out.println("Dog wags tail happily");
    }
    
    // Method that uses super to access parent method
    public void makeAllSounds() {
        super.makeSound(); // Call parent version
        this.makeSound();  // Call overridden version
    }
}

class Bird extends Animal {
    private double wingspan;
    private boolean canFly;
    
    public Bird(double wingspan, boolean canFly, int age) {
        super("Avian", age);
        this.wingspan = wingspan;
        this.canFly = canFly;
    }
    
    @Override
    public void makeSound() {
        System.out.println("Bird chirps: Tweet tweet!");
    }
    
    @Override
    public void move() {
        if (canFly) {
            System.out.println("Bird flies with " + wingspan + "m wingspan");
        } else {
            System.out.println("Bird walks on ground");
        }
    }
    
    @Override
    public void displayInfo() {
        super.displayInfo();
        System.out.printf("Wingspan: %.1fm, Can fly: %s%n", wingspan, canFly);
    }
    
    public void buildNest() {
        System.out.println("Bird builds a nest");
    }
}

// Demonstration of polymorphism with inheritance
public class OverridingDemo {
    public static void main(String[] args) {
        System.out.println("=== Creating Animals ===");
        
        Dog dog = new Dog("Golden Retriever", 3);
        Bird bird = new Bird(2.5, true, 2);
        
        // Method overriding demonstration
        System.out.println("\n=== Method Overriding ===");
        dog.makeSound();     // Calls Dog's version
        bird.makeSound();    // Calls Bird's version
        
        dog.makeAllSounds(); // Shows both parent and child versions
        
        // Polymorphism with inheritance
        System.out.println("\n=== Polymorphism ===");
        Animal[] animals = {dog, bird};
        
        for (Animal animal : animals) {
            animal.displayInfo(); // Calls appropriate overridden method
            animal.makeSound();   // Calls appropriate overridden method
            animal.move();        // Calls appropriate overridden method
            animal.breathe();     // Final method - same for all
            System.out.println("---");
        }
    }
}
```

### Example 4: Constructor Chaining

```java
class Shape {
    protected String color;
    protected boolean filled;
    
    // Default constructor
    public Shape() {
        this("Red", false);
        System.out.println("Shape default constructor");
    }
    
    // Parameterized constructor
    public Shape(String color, boolean filled) {
        this.color = color;
        this.filled = filled;
        System.out.println("Shape parameterized constructor: " + color);
    }
    
    public double getArea() {
        return 0.0; // Default implementation
    }
    
    public void displayInfo() {
        System.out.printf("Shape - Color: %s, Filled: %s%n", color, filled);
    }
}

class Rectangle extends Shape {
    private double length;
    private double width;
    
    // Constructor chaining with super()
    public Rectangle() {
        super(); // Calls Shape default constructor
        this.length = 1.0;
        this.width = 1.0;
        System.out.println("Rectangle default constructor");
    }
    
    public Rectangle(double length, double width) {
        super("Blue", true); // Calls Shape parameterized constructor
        this.length = length;
        this.width = width;
        System.out.println("Rectangle parameterized constructor");
    }
    
    public Rectangle(String color, boolean filled, double length, double width) {
        super(color, filled); // Calls Shape constructor
        this.length = length;
        this.width = width;
        System.out.println("Rectangle full constructor");
    }
    
    @Override
    public double getArea() {
        return length * width;
    }
    
    @Override
    public void displayInfo() {
        super.displayInfo(); // Call parent method
        System.out.printf("Rectangle - Length: %.1f, Width: %.1f, Area: %.2f%n", 
                length, width, getArea());
    }
}

class Square extends Rectangle {
    // Square is a special rectangle where length = width
    public Square() {
        super(); // Calls Rectangle default constructor
        System.out.println("Square default constructor");
    }
    
    public Square(double side) {
        super(side, side); // Calls Rectangle parameterized constructor
        System.out.println("Square parameterized constructor");
    }
    
    public Square(String color, boolean filled, double side) {
        super(color, filled, side, side); // Calls Rectangle full constructor
        System.out.println("Square full constructor");
    }
    
    @Override
    public void displayInfo() {
        System.out.printf("Square - Color: %s, Filled: %s, Side: %.1f, Area: %.2f%n",
                color, filled, length, getArea());
    }
}

// Demonstration
public class ConstructorChainingDemo {
    public static void main(String[] args) {
        System.out.println("=== Creating Square with default constructor ===");
        Square square1 = new Square();
        square1.displayInfo();
        
        System.out.println("\n=== Creating Square with parameterized constructor ===");
        Square square2 = new Square(5.0);
        square2.displayInfo();
        
        System.out.println("\n=== Creating Square with full constructor ===");
        Square square3 = new Square("Green", true, 3.0);
        square3.displayInfo();
    }
}
```

## ⚠️ Common Mistakes to Avoid

### 1. Forgetting to Call Super Constructor
```java
// ❌ Bad Practice
class Child extends Parent {
    public Child(String value) {
        // If Parent doesn't have default constructor, this will fail
        this.value = value;
    }
}

// ✅ Good Practice
class Child extends Parent {
    public Child(String value) {
        super(value); // Explicitly call parent constructor
        // Additional child-specific initialization
    }
}
```

### 2. Overriding Methods Incorrectly
```java
// ❌ Bad Practice - Wrong signature
class Parent {
    public void method(String s) { }
}

class Child extends Parent {
    public void method(int i) { } // This is overloading, not overriding!
}

// ✅ Good Practice - Correct override
class Child extends Parent {
    @Override
    public void method(String s) { // Same signature = overriding
        super.method(s); // Call parent if needed
        // Additional logic
    }
}
```

### 3. Making Everything Protected
```java
// ❌ Bad Practice - Unnecessarily exposing internals
class Parent {
    protected int value1; // Should this really be protected?
    protected int value2;
    protected void internalMethod() { } // Exposing implementation
}

// ✅ Good Practice - Proper access control
class Parent {
    private int value1;    // Keep private if not needed by children
    protected int value2;  // Only expose what children need
    
    protected final void templateMethod() { // Template method pattern
        step1();
        step2();
    }
    
    protected abstract void step1(); // For children to implement
    protected abstract void step2();
}
```

## 🎯 Benefits of Inheritance

### 1. **Code Reusability**
```java
// Write once, use many times
class Vehicle { /* common code */ }
class Car extends Vehicle { /* car-specific code */ }
class Truck extends Vehicle { /* truck-specific code */ }
```

### 2. **Method Overriding**
```java
// Customize behavior for different types
Animal animal = new Dog();
animal.makeSound(); // Calls Dog's version, not Animal's
```

### 3. **Polymorphism Support**
```java
// Treat different objects uniformly
Animal[] zoo = {new Dog(), new Cat(), new Bird()};
for (Animal animal : zoo) {
    animal.makeSound(); // Different sound for each animal
}
```

## 🔥 Interview Questions & Answers

### Q1: What is inheritance and what are its types?
**Answer**: Inheritance is a mechanism where a child class acquires properties and methods of a parent class. Types in Java:
- **Single**: Child inherits from one parent
- **Multilevel**: Chain of inheritance (A→B→C)
- **Hierarchical**: Multiple children from one parent
- **Multiple**: Not supported for classes (diamond problem), only through interfaces

### Q2: What's the difference between method overriding and method overloading?
**Answer**: 
- **Overriding**: Same method signature in child class, runtime polymorphism, @Override annotation
- **Overloading**: Different parameters for same method name, compile-time polymorphism, same class
- Overriding is about inheritance, overloading is about having multiple versions of a method

### Q3: What is the diamond problem and how does Java solve it?
**Answer**: Diamond problem occurs in multiple inheritance when a class inherits from two classes that have the same method. Java solves it by:
- **Not allowing multiple class inheritance**
- **Allowing multiple interface inheritance** with default method resolution rules
- **Using explicit interface method calls** when conflicts arise

### Q4: When would you use inheritance vs composition?
**Answer**: 
- **Use Inheritance when**: IS-A relationship exists, want to leverage polymorphism, extending existing functionality
- **Use Composition when**: HAS-A relationship exists, want more flexibility, avoiding tight coupling
- **"Favor composition over inheritance"** - composition is more flexible and less coupled

### Q5: What happens if you don't call super() in a constructor?
**Answer**: 
- **If parent has default constructor**: Java automatically calls super() as first statement
- **If parent has only parameterized constructor**: Compilation error occurs
- **super() must be first statement** in constructor if explicitly called
- **Cannot call both this() and super()** in same constructor

---
[← Back to Main Guide](./README.md) | [← Previous: Encapsulation](./encapsulation.md) | [Next: Polymorphism →](./polymorphism.md)
