# 🔄 Method Overriding in Java - Runtime Polymorphism

Method overriding is a fundamental concept in Java that enables runtime polymorphism, allowing subclasses to provide specific implementations of methods defined in their parent classes. This mechanism is crucial for achieving dynamic behavior and flexible design patterns in object-oriented programming.

## 🎯 Key Concepts

- **Runtime Polymorphism**: Method resolution happens at runtime based on actual object type
- **Dynamic Method Dispatch**: JVM determines which method to call based on object type
- **@Override Annotation**: Compiler annotation to ensure proper overriding
- **Method Signature**: Must match exactly (name, parameters, return type rules)
- **Access Modifier Rules**: Cannot reduce visibility in overridden methods

## 🌍 Real-World Analogy

**Restaurant Service Analogy**:
- **Base Service** = General restaurant service (greeting, serving, billing)
- **Fine Dining Override** = Same services but with premium implementation
- **Fast Food Override** = Same services but with quick, efficient implementation
- **Café Override** = Same services but with casual, friendly implementation
- **Method Signature** = Service contract (must provide same services)
- **Different Implementation** = How each restaurant type delivers the service

## 🏗️ Method Overriding Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   Method Overriding Flow                   │
├─────────────────────────────────────────────────────────────┤
│  Compile Time                    Runtime                    │
│  ┌─────────────────┐            ┌─────────────────┐        │
│  │  Reference Type │            │  Actual Object  │        │
│  │  Parent parent  │            │  Child object   │        │
│  │  parent.method()│   ──────>  │  child.method() │        │
│  │  (Static Check) │            │  (Dynamic Call) │        │
│  └─────────────────┘            └─────────────────┘        │
└─────────────────────────────────────────────────────────────┘

Method Resolution Order:
┌──────────────────┬──────────────────┬─────────────────┐
│   Call Type      │   Resolution     │   Binding Time  │
├──────────────────┼──────────────────┼─────────────────┤
│ obj.method()     │ Actual object    │ Runtime         │
│ super.method()   │ Parent class     │ Compile time    │
│ this.method()    │ Current class    │ Runtime         │
│ Class.static()   │ Class reference  │ Compile time    │
└──────────────────┴──────────────────┴─────────────────┘

Override Rules Hierarchy:
┌─────────────────────────────────────────────────────┐
│                    Parent Method                    │
│  public/protected ReturnType methodName(params)    │
└─────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────┐
│                   Child Override                    │
│  ✅ Same or wider access (public >= protected)      │
│  ✅ Same or covariant return type                  │
│  ✅ Same method name and parameters                │
│  ✅ Same or fewer checked exceptions               │
│  ❌ Cannot reduce access modifier                  │
│  ❌ Cannot change static to instance or vice versa │
└─────────────────────────────────────────────────────┘
```

## 💻 Practical Examples

### Example 1: Basic Method Overriding Rules

```java
// MethodOverridingBasics.java - Understanding method overriding fundamentals
public class MethodOverridingBasics {
    
    // Base class with methods to be overridden
    static class Vehicle {
        protected String brand;
        protected int year;
        protected double price;
        
        public Vehicle(String brand, int year, double price) {
            this.brand = brand;
            this.year = year;
            this.price = price;
        }
        
        // Method to be overridden - basic implementation
        public void start() {
            System.out.printf("Starting %s %d vehicle%n", brand, year);
        }
        
        // Method to be overridden - with return value
        public String getVehicleType() {
            return "Generic Vehicle";
        }
        
        // Method to be overridden - with parameters
        public double calculateMaintenanceCost(int miles) {
            return miles * 0.05; // Basic rate
        }
        
        // Protected method - can be overridden with same or wider access
        protected void performMaintenance() {
            System.out.printf("Performing basic maintenance on %s%n", brand);
        }
        
        // Method with checked exception
        public void startEngine() throws Exception {
            System.out.println("Starting generic engine");
            if (year < 1990) {
                throw new Exception("Engine too old to start reliably");
            }
        }
        
        // Method demonstrating covariant return types
        public Vehicle getRecommendedUpgrade() {
            return new Vehicle("Generic Upgrade", year + 5, price * 1.2);
        }
        
        // Final method - cannot be overridden
        public final void displayWarranty() {
            System.out.printf("Standard warranty for %s %d%n", brand, year);
        }
        
        // Static method - can be hidden but not overridden
        public static void displayCompanyInfo() {
            System.out.println("Vehicle Company - Generic Vehicles Division");
        }
        
        public void displayInfo() {
            System.out.printf("%s: %s %d, $%.2f%n", 
                    getVehicleType(), brand, year, price);
        }
        
        @Override
        public String toString() {
            return String.format("Vehicle{brand='%s', year=%d, price=%.2f}", 
                    brand, year, price);
        }
    }
    
    // Subclass demonstrating proper method overriding
    static class Car extends Vehicle {
        private int doors;
        private String fuelType;
        
        public Car(String brand, int year, double price, int doors, String fuelType) {
            super(brand, year, price);
            this.doors = doors;
            this.fuelType = fuelType;
        }
        
        // ✅ Proper override - same signature
        @Override
        public void start() {
            System.out.printf("Starting %s %d car with %s engine%n", 
                    brand, year, fuelType);
        }
        
        // ✅ Proper override - covariant return type
        @Override
        public String getVehicleType() {
            return "Car";
        }
        
        // ✅ Proper override - same parameters, enhanced logic
        @Override
        public double calculateMaintenanceCost(int miles) {
            double baseCost = super.calculateMaintenanceCost(miles); // Call parent implementation
            return baseCost + (doors * 5.0); // Add door-specific cost
        }
        
        // ✅ Proper override - widening access modifier (protected -> public)
        @Override
        public void performMaintenance() {
            super.performMaintenance(); // Call parent implementation
            System.out.printf("Checking %d doors and %s fuel system%n", doors, fuelType);
        }
        
        // ✅ Proper override - same or fewer checked exceptions
        @Override
        public void startEngine() throws Exception {
            System.out.printf("Starting %s car engine%n", fuelType);
            if (fuelType.equals("Electric") && year < 2010) {
                throw new Exception("Electric system needs update");
            }
            super.startEngine(); // Call parent implementation
        }
        
        // ✅ Covariant return type - can return subtype
        @Override
        public Car getRecommendedUpgrade() {
            return new Car(brand + " Upgraded", year + 3, price * 1.3, doors, "Hybrid");
        }
        
        // ❌ Cannot override final method
        // @Override
        // public void displayWarranty() { } // Compilation error
        
        // Static method hiding (not overriding)
        public static void displayCompanyInfo() {
            System.out.println("Vehicle Company - Car Division");
        }
        
        // Car-specific method
        public void openDoors() {
            System.out.printf("Opening %d doors of the %s%n", doors, brand);
        }
        
        public int getDoors() { return doors; }
        public String getFuelType() { return fuelType; }
        
        @Override
        public String toString() {
            return String.format("Car{brand='%s', year=%d, price=%.2f, doors=%d, fuel='%s'}", 
                    brand, year, price, doors, fuelType);
        }
    }
    
    // Another subclass showing different overriding patterns
    static class ElectricCar extends Car {
        private int batteryCapacity;
        private int range;
        
        public ElectricCar(String brand, int year, double price, int doors, 
                          int batteryCapacity, int range) {
            super(brand, year, price, doors, "Electric");
            this.batteryCapacity = batteryCapacity;
            this.range = range;
        }
        
        // ✅ Override Car's override
        @Override
        public void start() {
            System.out.printf("Silently starting %s %d electric car (%d kWh battery)%n", 
                    brand, year, batteryCapacity);
        }
        
        // ✅ Further specialization of return type
        @Override
        public String getVehicleType() {
            return "Electric Car";
        }
        
        // ✅ Override with completely different logic
        @Override
        public double calculateMaintenanceCost(int miles) {
            // Electric cars have lower maintenance costs
            return miles * 0.02 + (batteryCapacity * 0.01);
        }
        
        // ✅ Override without calling super (complete replacement)
        @Override
        public void performMaintenance() {
            System.out.printf("Electric car maintenance: Checking battery (%d kWh) and software%n", 
                    batteryCapacity);
        }
        
        // ✅ Override with no exceptions (removing parent exception)
        @Override
        public void startEngine() {
            // Electric cars are more reliable - no exceptions thrown
            System.out.printf("Starting electric motor (%d mile range)%n", range);
        }
        
        // ✅ Covariant return type with further specialization
        @Override
        public ElectricCar getRecommendedUpgrade() {
            return new ElectricCar(brand + " Plus", year + 2, price * 1.4, 
                                 getDoors(), batteryCapacity + 20, range + 50);
        }
        
        // Electric car specific methods
        public void chargeBattery() {
            System.out.printf("Charging %d kWh battery of %s%n", batteryCapacity, brand);
        }
        
        public int getBatteryCapacity() { return batteryCapacity; }
        public int getRange() { return range; }
        
        @Override
        public String toString() {
            return String.format("ElectricCar{brand='%s', year=%d, price=%.2f, " +
                               "doors=%d, battery=%dkWh, range=%d}", 
                    brand, year, price, getDoors(), batteryCapacity, range);
        }
    }
    
    // Class demonstrating invalid overriding attempts (commented out)
    static class InvalidOverrideExamples extends Vehicle {
        public InvalidOverrideExamples() {
            super("Invalid", 2020, 30000);
        }
        
        // ❌ Cannot reduce access modifier (public -> private)
        // @Override
        // private void start() { } // Compilation error
        
        // ❌ Cannot change return type (unless covariant)
        // @Override
        // public int getVehicleType() { return 1; } // Compilation error
        
        // ❌ Cannot change parameter list
        // @Override
        // public void start(boolean force) { } // Not an override, it's overloading
        
        // ❌ Cannot add more restrictive checked exceptions
        // @Override
        // public void startEngine() throws IOException { } // Compilation error
        
        // ❌ Cannot override static method (this is method hiding)
        // @Override
        // public static void displayCompanyInfo() { } // Warning, not override
        
        // ✅ Valid overrides
        @Override
        public void start() {
            System.out.println("Invalid vehicle starting mechanism");
        }
        
        @Override
        public String getVehicleType() {
            return "Invalid Vehicle";
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Method Overriding Basics Demonstration ===\n");
        
        demonstrateBasicOverriding();
        demonstratePolymorphism();
        demonstrateCovariantReturnTypes();
        demonstrateExceptionHandling();
        demonstrateStaticMethodHiding();
    }
    
    private static void demonstrateBasicOverriding() {
        System.out.println("--- Basic Method Overriding ---");
        
        Vehicle vehicle = new Vehicle("Generic", 2020, 25000);
        Car car = new Car("Toyota", 2022, 30000, 4, "Hybrid");
        ElectricCar electricCar = new ElectricCar("Tesla", 2023, 50000, 4, 75, 300);
        
        System.out.println("Vehicle start:");
        vehicle.start();
        
        System.out.println("Car start (overridden):");
        car.start();
        
        System.out.println("Electric car start (overridden again):");
        electricCar.start();
        
        System.out.println("\nVehicle types:");
        System.out.println("Vehicle type: " + vehicle.getVehicleType());
        System.out.println("Car type: " + car.getVehicleType());
        System.out.println("Electric car type: " + electricCar.getVehicleType());
        
        System.out.println();
    }
    
    private static void demonstratePolymorphism() {
        System.out.println("--- Runtime Polymorphism ---");
        
        // Array of Vehicle references pointing to different object types
        Vehicle[] vehicles = {
            new Vehicle("Generic", 2020, 25000),
            new Car("Honda", 2021, 28000, 4, "Gasoline"),
            new ElectricCar("Nissan", 2022, 35000, 4, 60, 250)
        };
        
        System.out.println("Polymorphic method calls:");
        for (Vehicle vehicle : vehicles) {
            System.out.printf("Processing %s:%n", vehicle.getClass().getSimpleName());
            vehicle.displayInfo(); // Calls overridden displayInfo()
            vehicle.start();       // Calls overridden start()
            
            // Calculate maintenance cost
            double cost = vehicle.calculateMaintenanceCost(10000);
            System.out.printf("Maintenance cost for 10,000 miles: $%.2f%n", cost);
            
            System.out.println();
        }
    }
    
    private static void demonstrateCovariantReturnTypes() {
        System.out.println("--- Covariant Return Types ---");
        
        Vehicle vehicle = new Vehicle("Base", 2020, 20000);
        Car car = new Car("Upgraded", 2021, 25000, 4, "Hybrid");
        ElectricCar electricCar = new ElectricCar("Premium", 2022, 40000, 4, 80, 350);
        
        // Covariant return types allow more specific return types
        Vehicle vehicleUpgrade = vehicle.getRecommendedUpgrade();
        Car carUpgrade = car.getRecommendedUpgrade(); // Returns Car, not Vehicle
        ElectricCar electricUpgrade = electricCar.getRecommendedUpgrade(); // Returns ElectricCar
        
        System.out.println("Recommended upgrades:");
        System.out.printf("Vehicle upgrade: %s%n", vehicleUpgrade);
        System.out.printf("Car upgrade: %s%n", carUpgrade);
        System.out.printf("Electric car upgrade: %s%n", electricUpgrade);
        
        // Demonstrate that return type is more specific
        System.out.printf("Car upgrade doors: %d%n", carUpgrade.getDoors());
        System.out.printf("Electric upgrade battery: %d kWh%n", electricUpgrade.getBatteryCapacity());
        
        System.out.println();
    }
    
    private static void demonstrateExceptionHandling() {
        System.out.println("--- Exception Handling in Overrides ---");
        
        Vehicle[] vehicles = {
            new Vehicle("Old", 1985, 5000),
            new Car("Modern", 2020, 30000, 4, "Gasoline"),
            new ElectricCar("Future", 2023, 45000, 4, 90, 400)
        };
        
        for (Vehicle vehicle : vehicles) {
            try {
                System.out.printf("Starting %s:%n", vehicle.getClass().getSimpleName());
                vehicle.startEngine(); // May throw exception
                System.out.println("Engine started successfully");
            } catch (Exception e) {
                System.out.printf("Failed to start: %s%n", e.getMessage());
            }
            System.out.println();
        }
    }
    
    private static void demonstrateStaticMethodHiding() {
        System.out.println("--- Static Method Hiding vs Overriding ---");
        
        Vehicle vehicle = new Vehicle("Test", 2020, 25000);
        Car car = new Car("Test", 2020, 25000, 4, "Test");
        
        // Static method calls - resolved at compile time
        System.out.println("Direct static method calls:");
        Vehicle.displayCompanyInfo();
        Car.displayCompanyInfo();
        
        // Static method calls through references - still compile-time binding
        System.out.println("\nStatic methods through references:");
        Vehicle vehicleRef = car; // Car object, Vehicle reference
        vehicleRef.displayCompanyInfo(); // Still calls Vehicle.displayCompanyInfo()
        car.displayCompanyInfo(); // Calls Car.displayCompanyInfo()
        
        System.out.println("\nNote: Static methods are not overridden, they are hidden");
    }
}

/*
Method Overriding Rules Summary:
1. Must have same method signature (name + parameters)
2. Cannot reduce access modifier visibility
3. Can have covariant return types
4. Cannot throw broader checked exceptions
5. Cannot override static, final, or private methods
6. Use @Override annotation for compile-time checking
7. Runtime binding determines which method executes
8. super.method() calls parent implementation
*/
```

### Example 2: Advanced Overriding Patterns and Best Practices

```java
// AdvancedOverridingPatterns.java - Complex overriding scenarios
public class AdvancedOverridingPatterns {
    
    // Abstract base class with template method pattern
    abstract static class PaymentProcessor {
        
        // Template method - final to prevent override
        public final PaymentResult processPayment(PaymentRequest request) {
            // Step 1: Validate (can be overridden)
            if (!validatePayment(request)) {
                return new PaymentResult(false, "Validation failed");
            }
            
            // Step 2: Pre-processing (can be overridden)
            preprocessPayment(request);
            
            // Step 3: Execute payment (must be implemented)
            PaymentResult result = executePayment(request);
            
            // Step 4: Post-processing (can be overridden)
            postprocessPayment(request, result);
            
            // Step 5: Log (final - cannot be overridden)
            logPayment(request, result);
            
            return result;
        }
        
        // Hook method with default implementation
        protected boolean validatePayment(PaymentRequest request) {
            return request != null && request.getAmount() > 0;
        }
        
        // Hook method with default implementation
        protected void preprocessPayment(PaymentRequest request) {
            System.out.printf("Pre-processing payment of $%.2f%n", request.getAmount());
        }
        
        // Abstract method - must be overridden
        protected abstract PaymentResult executePayment(PaymentRequest request);
        
        // Hook method with default implementation
        protected void postprocessPayment(PaymentRequest request, PaymentResult result) {
            System.out.printf("Post-processing: Payment %s%n", 
                    result.isSuccess() ? "successful" : "failed");
        }
        
        // Final method - cannot be overridden
        private final void logPayment(PaymentRequest request, PaymentResult result) {
            System.out.printf("LOG: Payment of $%.2f %s at %s%n", 
                    request.getAmount(), 
                    result.isSuccess() ? "succeeded" : "failed",
                    new java.util.Date());
        }
        
        // Default implementation that can be overridden
        public String getProcessorName() {
            return "Generic Payment Processor";
        }
        
        // Method with multiple overriding possibilities
        public double calculateFee(double amount) {
            return amount * 0.03; // 3% default fee
        }
    }
    
    // Payment request data class
    static class PaymentRequest {
        private final double amount;
        private final String currency;
        private final String cardNumber;
        private final String merchantId;
        
        public PaymentRequest(double amount, String currency, String cardNumber, String merchantId) {
            this.amount = amount;
            this.currency = currency;
            this.cardNumber = cardNumber;
            this.merchantId = merchantId;
        }
        
        public double getAmount() { return amount; }
        public String getCurrency() { return currency; }
        public String cardNumber() { return cardNumber; }
        public String getMerchantId() { return merchantId; }
    }
    
    // Payment result data class
    static class PaymentResult {
        private final boolean success;
        private final String message;
        private final String transactionId;
        
        public PaymentResult(boolean success, String message) {
            this(success, message, null);
        }
        
        public PaymentResult(boolean success, String message, String transactionId) {
            this.success = success;
            this.message = message;
            this.transactionId = transactionId;
        }
        
        public boolean isSuccess() { return success; }
        public String getMessage() { return message; }
        public String getTransactionId() { return transactionId; }
    }
    
    // Credit card processor implementation
    static class CreditCardProcessor extends PaymentProcessor {
        private final String processorId;
        
        public CreditCardProcessor(String processorId) {
            this.processorId = processorId;
        }
        
        // Override validation with enhanced logic
        @Override
        protected boolean validatePayment(PaymentRequest request) {
            // Call parent validation first
            if (!super.validatePayment(request)) {
                return false;
            }
            
            // Additional credit card validation
            return request.cardNumber() != null && 
                   request.cardNumber().length() >= 13 &&
                   request.getCurrency().equals("USD");
        }
        
        // Override preprocessing
        @Override
        protected void preprocessPayment(PaymentRequest request) {
            super.preprocessPayment(request); // Call parent implementation
            System.out.printf("Credit card validation: Checking card %s%n", 
                    maskCardNumber(request.cardNumber()));
        }
        
        // Implement abstract method
        @Override
        protected PaymentResult executePayment(PaymentRequest request) {
            System.out.printf("Processing credit card payment of $%.2f%n", request.getAmount());
            
            // Simulate payment processing
            if (request.getAmount() > 10000) {
                return new PaymentResult(false, "Amount exceeds credit limit");
            }
            
            String transactionId = "CC-" + System.currentTimeMillis();
            return new PaymentResult(true, "Credit card payment successful", transactionId);
        }
        
        // Override postprocessing
        @Override
        protected void postprocessPayment(PaymentRequest request, PaymentResult result) {
            super.postprocessPayment(request, result);
            if (result.isSuccess()) {
                System.out.printf("Credit card transaction ID: %s%n", result.getTransactionId());
            }
        }
        
        // Override processor name
        @Override
        public String getProcessorName() {
            return "Credit Card Processor (" + processorId + ")";
        }
        
        // Override fee calculation
        @Override
        public double calculateFee(double amount) {
            if (amount > 1000) {
                return amount * 0.025; // Lower fee for high amounts
            }
            return super.calculateFee(amount); // Use parent implementation
        }
        
        private String maskCardNumber(String cardNumber) {
            if (cardNumber.length() < 4) return cardNumber;
            return "*".repeat(cardNumber.length() - 4) + cardNumber.substring(cardNumber.length() - 4);
        }
    }
    
    // PayPal processor implementation
    static class PayPalProcessor extends PaymentProcessor {
        private final String apiKey;
        
        public PayPalProcessor(String apiKey) {
            this.apiKey = apiKey;
        }
        
        // Override validation with PayPal-specific logic
        @Override
        protected boolean validatePayment(PaymentRequest request) {
            boolean parentValid = super.validatePayment(request);
            // PayPal supports multiple currencies
            return parentValid && request.getMerchantId() != null;
        }
        
        // Override preprocessing completely (no super call)
        @Override
        protected void preprocessPayment(PaymentRequest request) {
            System.out.printf("PayPal preprocessing: Merchant %s, Amount %s %.2f%n", 
                    request.getMerchantId(), request.getCurrency(), request.getAmount());
        }
        
        // Implement abstract method
        @Override
        protected PaymentResult executePayment(PaymentRequest request) {
            System.out.printf("Processing PayPal payment via API%n");
            
            // Simulate PayPal API call
            if (!apiKey.startsWith("PP_")) {
                return new PaymentResult(false, "Invalid PayPal API key");
            }
            
            String transactionId = "PP-" + System.currentTimeMillis();
            return new PaymentResult(true, "PayPal payment successful", transactionId);
        }
        
        // Override processor name
        @Override
        public String getProcessorName() {
            return "PayPal Processor";
        }
        
        // Override fee calculation with different structure
        @Override
        public double calculateFee(double amount) {
            return 0.30 + (amount * 0.029); // PayPal's fee structure
        }
    }
    
    // Bitcoin processor with completely different approach
    static class BitcoinProcessor extends PaymentProcessor {
        private final String walletAddress;
        
        public BitcoinProcessor(String walletAddress) {
            this.walletAddress = walletAddress;
        }
        
        // Override validation with crypto-specific logic
        @Override
        protected boolean validatePayment(PaymentRequest request) {
            // Bitcoin has different validation rules
            return request.getAmount() > 0 && 
                   request.getCurrency().equals("BTC") &&
                   walletAddress != null;
        }
        
        // Skip preprocessing entirely
        @Override
        protected void preprocessPayment(PaymentRequest request) {
            // Bitcoin doesn't need traditional preprocessing
            System.out.println("Bitcoin: Preparing blockchain transaction");
        }
        
        // Implement abstract method with async-like behavior
        @Override
        protected PaymentResult executePayment(PaymentRequest request) {
            System.out.printf("Broadcasting Bitcoin transaction for %.8f BTC%n", request.getAmount());
            
            // Simulate blockchain confirmation time
            try {
                Thread.sleep(100); // Simulate network delay
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            
            String transactionId = "BTC-" + Integer.toHexString((int)(System.currentTimeMillis() % 100000));
            return new PaymentResult(true, "Bitcoin transaction broadcasted", transactionId);
        }
        
        // Override postprocessing for blockchain-specific actions
        @Override
        protected void postprocessPayment(PaymentRequest request, PaymentResult result) {
            if (result.isSuccess()) {
                System.out.printf("Bitcoin transaction %s added to mempool%n", result.getTransactionId());
                System.out.println("Waiting for blockchain confirmations...");
            } else {
                System.out.println("Bitcoin transaction failed - no network fees charged");
            }
        }
        
        @Override
        public String getProcessorName() {
            return "Bitcoin Processor";
        }
        
        // Override with crypto-specific fee calculation
        @Override
        public double calculateFee(double amount) {
            return 0.0001; // Fixed Bitcoin network fee regardless of amount
        }
    }
    
    // Demonstration of method overriding in interfaces
    interface Refundable {
        // Default method that can be overridden
        default RefundResult processRefund(String transactionId, double amount) {
            return new RefundResult(false, "Refunds not supported by default");
        }
        
        // Abstract method
        boolean supportsRefunds();
    }
    
    // Refund result class
    static class RefundResult {
        private final boolean success;
        private final String message;
        
        public RefundResult(boolean success, String message) {
            this.success = success;
            this.message = message;
        }
        
        public boolean isSuccess() { return success; }
        public String getMessage() { return message; }
    }
    
    // Enhanced credit card processor with refund capability
    static class RefundableCreditCardProcessor extends CreditCardProcessor implements Refundable {
        
        public RefundableCreditCardProcessor(String processorId) {
            super(processorId);
        }
        
        // Override interface default method
        @Override
        public RefundResult processRefund(String transactionId, double amount) {
            System.out.printf("Processing credit card refund of $%.2f for transaction %s%n", 
                    amount, transactionId);
            
            if (amount > 5000) {
                return new RefundResult(false, "Refund amount too large - manual review required");
            }
            
            return new RefundResult(true, "Credit card refund processed successfully");
        }
        
        // Implement abstract interface method
        @Override
        public boolean supportsRefunds() {
            return true;
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Advanced Method Overriding Patterns ===\n");
        
        demonstrateTemplateMethod();
        demonstratePolymorphicProcessing();
        demonstrateInterfaceOverriding();
        demonstrateFeeCalculations();
    }
    
    private static void demonstrateTemplateMethod() {
        System.out.println("--- Template Method Pattern ---");
        
        PaymentProcessor[] processors = {
            new CreditCardProcessor("VISA-001"),
            new PayPalProcessor("PP_abc123"),
            new BitcoinProcessor("1A2B3C4D5E6F7G8H9I")
        };
        
        PaymentRequest request = new PaymentRequest(150.00, "USD", "4111111111111111", "MERCHANT_123");
        
        for (PaymentProcessor processor : processors) {
            System.out.printf("\n=== Processing with %s ===%n", processor.getProcessorName());
            PaymentResult result = processor.processPayment(request);
            System.out.printf("Final result: %s - %s%n", 
                    result.isSuccess() ? "SUCCESS" : "FAILURE", result.getMessage());
            System.out.println("-".repeat(50));
        }
    }
    
    private static void demonstratePolymorphicProcessing() {
        System.out.println("\n--- Polymorphic Processing ---");
        
        // Array of different processor types
        PaymentProcessor[] processors = {
            new CreditCardProcessor("MC-002"),
            new PayPalProcessor("PP_xyz789"),
            new BitcoinProcessor("bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh")
        };
        
        // Different payment requests
        PaymentRequest[] requests = {
            new PaymentRequest(25.50, "USD", "5555555555554444", "SHOP_A"),
            new PaymentRequest(1500.00, "EUR", null, "SHOP_B"),
            new PaymentRequest(0.005, "BTC", null, "CRYPTO_SHOP")
        };
        
        // Process all combinations polymorphically
        for (int i = 0; i < processors.length; i++) {
            PaymentProcessor processor = processors[i];
            PaymentRequest request = requests[i];
            
            System.out.printf("Processor: %s%n", processor.getProcessorName());
            System.out.printf("Request: %.8f %s%n", request.getAmount(), request.getCurrency());
            
            PaymentResult result = processor.processPayment(request);
            System.out.printf("Result: %s%n", result.getMessage());
            System.out.println();
        }
    }
    
    private static void demonstrateInterfaceOverriding() {
        System.out.println("--- Interface Method Overriding ---");
        
        RefundableCreditCardProcessor refundableProcessor = 
                new RefundableCreditCardProcessor("REFUND-CC-001");
        
        // Test payment processing (inherited behavior)
        PaymentRequest paymentRequest = new PaymentRequest(200.0, "USD", "4000000000000002", "REFUND_MERCHANT");
        PaymentResult paymentResult = refundableProcessor.processPayment(paymentRequest);
        
        System.out.printf("Payment result: %s%n", paymentResult.getMessage());
        
        // Test refund capability (interface implementation)
        if (refundableProcessor.supportsRefunds() && paymentResult.isSuccess()) {
            System.out.println("\nProcessing refund...");
            RefundResult refundResult = refundableProcessor.processRefund(
                    paymentResult.getTransactionId(), 50.0);
            System.out.printf("Refund result: %s%n", refundResult.getMessage());
        }
    }
    
    private static void demonstrateFeeCalculations() {
        System.out.println("\n--- Fee Calculation Overrides ---");
        
        PaymentProcessor[] processors = {
            new CreditCardProcessor("FEE-CC"),
            new PayPalProcessor("PP_fees"),
            new BitcoinProcessor("btc_wallet")
        };
        
        double[] amounts = {50.0, 500.0, 5000.0};
        
        System.out.printf("%-25s", "Processor");
        for (double amount : amounts) {
            System.out.printf("%10s", "$" + amount);
        }
        System.out.println();
        System.out.println("-".repeat(55));
        
        for (PaymentProcessor processor : processors) {
            System.out.printf("%-25s", processor.getProcessorName());
            for (double amount : amounts) {
                double fee = processor.calculateFee(amount);
                System.out.printf("%10s", "$" + String.format("%.2f", fee));
            }
            System.out.println();
        }
    }
}

/*
Advanced Overriding Patterns:
1. Template Method: Final template with overridable hooks
2. Strategy Pattern: Different implementations via overriding
3. Interface Default Methods: Can be overridden in implementations
4. Covariant Return Types: Return more specific types
5. Exception Narrowing: Throw fewer exceptions in overrides
6. Hook Methods: Optional override points in template
7. Chain of Responsibility: Override to change processing chain
8. Polymorphic Collections: Runtime method resolution
*/
```

### Example 3: Method Overriding Edge Cases and Common Pitfalls

```java
// OverridingEdgeCases.java - Tricky scenarios and common mistakes
public class OverridingEdgeCases {
    
    // Base class for demonstrating edge cases
    static class EdgeCaseParent {
        public String value = "Parent Value";
        
        // Method with package-private access
        void packageMethod() {
            System.out.println("Parent package method");
        }
        
        // Protected method
        protected void protectedMethod() {
            System.out.println("Parent protected method");
        }
        
        // Public method
        public void publicMethod() {
            System.out.println("Parent public method");
        }
        
        // Method with primitive parameter
        public void processValue(int value) {
            System.out.printf("Parent processing int: %d%n", value);
        }
        
        // Method with wrapper parameter (overloading, not overriding)
        public void processValue(Integer value) {
            System.out.printf("Parent processing Integer: %s%n", value);
        }
        
        // Method with generic parameter
        public <T> void processGeneric(T item) {
            System.out.printf("Parent processing generic: %s%n", item);
        }
        
        // Method returning Object
        public Object getData() {
            return "Parent Data";
        }
        
        // Method with checked exception
        public void riskyOperation() throws Exception {
            System.out.println("Parent risky operation");
            if (Math.random() > 0.7) {
                throw new Exception("Parent exception");
            }
        }
        
        // Static method for hiding demonstration
        public static void staticMethod() {
            System.out.println("Parent static method");
        }
        
        // Method that calls other methods
        public void orchestrateOperations() {
            System.out.println("=== Parent Orchestration ===");
            this.publicMethod();        // Will call overridden version
            this.protectedMethod();     // Will call overridden version
            this.processValue(42);      // Will call overridden version
        }
    }
    
    // Child class demonstrating various override scenarios
    static class EdgeCaseChild extends EdgeCaseParent {
        public String value = "Child Value"; // Field hiding, not overriding
        
        // ✅ Valid: Same access level
        @Override
        void packageMethod() {
            System.out.println("Child package method");
        }
        
        // ✅ Valid: Widening access (protected -> public)
        @Override
        public void protectedMethod() {
            System.out.println("Child protected->public method");
            super.protectedMethod(); // Call parent implementation
        }
        
        // ✅ Valid: Same access level
        @Override
        public void publicMethod() {
            System.out.println("Child public method");
        }
        
        // ✅ Valid: Exact same signature
        @Override
        public void processValue(int value) {
            System.out.printf("Child processing int: %d (doubled: %d)%n", value, value * 2);
        }
        
        // This is NOT an override - it's an overload (different parameter type)
        // The parent class also has this method
        public void processValue(Integer value) {
            System.out.printf("Child processing Integer: %s (with null check)%n", 
                    value != null ? value : "NULL");
        }
        
        // ✅ Valid: Generic method override
        @Override
        public <T> void processGeneric(T item) {
            System.out.printf("Child processing generic with type info: %s (%s)%n", 
                    item, item.getClass().getSimpleName());
        }
        
        // ✅ Valid: Covariant return type (Object -> String)
        @Override
        public String getData() {
            return "Child Data (String)";
        }
        
        // ✅ Valid: Removing checked exception
        @Override
        public void riskyOperation() {
            System.out.println("Child risky operation (no exceptions)");
            // Child implementation doesn't throw exception
        }
        
        // Static method hiding (not overriding)
        // This hides the parent static method
        public static void staticMethod() {
            System.out.println("Child static method (hiding parent)");
        }
        
        // Override orchestration to show method resolution
        @Override
        public void orchestrateOperations() {
            System.out.println("=== Child Orchestration ===");
            super.orchestrateOperations(); // Call parent orchestration
            
            System.out.println("Additional child operations:");
            this.publicMethod();        // Calls child version
            super.publicMethod();       // Explicitly calls parent version
        }
        
        // Method to demonstrate field access
        public void demonstrateFieldAccess() {
            System.out.println("=== Field Access Demonstration ===");
            System.out.printf("this.value: %s%n", this.value);     // Child field
            System.out.printf("super.value: %s%n", super.value);   // Parent field
            
            // Local variable with same name
            String value = "Local Value";
            System.out.printf("local value: %s%n", value);         // Local variable
        }
    }
    
    // Class demonstrating invalid overriding attempts
    static class InvalidOverrideAttempts extends EdgeCaseParent {
        
        // ❌ Cannot reduce access modifier (public -> protected)
        // This would be a compilation error if uncommented
        /*
        @Override
        protected void publicMethod() {
            System.out.println("Trying to reduce access");
        }
        */
        
        // ❌ Cannot change return type to incompatible type
        // This would be a compilation error if uncommented
        /*
        @Override
        public int getData() {
            return 42;
        }
        */
        
        // ❌ Cannot add broader checked exceptions
        // This would be a compilation error if uncommented
        /*
        @Override
        public void riskyOperation() throws IOException {
            throw new IOException("Broader exception");
        }
        */
        
        // ❌ Cannot override static method (this is hiding, not overriding)
        // The @Override annotation would cause a compilation error
        /*
        @Override
        public static void staticMethod() {
            System.out.println("Attempting to override static method");
        }
        */
        
        // Valid overrides for demonstration
        @Override
        public void publicMethod() {
            System.out.println("InvalidOverrideAttempts public method");
        }
    }
    
    // Class demonstrating method resolution in complex inheritance
    static class GrandChild extends EdgeCaseChild {
        
        // Override child's override
        @Override
        public void publicMethod() {
            System.out.println("GrandChild public method");
        }
        
        // Override with call to grandparent
        @Override
        public void protectedMethod() {
            System.out.println("GrandChild protected method");
            // Cannot directly call grandparent method (no super.super syntax)
            super.protectedMethod(); // Calls EdgeCaseChild's version
        }
        
        // Override generic method with more specific behavior
        @Override
        public <T> void processGeneric(T item) {
            System.out.printf("GrandChild processing: %s%n", item);
            if (item instanceof String) {
                String str = (String) item;
                System.out.printf("String-specific processing: length=%d%n", str.length());
            }
            super.processGeneric(item); // Call parent implementation
        }
        
        // Method to demonstrate multi-level inheritance resolution
        public void demonstrateMethodResolution() {
            System.out.println("=== Multi-level Method Resolution ===");
            
            this.publicMethod();        // GrandChild version
            super.publicMethod();       // EdgeCaseChild version
            // super.super.publicMethod(); // ❌ Invalid syntax
            
            // To call grandparent method, need different approach
            this.callGrandparentMethod();
        }
        
        // Helper method to access grandparent behavior
        private void callGrandparentMethod() {
            // One way is to cast to parent and call super
            EdgeCaseParent parentView = this;
            // But this still calls the overridden version due to polymorphism
            
            // Another approach is to have parent provide access method
            System.out.println("Cannot directly call grandparent method in Java");
        }
    }
    
    // Interface for multiple inheritance demonstration
    interface Drawable {
        // Default method that can be overridden
        default void draw() {
            System.out.println("Drawing with default implementation");
        }
        
        // Abstract method
        void render();
    }
    
    interface Colorable {
        // Default method with same name as Drawable
        default void draw() {
            System.out.println("Drawing with color");
        }
        
        void applyColor(String color);
    }
    
    // Class implementing multiple interfaces with conflicting default methods
    static class Shape extends EdgeCaseParent implements Drawable, Colorable {
        
        // Must override conflicting default methods
        @Override
        public void draw() {
            System.out.println("Shape.draw() - resolving interface conflict");
            // Can call specific interface default method
            Drawable.super.draw();
            Colorable.super.draw();
        }
        
        @Override
        public void render() {
            System.out.println("Shape rendering implementation");
        }
        
        @Override
        public void applyColor(String color) {
            System.out.printf("Applying color: %s to shape%n", color);
        }
        
        // Also overrides parent class method
        @Override
        public void publicMethod() {
            System.out.println("Shape public method override");
            super.publicMethod(); // Call parent class method
        }
    }
    
    // Generic class with overriding
    static class GenericParent<T> {
        protected T data;
        
        public GenericParent(T data) {
            this.data = data;
        }
        
        public void processData() {
            System.out.printf("GenericParent processing: %s%n", data);
        }
        
        public T getData() {
            return data;
        }
    }
    
    static class GenericChild<T> extends GenericParent<T> {
        
        public GenericChild(T data) {
            super(data);
        }
        
        @Override
        public void processData() {
            System.out.printf("GenericChild processing: %s (type: %s)%n", 
                    data, data.getClass().getSimpleName());
        }
        
        // Covariant return type not applicable with generics in this context
        @Override
        public T getData() {
            System.out.printf("GenericChild getData called%n");
            return super.getData();
        }
    }
    
    // Specific generic child
    static class StringChild extends GenericParent<String> {
        
        public StringChild(String data) {
            super(data);
        }
        
        @Override
        public void processData() {
            System.out.printf("StringChild processing: '%s' (length: %d)%n", 
                    data, data.length());
        }
        
        // Covariant return type: T -> String
        @Override
        public String getData() {
            return "PREFIX_" + super.getData();
        }
    }
    
    public static void main(String[] args) {
        System.out.println("=== Method Overriding Edge Cases ===\n");
        
        demonstrateBasicOverriding();
        demonstrateFieldVsMethodOverriding();
        demonstrateStaticMethodHiding();
        demonstrateMultiLevelInheritance();
        demonstrateInterfaceConflicts();
        demonstrateGenericOverriding();
    }
    
    private static void demonstrateBasicOverriding() {
        System.out.println("--- Basic Override Scenarios ---");
        
        EdgeCaseParent parent = new EdgeCaseParent();
        EdgeCaseChild child = new EdgeCaseChild();
        EdgeCaseParent polymorphicChild = new EdgeCaseChild();
        
        System.out.println("Parent object:");
        parent.orchestrateOperations();
        
        System.out.println("\nChild object:");
        child.orchestrateOperations();
        
        System.out.println("\nPolymorphic reference (Parent ref -> Child object):");
        polymorphicChild.orchestrateOperations();
        
        System.out.println();
    }
    
    private static void demonstrateFieldVsMethodOverriding() {
        System.out.println("--- Field Hiding vs Method Overriding ---");
        
        EdgeCaseChild child = new EdgeCaseChild();
        EdgeCaseParent parentRef = child;
        
        System.out.println("Method calls (dynamic binding):");
        System.out.print("child.publicMethod(): ");
        child.publicMethod();
        System.out.print("parentRef.publicMethod(): ");
        parentRef.publicMethod(); // Calls child's overridden method
        
        System.out.println("\nField access (static binding):");
        System.out.printf("child.value: %s%n", child.value);
        System.out.printf("parentRef.value: %s%n", parentRef.value); // Accesses parent field!
        
        System.out.println("\nField access demonstration:");
        child.demonstrateFieldAccess();
        
        System.out.println();
    }
    
    private static void demonstrateStaticMethodHiding() {
        System.out.println("--- Static Method Hiding ---");
        
        EdgeCaseChild child = new EdgeCaseChild();
        EdgeCaseParent parentRef = child;
        
        System.out.println("Direct static method calls:");
        EdgeCaseParent.staticMethod();
        EdgeCaseChild.staticMethod();
        
        System.out.println("\nStatic method calls through references:");
        parentRef.staticMethod(); // Calls parent static method (compile-time binding)
        child.staticMethod();     // Calls child static method
        
        System.out.println("Note: Static methods are bound at compile time, not runtime");
        System.out.println();
    }
    
    private static void demonstrateMultiLevelInheritance() {
        System.out.println("--- Multi-level Inheritance ---");
        
        GrandChild grandChild = new GrandChild();
        
        System.out.println("GrandChild method calls:");
        grandChild.demonstrateMethodResolution();
        
        System.out.println("\nGeneric method with inheritance:");
        grandChild.processGeneric("Hello World");
        
        System.out.println();
    }
    
    private static void demonstrateInterfaceConflicts() {
        System.out.println("--- Interface Method Conflicts ---");
        
        Shape shape = new Shape();
        
        System.out.println("Resolving interface conflicts:");
        shape.draw(); // Must override due to conflict
        
        System.out.println("\nInterface implementations:");
        shape.render();
        shape.applyColor("Blue");
        
        System.out.println("\nParent class method override:");
        shape.publicMethod();
        
        System.out.println();
    }
    
    private static void demonstrateGenericOverriding() {
        System.out.println("--- Generic Method Overriding ---");
        
        GenericParent<Integer> genericParent = new GenericParent<>(42);
        GenericChild<Integer> genericChild = new GenericChild<>(100);
        StringChild stringChild = new StringChild("Hello");
        
        System.out.println("Generic parent:");
        genericParent.processData();
        System.out.printf("Data: %s%n", genericParent.getData());
        
        System.out.println("\nGeneric child:");
        genericChild.processData();
        System.out.printf("Data: %s%n", genericChild.getData());
        
        System.out.println("\nString-specific child:");
        stringChild.processData();
        System.out.printf("Data: %s%n", stringChild.getData());
        
        // Polymorphic calls
        System.out.println("\nPolymorphic calls:");
        GenericParent<String> polyRef = stringChild;
        polyRef.processData(); // Calls StringChild's overridden method
    }
}

/*
Method Overriding Edge Cases Summary:

1. Field Hiding vs Method Overriding:
   - Fields use static binding (compile-time)
   - Methods use dynamic binding (runtime)

2. Static Method Hiding:
   - Static methods cannot be overridden
   - Subclass static methods hide parent static methods
   - Binding is compile-time based on reference type

3. Access Modifier Rules:
   - Cannot reduce visibility (public -> protected ❌)
   - Can increase visibility (protected -> public ✅)

4. Exception Rules:
   - Cannot throw broader checked exceptions
   - Can throw fewer or more specific exceptions
   - Unchecked exceptions have no restrictions

5. Return Type Rules:
   - Must be same type or covariant (subtype)
   - Primitive types must match exactly
   - Generic types follow special rules

6. Interface Conflicts:
   - Must override conflicting default methods
   - Can call specific interface default with Interface.super.method()

7. Multi-level Inheritance:
   - No super.super syntax available
   - Each level can override previous overrides
   - Method resolution follows inheritance chain
*/
```

## 📊 Method Overriding Rules Summary

| Rule Category | Requirement | Example |
|---------------|-------------|---------|
| **Method Signature** | Must match exactly | `void method(int x)` |
| **Access Modifier** | Same or wider access | `protected` → `public` ✅ |
| **Return Type** | Same or covariant | `Object` → `String` ✅ |
| **Exceptions** | Same or fewer checked | `throws Exception` → `throws IOException` ✅ |
| **Static Methods** | Cannot override | Static methods are hidden, not overridden |
| **Final Methods** | Cannot override | Final methods prevent overriding |

## ⚠️ Common Method Overriding Mistakes

### 1. Reducing access modifier
```java
class Parent {
    public void method() { }
}

class Child extends Parent {
    // ❌ Cannot reduce from public to protected
    // protected void method() { } // Compilation error
}
```

### 2. Changing parameter list (creates overloading, not overriding)
```java
class Parent {
    public void process(int value) { }
}

class Child extends Parent {
    // ❌ This is overloading, not overriding
    public void process(long value) { }
    
    // ✅ Correct override
    @Override
    public void process(int value) { }
}
```

### 3. Forgetting @Override annotation
```java
class Child extends Parent {
    // ❌ Without @Override, typos create new methods instead of overrides
    public void proces(int value) { } // Typo - creates new method
    
    // ✅ With @Override, compiler catches errors
    @Override
    public void process(int value) { }
}
```

## 🔥 Interview Questions & Answers

### Q1: What is the difference between method overriding and method overloading?
**Answer**: 
- **Method Overriding**: Same method signature in parent and child classes, runtime polymorphism, inheritance relationship required
- **Method Overloading**: Same method name with different parameters in same class, compile-time polymorphism, no inheritance required
- **Binding**: Overriding uses dynamic binding, overloading uses static binding

### Q2: Can you override static methods in Java? Why or why not?
**Answer**: **No**, static methods cannot be overridden because:
- **Class-level**: Static methods belong to the class, not instances
- **Compile-time binding**: Static method calls resolved at compile time
- **Method hiding**: Subclass static methods hide parent static methods
- **No polymorphism**: Static methods don't participate in runtime polymorphism

### Q3: What are covariant return types in method overriding?
**Answer**: Covariant return types allow an overriding method to return a more specific type than the parent method:
- **Rule**: Return type must be a subtype of parent's return type
- **Example**: `Object` → `String`, `Animal` → `Dog`
- **Primitive types**: Must match exactly (no covariance)
- **Added in**: Java 5 to support better type safety

### Q4: What happens if a method throws exceptions in overriding?
**Answer**: Exception rules for overriding:
- **Checked exceptions**: Can throw same, fewer, or more specific exceptions
- **Cannot throw**: Broader checked exceptions than parent
- **Unchecked exceptions**: No restrictions (can throw any)
- **Removing exceptions**: Child can choose not to throw parent's exceptions

### Q5: How does method resolution work in inheritance hierarchy?
**Answer**: Method resolution follows these steps:
1. **Start**: From actual object type (runtime type)
2. **Search**: Look for method in current class
3. **Found**: Execute the method
4. **Not found**: Move up inheritance hierarchy
5. **Continue**: Until method found or reach Object class
6. **Result**: Most specific implementation executes (dynamic dispatch)

---
[← Back to Main Guide](./README.md) | [← Previous: this & super Keywords](./this-super-keywords.md) | [Next: Exception Hierarchy →](./exception-hierarchy.md)
