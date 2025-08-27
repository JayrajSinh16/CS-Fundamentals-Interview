# 🔀 Multiple Inheritance in Java - Interfaces and Design Patterns

Java doesn't support multiple inheritance of classes but achieves multiple inheritance of type through interfaces. This design choice prevents the "diamond problem" while enabling flexible object-oriented designs through interface implementation and composition patterns.

## 🎯 Key Concepts

- **Multiple Inheritance of Type**: Implementing multiple interfaces
- **Diamond Problem**: Conflict when inheriting same method from multiple sources
- **Interface Composition**: Combining multiple interface contracts
- **Mixin Pattern**: Adding functionality through interfaces
- **Default Method Conflicts**: Resolution strategies for method conflicts
- **Composition over Inheritance**: Preferred design approach

## 🌍 Real-World Analogy

**Professional Skills Analogy**:
- **Multiple Interfaces** = Professional certifications (Java Developer, Project Manager, Team Lead)
- **Diamond Problem** = Conflicting instructions from different certifications
- **Resolution Strategy** = Choosing appropriate skill based on context
- **Composition** = Building team with specialists rather than requiring everyone to have all skills
- **Mixin** = Adding specific capabilities (public speaking, technical writing) to existing roles

## 📋 Multiple Inheritance Architecture

```
Multiple Inheritance in Java:
┌─────────────────────────────────────────────────────────────────┐
│                    Inheritance Types Comparison                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Single Class Inheritance:                                     │
│  class Child extends Parent { }                                │
│  ✅ Supported in Java                                          │
│  ✅ No ambiguity                                               │
│  ❌ Limited flexibility                                        │
│                                                                 │
│  Multiple Class Inheritance (NOT in Java):                     │
│  class Child extends Parent1, Parent2 { }                     │
│  ❌ Not supported in Java                                      │
│  ❌ Diamond problem                                            │
│  ❌ Method/field conflicts                                     │
│                                                                 │
│  Multiple Interface Implementation:                             │
│  class Child implements Interface1, Interface2, Interface3 {   │
│  ✅ Supported in Java                                          │
│  ✅ Type flexibility                                           │
│  ✅ Conflict resolution mechanisms                             │
│                                                                 │
│  Mixed Inheritance:                                             │
│  class Child extends Parent implements Interface1, Interface2  │
│  ✅ Best of both worlds                                        │
│  ✅ Code reuse + flexibility                                   │
└─────────────────────────────────────────────────────────────────┘

Diamond Problem Resolution:
┌─────────────────────────────────────────────────────────────────┐
│                      Conflict Resolution                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│     Interface A              Interface B                       │
│     ┌─────────┐              ┌─────────┐                       │
│     │method() │              │method() │                       │
│     └─────────┘              └─────────┘                       │
│           │                        │                           │
│           └────────┬───────────────┘                           │
│                    │                                           │
│              ┌─────────┐                                       │
│              │Class C  │ ← Must resolve conflict              │
│              │override │   explicitly                         │
│              │method() │                                       │
│              └─────────┘                                       │
│                                                                 │
│  Resolution Strategies:                                         │
│  • Override with custom implementation                         │
│  • Use Interface.super.method() for specific interface         │
│  • Combine behaviors from multiple interfaces                  │
│  • Choose one interface and delegate to it                     │
└─────────────────────────────────────────────────────────────────┘
```

## 💻 Practical Examples

### Example 1: Multiple Interface Implementation

```java
// MultipleInterfaceExample.java
import java.util.*;

public class MultipleInterfaceExample {
    
    // Core behavior interfaces
    interface Readable {
        String read();
        boolean canRead();
        
        default void openForReading() {
            System.out.println("Opening for reading");
        }
    }
    
    interface Writable {
        void write(String content);
        boolean canWrite();
        
        default void openForWriting() {
            System.out.println("Opening for writing");
        }
    }
    
    interface Closeable {
        void close();
        boolean isClosed();
        
        default void cleanup() {
            System.out.println("Performing cleanup");
        }
    }
    
    // Multimedia interfaces
    interface Playable {
        void play();
        void pause();
        void stop();
        
        default boolean isPlaying() {
            return false;
        }
    }
    
    interface Downloadable {
        void download(String url);
        int getDownloadProgress();
        
        default boolean isDownloadComplete() {
            return getDownloadProgress() == 100;
        }
    }
    
    // File implementation with multiple interfaces
    static class TextFile implements Readable, Writable, Closeable {
        private StringBuilder content = new StringBuilder();
        private boolean isOpen = true;
        private boolean readMode = false;
        private boolean writeMode = false;
        
        @Override
        public String read() {
            if (!canRead()) {
                throw new IllegalStateException("Cannot read file");
            }
            return content.toString();
        }
        
        @Override
        public boolean canRead() {
            return isOpen && !writeMode;
        }
        
        @Override
        public void openForReading() {
            Readable.super.openForReading();
            readMode = true;
            writeMode = false;
        }
        
        @Override
        public void write(String newContent) {
            if (!canWrite()) {
                throw new IllegalStateException("Cannot write to file");
            }
            content.append(newContent);
        }
        
        @Override
        public boolean canWrite() {
            return isOpen && !readMode;
        }
        
        @Override
        public void openForWriting() {
            Writable.super.openForWriting();
            writeMode = true;
            readMode = false;
        }
        
        @Override
        public void close() {
            isOpen = false;
            readMode = false;
            writeMode = false;
            System.out.println("Text file closed");
        }
        
        @Override
        public boolean isClosed() {
            return !isOpen;
        }
        
        public void openReadWrite() {
            isOpen = true;
            readMode = false;
            writeMode = false;
            System.out.println("File opened in read-write mode");
        }
    }
    
    // Media file with multiple capabilities
    static class MediaFile implements Readable, Playable, Downloadable, Closeable {
        private String content = "";
        private boolean isOpen = true;
        private boolean playing = false;
        private int downloadProgress = 0;
        
        @Override
        public String read() {
            return "Media metadata: " + content;
        }
        
        @Override
        public boolean canRead() {
            return isOpen;
        }
        
        @Override
        public void play() {
            if (isDownloadComplete()) {
                playing = true;
                System.out.println("Playing media file");
            } else {
                System.out.println("Cannot play - download incomplete");
            }
        }
        
        @Override
        public void pause() {
            playing = false;
            System.out.println("Media paused");
        }
        
        @Override
        public void stop() {
            playing = false;
            System.out.println("Media stopped");
        }
        
        @Override
        public boolean isPlaying() {
            return playing;
        }
        
        @Override
        public void download(String url) {
            System.out.printf("Downloading from: %s%n", url);
            // Simulate download progress
            for (int i = 0; i <= 100; i += 25) {
                downloadProgress = i;
                System.out.printf("Download progress: %d%%%n", i);
            }
            content = "Downloaded content from " + url;
        }
        
        @Override
        public int getDownloadProgress() {
            return downloadProgress;
        }
        
        @Override
        public void close() {
            stop();
            isOpen = false;
            System.out.println("Media file closed");
        }
        
        @Override
        public boolean isClosed() {
            return !isOpen;
        }
    }
    
    public static void demonstrateMultipleInterfaces() {
        System.out.println("=== Multiple Interface Implementation ===");
        
        TextFile textFile = new TextFile();
        MediaFile mediaFile = new MediaFile();
        
        // Demonstrate TextFile capabilities
        System.out.println("\n--- TextFile Operations ---");
        textFile.openForWriting();
        textFile.write("Hello, World!");
        
        textFile.openForReading();
        System.out.println("Content: " + textFile.read());
        
        textFile.cleanup();
        textFile.close();
        
        // Demonstrate MediaFile capabilities
        System.out.println("\n--- MediaFile Operations ---");
        mediaFile.download("https://example.com/video.mp4");
        
        if (mediaFile.isDownloadComplete()) {
            mediaFile.play();
            System.out.println("Is playing: " + mediaFile.isPlaying());
            mediaFile.pause();
        }
        
        System.out.println("Metadata: " + mediaFile.read());
        mediaFile.close();
        
        // Polymorphic usage
        System.out.println("\n--- Polymorphic Usage ---");
        List<Readable> readables = Arrays.asList(textFile, mediaFile);
        List<Closeable> closeables = Arrays.asList(textFile, mediaFile);
        
        for (Readable readable : readables) {
            if (readable.canRead()) {
                System.out.println("Reading: " + readable.read());
            }
        }
    }
    
    public static void main(String[] args) {
        demonstrateMultipleInterfaces();
    }
}
```

### Example 2: Diamond Problem and Resolution

```java
// DiamondProblemResolution.java
public class DiamondProblemResolution {
    
    // Base interfaces with conflicting default methods
    interface Logger {
        default void log(String message) {
            System.out.println("Logger: " + message);
        }
        
        default String getLogLevel() {
            return "INFO";
        }
    }
    
    interface Auditor {
        default void log(String message) {
            System.out.println("Auditor: " + message);
        }
        
        default String getLogLevel() {
            return "AUDIT";
        }
    }
    
    interface Monitor {
        default void log(String message) {
            System.out.println("Monitor: " + message);
        }
        
        default String getLogLevel() {
            return "MONITOR";
        }
    }
    
    // Class implementing multiple interfaces with conflicts
    static class SystemComponent implements Logger, Auditor, Monitor {
        private String componentName;
        private String currentMode = "NORMAL";
        
        public SystemComponent(String componentName) {
            this.componentName = componentName;
        }
        
        // Must override conflicting default methods
        @Override
        public void log(String message) {
            String logMessage = String.format("[%s] %s: %s", 
                    getLogLevel(), componentName, message);
            
            // Choose behavior based on current mode
            switch (currentMode) {
                case "DEBUG":
                    Logger.super.log(logMessage);
                    Monitor.super.log("Monitor: " + message);
                    break;
                case "AUDIT":
                    Auditor.super.log(logMessage);
                    break;
                case "MONITOR":
                    Monitor.super.log(logMessage);
                    break;
                default:
                    // Custom implementation combining all
                    System.out.printf("SYSTEM[%s]: %s%n", componentName, message);
            }
        }
        
        @Override
        public String getLogLevel() {
            return switch (currentMode) {
                case "DEBUG" -> Logger.super.getLogLevel();
                case "AUDIT" -> Auditor.super.getLogLevel();
                case "MONITOR" -> Monitor.super.getLogLevel();
                default -> "SYSTEM";
            };
        }
        
        public void setMode(String mode) {
            this.currentMode = mode;
            log("Mode changed to: " + mode);
        }
        
        public String getMode() {
            return currentMode;
        }
    }
    
    public static void demonstrateDiamondProblem() {
        System.out.println("=== Diamond Problem Resolution ===");
        
        SystemComponent component = new SystemComponent("DatabaseManager");
        
        // Test different modes
        component.log("System initialized");
        
        component.setMode("AUDIT");
        component.log("User login attempt");
        
        component.setMode("DEBUG");
        component.log("Database connection established");
        
        component.setMode("MONITOR");
        component.log("Performance metrics collected");
        
        component.setMode("NORMAL");
        component.log("Regular operation");
    }
    
    public static void main(String[] args) {
        demonstrateDiamondProblem();
    }
}
```

### Example 3: Composition vs Multiple Inheritance

```java
// CompositionVsInheritance.java
public class CompositionVsInheritance {
    
    // Service interfaces
    interface EmailService {
        void sendEmail(String to, String subject, String body);
        boolean isEmailValid(String email);
    }
    
    interface SMSService {
        void sendSMS(String phoneNumber, String message);
        boolean isPhoneNumberValid(String phoneNumber);
    }
    
    interface PushNotificationService {
        void sendPushNotification(String deviceId, String title, String message);
        boolean isDeviceRegistered(String deviceId);
    }
    
    // Implementation using multiple inheritance approach
    static class MultipleInheritanceNotificationManager 
            implements EmailService, SMSService, PushNotificationService {
        
        @Override
        public void sendEmail(String to, String subject, String body) {
            System.out.printf("Email sent to %s: %s%n", to, subject);
        }
        
        @Override
        public boolean isEmailValid(String email) {
            return email != null && email.contains("@");
        }
        
        @Override
        public void sendSMS(String phoneNumber, String message) {
            System.out.printf("SMS sent to %s: %s%n", phoneNumber, message);
        }
        
        @Override
        public boolean isPhoneNumberValid(String phoneNumber) {
            return phoneNumber != null && phoneNumber.matches("\\+?[0-9]{10,15}");
        }
        
        @Override
        public void sendPushNotification(String deviceId, String title, String message) {
            System.out.printf("Push notification sent to %s: %s%n", deviceId, title);
        }
        
        @Override
        public boolean isDeviceRegistered(String deviceId) {
            return deviceId != null && !deviceId.isEmpty();
        }
        
        // Convenience method using multiple capabilities
        public void sendAllNotifications(String email, String phone, String deviceId, 
                                       String subject, String message) {
            if (isEmailValid(email)) {
                sendEmail(email, subject, message);
            }
            if (isPhoneNumberValid(phone)) {
                sendSMS(phone, message);
            }
            if (isDeviceRegistered(deviceId)) {
                sendPushNotification(deviceId, subject, message);
            }
        }
    }
    
    // Implementation using composition approach
    static class CompositionNotificationManager {
        private final EmailService emailService;
        private final SMSService smsService;
        private final PushNotificationService pushService;
        
        public CompositionNotificationManager(EmailService emailService,
                                            SMSService smsService,
                                            PushNotificationService pushService) {
            this.emailService = emailService;
            this.smsService = smsService;
            this.pushService = pushService;
        }
        
        public void sendEmail(String to, String subject, String body) {
            emailService.sendEmail(to, subject, body);
        }
        
        public void sendSMS(String phoneNumber, String message) {
            smsService.sendSMS(phoneNumber, message);
        }
        
        public void sendPushNotification(String deviceId, String title, String message) {
            pushService.sendPushNotification(deviceId, title, message);
        }
        
        public void sendAllNotifications(String email, String phone, String deviceId,
                                       String subject, String message) {
            if (emailService.isEmailValid(email)) {
                emailService.sendEmail(email, subject, message);
            }
            if (smsService.isPhoneNumberValid(phone)) {
                smsService.sendSMS(phone, message);
            }
            if (pushService.isDeviceRegistered(deviceId)) {
                pushService.sendPushNotification(deviceId, subject, message);
            }
        }
    }
    
    // Specific service implementations for composition
    static class GmailService implements EmailService {
        @Override
        public void sendEmail(String to, String subject, String body) {
            System.out.printf("Gmail: Email sent to %s%n", to);
        }
        
        @Override
        public boolean isEmailValid(String email) {
            return email != null && email.contains("@") && email.contains(".");
        }
    }
    
    static class TwilioSMSService implements SMSService {
        @Override
        public void sendSMS(String phoneNumber, String message) {
            System.out.printf("Twilio: SMS sent to %s%n", phoneNumber);
        }
        
        @Override
        public boolean isPhoneNumberValid(String phoneNumber) {
            return phoneNumber != null && phoneNumber.matches("\\+?[0-9]{10,15}");
        }
    }
    
    static class FirebasePushService implements PushNotificationService {
        @Override
        public void sendPushNotification(String deviceId, String title, String message) {
            System.out.printf("Firebase: Push sent to device %s%n", deviceId);
        }
        
        @Override
        public boolean isDeviceRegistered(String deviceId) {
            return deviceId != null && deviceId.startsWith("device_");
        }
    }
    
    public static void demonstrateApproaches() {
        System.out.println("=== Multiple Inheritance vs Composition ===");
        
        // Multiple inheritance approach
        System.out.println("\n--- Multiple Inheritance Approach ---");
        MultipleInheritanceNotificationManager inheritanceManager = 
                new MultipleInheritanceNotificationManager();
        
        inheritanceManager.sendAllNotifications(
                "user@example.com",
                "+1234567890", 
                "device_12345",
                "Welcome!",
                "Thanks for joining!"
        );
        
        // Composition approach
        System.out.println("\n--- Composition Approach ---");
        CompositionNotificationManager compositionManager = 
                new CompositionNotificationManager(
                        new GmailService(),
                        new TwilioSMSService(),
                        new FirebasePushService()
                );
        
        compositionManager.sendAllNotifications(
                "user@example.com",
                "+1234567890",
                "device_12345", 
                "Welcome!",
                "Thanks for joining!"
        );
        
        System.out.println("\n=== Comparison ===");
        System.out.println("Multiple Inheritance:");
        System.out.println("✅ All interfaces in one class");
        System.out.println("✅ Direct interface implementation");
        System.out.println("❌ Tight coupling");
        System.out.println("❌ Harder to test individual services");
        
        System.out.println("\nComposition:");
        System.out.println("✅ Loose coupling");
        System.out.println("✅ Easy to test and mock");
        System.out.println("✅ Can swap implementations");
        System.out.println("❌ More code to write");
    }
    
    public static void main(String[] args) {
        demonstrateApproaches();
    }
}
```

## 📊 Multiple Inheritance Patterns Summary

| Pattern | Use Case | Benefits | Drawbacks |
|---------|----------|----------|-----------|
| **Multiple Interfaces** | Common contracts across unrelated classes | Type flexibility, polymorphism | Method conflicts, complexity |
| **Composition** | Building complex objects from simpler parts | Loose coupling, testability | More code, indirect access |
| **Mixin Interfaces** | Adding specific capabilities | Modular functionality | Interface bloat |
| **Strategy Pattern** | Interchangeable algorithms | Runtime flexibility | Extra abstraction layer |

## ⚠️ Common Mistakes and Best Practices

### Avoid Interface Pollution
```java
// ❌ Bad - too many unrelated methods
interface UserManager {
    void createUser(User user);
    void sendEmail(String email);
    void generateReport();
    void backupDatabase();
}

// ✅ Good - focused interfaces
interface UserRepository { void createUser(User user); }
interface EmailService { void sendEmail(String email); }
interface ReportGenerator { void generateReport(); }
```

## 🔥 Interview Questions & Answers

### Q1: Why doesn't Java support multiple class inheritance?
**Answer**: Java avoids the **diamond problem** where a class inherits the same method from multiple parent classes, creating ambiguity. Instead, Java uses **multiple interface inheritance** with default method conflict resolution.

### Q2: How do you resolve default method conflicts in interfaces?
**Answer**: When multiple interfaces have the same default method, the implementing class must **explicitly override** the method and can use `Interface.super.method()` to call specific implementations.

### Q3: What's the difference between composition and multiple inheritance?
**Answer**: **Composition** uses "has-a" relationships with dependency injection, while **multiple inheritance** uses "is-a" relationships. Composition provides better **loose coupling** and **testability**.

### Q4: When should you choose multiple inheritance over composition?
**Answer**: Use **multiple inheritance** when the class truly "is-a" instance of multiple types and needs polymorphic behavior. Use **composition** for better flexibility and when you need to swap implementations.

### Q5: How do mixin interfaces work in Java?
**Answer**: **Mixin interfaces** add specific capabilities (like `Serializable`, `Comparable`) to classes through default methods, providing **modular functionality** without inheritance hierarchy complications.

---
[← Back to Main Guide](./README.md) | [← Previous: Interfaces](./interfaces.md) | [Next: Collections Framework →](./collections-framework.md)
