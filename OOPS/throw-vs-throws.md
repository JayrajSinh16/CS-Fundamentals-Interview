# 🎯 throw vs throws in Java - Exception Propagation Mechanisms

Understanding the difference between `throw` and `throws` is crucial for proper exception handling in Java. These keywords serve different purposes in the exception mechanism: `throw` is used to explicitly throw exceptions, while `throws` is used to declare that a method might throw exceptions.

## 🎯 Key Concepts

- **throw**: Keyword to explicitly throw an exception instance
- **throws**: Keyword to declare exceptions a method might throw
- **Exception Propagation**: How exceptions travel up the call stack
- **Checked Exception Handling**: Compiler-enforced exception declarations
- **Method Signatures**: How throws affects method contracts
- **Exception Chaining**: Linking related exceptions together

## 🌍 Real-World Analogy

**Airport Security Analogy**:
- **throw** = Security guard actually stopping a passenger (actively throwing an exception)
- **throws** = Warning sign saying "Security checkpoint ahead - be prepared for delays" (declaring potential exceptions)
- **Exception Propagation** = Passenger issue escalated from guard → supervisor → manager → airport director
- **Checked Exceptions** = Required documentation (passport, visa) that must be declared beforehand
- **Method Contract** = Airport policies posted at entrance (what problems to expect)

## 📋 Syntax and Key Differences

```
throw vs throws Comparison:
┌─────────────────────────────────────────────────────────────────┐
│                    throw vs throws Keywords                     │
├─────────────────────────────────────────────────────────────────┤
│  throw:                                                         │
│  - Used inside method body                                      │
│  - Throws actual exception instance                            │
│  - throw new ExceptionType("message");                         │
│  - Only one exception per throw statement                      │
│  - Causes immediate transfer of control                        │
│                                                                 │
│  throws:                                                        │
│  - Used in method signature                                     │
│  - Declares potential exceptions                               │
│  - throws ExceptionType1, ExceptionType2                       │
│  - Can declare multiple exception types                        │
│  - Does not throw anything, just declares                      │
└─────────────────────────────────────────────────────────────────┘

Exception Flow with throw:
┌─────────────────────────────────────────────────────────────────┐
│                     throw Statement Flow                       │
├─────────────────────────────────────────────────────────────────┤
│  Method Execution:                                              │
│  1. Normal code execution                                       │
│  2. throw new Exception("error") ← Exception created & thrown   │
│  3. Immediate control transfer                                  │
│  4. Stack unwinding begins                                      │
│  5. Exception propagates up call stack                         │
│                                                                 │
│  Call Stack Unwinding:                                         │
│  ┌─────────────────┐    ┌─────────────────┐                   │
│  │   Method A      │ ←──│   Method B      │                   │
│  │  (handles or    │    │  (propagates)   │                   │
│  │   propagates)   │    │                 │                   │
│  └─────────────────┘    └─────────────────┘                   │
│                                 ↑                              │
│                         ┌─────────────────┐                   │
│                         │   Method C      │                   │
│                         │  (throws        │                   │
│                         │   exception)    │                   │
│                         └─────────────────┘                   │
└─────────────────────────────────────────────────────────────────┘

Method Signature with throws:
┌─────────────────────────────────────────────────────────────────┐
│                  Method Declaration Syntax                     │
├─────────────────────────────────────────────────────────────────┤
│  access_modifier return_type methodName(parameters)            │
│                              throws Exception1, Exception2 {   │
│      // Method body                                             │
│      // May throw declared exceptions                          │
│  }                                                              │
│                                                                 │
│  Examples:                                                      │
│  public void readFile(String filename)                         │
│                      throws IOException, FileNotFoundException │
│                                                                 │
│  private int parseNumber(String str)                           │
│                         throws NumberFormatException           │
│                                                                 │
│  protected void processData()                                  │
│                       throws BusinessException, SQLException   │
└─────────────────────────────────────────────────────────────────┘
```

## 💻 Practical Examples

### Example 1: Basic throw vs throws Usage

```java
// ThrowVsThrowsBasics.java - Understanding fundamental differences
import java.io.*;
import java.util.*;

public class ThrowVsThrowsBasics {
    
    // Example 1: Using throw to explicitly throw exceptions
    public static void demonstrateThrowUsage() {
        System.out.println("=== throw Keyword Usage ===");
        
        try {
            validateAge(15);
        } catch (IllegalArgumentException e) {
            System.out.printf("Validation failed: %s%n", e.getMessage());
        }
        
        try {
            validateEmail("");
        } catch (IllegalArgumentException e) {
            System.out.printf("Email validation failed: %s%n", e.getMessage());
        }
        
        try {
            validatePassword("123");
        } catch (SecurityException e) {
            System.out.printf("Password validation failed: %s%n", e.getMessage());
        }
    }
    
    // Method using throw for input validation
    private static void validateAge(int age) {
        if (age < 0) {
            // throw creates and throws exception instance
            throw new IllegalArgumentException("Age cannot be negative: " + age);
        }
        if (age < 18) {
            throw new IllegalArgumentException("Age must be at least 18: " + age);
        }
        if (age > 150) {
            throw new IllegalArgumentException("Age seems unrealistic: " + age);
        }
        System.out.printf("Valid age: %d%n", age);
    }
    
    private static void validateEmail(String email) {
        if (email == null) {
            throw new IllegalArgumentException("Email cannot be null");
        }
        if (email.trim().isEmpty()) {
            throw new IllegalArgumentException("Email cannot be empty");
        }
        if (!email.contains("@")) {
            throw new IllegalArgumentException("Email must contain @ symbol");
        }
        System.out.printf("Valid email: %s%n", email);
    }
    
    private static void validatePassword(String password) {
        if (password == null) {
            throw new SecurityException("Password cannot be null");
        }
        if (password.length() < 8) {
            throw new SecurityException("Password must be at least 8 characters");
        }
        if (!password.matches(".*[A-Z].*")) {
            throw new SecurityException("Password must contain uppercase letter");
        }
        if (!password.matches(".*[0-9].*")) {
            throw new SecurityException("Password must contain digit");
        }
        System.out.printf("Valid password: %s%n", "*".repeat(password.length()));
    }
    
    // Example 2: Using throws to declare exceptions
    public static void demonstrateThrowsUsage() {
        System.out.println("\n=== throws Keyword Usage ===");
        
        try {
            String content = readFileContent("example.txt");
            System.out.printf("File content: %s%n", content);
        } catch (IOException e) {
            System.out.printf("File reading failed: %s%n", e.getMessage());
        }
        
        try {
            saveDataToDatabase("user123", "John Doe");
        } catch (DatabaseException e) {
            System.out.printf("Database operation failed: %s%n", e.getMessage());
        }
        
        try {
            processBusinessLogic("invalid_data");
        } catch (BusinessException | ValidationException e) {
            System.out.printf("Business processing failed: %s%n", e.getMessage());
        }
    }
    
    // Method declares it might throw IOException (checked exception)
    private static String readFileContent(String filename) throws IOException {
        // throws declaration required for checked exceptions
        if (!filename.endsWith(".txt")) {
            // throw can be used inside method that declares throws
            throw new IOException("Only .txt files are supported: " + filename);
        }
        
        // Simulate file reading
        if (filename.equals("nonexistent.txt")) {
            throw new FileNotFoundException("File not found: " + filename);
        }
        
        return "Sample file content for " + filename;
    }
    
    // Method declares multiple exception types
    private static void saveDataToDatabase(String id, String data) 
            throws DatabaseException {
        
        if (id == null || id.trim().isEmpty()) {
            throw new DatabaseException("ID cannot be null or empty");
        }
        
        if (data == null) {
            throw new DatabaseException("Data cannot be null");
        }
        
        // Simulate database operation
        if (Math.random() > 0.7) {
            throw new DatabaseException("Database connection failed");
        }
        
        System.out.printf("Data saved: %s → %s%n", id, data);
    }
    
    // Method declaring multiple exception types
    private static void processBusinessLogic(String input) 
            throws BusinessException, ValidationException {
        
        // Input validation - throws ValidationException
        if (input == null || input.trim().isEmpty()) {
            throw new ValidationException("Input cannot be empty");
        }
        
        // Business rule validation - throws BusinessException
        if (input.equals("invalid_data")) {
            throw new BusinessException("Business rule violation: invalid data format");
        }
        
        System.out.printf("Business logic processed: %s%n", input);
    }
    
    // Example 3: Exception propagation
    public static void demonstrateExceptionPropagation() {
        System.out.println("\n=== Exception Propagation ===");
        
        try {
            methodLevel1();
        } catch (CustomException e) {
            System.out.printf("Caught at main level: %s%n", e.getMessage());
            System.out.printf("Exception chain length: %d%n", getExceptionChainLength(e));
        }
    }
    
    // Level 1: Catches and re-throws with additional context
    private static void methodLevel1() throws CustomException {
        try {
            methodLevel2();
        } catch (Exception e) {
            // Catch and re-throw with additional context
            throw new CustomException("Level 1 processing failed", e);
        }
    }
    
    // Level 2: Propagates exception from level 3
    private static void methodLevel2() throws DatabaseException {
        methodLevel3(); // Exception propagates up
    }
    
    // Level 3: Throws original exception
    private static void methodLevel3() throws DatabaseException {
        throw new DatabaseException("Database connection timeout at level 3");
    }
    
    private static int getExceptionChainLength(Throwable throwable) {
        int length = 0;
        Throwable current = throwable;
        while (current != null) {
            length++;
            current = current.getCause();
        }
        return length;
    }
    
    // Example 4: throw with different exception types
    public static void demonstrateThrowWithDifferentTypes() {
        System.out.println("\n=== throw with Different Exception Types ===");
        
        // Throwing unchecked exceptions (no throws declaration needed)
        try {
            throwRuntimeException();
        } catch (RuntimeException e) {
            System.out.printf("Caught runtime exception: %s%n", e.getMessage());
        }
        
        // Throwing checked exceptions (throws declaration required)
        try {
            throwCheckedException();
        } catch (Exception e) {
            System.out.printf("Caught checked exception: %s%n", e.getMessage());
        }
        
        // Throwing custom exceptions
        try {
            throwCustomException();
        } catch (CustomException e) {
            System.out.printf("Caught custom exception: %s%n", e.getMessage());
        }
    }
    
    // Method throwing unchecked exception (no throws declaration needed)
    private static void throwRuntimeException() {
        throw new IllegalStateException("This is a runtime exception");
    }
    
    // Method throwing checked exception (throws declaration required)
    private static void throwCheckedException() throws Exception {
        throw new Exception("This is a checked exception");
    }
    
    // Method throwing custom exception
    private static void throwCustomException() throws CustomException {
        throw new CustomException("This is a custom exception");
    }
    
    // Example 5: Conditional throwing
    public static void demonstrateConditionalThrowing() {
        System.out.println("\n=== Conditional Exception Throwing ===");
        
        String[] testCases = {"valid", "empty", "", null, "invalid"};
        
        for (String testCase : testCases) {
            try {
                String result = processWithConditions(testCase);
                System.out.printf("Processed '%s' → %s%n", testCase, result);
            } catch (IllegalArgumentException e) {
                System.out.printf("Invalid input '%s': %s%n", testCase, e.getMessage());
            } catch (IllegalStateException e) {
                System.out.printf("State error for '%s': %s%n", testCase, e.getMessage());
            }
        }
    }
    
    private static String processWithConditions(String input) {
        // Multiple conditional throw statements
        if (input == null) {
            throw new IllegalArgumentException("Input cannot be null");
        }
        
        if (input.trim().isEmpty()) {
            throw new IllegalArgumentException("Input cannot be empty");
        }
        
        if ("invalid".equals(input)) {
            throw new IllegalStateException("Input is in invalid state");
        }
        
        if ("empty".equals(input)) {
            throw new IllegalArgumentException("Input value 'empty' is not allowed");
        }
        
        return input.toUpperCase() + "_PROCESSED";
    }
    
    // Custom exception classes
    static class CustomException extends Exception {
        public CustomException(String message) {
            super(message);
        }
        
        public CustomException(String message, Throwable cause) {
            super(message, cause);
        }
    }
    
    static class DatabaseException extends Exception {
        public DatabaseException(String message) {
            super(message);
        }
    }
    
    static class BusinessException extends Exception {
        public BusinessException(String message) {
            super(message);
        }
    }
    
    static class ValidationException extends Exception {
        public ValidationException(String message) {
            super(message);
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== throw vs throws Comprehensive Examples ===\n");
        
        demonstrateThrowUsage();
        demonstrateThrowsUsage();
        demonstrateExceptionPropagation();
        demonstrateThrowWithDifferentTypes();
        demonstrateConditionalThrowing();
        
        System.out.println("\n=== throw vs throws Summary ===");
        System.out.println("✅ throw: Explicitly throws exception instances");
        System.out.println("✅ throws: Declares potential exceptions in method signature");
        System.out.println("✅ throw: Used inside method body");
        System.out.println("✅ throws: Used in method declaration");
        System.out.println("✅ throw: Transfers control immediately");
        System.out.println("✅ throws: Documents method contract");
        System.out.println("✅ Both enable proper exception handling and propagation");
    }
}

/*
Key Differences Summary:

throw:
- Keyword to explicitly throw an exception
- Used inside method body
- throw new ExceptionType("message");
- Only one exception per statement
- Immediately transfers control
- Creates and throws exception instance

throws:
- Keyword to declare potential exceptions
- Used in method signature
- throws ExceptionType1, ExceptionType2
- Can declare multiple exception types
- Documents method contract
- Doesn't actually throw anything

Usage Guidelines:
- Use throw for input validation
- Use throw for business rule violations
- Use throws for checked exceptions
- Use throws to document method behavior
- Use exception chaining for context preservation
*/
```

### Example 2: Advanced Exception Propagation and Handling

```java
// AdvancedThrowThrowsPatterns.java - Complex exception handling scenarios
import java.io.*;
import java.sql.*;
import java.util.*;
import java.util.concurrent.*;

public class AdvancedThrowThrowsPatterns {
    
    // Example 1: Exception transformation and propagation
    public static void demonstrateExceptionTransformation() {
        System.out.println("=== Exception Transformation ===");
        
        try {
            processUserData("invalid_user");
        } catch (UserProcessingException e) {
            System.out.printf("User processing failed: %s%n", e.getMessage());
            System.out.printf("Error code: %s%n", e.getErrorCode());
            
            // Show exception chain
            Throwable cause = e.getCause();
            if (cause != null) {
                System.out.printf("Root cause: %s%n", cause.getClass().getSimpleName());
            }
        }
    }
    
    // Service layer that transforms technical exceptions into business exceptions
    private static void processUserData(String userId) throws UserProcessingException {
        try {
            // Call data access layer
            User user = getUserFromDatabase(userId);
            validateUserBusinessRules(user);
            updateUserStatus(user);
            
        } catch (SQLException e) {
            // Transform technical exception to business exception
            throw new UserProcessingException(
                "Failed to process user data", 
                "DB_ERROR", 
                e
            );
        } catch (ValidationException e) {
            // Transform validation exception
            throw new UserProcessingException(
                "User data validation failed", 
                "VALIDATION_ERROR", 
                e
            );
        } catch (Exception e) {
            // Handle unexpected exceptions
            throw new UserProcessingException(
                "Unexpected error during user processing", 
                "SYSTEM_ERROR", 
                e
            );
        }
    }
    
    // Data access layer that might throw SQL exceptions
    private static User getUserFromDatabase(String userId) throws SQLException {
        if ("invalid_user".equals(userId)) {
            throw new SQLException("User not found in database", "23000", 1062);
        }
        return new User(userId, "John Doe", "john@example.com");
    }
    
    private static void validateUserBusinessRules(User user) throws ValidationException {
        if (user.getEmail() == null || !user.getEmail().contains("@")) {
            throw new ValidationException("Invalid email format: " + user.getEmail());
        }
    }
    
    private static void updateUserStatus(User user) throws SQLException {
        // Simulate database update
        if (Math.random() > 0.8) {
            throw new SQLException("Database update failed", "08001", 2003);
        }
    }
    
    // Example 2: Method overriding and exception handling
    public static void demonstrateMethodOverridingExceptions() {
        System.out.println("\n=== Method Overriding Exception Rules ===");
        
        // Parent class reference, child class object
        DataProcessor processor = new FileDataProcessor();
        
        try {
            processor.processData("test_data");
        } catch (IOException e) {
            System.out.printf("IO Exception: %s%n", e.getMessage());
        } catch (ProcessingException e) {
            System.out.printf("Processing Exception: %s%n", e.getMessage());
        }
        
        // Different implementation
        processor = new DatabaseDataProcessor();
        
        try {
            processor.processData("test_data");
        } catch (IOException e) {
            System.out.printf("IO Exception: %s%n", e.getMessage());
        } catch (ProcessingException e) {
            System.out.printf("Processing Exception: %s%n", e.getMessage());
        }
    }
    
    // Abstract parent class defining exception contract
    abstract static class DataProcessor {
        // Parent method declares potential exceptions
        public abstract void processData(String data) 
                throws IOException, ProcessingException;
    }
    
    // Child class can throw same or fewer exceptions
    static class FileDataProcessor extends DataProcessor {
        @Override
        public void processData(String data) throws IOException {
            // Only throws IOException (subset of parent's exceptions)
            if ("invalid_file_data".equals(data)) {
                throw new IOException("Invalid file data format");
            }
            System.out.printf("File processing completed for: %s%n", data);
        }
    }
    
    // Child class can throw different exceptions that are subclasses
    static class DatabaseDataProcessor extends DataProcessor {
        @Override
        public void processData(String data) throws ProcessingException {
            // Only throws ProcessingException (subset of parent's exceptions)
            if ("invalid_db_data".equals(data)) {
                throw new ProcessingException("Invalid database data format");
            }
            System.out.printf("Database processing completed for: %s%n", data);
        }
    }
    
    // Example 3: Exception handling in concurrent programming
    public static void demonstrateConcurrentExceptionHandling() {
        System.out.println("\n=== Concurrent Exception Handling ===");
        
        ExecutorService executor = Executors.newFixedThreadPool(3);
        List<Future<String>> futures = new ArrayList<>();
        
        // Submit tasks that might throw exceptions
        for (int i = 0; i < 5; i++) {
            final int taskId = i;
            Future<String> future = executor.submit(() -> {
                return processTaskWithExceptions(taskId);
            });
            futures.add(future);
        }
        
        // Collect results and handle exceptions
        for (int i = 0; i < futures.size(); i++) {
            try {
                String result = futures.get(i).get(1, TimeUnit.SECONDS);
                System.out.printf("Task %d result: %s%n", i, result);
            } catch (TimeoutException e) {
                System.out.printf("Task %d timed out%n", i);
            } catch (ExecutionException e) {
                Throwable cause = e.getCause();
                System.out.printf("Task %d failed: %s%n", i, cause.getMessage());
            } catch (InterruptedException e) {
                System.out.printf("Task %d interrupted%n", i);
                Thread.currentThread().interrupt();
            }
        }
        
        executor.shutdown();
    }
    
    private static String processTaskWithExceptions(int taskId) throws TaskProcessingException {
        try {
            // Simulate work
            Thread.sleep(100);
            
            if (taskId == 2) {
                throw new RuntimeException("Task " + taskId + " encountered runtime error");
            }
            
            if (taskId == 4) {
                throw new TaskProcessingException("Task " + taskId + " processing failed");
            }
            
            return "Task " + taskId + " completed successfully";
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new TaskProcessingException("Task " + taskId + " was interrupted", e);
        }
    }
    
    // Example 4: Resource management with exception handling
    public static void demonstrateResourceManagementExceptions() {
        System.out.println("\n=== Resource Management with Exceptions ===");
        
        try {
            processFileWithMultipleResources("data.txt");
        } catch (FileProcessingException e) {
            System.out.printf("File processing failed: %s%n", e.getMessage());
            
            // Show suppressed exceptions from resource cleanup
            Throwable[] suppressed = e.getSuppressed();
            for (int i = 0; i < suppressed.length; i++) {
                System.out.printf("Suppressed exception %d: %s%n", i + 1, suppressed[i].getMessage());
            }
        }
    }
    
    private static void processFileWithMultipleResources(String filename) 
            throws FileProcessingException {
        
        CustomResource resource1 = null;
        CustomResource resource2 = null;
        FileProcessingException primaryException = null;
        
        try {
            resource1 = new CustomResource("Resource1");
            resource2 = new CustomResource("Resource2");
            
            // Simulate file processing
            if ("invalid.txt".equals(filename)) {
                throw new FileProcessingException("Invalid file format: " + filename);
            }
            
            System.out.printf("Successfully processed file: %s%n", filename);
            
        } catch (FileProcessingException e) {
            primaryException = e;
            throw e;
        } catch (Exception e) {
            primaryException = new FileProcessingException("Unexpected error processing file", e);
            throw primaryException;
        } finally {
            // Cleanup resources and handle cleanup exceptions
            closeResourceSafely(resource2, primaryException);
            closeResourceSafely(resource1, primaryException);
        }
    }
    
    private static void closeResourceSafely(CustomResource resource, 
                                          FileProcessingException primaryException) {
        if (resource != null) {
            try {
                resource.close();
            } catch (Exception e) {
                if (primaryException != null) {
                    primaryException.addSuppressed(e);
                } else {
                    // If no primary exception, this becomes the primary
                    throw new RuntimeException("Error closing resource", e);
                }
            }
        }
    }
    
    // Example 5: Custom exception hierarchy and throwing patterns
    public static void demonstrateCustomExceptionHierarchy() {
        System.out.println("\n=== Custom Exception Hierarchy ===");
        
        try {
            businessOperationWithHierarchy("invalid_operation");
        } catch (CriticalBusinessException e) {
            System.out.printf("Critical business error: %s (Priority: %s)%n", 
                    e.getMessage(), e.getPriority());
        } catch (BusinessRuleException e) {
            System.out.printf("Business rule violation: %s (Rule: %s)%n", 
                    e.getMessage(), e.getRuleName());
        } catch (BusinessException e) {
            System.out.printf("General business error: %s%n", e.getMessage());
        }
    }
    
    private static void businessOperationWithHierarchy(String operation) 
            throws BusinessException {
        
        switch (operation) {
            case "critical_failure":
                throw new CriticalBusinessException(
                    "System is in critical state", 
                    CriticalBusinessException.Priority.HIGH
                );
                
            case "rule_violation":
                throw new BusinessRuleException(
                    "Maximum transaction limit exceeded", 
                    "MAX_TRANSACTION_LIMIT"
                );
                
            case "invalid_operation":
                throw new BusinessException("Operation not supported: " + operation);
                
            default:
                System.out.printf("Business operation completed: %s%n", operation);
        }
    }
    
    // Supporting classes
    static class User {
        private final String id;
        private final String name;
        private final String email;
        
        public User(String id, String name, String email) {
            this.id = id;
            this.name = name;
            this.email = email;
        }
        
        public String getId() { return id; }
        public String getName() { return name; }
        public String getEmail() { return email; }
    }
    
    static class CustomResource implements AutoCloseable {
        private final String name;
        private boolean open = true;
        
        public CustomResource(String name) {
            this.name = name;
            System.out.printf("Resource opened: %s%n", name);
        }
        
        @Override
        public void close() throws Exception {
            if (open) {
                open = false;
                // Simulate potential cleanup exception
                if (Math.random() > 0.7) {
                    throw new RuntimeException("Error closing " + name);
                }
                System.out.printf("Resource closed: %s%n", name);
            }
        }
    }
    
    // Exception class hierarchy
    static class BusinessException extends Exception {
        public BusinessException(String message) {
            super(message);
        }
        
        public BusinessException(String message, Throwable cause) {
            super(message, cause);
        }
    }
    
    static class BusinessRuleException extends BusinessException {
        private final String ruleName;
        
        public BusinessRuleException(String message, String ruleName) {
            super(message);
            this.ruleName = ruleName;
        }
        
        public String getRuleName() {
            return ruleName;
        }
    }
    
    static class CriticalBusinessException extends BusinessException {
        public enum Priority { LOW, MEDIUM, HIGH, CRITICAL }
        
        private final Priority priority;
        
        public CriticalBusinessException(String message, Priority priority) {
            super(message);
            this.priority = priority;
        }
        
        public Priority getPriority() {
            return priority;
        }
    }
    
    static class UserProcessingException extends Exception {
        private final String errorCode;
        
        public UserProcessingException(String message, String errorCode, Throwable cause) {
            super(message, cause);
            this.errorCode = errorCode;
        }
        
        public String getErrorCode() {
            return errorCode;
        }
    }
    
    static class ValidationException extends Exception {
        public ValidationException(String message) {
            super(message);
        }
    }
    
    static class ProcessingException extends Exception {
        public ProcessingException(String message) {
            super(message);
        }
    }
    
    static class TaskProcessingException extends Exception {
        public TaskProcessingException(String message) {
            super(message);
        }
        
        public TaskProcessingException(String message, Throwable cause) {
            super(message, cause);
        }
    }
    
    static class FileProcessingException extends Exception {
        public FileProcessingException(String message) {
            super(message);
        }
        
        public FileProcessingException(String message, Throwable cause) {
            super(message, cause);
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Advanced throw vs throws Patterns ===\n");
        
        demonstrateExceptionTransformation();
        demonstrateMethodOverridingExceptions();
        demonstrateConcurrentExceptionHandling();
        demonstrateResourceManagementExceptions();
        demonstrateCustomExceptionHierarchy();
        
        System.out.println("\n=== Advanced Exception Handling Summary ===");
        System.out.println("✅ Exception transformation preserves context");
        System.out.println("✅ Method overriding has specific exception rules");
        System.out.println("✅ Concurrent programming requires special exception handling");
        System.out.println("✅ Resource management needs careful exception coordination");
        System.out.println("✅ Custom exception hierarchies enable precise error handling");
    }
}

/*
Advanced Exception Patterns:

1. Exception Transformation:
   - Convert technical exceptions to business exceptions
   - Preserve original exception as cause
   - Add business context and error codes

2. Method Overriding Rules:
   - Overriding method cannot throw broader exceptions
   - Can throw same, fewer, or more specific exceptions
   - Unchecked exceptions can be thrown freely

3. Concurrent Exception Handling:
   - ExecutionException wraps task exceptions
   - Use Future.get() to retrieve exceptions
   - Handle TimeoutException and InterruptedException

4. Resource Management:
   - Coordinate primary and cleanup exceptions
   - Use addSuppressed() for cleanup exceptions
   - Ensure resources are closed even with exceptions

5. Exception Hierarchy Design:
   - Create meaningful exception hierarchies
   - Include contextual information
   - Enable precise exception handling
*/
```

### Example 3: Exception Propagation and Method Contracts

```java
// ExceptionContractsDemo.java - Method contracts and exception propagation
import java.io.*;
import java.util.*;

public class ExceptionContractsDemo {
    
    // Example 1: Method contracts with throws clauses
    public static void demonstrateMethodContracts() {
        System.out.println("=== Method Contracts with throws ===");
        
        // Client code must handle declared exceptions
        try {
            performFileOperation("test.txt");
        } catch (IOException e) {
            System.out.printf("File operation failed: %s%n", e.getMessage());
        } catch (SecurityException e) {
            System.out.printf("Security violation: %s%n", e.getMessage());
        }
        
        try {
            performDatabaseOperation("SELECT * FROM users");
        } catch (SQLException e) {
            System.out.printf("Database error: %s (Code: %d)%n", 
                    e.getMessage(), e.getErrorCode());
        } catch (DataAccessException e) {
            System.out.printf("Data access error: %s%n", e.getMessage());
        }
    }
    
    // Method contract: declares specific exceptions that can be thrown
    private static void performFileOperation(String filename) 
            throws IOException, SecurityException {
        
        // Input validation - throws appropriate exceptions
        if (filename == null || filename.trim().isEmpty()) {
            throw new IllegalArgumentException("Filename cannot be null or empty");
        }
        
        // Security check - throws SecurityException
        if (filename.contains("..") || filename.startsWith("/")) {
            throw new SecurityException("Invalid file path: " + filename);
        }
        
        // File operation - throws IOException
        if (!filename.endsWith(".txt")) {
            throw new IOException("Only .txt files are supported");
        }
        
        System.out.printf("File operation completed: %s%n", filename);
    }
    
    // Method contract: declares multiple exception types
    private static void performDatabaseOperation(String sql) 
            throws SQLException, DataAccessException {
        
        // Validate SQL
        if (sql == null || sql.trim().isEmpty()) {
            throw new IllegalArgumentException("SQL cannot be null or empty");
        }
        
        // Business rule validation
        if (sql.toUpperCase().contains("DROP") || sql.toUpperCase().contains("DELETE")) {
            throw new DataAccessException("Destructive operations not allowed");
        }
        
        // Database operation simulation
        if (Math.random() > 0.7) {
            throw new SQLException("Database connection timeout", "08001", 1234);
        }
        
        System.out.printf("Database operation completed: %s%n", sql);
    }
    
    // Example 2: Exception propagation through call chain
    public static void demonstrateExceptionPropagation() {
        System.out.println("\n=== Exception Propagation Chain ===");
        
        try {
            serviceLayerMethod("process_data");
        } catch (ServiceException e) {
            System.out.printf("Service layer error: %s%n", e.getMessage());
            
            // Trace exception chain
            System.out.println("Exception chain:");
            Throwable current = e;
            int level = 1;
            while (current != null) {
                System.out.printf("  %d. %s: %s%n", level, 
                        current.getClass().getSimpleName(), current.getMessage());
                current = current.getCause();
                level++;
            }
        }
    }
    
    // Service layer - transforms and propagates exceptions
    private static void serviceLayerMethod(String request) throws ServiceException {
        try {
            businessLayerMethod(request);
        } catch (BusinessLogicException e) {
            // Transform business exception to service exception
            throw new ServiceException("Service processing failed for: " + request, e);
        } catch (DataAccessException e) {
            // Transform data access exception to service exception
            throw new ServiceException("Data access failed during: " + request, e);
        }
    }
    
    // Business layer - handles business logic exceptions
    private static void businessLayerMethod(String request) 
            throws BusinessLogicException, DataAccessException {
        
        try {
            dataAccessLayerMethod(request);
            
            // Business logic validation
            if ("invalid_business_data".equals(request)) {
                throw new BusinessLogicException("Business rule violation: " + request);
            }
            
        } catch (SQLException e) {
            // Transform SQL exception to data access exception
            throw new DataAccessException("Database operation failed", e);
        }
    }
    
    // Data access layer - throws SQL exceptions
    private static void dataAccessLayerMethod(String request) throws SQLException {
        if ("database_error".equals(request)) {
            throw new SQLException("Table not found", "42S02", 1146);
        }
        
        if ("process_data".equals(request)) {
            throw new SQLException("Connection timeout", "08001", 2003);
        }
        
        System.out.printf("Data access completed: %s%n", request);
    }
    
    // Example 3: Checked vs unchecked exception propagation
    public static void demonstrateCheckedVsUncheckedPropagation() {
        System.out.println("\n=== Checked vs Unchecked Exception Propagation ===");
        
        // Checked exceptions must be handled or declared
        try {
            methodWithCheckedException();
        } catch (IOException e) {
            System.out.printf("Caught checked exception: %s%n", e.getMessage());
        }
        
        // Unchecked exceptions can propagate without declaration
        try {
            methodWithUncheckedException();
        } catch (RuntimeException e) {
            System.out.printf("Caught unchecked exception: %s%n", e.getMessage());
        }
        
        // Mixed exception handling
        try {
            methodWithMixedExceptions("test_case");
        } catch (IOException e) {
            System.out.printf("Caught IOException: %s%n", e.getMessage());
        } catch (RuntimeException e) {
            System.out.printf("Caught RuntimeException: %s%n", e.getMessage());
        }
    }
    
    // Method declaring checked exception
    private static void methodWithCheckedException() throws IOException {
        throw new IOException("This is a checked exception");
    }
    
    // Method not declaring unchecked exception (but can still throw)
    private static void methodWithUncheckedException() {
        throw new IllegalArgumentException("This is an unchecked exception");
    }
    
    // Method with mixed exception types
    private static void methodWithMixedExceptions(String input) throws IOException {
        if (input == null) {
            // Unchecked exception - no throws declaration needed
            throw new IllegalArgumentException("Input cannot be null");
        }
        
        if (input.equals("io_error")) {
            // Checked exception - throws declaration required
            throw new IOException("IO operation failed");
        }
        
        if (input.equals("state_error")) {
            // Unchecked exception - no throws declaration needed
            throw new IllegalStateException("Invalid state");
        }
        
        System.out.printf("Input processed: %s%n", input);
    }
    
    // Example 4: Exception handling in inheritance
    public static void demonstrateInheritanceExceptionRules() {
        System.out.println("\n=== Inheritance Exception Rules ===");
        
        // Parent class reference
        DataReader reader = new FileDataReader();
        
        try {
            String data = reader.readData("file.txt");
            System.out.printf("Read data: %s%n", data);
        } catch (IOException e) {
            System.out.printf("File reading failed: %s%n", e.getMessage());
        } catch (DataReaderException e) {
            System.out.printf("Data reader failed: %s%n", e.getMessage());
        }
        
        // Different implementation
        reader = new NetworkDataReader();
        
        try {
            String data = reader.readData("http://example.com/data");
            System.out.printf("Read data: %s%n", data);
        } catch (IOException e) {
            System.out.printf("Network reading failed: %s%n", e.getMessage());
        } catch (DataReaderException e) {
            System.out.printf("Data reader failed: %s%n", e.getMessage());
        }
    }
    
    // Parent class defining exception contract
    abstract static class DataReader {
        // Parent method declares potential exceptions
        public abstract String readData(String source) 
                throws IOException, DataReaderException;
    }
    
    // Child class following exception contract
    static class FileDataReader extends DataReader {
        @Override
        public String readData(String source) throws IOException {
            // Can throw same or subset of parent's exceptions
            if (source == null || !source.endsWith(".txt")) {
                throw new IOException("Invalid file source: " + source);
            }
            return "File data from " + source;
        }
    }
    
    static class NetworkDataReader extends DataReader {
        @Override
        public String readData(String source) throws IOException, DataReaderException {
            // Can throw all parent's exceptions
            if (source == null || !source.startsWith("http")) {
                throw new DataReaderException("Invalid network source: " + source);
            }
            
            if (Math.random() > 0.8) {
                throw new IOException("Network connection failed");
            }
            
            return "Network data from " + source;
        }
    }
    
    // Example 5: Exception handling best practices
    public static void demonstrateExceptionHandlingBestPractices() {
        System.out.println("\n=== Exception Handling Best Practices ===");
        
        // Best practice: Specific exception handling
        try {
            complexOperationWithMultipleExceptions("test_input");
        } catch (ValidationException e) {
            System.out.printf("Validation error: %s%n", e.getMessage());
            // Handle validation errors specifically
        } catch (ProcessingException e) {
            System.out.printf("Processing error: %s%n", e.getMessage());
            // Handle processing errors specifically
        } catch (IOException e) {
            System.out.printf("IO error: %s%n", e.getMessage());
            // Handle IO errors specifically
        } catch (Exception e) {
            System.out.printf("Unexpected error: %s%n", e.getMessage());
            // Handle any other unexpected errors
        }
    }
    
    private static void complexOperationWithMultipleExceptions(String input) 
            throws ValidationException, ProcessingException, IOException {
        
        // Input validation
        if (input == null || input.trim().isEmpty()) {
            throw new ValidationException("Input cannot be null or empty");
        }
        
        // Processing logic
        if ("invalid_processing".equals(input)) {
            throw new ProcessingException("Unable to process input: " + input);
        }
        
        // IO operation
        if ("io_failure".equals(input)) {
            throw new IOException("IO operation failed for input: " + input);
        }
        
        System.out.printf("Complex operation completed for: %s%n", input);
    }
    
    // Custom exception classes
    static class ServiceException extends Exception {
        public ServiceException(String message, Throwable cause) {
            super(message, cause);
        }
    }
    
    static class BusinessLogicException extends Exception {
        public BusinessLogicException(String message) {
            super(message);
        }
    }
    
    static class DataAccessException extends Exception {
        public DataAccessException(String message) {
            super(message);
        }
        
        public DataAccessException(String message, Throwable cause) {
            super(message, cause);
        }
    }
    
    static class SQLException extends Exception {
        private final String sqlState;
        private final int errorCode;
        
        public SQLException(String message, String sqlState, int errorCode) {
            super(message);
            this.sqlState = sqlState;
            this.errorCode = errorCode;
        }
        
        public String getSqlState() { return sqlState; }
        public int getErrorCode() { return errorCode; }
    }
    
    static class DataReaderException extends Exception {
        public DataReaderException(String message) {
            super(message);
        }
    }
    
    static class ValidationException extends Exception {
        public ValidationException(String message) {
            super(message);
        }
    }
    
    static class ProcessingException extends Exception {
        public ProcessingException(String message) {
            super(message);
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Exception Contracts and Propagation Demo ===\n");
        
        demonstrateMethodContracts();
        demonstrateExceptionPropagation();
        demonstrateCheckedVsUncheckedPropagation();
        demonstrateInheritanceExceptionRules();
        demonstrateExceptionHandlingBestPractices();
        
        System.out.println("\n=== Exception Contracts Summary ===");
        System.out.println("✅ throws declares method exception contract");
        System.out.println("✅ Checked exceptions must be handled or declared");
        System.out.println("✅ Exception propagation follows call stack");
        System.out.println("✅ Inheritance restricts exception throwing");
        System.out.println("✅ Specific exception handling improves error handling");
    }
}

/*
Exception Contract Rules:

1. Method Contracts:
   - throws clause declares potential exceptions
   - Clients must handle declared checked exceptions
   - Documents method behavior and error conditions

2. Exception Propagation:
   - Exceptions travel up call stack if not caught
   - Each layer can transform exceptions
   - Preserve original exception as cause

3. Checked vs Unchecked:
   - Checked: Must be handled or declared
   - Unchecked: Can propagate without declaration
   - Mixed: Handle appropriately based on type

4. Inheritance Rules:
   - Override methods cannot throw broader exceptions
   - Can throw same, fewer, or more specific exceptions
   - Unchecked exceptions not restricted

5. Best Practices:
   - Handle specific exceptions before general ones
   - Transform exceptions at layer boundaries
   - Preserve exception context and causes
   - Document exception behavior clearly
*/
```

## 📊 throw vs throws Comparison Table

| Aspect | throw | throws |
|--------|-------|--------|
| **Purpose** | Explicitly throw exception | Declare potential exceptions |
| **Location** | Inside method body | Method signature |
| **Syntax** | `throw new Exception("msg");` | `throws Exception1, Exception2` |
| **Quantity** | One exception per statement | Multiple exceptions per declaration |
| **Action** | Creates and throws exception | Documents method behavior |
| **Control Flow** | Immediately transfers control | No control transfer |
| **Usage** | Input validation, error conditions | Method contracts, API documentation |

## ⚠️ Common throw vs throws Mistakes

### 1. Confusing throw and throws
```java
// ❌ Wrong - using throws inside method body
void method() {
    throws new Exception("error"); // Compilation error
}

// ✅ Correct - using throw inside method body
void method() throws Exception {
    throw new Exception("error");
}
```

### 2. Missing throws declaration for checked exceptions
```java
// ❌ Wrong - checked exception not declared
void readFile(String filename) {
    throw new IOException("File not found"); // Compilation error
}

// ✅ Correct - checked exception declared
void readFile(String filename) throws IOException {
    throw new IOException("File not found");
}
```

### 3. Overly broad exception declarations
```java
// ❌ Poor - too broad exception declaration
void processData() throws Exception {
    // Specific operations
}

// ✅ Better - specific exception declarations
void processData() throws IOException, SQLException {
    // Specific operations
}
```

## 🔥 Interview Questions & Answers

### Q1: What's the difference between throw and throws in Java?
**Answer**: 
- **throw**: Keyword used to explicitly throw an exception instance inside a method body
- **throws**: Keyword used in method signature to declare that method might throw specific exceptions
- **Usage**: throw is for actual exception throwing, throws is for declaring potential exceptions
- **Syntax**: `throw new Exception("msg");` vs `method() throws Exception`
- **Control**: throw immediately transfers control, throws just documents behavior

### Q2: Can a method have multiple throw statements?
**Answer**: **Yes**, methods can have multiple throw statements:
- **Conditional throwing**: Different conditions can throw different exceptions
- **Single execution**: Only one throw statement executes per method call
- **Control transfer**: First throw statement that executes terminates method
- **Exception types**: Each throw can create different exception types

### Q3: What happens if you don't handle a checked exception that's thrown?
**Answer**: 
- **Compilation error**: Code won't compile if checked exception not handled
- **Must handle**: Either catch with try-catch or declare with throws
- **Propagation**: If declared with throws, exception propagates to caller
- **Chain responsibility**: Each caller must handle or declare the exception

### Q4: Can overriding methods change the throws clause?
**Answer**: **Partially**:
- **Restrictions**: Cannot throw broader checked exceptions than parent
- **Allowed**: Can throw same, fewer, or more specific checked exceptions
- **Unchecked exceptions**: No restrictions on RuntimeException and subclasses
- **Interface compliance**: Must maintain exception contract compatibility

### Q5: How do you handle exceptions in lambda expressions?
**Answer**: 
- **Unchecked exceptions**: Can throw directly from lambda
- **Checked exceptions**: Must be caught inside lambda or use wrapper methods
- **Functional interfaces**: Standard functional interfaces don't declare checked exceptions
- **Workarounds**: Create custom functional interfaces or use exception wrapping utilities

---
[← Back to Main Guide](./README.md) | [← Previous: try-catch-finally](./try-catch-finally.md) | [Next: Custom Exceptions →](./custom-exceptions.md)
