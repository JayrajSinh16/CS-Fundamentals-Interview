# Wrapper Classes - Primitive Object Wrappers

## Navigation
- [🏠 Main Menu](README.md)

---

## Table of Contents
1. [Wrapper Classes Overview](#wrapper-classes-overview)
2. [Primitive to Wrapper Mapping](#primitive-to-wrapper-mapping)
3. [Autoboxing and Unboxing](#autoboxing-and-unboxing)
4. [Utility Methods](#utility-methods)
5. [Performance Considerations](#performance-considerations)
6. [Interview Questions](#interview-questions)

---

## Wrapper Classes Overview

### What are Wrapper Classes?
Wrapper classes are object-oriented representations of primitive data types in Java. They "wrap" primitive values in objects, allowing primitives to be used where objects are required.

### Real-world Analogy 📦
Think of wrapper classes like **gift boxes**:
- **Primitive**: The actual gift (int, char, boolean)
- **Wrapper**: The decorative box that holds the gift (Integer, Character, Boolean)
- **Boxing**: Putting the gift in the box
- **Unboxing**: Taking the gift out of the box

You need the box when:
- Shipping gifts (storing in collections)
- Adding gift tags (calling methods)
- Following formal protocols (using generics)

### Why Wrapper Classes?
```java
import java.util.*;

public class WhyWrapperClasses {
    public static void main(String[] args) {
        // 1. Collections require objects, not primitives
        List<Integer> numbers = new ArrayList<>();
        // List<int> numbers = new ArrayList<>(); // Compilation error!
        
        numbers.add(10);    // Autoboxing: int → Integer
        numbers.add(20);
        numbers.add(30);
        
        System.out.println("Numbers in list: " + numbers);
        
        // 2. Generics work only with objects
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Alice", 95);
        scores.put("Bob", 87);
        
        // 3. Null values (primitives can't be null)
        Integer nullableNumber = null;
        // int primitiveNumber = null; // Compilation error!
        
        // 4. Utility methods
        String numberStr = "123";
        int parsed = Integer.parseInt(numberStr);
        System.out.println("Parsed: " + parsed);
        
        // 5. Constants
        System.out.println("Max Integer: " + Integer.MAX_VALUE);
        System.out.println("Min Integer: " + Integer.MIN_VALUE);
        
        demonstrateObjectMethods();
    }
    
    private static void demonstrateObjectMethods() {
        System.out.println("\n=== Object Methods ===");
        
        Integer num1 = 100;
        Integer num2 = 100;
        Integer num3 = new Integer(100);
        
        // toString() method
        System.out.println("num1.toString(): " + num1.toString());
        
        // equals() method
        System.out.println("num1.equals(num2): " + num1.equals(num2));
        System.out.println("num1.equals(num3): " + num1.equals(num3));
        
        // hashCode() method
        System.out.println("num1.hashCode(): " + num1.hashCode());
        
        // compareTo() method
        System.out.println("num1.compareTo(num2): " + num1.compareTo(num2));
    }
}
```

---

## Primitive to Wrapper Mapping

### Complete Mapping Table
```java
public class PrimitiveWrapperMapping {
    public static void main(String[] args) {
        // Primitive → Wrapper Class mapping
        
        // 1. byte → Byte
        byte primitiveByte = 10;
        Byte wrapperByte = Byte.valueOf(primitiveByte);
        System.out.println("byte: " + primitiveByte + " → Byte: " + wrapperByte);
        
        // 2. short → Short  
        short primitiveShort = 1000;
        Short wrapperShort = Short.valueOf(primitiveShort);
        System.out.println("short: " + primitiveShort + " → Short: " + wrapperShort);
        
        // 3. int → Integer
        int primitiveInt = 42;
        Integer wrapperInteger = Integer.valueOf(primitiveInt);
        System.out.println("int: " + primitiveInt + " → Integer: " + wrapperInteger);
        
        // 4. long → Long
        long primitiveLong = 123456789L;
        Long wrapperLong = Long.valueOf(primitiveLong);
        System.out.println("long: " + primitiveLong + " → Long: " + wrapperLong);
        
        // 5. float → Float
        float primitiveFloat = 3.14f;
        Float wrapperFloat = Float.valueOf(primitiveFloat);
        System.out.println("float: " + primitiveFloat + " → Float: " + wrapperFloat);
        
        // 6. double → Double
        double primitiveDouble = 2.71828;
        Double wrapperDouble = Double.valueOf(primitiveDouble);
        System.out.println("double: " + primitiveDouble + " → Double: " + wrapperDouble);
        
        // 7. char → Character
        char primitiveChar = 'A';
        Character wrapperCharacter = Character.valueOf(primitiveChar);
        System.out.println("char: " + primitiveChar + " → Character: " + wrapperCharacter);
        
        // 8. boolean → Boolean
        boolean primitiveBoolean = true;
        Boolean wrapperBoolean = Boolean.valueOf(primitiveBoolean);
        System.out.println("boolean: " + primitiveBoolean + " → Boolean: " + wrapperBoolean);
        
        demonstrateConstructors();
    }
    
    private static void demonstrateConstructors() {
        System.out.println("\n=== Constructor Methods ===");
        
        // Different ways to create wrapper objects
        
        // 1. Constructor (deprecated in Java 9+)
        Integer int1 = new Integer(42);
        Integer int2 = new Integer("42");
        
        // 2. valueOf() method (preferred)
        Integer int3 = Integer.valueOf(42);
        Integer int4 = Integer.valueOf("42");
        Integer int5 = Integer.valueOf("101010", 2); // Binary to decimal
        
        // 3. Autoboxing (automatic)
        Integer int6 = 42;
        
        System.out.println("Constructor: " + int1);
        System.out.println("valueOf(int): " + int3);
        System.out.println("valueOf(String): " + int4);
        System.out.println("valueOf(String, radix): " + int5);
        System.out.println("Autoboxing: " + int6);
        
        // Character has different constructors
        Character char1 = new Character('A');
        Character char2 = Character.valueOf('A');
        Character char3 = 'A'; // Autoboxing
        
        System.out.println("Character examples: " + char1 + ", " + char2 + ", " + char3);
    }
}
```

### Wrapper Class Hierarchy
```java
public class WrapperClassHierarchy {
    public static void main(String[] args) {
        // All wrapper classes extend Object
        // Number class is parent of numeric wrappers
        
        demonstrateNumberClass();
        demonstrateComparable();
    }
    
    private static void demonstrateNumberClass() {
        System.out.println("=== Number Class Methods ===");
        
        Integer integer = 42;
        Double doubleValue = 42.5;
        
        // Number class methods
        System.out.println("Integer as byte: " + integer.byteValue());
        System.out.println("Integer as short: " + integer.shortValue());
        System.out.println("Integer as int: " + integer.intValue());
        System.out.println("Integer as long: " + integer.longValue());
        System.out.println("Integer as float: " + integer.floatValue());
        System.out.println("Integer as double: " + integer.doubleValue());
        
        System.out.println("Double as int: " + doubleValue.intValue());
        System.out.println("Double as long: " + doubleValue.longValue());
    }
    
    private static void demonstrateComparable() {
        System.out.println("\n=== Comparable Interface ===");
        
        // All wrapper classes implement Comparable
        Integer a = 10;
        Integer b = 20;
        Integer c = 10;
        
        System.out.println("a.compareTo(b): " + a.compareTo(b)); // -1 (a < b)
        System.out.println("b.compareTo(a): " + b.compareTo(a)); // 1 (b > a)
        System.out.println("a.compareTo(c): " + a.compareTo(c)); // 0 (a == c)
        
        // Sorting using Comparable
        List<Integer> numbers = Arrays.asList(5, 2, 8, 1, 9);
        Collections.sort(numbers);
        System.out.println("Sorted numbers: " + numbers);
        
        // Character comparison
        Character ch1 = 'A';
        Character ch2 = 'Z';
        System.out.println("'A'.compareTo('Z'): " + ch1.compareTo(ch2));
    }
}
```

---

## Autoboxing and Unboxing

### Automatic Conversion
```java
import java.util.*;

public class AutoboxingUnboxing {
    public static void main(String[] args) {
        demonstrateBasicAutoboxing();
        demonstrateCollectionAutoboxing();
        demonstrateMethodAutoboxing();
        demonstrateArithmeticOperations();
    }
    
    private static void demonstrateBasicAutoboxing() {
        System.out.println("=== Basic Autoboxing/Unboxing ===");
        
        // Autoboxing: primitive → wrapper
        int primitive = 42;
        Integer wrapper = primitive; // Automatic boxing
        System.out.println("Autoboxing: " + primitive + " → " + wrapper);
        
        // Unboxing: wrapper → primitive
        Integer wrapperInt = 100;
        int primitiveInt = wrapperInt; // Automatic unboxing
        System.out.println("Unboxing: " + wrapperInt + " → " + primitiveInt);
        
        // Multiple autoboxing examples
        Boolean bool = true;           // boolean → Boolean
        Character ch = 'A';           // char → Character
        Double d = 3.14;              // double → Double
        
        // Multiple unboxing examples
        boolean boolPrim = bool;      // Boolean → boolean
        char charPrim = ch;           // Character → char
        double doublePrim = d;        // Double → double
        
        System.out.println("Multiple examples:");
        System.out.println("Boolean: " + bool + " → boolean: " + boolPrim);
        System.out.println("Character: " + ch + " → char: " + charPrim);
        System.out.println("Double: " + d + " → double: " + doublePrim);
    }
    
    private static void demonstrateCollectionAutoboxing() {
        System.out.println("\n=== Collection Autoboxing ===");
        
        List<Integer> numbers = new ArrayList<>();
        
        // Adding primitives (autoboxing)
        numbers.add(1);    // int → Integer
        numbers.add(2);
        numbers.add(3);
        
        System.out.println("List: " + numbers);
        
        // Retrieving as primitives (unboxing)
        int first = numbers.get(0);   // Integer → int
        int sum = 0;
        for (int num : numbers) {     // Integer → int in loop
            sum += num;
        }
        
        System.out.println("First element: " + first);
        System.out.println("Sum: " + sum);
        
        // Map with autoboxing
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Alice", 95);      // int → Integer
        scores.put("Bob", 87);
        
        int aliceScore = scores.get("Alice"); // Integer → int
        System.out.println("Alice's score: " + aliceScore);
    }
    
    private static void demonstrateMethodAutoboxing() {
        System.out.println("\n=== Method Parameter Autoboxing ===");
        
        // Method expecting wrapper, passing primitive
        processInteger(42);           // int → Integer
        
        // Method expecting primitive, passing wrapper
        Integer wrapper = 100;
        processPrimitive(wrapper);    // Integer → int
        
        // Varargs with autoboxing
        processNumbers(1, 2, 3, 4, 5); // int → Integer
    }
    
    private static void processInteger(Integer value) {
        System.out.println("Processing Integer: " + value);
    }
    
    private static void processPrimitive(int value) {
        System.out.println("Processing primitive: " + value);
    }
    
    private static void processNumbers(Integer... numbers) {
        System.out.println("Processing varargs: " + Arrays.toString(numbers));
    }
    
    private static void demonstrateArithmeticOperations() {
        System.out.println("\n=== Arithmetic with Wrappers ===");
        
        Integer a = 10;
        Integer b = 20;
        
        // Arithmetic operations (automatic unboxing and boxing)
        Integer sum = a + b;          // Unbox a,b → add → box result
        Integer diff = b - a;         // Unbox b,a → subtract → box result
        Integer product = a * b;      // Unbox a,b → multiply → box result
        
        System.out.println("a + b = " + sum);
        System.out.println("b - a = " + diff);
        System.out.println("a * b = " + product);
        
        // Comparison (unboxing)
        if (a < b) {                  // Unbox both for comparison
            System.out.println("a is less than b");
        }
        
        // Increment/Decrement
        Integer counter = 5;
        counter++;                    // Unbox → increment → box
        System.out.println("Counter after increment: " + counter);
    }
}
```

### Autoboxing Caching
```java
public class AutoboxingCaching {
    public static void main(String[] args) {
        demonstrateIntegerCaching();
        demonstrateOtherWrapperCaching();
        demonstrateCachingLimitations();
    }
    
    private static void demonstrateIntegerCaching() {
        System.out.println("=== Integer Caching (-128 to 127) ===");
        
        // Values in cache range (-128 to 127)
        Integer a1 = 100;
        Integer a2 = 100;
        Integer a3 = Integer.valueOf(100);
        
        System.out.println("a1 == a2: " + (a1 == a2));           // true (same object)
        System.out.println("a1 == a3: " + (a1 == a3));           // true (same object)
        System.out.println("a1.equals(a2): " + a1.equals(a2));   // true (value comparison)
        
        // Values outside cache range
        Integer b1 = 200;
        Integer b2 = 200;
        Integer b3 = Integer.valueOf(200);
        
        System.out.println("b1 == b2: " + (b1 == b2));           // false (different objects)
        System.out.println("b1 == b3: " + (b1 == b3));           // false (different objects)
        System.out.println("b1.equals(b2): " + b1.equals(b2));   // true (value comparison)
        
        // Constructor always creates new object
        Integer c1 = new Integer(100);
        Integer c2 = new Integer(100);
        Integer c3 = 100;
        
        System.out.println("c1 == c2: " + (c1 == c2));           // false (different objects)
        System.out.println("c1 == c3: " + (c1 == c3));           // false (different objects)
        System.out.println("c1.equals(c3): " + c1.equals(c3));   // true (value comparison)
    }
    
    private static void demonstrateOtherWrapperCaching() {
        System.out.println("\n=== Other Wrapper Caching ===");
        
        // Boolean - TRUE and FALSE are cached
        Boolean bool1 = true;
        Boolean bool2 = Boolean.valueOf(true);
        System.out.println("Boolean true caching: " + (bool1 == bool2)); // true
        
        // Character - 0 to 127 are cached
        Character char1 = 'A';  // 65
        Character char2 = Character.valueOf('A');
        System.out.println("Character 'A' caching: " + (char1 == char2)); // true
        
        Character char3 = (char) 200;
        Character char4 = Character.valueOf((char) 200);
        System.out.println("Character 200 caching: " + (char3 == char4)); // false
        
        // Byte - all values (-128 to 127) are cached
        Byte byte1 = 50;
        Byte byte2 = Byte.valueOf((byte) 50);
        System.out.println("Byte 50 caching: " + (byte1 == byte2)); // true
        
        // Short - -128 to 127 are cached
        Short short1 = 100;
        Short short2 = Short.valueOf((short) 100);
        System.out.println("Short 100 caching: " + (short1 == short2)); // true
        
        Short short3 = 200;
        Short short4 = Short.valueOf((short) 200);
        System.out.println("Short 200 caching: " + (short3 == short4)); // false
    }
    
    private static void demonstrateCachingLimitations() {
        System.out.println("\n=== Caching Best Practices ===");
        
        // Always use equals() for value comparison
        Integer x = 1000;
        Integer y = 1000;
        
        System.out.println("Wrong way - using ==:");
        System.out.println("x == y: " + (x == y)); // false (unreliable)
        
        System.out.println("Correct way - using equals():");
        System.out.println("x.equals(y): " + x.equals(y)); // true (reliable)
        
        // Null safety
        Integer nullInt = null;
        Integer nonNullInt = 42;
        
        // Safe comparison
        System.out.println("Safe null comparison: " + Objects.equals(nullInt, nonNullInt));
        
        // Unsafe comparison (would throw NullPointerException)
        // System.out.println("Unsafe: " + nullInt.equals(nonNullInt));
    }
}
```

---

## Utility Methods

### Parsing and String Conversion
```java
public class WrapperUtilityMethods {
    public static void main(String[] args) {
        demonstrateParsingMethods();
        demonstrateStringConversion();
        demonstrateRadixOperations();
        demonstrateComparisonMethods();
        demonstrateConstantsAndValidation();
    }
    
    private static void demonstrateParsingMethods() {
        System.out.println("=== Parsing Methods ===");
        
        // Integer parsing
        String intStr = "123";
        int parsedInt = Integer.parseInt(intStr);
        Integer wrapperInt = Integer.valueOf(intStr);
        
        System.out.println("parseInt(\"123\"): " + parsedInt);
        System.out.println("valueOf(\"123\"): " + wrapperInt);
        
        // Different numeric types
        double parsedDouble = Double.parseDouble("3.14159");
        float parsedFloat = Float.parseFloat("2.718f");
        long parsedLong = Long.parseLong("9876543210");
        boolean parsedBoolean = Boolean.parseBoolean("true");
        
        System.out.println("parseDouble(\"3.14159\"): " + parsedDouble);
        System.out.println("parseFloat(\"2.718f\"): " + parsedFloat);
        System.out.println("parseLong(\"9876543210\"): " + parsedLong);
        System.out.println("parseBoolean(\"true\"): " + parsedBoolean);
        
        // Error handling
        try {
            int invalid = Integer.parseInt("not_a_number");
        } catch (NumberFormatException e) {
            System.out.println("NumberFormatException: " + e.getMessage());
        }
    }
    
    private static void demonstrateStringConversion() {
        System.out.println("\n=== String Conversion ===");
        
        // toString() methods
        int num = 42;
        String str1 = Integer.toString(num);
        String str2 = String.valueOf(num);
        String str3 = "" + num; // String concatenation
        
        System.out.println("Integer.toString(42): " + str1);
        System.out.println("String.valueOf(42): " + str2);
        System.out.println("\"\" + 42: " + str3);
        
        // Different formats
        double pi = Math.PI;
        System.out.println("Double.toString(PI): " + Double.toString(pi));
        
        boolean flag = true;
        System.out.println("Boolean.toString(true): " + Boolean.toString(flag));
        
        char letter = 'A';
        System.out.println("Character.toString('A'): " + Character.toString(letter));
    }
    
    private static void demonstrateRadixOperations() {
        System.out.println("\n=== Radix Operations ===");
        
        int number = 255;
        
        // Convert to different bases
        String binary = Integer.toBinaryString(number);
        String octal = Integer.toOctalString(number);
        String hex = Integer.toHexString(number);
        
        System.out.println("Number 255 in different bases:");
        System.out.println("Binary: " + binary);
        System.out.println("Octal: " + octal);
        System.out.println("Hexadecimal: " + hex);
        
        // Custom radix
        String base16 = Integer.toString(number, 16);
        String base8 = Integer.toString(number, 8);
        String base2 = Integer.toString(number, 2);
        
        System.out.println("Using toString(num, radix):");
        System.out.println("Base 16: " + base16);
        System.out.println("Base 8: " + base8);
        System.out.println("Base 2: " + base2);
        
        // Parse from different bases
        int fromBinary = Integer.parseInt("11111111", 2);
        int fromOctal = Integer.parseInt("377", 8);
        int fromHex = Integer.parseInt("FF", 16);
        
        System.out.println("Parsing from different bases:");
        System.out.println("Binary \"11111111\": " + fromBinary);
        System.out.println("Octal \"377\": " + fromOctal);
        System.out.println("Hex \"FF\": " + fromHex);
    }
    
    private static void demonstrateComparisonMethods() {
        System.out.println("\n=== Comparison Methods ===");
        
        // Integer comparison
        int a = 10, b = 20, c = 10;
        
        System.out.println("Integer.compare(10, 20): " + Integer.compare(a, b)); // -1
        System.out.println("Integer.compare(20, 10): " + Integer.compare(b, a)); // 1
        System.out.println("Integer.compare(10, 10): " + Integer.compare(a, c)); // 0
        
        // Double comparison (handles NaN and infinity)
        double d1 = 3.14, d2 = Double.NaN, d3 = Double.POSITIVE_INFINITY;
        
        System.out.println("Double.compare(3.14, NaN): " + Double.compare(d1, d2));
        System.out.println("Double.compare(NaN, 3.14): " + Double.compare(d2, d1));
        System.out.println("Double.compare(3.14, POSITIVE_INFINITY): " + Double.compare(d1, d3));
        
        // Boolean comparison
        System.out.println("Boolean.compare(true, false): " + Boolean.compare(true, false)); // 1
        System.out.println("Boolean.compare(false, true): " + Boolean.compare(false, true)); // -1
    }
    
    private static void demonstrateConstantsAndValidation() {
        System.out.println("\n=== Constants and Validation ===");
        
        // Numeric constants
        System.out.println("Integer.MAX_VALUE: " + Integer.MAX_VALUE);
        System.out.println("Integer.MIN_VALUE: " + Integer.MIN_VALUE);
        System.out.println("Integer.SIZE: " + Integer.SIZE + " bits");
        System.out.println("Integer.BYTES: " + Integer.BYTES + " bytes");
        
        System.out.println("Double.MAX_VALUE: " + Double.MAX_VALUE);
        System.out.println("Double.MIN_VALUE: " + Double.MIN_VALUE);
        System.out.println("Double.MIN_NORMAL: " + Double.MIN_NORMAL);
        System.out.println("Double.POSITIVE_INFINITY: " + Double.POSITIVE_INFINITY);
        System.out.println("Double.NEGATIVE_INFINITY: " + Double.NEGATIVE_INFINITY);
        System.out.println("Double.NaN: " + Double.NaN);
        
        // Character constants
        System.out.println("Character.MIN_VALUE: " + (int)Character.MIN_VALUE);
        System.out.println("Character.MAX_VALUE: " + (int)Character.MAX_VALUE);
        System.out.println("Character.SIZE: " + Character.SIZE + " bits");
        
        // Validation methods
        double testValue = Double.NaN;
        System.out.println("Double.isNaN(NaN): " + Double.isNaN(testValue));
        System.out.println("Double.isInfinite(POSITIVE_INFINITY): " + Double.isInfinite(Double.POSITIVE_INFINITY));
        System.out.println("Double.isFinite(3.14): " + Double.isFinite(3.14));
        
        // Character validation
        char testChar = 'A';
        System.out.println("Character.isLetter('A'): " + Character.isLetter(testChar));
        System.out.println("Character.isDigit('5'): " + Character.isDigit('5'));
        System.out.println("Character.isWhitespace(' '): " + Character.isWhitespace(' '));
        System.out.println("Character.isUpperCase('A'): " + Character.isUpperCase(testChar));
        System.out.println("Character.toLowerCase('A'): " + Character.toLowerCase(testChar));
    }
}
```

### Advanced Utility Operations
```java
import java.util.*;

public class AdvancedWrapperUtilities {
    public static void main(String[] args) {
        demonstrateMathOperations();
        demonstrateBitOperations();
        demonstrateSpecialValues();
        demonstrateUnsignedOperations();
    }
    
    private static void demonstrateMathOperations() {
        System.out.println("=== Math Operations ===");
        
        // Min/Max operations
        int min = Integer.min(10, 20);
        int max = Integer.max(10, 20);
        long minLong = Long.min(100L, 200L);
        double maxDouble = Double.max(3.14, 2.71);
        
        System.out.println("Integer.min(10, 20): " + min);
        System.out.println("Integer.max(10, 20): " + max);
        System.out.println("Long.min(100L, 200L): " + minLong);
        System.out.println("Double.max(3.14, 2.71): " + maxDouble);
        
        // Sum operations (overflow-aware)
        try {
            int sum = Math.addExact(Integer.MAX_VALUE, 1);
        } catch (ArithmeticException e) {
            System.out.println("Overflow detected: " + e.getMessage());
        }
        
        // Safe operations
        int a = 1000000;
        int b = 2000;
        long safeMult = Math.multiplyExact((long)a, (long)b);
        System.out.println("Safe multiplication: " + safeMult);
    }
    
    private static void demonstrateBitOperations() {
        System.out.println("\n=== Bit Operations ===");
        
        int number = 42; // Binary: 101010
        
        // Bit counting
        System.out.println("Number: " + number + " (Binary: " + Integer.toBinaryString(number) + ")");
        System.out.println("Number of 1 bits: " + Integer.bitCount(number));
        System.out.println("Number of leading zeros: " + Integer.numberOfLeadingZeros(number));
        System.out.println("Number of trailing zeros: " + Integer.numberOfTrailingZeros(number));
        
        // Bit manipulation
        int reversed = Integer.reverse(number);
        int rotatedLeft = Integer.rotateLeft(number, 2);
        int rotatedRight = Integer.rotateRight(number, 2);
        
        System.out.println("Reversed bits: " + reversed + " (Binary: " + Integer.toBinaryString(reversed) + ")");
        System.out.println("Rotated left by 2: " + rotatedLeft + " (Binary: " + Integer.toBinaryString(rotatedLeft) + ")");
        System.out.println("Rotated right by 2: " + rotatedRight + " (Binary: " + Integer.toBinaryString(rotatedRight) + ")");
        
        // Highest/Lowest bit
        System.out.println("Highest one bit: " + Integer.highestOneBit(number));
        System.out.println("Lowest one bit: " + Integer.lowestOneBit(number));
    }
    
    private static void demonstrateSpecialValues() {
        System.out.println("\n=== Special Values ===");
        
        // Double special values
        double positiveInf = Double.POSITIVE_INFINITY;
        double negativeInf = Double.NEGATIVE_INFINITY;
        double nan = Double.NaN;
        
        System.out.println("Special double values:");
        System.out.println("POSITIVE_INFINITY: " + positiveInf);
        System.out.println("NEGATIVE_INFINITY: " + negativeInf);
        System.out.println("NaN: " + nan);
        
        // Testing special values
        System.out.println("1.0 / 0.0 = " + (1.0 / 0.0));
        System.out.println("-1.0 / 0.0 = " + (-1.0 / 0.0));
        System.out.println("0.0 / 0.0 = " + (0.0 / 0.0));
        
        // NaN comparisons
        System.out.println("NaN == NaN: " + (nan == nan)); // false!
        System.out.println("Double.isNaN(NaN): " + Double.isNaN(nan)); // true
        
        // Float special values
        float floatNaN = Float.NaN;
        System.out.println("Float.isNaN(Float.NaN): " + Float.isNaN(floatNaN));
        
        // Subnormal numbers
        double minNormal = Double.MIN_NORMAL;
        double minValue = Double.MIN_VALUE;
        System.out.println("MIN_NORMAL: " + minNormal);
        System.out.println("MIN_VALUE (subnormal): " + minValue);
    }
    
    private static void demonstrateUnsignedOperations() {
        System.out.println("\n=== Unsigned Operations (Java 8+) ===");
        
        // Unsigned integer operations
        int signedValue = -1;
        String unsignedString = Integer.toUnsignedString(signedValue);
        long unsignedLong = Integer.toUnsignedLong(signedValue);
        
        System.out.println("Signed value: " + signedValue);
        System.out.println("As unsigned string: " + unsignedString);
        System.out.println("As unsigned long: " + unsignedLong);
        
        // Unsigned comparison
        int a = -1;  // As unsigned: 4294967295
        int b = 1;   // As unsigned: 1
        
        System.out.println("Signed comparison (-1 < 1): " + (a < b));
        System.out.println("Unsigned comparison: " + (Integer.compareUnsigned(a, b) < 0));
        
        // Unsigned division
        int dividend = -2;  // As unsigned: 4294967294
        int divisor = 2;
        int unsignedQuotient = Integer.divideUnsigned(dividend, divisor);
        int unsignedRemainder = Integer.remainderUnsigned(dividend, divisor);
        
        System.out.println("Unsigned division: " + Integer.toUnsignedString(dividend) + " / " + divisor + " = " + unsignedQuotient);
        System.out.println("Unsigned remainder: " + unsignedRemainder);
        
        // Parsing unsigned
        try {
            int parsed = Integer.parseUnsignedInt("4294967295"); // Max unsigned int
            System.out.println("Parsed unsigned: " + Integer.toUnsignedString(parsed));
        } catch (NumberFormatException e) {
            System.out.println("Parse error: " + e.getMessage());
        }
    }
}
```

---

## Performance Considerations

### Memory and Performance Impact
```java
import java.util.*;

public class WrapperPerformance {
    private static final int ITERATIONS = 10_000_000;
    
    public static void main(String[] args) {
        benchmarkAutoboxing();
        benchmarkMemoryUsage();
        benchmarkCollections();
        demonstrateBestPractices();
    }
    
    private static void benchmarkAutoboxing() {
        System.out.println("=== Autoboxing Performance ===");
        
        // Primitive operations
        long start = System.nanoTime();
        int sum1 = 0;
        for (int i = 0; i < ITERATIONS; i++) {
            sum1 += i;
        }
        long primitiveTime = System.nanoTime() - start;
        
        // Wrapper operations (with autoboxing)
        start = System.nanoTime();
        Integer sum2 = 0;
        for (Integer i = 0; i < ITERATIONS; i++) { // Autoboxing overhead
            sum2 += i; // Unboxing + boxing overhead
        }
        long wrapperTime = System.nanoTime() - start;
        
        System.out.printf("Primitive sum: %d (Time: %.2f ms)%n", sum1, primitiveTime / 1_000_000.0);
        System.out.printf("Wrapper sum: %d (Time: %.2f ms)%n", sum2, wrapperTime / 1_000_000.0);
        System.out.printf("Wrapper is %.1fx slower%n", (double)wrapperTime / primitiveTime);
    }
    
    private static void benchmarkMemoryUsage() {
        System.out.println("\n=== Memory Usage Comparison ===");
        
        Runtime runtime = Runtime.getRuntime();
        
        // Baseline memory
        System.gc();
        long baseMem = runtime.totalMemory() - runtime.freeMemory();
        
        // Create primitive array
        System.gc();
        long beforePrimitive = runtime.totalMemory() - runtime.freeMemory();
        int[] primitiveArray = new int[1_000_000];
        for (int i = 0; i < primitiveArray.length; i++) {
            primitiveArray[i] = i;
        }
        System.gc();
        long afterPrimitive = runtime.totalMemory() - runtime.freeMemory();
        
        // Create wrapper array
        System.gc();
        long beforeWrapper = runtime.totalMemory() - runtime.freeMemory();
        Integer[] wrapperArray = new Integer[1_000_000];
        for (int i = 0; i < wrapperArray.length; i++) {
            wrapperArray[i] = i; // Autoboxing
        }
        System.gc();
        long afterWrapper = runtime.totalMemory() - runtime.freeMemory();
        
        long primitiveMemory = afterPrimitive - beforePrimitive;
        long wrapperMemory = afterWrapper - beforeWrapper;
        
        System.out.printf("Primitive array memory: %d bytes%n", primitiveMemory);
        System.out.printf("Wrapper array memory: %d bytes%n", wrapperMemory);
        System.out.printf("Wrapper uses %.1fx more memory%n", (double)wrapperMemory / primitiveMemory);
        
        // Clean up
        primitiveArray = null;
        wrapperArray = null;
        System.gc();
    }
    
    private static void benchmarkCollections() {
        System.out.println("\n=== Collection Performance ===");
        
        int size = 1_000_000;
        
        // ArrayList with autoboxing
        long start = System.nanoTime();
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            list.add(i); // Autoboxing
        }
        long sum = 0;
        for (Integer i : list) {
            sum += i; // Unboxing
        }
        long arrayListTime = System.nanoTime() - start;
        
        // Primitive array
        start = System.nanoTime();
        int[] array = new int[size];
        for (int i = 0; i < size; i++) {
            array[i] = i;
        }
        long primitiveSum = 0;
        for (int i : array) {
            primitiveSum += i;
        }
        long arrayTime = System.nanoTime() - start;
        
        System.out.printf("ArrayList<Integer>: %.2f ms (sum: %d)%n", arrayListTime / 1_000_000.0, sum);
        System.out.printf("int[]: %.2f ms (sum: %d)%n", arrayTime / 1_000_000.0, primitiveSum);
        System.out.printf("Primitive array is %.1fx faster%n", (double)arrayListTime / arrayTime);
    }
    
    private static void demonstrateBestPractices() {
        System.out.println("\n=== Best Practices ===");
        
        // 1. Avoid unnecessary boxing in loops
        System.out.println("❌ Bad: Unnecessary boxing in loop");
        System.out.println("for (Integer i = 0; i < 1000; i++) { ... }");
        
        System.out.println("✅ Good: Use primitive in loop");
        System.out.println("for (int i = 0; i < 1000; i++) { ... }");
        
        // 2. Use primitive collections when possible
        System.out.println("\n❌ Bad: Wrapper collections for large datasets");
        System.out.println("List<Integer> numbers = new ArrayList<>();");
        
        System.out.println("✅ Good: Consider primitive collections");
        System.out.println("// Use libraries like Eclipse Collections, Trove, or HPPC");
        System.out.println("// Or use arrays when appropriate");
        
        // 3. Be aware of caching
        System.out.println("\n❌ Bad: Using == for wrapper comparison");
        System.out.println("Integer a = 1000; Integer b = 1000;");
        System.out.println("if (a == b) { ... } // May be false!");
        
        System.out.println("✅ Good: Use equals() for value comparison");
        System.out.println("if (a.equals(b)) { ... } // Always correct");
        
        // 4. Null safety
        System.out.println("\n❌ Bad: No null checks");
        System.out.println("Integer value = getValue();");
        System.out.println("int primitive = value; // NPE if value is null");
        
        System.out.println("✅ Good: Null safety");
        System.out.println("Integer value = getValue();");
        System.out.println("int primitive = (value != null) ? value : 0;");
        
        // Demonstrate null safety
        demonstrateNullSafety();
    }
    
    private static void demonstrateNullSafety() {
        System.out.println("\n=== Null Safety Examples ===");
        
        Integer nullableInt = null;
        
        // Safe unboxing
        int safeValue = (nullableInt != null) ? nullableInt : 0;
        System.out.println("Safe unboxing result: " + safeValue);
        
        // Using Optional (Java 8+)
        Optional<Integer> optionalInt = Optional.ofNullable(nullableInt);
        int defaultValue = optionalInt.orElse(42);
        System.out.println("Optional with default: " + defaultValue);
        
        // Safe arithmetic
        Integer a = 10;
        Integer b = null;
        
        Integer safeSum = (a != null && b != null) ? a + b : null;
        System.out.println("Safe sum (10 + null): " + safeSum);
        
        // Objects.equals for null-safe comparison
        System.out.println("Objects.equals(null, null): " + Objects.equals(nullableInt, null));
        System.out.println("Objects.equals(10, null): " + Objects.equals(10, null));
    }
}
```

---

## Interview Questions

### Q1: What are wrapper classes and why do we need them?
**Answer:**
Wrapper classes are object representations of primitive data types. We need them because:

1. **Collections require objects**: `List<int>` is invalid, must use `List<Integer>`
2. **Generics work only with objects**: `Map<String, int>` is invalid
3. **Object methods**: `toString()`, `equals()`, `hashCode()`
4. **Utility methods**: `parseInt()`, `valueOf()`, constants
5. **Null values**: Primitives can't be null, wrappers can

### Q2: Explain autoboxing and unboxing with examples.
**Answer:**
```java
// Autoboxing: primitive → wrapper (automatic)
int primitive = 42;
Integer wrapper = primitive; // Autoboxing

// Unboxing: wrapper → primitive (automatic)
Integer wrapperInt = 100;
int primitiveInt = wrapperInt; // Unboxing

// In collections
List<Integer> list = new ArrayList<>();
list.add(10); // Autoboxing: int → Integer

// In arithmetic
Integer a = 5, b = 10;
Integer sum = a + b; // Unboxing → arithmetic → autoboxing
```

### Q3: What is the Integer cache and how does it work?
**Answer:**
Integer cache stores Integer objects for values from -128 to 127:

```java
Integer a = 100; // From cache
Integer b = 100; // Same object from cache
System.out.println(a == b); // true (same object)

Integer x = 200; // New object
Integer y = 200; // New object  
System.out.println(x == y); // false (different objects)

// Always use equals() for value comparison
System.out.println(x.equals(y)); // true (value comparison)
```

**Cache ranges:**
- Integer, Short, Byte: -128 to 127
- Character: 0 to 127
- Boolean: TRUE and FALSE are cached

### Q4: What are the performance implications of using wrapper classes?
**Answer:**

| Aspect | Primitive | Wrapper | Impact |
|--------|-----------|---------|--------|
| **Memory** | 4 bytes (int) | 16+ bytes (Integer) | 4x more memory |
| **Speed** | Direct operations | Boxing/unboxing overhead | 2-10x slower |
| **Null safety** | Cannot be null | Can be null | NPE risk |
| **Collections** | Not supported | Required | No choice |

**Best practices:**
- Use primitives for loops and calculations
- Use wrappers only when necessary (collections, generics)
- Be careful with null values

### Q5: How do you safely handle null wrapper values?
**Answer:**
```java
// Problem: NPE risk
Integer value = getValue(); // Might return null
int primitive = value; // NPE if value is null

// Solutions:
// 1. Null check
int safe1 = (value != null) ? value : 0;

// 2. Optional (Java 8+)
int safe2 = Optional.ofNullable(value).orElse(0);

// 3. Objects.equals for comparison
boolean equal = Objects.equals(value1, value2); // Null-safe

// 4. Defensive programming
public void processValue(Integer input) {
    if (input == null) {
        throw new IllegalArgumentException("Value cannot be null");
    }
    int primitive = input; // Safe unboxing
}
```

### Q6: What utility methods do wrapper classes provide?
**Answer:**
```java
// Parsing
int num = Integer.parseInt("123");
double d = Double.parseDouble("3.14");

// String conversion  
String str = Integer.toString(42);
String binary = Integer.toBinaryString(255); // "11111111"
String hex = Integer.toHexString(255);       // "ff"

// Comparison
int result = Integer.compare(10, 20); // -1, 0, or 1

// Constants
int maxInt = Integer.MAX_VALUE;
int minInt = Integer.MIN_VALUE;
int bits = Integer.SIZE; // 32

// Validation
boolean isNaN = Double.isNaN(Double.NaN);
boolean isInfinite = Double.isInfinite(Double.POSITIVE_INFINITY);

// Character utilities
boolean isDigit = Character.isDigit('5');
boolean isLetter = Character.isLetter('A');
char upper = Character.toUpperCase('a');
```

---

## Navigation
- [🏠 Main Menu](README.md)

---

*This comprehensive guide covers wrapper classes in Java. Understanding autoboxing, unboxing, performance implications, and utility methods is crucial for efficient Java programming and technical interviews.*
