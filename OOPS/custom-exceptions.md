# 🎨 Custom Exceptions in Java - User-Defined Exception Classes

Custom exceptions in Java allow developers to create domain-specific error types that provide meaningful context about application-specific problems. They enable better error handling, clearer debugging, and more maintainable code by representing business logic violations and application-specific error conditions.

## 🎯 Key Concepts

- **Custom Exception Classes**: User-defined exception types extending Exception or RuntimeException
- **Checked vs Unchecked**: Choice between compile-time and runtime exception handling
- **Exception Hierarchy**: Creating meaningful inheritance structures for exceptions
- **Context Information**: Adding domain-specific data to exceptions
- **Exception Chaining**: Preserving root causes while adding context
- **Best Practices**: Design patterns for effective custom exceptions

## 🌍 Real-World Analogy

**Hospital Emergency System Analogy**:
- **Custom Exceptions** = Specialized medical protocols for different conditions
- **Checked Exceptions** = Scheduled procedures requiring preparation and documentation
- **Unchecked Exceptions** = Emergency situations requiring immediate response
- **Exception Hierarchy** = Medical department specializations (Cardiology, Neurology, etc.)
- **Context Information** = Patient medical history, symptoms, and vital signs
- **Exception Chaining** = Referring patient from general practice → specialist → surgeon

## 📋 Custom Exception Design Patterns

```
Custom Exception Hierarchy Design:
┌─────────────────────────────────────────────────────────────────┐
│                   Application Exception Hierarchy              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                    java.lang.Exception                         │
│                           │                                     │
│              ┌────────────┴────────────┐                       │
│              │                         │                       │
│    BaseApplicationException    java.lang.RuntimeException      │
│              │                         │                       │
│    ┌─────────┼─────────┐              │                       │
│    │         │         │              │                       │
│ Business   Data     Config        Validation                   │
│Exception  Exception Exception     Exception                    │
│    │         │         │              │                       │
│ ┌──┴──┐   ┌─┴─┐    ┌──┴──┐        ┌──┴──┐                    │
│ │Order│   │DB │    │File │        │Input│                    │
│ │Proc │   │Conn│    │Load │        │Val  │                    │
│ │Exc  │   │Exc │    │Exc  │        │Exc  │                    │
│ └─────┘   └───┘    └─────┘        └─────┘                    │
└─────────────────────────────────────────────────────────────────┘

Exception Information Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                Custom Exception Components                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                Exception Message                        │   │
│  │  "Human-readable description of the error"             │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                Error Code                               │   │
│  │  "BUSINESS_001", "DATA_ACCESS_002", "VALIDATION_003"   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                Context Data                             │   │
│  │  userId: "12345", operation: "transfer",               │   │
│  │  amount: 1000.00, timestamp: "2023-08-27T10:30:00"    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                Root Cause                               │   │
│  │  Original exception that caused this exception         │   │
│  │  (SQLException, IOException, etc.)                     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                Stack Trace                              │   │
│  │  Method call sequence leading to the exception         │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘

Exception Design Checklist:
┌─────────────────────────────────────────────────────────────────┐
│                 Custom Exception Design Guide                  │
├─────────────────────────────────────────────────────────────────┤
│  ✅ Choose appropriate base class (Exception vs RuntimeException)│
│  ✅ Follow consistent naming convention (ending with Exception) │
│  ✅ Provide multiple constructors for flexibility              │
│  ✅ Include meaningful error messages                          │
│  ✅ Add error codes for programmatic handling                  │
│  ✅ Include context information for debugging                  │
│  ✅ Support exception chaining for root cause analysis        │
│  ✅ Override toString() for better debugging output            │
│  ✅ Document when and how to use the exception                │
│  ✅ Consider serialization if exceptions cross JVM boundaries  │
└─────────────────────────────────────────────────────────────────┘
```

## 💻 Practical Examples

### Example 1: Basic Custom Exception Design

```java
// BasicCustomExceptions.java - Fundamental custom exception patterns
import java.time.LocalDateTime;
import java.util.*;

public class BasicCustomExceptions {
    
    // Basic checked custom exception
    static class BusinessException extends Exception {
        private final String errorCode;
        private final LocalDateTime timestamp;
        
        public BusinessException(String message) {
            super(message);
            this.errorCode = "BUSINESS_ERROR";
            this.timestamp = LocalDateTime.now();
        }
        
        public BusinessException(String message, String errorCode) {
            super(message);
            this.errorCode = errorCode;
            this.timestamp = LocalDateTime.now();
        }
        
        public BusinessException(String message, Throwable cause) {
            super(message, cause);
            this.errorCode = "BUSINESS_ERROR";
            this.timestamp = LocalDateTime.now();
        }
        
        public BusinessException(String message, String errorCode, Throwable cause) {
            super(message, cause);
            this.errorCode = errorCode;
            this.timestamp = LocalDateTime.now();
        }
        
        public String getErrorCode() {
            return errorCode;
        }
        
        public LocalDateTime getTimestamp() {
            return timestamp;
        }
        
        @Override
        public String toString() {
            return String.format("%s{errorCode='%s', message='%s', timestamp=%s}", 
                    getClass().getSimpleName(), errorCode, getMessage(), timestamp);
        }
    }
    
    // Basic unchecked custom exception
    static class ValidationException extends RuntimeException {
        private final String fieldName;
        private final Object invalidValue;
        private final String validationRule;
        
        public ValidationException(String fieldName, Object invalidValue, String validationRule) {
            super(String.format("Validation failed for field '%s' with value '%s': %s", 
                    fieldName, invalidValue, validationRule));
            this.fieldName = fieldName;
            this.invalidValue = invalidValue;
            this.validationRule = validationRule;
        }
        
        public ValidationException(String fieldName, Object invalidValue, 
                                 String validationRule, Throwable cause) {
            super(String.format("Validation failed for field '%s' with value '%s': %s", 
                    fieldName, invalidValue, validationRule), cause);
            this.fieldName = fieldName;
            this.invalidValue = invalidValue;
            this.validationRule = validationRule;
        }
        
        public String getFieldName() { return fieldName; }
        public Object getInvalidValue() { return invalidValue; }
        public String getValidationRule() { return validationRule; }
        
        @Override
        public String toString() {
            return String.format("ValidationException{field='%s', value='%s', rule='%s', message='%s'}", 
                    fieldName, invalidValue, validationRule, getMessage());
        }
    }
    
    // Custom exception with multiple error details
    static class OrderProcessingException extends Exception {
        private final String orderId;
        private final String customerId;
        private final ProcessingStage failedStage;
        private final Map<String, Object> contextData;
        
        public enum ProcessingStage {
            VALIDATION, PAYMENT, INVENTORY, SHIPPING, CONFIRMATION
        }
        
        public OrderProcessingException(String orderId, String customerId, 
                                      ProcessingStage failedStage, String message) {
            super(message);
            this.orderId = orderId;
            this.customerId = customerId;
            this.failedStage = failedStage;
            this.contextData = new HashMap<>();
        }
        
        public OrderProcessingException(String orderId, String customerId, 
                                      ProcessingStage failedStage, String message, Throwable cause) {
            super(message, cause);
            this.orderId = orderId;
            this.customerId = customerId;
            this.failedStage = failedStage;
            this.contextData = new HashMap<>();
        }
        
        public OrderProcessingException addContext(String key, Object value) {
            contextData.put(key, value);
            return this; // Method chaining
        }
        
        public String getOrderId() { return orderId; }
        public String getCustomerId() { return customerId; }
        public ProcessingStage getFailedStage() { return failedStage; }
        public Map<String, Object> getContextData() { return new HashMap<>(contextData); }
        
        @Override
        public String toString() {
            return String.format("OrderProcessingException{orderId='%s', customerId='%s', " +
                    "failedStage=%s, message='%s', context=%s}", 
                    orderId, customerId, failedStage, getMessage(), contextData);
        }
    }
    
    // Demonstration of custom exception usage
    public static void demonstrateBasicCustomExceptions() {
        System.out.println("=== Basic Custom Exceptions ===");
        
        // Business exception example
        try {
            validateBusinessRule("INVALID_OPERATION");
        } catch (BusinessException e) {
            System.out.printf("Business error occurred: %s%n", e);
            System.out.printf("Error code: %s%n", e.getErrorCode());
            System.out.printf("Timestamp: %s%n", e.getTimestamp());
        }
        
        // Validation exception example
        try {
            validateUserInput("", "username");
        } catch (ValidationException e) {
            System.out.printf("Validation error: %s%n", e);
            System.out.printf("Field: %s, Value: %s, Rule: %s%n", 
                    e.getFieldName(), e.getInvalidValue(), e.getValidationRule());
        }
        
        // Order processing exception example
        try {
            processOrder("ORD-001", "CUST-123");
        } catch (OrderProcessingException e) {
            System.out.printf("Order processing failed: %s%n", e);
            System.out.printf("Failed at stage: %s%n", e.getFailedStage());
            System.out.printf("Context data: %s%n", e.getContextData());
        }
    }
    
    private static void validateBusinessRule(String operation) throws BusinessException {
        if ("INVALID_OPERATION".equals(operation)) {
            throw new BusinessException("Operation not allowed in current business context", "BUS001");
        }
    }
    
    private static void validateUserInput(String value, String fieldName) {
        if (value == null || value.trim().isEmpty()) {
            throw new ValidationException(fieldName, value, "Field cannot be empty");
        }
    }
    
    private static void processOrder(String orderId, String customerId) throws OrderProcessingException {
        try {
            // Simulate order processing
            if ("CUST-123".equals(customerId)) {
                throw new OrderProcessingException(orderId, customerId, 
                        OrderProcessingException.ProcessingStage.PAYMENT, 
                        "Payment method declined")
                        .addContext("paymentMethod", "CREDIT_CARD")
                        .addContext("amount", 299.99)
                        .addContext("attemptNumber", 1);
            }
        } catch (Exception e) {
            if (e instanceof OrderProcessingException) {
                throw e;
            }
            throw new OrderProcessingException(orderId, customerId, 
                    OrderProcessingException.ProcessingStage.VALIDATION, 
                    "Unexpected error during order processing", e);
        }
    }
    
    public static void main(String[] args) {
        demonstrateBasicCustomExceptions();
        
        System.out.println("\n=== Basic Custom Exception Benefits ===");
        System.out.println("✅ Domain-specific error information");
        System.out.println("✅ Better debugging with context data");
        System.out.println("✅ Programmatic error handling with error codes");
        System.out.println("✅ Flexible constructor patterns");
        System.out.println("✅ Method chaining for context building");
    }
}

/*
Basic Custom Exception Design Principles:

1. Naming Convention:
   - End with "Exception" suffix
   - Describe the error condition clearly
   - Use domain-specific terminology

2. Constructor Patterns:
   - Message only
   - Message + error code
   - Message + cause
   - Message + error code + cause

3. Additional Information:
   - Error codes for programmatic handling
   - Timestamps for logging
   - Context data for debugging
   - Domain-specific fields

4. Method Chaining:
   - Fluent interface for adding context
   - Return 'this' from setter methods
   - Enable readable exception creation

5. toString() Override:
   - Include all relevant information
   - Format for easy reading
   - Help with debugging and logging
*/
```

### Example 2: Advanced Custom Exception Architecture

```java
// AdvancedCustomExceptions.java - Enterprise-level exception design
import java.time.LocalDateTime;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

public class AdvancedCustomExceptions {
    
    // Base application exception with rich context
    abstract static class BaseApplicationException extends Exception {
        private final String errorCode;
        private final ErrorSeverity severity;
        private final LocalDateTime timestamp;
        private final Map<String, Object> contextData;
        private final String correlationId;
        
        public enum ErrorSeverity {
            LOW, MEDIUM, HIGH, CRITICAL
        }
        
        protected BaseApplicationException(String message, String errorCode, ErrorSeverity severity) {
            super(message);
            this.errorCode = errorCode;
            this.severity = severity;
            this.timestamp = LocalDateTime.now();
            this.contextData = new ConcurrentHashMap<>();
            this.correlationId = generateCorrelationId();
        }
        
        protected BaseApplicationException(String message, String errorCode, 
                                         ErrorSeverity severity, Throwable cause) {
            super(message, cause);
            this.errorCode = errorCode;
            this.severity = severity;
            this.timestamp = LocalDateTime.now();
            this.contextData = new ConcurrentHashMap<>();
            this.correlationId = generateCorrelationId();
        }
        
        public BaseApplicationException addContext(String key, Object value) {
            contextData.put(key, value);
            return this;
        }
        
        public BaseApplicationException addContextMap(Map<String, Object> context) {
            contextData.putAll(context);
            return this;
        }
        
        private String generateCorrelationId() {
            return "ERR-" + System.currentTimeMillis() + "-" + Thread.currentThread().getId();
        }
        
        // Getters
        public String getErrorCode() { return errorCode; }
        public ErrorSeverity getSeverity() { return severity; }
        public LocalDateTime getTimestamp() { return timestamp; }
        public Map<String, Object> getContextData() { return new HashMap<>(contextData); }
        public String getCorrelationId() { return correlationId; }
        
        // Abstract method for exception category
        public abstract String getCategory();
        
        // Utility methods
        public boolean isCritical() {
            return severity == ErrorSeverity.CRITICAL || severity == ErrorSeverity.HIGH;
        }
        
        public String getContextAsString() {
            return contextData.entrySet().stream()
                    .map(entry -> entry.getKey() + "=" + entry.getValue())
                    .reduce((a, b) -> a + ", " + b)
                    .orElse("No context");
        }
        
        @Override
        public String toString() {
            return String.format("%s{category='%s', errorCode='%s', severity=%s, " +
                    "correlationId='%s', message='%s', timestamp=%s, context=[%s]}", 
                    getClass().getSimpleName(), getCategory(), errorCode, severity, 
                    correlationId, getMessage(), timestamp, getContextAsString());
        }
    }
    
    // Business logic exceptions
    static class BusinessLogicException extends BaseApplicationException {
        private final String businessRule;
        private final String entityType;
        private final String entityId;
        
        public BusinessLogicException(String businessRule, String entityType, String entityId, 
                                    String message, String errorCode) {
            super(message, errorCode, ErrorSeverity.HIGH);
            this.businessRule = businessRule;
            this.entityType = entityType;
            this.entityId = entityId;
            addContext("businessRule", businessRule);
            addContext("entityType", entityType);
            addContext("entityId", entityId);
        }
        
        public BusinessLogicException(String businessRule, String entityType, String entityId, 
                                    String message, String errorCode, Throwable cause) {
            super(message, errorCode, ErrorSeverity.HIGH, cause);
            this.businessRule = businessRule;
            this.entityType = entityType;
            this.entityId = entityId;
            addContext("businessRule", businessRule);
            addContext("entityType", entityType);
            addContext("entityId", entityId);
        }
        
        @Override
        public String getCategory() {
            return "BUSINESS_LOGIC";
        }
        
        public String getBusinessRule() { return businessRule; }
        public String getEntityType() { return entityType; }
        public String getEntityId() { return entityId; }
    }
    
    // Data access exceptions
    static class DataAccessException extends BaseApplicationException {
        private final String operation;
        private final String dataSource;
        private final String query;
        
        public DataAccessException(String operation, String dataSource, String message, String errorCode) {
            super(message, errorCode, ErrorSeverity.MEDIUM);
            this.operation = operation;
            this.dataSource = dataSource;
            this.query = null;
            addContext("operation", operation);
            addContext("dataSource", dataSource);
        }
        
        public DataAccessException(String operation, String dataSource, String query, 
                                 String message, String errorCode, Throwable cause) {
            super(message, errorCode, ErrorSeverity.MEDIUM, cause);
            this.operation = operation;
            this.dataSource = dataSource;
            this.query = query;
            addContext("operation", operation);
            addContext("dataSource", dataSource);
            addContext("query", query);
        }
        
        @Override
        public String getCategory() {
            return "DATA_ACCESS";
        }
        
        public String getOperation() { return operation; }
        public String getDataSource() { return dataSource; }
        public String getQuery() { return query; }
    }
    
    // Security exceptions
    static class SecurityException extends BaseApplicationException {
        private final String userId;
        private final String resource;
        private final String action;
        private final String reason;
        
        public SecurityException(String userId, String resource, String action, 
                               String reason, String errorCode) {
            super(String.format("Security violation: %s", reason), errorCode, ErrorSeverity.CRITICAL);
            this.userId = userId;
            this.resource = resource;
            this.action = action;
            this.reason = reason;
            addContext("userId", userId);
            addContext("resource", resource);
            addContext("action", action);
            addContext("reason", reason);
            addContext("remoteIP", getClientIP());
        }
        
        private String getClientIP() {
            // In real application, would get from request context
            return "192.168.1.100";
        }
        
        @Override
        public String getCategory() {
            return "SECURITY";
        }
        
        public String getUserId() { return userId; }
        public String getResource() { return resource; }
        public String getAction() { return action; }
        public String getReason() { return reason; }
    }
    
    // Configuration exceptions
    static class ConfigurationException extends BaseApplicationException {
        private final String configKey;
        private final String configValue;
        private final String expectedFormat;
        private final String configSource;
        
        public ConfigurationException(String configKey, String configValue, 
                                    String expectedFormat, String configSource, String errorCode) {
            super(String.format("Configuration error for key '%s': expected %s, got '%s'", 
                    configKey, expectedFormat, configValue), errorCode, ErrorSeverity.HIGH);
            this.configKey = configKey;
            this.configValue = configValue;
            this.expectedFormat = expectedFormat;
            this.configSource = configSource;
            addContext("configKey", configKey);
            addContext("configValue", configValue);
            addContext("expectedFormat", expectedFormat);
            addContext("configSource", configSource);
        }
        
        @Override
        public String getCategory() {
            return "CONFIGURATION";
        }
        
        public String getConfigKey() { return configKey; }
        public String getConfigValue() { return configValue; }
        public String getExpectedFormat() { return expectedFormat; }
        public String getConfigSource() { return configSource; }
    }
    
    // Exception with retry information
    static class RetryableException extends BaseApplicationException {
        private final int attemptNumber;
        private final int maxAttempts;
        private final long retryDelayMs;
        private final boolean canRetry;
        
        public RetryableException(String message, String errorCode, int attemptNumber, 
                                int maxAttempts, long retryDelayMs) {
            super(message, errorCode, ErrorSeverity.MEDIUM);
            this.attemptNumber = attemptNumber;
            this.maxAttempts = maxAttempts;
            this.retryDelayMs = retryDelayMs;
            this.canRetry = attemptNumber < maxAttempts;
            addContext("attemptNumber", attemptNumber);
            addContext("maxAttempts", maxAttempts);
            addContext("retryDelayMs", retryDelayMs);
            addContext("canRetry", canRetry);
        }
        
        public RetryableException(String message, String errorCode, int attemptNumber, 
                                int maxAttempts, long retryDelayMs, Throwable cause) {
            super(message, errorCode, ErrorSeverity.MEDIUM, cause);
            this.attemptNumber = attemptNumber;
            this.maxAttempts = maxAttempts;
            this.retryDelayMs = retryDelayMs;
            this.canRetry = attemptNumber < maxAttempts;
            addContext("attemptNumber", attemptNumber);
            addContext("maxAttempts", maxAttempts);
            addContext("retryDelayMs", retryDelayMs);
            addContext("canRetry", canRetry);
        }
        
        @Override
        public String getCategory() {
            return "RETRYABLE_OPERATION";
        }
        
        public int getAttemptNumber() { return attemptNumber; }
        public int getMaxAttempts() { return maxAttempts; }
        public long getRetryDelayMs() { return retryDelayMs; }
        public boolean canRetry() { return canRetry; }
        
        public RetryableException nextAttempt() {
            return new RetryableException(getMessage(), getErrorCode(), 
                    attemptNumber + 1, maxAttempts, retryDelayMs * 2, getCause());
        }
    }
    
    // Exception handler utility
    static class ExceptionHandler {
        private static final Map<String, Integer> errorCounts = new ConcurrentHashMap<>();
        
        public static void handleException(BaseApplicationException exception) {
            // Log exception
            logException(exception);
            
            // Track error counts
            trackErrorCount(exception.getErrorCode());
            
            // Send alerts for critical errors
            if (exception.isCritical()) {
                sendAlert(exception);
            }
            
            // Handle specific exception types
            if (exception instanceof SecurityException) {
                handleSecurityException((SecurityException) exception);
            } else if (exception instanceof RetryableException) {
                handleRetryableException((RetryableException) exception);
            }
        }
        
        private static void logException(BaseApplicationException exception) {
            System.out.printf("[%s] %s - %s%n", 
                    exception.getTimestamp(), exception.getSeverity(), exception);
        }
        
        private static void trackErrorCount(String errorCode) {
            errorCounts.merge(errorCode, 1, Integer::sum);
        }
        
        private static void sendAlert(BaseApplicationException exception) {
            System.out.printf("🚨 CRITICAL ALERT: %s - %s%n", 
                    exception.getErrorCode(), exception.getMessage());
        }
        
        private static void handleSecurityException(SecurityException secException) {
            System.out.printf("🔒 Security breach detected: User %s attempted %s on %s%n", 
                    secException.getUserId(), secException.getAction(), secException.getResource());
        }
        
        private static void handleRetryableException(RetryableException retryException) {
            if (retryException.canRetry()) {
                System.out.printf("🔄 Retry possible: Attempt %d/%d, next retry in %dms%n", 
                        retryException.getAttemptNumber(), retryException.getMaxAttempts(), 
                        retryException.getRetryDelayMs());
            } else {
                System.out.printf("❌ Max retries exceeded: %d/%d attempts failed%n", 
                        retryException.getAttemptNumber(), retryException.getMaxAttempts());
            }
        }
        
        public static Map<String, Integer> getErrorStatistics() {
            return new HashMap<>(errorCounts);
        }
    }
    
    // Service demonstrating advanced exception usage
    static class AdvancedService {
        
        public void processUserTransaction(String userId, String transactionId, double amount) 
                throws BusinessLogicException, DataAccessException, SecurityException {
            
            try {
                // Security validation
                validateUserPermissions(userId, "TRANSACTION", "CREATE");
                
                // Business logic validation
                validateTransactionRules(userId, amount);
                
                // Data access
                saveTransaction(transactionId, userId, amount);
                
            } catch (SecurityException e) {
                throw e; // Re-throw security exceptions as-is
            } catch (Exception e) {
                throw new DataAccessException("SAVE", "transaction_db", 
                        "INSERT INTO transactions...", 
                        "Failed to save transaction", "DAT001", e)
                        .addContext("userId", userId)
                        .addContext("transactionId", transactionId)
                        .addContext("amount", amount);
            }
        }
        
        private void validateUserPermissions(String userId, String resource, String action) 
                throws SecurityException {
            if ("blocked_user".equals(userId)) {
                throw new SecurityException(userId, resource, action, 
                        "User account is blocked", "SEC001");
            }
        }
        
        private void validateTransactionRules(String userId, double amount) 
                throws BusinessLogicException {
            if (amount > 10000) {
                throw new BusinessLogicException("MAX_TRANSACTION_AMOUNT", "Transaction", 
                        "TXN-" + System.currentTimeMillis(), 
                        "Transaction amount exceeds daily limit", "BUS001")
                        .addContext("dailyLimit", 10000)
                        .addContext("requestedAmount", amount);
            }
        }
        
        private void saveTransaction(String transactionId, String userId, double amount) {
            // Simulate database save
            if (Math.random() > 0.8) {
                throw new RuntimeException("Database connection timeout");
            }
        }
        
        public void processWithRetry(String operationId) throws RetryableException {
            RetryableException lastException = null;
            
            for (int attempt = 1; attempt <= 3; attempt++) {
                try {
                    performOperation(operationId, attempt);
                    return; // Success
                } catch (Exception e) {
                    lastException = new RetryableException(
                            "Operation failed on attempt " + attempt, 
                            "RET001", attempt, 3, 1000 * attempt, e);
                    
                    if (lastException.canRetry()) {
                        System.out.printf("Retrying operation after %dms...%n", 
                                lastException.getRetryDelayMs());
                        try {
                            Thread.sleep(lastException.getRetryDelayMs());
                        } catch (InterruptedException ie) {
                            Thread.currentThread().interrupt();
                            throw lastException;
                        }
                    }
                }
            }
            
            throw lastException; // All retries exhausted
        }
        
        private void performOperation(String operationId, int attempt) {
            if (attempt < 3) {
                throw new RuntimeException("Simulated operation failure on attempt " + attempt);
            }
        }
    }
    
    public static void demonstrateAdvancedExceptions() {
        System.out.println("=== Advanced Custom Exceptions ===");
        
        AdvancedService service = new AdvancedService();
        
        // Business logic exception
        try {
            service.processUserTransaction("user123", "txn456", 15000);
        } catch (BaseApplicationException e) {
            ExceptionHandler.handleException(e);
        }
        
        // Security exception
        try {
            service.processUserTransaction("blocked_user", "txn789", 500);
        } catch (BaseApplicationException e) {
            ExceptionHandler.handleException(e);
        }
        
        // Retryable exception
        try {
            service.processWithRetry("op123");
        } catch (RetryableException e) {
            ExceptionHandler.handleException(e);
        }
        
        // Configuration exception
        try {
            throw new ConfigurationException("database.timeout", "abc", 
                    "positive integer", "application.properties", "CFG001");
        } catch (BaseApplicationException e) {
            ExceptionHandler.handleException(e);
        }
        
        // Show error statistics
        System.out.println("\n=== Error Statistics ===");
        ExceptionHandler.getErrorStatistics().forEach((code, count) -> 
                System.out.printf("Error %s: %d occurrences%n", code, count));
    }
    
    public static void main(String[] args) {
        demonstrateAdvancedExceptions();
        
        System.out.println("\n=== Advanced Custom Exception Features ===");
        System.out.println("✅ Rich context data with correlation IDs");
        System.out.println("✅ Severity levels and categorization");
        System.out.println("✅ Automatic error tracking and statistics");
        System.out.println("✅ Retry logic with backoff strategies");
        System.out.println("✅ Security breach detection and alerting");
        System.out.println("✅ Method chaining for fluent exception building");
        System.out.println("✅ Comprehensive logging and monitoring support");
    }
}

/*
Advanced Custom Exception Architecture:

1. Base Exception Class:
   - Common fields: error code, severity, timestamp, correlation ID
   - Context data management with thread-safe maps
   - Abstract methods for categorization
   - Utility methods for common operations

2. Exception Specialization:
   - Domain-specific exception types
   - Specialized fields for each exception type
   - Context-aware error messages
   - Category-specific handling logic

3. Exception Handling Infrastructure:
   - Centralized exception handler
   - Error tracking and statistics
   - Severity-based alerting
   - Correlation ID for request tracing

4. Advanced Features:
   - Retry logic with exponential backoff
   - Security exception monitoring
   - Configuration validation
   - Business rule violation tracking

5. Best Practices:
   - Thread-safe context management
   - Fluent interface for exception building
   - Comprehensive logging support
   - Performance monitoring integration
*/
```

### Example 3: Custom Exception Best Practices and Patterns

```java
// CustomExceptionBestPractices.java - Production-ready exception patterns
import java.io.Serializable;
import java.time.LocalDateTime;
import java.util.*;
import java.util.function.Supplier;
import java.util.stream.Collectors;

public class CustomExceptionBestPractices {
    
    // Exception factory for consistent exception creation
    static class ExceptionFactory {
        
        public static ValidationException createValidationException(String field, Object value, String rule) {
            return new ValidationException(field, value, rule)
                    .addContext("validationTimestamp", LocalDateTime.now())
                    .addContext("validatorVersion", "1.0.0");
        }
        
        public static BusinessRuleException createBusinessRuleException(String rule, Object entity, String message) {
            return new BusinessRuleException(rule, entity, message)
                    .addContext("businessContext", getCurrentBusinessContext())
                    .addContext("ruleEngine", "v2.1.0");
        }
        
        public static DataIntegrityException createDataIntegrityException(String operation, String entity, String constraint) {
            return new DataIntegrityException(operation, entity, constraint)
                    .addContext("databaseVersion", "PostgreSQL 13.4")
                    .addContext("schema", "production");
        }
        
        private static Map<String, Object> getCurrentBusinessContext() {
            Map<String, Object> context = new HashMap<>();
            context.put("tenant", "tenant123");
            context.put("region", "us-east-1");
            context.put("environment", "production");
            return context;
        }
    }
    
    // Serializable custom exception for distributed systems
    static class DistributedSystemException extends Exception implements Serializable {
        private static final long serialVersionUID = 1L;
        
        private final String serviceId;
        private final String requestId;
        private final String nodeId;
        private final Map<String, String> serviceContext;
        
        public DistributedSystemException(String serviceId, String requestId, String nodeId, String message) {
            super(message);
            this.serviceId = serviceId;
            this.requestId = requestId;
            this.nodeId = nodeId;
            this.serviceContext = new HashMap<>();
        }
        
        public DistributedSystemException addServiceContext(String key, String value) {
            serviceContext.put(key, value);
            return this;
        }
        
        public String getServiceId() { return serviceId; }
        public String getRequestId() { return requestId; }
        public String getNodeId() { return nodeId; }
        public Map<String, String> getServiceContext() { return new HashMap<>(serviceContext); }
        
        @Override
        public String toString() {
            return String.format("DistributedSystemException{service='%s', request='%s', node='%s', message='%s', context=%s}", 
                    serviceId, requestId, nodeId, getMessage(), serviceContext);
        }
    }
    
    // Exception with builder pattern
    static class ComplexBusinessException extends Exception {
        private final String errorCode;
        private final Priority priority;
        private final String businessDomain;
        private final String affectedEntity;
        private final Map<String, Object> metadata;
        private final List<String> suggestedActions;
        
        public enum Priority {
            LOW(1), MEDIUM(2), HIGH(3), CRITICAL(4);
            
            private final int level;
            Priority(int level) { this.level = level; }
            public int getLevel() { return level; }
        }
        
        private ComplexBusinessException(Builder builder) {
            super(builder.message, builder.cause);
            this.errorCode = builder.errorCode;
            this.priority = builder.priority;
            this.businessDomain = builder.businessDomain;
            this.affectedEntity = builder.affectedEntity;
            this.metadata = new HashMap<>(builder.metadata);
            this.suggestedActions = new ArrayList<>(builder.suggestedActions);
        }
        
        public static class Builder {
            private String message;
            private String errorCode;
            private Priority priority = Priority.MEDIUM;
            private String businessDomain;
            private String affectedEntity;
            private Throwable cause;
            private final Map<String, Object> metadata = new HashMap<>();
            private final List<String> suggestedActions = new ArrayList<>();
            
            public Builder(String message, String errorCode) {
                this.message = message;
                this.errorCode = errorCode;
            }
            
            public Builder priority(Priority priority) {
                this.priority = priority;
                return this;
            }
            
            public Builder businessDomain(String domain) {
                this.businessDomain = domain;
                return this;
            }
            
            public Builder affectedEntity(String entity) {
                this.affectedEntity = entity;
                return this;
            }
            
            public Builder cause(Throwable cause) {
                this.cause = cause;
                return this;
            }
            
            public Builder addMetadata(String key, Object value) {
                this.metadata.put(key, value);
                return this;
            }
            
            public Builder addSuggestedAction(String action) {
                this.suggestedActions.add(action);
                return this;
            }
            
            public ComplexBusinessException build() {
                return new ComplexBusinessException(this);
            }
        }
        
        // Getters
        public String getErrorCode() { return errorCode; }
        public Priority getPriority() { return priority; }
        public String getBusinessDomain() { return businessDomain; }
        public String getAffectedEntity() { return affectedEntity; }
        public Map<String, Object> getMetadata() { return new HashMap<>(metadata); }
        public List<String> getSuggestedActions() { return new ArrayList<>(suggestedActions); }
        
        @Override
        public String toString() {
            return String.format("ComplexBusinessException{code='%s', priority=%s, domain='%s', " +
                    "entity='%s', message='%s', metadata=%s, actions=%s}", 
                    errorCode, priority, businessDomain, affectedEntity, getMessage(), 
                    metadata, suggestedActions);
        }
    }
    
    // Exception with functional interfaces for lazy evaluation
    static class LazyEvaluationException extends RuntimeException {
        private final Supplier<String> messageSupplier;
        private final Supplier<Map<String, Object>> contextSupplier;
        private String evaluatedMessage;
        private Map<String, Object> evaluatedContext;
        
        public LazyEvaluationException(Supplier<String> messageSupplier, 
                                     Supplier<Map<String, Object>> contextSupplier) {
            this.messageSupplier = messageSupplier;
            this.contextSupplier = contextSupplier;
        }
        
        @Override
        public String getMessage() {
            if (evaluatedMessage == null) {
                evaluatedMessage = messageSupplier.get();
            }
            return evaluatedMessage;
        }
        
        public Map<String, Object> getContext() {
            if (evaluatedContext == null) {
                evaluatedContext = contextSupplier.get();
            }
            return new HashMap<>(evaluatedContext);
        }
        
        @Override
        public String toString() {
            return String.format("LazyEvaluationException{message='%s', context=%s}", 
                    getMessage(), getContext());
        }
    }
    
    // Exception aggregator for multiple errors
    static class AggregateException extends Exception {
        private final List<Exception> innerExceptions;
        private final String operationName;
        
        public AggregateException(String operationName, List<Exception> innerExceptions) {
            super(createAggregateMessage(operationName, innerExceptions));
            this.operationName = operationName;
            this.innerExceptions = new ArrayList<>(innerExceptions);
        }
        
        private static String createAggregateMessage(String operationName, List<Exception> exceptions) {
            return String.format("Operation '%s' failed with %d errors: %s", 
                    operationName, 
                    exceptions.size(),
                    exceptions.stream()
                            .map(Throwable::getMessage)
                            .collect(Collectors.joining("; ")));
        }
        
        public List<Exception> getInnerExceptions() {
            return new ArrayList<>(innerExceptions);
        }
        
        public String getOperationName() {
            return operationName;
        }
        
        public List<Exception> getExceptionsOfType(Class<? extends Exception> exceptionType) {
            return innerExceptions.stream()
                    .filter(exceptionType::isInstance)
                    .collect(Collectors.toList());
        }
        
        public boolean hasExceptionOfType(Class<? extends Exception> exceptionType) {
            return innerExceptions.stream().anyMatch(exceptionType::isInstance);
        }
        
        @Override
        public void printStackTrace() {
            super.printStackTrace();
            System.out.println("Inner exceptions:");
            for (int i = 0; i < innerExceptions.size(); i++) {
                System.out.printf("  [%d] %s\n", i + 1, innerExceptions.get(i).getMessage());
            }
        }
    }
    
    // Basic exception types used in examples
    static class ValidationException extends RuntimeException {
        private final String field;
        private final Object value;
        private final String rule;
        private final Map<String, Object> context = new HashMap<>();
        
        public ValidationException(String field, Object value, String rule) {
            super(String.format("Validation failed for field '%s': %s", field, rule));
            this.field = field;
            this.value = value;
            this.rule = rule;
        }
        
        public ValidationException addContext(String key, Object value) {
            context.put(key, value);
            return this;
        }
        
        public String getField() { return field; }
        public Object getValue() { return value; }
        public String getRule() { return rule; }
        public Map<String, Object> getContext() { return new HashMap<>(context); }
    }
    
    static class BusinessRuleException extends Exception {
        private final String rule;
        private final Object entity;
        private final Map<String, Object> context = new HashMap<>();
        
        public BusinessRuleException(String rule, Object entity, String message) {
            super(message);
            this.rule = rule;
            this.entity = entity;
        }
        
        public BusinessRuleException addContext(String key, Object value) {
            context.put(key, value);
            return this;
        }
        
        public String getRule() { return rule; }
        public Object getEntity() { return entity; }
        public Map<String, Object> getContext() { return new HashMap<>(context); }
    }
    
    static class DataIntegrityException extends Exception {
        private final String operation;
        private final String entity;
        private final String constraint;
        private final Map<String, Object> context = new HashMap<>();
        
        public DataIntegrityException(String operation, String entity, String constraint) {
            super(String.format("Data integrity violation during %s on %s: %s", operation, entity, constraint));
            this.operation = operation;
            this.entity = entity;
            this.constraint = constraint;
        }
        
        public DataIntegrityException addContext(String key, Object value) {
            context.put(key, value);
            return this;
        }
        
        public String getOperation() { return operation; }
        public String getEntity() { return entity; }
        public String getConstraint() { return constraint; }
        public Map<String, Object> getContext() { return new HashMap<>(context); }
    }
    
    // Service demonstrating best practices
    static class ProductionService {
        
        public void processMultipleOperations(List<String> operations) throws AggregateException {
            List<Exception> errors = new ArrayList<>();
            
            for (String operation : operations) {
                try {
                    processOperation(operation);
                } catch (Exception e) {
                    errors.add(e);
                }
            }
            
            if (!errors.isEmpty()) {
                throw new AggregateException("Batch Operation Processing", errors);
            }
        }
        
        private void processOperation(String operation) throws ValidationException, BusinessRuleException {
            if ("invalid".equals(operation)) {
                throw ExceptionFactory.createValidationException("operation", operation, "must not be 'invalid'");
            }
            if ("forbidden".equals(operation)) {
                throw ExceptionFactory.createBusinessRuleException("OPERATION_ALLOWED", operation, "Operation is forbidden");
            }
        }
        
        public void demonstrateBuilderPattern() {
            try {
                throw new ComplexBusinessException.Builder("Payment processing failed", "PAY001")
                        .priority(ComplexBusinessException.Priority.CRITICAL)
                        .businessDomain("PAYMENT")
                        .affectedEntity("payment-12345")
                        .addMetadata("amount", 299.99)
                        .addMetadata("currency", "USD")
                        .addMetadata("gateway", "stripe")
                        .addSuggestedAction("Retry payment with different card")
                        .addSuggestedAction("Contact customer service")
                        .build();
            } catch (ComplexBusinessException e) {
                System.out.printf("Complex exception occurred: %s%n", e);
                System.out.printf("Priority: %d, Actions: %s%n", 
                        e.getPriority().getLevel(), e.getSuggestedActions());
            }
        }
        
        public void demonstrateLazyEvaluation() {
            try {
                throw new LazyEvaluationException(
                        () -> {
                            // Expensive message creation only if needed
                            return "Complex calculation failed: " + performExpensiveCalculation();
                        },
                        () -> {
                            // Expensive context gathering only if needed
                            Map<String, Object> context = new HashMap<>();
                            context.put("systemMemory", Runtime.getRuntime().totalMemory());
                            context.put("availableProcessors", Runtime.getRuntime().availableProcessors());
                            context.put("timestamp", LocalDateTime.now());
                            return context;
                        }
                );
            } catch (LazyEvaluationException e) {
                System.out.printf("Lazy exception: %s%n", e);
            }
        }
        
        private String performExpensiveCalculation() {
            // Simulate expensive operation
            return "result_" + System.currentTimeMillis();
        }
        
        public void demonstrateDistributedException() {
            try {
                throw new DistributedSystemException("user-service", "req-123", "node-east-1", 
                        "Service unavailable")
                        .addServiceContext("loadBalancer", "nginx-proxy")
                        .addServiceContext("cluster", "production-cluster")
                        .addServiceContext("version", "v1.2.3");
            } catch (DistributedSystemException e) {
                System.out.printf("Distributed system error: %s%n", e);
            }
        }
    }
    
    public static void demonstrateBestPractices() {
        System.out.println("=== Custom Exception Best Practices ===");
        
        ProductionService service = new ProductionService();
        
        // Aggregate exception handling
        try {
            service.processMultipleOperations(Arrays.asList("valid", "invalid", "forbidden", "another_valid"));
        } catch (AggregateException e) {
            System.out.printf("Aggregate exception: %s%n", e.getMessage());
            System.out.printf("Validation errors: %d%n", 
                    e.getExceptionsOfType(ValidationException.class).size());
            System.out.printf("Business rule errors: %d%n", 
                    e.getExceptionsOfType(BusinessRuleException.class).size());
        }
        
        // Builder pattern
        service.demonstrateBuilderPattern();
        
        // Lazy evaluation
        service.demonstrateLazyEvaluation();
        
        // Distributed system context
        service.demonstrateDistributedException();
    }
    
    public static void main(String[] args) {
        demonstrateBestPractices();
        
        System.out.println("\n=== Custom Exception Best Practices Summary ===");
        System.out.println("✅ Use exception factories for consistency");
        System.out.println("✅ Implement Serializable for distributed systems");
        System.out.println("✅ Apply builder pattern for complex exceptions");
        System.out.println("✅ Use lazy evaluation for expensive operations");
        System.out.println("✅ Aggregate multiple related errors");
        System.out.println("✅ Include comprehensive context information");
        System.out.println("✅ Provide suggested actions for resolution");
        System.out.println("✅ Design for monitoring and observability");
    }
}

/*
Custom Exception Best Practices:

1. Exception Factory Pattern:
   - Centralized exception creation
   - Consistent context and metadata
   - Version information and environment details
   - Reusable exception construction logic

2. Builder Pattern:
   - Complex exception construction
   - Fluent interface for readability
   - Optional parameters handling
   - Immutable exception objects

3. Lazy Evaluation:
   - Defer expensive operations until needed
   - Optimize performance for rarely accessed data
   - Use Supplier functional interfaces
   - Reduce exception creation overhead

4. Serialization Support:
   - Distributed system compatibility
   - Remote exception propagation
   - Version compatibility considerations
   - Custom serialization logic if needed

5. Exception Aggregation:
   - Collect multiple related errors
   - Batch operation error handling
   - Type-specific error filtering
   - Comprehensive error reporting

6. Production Considerations:
   - Monitoring and alerting integration
   - Performance impact minimization
   - Security information handling
   - Debugging and troubleshooting support
*/
```

## 📊 Custom Exception Design Comparison

| Approach | Pros | Cons | Use Cases |
|----------|------|------|-----------|
| **Basic Custom Exception** | Simple, Easy to implement | Limited context | Basic domain errors |
| **Rich Context Exception** | Detailed debugging info | Memory overhead | Production applications |
| **Builder Pattern** | Flexible construction | More complex code | Complex business logic |
| **Factory Pattern** | Consistent creation | Additional abstraction | Enterprise applications |
| **Aggregate Exception** | Multiple error handling | Complex processing | Batch operations |

## ⚠️ Common Custom Exception Mistakes

### 1. Overly generic exceptions
```java
// ❌ Bad - too generic
class ApplicationException extends Exception {
    // No specific context
}

// ✅ Good - specific domain exceptions
class PaymentProcessingException extends Exception {
    private final String paymentId;
    private final PaymentMethod method;
    // Specific context
}
```

### 2. Missing exception chaining
```java
// ❌ Bad - loses root cause
catch (SQLException e) {
    throw new DataAccessException("Database error");
}

// ✅ Good - preserves root cause
catch (SQLException e) {
    throw new DataAccessException("Database error", e);
}
```

### 3. Poor exception hierarchy
```java
// ❌ Bad - flat hierarchy
class UserNotFoundException extends Exception {}
class UserValidationException extends Exception {}
class UserPermissionException extends Exception {}

// ✅ Good - logical hierarchy
abstract class UserException extends Exception {}
class UserNotFoundException extends UserException {}
class UserValidationException extends UserException {}
```

## 🔥 Interview Questions & Answers

### Q1: When should you create custom checked vs unchecked exceptions?
**Answer**: 
- **Checked exceptions**: For recoverable business conditions that callers should handle (payment failures, validation errors)
- **Unchecked exceptions**: For programming errors or unrecoverable conditions (invalid arguments, illegal state)
- **Recovery**: If caller can reasonably recover, use checked; if it indicates bug, use unchecked
- **API design**: Checked exceptions become part of method contract, unchecked don't force handling

### Q2: How do you design exception hierarchies effectively?
**Answer**: 
- **Domain-based**: Group exceptions by business domain (User, Payment, Order)
- **Inheritance**: Create abstract base exceptions for common behavior
- **Granularity**: Balance between too specific (many classes) and too generic (no context)
- **Consistency**: Follow naming conventions and include similar information across hierarchy

### Q3: What information should custom exceptions contain?
**Answer**: 
- **Essential**: Clear message, error code, timestamp
- **Context**: Relevant data that caused the error (user ID, transaction amount)
- **Debugging**: Correlation IDs, operation context, system state
- **Recovery**: Suggested actions, retry information, alternative approaches
- **Monitoring**: Severity levels, categorization for alerting

### Q4: How do you handle exceptions in microservices architectures?
**Answer**: 
- **Serialization**: Make exceptions Serializable for cross-service calls
- **Context preservation**: Include service ID, request ID, node information
- **Error codes**: Use consistent error codes across services
- **Circuit breakers**: Implement retry logic and fallback mechanisms
- **Monitoring**: Centralized logging and distributed tracing

### Q5: What are the performance considerations for custom exceptions?
**Answer**: 
- **Stack trace cost**: Exception creation includes stack trace generation
- **Lazy evaluation**: Defer expensive operations until exception information is needed
- **Object creation**: Reuse exception instances for common errors if immutable
- **Context data**: Balance between useful information and memory usage
- **Frequency**: Monitor exception frequency to identify performance hotspots

---
[← Back to Main Guide](./README.md) | [← Previous: throw vs throws](./throw-vs-throws.md) | [Next: Abstract Classes →](./abstract-classes.md)
