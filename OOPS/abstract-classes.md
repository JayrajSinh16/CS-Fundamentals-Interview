# 🏛️ Abstract Classes in Java - Partial Implementation and Code Reuse

Abstract classes in Java provide a foundation for creating hierarchies with shared code and enforced contracts. They represent incomplete classes that cannot be instantiated directly but serve as blueprints for concrete subclasses, combining the benefits of inheritance with interface-like contracts.

## 🎯 Key Concepts

- **Abstract Classes**: Classes with `abstract` keyword that cannot be instantiated
- **Abstract Methods**: Methods declared without implementation (must be implemented by subclasses)
- **Concrete Methods**: Regular methods with implementation that subclasses inherit
- **Partial Implementation**: Mix of implemented and abstract methods
- **Template Method Pattern**: Define algorithm structure, let subclasses fill in details
- **Code Reuse**: Share common functionality across related classes

## 🌍 Real-World Analogy

**Vehicle Manufacturing Plant Analogy**:
- **Abstract Class** = Vehicle assembly blueprint (defines common structure but can't build a generic "vehicle")
- **Abstract Methods** = Required specifications that each vehicle type must define (engine type, fuel system)
- **Concrete Methods** = Common assembly processes (install wheels, paint, quality check)
- **Subclasses** = Specific vehicle types (Car, Truck, Motorcycle) that implement specific requirements
- **Template Method** = Standard manufacturing process with customizable steps

## 📋 Abstract Class Structure and Rules

```
Abstract Class Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                   Abstract Class Components                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  abstract class AbstractClassName {                            │
│                                                                 │
│    // Instance variables (regular fields)                      │
│    protected String commonField;                               │
│    private int privateField;                                   │
│                                                                 │
│    // Constructor (can exist, called via super())              │
│    public AbstractClassName(String commonField) {              │
│        this.commonField = commonField;                         │
│    }                                                            │
│                                                                 │
│    // Abstract methods (no implementation)                     │
│    public abstract void abstractMethod();                      │
│    protected abstract String getSpecificInfo();                │
│                                                                 │
│    // Concrete methods (with implementation)                   │
│    public void concreteMethod() {                              │
│        System.out.println("Shared functionality");            │
│    }                                                            │
│                                                                 │
│    // Template method (uses abstract methods)                  │
│    public final void templateMethod() {                        │
│        concreteMethod();                                        │
│        abstractMethod();    // Implemented by subclass         │
│        System.out.println(getSpecificInfo());                  │
│    }                                                            │
│  }                                                              │
└─────────────────────────────────────────────────────────────────┘

Abstract Class Rules:
┌─────────────────────────────────────────────────────────────────┐
│                    Abstract Class Constraints                  │
├─────────────────────────────────────────────────────────────────┤
│  ✅ CAN have:                    ❌ CANNOT have:                │
│  • Abstract methods               • Direct instantiation        │
│  • Concrete methods              • Final + abstract together    │
│  • Instance variables            • Private abstract methods     │
│  • Static methods/variables      • Static abstract methods      │
│  • Constructors                  • Abstract constructors       │
│  • Final methods                                                │
│  • Access modifiers                                             │
│                                                                 │
│  📝 Inheritance Rules:                                          │
│  • Subclass must implement ALL abstract methods                │
│  • OR subclass must also be abstract                          │
│  • Single inheritance (extends one abstract class)            │
│  • Can implement multiple interfaces                           │
└─────────────────────────────────────────────────────────────────┘

Abstract vs Interface vs Concrete Class:
┌─────────────────────────────────────────────────────────────────┐
│              Comparison of Class Types                         │
├─────────────────────────────────────────────────────────────────┤
│  Feature           │ Abstract Class │ Interface │ Concrete Class │
│  ─────────────────────────────────────────────────────────────  │
│  Instantiation     │ ❌ No          │ ❌ No     │ ✅ Yes         │
│  Abstract methods  │ ✅ Yes         │ ✅ Yes    │ ❌ No          │
│  Concrete methods  │ ✅ Yes         │ ✅ Yes*   │ ✅ Yes         │
│  Instance vars     │ ✅ Yes         │ ❌ No     │ ✅ Yes         │
│  Constructors      │ ✅ Yes         │ ❌ No     │ ✅ Yes         │
│  Multiple inheri.  │ ❌ No          │ ✅ Yes    │ ❌ No          │
│  Access modifiers  │ ✅ All         │ ✅ Public │ ✅ All         │
│  Static methods    │ ✅ Yes         │ ✅ Yes*   │ ✅ Yes         │
│                    │                │ *Java 8+  │                │
└─────────────────────────────────────────────────────────────────┘
```

## 💻 Practical Examples

### Example 1: Basic Abstract Class Implementation

```java
// BasicAbstractClasses.java - Fundamental abstract class concepts
import java.time.LocalDateTime;
import java.util.*;

public class BasicAbstractClasses {
    
    // Basic abstract class example
    abstract static class Animal {
        protected String name;
        protected int age;
        protected String species;
        
        // Constructor in abstract class
        public Animal(String name, int age, String species) {
            this.name = name;
            this.age = age;
            this.species = species;
            System.out.printf("Animal constructor called for %s%n", name);
        }
        
        // Abstract methods - must be implemented by subclasses
        public abstract void makeSound();
        public abstract void move();
        public abstract String getHabitat();
        
        // Concrete methods - inherited by all subclasses
        public void eat() {
            System.out.printf("%s is eating%n", name);
        }
        
        public void sleep() {
            System.out.printf("%s is sleeping%n", name);
        }
        
        // Template method - combines abstract and concrete methods
        public final void dailyRoutine() {
            System.out.printf("\n=== Daily routine for %s ===\n", name);
            makeSound();  // Abstract - implemented by subclass
            move();       // Abstract - implemented by subclass
            eat();        // Concrete - same for all animals
            sleep();      // Concrete - same for all animals
            System.out.printf("Lives in: %s%n", getHabitat()); // Abstract
        }
        
        // Getters
        public String getName() { return name; }
        public int getAge() { return age; }
        public String getSpecies() { return species; }
        
        @Override
        public String toString() {
            return String.format("%s{name='%s', age=%d, species='%s'}", 
                    getClass().getSimpleName(), name, age, species);
        }
    }
    
    // Concrete subclass implementing all abstract methods
    static class Dog extends Animal {
        private String breed;
        private boolean isTrained;
        
        public Dog(String name, int age, String breed, boolean isTrained) {
            super(name, age, "Canis lupus");
            this.breed = breed;
            this.isTrained = isTrained;
        }
        
        @Override
        public void makeSound() {
            System.out.printf("%s barks: Woof! Woof!%n", name);
        }
        
        @Override
        public void move() {
            System.out.printf("%s runs around energetically%n", name);
        }
        
        @Override
        public String getHabitat() {
            return "Domestic homes and yards";
        }
        
        // Dog-specific method
        public void fetch() {
            if (isTrained) {
                System.out.printf("%s fetches the ball!%n", name);
            } else {
                System.out.printf("%s doesn't know how to fetch yet%n", name);
            }
        }
        
        public String getBreed() { return breed; }
        public boolean isTrained() { return isTrained; }
    }
    
    static class Bird extends Animal {
        private double wingSpan;
        private boolean canFly;
        
        public Bird(String name, int age, String species, double wingSpan, boolean canFly) {
            super(name, age, species);
            this.wingSpan = wingSpan;
            this.canFly = canFly;
        }
        
        @Override
        public void makeSound() {
            System.out.printf("%s chirps melodiously%n", name);
        }
        
        @Override
        public void move() {
            if (canFly) {
                System.out.printf("%s soars through the sky%n", name);
            } else {
                System.out.printf("%s walks on the ground%n", name);
            }
        }
        
        @Override
        public String getHabitat() {
            return canFly ? "Trees and sky" : "Ground and low vegetation";
        }
        
        // Bird-specific method
        public void buildNest() {
            System.out.printf("%s is building a nest%n", name);
        }
        
        public double getWingSpan() { return wingSpan; }
        public boolean canFly() { return canFly; }
    }
    
    static class Fish extends Animal {
        private String waterType;
        private int maxDepth;
        
        public Fish(String name, int age, String species, String waterType, int maxDepth) {
            super(name, age, species);
            this.waterType = waterType;
            this.maxDepth = maxDepth;
        }
        
        @Override
        public void makeSound() {
            System.out.printf("%s makes bubbling sounds%n", name);
        }
        
        @Override
        public void move() {
            System.out.printf("%s swims gracefully through the water%n", name);
        }
        
        @Override
        public String getHabitat() {
            return String.format("%s water, up to %d meters depth", waterType, maxDepth);
        }
        
        // Fish-specific method
        public void swimInSchool() {
            System.out.printf("%s joins a school of fish%n", name);
        }
        
        public String getWaterType() { return waterType; }
        public int getMaxDepth() { return maxDepth; }
    }
    
    // Demonstration methods
    public static void demonstrateBasicAbstractClass() {
        System.out.println("=== Basic Abstract Class Example ===");
        
        // Cannot instantiate abstract class
        // Animal animal = new Animal("Generic", 5, "Unknown"); // Compilation error
        
        // Create concrete instances
        Dog dog = new Dog("Buddy", 3, "Golden Retriever", true);
        Bird bird = new Bird("Tweety", 2, "Canary", 0.2, true);
        Fish fish = new Fish("Nemo", 1, "Clownfish", "Saltwater", 30);
        
        // Store in Animal array (polymorphism)
        Animal[] animals = {dog, bird, fish};
        
        // Test polymorphism with abstract methods
        System.out.println("\n--- Testing polymorphic behavior ---");
        for (Animal animal : animals) {
            System.out.printf("Animal: %s%n", animal);
            animal.dailyRoutine(); // Template method calls abstract methods
        }
        
        // Test specific subclass methods
        System.out.println("\n--- Testing subclass-specific methods ---");
        dog.fetch();
        bird.buildNest();
        fish.swimInSchool();
    }
    
    public static void main(String[] args) {
        demonstrateBasicAbstractClass();
        
        System.out.println("\n=== Abstract Class Benefits ===");
        System.out.println("✅ Code reuse through concrete methods");
        System.out.println("✅ Enforced contracts through abstract methods");
        System.out.println("✅ Template method pattern implementation");
        System.out.println("✅ Polymorphic behavior with shared interface");
        System.out.println("✅ Constructor support for initialization");
    }
}

/*
Abstract Class Key Points:

1. Purpose:
   - Provide partial implementation for related classes
   - Enforce common interface through abstract methods
   - Enable code reuse through concrete methods
   - Support template method pattern

2. Abstract Methods:
   - Declared with abstract keyword
   - No implementation in abstract class
   - Must be implemented by concrete subclasses
   - Cannot be private or static

3. Concrete Methods:
   - Regular methods with implementation
   - Inherited by all subclasses
   - Can call abstract methods (template pattern)
   - Can be final to prevent overriding

4. Constructors:
   - Abstract classes can have constructors
   - Called through super() in subclass constructors
   - Used for initializing common fields

5. Instance Variables:
   - Abstract classes can have instance variables
   - Inherited by subclasses
   - Support various access modifiers
*/
```

### Example 2: Advanced Abstract Class Patterns

```java
// AdvancedAbstractPatterns.java - Complex abstract class usage patterns
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

public class AdvancedAbstractPatterns {
    
    // Template Method Pattern with hooks
    abstract static class DataProcessor {
        protected Map<String, Object> processingContext;
        
        public DataProcessor() {
            this.processingContext = new ConcurrentHashMap<>();
        }
        
        // Template method - defines the algorithm structure
        public final ProcessingResult processData(Object input) {
            long startTime = System.currentTimeMillis();
            
            try {
                // Step 1: Validation (hook method - optional override)
                if (shouldValidateInput()) {
                    validateInput(input);
                }
                
                // Step 2: Preprocessing (abstract - must implement)
                Object preprocessed = preprocessData(input);
                
                // Step 3: Core processing (abstract - must implement)
                Object processed = performCoreProcessing(preprocessed);
                
                // Step 4: Post-processing (hook method - optional override)
                if (shouldPostProcess()) {
                    processed = postProcessData(processed);
                }
                
                // Step 5: Result creation
                long duration = System.currentTimeMillis() - startTime;
                return new ProcessingResult(processed, true, null, duration, processingContext);
                
            } catch (Exception e) {
                long duration = System.currentTimeMillis() - startTime;
                return new ProcessingResult(null, false, e.getMessage(), duration, processingContext);
            }
        }
        
        // Abstract methods - must be implemented by subclasses
        protected abstract Object preprocessData(Object input) throws ProcessingException;
        protected abstract Object performCoreProcessing(Object data) throws ProcessingException;
        
        // Hook methods - can be overridden by subclasses (optional)
        protected boolean shouldValidateInput() {
            return true; // Default behavior
        }
        
        protected boolean shouldPostProcess() {
            return false; // Default behavior
        }
        
        protected void validateInput(Object input) throws ValidationException {
            if (input == null) {
                throw new ValidationException("Input cannot be null");
            }
        }
        
        protected Object postProcessData(Object data) {
            // Default implementation - can be overridden
            return data;
        }
        
        // Utility methods for subclasses
        protected void addContextInfo(String key, Object value) {
            processingContext.put(key, value);
        }
        
        protected Object getContextInfo(String key) {
            return processingContext.get(key);
        }
        
        // Abstract getter for processor type
        public abstract String getProcessorType();
    }
    
    // Text processing implementation
    static class TextDataProcessor extends DataProcessor {
        private final boolean caseSensitive;
        private final String encoding;
        
        public TextDataProcessor(boolean caseSensitive, String encoding) {
            super();
            this.caseSensitive = caseSensitive;
            this.encoding = encoding;
            addContextInfo("caseSensitive", caseSensitive);
            addContextInfo("encoding", encoding);
        }
        
        @Override
        protected Object preprocessData(Object input) throws ProcessingException {
            if (!(input instanceof String)) {
                throw new ProcessingException("TextDataProcessor expects String input");
            }
            
            String text = (String) input;
            addContextInfo("originalLength", text.length());
            
            // Preprocessing steps
            text = text.trim();
            if (!caseSensitive) {
                text = text.toLowerCase();
            }
            
            addContextInfo("preprocessedLength", text.length());
            return text;
        }
        
        @Override
        protected Object performCoreProcessing(Object data) throws ProcessingException {
            String text = (String) data;
            
            // Core text processing: word count, sentence analysis
            String[] words = text.split("\\s+");
            String[] sentences = text.split("[.!?]+");
            
            Map<String, Object> result = new HashMap<>();
            result.put("originalText", text);
            result.put("wordCount", words.length);
            result.put("sentenceCount", sentences.length);
            result.put("characterCount", text.length());
            result.put("words", Arrays.asList(words));
            
            addContextInfo("wordCount", words.length);
            addContextInfo("sentenceCount", sentences.length);
            
            return result;
        }
        
        @Override
        protected boolean shouldPostProcess() {
            return true; // Enable post-processing for text
        }
        
        @Override
        protected Object postProcessData(Object data) {
            @SuppressWarnings("unchecked")
            Map<String, Object> textResult = (Map<String, Object>) data;
            
            // Add readability analysis
            int wordCount = (Integer) textResult.get("wordCount");
            int sentenceCount = (Integer) textResult.get("sentenceCount");
            
            double avgWordsPerSentence = sentenceCount > 0 ? (double) wordCount / sentenceCount : 0;
            textResult.put("avgWordsPerSentence", avgWordsPerSentence);
            
            // Simple readability score
            String readabilityLevel;
            if (avgWordsPerSentence < 10) {
                readabilityLevel = "Easy";
            } else if (avgWordsPerSentence < 20) {
                readabilityLevel = "Medium";
            } else {
                readabilityLevel = "Difficult";
            }
            textResult.put("readabilityLevel", readabilityLevel);
            
            return textResult;
        }
        
        @Override
        public String getProcessorType() {
            return "TEXT_PROCESSOR";
        }
    }
    
    // Numeric data processing implementation
    static class NumericDataProcessor extends DataProcessor {
        private final boolean allowNegatives;
        private final double scaleFactor;
        
        public NumericDataProcessor(boolean allowNegatives, double scaleFactor) {
            super();
            this.allowNegatives = allowNegatives;
            this.scaleFactor = scaleFactor;
            addContextInfo("allowNegatives", allowNegatives);
            addContextInfo("scaleFactor", scaleFactor);
        }
        
        @Override
        protected void validateInput(Object input) throws ValidationException {
            super.validateInput(input);
            
            if (!(input instanceof Number[])) {
                throw new ValidationException("NumericDataProcessor expects Number[] input");
            }
            
            Number[] numbers = (Number[]) input;
            if (numbers.length == 0) {
                throw new ValidationException("Input array cannot be empty");
            }
        }
        
        @Override
        protected Object preprocessData(Object input) throws ProcessingException {
            Number[] numbers = (Number[]) input;
            addContextInfo("originalCount", numbers.length);
            
            List<Double> processed = new ArrayList<>();
            int negativeCount = 0;
            
            for (Number num : numbers) {
                double value = num.doubleValue();
                
                if (value < 0) {
                    negativeCount++;
                    if (!allowNegatives) {
                        continue; // Skip negative numbers
                    }
                }
                
                // Apply scaling
                processed.add(value * scaleFactor);
            }
            
            addContextInfo("negativeCount", negativeCount);
            addContextInfo("processedCount", processed.size());
            
            return processed.toArray(new Double[0]);
        }
        
        @Override
        protected Object performCoreProcessing(Object data) throws ProcessingException {
            Double[] numbers = (Double[]) data;
            
            if (numbers.length == 0) {
                throw new ProcessingException("No valid numbers to process");
            }
            
            // Statistical analysis
            double sum = Arrays.stream(numbers).mapToDouble(Double::doubleValue).sum();
            double avg = sum / numbers.length;
            double min = Arrays.stream(numbers).mapToDouble(Double::doubleValue).min().orElse(0);
            double max = Arrays.stream(numbers).mapToDouble(Double::doubleValue).max().orElse(0);
            
            // Calculate standard deviation
            double variance = Arrays.stream(numbers)
                    .mapToDouble(Double::doubleValue)
                    .map(x -> Math.pow(x - avg, 2))
                    .average()
                    .orElse(0);
            double stdDev = Math.sqrt(variance);
            
            Map<String, Object> result = new HashMap<>();
            result.put("numbers", Arrays.asList(numbers));
            result.put("count", numbers.length);
            result.put("sum", sum);
            result.put("average", avg);
            result.put("minimum", min);
            result.put("maximum", max);
            result.put("standardDeviation", stdDev);
            result.put("variance", variance);
            
            addContextInfo("statisticsCalculated", true);
            return result;
        }
        
        @Override
        public String getProcessorType() {
            return "NUMERIC_PROCESSOR";
        }
    }
    
    // Image data processing implementation
    static class ImageDataProcessor extends DataProcessor {
        private final String format;
        private final int targetWidth;
        private final int targetHeight;
        
        public ImageDataProcessor(String format, int targetWidth, int targetHeight) {
            super();
            this.format = format;
            this.targetWidth = targetWidth;
            this.targetHeight = targetHeight;
            addContextInfo("format", format);
            addContextInfo("targetWidth", targetWidth);
            addContextInfo("targetHeight", targetHeight);
        }
        
        @Override
        protected boolean shouldValidateInput() {
            return true; // Always validate image data
        }
        
        @Override
        protected void validateInput(Object input) throws ValidationException {
            super.validateInput(input);
            
            if (!(input instanceof ImageData)) {
                throw new ValidationException("ImageDataProcessor expects ImageData input");
            }
            
            ImageData image = (ImageData) input;
            if (image.getWidth() <= 0 || image.getHeight() <= 0) {
                throw new ValidationException("Invalid image dimensions");
            }
        }
        
        @Override
        protected Object preprocessData(Object input) throws ProcessingException {
            ImageData image = (ImageData) input;
            addContextInfo("originalWidth", image.getWidth());
            addContextInfo("originalHeight", image.getHeight());
            addContextInfo("originalFormat", image.getFormat());
            
            // Resize image if needed
            if (image.getWidth() != targetWidth || image.getHeight() != targetHeight) {
                image = resizeImage(image, targetWidth, targetHeight);
                addContextInfo("resized", true);
            }
            
            // Convert format if needed
            if (!image.getFormat().equals(format)) {
                image = convertFormat(image, format);
                addContextInfo("formatConverted", true);
            }
            
            return image;
        }
        
        @Override
        protected Object performCoreProcessing(Object data) throws ProcessingException {
            ImageData image = (ImageData) data;
            
            // Image analysis
            Map<String, Object> result = new HashMap<>();
            result.put("width", image.getWidth());
            result.put("height", image.getHeight());
            result.put("format", image.getFormat());
            result.put("aspectRatio", (double) image.getWidth() / image.getHeight());
            result.put("totalPixels", image.getWidth() * image.getHeight());
            
            // Simulate color analysis
            result.put("dominantColor", "RGB(128, 64, 192)");
            result.put("brightness", 0.65);
            result.put("contrast", 0.78);
            
            addContextInfo("analysisComplete", true);
            return result;
        }
        
        private ImageData resizeImage(ImageData image, int newWidth, int newHeight) {
            // Simulate image resizing
            return new ImageData(newWidth, newHeight, image.getFormat());
        }
        
        private ImageData convertFormat(ImageData image, String newFormat) {
            // Simulate format conversion
            return new ImageData(image.getWidth(), image.getHeight(), newFormat);
        }
        
        @Override
        public String getProcessorType() {
            return "IMAGE_PROCESSOR";
        }
    }
    
    // Supporting classes
    static class ProcessingResult {
        private final Object result;
        private final boolean success;
        private final String errorMessage;
        private final long processingTimeMs;
        private final Map<String, Object> context;
        
        public ProcessingResult(Object result, boolean success, String errorMessage, 
                              long processingTimeMs, Map<String, Object> context) {
            this.result = result;
            this.success = success;
            this.errorMessage = errorMessage;
            this.processingTimeMs = processingTimeMs;
            this.context = new HashMap<>(context);
        }
        
        // Getters
        public Object getResult() { return result; }
        public boolean isSuccess() { return success; }
        public String getErrorMessage() { return errorMessage; }
        public long getProcessingTimeMs() { return processingTimeMs; }
        public Map<String, Object> getContext() { return new HashMap<>(context); }
        
        @Override
        public String toString() {
            return String.format("ProcessingResult{success=%s, time=%dms, context=%s%s}", 
                    success, processingTimeMs, context,
                    success ? "" : ", error='" + errorMessage + "'");
        }
    }
    
    static class ImageData {
        private final int width;
        private final int height;
        private final String format;
        
        public ImageData(int width, int height, String format) {
            this.width = width;
            this.height = height;
            this.format = format;
        }
        
        public int getWidth() { return width; }
        public int getHeight() { return height; }
        public String getFormat() { return format; }
        
        @Override
        public String toString() {
            return String.format("ImageData{%dx%d, %s}", width, height, format);
        }
    }
    
    // Custom exceptions
    static class ProcessingException extends Exception {
        public ProcessingException(String message) {
            super(message);
        }
    }
    
    static class ValidationException extends Exception {
        public ValidationException(String message) {
            super(message);
        }
    }
    
    // Demonstration
    public static void demonstrateAdvancedPatterns() {
        System.out.println("=== Advanced Abstract Class Patterns ===");
        
        // Text processing
        DataProcessor textProcessor = new TextDataProcessor(false, "UTF-8");
        ProcessingResult textResult = textProcessor.processData(
                "Hello World! This is a sample text for processing. It contains multiple sentences.");
        System.out.printf("Text Processing: %s%n", textResult);
        
        // Numeric processing
        DataProcessor numericProcessor = new NumericDataProcessor(true, 1.5);
        Number[] numbers = {1, 2, -3, 4, 5, -6, 7, 8, 9, 10};
        ProcessingResult numericResult = numericProcessor.processData(numbers);
        System.out.printf("Numeric Processing: %s%n", numericResult);
        
        // Image processing
        DataProcessor imageProcessor = new ImageDataProcessor("PNG", 800, 600);
        ImageData image = new ImageData(1024, 768, "JPEG");
        ProcessingResult imageResult = imageProcessor.processData(image);
        System.out.printf("Image Processing: %s%n", imageResult);
        
        // Demonstrate polymorphism
        System.out.println("\n--- Polymorphic Processor Array ---");
        DataProcessor[] processors = {textProcessor, numericProcessor, imageProcessor};
        for (DataProcessor processor : processors) {
            System.out.printf("Processor Type: %s%n", processor.getProcessorType());
        }
    }
    
    public static void main(String[] args) {
        demonstrateAdvancedPatterns();
        
        System.out.println("\n=== Advanced Abstract Class Benefits ===");
        System.out.println("✅ Template method pattern implementation");
        System.out.println("✅ Hook methods for customizable behavior");
        System.out.println("✅ Shared utility methods and context");
        System.out.println("✅ Polymorphic processing pipeline");
        System.out.println("✅ Consistent error handling across implementations");
    }
}

/*
Advanced Abstract Class Patterns:

1. Template Method Pattern:
   - Define algorithm skeleton in abstract class
   - Let subclasses implement specific steps
   - Use final methods to prevent algorithm changes
   - Provide hook methods for optional customization

2. Hook Methods:
   - Optional override points in template
   - Default implementations provided
   - Enable behavior customization without breaking template
   - Boolean hooks for conditional processing

3. Context Management:
   - Shared data storage across processing steps
   - Utility methods for context access
   - Processing metadata and statistics
   - Error tracking and debugging information

4. Polymorphic Processing:
   - Common interface for different processors
   - Type-specific implementations
   - Shared result structure
   - Consistent error handling

5. Validation Hooks:
   - Input validation in template method
   - Type-specific validation in subclasses
   - Validation bypass options
   - Error propagation strategies
*/
```

### Example 3: Abstract Classes vs Interfaces Design Patterns

```java
// AbstractVsInterfacePatterns.java - When to use abstract classes vs interfaces
import java.util.*;

public class AbstractVsInterfacePatterns {
    
    // Abstract class for shared implementation and state
    abstract static class GameCharacter {
        protected String name;
        protected int health;
        protected int level;
        protected Position position;
        protected List<Item> inventory;
        
        public GameCharacter(String name, int health, int level) {
            this.name = name;
            this.health = health;
            this.level = level;
            this.position = new Position(0, 0);
            this.inventory = new ArrayList<>();
        }
        
        // Abstract methods - each character type implements differently
        public abstract void attack(GameCharacter target);
        public abstract void useSpecialAbility();
        public abstract String getCharacterClass();
        
        // Concrete methods - shared behavior
        public void move(int x, int y) {
            position.setX(x);
            position.setY(y);
            System.out.printf("%s moved to position (%d, %d)%n", name, x, y);
        }
        
        public void takeDamage(int damage) {
            health = Math.max(0, health - damage);
            System.out.printf("%s took %d damage, health: %d%n", name, damage, health);
            if (health == 0) {
                System.out.printf("%s has been defeated!%n", name);
            }
        }
        
        public void addItem(Item item) {
            inventory.add(item);
            System.out.printf("%s picked up %s%n", name, item.getName());
        }
        
        public boolean isAlive() {
            return health > 0;
        }
        
        // Template method for character actions
        public final void performTurn(GameCharacter target) {
            if (!isAlive()) {
                System.out.printf("%s cannot act - defeated%n", name);
                return;
            }
            
            System.out.printf("\n=== %s's turn ===\n", name);
            
            // Use special ability if available
            if (canUseSpecialAbility()) {
                useSpecialAbility();
            } else {
                // Regular attack
                attack(target);
            }
        }
        
        protected boolean canUseSpecialAbility() {
            return level >= 3; // Default implementation
        }
        
        // Getters
        public String getName() { return name; }
        public int getHealth() { return health; }
        public int getLevel() { return level; }
        public Position getPosition() { return position; }
        public List<Item> getInventory() { return new ArrayList<>(inventory); }
    }
    
    // Interface for optional behaviors (multiple inheritance simulation)
    interface Flyable {
        void fly(int altitude);
        int getMaxAltitude();
        boolean isFlying();
        
        // Default method (Java 8+)
        default void land() {
            System.out.printf("%s is landing%n", getClass().getSimpleName());
        }
    }
    
    interface Stealthable {
        void enterStealth();
        void exitStealth();
        boolean isStealthed();
        
        default int getStealthDuration() {
            return 10; // Default stealth duration in seconds
        }
    }
    
    interface Magical {
        void castSpell(String spellName, GameCharacter target);
        int getManaPoints();
        List<String> getKnownSpells();
        
        default boolean canCastSpell(String spellName) {
            return getKnownSpells().contains(spellName) && getManaPoints() > 0;
        }
    }
    
    // Concrete classes implementing abstract class and interfaces
    static class Warrior extends GameCharacter {
        private int armor;
        private String weapon;
        
        public Warrior(String name, int health, int level, int armor, String weapon) {
            super(name, health, level);
            this.armor = armor;
            this.weapon = weapon;
        }
        
        @Override
        public void attack(GameCharacter target) {
            int damage = 20 + level * 2; // Base damage
            System.out.printf("%s attacks %s with %s for %d damage%n", 
                    name, target.getName(), weapon, damage);
            target.takeDamage(damage);
        }
        
        @Override
        public void useSpecialAbility() {
            System.out.printf("%s uses Berserker Rage! Attack increased for next turn%n", name);
        }
        
        @Override
        public String getCharacterClass() {
            return "Warrior";
        }
        
        public int getArmor() { return armor; }
        public String getWeapon() { return weapon; }
    }
    
    static class Mage extends GameCharacter implements Magical {
        private int manaPoints;
        private List<String> knownSpells;
        
        public Mage(String name, int health, int level, int manaPoints) {
            super(name, health, level);
            this.manaPoints = manaPoints;
            this.knownSpells = new ArrayList<>(Arrays.asList("Fireball", "Heal", "Lightning"));
        }
        
        @Override
        public void attack(GameCharacter target) {
            if (manaPoints >= 5) {
                castSpell("Fireball", target);
            } else {
                // Melee attack as fallback
                int damage = 10 + level;
                System.out.printf("%s hits %s with staff for %d damage%n", 
                        name, target.getName(), damage);
                target.takeDamage(damage);
            }
        }
        
        @Override
        public void useSpecialAbility() {
            System.out.printf("%s channels magical energy, restoring mana%n", name);
            manaPoints = Math.min(100, manaPoints + 20);
        }
        
        @Override
        public String getCharacterClass() {
            return "Mage";
        }
        
        @Override
        public void castSpell(String spellName, GameCharacter target) {
            if (!canCastSpell(spellName)) {
                System.out.printf("%s cannot cast %s%n", name, spellName);
                return;
            }
            
            manaPoints -= 5;
            
            switch (spellName) {
                case "Fireball":
                    int damage = 25 + level * 3;
                    System.out.printf("%s casts Fireball at %s for %d damage%n", 
                            name, target.getName(), damage);
                    target.takeDamage(damage);
                    break;
                case "Heal":
                    health = Math.min(100, health + 20);
                    System.out.printf("%s casts Heal, restoring health to %d%n", name, health);
                    break;
                case "Lightning":
                    int lightningDamage = 30 + level * 2;
                    System.out.printf("%s casts Lightning at %s for %d damage%n", 
                            name, target.getName(), lightningDamage);
                    target.takeDamage(lightningDamage);
                    break;
            }
        }
        
        @Override
        public int getManaPoints() { return manaPoints; }
        
        @Override
        public List<String> getKnownSpells() { return new ArrayList<>(knownSpells); }
    }
    
    static class Rogue extends GameCharacter implements Stealthable {
        private boolean stealthed = false;
        private int stealthEndsAt = 0;
        
        public Rogue(String name, int health, int level) {
            super(name, health, level);
        }
        
        @Override
        public void attack(GameCharacter target) {
            int damage = 15 + level * 2;
            
            if (stealthed) {
                damage *= 2; // Sneak attack
                System.out.printf("%s performs sneak attack on %s for %d damage%n", 
                        name, target.getName(), damage);
                exitStealth(); // Attack breaks stealth
            } else {
                System.out.printf("%s attacks %s for %d damage%n", 
                        name, target.getName(), damage);
            }
            
            target.takeDamage(damage);
        }
        
        @Override
        public void useSpecialAbility() {
            if (!stealthed) {
                enterStealth();
            } else {
                System.out.printf("%s is already stealthed%n", name);
            }
        }
        
        @Override
        public String getCharacterClass() {
            return "Rogue";
        }
        
        @Override
        public void enterStealth() {
            stealthed = true;
            stealthEndsAt = (int) (System.currentTimeMillis() / 1000) + getStealthDuration();
            System.out.printf("%s enters stealth mode%n", name);
        }
        
        @Override
        public void exitStealth() {
            stealthed = false;
            System.out.printf("%s exits stealth mode%n", name);
        }
        
        @Override
        public boolean isStealthed() {
            if (stealthed && System.currentTimeMillis() / 1000 > stealthEndsAt) {
                exitStealth(); // Auto-exit when duration expires
            }
            return stealthed;
        }
    }
    
    static class DragonRider extends GameCharacter implements Flyable, Magical {
        private boolean flying = false;
        private int altitude = 0;
        private int manaPoints;
        private List<String> knownSpells;
        
        public DragonRider(String name, int health, int level, int manaPoints) {
            super(name, health, level);
            this.manaPoints = manaPoints;
            this.knownSpells = new ArrayList<>(Arrays.asList("Dragon Breath", "Wind Gust"));
        }
        
        @Override
        public void attack(GameCharacter target) {
            if (flying) {
                // Aerial attack
                castSpell("Dragon Breath", target);
            } else {
                // Ground attack
                int damage = 18 + level * 2;
                System.out.printf("%s attacks %s with dragon claws for %d damage%n", 
                        name, target.getName(), damage);
                target.takeDamage(damage);
            }
        }
        
        @Override
        public void useSpecialAbility() {
            if (!flying) {
                fly(50);
            } else {
                land();
            }
        }
        
        @Override
        public String getCharacterClass() {
            return "Dragon Rider";
        }
        
        @Override
        public void fly(int altitude) {
            this.altitude = Math.min(altitude, getMaxAltitude());
            flying = true;
            System.out.printf("%s takes flight at altitude %d%n", name, this.altitude);
        }
        
        @Override
        public int getMaxAltitude() {
            return 100;
        }
        
        @Override
        public boolean isFlying() {
            return flying;
        }
        
        @Override
        public void land() {
            flying = false;
            altitude = 0;
            System.out.printf("%s lands gracefully%n", name);
        }
        
        @Override
        public void castSpell(String spellName, GameCharacter target) {
            if (!canCastSpell(spellName)) {
                System.out.printf("%s cannot cast %s%n", name, spellName);
                return;
            }
            
            manaPoints -= 10;
            
            switch (spellName) {
                case "Dragon Breath":
                    int damage = 35 + level * 4;
                    System.out.printf("%s breathes dragon fire at %s for %d damage%n", 
                            name, target.getName(), damage);
                    target.takeDamage(damage);
                    break;
                case "Wind Gust":
                    System.out.printf("%s creates a powerful wind gust, disorienting %s%n", 
                            name, target.getName());
                    // Could implement status effects here
                    break;
            }
        }
        
        @Override
        public int getManaPoints() { return manaPoints; }
        
        @Override
        public List<String> getKnownSpells() { return new ArrayList<>(knownSpells); }
    }
    
    // Supporting classes
    static class Position {
        private int x, y;
        
        public Position(int x, int y) {
            this.x = x;
            this.y = y;
        }
        
        public int getX() { return x; }
        public int getY() { return y; }
        public void setX(int x) { this.x = x; }
        public void setY(int y) { this.y = y; }
        
        @Override
        public String toString() {
            return String.format("(%d, %d)", x, y);
        }
    }
    
    static class Item {
        private final String name;
        private final String type;
        
        public Item(String name, String type) {
            this.name = name;
            this.type = type;
        }
        
        public String getName() { return name; }
        public String getType() { return type; }
        
        @Override
        public String toString() {
            return String.format("%s (%s)", name, type);
        }
    }
    
    // Game simulator
    static class GameSimulator {
        public static void simulateBattle() {
            System.out.println("=== Game Battle Simulation ===");
            
            // Create characters
            Warrior warrior = new Warrior("Aragorn", 100, 5, 15, "Sword");
            Mage mage = new Mage("Gandalf", 80, 4, 50);
            Rogue rogue = new Rogue("Legolas", 90, 4);
            DragonRider rider = new DragonRider("Daenerys", 120, 6, 60);
            
            // Add some items
            warrior.addItem(new Item("Health Potion", "Consumable"));
            mage.addItem(new Item("Magic Staff", "Weapon"));
            
            // Demonstrate polymorphism
            GameCharacter[] characters = {warrior, mage, rogue, rider};
            
            System.out.println("\n--- Character Information ---");
            for (GameCharacter character : characters) {
                System.out.printf("%s (Level %d %s) - Health: %d, Position: %s%n", 
                        character.getName(), character.getLevel(), character.getCharacterClass(), 
                        character.getHealth(), character.getPosition());
            }
            
            // Battle simulation
            System.out.println("\n--- Battle Begins ---");
            warrior.performTurn(mage);
            mage.performTurn(warrior);
            rogue.performTurn(mage);
            rider.performTurn(warrior);
            
            // Demonstrate interface-specific behaviors
            System.out.println("\n--- Special Abilities ---");
            if (mage instanceof Magical) {
                Magical magicalChar = (Magical) mage;
                System.out.printf("%s knows spells: %s%n", mage.getName(), magicalChar.getKnownSpells());
            }
            
            if (rider instanceof Flyable) {
                Flyable flyingChar = (Flyable) rider;
                flyingChar.fly(75);
                System.out.printf("Flying status: %s%n", flyingChar.isFlying());
            }
            
            if (rogue instanceof Stealthable) {
                Stealthable stealthChar = (Stealthable) rogue;
                stealthChar.enterStealth();
                System.out.printf("Stealth status: %s%n", stealthChar.isStealthed());
            }
        }
    }
    
    public static void demonstrateAbstractVsInterface() {
        System.out.println("=== Abstract Classes vs Interfaces Design ===");
        
        GameSimulator.simulateBattle();
        
        System.out.println("\n=== Design Decision Summary ===");
        System.out.println("📋 Abstract Class (GameCharacter) used for:");
        System.out.println("  • Shared state (health, position, inventory)");
        System.out.println("  • Common behavior (move, takeDamage, addItem)");
        System.out.println("  • Template method (performTurn)");
        System.out.println("  • Constructor for initialization");
        
        System.out.println("\n🔌 Interfaces used for:");
        System.out.println("  • Optional capabilities (Flyable, Stealthable, Magical)");
        System.out.println("  • Multiple inheritance simulation");
        System.out.println("  • Contract definition without implementation");
        System.out.println("  • Default methods for common interface behavior");
    }
    
    public static void main(String[] args) {
        demonstrateAbstractVsInterface();
        
        System.out.println("\n=== When to Use Abstract Classes vs Interfaces ===");
        System.out.println("🏛️ Use Abstract Classes when:");
        System.out.println("  ✅ You have shared state and behavior");
        System.out.println("  ✅ You want to provide partial implementation");
        System.out.println("  ✅ You need constructors for initialization");
        System.out.println("  ✅ Related classes share common code");
        
        System.out.println("\n🔌 Use Interfaces when:");
        System.out.println("  ✅ You want to define contracts only");
        System.out.println("  ✅ You need multiple inheritance");
        System.out.println("  ✅ Unrelated classes should have same behavior");
        System.out.println("  ✅ You want loose coupling between components");
    }
}

/*
Abstract Classes vs Interfaces Design Guidelines:

1. Abstract Classes Are Better For:
   - Shared state (instance variables)
   - Partial implementation with common code
   - Constructor-based initialization
   - Template method pattern
   - Closely related classes (IS-A relationship)

2. Interfaces Are Better For:
   - Pure contracts (behavior definition)
   - Multiple inheritance needs
   - Capability-based design
   - Loose coupling
   - Unrelated classes with common behavior

3. Modern Java (8+) Considerations:
   - Interfaces can have default methods
   - Interfaces can have static methods
   - Consider composition over inheritance
   - Use both together for maximum flexibility

4. Design Patterns:
   - Template Method: Abstract classes
   - Strategy Pattern: Interfaces
   - Adapter Pattern: Interfaces
   - Factory Method: Abstract classes

5. Best Practices:
   - Prefer interfaces for new APIs
   - Use abstract classes for framework code
   - Combine both for comprehensive design
   - Consider future evolution of your API
*/
```

## 📊 Abstract Class vs Interface vs Concrete Class Comparison

| Feature | Abstract Class | Interface | Concrete Class |
|---------|----------------|-----------|----------------|
| **Instantiation** | ❌ Cannot instantiate | ❌ Cannot instantiate | ✅ Can instantiate |
| **Abstract Methods** | ✅ Can have | ✅ Can have | ❌ Cannot have |
| **Concrete Methods** | ✅ Can have | ✅ Can have (default) | ✅ Can have |
| **Instance Variables** | ✅ Can have | ❌ Cannot have | ✅ Can have |
| **Constructors** | ✅ Can have | ❌ Cannot have | ✅ Can have |
| **Multiple Inheritance** | ❌ No (extends one) | ✅ Yes (implements many) | ❌ No (extends one) |
| **Access Modifiers** | ✅ All types | ✅ Public/default | ✅ All types |

## ⚠️ Common Abstract Class Mistakes

### 1. Making abstract class too specific
```java
// ❌ Bad - too specific abstract class
abstract class DatabaseUserRepository {
    // Too tied to database implementation
}

// ✅ Good - general abstract class
abstract class UserRepository {
    // General repository pattern
}
```

### 2. Abstract methods with private access
```java
// ❌ Bad - private abstract method
abstract class Example {
    private abstract void method(); // Compilation error
}

// ✅ Good - appropriate access modifier
abstract class Example {
    protected abstract void method();
}
```

### 3. Not using template method pattern effectively
```java
// ❌ Bad - no structure in abstract class
abstract class Processor {
    public abstract void process();
}

// ✅ Good - template method with structure
abstract class Processor {
    public final void execute() {
        initialize();
        process();
        cleanup();
    }
    
    protected abstract void process();
    protected void initialize() { /* default */ }
    protected void cleanup() { /* default */ }
}
```

## 🔥 Interview Questions & Answers

### Q1: What's the difference between abstract classes and interfaces?
**Answer**: 
- **Abstract classes**: Can have both abstract and concrete methods, instance variables, constructors, and support single inheritance
- **Interfaces**: Define contracts with abstract methods (and default methods in Java 8+), no instance variables, support multiple inheritance
- **Use case**: Abstract classes for shared implementation and state, interfaces for contracts and capabilities
- **Inheritance**: Classes extend one abstract class but can implement multiple interfaces

### Q2: Can abstract classes have constructors? Why?
**Answer**: **Yes**, abstract classes can have constructors:
- **Purpose**: Initialize common fields and perform setup for subclasses
- **Access**: Called through `super()` in subclass constructors
- **Cannot instantiate**: Constructors don't allow direct instantiation of abstract class
- **Inheritance**: Subclass constructors must call abstract class constructor

### Q3: What is the Template Method pattern and how do abstract classes support it?
**Answer**: 
- **Definition**: Define algorithm skeleton in abstract class, let subclasses implement specific steps
- **Structure**: Abstract class has template method (final) that calls abstract methods
- **Benefits**: Code reuse, consistent algorithm structure, customizable steps
- **Example**: Data processing pipeline with validation, processing, and cleanup steps

### Q4: Can you have an abstract class without abstract methods?
**Answer**: **Yes**, you can:
- **Purpose**: Prevent direct instantiation while providing concrete implementations
- **Use case**: Base classes that should only be extended, not instantiated
- **Example**: Utility base classes, framework classes
- **Note**: Usually better to use regular classes with protected constructors

### Q5: How do you decide between using abstract classes vs composition?
**Answer**: 
- **Abstract classes**: When subclasses have strong IS-A relationship and share significant code
- **Composition**: When you want loose coupling and HAS-A relationships
- **Flexibility**: Composition is more flexible than inheritance
- **Modern approach**: Favor composition over inheritance, use interfaces for contracts

---
[← Back to Main Guide](./README.md) | [← Previous: Custom Exceptions](./custom-exceptions.md) | [Next: Interfaces →](./interfaces.md)
