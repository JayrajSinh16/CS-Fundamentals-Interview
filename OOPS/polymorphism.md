# 🔄 Polymorphism - The Third Pillar of OOP

Polymorphism means "one interface, multiple implementations" or "many forms." It allows objects of different types to be treated as objects of a common base type, while each object responds in its own way. Think of it as a universal remote that works with different devices - same buttons, different behaviors.

## 🎯 Key Concepts

- **Runtime Polymorphism**: Method overriding, determined at runtime (dynamic binding)
- **Compile-time Polymorphism**: Method overloading, determined at compile time (static binding)
- **Dynamic Method Dispatch**: JVM decides which method to call based on actual object type
- **Virtual Method Table**: Mechanism used by JVM for method resolution
- **Late Binding**: Method binding happens at runtime

## 🌍 Real-World Analogy

**Musical Instruments**: A `Musician` can play any `Instrument` (piano, guitar, drums). The `play()` method exists for all instruments, but each produces different sounds. The musician doesn't need to know the specifics of each instrument - just call `play()`.

## 📚 Types of Polymorphism

### 1. Compile-Time Polymorphism (Method Overloading)
- Same method name, different parameters
- Resolved at compile time
- Also called Static Polymorphism

### 2. Runtime Polymorphism (Method Overriding)
- Same method signature in parent and child
- Resolved at runtime based on object type
- Also called Dynamic Polymorphism

## 💻 Practical Examples

### Example 1: Runtime Polymorphism - Payment System

```java
// Base class defining common interface
abstract class Payment {
    protected double amount;
    protected String transactionId;
    
    public Payment(double amount, String transactionId) {
        this.amount = amount;
        this.transactionId = transactionId;
    }
    
    // Abstract method - must be implemented by children
    public abstract boolean processPayment();
    
    // Common method for all payment types
    public void generateReceipt() {
        System.out.printf("Receipt: Transaction %s - Amount: $%.2f%n", 
                transactionId, amount);
    }
    
    // Virtual method - can be overridden
    public void sendNotification() {
        System.out.println("Payment notification sent");
    }
    
    public double getAmount() { return amount; }
    public String getTransactionId() { return transactionId; }
}

// Credit Card Payment
class CreditCardPayment extends Payment {
    private String cardNumber;
    private String expiryDate;
    private String cvv;
    
    public CreditCardPayment(double amount, String transactionId, 
                           String cardNumber, String expiryDate, String cvv) {
        super(amount, transactionId);
        this.cardNumber = cardNumber;
        this.expiryDate = expiryDate;
        this.cvv = cvv;
    }
    
    @Override
    public boolean processPayment() {
        System.out.println("Processing credit card payment...");
        
        // Validate card details
        if (validateCard()) {
            System.out.printf("Credit card payment of $%.2f processed successfully%n", amount);
            return true;
        } else {
            System.out.println("Credit card validation failed");
            return false;
        }
    }
    
    @Override
    public void sendNotification() {
        System.out.println("SMS notification sent to registered mobile number");
    }
    
    private boolean validateCard() {
        // Simulate card validation
        return cardNumber.length() == 16 && cvv.length() == 3;
    }
}

// PayPal Payment
class PayPalPayment extends Payment {
    private String email;
    private String password;
    
    public PayPalPayment(double amount, String transactionId, String email, String password) {
        super(amount, transactionId);
        this.email = email;
        this.password = password;
    }
    
    @Override
    public boolean processPayment() {
        System.out.println("Processing PayPal payment...");
        
        if (authenticatePayPal()) {
            System.out.printf("PayPal payment of $%.2f processed via %s%n", amount, email);
            return true;
        } else {
            System.out.println("PayPal authentication failed");
            return false;
        }
    }
    
    @Override
    public void sendNotification() {
        System.out.println("Email notification sent to " + email);
    }
    
    private boolean authenticatePayPal() {
        // Simulate PayPal authentication
        return email.contains("@") && password.length() >= 6;
    }
}

// Bank Transfer Payment
class BankTransferPayment extends Payment {
    private String accountNumber;
    private String routingNumber;
    private String bankName;
    
    public BankTransferPayment(double amount, String transactionId, 
                             String accountNumber, String routingNumber, String bankName) {
        super(amount, transactionId);
        this.accountNumber = accountNumber;
        this.routingNumber = routingNumber;
        this.bankName = bankName;
    }
    
    @Override
    public boolean processPayment() {
        System.out.println("Processing bank transfer...");
        
        if (validateBankDetails()) {
            System.out.printf("Bank transfer of $%.2f processed via %s%n", amount, bankName);
            return true;
        } else {
            System.out.println("Bank details validation failed");
            return false;
        }
    }
    
    @Override
    public void sendNotification() {
        System.out.println("Email notification sent to registered email address");
    }
    
    private boolean validateBankDetails() {
        // Simulate bank validation
        return accountNumber.length() >= 8 && routingNumber.length() == 9;
    }
}

// Payment Processor - demonstrates polymorphism
class PaymentProcessor {
    
    // This method can handle any type of payment
    public boolean processTransaction(Payment payment) {
        System.out.println("=== Starting Transaction: " + payment.getTransactionId() + " ===");
        
        // Polymorphic call - actual method depends on object type
        boolean success = payment.processPayment();
        
        if (success) {
            payment.generateReceipt();    // Common method
            payment.sendNotification();  // Polymorphic call
            return true;
        } else {
            System.out.println("Transaction failed!");
            return false;
        }
    }
    
    // Process multiple payments polymorphically
    public void processBatch(Payment[] payments) {
        System.out.println("Processing batch of " + payments.length + " payments...");
        
        for (Payment payment : payments) {
            processTransaction(payment); // Polymorphism in action
            System.out.println("---");
        }
    }
}

// Usage demonstration
public class RuntimePolymorphismDemo {
    public static void main(String[] args) {
        PaymentProcessor processor = new PaymentProcessor();
        
        // Create different payment objects
        Payment creditCard = new CreditCardPayment(100.0, "TXN001", 
                "1234567890123456", "12/25", "123");
        
        Payment paypal = new PayPalPayment(250.0, "TXN002", 
                "user@example.com", "password123");
        
        Payment bankTransfer = new BankTransferPayment(500.0, "TXN003", 
                "12345678", "123456789", "Bank of America");
        
        // Polymorphism - same method call, different behaviors
        System.out.println("=== Individual Transactions ===");
        processor.processTransaction(creditCard);   // Calls CreditCardPayment methods
        processor.processTransaction(paypal);       // Calls PayPalPayment methods
        processor.processTransaction(bankTransfer); // Calls BankTransferPayment methods
        
        // Batch processing with polymorphism
        System.out.println("\n=== Batch Processing ===");
        Payment[] payments = {creditCard, paypal, bankTransfer};
        processor.processBatch(payments);
        
        // Runtime type checking
        System.out.println("\n=== Runtime Type Information ===");
        for (Payment payment : payments) {
            System.out.println("Payment type: " + payment.getClass().getSimpleName());
            System.out.println("Is CreditCard: " + (payment instanceof CreditCardPayment));
            System.out.println("Is PayPal: " + (payment instanceof PayPalPayment));
            System.out.println("---");
        }
    }
}
```

### Example 2: Compile-Time Polymorphism - Method Overloading

```java
class Calculator {
    
    // Method overloading - same name, different parameters
    
    // Add two integers
    public int add(int a, int b) {
        System.out.println("Adding two integers");
        return a + b;
    }
    
    // Add three integers
    public int add(int a, int b, int c) {
        System.out.println("Adding three integers");
        return a + b + c;
    }
    
    // Add two doubles
    public double add(double a, double b) {
        System.out.println("Adding two doubles");
        return a + b;
    }
    
    // Add array of integers
    public int add(int[] numbers) {
        System.out.println("Adding array of integers");
        int sum = 0;
        for (int num : numbers) {
            sum += num;
        }
        return sum;
    }
    
    // Add two strings (concatenation)
    public String add(String a, String b) {
        System.out.println("Concatenating two strings");
        return a + b;
    }
    
    // Variable arguments (varargs) - special case of overloading
    public int add(int... numbers) {
        System.out.println("Adding variable number of integers");
        int sum = 0;
        for (int num : numbers) {
            sum += num;
        }
        return sum;
    }
    
    // Method overloading with different parameter order
    public void display(String name, int age) {
        System.out.printf("Name: %s, Age: %d%n", name, age);
    }
    
    public void display(int age, String name) {
        System.out.printf("Age: %d, Name: %s%n", age, name);
    }
}

// Constructor overloading
class Student {
    private String name;
    private int age;
    private String course;
    private double gpa;
    
    // Default constructor
    public Student() {
        this("Unknown", 18, "Undeclared", 0.0);
    }
    
    // Constructor with name only
    public Student(String name) {
        this(name, 18, "Undeclared", 0.0);
    }
    
    // Constructor with name and age
    public Student(String name, int age) {
        this(name, age, "Undeclared", 0.0);
    }
    
    // Constructor with name, age, and course
    public Student(String name, int age, String course) {
        this(name, age, course, 0.0);
    }
    
    // Full constructor
    public Student(String name, int age, String course, double gpa) {
        this.name = name;
        this.age = age;
        this.course = course;
        this.gpa = gpa;
    }
    
    @Override
    public String toString() {
        return String.format("Student{name='%s', age=%d, course='%s', gpa=%.2f}", 
                name, age, course, gpa);
    }
}

// Usage demonstration
public class CompileTimePolymorphismDemo {
    public static void main(String[] args) {
        Calculator calc = new Calculator();
        
        System.out.println("=== Method Overloading Examples ===");
        
        // Compiler chooses appropriate method based on arguments
        System.out.println("Result: " + calc.add(5, 3));                    // int, int
        System.out.println("Result: " + calc.add(5, 3, 2));                // int, int, int
        System.out.println("Result: " + calc.add(5.5, 3.2));               // double, double
        System.out.println("Result: " + calc.add(new int[]{1, 2, 3, 4}));  // int array
        System.out.println("Result: " + calc.add("Hello", " World"));       // String, String
        System.out.println("Result: " + calc.add(1, 2, 3, 4, 5));          // varargs
        
        System.out.println("\n=== Parameter Order Overloading ===");
        calc.display("Alice", 25);    // String, int
        calc.display(25, "Bob");      // int, String
        
        System.out.println("\n=== Constructor Overloading ===");
        Student s1 = new Student();
        Student s2 = new Student("Charlie");
        Student s3 = new Student("Diana", 20);
        Student s4 = new Student("Eve", 22, "Computer Science");
        Student s5 = new Student("Frank", 21, "Mathematics", 3.8);
        
        System.out.println(s1);
        System.out.println(s2);
        System.out.println(s3);
        System.out.println(s4);
        System.out.println(s5);
    }
}
```

### Example 3: Advanced Polymorphism - Shape Drawing System

```java
// Abstract base class
abstract class Shape {
    protected String color;
    protected boolean filled;
    
    public Shape(String color, boolean filled) {
        this.color = color;
        this.filled = filled;
    }
    
    // Abstract methods - must be implemented by children
    public abstract double getArea();
    public abstract double getPerimeter();
    public abstract void draw();
    
    // Template method - uses polymorphic calls
    public final void render() {
        System.out.println("=== Rendering Shape ===");
        draw();                    // Polymorphic call
        displayProperties();       // Common behavior
        displayMeasurements();     // Uses polymorphic calls
    }
    
    // Common method for all shapes
    public void displayProperties() {
        System.out.printf("Color: %s, Filled: %s%n", color, filled);
    }
    
    // Method that uses polymorphic behavior
    public void displayMeasurements() {
        System.out.printf("Area: %.2f, Perimeter: %.2f%n", getArea(), getPerimeter());
    }
    
    // Getters
    public String getColor() { return color; }
    public boolean isFilled() { return filled; }
}

// Circle implementation
class Circle extends Shape {
    private double radius;
    
    public Circle(String color, boolean filled, double radius) {
        super(color, filled);
        this.radius = radius;
    }
    
    @Override
    public double getArea() {
        return Math.PI * radius * radius;
    }
    
    @Override
    public double getPerimeter() {
        return 2 * Math.PI * radius;
    }
    
    @Override
    public void draw() {
        System.out.printf("Drawing a circle with radius %.1f%n", radius);
        if (filled) {
            System.out.println("Filling circle with " + color + " color");
        } else {
            System.out.println("Drawing circle outline in " + color);
        }
    }
    
    public double getRadius() { return radius; }
}

// Rectangle implementation
class Rectangle extends Shape {
    protected double length;
    protected double width;
    
    public Rectangle(String color, boolean filled, double length, double width) {
        super(color, filled);
        this.length = length;
        this.width = width;
    }
    
    @Override
    public double getArea() {
        return length * width;
    }
    
    @Override
    public double getPerimeter() {
        return 2 * (length + width);
    }
    
    @Override
    public void draw() {
        System.out.printf("Drawing a rectangle %.1f x %.1f%n", length, width);
        if (filled) {
            System.out.println("Filling rectangle with " + color + " color");
        } else {
            System.out.println("Drawing rectangle outline in " + color);
        }
    }
    
    public double getLength() { return length; }
    public double getWidth() { return width; }
}

// Square - special case of Rectangle
class Square extends Rectangle {
    public Square(String color, boolean filled, double side) {
        super(color, filled, side, side);
    }
    
    @Override
    public void draw() {
        System.out.printf("Drawing a square with side %.1f%n", length);
        if (filled) {
            System.out.println("Filling square with " + color + " color");
        } else {
            System.out.println("Drawing square outline in " + color);
        }
    }
    
    public double getSide() { return length; }
}

// Triangle implementation
class Triangle extends Shape {
    private double side1, side2, side3;
    
    public Triangle(String color, boolean filled, double side1, double side2, double side3) {
        super(color, filled);
        this.side1 = side1;
        this.side2 = side2;
        this.side3 = side3;
    }
    
    @Override
    public double getArea() {
        // Using Heron's formula
        double s = getPerimeter() / 2;
        return Math.sqrt(s * (s - side1) * (s - side2) * (s - side3));
    }
    
    @Override
    public double getPerimeter() {
        return side1 + side2 + side3;
    }
    
    @Override
    public void draw() {
        System.out.printf("Drawing a triangle with sides %.1f, %.1f, %.1f%n", 
                side1, side2, side3);
        if (filled) {
            System.out.println("Filling triangle with " + color + " color");
        } else {
            System.out.println("Drawing triangle outline in " + color);
        }
    }
}

// Graphics engine that uses polymorphism
class GraphicsEngine {
    
    // Polymorphic method - works with any Shape
    public void renderShape(Shape shape) {
        shape.render(); // Polymorphic call
    }
    
    // Process array of shapes polymorphically
    public void renderScene(Shape[] shapes) {
        System.out.println("Rendering scene with " + shapes.length + " shapes:");
        double totalArea = 0;
        
        for (Shape shape : shapes) {
            renderShape(shape);           // Polymorphic rendering
            totalArea += shape.getArea(); // Polymorphic area calculation
            System.out.println();
        }
        
        System.out.printf("Total scene area: %.2f%n", totalArea);
    }
    
    // Method demonstrating instanceof and casting
    public void analyzeShapes(Shape[] shapes) {
        System.out.println("=== Shape Analysis ===");
        
        for (Shape shape : shapes) {
            System.out.println("Shape type: " + shape.getClass().getSimpleName());
            
            // Runtime type checking and casting
            if (shape instanceof Circle) {
                Circle circle = (Circle) shape;
                System.out.println("Circle radius: " + circle.getRadius());
            } else if (shape instanceof Square) {
                Square square = (Square) shape;
                System.out.println("Square side: " + square.getSide());
            } else if (shape instanceof Rectangle) {
                Rectangle rect = (Rectangle) shape;
                System.out.printf("Rectangle dimensions: %.1f x %.1f%n", 
                        rect.getLength(), rect.getWidth());
            } else if (shape instanceof Triangle) {
                System.out.println("Triangle perimeter: " + shape.getPerimeter());
            }
            
            System.out.println("---");
        }
    }
}

// Usage demonstration
public class AdvancedPolymorphismDemo {
    public static void main(String[] args) {
        // Create different shapes
        Shape[] shapes = {
            new Circle("Red", true, 5.0),
            new Rectangle("Blue", false, 4.0, 6.0),
            new Square("Green", true, 3.0),
            new Triangle("Yellow", false, 3.0, 4.0, 5.0)
        };
        
        GraphicsEngine engine = new GraphicsEngine();
        
        // Polymorphic rendering
        System.out.println("=== Individual Shape Rendering ===");
        for (Shape shape : shapes) {
            engine.renderShape(shape);
            System.out.println();
        }
        
        // Scene rendering
        System.out.println("=== Scene Rendering ===");
        engine.renderScene(shapes);
        
        // Shape analysis
        System.out.println("\n=== Shape Analysis ===");
        engine.analyzeShapes(shapes);
    }
}
```

### Example 4: Interface-Based Polymorphism

```java
// Interface defining common contract
interface Drawable {
    void draw();
    void resize(double factor);
}

interface Movable {
    void move(int x, int y);
    void getPosition();
}

// Multiple interface implementation
class Button implements Drawable, Movable {
    private String text;
    private int x, y;
    private int width, height;
    
    public Button(String text, int x, int y, int width, int height) {
        this.text = text;
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
    }
    
    @Override
    public void draw() {
        System.out.printf("Drawing button '%s' at (%d,%d) size %dx%d%n", 
                text, x, y, width, height);
    }
    
    @Override
    public void resize(double factor) {
        width = (int)(width * factor);
        height = (int)(height * factor);
        System.out.printf("Button resized to %dx%d%n", width, height);
    }
    
    @Override
    public void move(int x, int y) {
        this.x = x;
        this.y = y;
        System.out.printf("Button moved to (%d,%d)%n", x, y);
    }
    
    @Override
    public void getPosition() {
        System.out.printf("Button position: (%d,%d)%n", x, y);
    }
}

class Image implements Drawable, Movable {
    private String filename;
    private int x, y;
    private int width, height;
    
    public Image(String filename, int x, int y, int width, int height) {
        this.filename = filename;
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
    }
    
    @Override
    public void draw() {
        System.out.printf("Drawing image '%s' at (%d,%d) size %dx%d%n", 
                filename, x, y, width, height);
    }
    
    @Override
    public void resize(double factor) {
        width = (int)(width * factor);
        height = (int)(height * factor);
        System.out.printf("Image resized to %dx%d%n", width, height);
    }
    
    @Override
    public void move(int x, int y) {
        this.x = x;
        this.y = y;
        System.out.printf("Image moved to (%d,%d)%n", x, y);
    }
    
    @Override
    public void getPosition() {
        System.out.printf("Image position: (%d,%d)%n", x, y);
    }
}

// GUI Framework using interface polymorphism
class GUIFramework {
    
    // Works with any Drawable object
    public void renderComponent(Drawable component) {
        component.draw();
    }
    
    // Works with any Movable object
    public void repositionComponent(Movable component, int x, int y) {
        component.move(x, y);
        component.getPosition();
    }
    
    // Works with objects implementing both interfaces
    public void transformComponent(Object component, int x, int y, double scaleFactor) {
        if (component instanceof Movable) {
            ((Movable) component).move(x, y);
        }
        
        if (component instanceof Drawable) {
            ((Drawable) component).resize(scaleFactor);
            ((Drawable) component).draw();
        }
    }
}
```

## ⚠️ Common Mistakes to Avoid

### 1. Confusing Overloading with Overriding
```java
// ❌ This is overloading, not overriding!
class Parent {
    public void method(int x) { }
}

class Child extends Parent {
    public void method(double x) { } // Different parameter type = overloading
}

// ✅ Correct overriding
class Child extends Parent {
    @Override
    public void method(int x) { } // Same signature = overriding
}
```

### 2. Incorrect Method Overriding Rules
```java
// ❌ Violating overriding rules
class Parent {
    public Number method() { return 1; }
}

class Child extends Parent {
    private Integer method() { return 1; } // ❌ Can't reduce visibility
    public String method() { return ""; }  // ❌ Incompatible return type
}

// ✅ Correct overriding
class Child extends Parent {
    @Override
    public Integer method() { return 1; } // ✅ Covariant return type
}
```

### 3. Not Using @Override Annotation
```java
// ❌ Typo in method name - won't override
class Child extends Parent {
    public void metod() { } // Should be "method"
}

// ✅ Using @Override catches errors
class Child extends Parent {
    @Override
    public void metod() { } // ✅ Compilation error - helps catch typos
}
```

## 🎯 Benefits of Polymorphism

### 1. **Code Flexibility**
```java
// Same code works with different object types
public void processPayment(Payment payment) {
    payment.processPayment(); // Works with any Payment type
}
```

### 2. **Extensibility**
```java
// Add new payment types without changing existing code
class CryptocurrencyPayment extends Payment {
    // New implementation
}
// Existing code automatically works with new type
```

### 3. **Maintainability**
```java
// Changes to implementation don't affect client code
// Client uses interface, not implementation details
```

## 🔥 Interview Questions & Answers

### Q1: What is polymorphism and what are its types?
**Answer**: Polymorphism means "many forms" - same interface, different implementations. Types:
- **Compile-time (Static)**: Method overloading, resolved at compile time
- **Runtime (Dynamic)**: Method overriding, resolved at runtime using virtual method table
- **Parametric**: Generics (compile-time type safety with runtime type erasure)

### Q2: How does method overriding enable runtime polymorphism?
**Answer**: When a child class overrides a parent method:
1. **Virtual Method Table**: JVM maintains table of method addresses for each class
2. **Dynamic Binding**: At runtime, JVM looks up actual object type to call correct method
3. **Late Binding**: Method resolution happens during execution, not compilation
4. **Polymorphic Behavior**: Same method call produces different behaviors based on object type

### Q3: What's the difference between method overloading and overriding?
**Answer**: 
| Aspect | Overloading | Overriding |
|--------|-------------|------------|
| **Definition** | Same method name, different parameters | Same method signature in parent/child |
| **Binding** | Compile-time (static) | Runtime (dynamic) |
| **Inheritance** | Same class | Requires inheritance |
| **Polymorphism** | Compile-time polymorphism | Runtime polymorphism |
| **Annotation** | No special annotation | @Override recommended |

### Q4: Can you override static methods? Why or why not?
**Answer**: **No**, static methods cannot be overridden because:
- **Static methods belong to class, not instance**
- **No dynamic binding for static methods** - resolved at compile time
- **Method hiding occurs instead** - child class method hides parent's static method
- **No virtual method table entry** for static methods
- If you declare same static method in child, it's method hiding, not overriding

### Q5: What are the rules for method overriding in Java?
**Answer**: 
1. **Same method signature** (name, parameters, return type - covariant allowed)
2. **Cannot reduce access level** (private < default < protected < public)
3. **Cannot override final methods**
4. **Cannot override static methods** (method hiding occurs)
5. **Can throw same or narrower checked exceptions**
6. **Use @Override annotation** for compile-time checking
7. **Must have IS-A relationship** between classes

---
[← Back to Main Guide](./README.md) | [← Previous: Inheritance](./inheritance.md) | [Next: Abstraction →](./abstraction.md)
