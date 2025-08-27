# 🧵 String Handling in Java - Immutability, Pool, and Performance

String handling is fundamental in Java programming and a frequent topic in technical interviews. Understanding String immutability, the String pool, and performance implications is crucial for writing efficient Java applications.

## 🎯 Key Concepts

- **String Immutability**: Strings cannot be modified after creation
- **String Pool**: JVM optimization for string storage and reuse
- **StringBuilder/StringBuffer**: Mutable alternatives for string manipulation
- **Performance Implications**: Understanding when to use different string classes
- **Encoding and Internals**: How strings are stored and compared

## 🌍 Real-World Analogy

**Library Book Analogy**:
- **String** = Published book (cannot be modified, shared copies)
- **String Pool** = Library catalog (shared references to same books)
- **StringBuilder** = Draft manuscript (can be edited before publishing)
- **String Concatenation** = Creating new edition when changing content
- **intern()** = Adding book to library catalog for sharing

## 🏗️ String Architecture in JVM

```
┌─────────────────────────────────────────────────────────────┐
│                        JVM Heap Memory                      │
├─────────────────────────────────────────────────────────────┤
│  String Pool (Part of Heap since Java 7)                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  "Hello"     "World"     "Java"                     │   │
│  │     ↑           ↑          ↑                        │   │
│  │  Shared    Shared     Shared                        │   │
│  │  String    String     String                        │   │
│  │  Objects   Objects    Objects                       │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│  Regular Heap Space                                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  String objects created with 'new' keyword          │   │
│  │  StringBuilder/StringBuffer objects                 │   │
│  │  char[] arrays backing the strings                  │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘

String Object Internal Structure (Java 8-):
┌─────────────────────────────────────┐
│         String Object               │
├─────────────────────────────────────┤
│  char[] value                       │  ← Character array (actual data)
│  int hash                           │  ← Cached hashcode
│  int offset (removed in Java 7)     │  
│  int count (removed in Java 7)      │  
└─────────────────────────────────────┘

String Object Internal Structure (Java 9+):
┌─────────────────────────────────────┐
│         String Object               │
├─────────────────────────────────────┤
│  byte[] value                       │  ← Compact strings (Latin-1 or UTF-16)
│  byte coder                         │  ← Encoding indicator
│  int hash                           │  ← Cached hashcode
└─────────────────────────────────────┘
```

## 💻 Practical Examples

### Example 1: String Immutability and Pool Demonstration

```java
// StringImmutabilityDemo.java - Understanding String behavior
public class StringImmutabilityDemo {
    
    public static void main(String[] args) {
        System.out.println("=== String Immutability and Pool Demonstration ===\n");
        
        StringImmutabilityDemo demo = new StringImmutabilityDemo();
        demo.demonstrateStringPool();
        demo.demonstrateImmutability();
        demo.demonstrateStringComparison();
        demo.demonstrateInternMethod();
    }
    
    public void demonstrateStringPool() {
        System.out.println("=== String Pool Demonstration ===");
        
        // String literals go to String Pool
        String str1 = "Hello World";
        String str2 = "Hello World";
        String str3 = "Hello" + " World";  // Compile-time concatenation
        
        // new keyword creates object in regular heap
        String str4 = new String("Hello World");
        String str5 = new String("Hello World");
        
        System.out.println("String Pool Behavior:");
        System.out.printf("str1 = \"%s\" (literal)\n", str1);
        System.out.printf("str2 = \"%s\" (literal)\n", str2);
        System.out.printf("str3 = \"%s\" (compile-time concat)\n", str3);
        System.out.printf("str4 = \"%s\" (new keyword)\n", str4);
        System.out.printf("str5 = \"%s\" (new keyword)\n", str5);
        
        System.out.println("\nReference Comparison (==):");
        System.out.printf("str1 == str2: %b (same pool reference)\n", str1 == str2);
        System.out.printf("str1 == str3: %b (same pool reference)\n", str1 == str3);
        System.out.printf("str1 == str4: %b (different objects)\n", str1 == str4);
        System.out.printf("str4 == str5: %b (different objects)\n", str4 == str5);
        
        System.out.println("\nContent Comparison (equals):");
        System.out.printf("str1.equals(str4): %b (same content)\n", str1.equals(str4));
        System.out.printf("str4.equals(str5): %b (same content)\n", str4.equals(str5));
        
        // Memory addresses (for illustration)
        System.out.println("\nObject Identity:");
        System.out.printf("str1 identity: %s\n", System.identityHashCode(str1));
        System.out.printf("str2 identity: %s\n", System.identityHashCode(str2));
        System.out.printf("str4 identity: %s\n", System.identityHashCode(str4));
        System.out.printf("str5 identity: %s\n", System.identityHashCode(str5));
        
        System.out.println();
    }
    
    public void demonstrateImmutability() {
        System.out.println("=== String Immutability Demonstration ===");
        
        String original = "Hello";
        String modified = original.concat(" World");
        String uppercase = original.toUpperCase();
        String substring = original.substring(1);
        
        System.out.printf("Original string: \"%s\"\n", original);
        System.out.printf("After concat(): \"%s\"\n", modified);
        System.out.printf("After toUpperCase(): \"%s\"\n", uppercase);
        System.out.printf("After substring(): \"%s\"\n", substring);
        System.out.printf("Original unchanged: \"%s\"\n", original);
        
        System.out.println("\nString operations create new objects:");
        System.out.printf("original == modified: %b\n", original == modified);
        System.out.printf("original == uppercase: %b\n", original == uppercase);
        System.out.printf("original == substring: %b\n", original == substring);
        
        // Attempting to modify (creates new objects)
        String mutable = "Start";
        System.out.printf("\nBefore operations: \"%s\"\n", mutable);
        System.out.printf("Identity before: %s\n", System.identityHashCode(mutable));
        
        mutable = mutable + " Middle";
        System.out.printf("After concatenation: \"%s\"\n", mutable);
        System.out.printf("Identity after: %s\n", System.identityHashCode(mutable));
        
        mutable = mutable + " End";
        System.out.printf("After second concat: \"%s\"\n", mutable);
        System.out.printf("Identity final: %s\n", System.identityHashCode(mutable));
        
        System.out.println("Each operation created a new String object!\n");
    }
    
    public void demonstrateStringComparison() {
        System.out.println("=== String Comparison Methods ===");
        
        String str1 = "Java";
        String str2 = "JAVA";
        String str3 = "java";
        String str4 = new String("Java");
        String str5 = null;
        
        System.out.println("Comparison Methods:");
        
        // equals() - case sensitive content comparison
        System.out.printf("str1.equals(str4): %b\n", str1.equals(str4));
        System.out.printf("str1.equals(str2): %b\n", str1.equals(str2));
        
        // equalsIgnoreCase() - case insensitive
        System.out.printf("str1.equalsIgnoreCase(str2): %b\n", str1.equalsIgnoreCase(str2));
        System.out.printf("str1.equalsIgnoreCase(str3): %b\n", str1.equalsIgnoreCase(str3));
        
        // compareTo() - lexicographic comparison
        System.out.printf("str1.compareTo(str2): %d\n", str1.compareTo(str2));
        System.out.printf("str2.compareTo(str3): %d\n", str2.compareTo(str3));
        
        // compareToIgnoreCase()
        System.out.printf("str1.compareToIgnoreCase(str2): %d\n", str1.compareToIgnoreCase(str2));
        
        // Safe comparison with null
        System.out.println("\nNull-safe Comparison:");
        System.out.printf("Objects.equals(str1, str5): %b\n", 
                java.util.Objects.equals(str1, str5));
        System.out.printf("Objects.equals(str5, str5): %b\n", 
                java.util.Objects.equals(str5, str5));
        
        // String contains, startsWith, endsWith
        String text = "Hello World Java Programming";
        System.out.println("\nPattern Matching:");
        System.out.printf("text.contains(\"World\"): %b\n", text.contains("World"));
        System.out.printf("text.startsWith(\"Hello\"): %b\n", text.startsWith("Hello"));
        System.out.printf("text.endsWith(\"Programming\"): %b\n", text.endsWith("Programming"));
        
        System.out.println();
    }
    
    public void demonstrateInternMethod() {
        System.out.println("=== String intern() Method ===");
        
        // Create strings outside pool
        String str1 = new String("Intern Example");
        String str2 = new String("Intern Example");
        
        System.out.println("Before intern():");
        System.out.printf("str1 == str2: %b\n", str1 == str2);
        System.out.printf("str1.equals(str2): %b\n", str1.equals(str2));
        
        // Add to string pool
        String str1Interned = str1.intern();
        String str2Interned = str2.intern();
        
        System.out.println("\nAfter intern():");
        System.out.printf("str1Interned == str2Interned: %b\n", str1Interned == str2Interned);
        System.out.printf("str1 == str1Interned: %b\n", str1 == str1Interned);
        
        // Literal comparison
        String literal = "Intern Example";
        System.out.printf("literal == str1Interned: %b\n", literal == str1Interned);
        
        // Performance implications
        System.out.println("\nPerformance Test:");
        long start = System.nanoTime();
        for (int i = 0; i < 100000; i++) {
            String temp = new String("Test" + i).intern();
        }
        long end = System.nanoTime();
        System.out.printf("intern() operations took: %.2f ms\n", (end - start) / 1_000_000.0);
        
        System.out.println("Note: intern() can be expensive for large numbers of unique strings\n");
    }
}

/*
String Pool Benefits:
1. Memory optimization - shared string objects
2. Faster equality comparison - reference comparison
3. Reduced garbage collection pressure

String Pool Considerations:
1. Limited size (can be configured)
2. intern() can be expensive
3. Memory leaks if too many interned strings
*/
```

### Example 2: StringBuilder vs StringBuffer Performance

```java
// StringPerformanceDemo.java - Comparing string manipulation performance
public class StringPerformanceDemo {
    
    private static final int ITERATIONS = 10000;
    
    public static void main(String[] args) {
        System.out.println("=== String Performance Comparison ===\n");
        
        StringPerformanceDemo demo = new StringPerformanceDemo();
        demo.compareStringConcatenation();
        demo.compareStringBuilderMethods();
        demo.demonstrateMemoryImpact();
        demo.showBestPractices();
    }
    
    public void compareStringConcatenation() {
        System.out.println("=== String Concatenation Performance ===");
        
        // Test 1: String concatenation (inefficient)
        long startTime = System.nanoTime();
        String result1 = stringConcatenation();
        long endTime = System.nanoTime();
        long stringTime = endTime - startTime;
        
        // Test 2: StringBuilder (efficient)
        startTime = System.nanoTime();
        String result2 = stringBuilderConcatenation();
        endTime = System.nanoTime();
        long builderTime = endTime - startTime;
        
        // Test 3: StringBuffer (thread-safe)
        startTime = System.nanoTime();
        String result3 = stringBufferConcatenation();
        endTime = System.nanoTime();
        long bufferTime = endTime - startTime;
        
        System.out.printf("Results for %d iterations:\n", ITERATIONS);
        System.out.printf("String concatenation: %.2f ms\n", stringTime / 1_000_000.0);
        System.out.printf("StringBuilder: %.2f ms\n", builderTime / 1_000_000.0);
        System.out.printf("StringBuffer: %.2f ms\n", bufferTime / 1_000_000.0);
        
        System.out.printf("\nPerformance improvement:\n");
        System.out.printf("StringBuilder is %.1fx faster than String\n", 
                (double) stringTime / builderTime);
        System.out.printf("StringBuilder is %.1fx faster than StringBuffer\n", 
                (double) bufferTime / builderTime);
        
        // Verify results are identical
        System.out.printf("\nResults identical: %b\n", 
                result1.equals(result2) && result2.equals(result3));
        System.out.printf("Final string length: %d characters\n", result1.length());
        
        System.out.println();
    }
    
    private String stringConcatenation() {
        String result = "";
        for (int i = 0; i < ITERATIONS; i++) {
            result += "Item " + i + " "; // Creates new String object each time
        }
        return result;
    }
    
    private String stringBuilderConcatenation() {
        StringBuilder builder = new StringBuilder();
        for (int i = 0; i < ITERATIONS; i++) {
            builder.append("Item ").append(i).append(" "); // Modifies internal buffer
        }
        return builder.toString();
    }
    
    private String stringBufferConcatenation() {
        StringBuffer buffer = new StringBuffer();
        for (int i = 0; i < ITERATIONS; i++) {
            buffer.append("Item ").append(i).append(" "); // Thread-safe version
        }
        return buffer.toString();
    }
    
    public void compareStringBuilderMethods() {
        System.out.println("=== StringBuilder Method Performance ===");
        
        StringBuilder sb = new StringBuilder();
        
        // Capacity management
        System.out.println("Capacity Management:");
        System.out.printf("Initial capacity: %d\n", sb.capacity());
        
        sb.append("Hello");
        System.out.printf("After 'Hello': length=%d, capacity=%d\n", sb.length(), sb.capacity());
        
        // Force capacity expansion
        String longString = "A".repeat(50);
        sb.append(longString);
        System.out.printf("After long append: length=%d, capacity=%d\n", sb.length(), sb.capacity());
        
        // Pre-sizing for better performance
        StringBuilder optimized = new StringBuilder(1000); // Pre-allocate capacity
        System.out.printf("Pre-sized capacity: %d\n", optimized.capacity());
        
        // Method chaining
        System.out.println("\nMethod Chaining Example:");
        String result = new StringBuilder()
                .append("Java")
                .append(" is")
                .append(" awesome")
                .append("!")
                .toString();
        System.out.printf("Chained result: %s\n", result);
        
        // Different append methods
        StringBuilder methods = new StringBuilder();
        methods.append("String: ").append("text")
                .append(", Int: ").append(42)
                .append(", Double: ").append(3.14)
                .append(", Boolean: ").append(true)
                .append(", Char: ").append('X');
        
        System.out.printf("Multiple types: %s\n", methods.toString());
        
        // Insert and delete operations
        StringBuilder modify = new StringBuilder("Hello World");
        modify.insert(5, " Beautiful");
        System.out.printf("After insert: %s\n", modify.toString());
        
        modify.delete(5, 15);
        System.out.printf("After delete: %s\n", modify.toString());
        
        modify.reverse();
        System.out.printf("After reverse: %s\n", modify.toString());
        
        System.out.println();
    }
    
    public void demonstrateMemoryImpact() {
        System.out.println("=== Memory Impact Analysis ===");
        
        Runtime runtime = Runtime.getRuntime();
        
        // Measure memory before operations
        System.gc(); // Suggest garbage collection
        long beforeMemory = runtime.totalMemory() - runtime.freeMemory();
        
        // Create many strings using concatenation (memory-intensive)
        System.out.println("Creating strings with concatenation...");
        List<String> strings = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            String temp = "";
            for (int j = 0; j < 100; j++) {
                temp += "x"; // Each += creates new String object
            }
            strings.add(temp);
        }
        
        long afterStringMemory = runtime.totalMemory() - runtime.freeMemory();
        
        // Clear and measure StringBuilder approach
        strings.clear();
        System.gc();
        
        System.out.println("Creating strings with StringBuilder...");
        for (int i = 0; i < 1000; i++) {
            StringBuilder sb = new StringBuilder(100); // Pre-size for efficiency
            for (int j = 0; j < 100; j++) {
                sb.append("x");
            }
            strings.add(sb.toString());
        }
        
        long afterBuilderMemory = runtime.totalMemory() - runtime.freeMemory();
        
        System.out.printf("Memory usage comparison:\n");
        System.out.printf("String concatenation: %d KB\n", 
                (afterStringMemory - beforeMemory) / 1024);
        System.out.printf("StringBuilder approach: %d KB\n", 
                (afterBuilderMemory - beforeMemory) / 1024);
        
        // Cleanup
        strings.clear();
        System.gc();
        
        System.out.println();
    }
    
    public void showBestPractices() {
        System.out.println("=== String Handling Best Practices ===");
        
        // 1. Use StringBuilder for multiple concatenations
        System.out.println("1. Multiple Concatenations:");
        
        // ❌ Inefficient
        String inefficient = "";
        long start = System.nanoTime();
        for (int i = 0; i < 1000; i++) {
            inefficient += "item" + i;
        }
        long inefficientTime = System.nanoTime() - start;
        
        // ✅ Efficient
        StringBuilder efficient = new StringBuilder();
        start = System.nanoTime();
        for (int i = 0; i < 1000; i++) {
            efficient.append("item").append(i);
        }
        String efficientResult = efficient.toString();
        long efficientTime = System.nanoTime() - start;
        
        System.out.printf("Inefficient: %.2f ms\n", inefficientTime / 1_000_000.0);
        System.out.printf("Efficient: %.2f ms\n", efficientTime / 1_000_000.0);
        System.out.printf("Improvement: %.1fx faster\n", 
                (double) inefficientTime / efficientTime);
        
        // 2. Pre-size StringBuilder when possible
        System.out.println("\n2. Pre-sizing StringBuilder:");
        
        start = System.nanoTime();
        StringBuilder defaultSize = new StringBuilder();
        for (int i = 0; i < 10000; i++) {
            defaultSize.append("x");
        }
        long defaultTime = System.nanoTime() - start;
        
        start = System.nanoTime();
        StringBuilder preSized = new StringBuilder(10000);
        for (int i = 0; i < 10000; i++) {
            preSized.append("x");
        }
        long preSizedTime = System.nanoTime() - start;
        
        System.out.printf("Default size: %.2f ms\n", defaultTime / 1_000_000.0);
        System.out.printf("Pre-sized: %.2f ms\n", preSizedTime / 1_000_000.0);
        System.out.printf("Improvement: %.1fx faster\n", 
                (double) defaultTime / preSizedTime);
        
        // 3. Use String.join() for joining collections
        System.out.println("\n3. Joining Collections:");
        
        List<String> items = Arrays.asList("apple", "banana", "cherry", "date");
        
        // ✅ Modern approach
        String joined = String.join(", ", items);
        System.out.printf("String.join(): %s\n", joined);
        
        // ✅ Stream approach
        String streamJoined = items.stream()
                .collect(java.util.stream.Collectors.joining(", "));
        System.out.printf("Stream joining: %s\n", streamJoined);
        
        // 4. String formatting
        System.out.println("\n4. String Formatting:");
        
        String name = "Java";
        int version = 17;
        double performance = 95.5;
        
        // Different formatting approaches
        String printf = String.format("Language: %s, Version: %d, Performance: %.1f%%", 
                name, version, performance);
        String textBlock = """
                Language: %s
                Version: %d
                Performance: %.1f%%
                """.formatted(name, version, performance);
        
        System.out.printf("String.format(): %s\n", printf);
        System.out.printf("Text block: %s\n", textBlock.trim());
        
        System.out.println("\n=== Summary of Best Practices ===");
        System.out.println("✅ Use StringBuilder for multiple concatenations");
        System.out.println("✅ Pre-size StringBuilder when length is known");
        System.out.println("✅ Use String.join() for collections");
        System.out.println("✅ Use String.format() for complex formatting");
        System.out.println("✅ Use equals() for content comparison");
        System.out.println("✅ Consider intern() only for limited, repeated strings");
        System.out.println("❌ Avoid + operator in loops");
        System.out.println("❌ Don't use == for content comparison");
        System.out.println("❌ Don't intern() arbitrary user input");
    }
    
    private static final List<String> Arrays.asList(String... values) {
        return java.util.Arrays.asList(values);
    }
}

/*
Performance Summary:
1. String concatenation: O(n²) time complexity
2. StringBuilder: O(n) time complexity  
3. StringBuffer: O(n) time complexity + synchronization overhead
4. String.join(): Optimized for collections
5. Pre-sizing: Reduces internal array copying
*/
```

### Example 3: Advanced String Operations and Encoding

```java
// AdvancedStringDemo.java - Advanced string operations and encoding
import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;
import java.text.Normalizer;
import java.util.*;
import java.util.regex.Pattern;

public class AdvancedStringDemo {
    
    public static void main(String[] args) {
        System.out.println("=== Advanced String Operations ===\n");
        
        AdvancedStringDemo demo = new AdvancedStringDemo();
        demo.demonstrateStringEncoding();
        demo.demonstrateStringNormalization();
        demo.demonstrateRegexOperations();
        demo.demonstrateStringTokenization();
        demo.demonstrateStringValidation();
    }
    
    public void demonstrateStringEncoding() {
        System.out.println("=== String Encoding and Character Sets ===");
        
        String text = "Hello 世界 🌍"; // Mixed ASCII, Unicode, Emoji
        
        System.out.printf("Original string: %s\n", text);
        System.out.printf("String length: %d characters\n", text.length());
        System.out.printf("Code point count: %d\n", text.codePointCount(0, text.length()));
        
        // Different encodings
        Map<String, Charset> charsets = new LinkedHashMap<>();
        charsets.put("UTF-8", StandardCharsets.UTF_8);
        charsets.put("UTF-16", StandardCharsets.UTF_16);
        charsets.put("ISO-8859-1", StandardCharsets.ISO_8859_1);
        charsets.put("US-ASCII", StandardCharsets.US_ASCII);
        
        System.out.println("\nEncoding Analysis:");
        for (Map.Entry<String, Charset> entry : charsets.entrySet()) {
            String charsetName = entry.getKey();
            Charset charset = entry.getValue();
            
            try {
                byte[] bytes = text.getBytes(charset);
                String decoded = new String(bytes, charset);
                
                System.out.printf("%s: %d bytes, decoded correctly: %b\n",
                        charsetName, bytes.length, text.equals(decoded));
                
                // Show byte representation for UTF-8
                if (charset == StandardCharsets.UTF_8) {
                    System.out.printf("UTF-8 bytes: %s\n", 
                            Arrays.toString(Arrays.copyOf(bytes, Math.min(20, bytes.length))));
                }
            } catch (Exception e) {
                System.out.printf("%s: Encoding failed - %s\n", charsetName, e.getMessage());
            }
        }
        
        // Unicode code points
        System.out.println("\nUnicode Analysis:");
        for (int i = 0; i < text.length(); ) {
            int codePoint = text.codePointAt(i);
            char[] chars = Character.toChars(codePoint);
            System.out.printf("'%s' -> U+%04X (%s)\n", 
                    new String(chars), codePoint, Character.getName(codePoint));
            i += Character.charCount(codePoint);
        }
        
        System.out.println();
    }
    
    public void demonstrateStringNormalization() {
        System.out.println("=== String Normalization ===");
        
        // Different ways to represent the same character
        String str1 = "café"; // Single character é
        String str2 = "cafe\u0301"; // e + combining acute accent
        
        System.out.printf("String 1: %s (length: %d)\n", str1, str1.length());
        System.out.printf("String 2: %s (length: %d)\n", str2, str2.length());
        System.out.printf("Are equal: %b\n", str1.equals(str2));
        
        // Normalization forms
        String nfc1 = Normalizer.normalize(str1, Normalizer.Form.NFC);
        String nfc2 = Normalizer.normalize(str2, Normalizer.Form.NFC);
        String nfd1 = Normalizer.normalize(str1, Normalizer.Form.NFD);
        String nfd2 = Normalizer.normalize(str2, Normalizer.Form.NFD);
        
        System.out.println("\nNormalization Results:");
        System.out.printf("NFC form 1: %s (length: %d)\n", nfc1, nfc1.length());
        System.out.printf("NFC form 2: %s (length: %d)\n", nfc2, nfc2.length());
        System.out.printf("NFD form 1: %s (length: %d)\n", nfd1, nfd1.length());
        System.out.printf("NFD form 2: %s (length: %d)\n", nfd2, nfd2.length());
        
        System.out.printf("NFC forms equal: %b\n", nfc1.equals(nfc2));
        System.out.printf("NFD forms equal: %b\n", nfd1.equals(nfd2));
        
        // Case folding for case-insensitive comparison
        String turkish1 = "İstanbul";
        String turkish2 = "ISTANBUL";
        
        System.out.println("\nCase Folding (Turkish Example):");
        System.out.printf("Turkish 1: %s\n", turkish1);
        System.out.printf("Turkish 2: %s\n", turkish2);
        System.out.printf("Simple comparison: %b\n", 
                turkish1.toLowerCase().equals(turkish2.toLowerCase()));
        System.out.printf("Turkish locale: %b\n",
                turkish1.toLowerCase(new Locale("tr", "TR"))
                        .equals(turkish2.toLowerCase(new Locale("tr", "TR"))));
        
        System.out.println();
    }
    
    public void demonstrateRegexOperations() {
        System.out.println("=== Regular Expression Operations ===");
        
        String text = "Contact us at support@example.com or sales@company.org. " +
                     "Phone: +1-555-123-4567 or 555.987.6543";
        
        System.out.printf("Text: %s\n\n", text);
        
        // Email pattern
        Pattern emailPattern = Pattern.compile(
                "\\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Z|a-z]{2,}\\b");
        
        System.out.println("Email addresses found:");
        emailPattern.matcher(text).results()
                .forEach(match -> System.out.printf("  %s (position %d-%d)\n", 
                        match.group(), match.start(), match.end()));
        
        // Phone pattern
        Pattern phonePattern = Pattern.compile(
                "\\+?1?[-.]?\\(?([0-9]{3})\\)?[-.]?([0-9]{3})[-.]?([0-9]{4})");
        
        System.out.println("\nPhone numbers found:");
        phonePattern.matcher(text).results()
                .forEach(match -> System.out.printf("  %s -> (%s) %s-%s\n", 
                        match.group(), match.group(1), match.group(2), match.group(3)));
        
        // String replacement with regex
        String cleaned = text.replaceAll("\\b\\d{3}[-.)]?\\d{3}[-.)]?\\d{4}\\b", "[PHONE]");
        cleaned = cleaned.replaceAll("\\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Z|a-z]{2,}\\b", "[EMAIL]");
        
        System.out.printf("\nSanitized text: %s\n", cleaned);
        
        // Split operations
        String csv = "apple,banana,cherry;date|elderberry:fig";
        String[] fruits = csv.split("[,;|:]");
        
        System.out.println("\nSplit operations:");
        System.out.printf("Original: %s\n", csv);
        System.out.printf("Split result: %s\n", Arrays.toString(fruits));
        
        System.out.println();
    }
    
    public void demonstrateStringTokenization() {
        System.out.println("=== String Tokenization ===");
        
        String data = "Java,Python;C++|JavaScript:TypeScript Ruby";
        
        // StringTokenizer (legacy approach)
        System.out.println("StringTokenizer approach:");
        StringTokenizer tokenizer = new StringTokenizer(data, ",;|: ");
        while (tokenizer.hasMoreTokens()) {
            System.out.printf("  Token: '%s'\n", tokenizer.nextToken());
        }
        
        // Split approach
        System.out.println("\nSplit approach:");
        String[] tokens = data.split("[,;|: ]+");
        Arrays.stream(tokens)
                .forEach(token -> System.out.printf("  Token: '%s'\n", token));
        
        // Scanner approach
        System.out.println("\nScanner approach:");
        Scanner scanner = new Scanner(data);
        scanner.useDelimiter("[,;|: ]+");
        while (scanner.hasNext()) {
            System.out.printf("  Token: '%s'\n", scanner.next());
        }
        scanner.close();
        
        // Stream approach with custom parsing
        System.out.println("\nStream approach:");
        Pattern.compile("[,;|: ]+")
                .splitAsStream(data)
                .filter(s -> !s.isEmpty())
                .forEach(token -> System.out.printf("  Token: '%s'\n", token));
        
        System.out.println();
    }
    
    public void demonstrateStringValidation() {
        System.out.println("=== String Validation Examples ===");
        
        String[] testInputs = {
            "user@example.com",
            "invalid-email",
            "555-123-4567",
            "abc123",
            "",
            null,
            "  whitespace  ",
            "ValidPassword123!",
            "weak"
        };
        
        System.out.println("Validation Results:");
        for (String input : testInputs) {
            System.out.printf("Input: %-20s", input == null ? "null" : "'" + input + "'");
            
            System.out.printf(" | Empty: %-5b", isEmpty(input));
            System.out.printf(" | Email: %-5b", isValidEmail(input));
            System.out.printf(" | Phone: %-5b", isValidPhone(input));
            System.out.printf(" | Strong Password: %-5b", isStrongPassword(input));
            
            System.out.println();
        }
        
        // Input sanitization
        System.out.println("\nInput Sanitization:");
        String userInput = "  <script>alert('xss')</script>Hello World!  ";
        System.out.printf("Original: %s\n", userInput);
        System.out.printf("Trimmed: %s\n", userInput.trim());
        System.out.printf("HTML escaped: %s\n", escapeHtml(userInput.trim()));
        System.out.printf("Alphanumeric only: %s\n", 
                userInput.replaceAll("[^a-zA-Z0-9\\s]", "").trim());
        
        System.out.println();
    }
    
    // Utility validation methods
    private boolean isEmpty(String str) {
        return str == null || str.trim().isEmpty();
    }
    
    private boolean isValidEmail(String email) {
        if (isEmpty(email)) return false;
        Pattern pattern = Pattern.compile(
                "^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$");
        return pattern.matcher(email).matches();
    }
    
    private boolean isValidPhone(String phone) {
        if (isEmpty(phone)) return false;
        Pattern pattern = Pattern.compile(
                "^\\+?1?[-.]?\\(?([0-9]{3})\\)?[-.]?([0-9]{3})[-.]?([0-9]{4})$");
        return pattern.matcher(phone).matches();
    }
    
    private boolean isStrongPassword(String password) {
        if (isEmpty(password) || password.length() < 8) return false;
        
        boolean hasUpper = password.chars().anyMatch(Character::isUpperCase);
        boolean hasLower = password.chars().anyMatch(Character::isLowerCase);
        boolean hasDigit = password.chars().anyMatch(Character::isDigit);
        boolean hasSpecial = password.chars()
                .anyMatch(ch -> "!@#$%^&*()_+-=[]{}|;:,.<>?".indexOf(ch) >= 0);
        
        return hasUpper && hasLower && hasDigit && hasSpecial;
    }
    
    private String escapeHtml(String str) {
        if (str == null) return null;
        return str.replace("&", "&amp;")
                 .replace("<", "&lt;")
                 .replace(">", "&gt;")
                 .replace("\"", "&quot;")
                 .replace("'", "&#x27;");
    }
}

/*
Advanced String Concepts:
1. Character encoding affects byte representation
2. Normalization handles equivalent Unicode forms
3. Regex provides powerful pattern matching
4. Tokenization breaks strings into components
5. Validation ensures data integrity
6. Sanitization prevents security issues
*/
```

## 📊 String Class Comparison

| Class | Mutability | Thread Safety | Performance | Use Case |
|-------|------------|---------------|-------------|----------|
| **String** | Immutable | Thread-safe | Slow for concatenation | Single values, literals |
| **StringBuilder** | Mutable | Not thread-safe | Fast | Multiple concatenations |
| **StringBuffer** | Mutable | Thread-safe | Medium | Concurrent access needed |
| **StringJoiner** | Mutable | Not thread-safe | Optimized for joining | Delimiter-separated values |

## ⚠️ Common String Mistakes

### 1. Using + in loops
```java
// ❌ Inefficient - creates many intermediate objects
String result = "";
for (int i = 0; i < 1000; i++) {
    result += "item" + i;  // O(n²) complexity
}

// ✅ Efficient - single mutable buffer
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append("item").append(i);  // O(n) complexity
}
String result = sb.toString();
```

### 2. Using == for content comparison
```java
// ❌ Compares references, not content
String str1 = new String("Hello");
String str2 = new String("Hello");
if (str1 == str2) { /* false! */ }

// ✅ Compares content
if (str1.equals(str2)) { /* true! */ }

// ✅ Null-safe comparison
if (Objects.equals(str1, str2)) { /* safe */ }
```

### 3. Inefficient String operations
```java
// ❌ Multiple method calls
String text = "  Hello World  ";
text = text.trim();
text = text.toLowerCase();
text = text.replace(" ", "_");

// ✅ Method chaining where possible
String text = "  Hello World  ".trim().toLowerCase().replace(" ", "_");
```

## 🔥 Interview Questions & Answers

### Q1: Why are Strings immutable in Java and what are the benefits?
**Answer**: Strings are immutable for several reasons:
1. **String Pool**: Enables sharing of string literals to save memory
2. **Thread Safety**: Immutable objects are inherently thread-safe
3. **Hashcode Caching**: Hashcode can be cached since content never changes
4. **Security**: Prevents modification of sensitive data like passwords
5. **Class Loading**: String literals used in class loading must be stable

### Q2: When should you use StringBuilder vs StringBuffer?
**Answer**: 
- **StringBuilder**: Single-threaded scenarios, better performance (no synchronization overhead)
- **StringBuffer**: Multi-threaded scenarios where thread safety is required
- **Performance**: StringBuilder is ~15% faster than StringBuffer
- **Modern Practice**: Use StringBuilder unless you specifically need thread safety

### Q3: What is the String Pool and how does intern() work?
**Answer**: String Pool is a memory area in heap storing unique string literals:
- **Automatic**: String literals automatically go to pool
- **intern()**: Adds non-literal strings to pool, returns pool reference
- **Memory Saving**: Multiple variables can reference same pool object
- **Performance**: Reference comparison (==) faster than equals()
- **Caution**: intern() can cause memory leaks if overused

### Q4: Explain the difference between equals() and == for Strings.
**Answer**: 
- **== operator**: Compares object references (memory addresses)
- **equals() method**: Compares actual content of strings
- **Example**: `new String("test") == "test"` is false, but `.equals()` is true
- **Best Practice**: Always use equals() for content comparison
- **Null Safety**: Use Objects.equals() to handle null values

### Q5: How has String implementation changed in recent Java versions?
**Answer**: 
- **Java 6**: String pool moved to heap from PermGen
- **Java 7**: Removed offset and count fields, pool fully in heap
- **Java 8**: PermGen replaced with Metaspace
- **Java 9+**: Compact Strings - uses byte[] instead of char[] when possible
- **Performance**: Latin-1 strings use 50% less memory in Java 9+

---
[← Back to Main Guide](./README.md) | [← Previous: Garbage Collection](./garbage-collection.md) | [Next: Constructors →](./constructors.md)
