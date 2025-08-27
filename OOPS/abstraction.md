# 🎭 Abstraction - The Fourth Pillar of OOP

Abstraction is the process of hiding implementation details while showing only essential features to the user. It focuses on "what" an object does rather than "how" it does it. Think of it as a car's steering wheel - you know turning it changes direction, but you don't need to understand the complex steering mechanism underneath.

## 🎯 Key Concepts

- **Data Abstraction**: Hiding internal data and showing only necessary information
- **Process Abstraction**: Hiding implementation details of methods
- **Abstract Classes**: Classes that cannot be instantiated, used as base classes
- **Interfaces**: Pure abstraction defining contracts without implementation
- **Concrete Classes**: Classes that provide complete implementation

## 🌍 Real-World Analogy

**Television Remote**: You press buttons to change channels, adjust volume, or turn on/off the TV. You don't need to know about the infrared signals, electronic circuits, or how the TV processes these commands. The remote provides an abstract interface to control complex TV functionality.

## 📚 Ways to Achieve Abstraction in Java

### 1. Abstract Classes (0-100% abstraction)
- Can have abstract and concrete methods
- Cannot be instantiated
- Can have constructors, instance variables
- Can provide partial implementation

### 2. Interfaces (100% abstraction)
- All methods are abstract by default (before Java 8)
- Cannot be instantiated
- Variables are public, static, final by default
- Pure contracts defining behavior

## 💻 Practical Examples

### Example 1: Abstract Classes - Database Connection

```java
// Abstract base class defining database operations
abstract class DatabaseConnection {
    protected String hostname;
    protected String username;
    protected String password;
    protected boolean isConnected;
    
    // Constructor in abstract class
    public DatabaseConnection(String hostname, String username, String password) {
        this.hostname = hostname;
        this.username = username;
        this.password = password;
        this.isConnected = false;
    }
    
    // Abstract methods - must be implemented by subclasses
    public abstract boolean connect();
    public abstract void disconnect();
    public abstract boolean executeQuery(String query);
    public abstract boolean executeUpdate(String updateQuery);
    
    // Concrete method - common implementation for all databases
    public void displayConnectionInfo() {
        System.out.printf("Database: %s@%s - Status: %s%n", 
                username, hostname, isConnected ? "Connected" : "Disconnected");
    }
    
    // Template method using abstract methods
    public final boolean performTransaction(String[] queries) {
        if (!connect()) {
            System.out.println("Failed to connect to database");
            return false;
        }
        
        System.out.println("Starting transaction...");
        boolean allSuccessful = true;
        
        for (String query : queries) {
            if (!executeQuery(query)) {
                allSuccessful = false;
                System.out.println("Transaction failed, rolling back...");
                break;
            }
        }
        
        if (allSuccessful) {
            System.out.println("Transaction completed successfully");
        }
        
        disconnect();
        return allSuccessful;
    }
    
    // Concrete helper methods
    protected boolean validateCredentials() {
        return username != null && !username.isEmpty() && 
               password != null && !password.isEmpty();
    }
    
    protected void logOperation(String operation) {
        System.out.println("LOG: " + operation + " at " + new java.util.Date());
    }
}

// MySQL implementation
class MySQLConnection extends DatabaseConnection {
    private int port;
    private String database;
    
    public MySQLConnection(String hostname, String username, String password, 
                          int port, String database) {
        super(hostname, username, password);
        this.port = port;
        this.database = database;
    }
    
    @Override
    public boolean connect() {
        if (!validateCredentials()) {
            System.out.println("Invalid MySQL credentials");
            return false;
        }
        
        System.out.printf("Connecting to MySQL: %s:%d/%s%n", hostname, port, database);
        // Simulate MySQL connection logic
        isConnected = true;
        logOperation("MySQL connection established");
        return true;
    }
    
    @Override
    public void disconnect() {
        if (isConnected) {
            System.out.println("Disconnecting from MySQL database");
            isConnected = false;
            logOperation("MySQL connection closed");
        }
    }
    
    @Override
    public boolean executeQuery(String query) {
        if (!isConnected) {
            System.out.println("Not connected to MySQL database");
            return false;
        }
        
        System.out.println("Executing MySQL query: " + query);
        // Simulate MySQL query execution
        logOperation("MySQL query executed: " + query);
        return true;
    }
    
    @Override
    public boolean executeUpdate(String updateQuery) {
        if (!isConnected) {
            System.out.println("Not connected to MySQL database");
            return false;
        }
        
        System.out.println("Executing MySQL update: " + updateQuery);
        // Simulate MySQL update execution
        logOperation("MySQL update executed: " + updateQuery);
        return true;
    }
    
    // MySQL-specific method
    public void optimizeTables() {
        if (isConnected) {
            System.out.println("Optimizing MySQL tables");
        }
    }
}

// PostgreSQL implementation
class PostgreSQLConnection extends DatabaseConnection {
    private String schema;
    private boolean useSSL;
    
    public PostgreSQLConnection(String hostname, String username, String password,
                               String schema, boolean useSSL) {
        super(hostname, username, password);
        this.schema = schema;
        this.useSSL = useSSL;
    }
    
    @Override
    public boolean connect() {
        if (!validateCredentials()) {
            System.out.println("Invalid PostgreSQL credentials");
            return false;
        }
        
        System.out.printf("Connecting to PostgreSQL: %s (SSL: %s, Schema: %s)%n", 
                hostname, useSSL ? "enabled" : "disabled", schema);
        // Simulate PostgreSQL connection logic
        isConnected = true;
        logOperation("PostgreSQL connection established");
        return true;
    }
    
    @Override
    public void disconnect() {
        if (isConnected) {
            System.out.println("Disconnecting from PostgreSQL database");
            isConnected = false;
            logOperation("PostgreSQL connection closed");
        }
    }
    
    @Override
    public boolean executeQuery(String query) {
        if (!isConnected) {
            System.out.println("Not connected to PostgreSQL database");
            return false;
        }
        
        System.out.println("Executing PostgreSQL query: " + query);
        // Simulate PostgreSQL query execution
        logOperation("PostgreSQL query executed: " + query);
        return true;
    }
    
    @Override
    public boolean executeUpdate(String updateQuery) {
        if (!isConnected) {
            System.out.println("Not connected to PostgreSQL database");
            return false;
        }
        
        System.out.println("Executing PostgreSQL update: " + updateQuery);
        // Simulate PostgreSQL update execution
        logOperation("PostgreSQL update executed: " + updateQuery);
        return true;
    }
    
    // PostgreSQL-specific method
    public void analyzePerformance() {
        if (isConnected) {
            System.out.println("Analyzing PostgreSQL performance");
        }
    }
}

// Database manager using abstraction
class DatabaseManager {
    
    // Works with any DatabaseConnection implementation
    public void performDatabaseOperations(DatabaseConnection db) {
        System.out.println("=== Database Operations ===");
        db.displayConnectionInfo();
        
        // Use abstract interface - don't care about implementation
        String[] queries = {
            "SELECT * FROM users",
            "SELECT * FROM products", 
            "SELECT * FROM orders"
        };
        
        db.performTransaction(queries);
        System.out.println();
    }
    
    public void compareConnections(DatabaseConnection[] connections) {
        System.out.println("=== Connection Comparison ===");
        
        for (DatabaseConnection db : connections) {
            System.out.println("Testing: " + db.getClass().getSimpleName());
            performDatabaseOperations(db);
        }
    }
}

// Usage demonstration
public class AbstractClassDemo {
    public static void main(String[] args) {
        DatabaseManager manager = new DatabaseManager();
        
        // Create different database connections
        DatabaseConnection mysql = new MySQLConnection("localhost", "root", "password", 3306, "myapp");
        DatabaseConnection postgres = new PostgreSQLConnection("localhost", "user", "pass", "public", true);
        
        // Use abstraction - same interface, different implementations
        manager.performDatabaseOperations(mysql);
        manager.performDatabaseOperations(postgres);
        
        // Polymorphic usage
        DatabaseConnection[] connections = {mysql, postgres};
        manager.compareConnections(connections);
        
        // Cannot instantiate abstract class
        // DatabaseConnection db = new DatabaseConnection(); // Compilation error
    }
}
```

### Example 2: Interfaces - Media Player System

```java
// Pure abstraction through interface
interface MediaPlayer {
    void play();
    void pause();
    void stop();
    void setVolume(int volume);
    int getDuration();
    boolean isPlaying();
}

// Interface for advanced features
interface AdvancedMediaPlayer extends MediaPlayer {
    void fastForward(int seconds);
    void rewind(int seconds);
    void setPlaybackSpeed(double speed);
    void addToPlaylist(String mediaFile);
}

// Interface for streaming capabilities
interface StreamingCapable {
    void streamFromURL(String url);
    void setBufferSize(int bufferSize);
    boolean isBuffering();
}

// Audio player implementation
class AudioPlayer implements MediaPlayer {
    private String currentTrack;
    private int volume;
    private int duration;
    private boolean playing;
    private int currentPosition;
    
    public AudioPlayer() {
        this.volume = 50;
        this.playing = false;
        this.currentPosition = 0;
    }
    
    @Override
    public void play() {
        if (currentTrack != null) {
            playing = true;
            System.out.println("Playing audio: " + currentTrack);
            System.out.println("♪ Audio playing... ♪");
        } else {
            System.out.println("No audio track loaded");
        }
    }
    
    @Override
    public void pause() {
        if (playing) {
            playing = false;
            System.out.println("Audio paused at position: " + currentPosition + "s");
        }
    }
    
    @Override
    public void stop() {
        playing = false;
        currentPosition = 0;
        System.out.println("Audio stopped");
    }
    
    @Override
    public void setVolume(int volume) {
        this.volume = Math.max(0, Math.min(100, volume));
        System.out.println("Audio volume set to: " + this.volume + "%");
    }
    
    @Override
    public int getDuration() {
        return duration;
    }
    
    @Override
    public boolean isPlaying() {
        return playing;
    }
    
    // Audio-specific methods
    public void loadTrack(String trackName, int duration) {
        this.currentTrack = trackName;
        this.duration = duration;
        System.out.println("Loaded audio track: " + trackName + " (" + duration + "s)");
    }
    
    public void showEqualizer() {
        System.out.println("Audio equalizer: Bass:+2, Mid:0, Treble:+1");
    }
}

// Video player with advanced features
class VideoPlayer implements AdvancedMediaPlayer, StreamingCapable {
    private String currentVideo;
    private int volume;
    private int duration;
    private boolean playing;
    private int currentPosition;
    private double playbackSpeed;
    private java.util.List<String> playlist;
    private boolean buffering;
    private int bufferSize;
    
    public VideoPlayer() {
        this.volume = 50;
        this.playing = false;
        this.currentPosition = 0;
        this.playbackSpeed = 1.0;
        this.playlist = new java.util.ArrayList<>();
        this.buffering = false;
        this.bufferSize = 1024;
    }
    
    @Override
    public void play() {
        if (currentVideo != null) {
            playing = true;
            System.out.println("Playing video: " + currentVideo);
            System.out.printf("🎬 Video playing at %.1fx speed... 🎬%n", playbackSpeed);
        } else {
            System.out.println("No video loaded");
        }
    }
    
    @Override
    public void pause() {
        if (playing) {
            playing = false;
            System.out.println("Video paused at position: " + currentPosition + "s");
        }
    }
    
    @Override
    public void stop() {
        playing = false;
        currentPosition = 0;
        System.out.println("Video stopped");
    }
    
    @Override
    public void setVolume(int volume) {
        this.volume = Math.max(0, Math.min(100, volume));
        System.out.println("Video volume set to: " + this.volume + "%");
    }
    
    @Override
    public int getDuration() {
        return duration;
    }
    
    @Override
    public boolean isPlaying() {
        return playing;
    }
    
    // Advanced features
    @Override
    public void fastForward(int seconds) {
        currentPosition += seconds;
        System.out.println("Fast forwarded " + seconds + "s. Position: " + currentPosition + "s");
    }
    
    @Override
    public void rewind(int seconds) {
        currentPosition = Math.max(0, currentPosition - seconds);
        System.out.println("Rewound " + seconds + "s. Position: " + currentPosition + "s");
    }
    
    @Override
    public void setPlaybackSpeed(double speed) {
        this.playbackSpeed = speed;
        System.out.printf("Playback speed set to %.1fx%n", speed);
    }
    
    @Override
    public void addToPlaylist(String mediaFile) {
        playlist.add(mediaFile);
        System.out.println("Added to playlist: " + mediaFile);
    }
    
    // Streaming capabilities
    @Override
    public void streamFromURL(String url) {
        buffering = true;
        System.out.println("Streaming from: " + url);
        System.out.println("Buffering... " + bufferSize + " KB buffer");
        
        // Simulate buffering
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        buffering = false;
        currentVideo = "Stream: " + url;
        System.out.println("Stream ready to play");
    }
    
    @Override
    public void setBufferSize(int bufferSize) {
        this.bufferSize = bufferSize;
        System.out.println("Buffer size set to: " + bufferSize + " KB");
    }
    
    @Override
    public boolean isBuffering() {
        return buffering;
    }
    
    // Video-specific methods
    public void loadVideo(String videoName, int duration) {
        this.currentVideo = videoName;
        this.duration = duration;
        System.out.println("Loaded video: " + videoName + " (" + duration + "s)");
    }
    
    public void showSubtitles(boolean enable) {
        System.out.println("Subtitles " + (enable ? "enabled" : "disabled"));
    }
}

// Simple media player (basic implementation)
class SimpleMediaPlayer implements MediaPlayer {
    private boolean playing = false;
    private int volume = 30;
    
    @Override
    public void play() {
        playing = true;
        System.out.println("Simple player: Playing media");
    }
    
    @Override
    public void pause() {
        playing = false;
        System.out.println("Simple player: Paused");
    }
    
    @Override
    public void stop() {
        playing = false;
        System.out.println("Simple player: Stopped");
    }
    
    @Override
    public void setVolume(int volume) {
        this.volume = volume;
        System.out.println("Simple player: Volume " + volume + "%");
    }
    
    @Override
    public int getDuration() {
        return 180; // Default 3 minutes
    }
    
    @Override
    public boolean isPlaying() {
        return playing;
    }
}

// Media controller using interface abstraction
class MediaController {
    
    // Works with any MediaPlayer implementation
    public void controlMedia(MediaPlayer player) {
        System.out.println("=== Media Control Session ===");
        System.out.println("Player type: " + player.getClass().getSimpleName());
        
        player.setVolume(75);
        player.play();
        
        if (player.isPlaying()) {
            System.out.println("Status: Playing, Duration: " + player.getDuration() + "s");
        }
        
        player.pause();
        player.stop();
        System.out.println();
    }
    
    // Advanced control for capable players
    public void advancedControl(MediaPlayer player) {
        if (player instanceof AdvancedMediaPlayer) {
            AdvancedMediaPlayer advanced = (AdvancedMediaPlayer) player;
            System.out.println("=== Advanced Controls ===");
            
            advanced.play();
            advanced.fastForward(30);
            advanced.setPlaybackSpeed(1.5);
            advanced.addToPlaylist("movie2.mp4");
        }
        
        if (player instanceof StreamingCapable) {
            StreamingCapable streaming = (StreamingCapable) player;
            System.out.println("=== Streaming Features ===");
            
            streaming.setBufferSize(2048);
            streaming.streamFromURL("https://example.com/stream");
        }
    }
    
    // Demonstrate polymorphism with interfaces
    public void batchControl(MediaPlayer[] players) {
        System.out.println("=== Batch Control ===");
        
        for (MediaPlayer player : players) {
            controlMedia(player); // Polymorphic call
        }
    }
}

// Usage demonstration
public class InterfaceDemo {
    public static void main(String[] args) {
        MediaController controller = new MediaController();
        
        // Create different media players
        AudioPlayer audioPlayer = new AudioPlayer();
        audioPlayer.loadTrack("song.mp3", 240);
        
        VideoPlayer videoPlayer = new VideoPlayer();
        videoPlayer.loadVideo("movie.mp4", 7200);
        
        SimpleMediaPlayer simplePlayer = new SimpleMediaPlayer();
        
        // Interface abstraction - treat all as MediaPlayer
        MediaPlayer[] players = {audioPlayer, videoPlayer, simplePlayer};
        
        // Polymorphic usage
        controller.batchControl(players);
        
        // Advanced features
        controller.advancedControl(videoPlayer);
        
        // Interface segregation - use specific interfaces
        System.out.println("=== Specific Interface Usage ===");
        if (videoPlayer instanceof StreamingCapable) {
            StreamingCapable streamer = videoPlayer;
            streamer.streamFromURL("https://netflix.com/movie");
        }
        
        // Cannot instantiate interface
        // MediaPlayer player = new MediaPlayer(); // Compilation error
    }
}
```

### Example 3: Abstract Classes vs Interfaces - Banking System

```java
// Abstract class for common banking operations
abstract class BankAccount {
    protected String accountNumber;
    protected String accountHolder;
    protected double balance;
    protected java.util.List<String> transactionHistory;
    
    // Constructor
    public BankAccount(String accountNumber, String accountHolder, double initialBalance) {
        this.accountNumber = accountNumber;
        this.accountHolder = accountHolder;
        this.balance = initialBalance;
        this.transactionHistory = new java.util.ArrayList<>();
        addTransaction("Account opened with balance: $" + initialBalance);
    }
    
    // Abstract methods - must be implemented
    public abstract boolean withdraw(double amount);
    public abstract double calculateInterest();
    public abstract String getAccountType();
    
    // Concrete methods - common implementation
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            addTransaction("Deposit: $" + amount);
            System.out.printf("Deposited $%.2f. New balance: $%.2f%n", amount, balance);
        } else {
            System.out.println("Deposit amount must be positive");
        }
    }
    
    public double getBalance() {
        return balance;
    }
    
    public void displayAccountInfo() {
        System.out.printf("Account: %s (%s)%n", accountNumber, getAccountType());
        System.out.printf("Holder: %s%n", accountHolder);
        System.out.printf("Balance: $%.2f%n", balance);
    }
    
    protected void addTransaction(String transaction) {
        String timestamp = new java.util.Date().toString();
        transactionHistory.add(timestamp + " - " + transaction);
    }
    
    public void showTransactionHistory() {
        System.out.println("Transaction History:");
        for (String transaction : transactionHistory) {
            System.out.println("  " + transaction);
        }
    }
    
    // Template method
    public final void monthlyProcessing() {
        System.out.println("=== Monthly Processing for " + accountNumber + " ===");
        double interest = calculateInterest();
        if (interest > 0) {
            balance += interest;
            addTransaction("Interest credited: $" + String.format("%.2f", interest));
            System.out.printf("Interest credited: $%.2f%n", interest);
        }
        generateStatement();
    }
    
    private void generateStatement() {
        System.out.println("Monthly statement generated");
    }
}

// Interfaces for additional capabilities
interface Transferable {
    boolean transfer(BankAccount targetAccount, double amount);
    boolean scheduleTransfer(BankAccount targetAccount, double amount, java.util.Date date);
}

interface CreditCapable {
    double getCreditLimit();
    double getAvailableCredit();
    boolean setCreditLimit(double limit);
}

interface InvestmentCapable {
    boolean buyStock(String symbol, int shares);
    boolean sellStock(String symbol, int shares);
    double getPortfolioValue();
}

// Savings Account implementation
class SavingsAccount extends BankAccount implements Transferable {
    private double interestRate;
    private int withdrawalCount;
    private static final int MAX_WITHDRAWALS = 6;
    
    public SavingsAccount(String accountNumber, String accountHolder, 
                         double initialBalance, double interestRate) {
        super(accountNumber, accountHolder, initialBalance);
        this.interestRate = interestRate;
        this.withdrawalCount = 0;
    }
    
    @Override
    public boolean withdraw(double amount) {
        if (withdrawalCount >= MAX_WITHDRAWALS) {
            System.out.println("Maximum monthly withdrawals exceeded");
            return false;
        }
        
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            withdrawalCount++;
            addTransaction("Withdrawal: $" + amount);
            System.out.printf("Withdrawn $%.2f. New balance: $%.2f%n", amount, balance);
            return true;
        } else {
            System.out.println("Invalid withdrawal amount or insufficient funds");
            return false;
        }
    }
    
    @Override
    public double calculateInterest() {
        return balance * (interestRate / 100) / 12; // Monthly interest
    }
    
    @Override
    public String getAccountType() {
        return "Savings Account";
    }
    
    @Override
    public boolean transfer(BankAccount targetAccount, double amount) {
        if (withdraw(amount)) {
            targetAccount.deposit(amount);
            addTransaction("Transfer to " + targetAccount.accountNumber + ": $" + amount);
            return true;
        }
        return false;
    }
    
    @Override
    public boolean scheduleTransfer(BankAccount targetAccount, double amount, java.util.Date date) {
        System.out.printf("Transfer of $%.2f to %s scheduled for %s%n", 
                amount, targetAccount.accountNumber, date);
        return true;
    }
    
    public void resetWithdrawalCount() {
        withdrawalCount = 0;
        System.out.println("Monthly withdrawal count reset");
    }
}

// Checking Account implementation
class CheckingAccount extends BankAccount implements Transferable {
    private double overdraftLimit;
    
    public CheckingAccount(String accountNumber, String accountHolder, 
                          double initialBalance, double overdraftLimit) {
        super(accountNumber, accountHolder, initialBalance);
        this.overdraftLimit = overdraftLimit;
    }
    
    @Override
    public boolean withdraw(double amount) {
        double availableBalance = balance + overdraftLimit;
        
        if (amount > 0 && amount <= availableBalance) {
            balance -= amount;
            addTransaction("Withdrawal: $" + amount);
            
            if (balance < 0) {
                System.out.printf("Withdrawn $%.2f using overdraft. Balance: $%.2f%n", 
                        amount, balance);
            } else {
                System.out.printf("Withdrawn $%.2f. Balance: $%.2f%n", amount, balance);
            }
            return true;
        } else {
            System.out.println("Withdrawal exceeds available balance and overdraft limit");
            return false;
        }
    }
    
    @Override
    public double calculateInterest() {
        return 0; // No interest on checking accounts
    }
    
    @Override
    public String getAccountType() {
        return "Checking Account";
    }
    
    @Override
    public boolean transfer(BankAccount targetAccount, double amount) {
        if (withdraw(amount)) {
            targetAccount.deposit(amount);
            addTransaction("Transfer to " + targetAccount.accountNumber + ": $" + amount);
            return true;
        }
        return false;
    }
    
    @Override
    public boolean scheduleTransfer(BankAccount targetAccount, double amount, java.util.Date date) {
        System.out.printf("Transfer of $%.2f to %s scheduled for %s%n", 
                amount, targetAccount.accountNumber, date);
        return true;
    }
    
    public double getOverdraftLimit() {
        return overdraftLimit;
    }
}

// Credit Card Account implementation
class CreditCardAccount extends BankAccount implements CreditCapable {
    private double creditLimit;
    private double interestRate;
    
    public CreditCardAccount(String accountNumber, String accountHolder, double creditLimit, double interestRate) {
        super(accountNumber, accountHolder, 0); // Credit cards start with 0 balance
        this.creditLimit = creditLimit;
        this.interestRate = interestRate;
    }
    
    @Override
    public boolean withdraw(double amount) {
        // For credit cards, withdrawal is like a cash advance
        if (amount > 0 && (balance + amount) <= creditLimit) {
            balance += amount; // Increases debt
            addTransaction("Cash advance: $" + amount);
            System.out.printf("Cash advance $%.2f. Current balance: $%.2f%n", amount, balance);
            return true;
        } else {
            System.out.println("Cash advance exceeds credit limit");
            return false;
        }
    }
    
    @Override
    public double calculateInterest() {
        if (balance > 0) {
            return balance * (interestRate / 100) / 12; // Monthly interest on debt
        }
        return 0;
    }
    
    @Override
    public String getAccountType() {
        return "Credit Card Account";
    }
    
    @Override
    public double getCreditLimit() {
        return creditLimit;
    }
    
    @Override
    public double getAvailableCredit() {
        return creditLimit - balance;
    }
    
    @Override
    public boolean setCreditLimit(double limit) {
        if (limit >= balance) {
            this.creditLimit = limit;
            System.out.printf("Credit limit updated to $%.2f%n", limit);
            return true;
        } else {
            System.out.println("Credit limit cannot be less than current balance");
            return false;
        }
    }
    
    // Credit card specific method
    public boolean makePurchase(double amount) {
        if (amount > 0 && (balance + amount) <= creditLimit) {
            balance += amount;
            addTransaction("Purchase: $" + amount);
            System.out.printf("Purchase $%.2f charged. Balance: $%.2f%n", amount, balance);
            return true;
        } else {
            System.out.println("Purchase exceeds available credit");
            return false;
        }
    }
}

// Bank service using abstraction
class BankService {
    
    // Works with any BankAccount (abstraction)
    public void processAccount(BankAccount account) {
        System.out.println("=== Processing Account ===");
        account.displayAccountInfo();
        account.monthlyProcessing();
        System.out.println();
    }
    
    // Transfer service using interface
    public void performTransfer(Transferable fromAccount, BankAccount toAccount, double amount) {
        System.out.println("=== Transfer Service ===");
        if (fromAccount.transfer(toAccount, amount)) {
            System.out.println("Transfer completed successfully");
        } else {
            System.out.println("Transfer failed");
        }
    }
    
    // Credit management using interface
    public void manageCreditAccount(CreditCapable creditAccount) {
        System.out.println("=== Credit Management ===");
        System.out.printf("Credit Limit: $%.2f%n", creditAccount.getCreditLimit());
        System.out.printf("Available Credit: $%.2f%n", creditAccount.getAvailableCredit());
    }
}

// Usage demonstration
public class AbstractionDemo {
    public static void main(String[] args) {
        BankService service = new BankService();
        
        // Create different account types
        SavingsAccount savings = new SavingsAccount("SAV001", "Alice Johnson", 5000, 2.5);
        CheckingAccount checking = new CheckingAccount("CHK001", "Bob Smith", 1000, 500);
        CreditCardAccount creditCard = new CreditCardAccount("CC001", "Carol Davis", 2000, 18.0);
        
        // Abstraction - treat all as BankAccount
        BankAccount[] accounts = {savings, checking, creditCard};
        
        for (BankAccount account : accounts) {
            service.processAccount(account); // Polymorphic call
        }
        
        // Interface-based operations
        System.out.println("=== Transfer Operations ===");
        service.performTransfer(savings, checking, 500);
        service.performTransfer(checking, savings, 200);
        
        // Credit management
        service.manageCreditAccount(creditCard);
        
        // Specific operations based on capabilities
        if (creditCard instanceof CreditCardAccount) {
            CreditCardAccount cc = (CreditCardAccount) creditCard;
            cc.makePurchase(150);
        }
        
        // Cannot instantiate abstract class
        // BankAccount account = new BankAccount(); // Compilation error
    }
}
```

## ⚠️ Common Mistakes to Avoid

### 1. Confusing Abstract Classes with Interfaces
```java
// ❌ Wrong usage
interface WrongInterface {
    private int value; // ❌ Cannot have instance variables
    public WrongInterface() { } // ❌ Cannot have constructors
}

abstract class WrongAbstractClass implements SomeInterface {
    // ❌ Trying to implement interface method as abstract again
    public abstract void interfaceMethod(); // Should provide implementation
}

// ✅ Correct usage
interface CorrectInterface {
    int VALUE = 10; // ✅ public static final by default
    void method();  // ✅ abstract by default
}

abstract class CorrectAbstractClass {
    protected int value; // ✅ Can have instance variables
    
    public CorrectAbstractClass(int value) { // ✅ Can have constructor
        this.value = value;
    }
    
    public abstract void abstractMethod(); // ✅ Abstract method
    public void concreteMethod() { } // ✅ Concrete method
}
```

### 2. Not Following Interface Naming Conventions
```java
// ❌ Poor naming
interface DatabaseClass { } // Should not end with "Class"
interface IDatabase { } // Hungarian notation not preferred in Java

// ✅ Good naming
interface Database { } // Clear, concise
interface Readable { } // Adjective form
interface PaymentProcessor { } // Descriptive of capability
```

### 3. Over-abstracting or Under-abstracting
```java
// ❌ Over-abstraction - too many small interfaces
interface Walkable {
    void walk();
}
interface Runnable {
    void run();
}
interface Jumpable {
    void jump();
}

// ✅ Better grouping
interface Movable {
    void walk();
    void run();
    void jump();
}

// ❌ Under-abstraction - concrete class doing too much
class VideoPlayer {
    public void playMP4() { } // Too specific
    public void playAVI() { } // Too specific
    public void playMOV() { } // Too specific
}

// ✅ Better abstraction
interface MediaPlayer {
    void play(String mediaFile); // Generic method
}
```

## 🎯 Benefits of Abstraction

### 1. **Complexity Hiding**
```java
// User doesn't need to know implementation details
MediaPlayer player = new VideoPlayer();
player.play(); // Simple interface, complex implementation hidden
```

### 2. **Code Flexibility**
```java
// Can switch implementations without changing client code
DatabaseConnection db = new MySQLConnection(); // Can change to PostgreSQL
db.connect(); // Same interface
```

### 3. **Maintainability**
```java
// Changes to implementation don't affect clients
abstract class Shape {
    public abstract void draw(); // Contract remains same
}
// Implementation can change without affecting users
```

## 🔥 Interview Questions & Answers

### Q1: What is abstraction and how is it achieved in Java?
**Answer**: Abstraction is hiding implementation details while showing only essential features. In Java, it's achieved through:
- **Abstract Classes**: 0-100% abstraction, can have abstract and concrete methods
- **Interfaces**: 100% abstraction (before Java 8), define contracts
- **Access Modifiers**: Control visibility of internal implementation
- **Encapsulation**: Hide internal data and provide controlled access

### Q2: What's the difference between abstract classes and interfaces?
**Answer**: 
| Aspect | Abstract Class | Interface |
|--------|----------------|-----------|
| **Implementation** | Can have abstract and concrete methods | All methods abstract (before Java 8) |
| **Variables** | Can have instance variables | Only public static final variables |
| **Constructor** | Can have constructors | Cannot have constructors |
| **Inheritance** | Single inheritance (extends) | Multiple inheritance (implements) |
| **Access Modifiers** | Any access modifier | Public by default |
| **Use Case** | When classes share common implementation | When classes share common contract |

### Q3: When should you use abstract classes vs interfaces?
**Answer**: 
**Use Abstract Classes when**:
- Classes share common implementation code
- You want to provide default behavior
- You need to declare non-public members
- Classes have IS-A relationship with common base

**Use Interfaces when**:
- You want to specify contract without implementation
- You need multiple inheritance
- Unrelated classes should implement same methods
- You want to achieve loose coupling

### Q4: Can an abstract class have constructors? Why?
**Answer**: **Yes**, abstract classes can have constructors because:
- **Subclasses need to initialize parent state** when they're instantiated
- **Constructor chaining** - child constructors call super()
- **Common initialization logic** can be placed in abstract class constructor
- **Abstract class cannot be instantiated directly**, but constructor runs when subclass is created

### Q5: What are the Java 8+ interface features and how do they affect abstraction?
**Answer**: Java 8+ added:
- **Default Methods**: Provide implementation in interface (`default` keyword)
- **Static Methods**: Interface can have static methods with implementation
- **Private Methods** (Java 9+): Helper methods within interface

**Impact on Abstraction**:
- **Reduced need for abstract classes** in some cases
- **Multiple inheritance of behavior** becomes possible
- **Interface evolution** without breaking existing implementations
- **Still maintains abstraction** - clients use interface contract, not implementation details

---
[← Back to Main Guide](./README.md) | [← Previous: Polymorphism](./polymorphism.md) | [Next: JDK vs JRE vs JVM →](./jdk-jre-jvm.md)
