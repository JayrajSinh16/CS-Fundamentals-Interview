# 🚨 Exception Hierarchy in Java - Understanding Error Types

Exception hierarchy in Java is a well-structured tree of classes that represents different types of errors and exceptional conditions. Understanding this hierarchy is crucial for effective error handling, debugging, and writing robust Java applications.

## 🎯 Key Concepts

- **Throwable Root**: All exceptions inherit from Throwable class
- **Checked vs Unchecked**: Compile-time vs runtime exception handling
- **Error vs Exception**: System errors vs application exceptions
- **Exception Propagation**: How exceptions flow up the call stack
- **Custom Exceptions**: Creating domain-specific exception types

## 🌍 Real-World Analogy

**Hospital Emergency System Analogy**:
- **Throwable** = Any medical emergency requiring attention
- **Error** = System failures (power outage, equipment failure) - hospital can't handle
- **Exception** = Patient conditions (treatable emergencies)
- **Checked Exception** = Scheduled surgeries (must be planned and handled)
- **Unchecked Exception** = Sudden accidents (happen unexpectedly)
- **RuntimeException** = Emergency room cases (immediate attention needed)
- **Custom Exceptions** = Specialized department protocols

## 🏗️ Java Exception Hierarchy

```
                           java.lang.Object
                                  │
                           java.lang.Throwable
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
            java.lang.Error              java.lang.Exception
                    │                           │
        ┌───────────┼───────────┐              │
        │           │           │              │
   OutOfMemory  StackOverflow VirtualMachine  │
    Error         Error        Error          │
                                              │
                                ┌─────────────┼─────────────┐
                                │             │             │
                        RuntimeException   IOException  SQLException
                                │             │             │
                    ┌───────────┼──────┐     │             │
                    │           │      │     │             │
            NullPointer  ArrayIndex ClassCast FileNotFound  DataAccess
            Exception    Exception  Exception   Exception    Exception
                                              │
                                              │
                               ┌──────────────┼──────────────┐
                               │              │              │
                         EOFException  InterruptedIO  MalformedURL
                                         Exception      Exception

Exception Categories:
┌─────────────────┬─────────────────┬─────────────────┬─────────────────┐
│   Category      │   Must Handle   │   Timing        │   Examples      │
├─────────────────┼─────────────────┼─────────────────┼─────────────────┤
│ Error           │ Should not      │ Runtime         │ OutOfMemoryError│
│ Checked         │ Must handle     │ Compile time    │ IOException     │
│ RuntimeException│ Optional        │ Runtime         │ NullPointer     │
│ Custom Checked  │ Must handle     │ Compile time    │ BusinessLogic   │
│ Custom Unchecked│ Optional        │ Runtime         │ Validation      │
└─────────────────┴─────────────────┴─────────────────┴─────────────────┘

Exception Flow:
┌─────────────────────────────────────────────────────────────────────┐
│                     Exception Propagation                          │
├─────────────────────────────────────────────────────────────────────┤
│  Method A() calls Method B() calls Method C()                      │
│     │              │              │                               │
│     │              │              └─ Exception thrown             │
│     │              └─ Exception propagates up (if not caught)     │
│     └─ Exception handled or propagates further                    │
│                                                                     │
│  Call Stack:                                                       │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐           │
│  │  Method A   │ ← ─│  Method B   │ ← ─│  Method C   │           │
│  │ (handles)   │    │ (propagates)│    │ (throws)    │           │
│  └─────────────┘    └─────────────┘    └─────────────┘           │
└─────────────────────────────────────────────────────────────────────┘
```

## 💻 Practical Examples

### Example 1: Understanding Exception Hierarchy Classes

```java
// ExceptionHierarchyDemo.java - Understanding different exception types
public class ExceptionHierarchyDemo {
    
    // Custom checked exception
    static class BusinessLogicException extends Exception {
        private final String errorCode;
        private final Object[] parameters;
        
        public BusinessLogicException(String message) {
            super(message);
            this.errorCode = "BUSINESS_ERROR";
            this.parameters = new Object[0];
        }
        
        public BusinessLogicException(String message, String errorCode) {
            super(message);
            this.errorCode = errorCode;
            this.parameters = new Object[0];
        }
        
        public BusinessLogicException(String message, Throwable cause) {
            super(message, cause);
            this.errorCode = "BUSINESS_ERROR";
            this.parameters = new Object[0];
        }
        
        public BusinessLogicException(String message, String errorCode, Object... parameters) {
            super(message);
            this.errorCode = errorCode;
            this.parameters = parameters.clone();
        }
        
        public String getErrorCode() { return errorCode; }
        public Object[] getParameters() { return parameters.clone(); }
        
        @Override
        public String toString() {
            return String.format("BusinessLogicException{errorCode='%s', message='%s', parameters=%s}", 
                    errorCode, getMessage(), java.util.Arrays.toString(parameters));
        }
    }
    
    // Custom unchecked exception
    static class ValidationException extends RuntimeException {
        private final String fieldName;
        private final Object invalidValue;
        
        public ValidationException(String fieldName, Object invalidValue) {
            super(String.format("Invalid value for field '%s': %s", fieldName, invalidValue));
            this.fieldName = fieldName;
            this.invalidValue = invalidValue;
        }
        
        public ValidationException(String fieldName, Object invalidValue, String customMessage) {
            super(customMessage);
            this.fieldName = fieldName;
            this.invalidValue = invalidValue;
        }
        
        public String getFieldName() { return fieldName; }
        public Object getInvalidValue() { return invalidValue; }
        
        @Override
        public String toString() {
            return String.format("ValidationException{field='%s', value='%s', message='%s'}", 
                    fieldName, invalidValue, getMessage());
        }
    }
    
    // Class demonstrating various exception scenarios
    static class ExceptionExamples {
        
        // Method demonstrating checked exceptions
        public static void demonstrateCheckedException() throws BusinessLogicException {
            System.out.println("=== Checked Exception Demonstration ===");
            
            try {
                // Simulate business logic that might fail
                processBusinessLogic("INVALID_DATA");
            } catch (BusinessLogicException e) {
                System.out.printf("Caught business exception: %s%n", e);
                // Re-throw with additional context
                throw new BusinessLogicException("Failed to process data", e);
            }
        }
        
        private static void processBusinessLogic(String data) throws BusinessLogicException {
            if ("INVALID_DATA".equals(data)) {
                throw new BusinessLogicException(
                    "Business rule violation: Invalid data format", 
                    "BL001", 
                    data, 
                    new java.util.Date()
                );
            }
            System.out.println("Business logic processed successfully");
        }
        
        // Method demonstrating unchecked exceptions
        public static void demonstrateUncheckedException() {
            System.out.println("\n=== Unchecked Exception Demonstration ===");
            
            try {
                validateUserInput(null);
                validateUserInput("");
                validateUserInput("valid@email.com");
            } catch (ValidationException e) {
                System.out.printf("Validation failed: %s%n", e);
            } catch (NullPointerException e) {
                System.out.printf("Null pointer error: %s%n", e.getMessage());
            }
        }
        
        private static void validateUserInput(String email) {
            if (email == null) {
                throw new NullPointerException("Email cannot be null");
            }
            if (email.trim().isEmpty()) {
                throw new ValidationException("email", email, "Email cannot be empty");
            }
            if (!email.contains("@")) {
                throw new ValidationException("email", email, "Invalid email format");
            }
            System.out.printf("Valid email: %s%n", email);
        }
        
        // Method demonstrating multiple exception types
        public static void demonstrateMultipleExceptions() {
            System.out.println("\n=== Multiple Exception Types ===");
            
            String[] testCases = {null, "", "invalid", "valid@test.com"};
            
            for (String testCase : testCases) {
                try {
                    System.out.printf("Testing: %s%n", testCase);
                    processEmailWithBusinessLogic(testCase);
                    System.out.println("Processing successful");
                } catch (ValidationException e) {
                    System.out.printf("Validation error: %s%n", e.getMessage());
                } catch (BusinessLogicException e) {
                    System.out.printf("Business logic error: %s%n", e.getMessage());
                } catch (RuntimeException e) {
                    System.out.printf("Runtime error: %s%n", e.getMessage());
                } catch (Exception e) {
                    System.out.printf("General error: %s%n", e.getMessage());
                }
                System.out.println();
            }
        }
        
        private static void processEmailWithBusinessLogic(String email) 
                throws BusinessLogicException {
            
            // This might throw ValidationException (unchecked)
            validateUserInput(email);
            
            // Business logic that might throw checked exception
            if ("invalid".equals(email)) {
                throw new BusinessLogicException(
                    "Email is in blacklist", 
                    "BL002", 
                    email
                );
            }
        }
        
        // Method demonstrating Error handling (generally not recommended)
        public static void demonstrateErrorHandling() {
            System.out.println("\n=== Error Handling (Not Recommended) ===");
            
            try {
                // Simulate OutOfMemoryError scenario
                simulateOutOfMemoryError();
            } catch (OutOfMemoryError e) {
                System.out.printf("Caught OutOfMemoryError: %s%n", e.getMessage());
                System.out.println("Note: Catching Error is generally not recommended");
            }
            
            try {
                // Simulate StackOverflowError
                simulateStackOverflowError(0);
            } catch (StackOverflowError e) {
                System.out.printf("Caught StackOverflowError: %s%n", e.getMessage());
                System.out.println("Note: This should be prevented, not caught");
            }
        }
        
        private static void simulateOutOfMemoryError() {
            // Simulate memory allocation (careful - this could actually cause OOM)
            System.out.println("Simulating OutOfMemoryError scenario");
            // In reality, you wouldn't intentionally cause this
            throw new OutOfMemoryError("Simulated OOM for demonstration");
        }
        
        private static void simulateStackOverflowError(int depth) {
            if (depth > 1000) { // Prevent actual stack overflow in demo
                throw new StackOverflowError("Simulated stack overflow");
            }
            simulateStackOverflowError(depth + 1); // Recursive call
        }
    }
    
    // Exception hierarchy analysis utility
    static class ExceptionAnalyzer {
        
        public static void analyzeException(Throwable throwable) {
            System.out.printf("\n=== Exception Analysis ===\n");
            System.out.printf("Exception Type: %s\n", throwable.getClass().getSimpleName());
            System.out.printf("Full Class Name: %s\n", throwable.getClass().getName());
            System.out.printf("Message: %s\n", throwable.getMessage());
            
            // Check exception category
            if (throwable instanceof Error) {
                System.out.println("Category: Error (System-level problem)");
            } else if (throwable instanceof RuntimeException) {
                System.out.println("Category: Unchecked Exception (Runtime)");
            } else if (throwable instanceof Exception) {
                System.out.println("Category: Checked Exception (Compile-time)");
            }
            
            // Print inheritance hierarchy
            System.out.println("Inheritance Hierarchy:");
            Class<?> currentClass = throwable.getClass();
            int level = 0;
            while (currentClass != null && currentClass != Object.class) {
                System.out.printf("%s%s\n", "  ".repeat(level), currentClass.getSimpleName());
                currentClass = currentClass.getSuperclass();
                level++;
            }
            
            // Print stack trace (first few lines)
            System.out.println("Stack Trace Preview:");
            StackTraceElement[] stackTrace = throwable.getStackTrace();
            for (int i = 0; i < Math.min(3, stackTrace.length); i++) {
                System.out.printf("  at %s\n", stackTrace[i]);
            }
            
            // Check for cause
            if (throwable.getCause() != null) {
                System.out.printf("Caused by: %s\n", throwable.getCause().getClass().getSimpleName());
            }
        }
        
        public static void demonstrateExceptionTypes() {
            System.out.println("=== Exception Type Examples ===\n");
            
            Throwable[] exceptions = {
                new NullPointerException("Null reference"),
                new IllegalArgumentException("Invalid argument"),
                new java.io.IOException("IO operation failed"),
                new java.sql.SQLException("Database error"),
                new BusinessLogicException("Business rule violation"),
                new ValidationException("email", null),
                new OutOfMemoryError("Memory exhausted"),
                new StackOverflowError("Stack limit exceeded")
            };
            
            for (Throwable exception : exceptions) {
                analyzeException(exception);
                System.out.println("-".repeat(50));
            }
        }
    }
    
    // Exception chaining demonstration
    static class ExceptionChaining {
        
        public static void demonstrateExceptionChaining() {
            System.out.println("\n=== Exception Chaining ===\n");
            
            try {
                methodA();
            } catch (BusinessLogicException e) {
                System.out.println("Final exception caught:");
                ExceptionAnalyzer.analyzeException(e);
                
                // Print full exception chain
                System.out.println("\nException Chain:");
                Throwable current = e;
                int level = 0;
                while (current != null) {
                    System.out.printf("%s%d. %s: %s\n", 
                            "  ".repeat(level), 
                            level + 1,
                            current.getClass().getSimpleName(), 
                            current.getMessage());
                    current = current.getCause();
                    level++;
                }
            }
        }
        
        private static void methodA() throws BusinessLogicException {
            try {
                methodB();
            } catch (Exception e) {
                throw new BusinessLogicException("Method A failed", e);
            }
        }
        
        private static void methodB() throws Exception {
            try {
                methodC();
            } catch (RuntimeException e) {
                throw new Exception("Method B failed", e);
            }
        }
        
        private static void methodC() {
            throw new IllegalStateException("Method C encountered invalid state");
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Exception Hierarchy Demonstration ===\n");
        
        // Demonstrate different exception types
        try {
            ExceptionExamples.demonstrateCheckedException();
        } catch (BusinessLogicException e) {
            System.out.printf("Main caught: %s%n", e);
        }
        
        ExceptionExamples.demonstrateUncheckedException();
        ExceptionExamples.demonstrateMultipleExceptions();
        ExceptionExamples.demonstrateErrorHandling();
        
        // Analyze exception types
        ExceptionAnalyzer.demonstrateExceptionTypes();
        
        // Demonstrate exception chaining
        ExceptionChaining.demonstrateExceptionChaining();
        
        // Summary
        System.out.println("\n=== Exception Hierarchy Summary ===");
        System.out.println("✅ Throwable is the root of all exceptions");
        System.out.println("✅ Error represents system-level problems");
        System.out.println("✅ Exception represents application-level problems");
        System.out.println("✅ Checked exceptions must be handled at compile time");
        System.out.println("✅ RuntimeExceptions are unchecked (optional handling)");
        System.out.println("✅ Custom exceptions should extend appropriate base classes");
        System.out.println("✅ Exception chaining preserves root cause information");
    }
}

/*
Exception Hierarchy Key Points:

1. Throwable Hierarchy:
   - All exceptions extend Throwable
   - Error: System problems (don't catch)
   - Exception: Application problems (handle appropriately)

2. Checked vs Unchecked:
   - Checked: Must handle or declare (IOException, SQLException)
   - Unchecked: Optional handling (RuntimeException subclasses)

3. Common Exception Types:
   - NullPointerException: Null reference access
   - IllegalArgumentException: Invalid method arguments
   - IllegalStateException: Object in invalid state
   - IOException: Input/output operations
   - SQLException: Database operations

4. Best Practices:
   - Catch specific exceptions, not generic Exception
   - Use exception chaining to preserve root causes
   - Create meaningful custom exceptions
   - Don't catch Error types
   - Handle exceptions at appropriate levels
*/
```

### Example 2: Common Java Exceptions Deep Dive

```java
// CommonExceptionsDemo.java - Detailed exploration of common Java exceptions
public class CommonExceptionsDemo {
    
    // Demonstration class for RuntimeExceptions
    static class RuntimeExceptionExamples {
        
        // NullPointerException scenarios
        public static void demonstrateNullPointerException() {
            System.out.println("=== NullPointerException Examples ===");
            
            // Scenario 1: Calling method on null reference
            try {
                String str = null;
                int length = str.length(); // NPE here
            } catch (NullPointerException e) {
                System.out.printf("NPE - Method call on null: %s%n", e.getMessage());
            }
            
            // Scenario 2: Array access on null reference
            try {
                int[] array = null;
                int value = array[0]; // NPE here
            } catch (NullPointerException e) {
                System.out.printf("NPE - Array access on null: %s%n", e.getMessage());
            }
            
            // Scenario 3: Field access on null reference
            try {
                Person person = null;
                String name = person.name; // NPE here
            } catch (NullPointerException e) {
                System.out.printf("NPE - Field access on null: %s%n", e.getMessage());
            }
            
            // Prevention strategy
            System.out.println("Prevention strategies:");
            String str = getValue();
            if (str != null) {
                System.out.printf("Safe access: length = %d%n", str.length());
            }
            
            // Using Optional (Java 8+)
            java.util.Optional<String> optional = java.util.Optional.ofNullable(getValue());
            optional.ifPresent(s -> System.out.printf("Optional access: length = %d%n", s.length()));
        }
        
        private static String getValue() {
            return Math.random() > 0.5 ? "Hello" : null;
        }
        
        // ArrayIndexOutOfBoundsException scenarios
        public static void demonstrateArrayIndexOutOfBoundsException() {
            System.out.println("\n=== ArrayIndexOutOfBoundsException Examples ===");
            
            int[] array = {1, 2, 3, 4, 5};
            
            // Scenario 1: Negative index
            try {
                int value = array[-1];
            } catch (ArrayIndexOutOfBoundsException e) {
                System.out.printf("Negative index: %s%n", e.getMessage());
            }
            
            // Scenario 2: Index >= array length
            try {
                int value = array[10];
            } catch (ArrayIndexOutOfBoundsException e) {
                System.out.printf("Index too large: %s%n", e.getMessage());
            }
            
            // Prevention strategy
            int index = 3;
            if (index >= 0 && index < array.length) {
                System.out.printf("Safe access: array[%d] = %d%n", index, array[index]);
            }
        }
        
        // ClassCastException scenarios
        public static void demonstrateClassCastException() {
            System.out.println("\n=== ClassCastException Examples ===");
            
            Object obj = "Hello World";
            
            // Scenario 1: Invalid cast
            try {
                Integer number = (Integer) obj; // String cannot be cast to Integer
            } catch (ClassCastException e) {
                System.out.printf("Invalid cast: %s%n", e.getMessage());
            }
            
            // Prevention strategy using instanceof
            if (obj instanceof Integer) {
                Integer number = (Integer) obj;
                System.out.printf("Safe cast: %d%n", number);
            } else {
                System.out.printf("Object is %s, not Integer%n", obj.getClass().getSimpleName());
            }
            
            // Generics help prevent ClassCastException
            java.util.List<String> stringList = new java.util.ArrayList<>();
            stringList.add("Hello");
            String str = stringList.get(0); // No cast needed, type-safe
            System.out.printf("Generic type safety: %s%n", str);
        }
        
        // IllegalArgumentException scenarios
        public static void demonstrateIllegalArgumentException() {
            System.out.println("\n=== IllegalArgumentException Examples ===");
            
            try {
                calculateSquareRoot(-5);
            } catch (IllegalArgumentException e) {
                System.out.printf("Invalid argument: %s%n", e.getMessage());
            }
            
            try {
                createPerson("", 25);
            } catch (IllegalArgumentException e) {
                System.out.printf("Invalid person data: %s%n", e.getMessage());
            }
            
            // Valid usage
            double result = calculateSquareRoot(25);
            System.out.printf("Valid calculation: sqrt(25) = %.2f%n", result);
        }
        
        private static double calculateSquareRoot(double number) {
            if (number < 0) {
                throw new IllegalArgumentException("Cannot calculate square root of negative number: " + number);
            }
            return Math.sqrt(number);
        }
        
        private static Person createPerson(String name, int age) {
            if (name == null || name.trim().isEmpty()) {
                throw new IllegalArgumentException("Name cannot be null or empty");
            }
            if (age < 0 || age > 150) {
                throw new IllegalArgumentException("Age must be between 0 and 150: " + age);
            }
            return new Person(name, age);
        }
        
        // IllegalStateException scenarios
        public static void demonstrateIllegalStateException() {
            System.out.println("\n=== IllegalStateException Examples ===");
            
            BankAccount account = new BankAccount(100.0);
            
            try {
                account.withdraw(150.0); // Insufficient funds
            } catch (IllegalStateException e) {
                System.out.printf("Invalid operation: %s%n", e.getMessage());
            }
            
            account.close();
            try {
                account.deposit(50.0); // Account is closed
            } catch (IllegalStateException e) {
                System.out.printf("Account closed: %s%n", e.getMessage());
            }
        }
    }
    
    // Demonstration class for Checked Exceptions
    static class CheckedExceptionExamples {
        
        // IOException scenarios
        public static void demonstrateIOException() {
            System.out.println("\n=== IOException Examples ===");
            
            // File reading with potential IOException
            try {
                readFile("nonexistent.txt");
            } catch (java.io.IOException e) {
                System.out.printf("IO Error: %s%n", e.getMessage());
            }
            
            // Network operation with potential IOException
            try {
                downloadFromUrl("http://invalid-url-example.com");
            } catch (java.io.IOException e) {
                System.out.printf("Network Error: %s%n", e.getMessage());
            }
        }
        
        private static String readFile(String filename) throws java.io.IOException {
            // Simulate file reading
            if (!filename.equals("valid.txt")) {
                throw new java.io.FileNotFoundException("File not found: " + filename);
            }
            return "File content";
        }
        
        private static String downloadFromUrl(String url) throws java.io.IOException {
            // Simulate network operation
            if (url.contains("invalid")) {
                throw new java.io.IOException("Unable to connect to: " + url);
            }
            return "Downloaded content";
        }
        
        // SQLException scenarios
        public static void demonstrateSQLException() {
            System.out.println("\n=== SQLException Examples ===");
            
            try {
                executeQuery("SELECT * FROM nonexistent_table");
            } catch (java.sql.SQLException e) {
                System.out.printf("Database Error: %s (Error Code: %d)%n", 
                        e.getMessage(), e.getErrorCode());
            }
            
            try {
                connectToDatabase("invalid://database");
            } catch (java.sql.SQLException e) {
                System.out.printf("Connection Error: %s%n", e.getMessage());
            }
        }
        
        private static void executeQuery(String sql) throws java.sql.SQLException {
            // Simulate database query
            if (sql.contains("nonexistent")) {
                throw new java.sql.SQLException("Table doesn't exist", "42S02", 1146);
            }
        }
        
        private static void connectToDatabase(String url) throws java.sql.SQLException {
            // Simulate database connection
            if (url.startsWith("invalid")) {
                throw new java.sql.SQLException("Invalid database URL: " + url);
            }
        }
        
        // ParseException scenarios
        public static void demonstrateParseException() {
            System.out.println("\n=== ParseException Examples ===");
            
            try {
                parseDate("invalid-date");
            } catch (java.text.ParseException e) {
                System.out.printf("Parse Error: %s at position %d%n", 
                        e.getMessage(), e.getErrorOffset());
            }
            
            try {
                parseNumber("not-a-number");
            } catch (java.text.ParseException e) {
                System.out.printf("Number Parse Error: %s%n", e.getMessage());
            }
            
            // Valid parsing
            try {
                java.util.Date date = parseDate("2023-12-25");
                System.out.printf("Parsed date: %s%n", date);
            } catch (java.text.ParseException e) {
                System.out.printf("Unexpected error: %s%n", e.getMessage());
            }
        }
        
        private static java.util.Date parseDate(String dateStr) throws java.text.ParseException {
            java.text.SimpleDateFormat format = new java.text.SimpleDateFormat("yyyy-MM-dd");
            return format.parse(dateStr);
        }
        
        private static Number parseNumber(String numberStr) throws java.text.ParseException {
            java.text.NumberFormat format = java.text.NumberFormat.getInstance();
            return format.parse(numberStr);
        }
    }
    
    // Error examples (for educational purposes only)
    static class ErrorExamples {
        
        public static void demonstrateErrors() {
            System.out.println("\n=== Error Examples (Educational Only) ===");
            
            // OutOfMemoryError simulation
            try {
                simulateOutOfMemoryError();
            } catch (OutOfMemoryError e) {
                System.out.printf("OutOfMemoryError: %s%n", e.getMessage());
            }
            
            // StackOverflowError simulation
            try {
                simulateStackOverflowError(0);
            } catch (StackOverflowError e) {
                System.out.printf("StackOverflowError: %s%n", e.getMessage());
            }
            
            // NoClassDefFoundError simulation
            try {
                simulateNoClassDefFoundError();
            } catch (NoClassDefFoundError e) {
                System.out.printf("NoClassDefFoundError: %s%n", e.getMessage());
            }
        }
        
        private static void simulateOutOfMemoryError() {
            // Don't actually allocate huge memory in demo
            System.out.println("Simulating OutOfMemoryError...");
            throw new OutOfMemoryError("Java heap space (simulated)");
        }
        
        private static void simulateStackOverflowError(int depth) {
            if (depth > 100) { // Prevent actual stack overflow
                throw new StackOverflowError("Stack overflow (simulated)");
            }
            simulateStackOverflowError(depth + 1);
        }
        
        private static void simulateNoClassDefFoundError() {
            throw new NoClassDefFoundError("com/example/MissingClass (simulated)");
        }
    }
    
    // Supporting classes
    static class Person {
        String name;
        int age;
        
        Person(String name, int age) {
            this.name = name;
            this.age = age;
        }
        
        @Override
        public String toString() {
            return String.format("Person{name='%s', age=%d}", name, age);
        }
    }
    
    static class BankAccount {
        private double balance;
        private boolean closed = false;
        
        BankAccount(double balance) {
            this.balance = balance;
        }
        
        void deposit(double amount) {
            if (closed) {
                throw new IllegalStateException("Cannot deposit to closed account");
            }
            if (amount <= 0) {
                throw new IllegalArgumentException("Deposit amount must be positive");
            }
            balance += amount;
        }
        
        void withdraw(double amount) {
            if (closed) {
                throw new IllegalStateException("Cannot withdraw from closed account");
            }
            if (amount <= 0) {
                throw new IllegalArgumentException("Withdrawal amount must be positive");
            }
            if (amount > balance) {
                throw new IllegalStateException(
                    String.format("Insufficient funds: balance=%.2f, requested=%.2f", 
                    balance, amount));
            }
            balance -= amount;
        }
        
        void close() {
            closed = true;
        }
        
        double getBalance() {
            return balance;
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Common Java Exceptions Deep Dive ===\n");
        
        // RuntimeException examples
        RuntimeExceptionExamples.demonstrateNullPointerException();
        RuntimeExceptionExamples.demonstrateArrayIndexOutOfBoundsException();
        RuntimeExceptionExamples.demonstrateClassCastException();
        RuntimeExceptionExamples.demonstrateIllegalArgumentException();
        RuntimeExceptionExamples.demonstrateIllegalStateException();
        
        // Checked exception examples
        CheckedExceptionExamples.demonstrateIOException();
        CheckedExceptionExamples.demonstrateSQLException();
        CheckedExceptionExamples.demonstrateParseException();
        
        // Error examples (educational only)
        ErrorExamples.demonstrateErrors();
        
        System.out.println("\n=== Exception Prevention Summary ===");
        System.out.println("✅ Always check for null before dereferencing");
        System.out.println("✅ Validate array indices before access");
        System.out.println("✅ Use instanceof before casting");
        System.out.println("✅ Validate method arguments");
        System.out.println("✅ Check object state before operations");
        System.out.println("✅ Handle checked exceptions appropriately");
        System.out.println("✅ Use defensive programming techniques");
    }
}

/*
Common Exception Categories:

1. NullPointerException:
   - Most common runtime exception
   - Caused by null reference access
   - Prevention: null checks, Optional, defensive programming

2. ArrayIndexOutOfBoundsException:
   - Array/List access with invalid index
   - Prevention: bounds checking, enhanced for loops

3. ClassCastException:
   - Invalid type casting
   - Prevention: instanceof checks, generics

4. IllegalArgumentException:
   - Invalid method arguments
   - Prevention: argument validation

5. IllegalStateException:
   - Object in invalid state for operation
   - Prevention: state validation

6. IOException:
   - Input/output operation failures
   - Must be handled (checked exception)

7. SQLException:
   - Database operation failures
   - Must be handled (checked exception)

8. ParseException:
   - String parsing failures
   - Must be handled (checked exception)

Best Practices:
- Catch specific exceptions, not generic Exception
- Validate inputs and state
- Use defensive programming
- Handle checked exceptions appropriately
- Don't catch Error types unless absolutely necessary
*/
```

### Example 3: Custom Exception Design Patterns

```java
// CustomExceptionPatterns.java - Best practices for custom exception design
public class CustomExceptionPatterns {
    
    // Base application exception
    abstract static class BaseApplicationException extends Exception {
        private final String errorCode;
        private final java.util.Map<String, Object> context;
        private final java.time.LocalDateTime timestamp;
        
        protected BaseApplicationException(String message, String errorCode) {
            super(message);
            this.errorCode = errorCode;
            this.context = new java.util.HashMap<>();
            this.timestamp = java.time.LocalDateTime.now();
        }
        
        protected BaseApplicationException(String message, String errorCode, Throwable cause) {
            super(message, cause);
            this.errorCode = errorCode;
            this.context = new java.util.HashMap<>();
            this.timestamp = java.time.LocalDateTime.now();
        }
        
        public String getErrorCode() { return errorCode; }
        public java.util.Map<String, Object> getContext() { return new java.util.HashMap<>(context); }
        public java.time.LocalDateTime getTimestamp() { return timestamp; }
        
        public BaseApplicationException addContext(String key, Object value) {
            context.put(key, value);
            return this;
        }
        
        public abstract String getCategory();
        
        @Override
        public String toString() {
            return String.format("%s{category='%s', errorCode='%s', message='%s', timestamp=%s, context=%s}", 
                    getClass().getSimpleName(), getCategory(), errorCode, getMessage(), timestamp, context);
        }
    }
    
    // Business logic exceptions
    static class BusinessRuleException extends BaseApplicationException {
        private final String ruleName;
        private final Object violatingValue;
        
        public BusinessRuleException(String ruleName, String message, String errorCode) {
            super(message, errorCode);
            this.ruleName = ruleName;
            this.violatingValue = null;
        }
        
        public BusinessRuleException(String ruleName, Object violatingValue, String message, String errorCode) {
            super(message, errorCode);
            this.ruleName = ruleName;
            this.violatingValue = violatingValue;
            addContext("ruleName", ruleName);
            addContext("violatingValue", violatingValue);
        }
        
        @Override
        public String getCategory() {
            return "BUSINESS_RULE";
        }
        
        public String getRuleName() { return ruleName; }
        public Object getViolatingValue() { return violatingValue; }
    }
    
    // Data access exceptions
    static class DataAccessException extends BaseApplicationException {
        private final String operation;
        private final String entityType;
        private final Object entityId;
        
        public DataAccessException(String operation, String entityType, Object entityId, 
                                 String message, String errorCode) {
            super(message, errorCode);
            this.operation = operation;
            this.entityType = entityType;
            this.entityId = entityId;
            addContext("operation", operation);
            addContext("entityType", entityType);
            addContext("entityId", entityId);
        }
        
        public DataAccessException(String operation, String entityType, Object entityId, 
                                 String message, String errorCode, Throwable cause) {
            super(message, errorCode, cause);
            this.operation = operation;
            this.entityType = entityType;
            this.entityId = entityId;
            addContext("operation", operation);
            addContext("entityType", entityType);
            addContext("entityId", entityId);
        }
        
        @Override
        public String getCategory() {
            return "DATA_ACCESS";
        }
        
        public String getOperation() { return operation; }
        public String getEntityType() { return entityType; }
        public Object getEntityId() { return entityId; }
    }
    
    // Validation exceptions (unchecked)
    static class ValidationException extends RuntimeException {
        private final java.util.List<ValidationError> errors;
        
        public ValidationException(String message) {
            super(message);
            this.errors = new java.util.ArrayList<>();
        }
        
        public ValidationException(java.util.List<ValidationError> errors) {
            super(createMessage(errors));
            this.errors = new java.util.ArrayList<>(errors);
        }
        
        public ValidationException(String field, Object value, String message) {
            this(java.util.Arrays.asList(new ValidationError(field, value, message)));
        }
        
        private static String createMessage(java.util.List<ValidationError> errors) {
            if (errors.isEmpty()) {
                return "Validation failed";
            }
            if (errors.size() == 1) {
                return "Validation failed: " + errors.get(0).getMessage();
            }
            return String.format("Validation failed with %d errors", errors.size());
        }
        
        public java.util.List<ValidationError> getErrors() {
            return new java.util.ArrayList<>(errors);
        }
        
        public boolean hasField(String field) {
            return errors.stream().anyMatch(error -> error.getField().equals(field));
        }
        
        @Override
        public String toString() {
            return String.format("ValidationException{message='%s', errors=%s}", 
                    getMessage(), errors);
        }
    }
    
    // Validation error data class
    static class ValidationError {
        private final String field;
        private final Object rejectedValue;
        private final String message;
        
        public ValidationError(String field, Object rejectedValue, String message) {
            this.field = field;
            this.rejectedValue = rejectedValue;
            this.message = message;
        }
        
        public String getField() { return field; }
        public Object getRejectedValue() { return rejectedValue; }
        public String getMessage() { return message; }
        
        @Override
        public String toString() {
            return String.format("ValidationError{field='%s', rejectedValue='%s', message='%s'}", 
                    field, rejectedValue, message);
        }
    }
    
    // Configuration exceptions
    static class ConfigurationException extends BaseApplicationException {
        private final String configKey;
        private final String configValue;
        private final String expectedFormat;
        
        public ConfigurationException(String configKey, String configValue, 
                                    String expectedFormat, String message, String errorCode) {
            super(message, errorCode);
            this.configKey = configKey;
            this.configValue = configValue;
            this.expectedFormat = expectedFormat;
            addContext("configKey", configKey);
            addContext("configValue", configValue);
            addContext("expectedFormat", expectedFormat);
        }
        
        @Override
        public String getCategory() {
            return "CONFIGURATION";
        }
        
        public String getConfigKey() { return configKey; }
        public String getConfigValue() { return configValue; }
        public String getExpectedFormat() { return expectedFormat; }
    }
    
    // Service layer that demonstrates exception usage
    static class UserService {
        private java.util.Map<String, User> users = new java.util.HashMap<>();
        
        public User createUser(String username, String email, int age) 
                throws BusinessRuleException, ValidationException {
            
            // Validation
            java.util.List<ValidationError> validationErrors = new java.util.ArrayList<>();
            
            if (username == null || username.trim().isEmpty()) {
                validationErrors.add(new ValidationError("username", username, "Username is required"));
            }
            if (email == null || !email.contains("@")) {
                validationErrors.add(new ValidationError("email", email, "Valid email is required"));
            }
            if (age < 0 || age > 150) {
                validationErrors.add(new ValidationError("age", age, "Age must be between 0 and 150"));
            }
            
            if (!validationErrors.isEmpty()) {
                throw new ValidationException(validationErrors);
            }
            
            // Business rules
            if (users.containsKey(username)) {
                throw new BusinessRuleException("unique_username", username, 
                    "Username already exists", "USR001");
            }
            
            if (age < 18) {
                throw new BusinessRuleException("minimum_age", age, 
                    "User must be at least 18 years old", "USR002");
            }
            
            // Create user
            User user = new User(username, email, age);
            users.put(username, user);
            return user;
        }
        
        public User getUser(String username) throws DataAccessException {
            if (username == null) {
                throw new DataAccessException("GET", "User", username, 
                    "Username cannot be null", "DAT001");
            }
            
            User user = users.get(username);
            if (user == null) {
                throw new DataAccessException("GET", "User", username, 
                    "User not found", "DAT002");
            }
            
            return user;
        }
        
        public void deleteUser(String username) throws DataAccessException, BusinessRuleException {
            User user = getUser(username); // May throw DataAccessException
            
            // Business rule: cannot delete admin users
            if ("admin".equals(user.getUsername())) {
                throw new BusinessRuleException("cannot_delete_admin", username, 
                    "Admin user cannot be deleted", "USR003");
            }
            
            users.remove(username);
        }
    }
    
    // Configuration service demonstrating configuration exceptions
    static class ConfigurationService {
        private java.util.Properties config = new java.util.Properties();
        
        public ConfigurationService() {
            // Initialize with some test configuration
            config.setProperty("max.connections", "10");
            config.setProperty("timeout", "30000");
            config.setProperty("invalid.number", "not-a-number");
        }
        
        public int getIntProperty(String key) throws ConfigurationException {
            String value = config.getProperty(key);
            if (value == null) {
                throw new ConfigurationException(key, null, "integer", 
                    "Configuration property not found", "CFG001");
            }
            
            try {
                return Integer.parseInt(value);
            } catch (NumberFormatException e) {
                throw new ConfigurationException(key, value, "integer", 
                    "Configuration value is not a valid integer", "CFG002");
            }
        }
        
        public String getStringProperty(String key) throws ConfigurationException {
            String value = config.getProperty(key);
            if (value == null) {
                throw new ConfigurationException(key, null, "string", 
                    "Configuration property not found", "CFG001");
            }
            return value;
        }
    }
    
    // User data class
    static class User {
        private final String username;
        private final String email;
        private final int age;
        
        public User(String username, String email, int age) {
            this.username = username;
            this.email = email;
            this.age = age;
        }
        
        public String getUsername() { return username; }
        public String getEmail() { return email; }
        public int getAge() { return age; }
        
        @Override
        public String toString() {
            return String.format("User{username='%s', email='%s', age=%d}", username, email, age);
        }
    }
    
    // Exception handler utility
    static class ExceptionHandler {
        
        public static void handleApplicationException(BaseApplicationException e) {
            System.out.printf("Application Exception Occurred:\n");
            System.out.printf("  Category: %s\n", e.getCategory());
            System.out.printf("  Error Code: %s\n", e.getErrorCode());
            System.out.printf("  Message: %s\n", e.getMessage());
            System.out.printf("  Timestamp: %s\n", e.getTimestamp());
            
            if (!e.getContext().isEmpty()) {
                System.out.printf("  Context: %s\n", e.getContext());
            }
            
            if (e.getCause() != null) {
                System.out.printf("  Root Cause: %s\n", e.getCause().getClass().getSimpleName());
            }
        }
        
        public static void handleValidationException(ValidationException e) {
            System.out.printf("Validation Exception Occurred:\n");
            System.out.printf("  Message: %s\n", e.getMessage());
            System.out.printf("  Field Errors:\n");
            
            for (ValidationError error : e.getErrors()) {
                System.out.printf("    - Field: %s, Value: %s, Error: %s\n", 
                        error.getField(), error.getRejectedValue(), error.getMessage());
            }
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Custom Exception Design Patterns ===\n");
        
        UserService userService = new UserService();
        ConfigurationService configService = new ConfigurationService();
        
        // Test validation exceptions
        System.out.println("--- Validation Exception Examples ---");
        try {
            userService.createUser("", "invalid-email", -5);
        } catch (ValidationException e) {
            ExceptionHandler.handleValidationException(e);
        } catch (BusinessRuleException e) {
            ExceptionHandler.handleApplicationException(e);
        }
        
        // Test business rule exceptions
        System.out.println("\n--- Business Rule Exception Examples ---");
        try {
            User user1 = userService.createUser("john", "john@example.com", 25);
            System.out.printf("Created user: %s\n", user1);
            
            // Try to create duplicate user
            userService.createUser("john", "john2@example.com", 30);
        } catch (ValidationException e) {
            ExceptionHandler.handleValidationException(e);
        } catch (BusinessRuleException e) {
            ExceptionHandler.handleApplicationException(e);
        }
        
        // Test age restriction
        try {
            userService.createUser("minor", "minor@example.com", 16);
        } catch (ValidationException e) {
            ExceptionHandler.handleValidationException(e);
        } catch (BusinessRuleException e) {
            ExceptionHandler.handleApplicationException(e);
        }
        
        // Test data access exceptions
        System.out.println("\n--- Data Access Exception Examples ---");
        try {
            userService.getUser("nonexistent");
        } catch (DataAccessException e) {
            ExceptionHandler.handleApplicationException(e);
        }
        
        // Test configuration exceptions
        System.out.println("\n--- Configuration Exception Examples ---");
        try {
            int maxConnections = configService.getIntProperty("max.connections");
            System.out.printf("Max connections: %d\n", maxConnections);
            
            configService.getIntProperty("nonexistent.property");
        } catch (ConfigurationException e) {
            ExceptionHandler.handleApplicationException(e);
        }
        
        try {
            configService.getIntProperty("invalid.number");
        } catch (ConfigurationException e) {
            ExceptionHandler.handleApplicationException(e);
        }
        
        // Test admin deletion
        System.out.println("\n--- Admin Deletion Business Rule ---");
        try {
            User admin = userService.createUser("admin", "admin@example.com", 30);
            System.out.printf("Created admin: %s\n", admin);
            
            userService.deleteUser("admin");
        } catch (Exception e) {
            if (e instanceof BaseApplicationException) {
                ExceptionHandler.handleApplicationException((BaseApplicationException) e);
            } else if (e instanceof ValidationException) {
                ExceptionHandler.handleValidationException((ValidationException) e);
            }
        }
        
        System.out.println("\n=== Custom Exception Best Practices ===");
        System.out.println("✅ Create meaningful exception hierarchies");
        System.out.println("✅ Include contextual information (error codes, timestamps)");
        System.out.println("✅ Use checked exceptions for recoverable errors");
        System.out.println("✅ Use unchecked exceptions for programming errors");
        System.out.println("✅ Provide multiple constructors for flexibility");
        System.out.println("✅ Include cause chains for root cause analysis");
        System.out.println("✅ Add context information for debugging");
        System.out.println("✅ Follow consistent naming conventions");
    }
}

/*
Custom Exception Design Patterns:

1. Exception Hierarchy:
   - Create base application exception
   - Extend for specific domains (business, data, config)
   - Include common fields (error code, timestamp, context)

2. Information Richness:
   - Error codes for programmatic handling
   - Context maps for debugging information
   - Timestamps for logging and auditing
   - Cause chains for root cause analysis

3. Validation Exceptions:
   - Collect multiple validation errors
   - Include field-level error details
   - Use unchecked exceptions for programming errors

4. Business Rule Exceptions:
   - Include rule name and violating value
   - Use checked exceptions for business logic
   - Provide recovery information

5. Data Access Exceptions:
   - Include operation, entity type, and ID
   - Chain underlying database exceptions
   - Provide context for debugging

6. Configuration Exceptions:
   - Include configuration key and expected format
   - Help with deployment and setup issues
   - Provide clear error messages

Best Practices:
- Design exception hierarchies thoughtfully
- Include sufficient context for debugging
- Use appropriate exception types (checked vs unchecked)
- Provide meaningful error messages
- Support exception chaining
- Follow consistent naming conventions
*/
```

## 📊 Exception Hierarchy Summary

| Exception Type | Checked | Common Use Cases | Handling Strategy |
|----------------|---------|------------------|-------------------|
| **Error** | No | System failures, OOM, Stack overflow | Don't catch (let JVM handle) |
| **RuntimeException** | No | Programming errors, validation | Optional handling, fix code |
| **IOException** | Yes | File operations, Network I/O | Must handle or declare |
| **SQLException** | Yes | Database operations | Must handle or declare |
| **Custom Checked** | Yes | Business logic, recoverable errors | Must handle or declare |
| **Custom Unchecked** | No | Validation, programming errors | Optional handling |

## ⚠️ Common Exception Handling Mistakes

### 1. Catching too generic exceptions
```java
// ❌ Bad - catches everything
try {
    riskyOperation();
} catch (Exception e) {
    // Handles all exceptions the same way
}

// ✅ Good - specific exception handling
try {
    riskyOperation();
} catch (IOException e) {
    // Handle I/O errors
} catch (SQLException e) {
    // Handle database errors
}
```

### 2. Swallowing exceptions
```java
// ❌ Bad - exception information lost
try {
    riskyOperation();
} catch (Exception e) {
    // Empty catch block - information lost
}

// ✅ Good - log or re-throw
try {
    riskyOperation();
} catch (Exception e) {
    logger.error("Operation failed", e);
    throw new BusinessException("Process failed", e);
}
```

### 3. Not using exception chaining
```java
// ❌ Bad - loses root cause
catch (SQLException e) {
    throw new BusinessException("Database error");
}

// ✅ Good - preserves root cause
catch (SQLException e) {
    throw new BusinessException("Database error", e);
}
```

## 🔥 Interview Questions & Answers

### Q1: What is the difference between Checked and Unchecked exceptions?
**Answer**: 
- **Checked Exceptions**: Must be handled at compile time using try-catch or declared with throws. Examples: IOException, SQLException
- **Unchecked Exceptions**: Optional handling, inherit from RuntimeException. Examples: NullPointerException, IllegalArgumentException
- **Compile-time**: Checked exceptions verified at compile time, unchecked at runtime
- **Recovery**: Checked for recoverable conditions, unchecked for programming errors

### Q2: Why does Java have both Error and Exception classes?
**Answer**: 
- **Error**: Represents serious system-level problems that applications shouldn't try to handle (OutOfMemoryError, StackOverflowError)
- **Exception**: Represents conditions applications can catch and handle (business logic errors, I/O failures)
- **Recovery**: Errors are generally unrecoverable, Exceptions are recoverable
- **Handling**: Applications shouldn't catch Errors, should handle Exceptions appropriately

### Q3: Can you throw an exception from a finally block?
**Answer**: **Yes**, but it's problematic:
- **Original exception lost**: If finally throws, it suppresses the original exception
- **Best practice**: Avoid throwing from finally blocks
- **Alternative**: Use try-with-resources for resource management
- **Java 7+**: Suppressed exceptions mechanism helps track multiple exceptions

### Q4: What happens if an exception is thrown in constructor?
**Answer**: 
- **Object creation fails**: Object is not created, reference remains null
- **Memory cleanup**: Partially constructed object eligible for garbage collection
- **Constructor chaining**: If super() throws, child constructor doesn't execute
- **Finally blocks**: Don't execute in constructors (no try-catch-finally in constructor)

### Q5: How do you create effective custom exceptions?
**Answer**: 
- **Extend appropriate base**: Exception for checked, RuntimeException for unchecked
- **Provide constructors**: Message, message+cause, default constructors
- **Include context**: Error codes, timestamps, relevant data
- **Follow naming**: End with "Exception" suffix
- **Document when to use**: Javadoc explaining usage scenarios

---
[← Back to Main Guide](./README.md) | [← Previous: Method Overriding](./method-overriding.md) | [Next: try-catch-finally →](./try-catch-finally.md)
