# 🔒 Encapsulation - The First Pillar of OOP

Encapsulation is the bundling of data (attributes) and methods that operate on that data within a single unit (class), while restricting direct access to internal components. It's like a protective capsule that controls what goes in and what comes out.

## 🎯 Key Concepts

- **Data Hiding**: Internal implementation details are hidden from outside world
- **Access Control**: Using access modifiers to control visibility
- **Information Security**: Preventing unauthorized access and modification
- **Code Maintainability**: Changes to internal implementation don't affect external code
- **Validation**: Control how data is set and retrieved through methods

## 🔑 Access Modifiers in Java

| Modifier | Same Class | Same Package | Subclass | Different Package |
|----------|------------|--------------|----------|-------------------|
| `private` | ✅ | ❌ | ❌ | ❌ |
| `default` | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

## 🌍 Real-World Analogy

**ATM Machine**: You can withdraw money (public interface) but cannot access the internal cash vault or banking system (private implementation). The ATM validates your PIN and balance before allowing transactions.

## 💻 Practical Examples

### Example 1: Basic Encapsulation - Bank Account

```java
// ❌ Without Encapsulation (Poor Design)
class BankAccountBad {
    public String accountNumber;
    public double balance;
    public String ownerName;
    
    // Anyone can directly modify these fields!
    // account.balance = -1000; // This should not be allowed!
}

// ✅ With Proper Encapsulation
class BankAccount {
    // Private fields - cannot be accessed directly
    private String accountNumber;
    private double balance;
    private String ownerName;
    private boolean isActive;
    
    // Constructor
    public BankAccount(String accountNumber, String ownerName, double initialBalance) {
        this.accountNumber = accountNumber;
        this.ownerName = ownerName;
        this.balance = initialBalance >= 0 ? initialBalance : 0;
        this.isActive = true;
    }
    
    // Public getter methods (controlled access)
    public String getAccountNumber() {
        return accountNumber;
    }
    
    public double getBalance() {
        return isActive ? balance : 0;
    }
    
    public String getOwnerName() {
        return ownerName;
    }
    
    // Public setter with validation
    public void setOwnerName(String ownerName) {
        if (ownerName != null && !ownerName.trim().isEmpty()) {
            this.ownerName = ownerName;
        } else {
            throw new IllegalArgumentException("Owner name cannot be empty");
        }
    }
    
    // Controlled operations instead of direct field access
    public boolean deposit(double amount) {
        if (!isActive) {
            System.out.println("Account is inactive");
            return false;
        }
        
        if (amount > 0) {
            balance += amount;
            System.out.println("Deposited: $" + amount);
            return true;
        } else {
            System.out.println("Deposit amount must be positive");
            return false;
        }
    }
    
    public boolean withdraw(double amount) {
        if (!isActive) {
            System.out.println("Account is inactive");
            return false;
        }
        
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            System.out.println("Withdrawn: $" + amount);
            return true;
        } else {
            System.out.println("Invalid withdrawal amount or insufficient funds");
            return false;
        }
    }
    
    // Private helper method
    private void logTransaction(String type, double amount) {
        // Internal logging logic
        System.out.println("LOG: " + type + " of $" + amount + " at " + new java.util.Date());
    }
}

// Usage
public class EncapsulationDemo {
    public static void main(String[] args) {
        BankAccount account = new BankAccount("12345", "John Doe", 1000.0);
        
        // ✅ Controlled access through methods
        System.out.println("Balance: $" + account.getBalance());
        account.deposit(500);
        account.withdraw(200);
        
        // ❌ Cannot access private fields directly
        // account.balance = -500; // Compilation error
        // account.accountNumber = "hack"; // Compilation error
    }
}
```

### Example 2: Advanced Encapsulation - Student Grade System

```java
class Student {
    // Private fields
    private String studentId;
    private String name;
    private int age;
    private double[] grades;
    private int gradeCount;
    private static final int MAX_GRADES = 10;
    
    // Constructor with validation
    public Student(String studentId, String name, int age) {
        setStudentId(studentId);
        setName(name);
        setAge(age);
        this.grades = new double[MAX_GRADES];
        this.gradeCount = 0;
    }
    
    // Getters
    public String getStudentId() {
        return studentId;
    }
    
    public String getName() {
        return name;
    }
    
    public int getAge() {
        return age;
    }
    
    // Setters with validation
    private void setStudentId(String studentId) {
        if (studentId != null && studentId.matches("\\d{6}")) {
            this.studentId = studentId;
        } else {
            throw new IllegalArgumentException("Student ID must be 6 digits");
        }
    }
    
    public void setName(String name) {
        if (name != null && name.trim().length() >= 2) {
            this.name = name.trim();
        } else {
            throw new IllegalArgumentException("Name must be at least 2 characters");
        }
    }
    
    public void setAge(int age) {
        if (age >= 16 && age <= 100) {
            this.age = age;
        } else {
            throw new IllegalArgumentException("Age must be between 16 and 100");
        }
    }
    
    // Controlled grade operations
    public boolean addGrade(double grade) {
        if (gradeCount >= MAX_GRADES) {
            System.out.println("Maximum grades limit reached");
            return false;
        }
        
        if (grade >= 0 && grade <= 100) {
            grades[gradeCount++] = grade;
            return true;
        } else {
            System.out.println("Grade must be between 0 and 100");
            return false;
        }
    }
    
    // Read-only access to calculated values
    public double getAverageGrade() {
        if (gradeCount == 0) return 0;
        
        double sum = 0;
        for (int i = 0; i < gradeCount; i++) {
            sum += grades[i];
        }
        return sum / gradeCount;
    }
    
    public char getLetterGrade() {
        double avg = getAverageGrade();
        if (avg >= 90) return 'A';
        else if (avg >= 80) return 'B';
        else if (avg >= 70) return 'C';
        else if (avg >= 60) return 'D';
        else return 'F';
    }
    
    // Returns copy of grades array (not direct reference)
    public double[] getGrades() {
        double[] copy = new double[gradeCount];
        System.arraycopy(grades, 0, copy, 0, gradeCount);
        return copy;
    }
    
    // Private helper method for internal logic
    private boolean isPassingGrade(double grade) {
        return grade >= 60;
    }
    
    public int getPassingGradeCount() {
        int count = 0;
        for (int i = 0; i < gradeCount; i++) {
            if (isPassingGrade(grades[i])) {
                count++;
            }
        }
        return count;
    }
    
    @Override
    public String toString() {
        return String.format("Student[ID=%s, Name=%s, Age=%d, Average=%.2f, Grade=%c]",
                studentId, name, age, getAverageGrade(), getLetterGrade());
    }
}
```

### Example 3: Encapsulation with Inner Classes - Car Engine

```java
class Car {
    private String model;
    private Engine engine;
    private boolean isRunning;
    
    public Car(String model, int engineHorsepower) {
        this.model = model;
        this.engine = new Engine(engineHorsepower);
        this.isRunning = false;
    }
    
    // Public interface methods
    public void start() {
        if (!isRunning && engine.start()) {
            isRunning = true;
            System.out.println(model + " started successfully");
        } else {
            System.out.println("Failed to start " + model);
        }
    }
    
    public void stop() {
        if (isRunning) {
            engine.stop();
            isRunning = false;
            System.out.println(model + " stopped");
        }
    }
    
    public void accelerate() {
        if (isRunning) {
            engine.increaseRpm();
        } else {
            System.out.println("Start the car first!");
        }
    }
    
    public String getStatus() {
        return String.format("%s - %s (RPM: %d)", 
                model, 
                isRunning ? "Running" : "Stopped", 
                engine.getCurrentRpm());
    }
    
    // Private inner class - completely encapsulated
    private class Engine {
        private int horsepower;
        private int currentRpm;
        private boolean isStarted;
        private static final int MAX_RPM = 7000;
        
        private Engine(int horsepower) {
            this.horsepower = horsepower;
            this.currentRpm = 0;
            this.isStarted = false;
        }
        
        private boolean start() {
            if (!isStarted) {
                isStarted = true;
                currentRpm = 800; // Idle RPM
                return true;
            }
            return false;
        }
        
        private void stop() {
            isStarted = false;
            currentRpm = 0;
        }
        
        private void increaseRpm() {
            if (isStarted && currentRpm < MAX_RPM) {
                currentRpm += 500;
                System.out.println("Engine RPM increased to: " + currentRpm);
            }
        }
        
        private int getCurrentRpm() {
            return currentRpm;
        }
    }
}

// Usage
public class CarDemo {
    public static void main(String[] args) {
        Car car = new Car("Toyota Camry", 300);
        
        // Can only interact with car through public interface
        car.start();
        car.accelerate();
        System.out.println(car.getStatus());
        car.stop();
        
        // Cannot access engine directly
        // car.engine.start(); // Compilation error
    }
}
```

## ⚠️ Common Mistakes to Avoid

### 1. Making Everything Public
```java
// ❌ Bad Practice
class BadClass {
    public int value1;
    public String value2;
    public double value3;
    // No control over data access
}

// ✅ Good Practice
class GoodClass {
    private int value1;
    private String value2;
    private double value3;
    
    // Controlled access through methods
    public int getValue1() { return value1; }
    public void setValue1(int value1) {
        if (value1 >= 0) this.value1 = value1;
    }
}
```

### 2. Returning Mutable Objects Directly
```java
// ❌ Bad Practice
class BadList {
    private List<String> items = new ArrayList<>();
    
    public List<String> getItems() {
        return items; // Direct reference - can be modified!
    }
}

// ✅ Good Practice
class GoodList {
    private List<String> items = new ArrayList<>();
    
    public List<String> getItems() {
        return new ArrayList<>(items); // Return copy
    }
    
    // Or use immutable view
    public List<String> getItemsView() {
        return Collections.unmodifiableList(items);
    }
}
```

### 3. No Validation in Setters
```java
// ❌ Bad Practice
public void setAge(int age) {
    this.age = age; // No validation
}

// ✅ Good Practice
public void setAge(int age) {
    if (age >= 0 && age <= 150) {
        this.age = age;
    } else {
        throw new IllegalArgumentException("Age must be between 0 and 150");
    }
}
```

## 🎯 Benefits of Encapsulation

### 1. **Data Security**
```java
// Internal data is protected from unauthorized access
private double salary; // Cannot be accessed directly

public void setSalary(double salary) {
    if (hasPermission() && salary > 0) {
        this.salary = salary;
    }
}
```

### 2. **Flexibility**
```java
// Can change internal implementation without affecting clients
private String name; // Can change to StringBuilder internally

public String getName() {
    return name; // Interface remains same
}
```

### 3. **Maintainability**
```java
// Adding logging, validation, or caching doesn't break existing code
public void setValue(int value) {
    logAccess("setValue called");
    validateValue(value);
    this.value = value;
    updateCache();
}
```

## 🔥 Interview Questions & Answers

### Q1: What is encapsulation and why is it important?
**Answer**: Encapsulation is bundling data and methods together while hiding internal implementation details. It's important because:
- **Data Security**: Prevents unauthorized access and modification
- **Code Maintainability**: Internal changes don't affect external code
- **Validation**: Control how data is set and retrieved
- **Flexibility**: Can change implementation without breaking client code

### Q2: What's the difference between encapsulation and data hiding?
**Answer**: 
- **Encapsulation** is the broader concept of bundling data and methods together
- **Data Hiding** is one aspect of encapsulation - making internal data private
- Encapsulation includes data hiding plus providing controlled access through public methods
- Data hiding is about "what to hide", encapsulation is about "how to hide and provide access"

### Q3: How do you achieve encapsulation in Java?
**Answer**: 
1. **Make fields private**: Prevent direct access
2. **Provide public getter/setter methods**: Controlled access
3. **Add validation in setters**: Ensure data integrity
4. **Use access modifiers appropriately**: Control visibility levels
5. **Return copies of mutable objects**: Prevent external modification

### Q4: Can you break encapsulation in Java? How?
**Answer**: Yes, encapsulation can be broken:
- **Reflection API**: Can access private fields and methods
- **Serialization**: Can expose internal state
- **Returning mutable objects**: Direct reference allows modification
- **Package-private fields**: Same package classes can access

### Q5: What's the difference between encapsulation and abstraction?
**Answer**: 
- **Encapsulation**: Focuses on "how" - implementation hiding and data security
- **Abstraction**: Focuses on "what" - showing only essential features
- Encapsulation is about bundling and access control
- Abstraction is about simplifying complexity by hiding unnecessary details
- Both work together - abstraction defines interface, encapsulation protects implementation

---
[← Back to Main Guide](./README.md) | [Next: Inheritance →](./inheritance.md)
