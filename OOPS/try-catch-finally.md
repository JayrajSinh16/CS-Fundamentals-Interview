# 🔧 try-catch-finally in Java - Exception Handling Blocks

The try-catch-finally construct is Java's primary mechanism for handling exceptions and managing resources. Understanding these blocks is essential for writing robust, reliable Java applications that gracefully handle errors and properly manage system resources.

## 🎯 Key Concepts

- **try Block**: Contains code that might throw exceptions
- **catch Block**: Handles specific types of exceptions
- **finally Block**: Always executes (cleanup code)
- **Multi-catch**: Handle multiple exception types in one catch
- **try-with-resources**: Automatic resource management (Java 7+)
- **Exception Propagation**: How uncaught exceptions flow up the call stack

## 🌍 Real-World Analogy

**Restaurant Kitchen Analogy**:
- **try Block** = Cooking process (might burn food, run out of ingredients)
- **catch Block** = Different responses to problems (burnt food → make new dish, no ingredients → substitute)
- **finally Block** = Kitchen cleanup (always wash dishes, turn off stove, regardless of success/failure)
- **try-with-resources** = Using automatic equipment that shuts off when done
- **Exception Propagation** = If kitchen can't handle problem, escalate to manager, then owner

## 📋 Basic Syntax and Structure

```
Basic Structure:
┌─────────────────────────────────────────────────────────────────┐
│                      try-catch-finally                         │
├─────────────────────────────────────────────────────────────────┤
│  try {                                                          │
│      // Code that might throw exceptions                       │
│      // Resource allocation                                     │
│  }                                                              │
│  catch (SpecificException e) {                                 │
│      // Handle specific exception                               │
│  }                                                              │
│  catch (AnotherException e) {                                  │
│      // Handle another exception type                          │
│  }                                                              │
│  catch (Exception e) {                                         │
│      // Handle any other exception (catch-all)                 │
│  }                                                              │
│  finally {                                                      │
│      // Cleanup code - always executes                         │
│      // Resource deallocation                                  │
│  }                                                              │
└─────────────────────────────────────────────────────────────────┘

Execution Flow:
┌─────────────────────────────────────────────────────────────────┐
│                    Exception Handling Flow                     │
├─────────────────────────────────────────────────────────────────┤
│  Normal Flow (No Exception):                                   │
│  try → finally → continue                                       │
│                                                                 │
│  Exception Flow (Caught):                                      │
│  try → catch → finally → continue                              │
│                                                                 │
│  Exception Flow (Uncaught):                                    │
│  try → finally → propagate exception                           │
│                                                                 │
│  Return in try/catch:                                          │
│  try/catch → finally → return                                  │
└─────────────────────────────────────────────────────────────────┘

Resource Management Evolution:
┌─────────────────────────────────────────────────────────────────┐
│          Traditional vs Modern Resource Management             │
├─────────────────────────────────────────────────────────────────┤
│  Traditional (Java 6 and earlier):                             │
│  FileInputStream fis = null;                                   │
│  try {                                                          │
│      fis = new FileInputStream("file.txt");                    │
│      // Process file                                           │
│  } catch (IOException e) {                                     │
│      // Handle exception                                       │
│  } finally {                                                    │
│      if (fis != null) {                                        │
│          try { fis.close(); } catch (IOException e) { }        │
│      }                                                          │
│  }                                                              │
│                                                                 │
│  Modern (Java 7+ try-with-resources):                          │
│  try (FileInputStream fis = new FileInputStream("file.txt")) {  │
│      // Process file                                           │
│  } catch (IOException e) {                                     │
│      // Handle exception - resource auto-closed               │
│  }                                                              │
└─────────────────────────────────────────────────────────────────┘
```

## 💻 Practical Examples

### Example 1: Basic try-catch-finally Usage

```java
// BasicExceptionHandling.java - Fundamental exception handling patterns
import java.io.*;
import java.util.*;

public class BasicExceptionHandling {
    
    // Basic try-catch example
    public static void demonstrateBasicTryCatch() {
        System.out.println("=== Basic try-catch Example ===");
        
        try {
            int result = divideNumbers(10, 0);
            System.out.printf("Result: %d%n", result);
        } catch (ArithmeticException e) {
            System.out.printf("Arithmetic error: %s%n", e.getMessage());
        }
        
        System.out.println("Program continues after exception handling");
    }
    
    private static int divideNumbers(int a, int b) {
        return a / b; // May throw ArithmeticException
    }
    
    // Multiple catch blocks example
    public static void demonstrateMultipleCatch() {
        System.out.println("\n=== Multiple catch blocks ===");
        
        String[] testInputs = {"123", "abc", null};
        
        for (String input : testInputs) {
            try {
                int number = parseAndProcess(input);
                System.out.printf("Processed: %s → %d%n", input, number);
            } catch (NumberFormatException e) {
                System.out.printf("Invalid number format: %s%n", input);
            } catch (NullPointerException e) {
                System.out.printf("Null input provided%n");
            } catch (IllegalArgumentException e) {
                System.out.printf("Invalid argument: %s%n", e.getMessage());
            }
        }
    }
    
    private static int parseAndProcess(String input) {
        if (input == null) {
            throw new NullPointerException("Input cannot be null");
        }
        
        int number = Integer.parseInt(input); // May throw NumberFormatException
        
        if (number < 0) {
            throw new IllegalArgumentException("Number must be positive");
        }
        
        return number * 2;
    }
    
    // Basic finally block example
    public static void demonstrateFinally() {
        System.out.println("\n=== finally block Example ===");
        
        boolean[] testCases = {true, false}; // true = exception, false = normal
        
        for (boolean shouldThrow : testCases) {
            System.out.printf("\nTesting with shouldThrow=%b:%n", shouldThrow);
            
            try {
                System.out.println("Entering try block");
                if (shouldThrow) {
                    throw new RuntimeException("Test exception");
                }
                System.out.println("try block completed normally");
            } catch (RuntimeException e) {
                System.out.printf("Caught exception: %s%n", e.getMessage());
            } finally {
                System.out.println("finally block always executes");
            }
            
            System.out.println("After try-catch-finally");
        }
    }
    
    // Resource management with finally
    public static void demonstrateResourceManagement() {
        System.out.println("\n=== Resource Management with finally ===");
        
        FileInputStream fileStream = null;
        BufferedReader reader = null;
        
        try {
            // Simulate file operations
            System.out.println("Opening resources...");
            fileStream = createMockFileInputStream();
            reader = new BufferedReader(new InputStreamReader(fileStream));
            
            System.out.println("Processing file...");
            String line = reader.readLine();
            if (line == null) {
                throw new IOException("File is empty");
            }
            System.out.printf("First line: %s%n", line);
            
        } catch (IOException e) {
            System.out.printf("File operation failed: %s%n", e.getMessage());
        } finally {
            // Resource cleanup - always executed
            System.out.println("Cleaning up resources...");
            
            if (reader != null) {
                try {
                    reader.close();
                    System.out.println("Reader closed");
                } catch (IOException e) {
                    System.out.printf("Failed to close reader: %s%n", e.getMessage());
                }
            }
            
            if (fileStream != null) {
                try {
                    fileStream.close();
                    System.out.println("File stream closed");
                } catch (IOException e) {
                    System.out.printf("Failed to close file stream: %s%n", e.getMessage());
                }
            }
        }
    }
    
    // Helper method to simulate file input stream
    private static FileInputStream createMockFileInputStream() throws IOException {
        // In real scenario, this would open an actual file
        // For demo, we'll simulate with exception
        throw new IOException("File not found: example.txt");
    }
    
    // Nested try-catch example
    public static void demonstrateNestedTryCatch() {
        System.out.println("\n=== Nested try-catch Example ===");
        
        try {
            System.out.println("Outer try block");
            
            try {
                System.out.println("Inner try block");
                throw new NumberFormatException("Inner exception");
            } catch (NumberFormatException e) {
                System.out.printf("Inner catch: %s%n", e.getMessage());
                // Re-throw as different exception type
                throw new IllegalStateException("Converted from NumberFormatException", e);
            } finally {
                System.out.println("Inner finally block");
            }
            
        } catch (IllegalStateException e) {
            System.out.printf("Outer catch: %s%n", e.getMessage());
            System.out.printf("Original cause: %s%n", e.getCause().getClass().getSimpleName());
        } finally {
            System.out.println("Outer finally block");
        }
    }
    
    // Exception in finally block
    public static void demonstrateExceptionInFinally() {
        System.out.println("\n=== Exception in finally Block ===");
        
        try {
            System.out.println("Original try block");
            throw new RuntimeException("Original exception");
        } catch (RuntimeException e) {
            System.out.printf("Caught original: %s%n", e.getMessage());
            // Note: If finally throws exception, it suppresses this one
        } finally {
            System.out.println("finally block executing");
            
            try {
                // Simulate cleanup that might fail
                throw new RuntimeException("Cleanup failed");
            } catch (RuntimeException e) {
                System.out.printf("Exception in cleanup: %s%n", e.getMessage());
                // Handle cleanup exception locally to avoid suppressing original
            }
        }
    }
    
    // Return statement in try-catch-finally
    public static void demonstrateReturnInTryCatch() {
        System.out.println("\n=== Return in try-catch-finally ===");
        
        int result1 = methodWithReturnInTry();
        System.out.printf("Method with return in try: %d%n", result1);
        
        int result2 = methodWithReturnInFinally();
        System.out.printf("Method with return in finally: %d%n", result2);
    }
    
    private static int methodWithReturnInTry() {
        try {
            System.out.println("try block - about to return 1");
            return 1;
        } catch (Exception e) {
            System.out.println("catch block - about to return 2");
            return 2;
        } finally {
            System.out.println("finally block executes even with return in try");
            // finally executes but doesn't override return value
        }
    }
    
    private static int methodWithReturnInFinally() {
        try {
            System.out.println("try block - about to return 1");
            return 1;
        } catch (Exception e) {
            System.out.println("catch block - about to return 2");
            return 2;
        } finally {
            System.out.println("finally block - about to return 3");
            return 3; // This overrides the try/catch return value!
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== try-catch-finally Comprehensive Examples ===\n");
        
        demonstrateBasicTryCatch();
        demonstrateMultipleCatch();
        demonstrateFinally();
        demonstrateResourceManagement();
        demonstrateNestedTryCatch();
        demonstrateExceptionInFinally();
        demonstrateReturnInTryCatch();
        
        System.out.println("\n=== Basic Exception Handling Summary ===");
        System.out.println("✅ try block contains risky code");
        System.out.println("✅ catch blocks handle specific exceptions");
        System.out.println("✅ finally block always executes (cleanup)");
        System.out.println("✅ Multiple catch blocks for different exception types");
        System.out.println("✅ finally executes even with return statements");
        System.out.println("✅ Proper resource cleanup is essential");
    }
}

/*
Basic Exception Handling Patterns:

1. Single Exception:
   try { } catch (SpecificException e) { } finally { }

2. Multiple Exceptions:
   try { } 
   catch (Exception1 e) { }
   catch (Exception2 e) { }
   finally { }

3. Generic Catch:
   try { } 
   catch (SpecificException e) { }
   catch (Exception e) { } // Catch-all
   finally { }

4. Resource Management:
   try { open resources }
   catch { handle errors }
   finally { close resources }

Key Points:
- finally always executes (unless JVM terminates)
- Multiple catch blocks: specific to general
- finally executes even with return statements
- Be careful with exceptions in finally blocks
*/
```

### Example 2: Advanced Exception Handling Patterns

```java
// AdvancedExceptionHandling.java - Advanced try-catch-finally patterns
import java.io.*;
import java.util.*;
import java.util.concurrent.*;

public class AdvancedExceptionHandling {
    
    // Multi-catch example (Java 7+)
    public static void demonstrateMultiCatch() {
        System.out.println("=== Multi-catch Example (Java 7+) ===");
        
        String[] testInputs = {"123", "abc", null, "-1"};
        
        for (String input : testInputs) {
            try {
                int result = processInput(input);
                System.out.printf("Processed '%s' → %d%n", input, result);
            } catch (NumberFormatException | IllegalArgumentException e) {
                // Multi-catch: handle multiple exception types the same way
                System.out.printf("Input error for '%s': %s%n", input, e.getMessage());
            } catch (NullPointerException e) {
                System.out.printf("Null input: %s%n", e.getMessage());
            }
        }
    }
    
    private static int processInput(String input) {
        if (input == null) {
            throw new NullPointerException("Input cannot be null");
        }
        
        int number = Integer.parseInt(input); // NumberFormatException
        
        if (number < 0) {
            throw new IllegalArgumentException("Number must be positive: " + number);
        }
        
        return number * number;
    }
    
    // Exception handling with inheritance hierarchy
    public static void demonstrateExceptionHierarchy() {
        System.out.println("\n=== Exception Hierarchy Handling ===");
        
        Exception[] testExceptions = {
            new FileNotFoundException("File not found"),
            new IOException("General IO error"),
            new Exception("General exception"),
            new RuntimeException("Runtime error"),
            new IllegalArgumentException("Bad argument")
        };
        
        for (Exception testException : testExceptions) {
            handleException(testException);
        }
    }
    
    private static void handleException(Exception e) {
        try {
            throw e;
        } catch (FileNotFoundException fnfe) {
            System.out.printf("File not found: %s%n", fnfe.getMessage());
        } catch (IOException ioe) {
            System.out.printf("IO error: %s%n", ioe.getMessage());
        } catch (IllegalArgumentException iae) {
            System.out.printf("Argument error: %s%n", iae.getMessage());
        } catch (RuntimeException re) {
            System.out.printf("Runtime error: %s%n", re.getMessage());
        } catch (Exception ex) {
            System.out.printf("General error: %s%n", ex.getMessage());
        }
    }
    
    // Exception suppression demonstration
    public static void demonstrateExceptionSuppression() {
        System.out.println("\n=== Exception Suppression Example ===");
        
        try {
            methodWithSuppressedException();
        } catch (Exception e) {
            System.out.printf("Main exception: %s%n", e.getMessage());
            
            // Check for suppressed exceptions
            Throwable[] suppressed = e.getSuppressed();
            if (suppressed.length > 0) {
                System.out.printf("Suppressed exceptions (%d):%n", suppressed.length);
                for (int i = 0; i < suppressed.length; i++) {
                    System.out.printf("  %d. %s: %s%n", i + 1, 
                            suppressed[i].getClass().getSimpleName(), 
                            suppressed[i].getMessage());
                }
            }
        }
    }
    
    private static void methodWithSuppressedException() throws Exception {
        Exception primaryException = null;
        
        try {
            throw new IOException("Primary exception");
        } catch (IOException e) {
            primaryException = e;
            throw e;
        } finally {
            try {
                // Simulate cleanup that fails
                throw new RuntimeException("Cleanup failed");
            } catch (RuntimeException cleanupException) {
                if (primaryException != null) {
                    primaryException.addSuppressed(cleanupException);
                } else {
                    throw cleanupException;
                }
            }
        }
    }
    
    // Complex resource management example
    public static void demonstrateComplexResourceManagement() {
        System.out.println("\n=== Complex Resource Management ===");
        
        DatabaseConnection connection = null;
        FileWriter writer = null;
        NetworkSocket socket = null;
        
        try {
            System.out.println("Acquiring multiple resources...");
            
            connection = new DatabaseConnection("localhost:5432");
            connection.connect();
            
            writer = new FileWriter("output.txt");
            
            socket = new NetworkSocket("api.example.com", 80);
            socket.connect();
            
            // Simulate work with resources
            processDataWithMultipleResources(connection, writer, socket);
            
        } catch (DatabaseException e) {
            System.out.printf("Database error: %s%n", e.getMessage());
        } catch (IOException e) {
            System.out.printf("File/Network error: %s%n", e.getMessage());
        } catch (Exception e) {
            System.out.printf("Unexpected error: %s%n", e.getMessage());
        } finally {
            // Close resources in reverse order of acquisition
            closeResource("Socket", socket);
            closeResource("FileWriter", writer);
            closeResource("DatabaseConnection", connection);
        }
    }
    
    private static void processDataWithMultipleResources(
            DatabaseConnection connection, FileWriter writer, NetworkSocket socket) 
            throws DatabaseException, IOException {
        
        // Simulate processing that might fail
        if (Math.random() > 0.7) {
            throw new DatabaseException("Query failed");
        }
        
        if (Math.random() > 0.8) {
            throw new IOException("Write operation failed");
        }
        
        System.out.println("Successfully processed data with all resources");
    }
    
    private static void closeResource(String resourceName, AutoCloseable resource) {
        if (resource != null) {
            try {
                resource.close();
                System.out.printf("%s closed successfully%n", resourceName);
            } catch (Exception e) {
                System.out.printf("Failed to close %s: %s%n", resourceName, e.getMessage());
            }
        }
    }
    
    // Method that demonstrates re-throwing exceptions
    public static void demonstrateRethrowingExceptions() {
        System.out.println("\n=== Re-throwing Exceptions ===");
        
        try {
            methodThatRethrows();
        } catch (CustomBusinessException e) {
            System.out.printf("Business error: %s%n", e.getMessage());
            System.out.printf("Error code: %s%n", e.getErrorCode());
            if (e.getCause() != null) {
                System.out.printf("Root cause: %s%n", e.getCause().getMessage());
            }
        }
    }
    
    private static void methodThatRethrows() throws CustomBusinessException {
        try {
            // Simulate operation that might fail
            performDatabaseOperation();
        } catch (SQLException e) {
            // Log the technical error
            System.out.printf("Technical error logged: %s%n", e.getMessage());
            
            // Re-throw as business exception
            throw new CustomBusinessException("Failed to retrieve customer data", "BIZ001", e);
        } catch (Exception e) {
            // Handle unexpected errors
            System.out.printf("Unexpected error logged: %s%n", e.getMessage());
            throw new CustomBusinessException("System error occurred", "SYS001", e);
        }
    }
    
    private static void performDatabaseOperation() throws SQLException {
        // Simulate database operation that fails
        throw new SQLException("Connection timeout", "08001", 1234);
    }
    
    // Exception handling in loops
    public static void demonstrateExceptionHandlingInLoops() {
        System.out.println("\n=== Exception Handling in Loops ===");
        
        String[] dataItems = {"valid1", "invalid", "valid2", null, "valid3"};
        List<String> processedItems = new ArrayList<>();
        List<String> failedItems = new ArrayList<>();
        
        // Process all items, collecting successes and failures
        for (String item : dataItems) {
            try {
                String processed = processDataItem(item);
                processedItems.add(processed);
                System.out.printf("Processed: %s → %s%n", item, processed);
            } catch (Exception e) {
                failedItems.add(item);
                System.out.printf("Failed to process '%s': %s%n", item, e.getMessage());
                // Continue processing other items
            }
        }
        
        System.out.printf("Summary: %d processed, %d failed%n", 
                processedItems.size(), failedItems.size());
    }
    
    private static String processDataItem(String item) {
        if (item == null) {
            throw new IllegalArgumentException("Item cannot be null");
        }
        if ("invalid".equals(item)) {
            throw new RuntimeException("Invalid item format");
        }
        return item.toUpperCase() + "_PROCESSED";
    }
    
    // Supporting classes for examples
    static class DatabaseConnection implements AutoCloseable {
        private final String url;
        private boolean connected = false;
        
        public DatabaseConnection(String url) {
            this.url = url;
        }
        
        public void connect() throws DatabaseException {
            if (Math.random() > 0.8) {
                throw new DatabaseException("Failed to connect to: " + url);
            }
            connected = true;
            System.out.printf("Connected to database: %s%n", url);
        }
        
        @Override
        public void close() throws DatabaseException {
            if (connected) {
                connected = false;
                if (Math.random() > 0.9) {
                    throw new DatabaseException("Error closing connection");
                }
            }
        }
    }
    
    static class NetworkSocket implements AutoCloseable {
        private final String host;
        private final int port;
        private boolean connected = false;
        
        public NetworkSocket(String host, int port) {
            this.host = host;
            this.port = port;
        }
        
        public void connect() throws IOException {
            if (Math.random() > 0.7) {
                throw new IOException(String.format("Failed to connect to %s:%d", host, port));
            }
            connected = true;
            System.out.printf("Connected to %s:%d%n", host, port);
        }
        
        @Override
        public void close() throws IOException {
            if (connected) {
                connected = false;
                if (Math.random() > 0.95) {
                    throw new IOException("Error closing socket");
                }
            }
        }
    }
    
    static class CustomBusinessException extends Exception {
        private final String errorCode;
        
        public CustomBusinessException(String message, String errorCode) {
            super(message);
            this.errorCode = errorCode;
        }
        
        public CustomBusinessException(String message, String errorCode, Throwable cause) {
            super(message, cause);
            this.errorCode = errorCode;
        }
        
        public String getErrorCode() {
            return errorCode;
        }
    }
    
    static class DatabaseException extends Exception {
        public DatabaseException(String message) {
            super(message);
        }
        
        public DatabaseException(String message, Throwable cause) {
            super(message, cause);
        }
    }
    
    // Java 9+ enhanced FileWriter (simulation)
    static class FileWriter implements AutoCloseable {
        private final String filename;
        private boolean open = false;
        
        public FileWriter(String filename) throws IOException {
            this.filename = filename;
            if (Math.random() > 0.9) {
                throw new IOException("Cannot create file: " + filename);
            }
            open = true;
            System.out.printf("File opened: %s%n", filename);
        }
        
        @Override
        public void close() throws IOException {
            if (open) {
                open = false;
                if (Math.random() > 0.95) {
                    throw new IOException("Error closing file: " + filename);
                }
            }
        }
    }
    
    // SQL exception simulation
    static class SQLException extends Exception {
        private final String sqlState;
        private final int errorCode;
        
        public SQLException(String message, String sqlState, int errorCode) {
            super(message);
            this.sqlState = sqlState;
            this.errorCode = errorCode;
        }
        
        public String getSQLState() { return sqlState; }
        public int getErrorCode() { return errorCode; }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Advanced Exception Handling Patterns ===\n");
        
        demonstrateMultiCatch();
        demonstrateExceptionHierarchy();
        demonstrateExceptionSuppression();
        demonstrateComplexResourceManagement();
        demonstrateRethrowingExceptions();
        demonstrateExceptionHandlingInLoops();
        
        System.out.println("\n=== Advanced Exception Handling Summary ===");
        System.out.println("✅ Multi-catch for handling similar exceptions");
        System.out.println("✅ Exception hierarchy affects catch order");
        System.out.println("✅ Suppressed exceptions preserve cleanup errors");
        System.out.println("✅ Complex resource management requires careful cleanup");
        System.out.println("✅ Re-throwing with context preserves root causes");
        System.out.println("✅ Loop exception handling enables partial processing");
    }
}

/*
Advanced Exception Handling Patterns:

1. Multi-catch (Java 7+):
   catch (Exception1 | Exception2 e) { }

2. Exception Hierarchy:
   - More specific exceptions first
   - General exceptions last
   - FileNotFoundException before IOException

3. Exception Suppression:
   - try-finally with both throwing exceptions
   - addSuppressed() preserves cleanup exceptions
   - getSuppressed() retrieves suppressed exceptions

4. Complex Resource Management:
   - Multiple resources acquired/released
   - Reverse order cleanup
   - Individual resource cleanup error handling

5. Re-throwing with Context:
   - Catch technical exceptions
   - Re-throw as business exceptions
   - Preserve root cause with exception chaining

Best Practices:
- Handle specific exceptions before general ones
- Always clean up resources in finally
- Use multi-catch for similar handling
- Preserve exception context when re-throwing
- Continue processing after recoverable errors
*/
```

### Example 3: try-with-resources (Modern Resource Management)

```java
// TryWithResourcesDemo.java - Modern resource management (Java 7+)
import java.io.*;
import java.nio.file.*;
import java.util.*;
import java.util.zip.*;

public class TryWithResourcesDemo {
    
    // Basic try-with-resources example
    public static void demonstrateBasicTryWithResources() {
        System.out.println("=== Basic try-with-resources Example ===");
        
        // Old way (verbose and error-prone)
        demonstrateTraditionalResourceManagement();
        
        // New way (clean and automatic)
        demonstrateModernResourceManagement();
    }
    
    private static void demonstrateTraditionalResourceManagement() {
        System.out.println("\n--- Traditional Resource Management ---");
        
        FileInputStream fis = null;
        BufferedInputStream bis = null;
        
        try {
            fis = new FileInputStream("example.txt");
            bis = new BufferedInputStream(fis);
            
            // Process file
            int data = bis.read();
            System.out.printf("First byte: %d%n", data);
            
        } catch (FileNotFoundException e) {
            System.out.printf("File not found: %s%n", e.getMessage());
        } catch (IOException e) {
            System.out.printf("IO error: %s%n", e.getMessage());
        } finally {
            // Manual cleanup - verbose and error-prone
            if (bis != null) {
                try {
                    bis.close();
                } catch (IOException e) {
                    System.out.printf("Error closing BufferedInputStream: %s%n", e.getMessage());
                }
            }
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    System.out.printf("Error closing FileInputStream: %s%n", e.getMessage());
                }
            }
        }
    }
    
    private static void demonstrateModernResourceManagement() {
        System.out.println("\n--- Modern Resource Management (try-with-resources) ---");
        
        // Resources automatically closed
        try (FileInputStream fis = new FileInputStream("example.txt");
             BufferedInputStream bis = new BufferedInputStream(fis)) {
            
            // Process file
            int data = bis.read();
            System.out.printf("First byte: %d%n", data);
            
        } catch (FileNotFoundException e) {
            System.out.printf("File not found: %s%n", e.getMessage());
        } catch (IOException e) {
            System.out.printf("IO error: %s%n", e.getMessage());
        }
        // Resources automatically closed here, even if exception occurs
    }
    
    // Multiple resources in try-with-resources
    public static void demonstrateMultipleResources() {
        System.out.println("\n=== Multiple Resources Example ===");
        
        try (FileReader reader = new FileReader("input.txt");
             FileWriter writer = new FileWriter("output.txt");
             BufferedReader bufferedReader = new BufferedReader(reader);
             PrintWriter printWriter = new PrintWriter(writer)) {
            
            System.out.println("Processing with multiple resources...");
            
            String line;
            while ((line = bufferedReader.readLine()) != null) {
                printWriter.println(line.toUpperCase());
            }
            
            System.out.println("File processing completed");
            
        } catch (IOException e) {
            System.out.printf("File operation failed: %s%n", e.getMessage());
        }
        // All resources closed automatically in reverse order
    }
    
    // Custom resource with AutoCloseable
    public static void demonstrateCustomResource() {
        System.out.println("\n=== Custom AutoCloseable Resource ===");
        
        try (DatabaseSession session = new DatabaseSession("localhost:5432");
             TransactionContext transaction = session.beginTransaction()) {
            
            System.out.println("Performing database operations...");
            
            // Simulate database work
            session.executeQuery("SELECT * FROM users");
            session.executeUpdate("UPDATE users SET last_login = NOW()");
            
            transaction.commit();
            System.out.println("Transaction committed successfully");
            
        } catch (DatabaseException e) {
            System.out.printf("Database error: %s%n", e.getMessage());
        }
        // Resources closed automatically: transaction first, then session
    }
    
    // Nested try-with-resources
    public static void demonstrateNestedTryWithResources() {
        System.out.println("\n=== Nested try-with-resources ===");
        
        try (ConnectionPool pool = new ConnectionPool("database.properties")) {
            
            System.out.println("Connection pool initialized");
            
            // Inner try-with-resources
            try (DatabaseConnection conn = pool.getConnection();
                 PreparedStatement stmt = conn.prepareStatement("SELECT * FROM products")) {
                
                System.out.println("Executing query...");
                ResultSetWrapper results = stmt.executeQuery();
                
                try (ResultSetWrapper autoCloseResults = results) {
                    while (autoCloseResults.next()) {
                        System.out.printf("Product: %s%n", autoCloseResults.getString("name"));
                    }
                }
                
            } catch (SQLException e) {
                System.out.printf("Query execution failed: %s%n", e.getMessage());
            }
            
        } catch (DatabaseException e) {
            System.out.printf("Connection pool error: %s%n", e.getMessage());
        }
    }
    
    // Exception suppression in try-with-resources
    public static void demonstrateExceptionSuppression() {
        System.out.println("\n=== Exception Suppression in try-with-resources ===");
        
        try (ProblematicResource resource = new ProblematicResource()) {
            
            System.out.println("Using problematic resource...");
            resource.doWork();
            
            // This will throw an exception
            throw new RuntimeException("Main operation failed");
            
        } catch (Exception e) {
            System.out.printf("Main exception: %s%n", e.getMessage());
            
            // Check suppressed exceptions from resource.close()
            Throwable[] suppressed = e.getSuppressed();
            for (int i = 0; i < suppressed.length; i++) {
                System.out.printf("Suppressed exception %d: %s%n", i + 1, suppressed[i].getMessage());
            }
        }
    }
    
    // Java 9+ enhanced try-with-resources (effectively final variables)
    public static void demonstrateJava9TryWithResources() {
        System.out.println("\n=== Java 9+ Enhanced try-with-resources ===");
        
        // Pre-Java 9: had to declare resources in try statement
        // Java 9+: can use effectively final variables
        
        FileReader reader = createFileReader("data.txt");
        FileWriter writer = createFileWriter("output.txt");
        
        if (reader != null && writer != null) {
            // Java 9+ syntax - use existing variables
            try (reader; writer) {
                System.out.println("Processing with pre-initialized resources...");
                
                char[] buffer = new char[1024];
                int charsRead = reader.read(buffer);
                if (charsRead > 0) {
                    writer.write(buffer, 0, charsRead);
                }
                
                System.out.println("File copied successfully");
                
            } catch (IOException e) {
                System.out.printf("File operation failed: %s%n", e.getMessage());
            }
        }
    }
    
    private static FileReader createFileReader(String filename) {
        try {
            return new FileReader(filename);
        } catch (FileNotFoundException e) {
            System.out.printf("Cannot create FileReader for %s: %s%n", filename, e.getMessage());
            return null;
        }
    }
    
    private static FileWriter createFileWriter(String filename) {
        try {
            return new FileWriter(filename);
        } catch (IOException e) {
            System.out.printf("Cannot create FileWriter for %s: %s%n", filename, e.getMessage());
            return null;
        }
    }
    
    // Resource leak prevention demonstration
    public static void demonstrateResourceLeakPrevention() {
        System.out.println("\n=== Resource Leak Prevention ===");
        
        // This would cause resource leaks without proper handling
        for (int i = 0; i < 5; i++) {
            try (MockResource resource = new MockResource("Resource-" + i)) {
                
                System.out.printf("Using %s%n", resource.getName());
                
                if (i == 2) {
                    throw new RuntimeException("Simulated error");
                }
                
                resource.doWork();
                
            } catch (Exception e) {
                System.out.printf("Error with resource %d: %s%n", i, e.getMessage());
                // Resource still closed automatically despite exception
            }
        }
        
        System.out.println("All resources properly closed despite exceptions");
    }
    
    // Supporting classes for examples
    static class DatabaseSession implements AutoCloseable {
        private final String connectionString;
        private boolean connected = false;
        
        public DatabaseSession(String connectionString) throws DatabaseException {
            this.connectionString = connectionString;
            connect();
        }
        
        private void connect() throws DatabaseException {
            if (Math.random() > 0.8) {
                throw new DatabaseException("Failed to connect to: " + connectionString);
            }
            connected = true;
            System.out.printf("Database session connected: %s%n", connectionString);
        }
        
        public TransactionContext beginTransaction() throws DatabaseException {
            if (!connected) {
                throw new DatabaseException("Session not connected");
            }
            return new TransactionContext(this);
        }
        
        public void executeQuery(String sql) throws DatabaseException {
            if (!connected) {
                throw new DatabaseException("Session not connected");
            }
            System.out.printf("Executed query: %s%n", sql);
        }
        
        public void executeUpdate(String sql) throws DatabaseException {
            if (!connected) {
                throw new DatabaseException("Session not connected");
            }
            System.out.printf("Executed update: %s%n", sql);
        }
        
        @Override
        public void close() throws DatabaseException {
            if (connected) {
                connected = false;
                System.out.printf("Database session closed: %s%n", connectionString);
                
                if (Math.random() > 0.9) {
                    throw new DatabaseException("Error closing session");
                }
            }
        }
    }
    
    static class TransactionContext implements AutoCloseable {
        private final DatabaseSession session;
        private boolean active = true;
        private boolean committed = false;
        
        public TransactionContext(DatabaseSession session) {
            this.session = session;
            System.out.println("Transaction started");
        }
        
        public void commit() throws DatabaseException {
            if (!active) {
                throw new DatabaseException("Transaction not active");
            }
            committed = true;
            System.out.println("Transaction committed");
        }
        
        @Override
        public void close() throws DatabaseException {
            if (active) {
                active = false;
                if (!committed) {
                    System.out.println("Transaction rolled back");
                }
                
                if (Math.random() > 0.95) {
                    throw new DatabaseException("Error closing transaction");
                }
            }
        }
    }
    
    static class ConnectionPool implements AutoCloseable {
        private final String configFile;
        private final List<DatabaseConnection> connections = new ArrayList<>();
        
        public ConnectionPool(String configFile) throws DatabaseException {
            this.configFile = configFile;
            initializePool();
        }
        
        private void initializePool() throws DatabaseException {
            System.out.printf("Initializing connection pool from: %s%n", configFile);
            for (int i = 0; i < 3; i++) {
                connections.add(new DatabaseConnection("conn-" + i));
            }
        }
        
        public DatabaseConnection getConnection() throws DatabaseException {
            if (connections.isEmpty()) {
                throw new DatabaseException("No connections available");
            }
            return connections.remove(0);
        }
        
        @Override
        public void close() throws DatabaseException {
            System.out.println("Closing connection pool");
            for (DatabaseConnection conn : connections) {
                conn.close();
            }
            connections.clear();
        }
    }
    
    static class DatabaseConnection implements AutoCloseable {
        private final String id;
        private boolean open = true;
        
        public DatabaseConnection(String id) {
            this.id = id;
            System.out.printf("Database connection opened: %s%n", id);
        }
        
        public PreparedStatement prepareStatement(String sql) throws SQLException {
            if (!open) {
                throw new SQLException("Connection closed");
            }
            return new PreparedStatement(sql);
        }
        
        @Override
        public void close() throws DatabaseException {
            if (open) {
                open = false;
                System.out.printf("Database connection closed: %s%n", id);
            }
        }
    }
    
    static class PreparedStatement implements AutoCloseable {
        private final String sql;
        
        public PreparedStatement(String sql) {
            this.sql = sql;
        }
        
        public ResultSetWrapper executeQuery() throws SQLException {
            System.out.printf("Executing: %s%n", sql);
            return new ResultSetWrapper();
        }
        
        @Override
        public void close() {
            System.out.println("PreparedStatement closed");
        }
    }
    
    static class ResultSetWrapper implements AutoCloseable {
        private int position = 0;
        private final String[] data = {"Product1", "Product2", "Product3"};
        
        public boolean next() {
            return position < data.length;
        }
        
        public String getString(String columnName) {
            return data[position++];
        }
        
        @Override
        public void close() {
            System.out.println("ResultSet closed");
        }
    }
    
    static class ProblematicResource implements AutoCloseable {
        public ProblematicResource() {
            System.out.println("ProblematicResource created");
        }
        
        public void doWork() {
            System.out.println("ProblematicResource doing work");
        }
        
        @Override
        public void close() throws Exception {
            System.out.println("ProblematicResource closing...");
            throw new RuntimeException("Error during resource cleanup");
        }
    }
    
    static class MockResource implements AutoCloseable {
        private final String name;
        
        public MockResource(String name) {
            this.name = name;
            System.out.printf("MockResource created: %s%n", name);
        }
        
        public String getName() {
            return name;
        }
        
        public void doWork() {
            System.out.printf("%s doing work%n", name);
        }
        
        @Override
        public void close() {
            System.out.printf("MockResource closed: %s%n", name);
        }
    }
    
    // Exception classes
    static class DatabaseException extends Exception {
        public DatabaseException(String message) {
            super(message);
        }
    }
    
    static class SQLException extends Exception {
        public SQLException(String message) {
            super(message);
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== try-with-resources Comprehensive Demo ===\n");
        
        demonstrateBasicTryWithResources();
        demonstrateMultipleResources();
        demonstrateCustomResource();
        demonstrateNestedTryWithResources();
        demonstrateExceptionSuppression();
        demonstrateJava9TryWithResources();
        demonstrateResourceLeakPrevention();
        
        System.out.println("\n=== try-with-resources Summary ===");
        System.out.println("✅ Automatic resource management");
        System.out.println("✅ Resources closed in reverse order");
        System.out.println("✅ Exception suppression preserves cleanup errors");
        System.out.println("✅ Cleaner, more readable code");
        System.out.println("✅ Prevents resource leaks");
        System.out.println("✅ Works with any AutoCloseable resource");
        System.out.println("✅ Java 9+ supports effectively final variables");
    }
}

/*
try-with-resources Benefits:

1. Automatic Resource Management:
   - Resources closed automatically
   - No need for explicit finally blocks
   - Prevents resource leaks

2. Exception Handling:
   - Suppressed exceptions from close() methods
   - Main exception preserved
   - Full exception chain available

3. Syntax:
   - try (Resource r = new Resource()) { }
   - Multiple resources separated by semicolons
   - Java 9+: effectively final variables

4. Best Practices:
   - Use for all AutoCloseable resources
   - Prefer over manual resource management
   - Resources closed in reverse order of declaration
   - Combine with regular exception handling

Resource Types:
- File I/O streams
- Database connections
- Network sockets
- Custom AutoCloseable implementations
*/
```

## 📊 Exception Handling Execution Flow

| Scenario | try | catch | finally | Result |
|----------|-----|-------|---------|--------|
| **No Exception** | ✅ Executes | ❌ Skipped | ✅ Executes | Normal flow |
| **Caught Exception** | ⚠️ Partial | ✅ Executes | ✅ Executes | Handled flow |
| **Uncaught Exception** | ⚠️ Partial | ❌ Skipped | ✅ Executes | Exception propagated |
| **Return in try** | ✅ Executes | ❌ Skipped | ✅ Executes | Return after finally |
| **Exception in finally** | ✅ Executes | ✅ Executes | ⚠️ Throws | finally exception wins |

## ⚠️ Common try-catch-finally Mistakes

### 1. Empty catch blocks
```java
// ❌ Bad - silently ignores errors
try {
    riskyOperation();
} catch (Exception e) {
    // Empty - error information lost
}

// ✅ Good - at minimum log the error
try {
    riskyOperation();
} catch (Exception e) {
    logger.error("Operation failed", e);
}
```

### 2. Resource leaks in finally
```java
// ❌ Bad - resource leak if close() throws
finally {
    resource.close(); // May throw exception
}

// ✅ Good - handle close() exceptions
finally {
    if (resource != null) {
        try {
            resource.close();
        } catch (IOException e) {
            logger.warn("Failed to close resource", e);
        }
    }
}
```

### 3. Wrong catch order
```java
// ❌ Bad - specific exception unreachable
try {
    operation();
} catch (Exception e) {
    // Generic catch
} catch (IOException e) { // Unreachable!
    // Specific catch
}

// ✅ Good - specific first, general last
try {
    operation();
} catch (IOException e) {
    // Specific catch
} catch (Exception e) {
    // Generic catch
}
```

## 🔥 Interview Questions & Answers

### Q1: What's the difference between try-catch-finally and try-with-resources?
**Answer**: 
- **try-catch-finally**: Manual resource management, requires explicit cleanup in finally block
- **try-with-resources**: Automatic resource management for AutoCloseable resources
- **Code clarity**: try-with-resources is cleaner and less error-prone
- **Exception handling**: try-with-resources handles suppressed exceptions from close() methods
- **Java version**: try-with-resources available from Java 7+

### Q2: Can finally block prevent exception propagation?
**Answer**: **Yes**, in two ways:
- **Return statement**: finally block with return statement prevents exception propagation
- **Throwing exception**: finally block throwing exception suppresses original exception
- **Best practice**: Avoid return statements and throwing exceptions in finally blocks
- **Suppressed exceptions**: Use exception.addSuppressed() to preserve cleanup exceptions

### Q3: What happens if both try and finally blocks throw exceptions?
**Answer**: 
- **finally wins**: Exception from finally block suppresses try block exception
- **Original lost**: try block exception becomes suppressed exception
- **Access suppressed**: Use exception.getSuppressed() to access original exception
- **Best practice**: Handle finally block exceptions locally to preserve original

### Q4: Can you have try without catch or finally?
**Answer**: 
- **try-catch**: Valid, no finally needed
- **try-finally**: Valid, no catch needed (for cleanup only)
- **try alone**: Invalid, must have catch or finally
- **try-with-resources**: Valid without catch or finally (resources auto-closed)

### Q5: How does multi-catch work and what are its limitations?
**Answer**: 
- **Syntax**: catch (Exception1 | Exception2 e) { }
- **Same handling**: All exception types handled the same way
- **Type restrictions**: Exception types cannot be subclasses of each other
- **Variable type**: Exception variable is effectively final
- **Benefits**: Reduces code duplication for similar exception handling

---
[← Back to Main Guide](./README.md) | [← Previous: Exception Hierarchy](./exception-hierarchy.md) | [Next: throw vs throws →](./throw-vs-throws.md)
