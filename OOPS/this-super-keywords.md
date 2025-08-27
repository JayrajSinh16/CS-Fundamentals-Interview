# 🔗 this & super Keywords in Java - Reference Keywords

The `this` and `super` keywords are fundamental reference keywords in Java that provide access to current object and parent class members respectively. Understanding these keywords is essential for inheritance, method overriding, and constructor chaining in object-oriented programming.

## 🎯 Key Concepts

- **this Keyword**: References the current object instance
- **super Keyword**: References the immediate parent class
- **Constructor Chaining**: Calling constructors using this() and super()
- **Method Resolution**: Accessing overridden methods and hidden variables
- **Scope Resolution**: Distinguishing between local and instance variables

## 🌍 Real-World Analogy

**Family Tree Analogy**:
- **this** = "I myself" (referring to current person)
- **super** = "My parent" (referring to immediate parent)
- **this()** = "Let me introduce myself differently" (different constructor)
- **super()** = "Let my parent introduce me first" (parent constructor)
- **Method calls** = Family traditions (this.tradition() vs super.tradition())
- **Variable access** = Family attributes (my height vs parent's height)

## 🏗️ this & super Reference Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Object Memory Layout                     │
├─────────────────────────────────────────────────────────────┤
│  Child Object Instance                                      │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Child Class Data                       │   │
│  │  • Child instance variables                         │   │
│  │  • Child method references                          │   │
│  │  • this → points to entire object                  │   │
│  └─────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Parent Class Data                      │   │
│  │  • Parent instance variables                        │   │
│  │  • Parent method references                         │   │
│  │  • super → points to parent portion                │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘

Reference Resolution:
┌──────────────┬──────────────┬──────────────────────┐
│   Reference  │    Points    │      Description     │
├──────────────┼──────────────┼──────────────────────┤
│ this.field   │ Current obj  │ Instance variable    │
│ super.field  │ Parent part  │ Parent variable      │
│ this.method()│ Current obj  │ Overridden method    │
│ super.method│ Parent part  │ Parent method        │
│ this()       │ Constructor  │ Current class constr │
│ super()      │ Constructor  │ Parent class constr  │
└──────────────┴──────────────┴──────────────────────┘
```

## 💻 Practical Examples

### Example 1: Basic this and super Usage

```java
// ThisSuperBasicsDemo.java - Understanding this and super keywords
public class ThisSuperBasicsDemo {
    
    // Base class demonstrating super keyword usage
    static class Vehicle {
        protected String brand;
        protected int year;
        protected double price;
        
        // Constructor
        public Vehicle(String brand, int year, double price) {
            this.brand = brand;  // 'this' to distinguish parameter from field
            this.year = year;
            this.price = price;
            System.out.printf("Vehicle constructor: %s %d ($%.2f)%n", 
                    this.brand, this.year, this.price);
        }
        
        // Method to be overridden
        public void start() {
            System.out.printf("Starting %s %d vehicle%n", this.brand, this.year);
        }
        
        public void displayInfo() {
            System.out.printf("Vehicle: %s %d, Price: $%.2f%n", 
                    this.brand, this.year, this.price);
        }
        
        // Method using this to return current object (method chaining)
        public Vehicle setBrand(String brand) {
            this.brand = brand;
            return this; // Return current object for chaining
        }
        
        public Vehicle setYear(int year) {
            this.year = year;
            return this;
        }
        
        public Vehicle setPrice(double price) {
            this.price = price;
            return this;
        }
        
        // Protected method for subclass access
        protected void performMaintenance() {
            System.out.printf("Performing basic maintenance on %s%n", this.brand);
        }
        
        @Override
        public String toString() {
            return String.format("%s{brand='%s', year=%d, price=%.2f}", 
                    this.getClass().getSimpleName(), this.brand, this.year, this.price);
        }
    }
    
    // Subclass demonstrating this and super
    static class Car extends Vehicle {
        private int doors;
        private String fuelType;
        
        // Constructor using super() and this
        public Car(String brand, int year, double price, int doors, String fuelType) {
            // Must call parent constructor first
            super(brand, year, price); // Call parent constructor
            
            // Initialize child-specific fields using this
            this.doors = doors;
            this.fuelType = fuelType;
            
            System.out.printf("Car constructor: %d doors, %s fuel%n", 
                    this.doors, this.fuelType);
        }
        
        // Constructor chaining using this()
        public Car(String brand, int year, double price) {
            this(brand, year, price, 4, "Gasoline"); // Call another constructor
            System.out.println("Car constructor: Using default doors and fuel type");
        }
        
        // Overriding parent method - can use super to call parent version
        @Override
        public void start() {
            System.out.print("Car starting sequence: ");
            super.start(); // Call parent implementation
            System.out.printf("Car-specific startup: Checking %d doors and %s engine%n", 
                    this.doors, this.fuelType);
        }
        
        // Method demonstrating variable shadowing resolution
        public void demonstrateVariableAccess() {
            String brand = "Local Brand"; // Local variable shadows instance variable
            
            System.out.println("=== Variable Access Demonstration ===");
            System.out.printf("Local variable 'brand': %s%n", brand);
            System.out.printf("Instance variable 'this.brand': %s%n", this.brand);
            System.out.printf("Parent variable 'super.brand': %s%n", super.brand);
            // Note: this.brand and super.brand refer to same memory location
        }
        
        // Method chaining using this
        @Override
        public Car setBrand(String brand) {
            super.setBrand(brand); // Call parent method
            return this; // Return Car type for chaining
        }
        
        @Override
        public Car setYear(int year) {
            super.setYear(year);
            return this;
        }
        
        @Override
        public Car setPrice(double price) {
            super.setPrice(price);
            return this;
        }
        
        // Car-specific methods with this
        public Car setDoors(int doors) {
            this.doors = doors;
            return this;
        }
        
        public Car setFuelType(String fuelType) {
            this.fuelType = fuelType;
            return this;
        }
        
        // Override parent method while calling it
        @Override
        protected void performMaintenance() {
            super.performMaintenance(); // Call parent maintenance
            System.out.printf("Car-specific maintenance: Checking %s engine and %d doors%n", 
                    this.fuelType, this.doors);
        }
        
        // Method demonstrating super method call
        public void fullDisplayInfo() {
            super.displayInfo(); // Call parent display method
            System.out.printf("Car details: %d doors, %s fuel%n", 
                    this.doors, this.fuelType);
        }
        
        public int getDoors() {
            return this.doors;
        }
        
        public String getFuelType() {
            return this.fuelType;
        }
        
        @Override
        public String toString() {
            return String.format("Car{brand='%s', year=%d, price=%.2f, doors=%d, fuel='%s'}", 
                    this.brand, this.year, this.price, this.doors, this.fuelType);
        }
    }
    
    // Another subclass for comparison
    static class Motorcycle extends Vehicle {
        private boolean hasSidecar;
        private int engineCC;
        
        public Motorcycle(String brand, int year, double price, int engineCC, boolean hasSidecar) {
            super(brand, year, price);
            this.engineCC = engineCC;
            this.hasSidecar = hasSidecar;
            System.out.printf("Motorcycle constructor: %dCC engine, sidecar: %b%n", 
                    this.engineCC, this.hasSidecar);
        }
        
        // Constructor with this() chaining
        public Motorcycle(String brand, int year, double price, int engineCC) {
            this(brand, year, price, engineCC, false); // Default no sidecar
            System.out.println("Motorcycle constructor: Default no sidecar");
        }
        
        @Override
        public void start() {
            System.out.print("Motorcycle starting: ");
            super.start();
            System.out.printf("Kick-starting %dCC engine%s%n", 
                    this.engineCC, this.hasSidecar ? " with sidecar" : "");
        }
        
        @Override
        protected void performMaintenance() {
            super.performMaintenance();
            System.out.printf("Motorcycle maintenance: Checking %dCC engine and chain%n", 
                    this.engineCC);
        }
        
        public boolean isHasSidecar() {
            return this.hasSidecar;
        }
        
        public int getEngineCC() {
            return this.engineCC;
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== this & super Keywords Demonstration ===\n");
        
        demonstrateConstructorChaining();
        demonstrateMethodOverriding();
        demonstrateVariableAccess();
        demonstrateMethodChaining();
    }
    
    private static void demonstrateConstructorChaining() {
        System.out.println("--- Constructor Chaining ---");
        
        System.out.println("Creating Car with all parameters:");
        Car car1 = new Car("Toyota", 2022, 25000.0, 4, "Hybrid");
        
        System.out.println("\nCreating Car with default parameters (using this()):");
        Car car2 = new Car("Honda", 2021, 22000.0);
        
        System.out.println("\nCreating Motorcycle with all parameters:");
        Motorcycle bike1 = new Motorcycle("Harley", 2020, 15000.0, 1200, true);
        
        System.out.println("\nCreating Motorcycle with default sidecar (using this()):");
        Motorcycle bike2 = new Motorcycle("Yamaha", 2023, 8000.0, 600);
        
        System.out.println();
    }
    
    private static void demonstrateMethodOverriding() {
        System.out.println("--- Method Overriding with super ---");
        
        Vehicle vehicle = new Vehicle("Generic", 2020, 15000.0);
        Car car = new Car("BMW", 2022, 45000.0, 2, "Electric");
        Motorcycle motorcycle = new Motorcycle("Ducati", 2021, 18000.0, 900);
        
        System.out.println("Vehicle start:");
        vehicle.start();
        
        System.out.println("\nCar start (overridden with super call):");
        car.start();
        
        System.out.println("\nMotorcycle start (overridden with super call):");
        motorcycle.start();
        
        System.out.println("\nMaintenance procedures:");
        vehicle.performMaintenance();
        car.performMaintenance();
        motorcycle.performMaintenance();
        
        System.out.println();
    }
    
    private static void demonstrateVariableAccess() {
        System.out.println("--- Variable Access with this ---");
        
        Car car = new Car("Tesla", 2023, 55000.0, 4, "Electric");
        car.demonstrateVariableAccess();
        
        System.out.println("\nDisplay methods:");
        car.displayInfo(); // Parent method
        car.fullDisplayInfo(); // Child method calling super
        
        System.out.println();
    }
    
    private static void demonstrateMethodChaining() {
        System.out.println("--- Method Chaining with this ---");
        
        // Method chaining using this keyword
        Car car = new Car("Ford", 2020, 30000.0)
                .setBrand("Ford Mustang")
                .setYear(2023)
                .setPrice(45000.0)
                .setDoors(2)
                .setFuelType("V8 Gasoline");
        
        System.out.printf("Chained configuration result: %s%n", car);
        
        // Demonstrate that this refers to the same object
        Car sameCar = car.setBrand("Modified Ford");
        System.out.printf("Same object reference: %b%n", car == sameCar);
        System.out.printf("Final car state: %s%n", car);
    }
}

/*
this & super Key Points:
1. this refers to current object instance
2. super refers to immediate parent class
3. this() calls another constructor in same class
4. super() calls parent class constructor
5. this.method() calls current class method
6. super.method() calls parent class method
7. Used for variable shadowing resolution
8. Enable method chaining and constructor chaining
*/
```

### Example 2: Constructor Chaining Patterns

```java
// ConstructorChainingDemo.java - Advanced constructor chaining with this and super
public class ConstructorChainingDemo {
    
    // Base class with multiple constructors
    static class Employee {
        protected String firstName;
        protected String lastName;
        protected String email;
        protected double salary;
        protected String department;
        
        // Primary constructor - all parameters
        public Employee(String firstName, String lastName, String email, 
                       double salary, String department) {
            this.firstName = firstName;
            this.lastName = lastName;
            this.email = email;
            this.salary = salary;
            this.department = department;
            
            System.out.printf("Employee primary constructor: %s %s (%s)%n", 
                    this.firstName, this.lastName, this.department);
        }
        
        // Constructor with default department - uses this()
        public Employee(String firstName, String lastName, String email, double salary) {
            this(firstName, lastName, email, salary, "General"); // Call primary constructor
            System.out.println("Employee constructor: Using default department");
        }
        
        // Constructor with default salary and department - uses this()
        public Employee(String firstName, String lastName, String email) {
            this(firstName, lastName, email, 50000.0); // Call 4-parameter constructor
            System.out.println("Employee constructor: Using default salary and department");
        }
        
        // Constructor with auto-generated email - uses this()
        public Employee(String firstName, String lastName) {
            this(firstName, lastName, generateEmail(firstName, lastName)); // Call 3-param constructor
            System.out.println("Employee constructor: Using generated email");
        }
        
        // Helper method for email generation
        private static String generateEmail(String firstName, String lastName) {
            return (firstName + "." + lastName + "@company.com").toLowerCase();
        }
        
        // Copy constructor pattern using this()
        public Employee(Employee other) {
            this(other.firstName, other.lastName, other.email, other.salary, other.department);
            System.out.println("Employee copy constructor: Copying existing employee");
        }
        
        public void displayInfo() {
            System.out.printf("Employee: %s %s, %s, $%.2f, %s%n", 
                    this.firstName, this.lastName, this.email, this.salary, this.department);
        }
        
        protected void logCreation() {
            System.out.printf("Created employee: %s %s in %s department%n", 
                    this.firstName, this.lastName, this.department);
        }
        
        // Getters
        public String getFirstName() { return this.firstName; }
        public String getLastName() { return this.lastName; }
        public String getEmail() { return this.email; }
        public double getSalary() { return this.salary; }
        public String getDepartment() { return this.department; }
        
        @Override
        public String toString() {
            return String.format("Employee{name='%s %s', email='%s', salary=%.2f, dept='%s'}", 
                    this.firstName, this.lastName, this.email, this.salary, this.department);
        }
    }
    
    // Subclass with complex constructor chaining
    static class Manager extends Employee {
        private int teamSize;
        private String managerLevel;
        private double bonus;
        
        // Primary constructor for Manager
        public Manager(String firstName, String lastName, String email, 
                      double salary, String department, int teamSize, 
                      String managerLevel, double bonus) {
            super(firstName, lastName, email, salary, department); // Call parent constructor
            this.teamSize = teamSize;
            this.managerLevel = managerLevel;
            this.bonus = bonus;
            
            this.logCreation(); // Call parent method
            System.out.printf("Manager primary constructor: Level %s, Team size %d, Bonus $%.2f%n", 
                    this.managerLevel, this.teamSize, this.bonus);
        }
        
        // Constructor with default bonus - uses this()
        public Manager(String firstName, String lastName, String email, 
                      double salary, String department, int teamSize, String managerLevel) {
            this(firstName, lastName, email, salary, department, teamSize, managerLevel, 0.0);
            System.out.println("Manager constructor: Using default bonus (0)");
        }
        
        // Constructor with default level and bonus - uses this()
        public Manager(String firstName, String lastName, String email, 
                      double salary, String department, int teamSize) {
            this(firstName, lastName, email, salary, department, teamSize, "Junior", 0.0);
            System.out.println("Manager constructor: Using default level and bonus");
        }
        
        // Constructor inheriting from Employee constructors - uses super()
        public Manager(String firstName, String lastName, String email, double salary) {
            super(firstName, lastName, email, salary); // Call parent 4-param constructor
            this.teamSize = 5; // Default team size
            this.managerLevel = "Junior";
            this.bonus = 0.0;
            System.out.println("Manager constructor: From Employee 4-param, default manager values");
        }
        
        // Constructor using Employee 3-param constructor
        public Manager(String firstName, String lastName, String email) {
            super(firstName, lastName, email); // Call parent 3-param constructor
            this.teamSize = 3;
            this.managerLevel = "Trainee";
            this.bonus = 0.0;
            System.out.println("Manager constructor: From Employee 3-param, trainee manager");
        }
        
        // Copy constructor for Manager
        public Manager(Manager other) {
            super(other); // Call parent copy constructor
            this.teamSize = other.teamSize;
            this.managerLevel = other.managerLevel;
            this.bonus = other.bonus;
            System.out.println("Manager copy constructor: Copying existing manager");
        }
        
        // Factory method using constructor chaining
        public static Manager createSeniorManager(String firstName, String lastName, 
                                                 String department, int teamSize) {
            String email = firstName.toLowerCase() + "." + lastName.toLowerCase() + 
                          ".manager@company.com";
            return new Manager(firstName, lastName, email, 120000.0, department, 
                             teamSize, "Senior", 25000.0);
        }
        
        @Override
        public void displayInfo() {
            super.displayInfo(); // Call parent method
            System.out.printf("Manager details: Level %s, Team size %d, Bonus $%.2f%n", 
                    this.managerLevel, this.teamSize, this.bonus);
        }
        
        // Override parent method while using super
        @Override
        protected void logCreation() {
            super.logCreation(); // Call parent logging
            System.out.printf("Manager-specific log: %s level manager with %d team members%n", 
                    this.managerLevel, this.teamSize);
        }
        
        public int getTeamSize() { return this.teamSize; }
        public String getManagerLevel() { return this.managerLevel; }
        public double getBonus() { return this.bonus; }
        
        @Override
        public String toString() {
            return String.format("Manager{name='%s %s', email='%s', salary=%.2f, " +
                               "dept='%s', level='%s', team=%d, bonus=%.2f}", 
                    this.firstName, this.lastName, this.email, this.salary, 
                    this.department, this.managerLevel, this.teamSize, this.bonus);
        }
    }
    
    // Further subclass demonstrating deep inheritance chain
    static class Director extends Manager {
        private String region;
        private int managersUnder;
        private double stockOptions;
        
        // Primary constructor
        public Director(String firstName, String lastName, String email, 
                       double salary, String department, int teamSize, 
                       String managerLevel, double bonus, String region, 
                       int managersUnder, double stockOptions) {
            super(firstName, lastName, email, salary, department, 
                  teamSize, managerLevel, bonus); // Call Manager constructor
            this.region = region;
            this.managersUnder = managersUnder;
            this.stockOptions = stockOptions;
            
            System.out.printf("Director constructor: Region %s, %d managers, $%.2f stock options%n", 
                    this.region, this.managersUnder, this.stockOptions);
        }
        
        // Constructor using Manager factory method
        public Director(String firstName, String lastName, String department, 
                       String region, int managersUnder, double stockOptions) {
            // Can't directly call static method in super(), so we use this approach
            super(firstName, lastName, 
                  firstName.toLowerCase() + "." + lastName.toLowerCase() + ".director@company.com",
                  180000.0, department, managersUnder * 5, "Director", 50000.0);
            
            this.region = region;
            this.managersUnder = managersUnder;
            this.stockOptions = stockOptions;
            
            System.out.println("Director constructor: Using simplified parameters");
        }
        
        // Constructor with all defaults except name and region
        public Director(String firstName, String lastName, String region) {
            this(firstName, lastName, "Executive", region, 3, 100000.0);
            System.out.println("Director constructor: Using regional defaults");
        }
        
        @Override
        public void displayInfo() {
            super.displayInfo(); // Call Manager's displayInfo (which calls Employee's)
            System.out.printf("Director details: Region %s, %d managers, $%.2f stock options%n", 
                    this.region, this.managersUnder, this.stockOptions);
        }
        
        @Override
        protected void logCreation() {
            super.logCreation(); // Call Manager's logCreation
            System.out.printf("Director log: Responsible for %s region with %d managers%n", 
                    this.region, this.managersUnder);
        }
        
        public String getRegion() { return this.region; }
        public int getManagersUnder() { return this.managersUnder; }
        public double getStockOptions() { return this.stockOptions; }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Constructor Chaining Demonstration ===\n");
        
        demonstrateEmployeeConstructors();
        demonstrateManagerConstructors();
        demonstrateDirectorConstructors();
        demonstrateCopyConstructors();
    }
    
    private static void demonstrateEmployeeConstructors() {
        System.out.println("--- Employee Constructor Chaining ---");
        
        System.out.println("1. Primary constructor (all parameters):");
        Employee emp1 = new Employee("John", "Doe", "john.doe@company.com", 
                                   75000.0, "Engineering");
        
        System.out.println("\n2. 4-parameter constructor (default department):");
        Employee emp2 = new Employee("Jane", "Smith", "jane.smith@company.com", 65000.0);
        
        System.out.println("\n3. 3-parameter constructor (default salary and department):");
        Employee emp3 = new Employee("Bob", "Johnson", "bob.johnson@company.com");
        
        System.out.println("\n4. 2-parameter constructor (generated email):");
        Employee emp4 = new Employee("Alice", "Brown");
        
        System.out.println("\nEmployee details:");
        emp1.displayInfo();
        emp2.displayInfo();
        emp3.displayInfo();
        emp4.displayInfo();
        
        System.out.println();
    }
    
    private static void demonstrateManagerConstructors() {
        System.out.println("--- Manager Constructor Chaining ---");
        
        System.out.println("1. Manager primary constructor:");
        Manager mgr1 = new Manager("Sarah", "Wilson", "sarah.wilson@company.com", 
                                 95000.0, "Sales", 8, "Senior", 15000.0);
        
        System.out.println("\n2. Manager with default bonus:");
        Manager mgr2 = new Manager("Mike", "Davis", "mike.davis@company.com", 
                                 85000.0, "Marketing", 6, "Junior");
        
        System.out.println("\n3. Manager from Employee 4-param:");
        Manager mgr3 = new Manager("Lisa", "Garcia", "lisa.garcia@company.com", 90000.0);
        
        System.out.println("\n4. Manager factory method:");
        Manager mgr4 = Manager.createSeniorManager("Tom", "Anderson", "Operations", 12);
        
        System.out.println("\nManager details:");
        mgr1.displayInfo();
        mgr2.displayInfo();
        mgr3.displayInfo();
        mgr4.displayInfo();
        
        System.out.println();
    }
    
    private static void demonstrateDirectorConstructors() {
        System.out.println("--- Director Constructor Chaining ---");
        
        System.out.println("1. Director simplified constructor:");
        Director dir1 = new Director("Robert", "Taylor", "Technology", "West Coast", 4, 150000.0);
        
        System.out.println("\n2. Director with regional defaults:");
        Director dir2 = new Director("Maria", "Rodriguez", "East Coast");
        
        System.out.println("\nDirector details:");
        dir1.displayInfo();
        dir2.displayInfo();
        
        System.out.println();
    }
    
    private static void demonstrateCopyConstructors() {
        System.out.println("--- Copy Constructor Patterns ---");
        
        System.out.println("Original Employee:");
        Employee originalEmp = new Employee("David", "Lee", "david.lee@company.com", 
                                          70000.0, "HR");
        originalEmp.displayInfo();
        
        System.out.println("\nCopying Employee:");
        Employee copiedEmp = new Employee(originalEmp);
        copiedEmp.displayInfo();
        
        System.out.println("\nOriginal Manager:");
        Manager originalMgr = new Manager("Emma", "White", "emma.white@company.com", 
                                        100000.0, "Finance", 7, "Senior", 20000.0);
        originalMgr.displayInfo();
        
        System.out.println("\nCopying Manager:");
        Manager copiedMgr = new Manager(originalMgr);
        copiedMgr.displayInfo();
        
        System.out.printf("\nObject identity check - Employee: %b%n", originalEmp == copiedEmp);
        System.out.printf("Object identity check - Manager: %b%n", originalMgr == copiedMgr);
    }
}

/*
Constructor Chaining Best Practices:
1. Always call super() first in subclass constructors
2. Use this() to avoid code duplication in constructors
3. Design primary constructor with all parameters
4. Create convenience constructors using this()
5. Provide copy constructors when needed
6. Use factory methods for complex object creation
7. Validate parameters in primary constructor
*/
```

### Example 3: Method Resolution and Variable Shadowing

```java
// MethodResolutionDemo.java - Advanced this/super usage for method and variable resolution
public class MethodResolutionDemo {
    
    // Base class with methods and variables
    static class Animal {
        protected String name = "Generic Animal";
        protected String species;
        protected int age;
        
        public Animal(String species, int age) {
            this.species = species;
            this.age = age;
        }
        
        // Method to be overridden
        public void makeSound() {
            System.out.printf("%s makes a generic animal sound%n", this.name);
        }
        
        // Method to be hidden (same signature in subclass)
        public void displayType() {
            System.out.printf("Animal type: %s (from Animal class)%n", this.getClass().getSimpleName());
        }
        
        // Method that calls other methods - demonstrates dynamic binding
        public void performActions() {
            System.out.println("=== Animal Actions ===");
            this.makeSound(); // Will call overridden version if available
            this.displayType(); // Will call current class version
            this.showDetails(); // Will call overridden version if available
        }
        
        // Protected method for subclass access
        protected void showDetails() {
            System.out.printf("Animal details: %s, %s, %d years old%n", 
                    this.name, this.species, this.age);
        }
        
        // Final method - cannot be overridden
        public final void breathe() {
            System.out.printf("%s is breathing%n", this.name);
        }
        
        public String getName() { return this.name; }
        public String getSpecies() { return this.species; }
        public int getAge() { return this.age; }
        
        @Override
        public String toString() {
            return String.format("Animal{name='%s', species='%s', age=%d}", 
                    this.name, this.species, this.age);
        }
    }
    
    // Subclass demonstrating method overriding and variable shadowing
    static class Dog extends Animal {
        // Variable shadowing - hides parent variable
        protected String name = "Dog"; // Shadows Animal.name
        private String breed;
        private boolean isTrained;
        
        public Dog(String breed, int age, boolean isTrained) {
            super("Canine", age); // Call parent constructor
            this.breed = breed;
            this.isTrained = isTrained;
        }
        
        // Override parent method
        @Override
        public void makeSound() {
            System.out.printf("%s barks: Woof! Woof!%n", this.name);
        }
        
        // Override parent method with super call
        @Override
        protected void showDetails() {
            super.showDetails(); // Call parent implementation
            System.out.printf("Dog-specific details: %s breed, trained: %b%n", 
                    this.breed, this.isTrained);
        }
        
        // Method demonstrating variable shadowing resolution
        public void demonstrateVariableShadowing() {
            String name = "Local Dog"; // Local variable
            
            System.out.println("=== Variable Shadowing Demonstration ===");
            System.out.printf("Local variable 'name': %s%n", name);
            System.out.printf("This class variable 'this.name': %s%n", this.name);
            System.out.printf("Parent class variable 'super.name': %s%n", super.name);
            
            // Accessing different versions of the same variable name
            System.out.printf("Species from parent: %s%n", super.species);
            System.out.printf("Species from this: %s%n", this.species); // Same as super.species
        }
        
        // Method demonstrating explicit method calls
        public void demonstrateMethodCalls() {
            System.out.println("=== Method Call Demonstration ===");
            
            System.out.println("Calling this.makeSound():");
            this.makeSound(); // Calls Dog's overridden version
            
            System.out.println("Calling super.makeSound():");
            super.makeSound(); // Calls Animal's original version
            
            System.out.println("Calling this.displayType():");
            this.displayType(); // Calls inherited method (Dog inherits Animal's version)
            
            System.out.println("Calling this.breathe():");
            this.breathe(); // Calls final method from parent
        }
        
        // Method that calls super method and adds functionality
        public void extendedPerformActions() {
            super.performActions(); // Call parent's implementation
            System.out.println("=== Additional Dog Actions ===");
            this.wagTail();
            this.playFetch();
        }
        
        // Dog-specific methods
        public void wagTail() {
            System.out.printf("%s is wagging tail happily%n", this.name);
        }
        
        public void playFetch() {
            if (this.isTrained) {
                System.out.printf("%s is playing fetch%n", this.name);
            } else {
                System.out.printf("%s doesn't know how to play fetch yet%n", this.name);
            }
        }
        
        public String getBreed() { return this.breed; }
        public boolean isTrained() { return this.isTrained; }
        
        @Override
        public String toString() {
            return String.format("Dog{name='%s', breed='%s', age=%d, trained=%b}", 
                    this.name, this.breed, this.age, this.isTrained);
        }
    }
    
    // Another subclass for comparison
    static class Cat extends Animal {
        protected String name = "Cat"; // Also shadows Animal.name
        private boolean isIndoor;
        private String favoriteSpot;
        
        public Cat(int age, boolean isIndoor, String favoriteSpot) {
            super("Feline", age);
            this.isIndoor = isIndoor;
            this.favoriteSpot = favoriteSpot;
        }
        
        @Override
        public void makeSound() {
            System.out.printf("%s meows: Meow! Purr...%n", this.name);
        }
        
        @Override
        protected void showDetails() {
            super.showDetails();
            System.out.printf("Cat-specific details: %s cat, favorite spot: %s%n", 
                    this.isIndoor ? "Indoor" : "Outdoor", this.favoriteSpot);
        }
        
        // Cat-specific method using this
        public void demonstrateCatBehavior() {
            System.out.println("=== Cat Behavior ===");
            this.makeSound();
            this.purr();
            this.sleep();
        }
        
        public void purr() {
            System.out.printf("%s is purring contentedly%n", this.name);
        }
        
        public void sleep() {
            System.out.printf("%s is sleeping in %s%n", this.name, this.favoriteSpot);
        }
        
        // Method demonstrating variable access patterns
        public void compareVariableAccess() {
            System.out.println("=== Cat Variable Access ===");
            System.out.printf("Cat name (this.name): %s%n", this.name);
            System.out.printf("Animal name (super.name): %s%n", super.name);
            System.out.printf("Are they different objects? %b%n", this.name != super.name);
            System.out.printf("Species (inherited): %s%n", this.species);
        }
        
        public boolean isIndoor() { return this.isIndoor; }
        public String getFavoriteSpot() { return this.favoriteSpot; }
    }
    
    // Class demonstrating complex inheritance with method chaining
    static class ServiceDog extends Dog {
        private String serviceType;
        private String certification;
        
        public ServiceDog(String breed, int age, String serviceType, String certification) {
            super(breed, age, true); // Service dogs are always trained
            this.serviceType = serviceType;
            this.certification = certification;
        }
        
        @Override
        public void makeSound() {
            System.out.printf("%s (Service Dog) barks professionally: Woof!%n", super.name);
        }
        
        @Override
        protected void showDetails() {
            super.showDetails(); // Calls Dog's showDetails (which calls Animal's)
            System.out.printf("Service Dog details: %s service, certification: %s%n", 
                    this.serviceType, this.certification);
        }
        
        // Method demonstrating complex super usage
        public void performServiceActions() {
            System.out.println("=== Service Dog Actions ===");
            
            // Call grandparent method directly (skip Dog's implementation)
            System.out.println("Calling Animal.makeSound() through super.super pattern:");
            // Note: super.super.makeSound() is not valid syntax in Java
            // Instead, we need to use a different approach
            this.callGrandparentMakeSound();
            
            // Call immediate parent method
            System.out.println("Calling Dog.makeSound():");
            super.makeSound();
            
            // Call own method
            System.out.println("Calling ServiceDog.makeSound():");
            this.makeSound();
            
            // Service-specific actions
            this.performService();
        }
        
        // Workaround to call grandparent method
        private void callGrandparentMakeSound() {
            // We can access grandparent behavior by temporarily using super in Dog
            System.out.printf("%s makes a generic animal sound (simulated grandparent call)%n", 
                    super.name);
        }
        
        public void performService() {
            System.out.printf("%s is performing %s service%n", super.name, this.serviceType);
        }
        
        public String getServiceType() { return this.serviceType; }
        public String getCertification() { return this.certification; }
        
        @Override
        public String toString() {
            return String.format("ServiceDog{name='%s', breed='%s', age=%d, service='%s', cert='%s'}", 
                    super.name, this.getBreed(), this.age, this.serviceType, this.certification);
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Method Resolution and Variable Shadowing ===\n");
        
        demonstrateBasicInheritance();
        demonstrateVariableShadowing();
        demonstrateMethodResolution();
        demonstrateComplexInheritance();
    }
    
    private static void demonstrateBasicInheritance() {
        System.out.println("--- Basic Inheritance ---");
        
        Animal animal = new Animal("Generic", 5);
        Dog dog = new Dog("Golden Retriever", 3, true);
        Cat cat = new Cat(2, true, "sunny windowsill");
        
        System.out.println("Animal actions:");
        animal.performActions();
        
        System.out.println("\nDog actions:");
        dog.performActions();
        
        System.out.println("\nCat actions:");
        cat.performActions();
        
        System.out.println();
    }
    
    private static void demonstrateVariableShadowing() {
        System.out.println("--- Variable Shadowing ---");
        
        Dog dog = new Dog("Labrador", 4, false);
        dog.demonstrateVariableShadowing();
        
        Cat cat = new Cat(3, false, "garden");
        cat.compareVariableAccess();
        
        System.out.println();
    }
    
    private static void demonstrateMethodResolution() {
        System.out.println("--- Method Resolution ---");
        
        Dog dog = new Dog("Beagle", 2, true);
        dog.demonstrateMethodCalls();
        
        System.out.println("\nExtended actions:");
        dog.extendedPerformActions();
        
        System.out.println();
    }
    
    private static void demonstrateComplexInheritance() {
        System.out.println("--- Complex Inheritance (ServiceDog) ---");
        
        ServiceDog serviceDog = new ServiceDog("German Shepherd", 4, "Guide", "CCI-2023");
        
        System.out.printf("Service Dog: %s%n", serviceDog);
        serviceDog.showDetails();
        
        serviceDog.performServiceActions();
        
        System.out.println("\nPolymorphism test:");
        Animal animalRef = serviceDog;
        Dog dogRef = serviceDog;
        
        System.out.println("Through Animal reference:");
        animalRef.makeSound(); // Calls ServiceDog's overridden method
        
        System.out.println("Through Dog reference:");
        dogRef.makeSound(); // Calls ServiceDog's overridden method
        
        System.out.println("Direct ServiceDog reference:");
        serviceDog.makeSound(); // Calls ServiceDog's method
    }
}

/*
Method Resolution and Variable Shadowing Rules:
1. Method calls use dynamic binding (runtime polymorphism)
2. Variable access uses static binding (compile-time resolution)
3. this.method() calls most specific implementation
4. super.method() calls parent implementation
5. Variable shadowing hides parent variables
6. super.variable accesses parent variable
7. this.variable accesses current class variable
8. Local variables shadow instance variables
*/
```

### Example 4: Advanced this/super Patterns and Best Practices

```java
// AdvancedThisSuperDemo.java - Advanced patterns and best practices
public class AdvancedThisSuperDemo {
    
    // Builder pattern using this for method chaining
    static class PersonBuilder {
        private String firstName;
        private String lastName;
        private int age;
        private String email;
        private String address;
        private String phone;
        
        // Each setter returns this for chaining
        public PersonBuilder setFirstName(String firstName) {
            this.firstName = firstName;
            return this; // Return current object for chaining
        }
        
        public PersonBuilder setLastName(String lastName) {
            this.lastName = lastName;
            return this;
        }
        
        public PersonBuilder setAge(int age) {
            if (age < 0 || age > 150) {
                throw new IllegalArgumentException("Invalid age: " + age);
            }
            this.age = age;
            return this;
        }
        
        public PersonBuilder setEmail(String email) {
            this.email = email;
            return this;
        }
        
        public PersonBuilder setAddress(String address) {
            this.address = address;
            return this;
        }
        
        public PersonBuilder setPhone(String phone) {
            this.phone = phone;
            return this;
        }
        
        // Validate and build
        public Person build() {
            this.validate(); // Use this explicitly for clarity
            return new Person(this); // Pass current builder to Person constructor
        }
        
        private void validate() {
            if (this.firstName == null || this.firstName.trim().isEmpty()) {
                throw new IllegalStateException("First name is required");
            }
            if (this.lastName == null || this.lastName.trim().isEmpty()) {
                throw new IllegalStateException("Last name is required");
            }
        }
        
        // Getters for Person constructor access
        String getFirstName() { return this.firstName; }
        String getLastName() { return this.lastName; }
        int getAge() { return this.age; }
        String getEmail() { return this.email; }
        String getAddress() { return this.address; }
        String getPhone() { return this.phone; }
    }
    
    // Person class using builder
    static class Person {
        private final String firstName;
        private final String lastName;
        private final int age;
        private final String email;
        private final String address;
        private final String phone;
        
        // Constructor takes builder
        Person(PersonBuilder builder) {
            this.firstName = builder.getFirstName();
            this.lastName = builder.getLastName();
            this.age = builder.getAge();
            this.email = builder.getEmail();
            this.address = builder.getAddress();
            this.phone = builder.getPhone();
        }
        
        // Factory method using builder pattern
        public static PersonBuilder builder() {
            return new PersonBuilder();
        }
        
        // Copy constructor using this()
        public Person(Person other) {
            this.firstName = other.firstName;
            this.lastName = other.lastName;
            this.age = other.age;
            this.email = other.email;
            this.address = other.address;
            this.phone = other.phone;
        }
        
        // Method using this for current object operations
        public Person withUpdatedAge(int newAge) {
            return Person.builder()
                    .setFirstName(this.firstName)
                    .setLastName(this.lastName)
                    .setAge(newAge)
                    .setEmail(this.email)
                    .setAddress(this.address)
                    .setPhone(this.phone)
                    .build();
        }
        
        public void displayInfo() {
            System.out.printf("Person: %s %s, Age: %d%n", this.firstName, this.lastName, this.age);
            if (this.email != null) System.out.printf("  Email: %s%n", this.email);
            if (this.address != null) System.out.printf("  Address: %s%n", this.address);
            if (this.phone != null) System.out.printf("  Phone: %s%n", this.phone);
        }
        
        // Getters
        public String getFirstName() { return this.firstName; }
        public String getLastName() { return this.lastName; }
        public int getAge() { return this.age; }
        public String getEmail() { return this.email; }
        public String getAddress() { return this.address; }
        public String getPhone() { return this.phone; }
        
        @Override
        public String toString() {
            return String.format("Person{name='%s %s', age=%d}", 
                    this.firstName, this.lastName, this.age);
        }
    }
    
    // Template method pattern with this/super
    abstract static class DataValidator {
        // Template method using this to call other methods
        public final boolean validate(String data) {
            if (this.preValidate(data)) {
                boolean result = this.performValidation(data);
                this.postValidate(data, result);
                return result;
            }
            return false;
        }
        
        // Hook method - can be overridden
        protected boolean preValidate(String data) {
            return data != null && !data.trim().isEmpty();
        }
        
        // Abstract method - must be implemented
        protected abstract boolean performValidation(String data);
        
        // Hook method - can be overridden
        protected void postValidate(String data, boolean result) {
            System.out.printf("Validation of '%s': %s%n", data, result ? "PASSED" : "FAILED");
        }
    }
    
    // Concrete validator
    static class EmailValidator extends DataValidator {
        @Override
        protected boolean performValidation(String data) {
            return data.contains("@") && data.contains(".") && 
                   !data.startsWith("@") && !data.endsWith("@");
        }
        
        @Override
        protected void postValidate(String data, boolean result) {
            super.postValidate(data, result); // Call parent implementation
            if (!result) {
                System.out.println("  Reason: Invalid email format");
            }
        }
    }
    
    // Another concrete validator
    static class PhoneValidator extends DataValidator {
        @Override
        protected boolean preValidate(String data) {
            boolean parentResult = super.preValidate(data); // Call parent pre-validation
            return parentResult && data.replaceAll("[^0-9]", "").length() >= 10;
        }
        
        @Override
        protected boolean performValidation(String data) {
            String digits = data.replaceAll("[^0-9]", "");
            return digits.length() >= 10 && digits.length() <= 15;
        }
    }
    
    // Fluent interface pattern using this
    static class QueryBuilder {
        private StringBuilder query;
        private boolean hasWhere;
        
        public QueryBuilder() {
            this.query = new StringBuilder();
            this.hasWhere = false;
        }
        
        public QueryBuilder select(String... columns) {
            this.query.append("SELECT ");
            this.query.append(String.join(", ", columns));
            return this; // Return this for chaining
        }
        
        public QueryBuilder from(String table) {
            this.query.append(" FROM ").append(table);
            return this;
        }
        
        public QueryBuilder where(String condition) {
            if (!this.hasWhere) {
                this.query.append(" WHERE ");
                this.hasWhere = true;
            } else {
                this.query.append(" AND ");
            }
            this.query.append(condition);
            return this;
        }
        
        public QueryBuilder orderBy(String column) {
            this.query.append(" ORDER BY ").append(column);
            return this;
        }
        
        public QueryBuilder limit(int count) {
            this.query.append(" LIMIT ").append(count);
            return this;
        }
        
        public String build() {
            return this.query.toString();
        }
        
        // Reset for reuse
        public QueryBuilder reset() {
            this.query = new StringBuilder();
            this.hasWhere = false;
            return this;
        }
    }
    
    // Callback pattern using this
    interface EventHandler {
        void handle(String event);
    }
    
    static class EventManager {
        private java.util.List<EventHandler> handlers;
        
        public EventManager() {
            this.handlers = new java.util.ArrayList<>();
        }
        
        public EventManager addHandler(EventHandler handler) {
            this.handlers.add(handler);
            return this; // Return this for chaining
        }
        
        public EventManager removeHandler(EventHandler handler) {
            this.handlers.remove(handler);
            return this;
        }
        
        public void fireEvent(String event) {
            System.out.printf("EventManager: Firing event '%s'%n", event);
            for (EventHandler handler : this.handlers) {
                handler.handle(event);
            }
        }
        
        // Inner class using outer this
        public class EventLogger implements EventHandler {
            @Override
            public void handle(String event) {
                // Access outer class instance using EventManager.this
                System.out.printf("Logger: Event '%s' handled by %s%n", 
                        event, EventManager.this.getClass().getSimpleName());
            }
        }
        
        // Factory method for inner class
        public EventLogger createLogger() {
            return this.new EventLogger(); // Create inner class instance
        }
    }
    
    // Recursive structure using this
    static class TreeNode {
        private String value;
        private java.util.List<TreeNode> children;
        private TreeNode parent;
        
        public TreeNode(String value) {
            this.value = value;
            this.children = new java.util.ArrayList<>();
            this.parent = null;
        }
        
        public TreeNode addChild(String childValue) {
            TreeNode child = new TreeNode(childValue);
            child.parent = this; // Set parent to current node
            this.children.add(child);
            return child; // Return child for potential chaining
        }
        
        public TreeNode addChild(TreeNode child) {
            child.parent = this;
            this.children.add(child);
            return this; // Return this for chaining
        }
        
        public TreeNode getParent() {
            return this.parent;
        }
        
        public TreeNode getRoot() {
            TreeNode current = this;
            while (current.parent != null) {
                current = current.parent;
            }
            return current;
        }
        
        public void printTree(int level) {
            String indent = "  ".repeat(level);
            System.out.printf("%s%s%n", indent, this.value);
            for (TreeNode child : this.children) {
                child.printTree(level + 1); // Recursive call on children
            }
        }
        
        public void printTree() {
            this.printTree(0); // Start with level 0
        }
        
        public String getValue() { return this.value; }
        public java.util.List<TreeNode> getChildren() { return this.children; }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Advanced this/super Patterns ===\n");
        
        demonstrateBuilderPattern();
        demonstrateTemplateMethod();
        demonstrateFluentInterface();
        demonstrateEventHandling();
        demonstrateTreeStructure();
    }
    
    private static void demonstrateBuilderPattern() {
        System.out.println("--- Builder Pattern with this ---");
        
        Person person = Person.builder()
                .setFirstName("John")
                .setLastName("Doe")
                .setAge(30)
                .setEmail("john.doe@example.com")
                .setAddress("123 Main St")
                .setPhone("555-1234")
                .build();
        
        person.displayInfo();
        
        // Create modified copy
        Person olderPerson = person.withUpdatedAge(35);
        System.out.println("\nAfter age update:");
        olderPerson.displayInfo();
        
        System.out.println();
    }
    
    private static void demonstrateTemplateMethod() {
        System.out.println("--- Template Method with super ---");
        
        DataValidator emailValidator = new EmailValidator();
        DataValidator phoneValidator = new PhoneValidator();
        
        String[] testEmails = {"user@example.com", "invalid-email", "@invalid.com"};
        String[] testPhones = {"555-123-4567", "123", "+1-800-555-0199"};
        
        System.out.println("Email validation:");
        for (String email : testEmails) {
            emailValidator.validate(email);
        }
        
        System.out.println("\nPhone validation:");
        for (String phone : testPhones) {
            phoneValidator.validate(phone);
        }
        
        System.out.println();
    }
    
    private static void demonstrateFluentInterface() {
        System.out.println("--- Fluent Interface with this ---");
        
        QueryBuilder builder = new QueryBuilder();
        
        String query1 = builder
                .select("name", "email", "age")
                .from("users")
                .where("age > 18")
                .where("status = 'active'")
                .orderBy("name")
                .limit(10)
                .build();
        
        System.out.printf("Query 1: %s%n", query1);
        
        String query2 = builder
                .reset()
                .select("*")
                .from("products")
                .where("price > 100")
                .orderBy("price DESC")
                .build();
        
        System.out.printf("Query 2: %s%n", query2);
        
        System.out.println();
    }
    
    private static void demonstrateEventHandling() {
        System.out.println("--- Event Handling with Inner Classes ---");
        
        EventManager manager = new EventManager();
        
        // Add lambda handler
        manager.addHandler(event -> System.out.printf("Lambda handler: Processing '%s'%n", event));
        
        // Add anonymous class handler
        manager.addHandler(new EventHandler() {
            @Override
            public void handle(String event) {
                System.out.printf("Anonymous handler: Received '%s'%n", event);
            }
        });
        
        // Add inner class handler
        EventManager.EventLogger logger = manager.createLogger();
        manager.addHandler(logger);
        
        // Fire events
        manager.fireEvent("UserLogin");
        manager.fireEvent("DataUpdated");
        
        System.out.println();
    }
    
    private static void demonstrateTreeStructure() {
        System.out.println("--- Tree Structure with this ---");
        
        TreeNode root = new TreeNode("Root");
        TreeNode child1 = root.addChild("Child 1");
        TreeNode child2 = root.addChild("Child 2");
        
        child1.addChild("Grandchild 1.1");
        child1.addChild("Grandchild 1.2");
        
        child2.addChild("Grandchild 2.1");
        
        System.out.println("Complete tree:");
        root.printTree();
        
        // Navigate from leaf to root
        TreeNode leaf = child1.getChildren().get(0);
        System.out.printf("\nLeaf node '%s' has root: '%s'%n", 
                leaf.getValue(), leaf.getRoot().getValue());
    }
}

/*
Advanced this/super Best Practices:
1. Use this for method chaining in builders
2. Use super to extend parent functionality
3. this() for constructor chaining within class
4. super() for parent constructor calls
5. Outer.this for inner class access to outer
6. this for disambiguation in variable shadowing
7. Return this from setters for fluent interfaces
8. Use super to avoid infinite recursion in overrides
*/
```

## 📊 this vs super Comparison

| Aspect | this | super |
|--------|------|-------|
| **References** | Current object | Parent class |
| **Constructor** | this() - same class | super() - parent class |
| **Methods** | this.method() - current/overridden | super.method() - parent version |
| **Variables** | this.field - current class | super.field - parent class |
| **Usage** | Disambiguation, chaining | Extending, accessing parent |

## ⚠️ Common this/super Mistakes

### 1. Forgetting super() call in constructor
```java
class Child extends Parent {
    public Child() {
        // ❌ If Parent has no default constructor, this fails
        // super(); // Should be explicit if Parent constructor has parameters
    }
}
```

### 2. Using this() and super() together
```java
class Example {
    public Example() {
        this(10); // ❌ Cannot use both this() and super() in same constructor
        super();  // Compilation error
    }
}
```

### 3. Calling super() not as first statement
```java
class Child extends Parent {
    public Child() {
        int x = 10;   // ❌ Cannot have statements before super()
        super();      // Compilation error
    }
}
```

## 🔥 Interview Questions & Answers

### Q1: What is the difference between this and super keywords?
**Answer**: 
- **this**: References current object instance, used for accessing current class members and constructor chaining
- **super**: References immediate parent class, used for accessing parent class members and calling parent constructor
- **this()**: Calls another constructor in the same class
- **super()**: Calls parent class constructor

### Q2: Can you use this() and super() in the same constructor?
**Answer**: **No**, you cannot use both `this()` and `super()` in the same constructor because:
- Both must be the first statement in constructor
- You can only have one first statement
- Use `this()` for constructor chaining within the same class
- Use `super()` for calling parent constructor

### Q3: What happens if you don't call super() explicitly?
**Answer**: Java automatically inserts `super()` call:
- **Default constructor exists**: Calls parent's no-argument constructor
- **No default constructor**: Compilation error if parent has only parameterized constructors
- **Best practice**: Always explicitly call `super()` with appropriate parameters

### Q4: How do you access a grandparent method in Java?
**Answer**: Java doesn't support `super.super.method()` syntax:
- **No direct access**: Cannot directly call grandparent methods
- **Workaround 1**: Create a helper method in parent class
- **Workaround 2**: Use composition instead of inheritance
- **Design consideration**: Deep inheritance hierarchies should be avoided

### Q5: When should you use this keyword explicitly?
**Answer**: Use `this` explicitly when:
- **Parameter shadowing**: Parameter name same as instance variable
- **Method chaining**: Return `this` for fluent interfaces
- **Constructor chaining**: Call `this()` to avoid code duplication
- **Clarity**: Make code more readable and explicit
- **Inner classes**: Distinguish between inner and outer class members

---
[← Back to Main Guide](./README.md) | [← Previous: Final Keyword](./final-keyword.md) | [Next: Method Overriding →](./method-overriding.md)
