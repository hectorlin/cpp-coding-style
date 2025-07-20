# Top 10 Linux C++11 Clean Architecture Principles

## 1. Dependency Inversion with Interfaces
**Description:** High-level modules should not depend on low-level modules. Both should depend on abstractions. In Linux C++ development, this means using abstract base classes and interfaces to decouple components.

**Purpose:** Introduced to create loosely coupled, testable, and maintainable systems that can easily adapt to changing requirements and different Linux environments.

**Before (Traditional Approach):**
```cpp
// High-level module directly depends on low-level module
class FileProcessor {
private:
    LinuxFileSystem fileSystem; // Direct dependency
public:
    void processFile(const std::string& filename) {
        if (fileSystem.exists(filename)) {
            auto data = fileSystem.readFile(filename);
            // Process data...
        }
    }
};

class LinuxFileSystem {
public:
    bool exists(const std::string& filename) {
        return access(filename.c_str(), F_OK) == 0;
    }
    
    std::string readFile(const std::string& filename) {
        std::ifstream file(filename);
        return std::string(std::istreambuf_iterator<char>(file),
                          std::istreambuf_iterator<char>());
    }
};
```

**After (Clean Architecture):**
```cpp
// Abstract interface
class IFileSystem {
public:
    virtual ~IFileSystem() = default;
    virtual bool exists(const std::string& filename) const = 0;
    virtual std::string readFile(const std::string& filename) const = 0;
};

// High-level module depends on abstraction
class FileProcessor {
private:
    std::shared_ptr<IFileSystem> fileSystem; // Dependency injection
public:
    explicit FileProcessor(std::shared_ptr<IFileSystem> fs) 
        : fileSystem(std::move(fs)) {}
    
    void processFile(const std::string& filename) {
        if (fileSystem->exists(filename)) {
            auto data = fileSystem->readFile(filename);
            // Process data...
        }
    }
};

// Low-level implementation
class LinuxFileSystem : public IFileSystem {
public:
    bool exists(const std::string& filename) const override {
        return access(filename.c_str(), F_OK) == 0;
    }
    
    std::string readFile(const std::string& filename) const override {
        std::ifstream file(filename);
        return std::string(std::istreambuf_iterator<char>(file),
                          std::istreambuf_iterator<char>());
    }
};
```

**Usage:**
```cpp
// Dependency injection
auto fileSystem = std::make_shared<LinuxFileSystem>();
FileProcessor processor(fileSystem);

// Easy to test with mock
class MockFileSystem : public IFileSystem {
    // Mock implementation for testing
};

auto mockFS = std::make_shared<MockFileSystem>();
FileProcessor testProcessor(mockFS);
```

## 2. Repository Pattern for Data Access
**Description:** Encapsulates data access logic behind an interface, providing a clean separation between business logic and data persistence concerns.

**Purpose:** Introduced to abstract data storage details, enable easy testing with mocks, and support multiple storage backends (files, databases, network).

**Before (Direct Data Access):**
```cpp
class UserManager {
public:
    void saveUser(const User& user) {
        // Direct file I/O mixed with business logic
        std::ofstream file("/var/lib/app/users.txt", std::ios::app);
        file << user.getId() << "," << user.getName() << "," 
             << user.getEmail() << std::endl;
    }
    
    User getUser(int id) {
        std::ifstream file("/var/lib/app/users.txt");
        std::string line;
        while (std::getline(file, line)) {
            // Parse line and find user...
        }
        return User{}; // Not found
    }
    
    void deleteUser(int id) {
        // Complex file manipulation logic
        // Read all lines, filter out the one to delete, rewrite file
    }
};
```

**After (Repository Pattern):**
```cpp
// Repository interface
class IUserRepository {
public:
    virtual ~IUserRepository() = default;
    virtual void save(const User& user) = 0;
    virtual std::optional<User> findById(int id) = 0;
    virtual void remove(int id) = 0;
    virtual std::vector<User> findAll() = 0;
};

// Business logic depends on repository interface
class UserManager {
private:
    std::shared_ptr<IUserRepository> repository;
public:
    explicit UserManager(std::shared_ptr<IUserRepository> repo) 
        : repository(std::move(repo)) {}
    
    void createUser(const User& user) {
        // Business logic validation
        if (user.getName().empty()) {
            throw std::invalid_argument("User name cannot be empty");
        }
        repository->save(user);
    }
    
    User getUser(int id) {
        auto user = repository->findById(id);
        if (!user) {
            throw std::runtime_error("User not found");
        }
        return *user;
    }
};

// File-based implementation
class FileUserRepository : public IUserRepository {
private:
    std::string filePath;
public:
    explicit FileUserRepository(const std::string& path) : filePath(path) {}
    
    void save(const User& user) override {
        std::ofstream file(filePath, std::ios::app);
        file << user.getId() << "," << user.getName() << "," 
             << user.getEmail() << std::endl;
    }
    
    std::optional<User> findById(int id) override {
        std::ifstream file(filePath);
        std::string line;
        while (std::getline(file, line)) {
            // Parse and find user...
        }
        return std::nullopt;
    }
    
    void remove(int id) override {
        // Implementation for deletion
    }
    
    std::vector<User> findAll() override {
        // Implementation for getting all users
        return {};
    }
};
```

**Usage:**
```cpp
// Different storage implementations
auto fileRepo = std::make_shared<FileUserRepository>("/var/lib/app/users.txt");
auto dbRepo = std::make_shared<DatabaseUserRepository>(connectionString);
auto memoryRepo = std::make_shared<InMemoryUserRepository>();

UserManager manager(fileRepo); // Easy to switch implementations
```

## 3. Use Case Pattern for Business Logic
**Description:** Encapsulates specific business operations in dedicated classes, keeping business rules separate from infrastructure concerns.

**Purpose:** Introduced to organize business logic into focused, testable units that are independent of external dependencies and easy to understand.

**Before (Mixed Concerns):**
```cpp
class UserService {
private:
    std::ofstream logFile;
    std::string dbConnection;
public:
    void registerUser(const std::string& name, const std::string& email) {
        // Mixed business logic, logging, and data access
        if (name.empty()) {
            logFile << "Error: Empty name provided" << std::endl;
            return;
        }
        
        // Database connection and query
        sqlite3* db;
        sqlite3_open(dbConnection.c_str(), &db);
        // Complex SQL logic...
        
        logFile << "User registered: " << name << std::endl;
        sqlite3_close(db);
    }
};
```

**After (Use Case Pattern):**
```cpp
// Use case interface
class IRegisterUserUseCase {
public:
    virtual ~IRegisterUserUseCase() = default;
    virtual void execute(const std::string& name, const std::string& email) = 0;
};

// Pure business logic
class RegisterUserUseCase : public IRegisterUserUseCase {
private:
    std::shared_ptr<IUserRepository> userRepository;
    std::shared_ptr<ILogger> logger;
    std::shared_ptr<IEmailValidator> emailValidator;
public:
    RegisterUserUseCase(std::shared_ptr<IUserRepository> repo,
                       std::shared_ptr<ILogger> log,
                       std::shared_ptr<IEmailValidator> validator)
        : userRepository(std::move(repo))
        , logger(std::move(log))
        , emailValidator(std::move(validator)) {}
    
    void execute(const std::string& name, const std::string& email) override {
        // Business rules
        if (name.empty()) {
            throw std::invalid_argument("Name cannot be empty");
        }
        
        if (!emailValidator->isValid(email)) {
            throw std::invalid_argument("Invalid email format");
        }
        
        // Create user
        User user{name, email};
        userRepository->save(user);
        
        logger->info("User registered: " + name);
    }
};
```

**Usage:**
```cpp
// Dependency injection
auto repo = std::make_shared<FileUserRepository>("/var/lib/users.txt");
auto logger = std::make_shared<FileLogger>("/var/log/app.log");
auto validator = std::make_shared<EmailValidator>();

auto useCase = std::make_unique<RegisterUserUseCase>(repo, logger, validator);
useCase->execute("John Doe", "john@example.com");
```

## 4. Observer Pattern for Event Handling
**Description:** Defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

**Purpose:** Introduced to create loosely coupled event-driven systems, especially useful for Linux system monitoring, logging, and real-time updates.

**Before (Tight Coupling):**
```cpp
class SystemMonitor {
private:
    LogManager logManager;
    AlertManager alertManager;
    MetricsCollector metricsCollector;
public:
    void onSystemEvent(const SystemEvent& event) {
        // Direct calls to all components
        logManager.logEvent(event);
        alertManager.checkAlerts(event);
        metricsCollector.recordMetric(event);
        
        // Adding new component requires modifying this class
    }
};
```

**After (Observer Pattern):**
```cpp
// Observer interface
class ISystemEventObserver {
public:
    virtual ~ISystemEventObserver() = default;
    virtual void onSystemEvent(const SystemEvent& event) = 0;
};

// Subject interface
class ISystemEventSubject {
public:
    virtual ~ISystemEventSubject() = default;
    virtual void attach(std::shared_ptr<ISystemEventObserver> observer) = 0;
    virtual void detach(std::shared_ptr<ISystemEventObserver> observer) = 0;
    virtual void notify(const SystemEvent& event) = 0;
};

// Concrete subject
class SystemMonitor : public ISystemEventSubject {
private:
    std::vector<std::shared_ptr<ISystemEventObserver>> observers;
    std::mutex observersMutex;
public:
    void attach(std::shared_ptr<ISystemEventObserver> observer) override {
        std::lock_guard<std::mutex> lock(observersMutex);
        observers.push_back(observer);
    }
    
    void detach(std::shared_ptr<ISystemEventObserver> observer) override {
        std::lock_guard<std::mutex> lock(observersMutex);
        observers.erase(
            std::remove(observers.begin(), observers.end(), observer),
            observers.end()
        );
    }
    
    void notify(const SystemEvent& event) override {
        std::lock_guard<std::mutex> lock(observersMutex);
        for (auto& observer : observers) {
            observer->onSystemEvent(event);
        }
    }
    
    void processSystemEvent(const SystemEvent& event) {
        // Process event and notify all observers
        notify(event);
    }
};

// Concrete observers
class LogManager : public ISystemEventObserver {
public:
    void onSystemEvent(const SystemEvent& event) override {
        // Log the event
        std::cout << "Logging: " << event.getDescription() << std::endl;
    }
};

class AlertManager : public ISystemEventObserver {
public:
    void onSystemEvent(const SystemEvent& event) override {
        // Check for alerts
        if (event.getSeverity() > 5) {
            std::cout << "ALERT: " << event.getDescription() << std::endl;
        }
    }
};
```

**Usage:**
```cpp
auto monitor = std::make_shared<SystemMonitor>();
auto logger = std::make_shared<LogManager>();
auto alerter = std::make_shared<AlertManager>();

monitor->attach(logger);
monitor->attach(alerter);

// Easy to add new observers without modifying existing code
auto metrics = std::make_shared<MetricsCollector>();
monitor->attach(metrics);
```

## 5. Factory Pattern for Object Creation
**Description:** Encapsulates object creation logic, allowing the system to create objects without specifying their exact classes.

**Purpose:** Introduced to centralize object creation, support different creation strategies, and enable easy configuration changes in Linux environments.

**Before (Direct Instantiation):**
```cpp
class NetworkManager {
public:
    void createConnection(const std::string& type) {
        if (type == "tcp") {
            auto connection = std::make_unique<TCPConnection>();
            // Use connection...
        } else if (type == "udp") {
            auto connection = std::make_unique<UDPConnection>();
            // Use connection...
        } else if (type == "ssl") {
            auto connection = std::make_unique<SSLConnection>();
            // Use connection...
        }
        // Adding new types requires modifying this method
    }
};
```

**After (Factory Pattern):**
```cpp
// Abstract product
class INetworkConnection {
public:
    virtual ~INetworkConnection() = default;
    virtual void connect() = 0;
    virtual void disconnect() = 0;
    virtual void send(const std::string& data) = 0;
};

// Concrete products
class TCPConnection : public INetworkConnection {
public:
    void connect() override { /* TCP connection logic */ }
    void disconnect() override { /* TCP disconnect logic */ }
    void send(const std::string& data) override { /* TCP send logic */ }
};

class UDPConnection : public INetworkConnection {
public:
    void connect() override { /* UDP connection logic */ }
    void disconnect() override { /* UDP disconnect logic */ }
    void send(const std::string& data) override { /* UDP send logic */ }
};

// Factory interface
class INetworkConnectionFactory {
public:
    virtual ~INetworkConnectionFactory() = default;
    virtual std::unique_ptr<INetworkConnection> createConnection() = 0;
};

// Concrete factories
class TCPConnectionFactory : public INetworkConnectionFactory {
public:
    std::unique_ptr<INetworkConnection> createConnection() override {
        return std::make_unique<TCPConnection>();
    }
};

class UDPConnectionFactory : public INetworkConnectionFactory {
public:
    std::unique_ptr<INetworkConnection> createConnection() override {
        return std::make_unique<UDPConnection>();
    }
};

// Factory registry
class NetworkConnectionFactoryRegistry {
private:
    std::map<std::string, std::shared_ptr<INetworkConnectionFactory>> factories;
public:
    void registerFactory(const std::string& type, 
                        std::shared_ptr<INetworkConnectionFactory> factory) {
        factories[type] = factory;
    }
    
    std::unique_ptr<INetworkConnection> createConnection(const std::string& type) {
        auto it = factories.find(type);
        if (it == factories.end()) {
            throw std::invalid_argument("Unknown connection type: " + type);
        }
        return it->second->createConnection();
    }
};
```

**Usage:**
```cpp
NetworkConnectionFactoryRegistry registry;
registry.registerFactory("tcp", std::make_shared<TCPConnectionFactory>());
registry.registerFactory("udp", std::make_shared<UDPConnectionFactory>());

auto connection = registry.createConnection("tcp");
connection->connect();
```

## 6. Strategy Pattern for Algorithms
**Description:** Defines a family of algorithms, encapsulates each one, and makes them interchangeable at runtime.

**Purpose:** Introduced to enable runtime algorithm selection, support different Linux system configurations, and make systems more flexible and testable.

**Before (Hard-coded Algorithms):**
```cpp
class FileCompressor {
public:
    void compressFile(const std::string& filename, const std::string& algorithm) {
        if (algorithm == "gzip") {
            // Gzip compression logic
            system("gzip " + filename);
        } else if (algorithm == "bzip2") {
            // Bzip2 compression logic
            system("bzip2 " + filename);
        } else if (algorithm == "xz") {
            // XZ compression logic
            system("xz " + filename);
        }
        // Adding new algorithms requires modifying this class
    }
};
```

**After (Strategy Pattern):**
```cpp
// Strategy interface
class ICompressionStrategy {
public:
    virtual ~ICompressionStrategy() = default;
    virtual void compress(const std::string& filename) = 0;
    virtual void decompress(const std::string& filename) = 0;
};

// Concrete strategies
class GzipCompressionStrategy : public ICompressionStrategy {
public:
    void compress(const std::string& filename) override {
        std::string command = "gzip " + filename;
        system(command.c_str());
    }
    
    void decompress(const std::string& filename) override {
        std::string command = "gunzip " + filename;
        system(command.c_str());
    }
};

class Bzip2CompressionStrategy : public ICompressionStrategy {
public:
    void compress(const std::string& filename) override {
        std::string command = "bzip2 " + filename;
        system(command.c_str());
    }
    
    void decompress(const std::string& filename) override {
        std::string command = "bunzip2 " + filename;
        system(command.c_str());
    }
};

// Context class
class FileCompressor {
private:
    std::shared_ptr<ICompressionStrategy> strategy;
public:
    explicit FileCompressor(std::shared_ptr<ICompressionStrategy> strat) 
        : strategy(std::move(strat)) {}
    
    void setStrategy(std::shared_ptr<ICompressionStrategy> strat) {
        strategy = std::move(strat);
    }
    
    void compressFile(const std::string& filename) {
        strategy->compress(filename);
    }
    
    void decompressFile(const std::string& filename) {
        strategy->decompress(filename);
    }
};
```

**Usage:**
```cpp
auto gzipStrategy = std::make_shared<GzipCompressionStrategy>();
auto bzip2Strategy = std::make_shared<Bzip2CompressionStrategy>();

FileCompressor compressor(gzipStrategy);
compressor.compressFile("data.txt");

// Switch strategy at runtime
compressor.setStrategy(bzip2Strategy);
compressor.compressFile("backup.txt");
```

## 7. Command Pattern for Operations
**Description:** Encapsulates a request as an object, allowing parameterization of clients with different requests, queuing of operations, and logging of requests.

**Purpose:** Introduced to support undo/redo operations, command queuing, and remote execution in Linux system administration tools.

**Before (Direct Method Calls):**
```cpp
class SystemAdministrator {
public:
    void createUser(const std::string& username) {
        std::string command = "useradd " + username;
        system(command.c_str());
        // No way to undo or log the operation
    }
    
    void deleteUser(const std::string& username) {
        std::string command = "userdel " + username;
        system(command.c_str());
        // No way to undo or log the operation
    }
    
    void createDirectory(const std::string& path) {
        std::string command = "mkdir -p " + path;
        system(command.c_str());
        // No way to undo or log the operation
    }
};
```

**After (Command Pattern):**
```cpp
// Command interface
class ICommand {
public:
    virtual ~ICommand() = default;
    virtual void execute() = 0;
    virtual void undo() = 0;
    virtual std::string getDescription() const = 0;
};

// Concrete commands
class CreateUserCommand : public ICommand {
private:
    std::string username;
    std::shared_ptr<ILogger> logger;
public:
    CreateUserCommand(std::string user, std::shared_ptr<ILogger> log)
        : username(std::move(user)), logger(std::move(log)) {}
    
    void execute() override {
        std::string command = "useradd " + username;
        int result = system(command.c_str());
        if (result == 0) {
            logger->info("User created: " + username);
        } else {
            logger->error("Failed to create user: " + username);
        }
    }
    
    void undo() override {
        std::string command = "userdel " + username;
        system(command.c_str());
        logger->info("User deleted (undo): " + username);
    }
    
    std::string getDescription() const override {
        return "Create user: " + username;
    }
};

class CreateDirectoryCommand : public ICommand {
private:
    std::string path;
    std::shared_ptr<ILogger> logger;
public:
    CreateDirectoryCommand(std::string dirPath, std::shared_ptr<ILogger> log)
        : path(std::move(dirPath)), logger(std::move(log)) {}
    
    void execute() override {
        std::string command = "mkdir -p " + path;
        int result = system(command.c_str());
        if (result == 0) {
            logger->info("Directory created: " + path);
        } else {
            logger->error("Failed to create directory: " + path);
        }
    }
    
    void undo() override {
        std::string command = "rmdir " + path;
        system(command.c_str());
        logger->info("Directory removed (undo): " + path);
    }
    
    std::string getDescription() const override {
        return "Create directory: " + path;
    }
};

// Command invoker
class CommandInvoker {
private:
    std::vector<std::shared_ptr<ICommand>> commandHistory;
    std::shared_ptr<ILogger> logger;
public:
    explicit CommandInvoker(std::shared_ptr<ILogger> log) : logger(std::move(log)) {}
    
    void executeCommand(std::shared_ptr<ICommand> command) {
        command->execute();
        commandHistory.push_back(command);
        logger->info("Command executed: " + command->getDescription());
    }
    
    void undoLastCommand() {
        if (!commandHistory.empty()) {
            auto command = commandHistory.back();
            command->undo();
            commandHistory.pop_back();
            logger->info("Command undone: " + command->getDescription());
        }
    }
};
```

**Usage:**
```cpp
auto logger = std::make_shared<FileLogger>("/var/log/admin.log");
CommandInvoker invoker(logger);

auto createUserCmd = std::make_shared<CreateUserCommand>("john", logger);
auto createDirCmd = std::make_shared<CreateDirectoryCommand>("/home/john", logger);

invoker.executeCommand(createUserCmd);
invoker.executeCommand(createDirCmd);

// Undo last operation
invoker.undoLastCommand();
```

## 8. Adapter Pattern for System Integration
**Description:** Allows incompatible interfaces to work together by wrapping an existing class with a new interface.

**Purpose:** Introduced to integrate third-party Linux libraries, legacy systems, and different API versions without modifying existing code.

**Before (Direct Integration):**
```cpp
// Third-party library with incompatible interface
class LegacyLogger {
public:
    void logMessage(int level, const char* message) {
        // Legacy logging implementation
    }
};

// Application code must adapt to legacy interface
class Application {
private:
    LegacyLogger logger;
public:
    void processData() {
        // Must use legacy interface
        logger.logMessage(1, "Processing started");
        // Process data...
        logger.logMessage(0, "Processing completed");
    }
};
```

**After (Adapter Pattern):**
```cpp
// Modern logging interface
class ILogger {
public:
    virtual ~ILogger() = default;
    virtual void info(const std::string& message) = 0;
    virtual void error(const std::string& message) = 0;
    virtual void debug(const std::string& message) = 0;
};

// Adapter for legacy logger
class LegacyLoggerAdapter : public ILogger {
private:
    LegacyLogger legacyLogger;
public:
    void info(const std::string& message) override {
        legacyLogger.logMessage(0, message.c_str());
    }
    
    void error(const std::string& message) override {
        legacyLogger.logMessage(2, message.c_str());
    }
    
    void debug(const std::string& message) override {
        legacyLogger.logMessage(1, message.c_str());
    }
};

// Application uses modern interface
class Application {
private:
    std::shared_ptr<ILogger> logger;
public:
    explicit Application(std::shared_ptr<ILogger> log) : logger(std::move(log)) {}
    
    void processData() {
        logger->info("Processing started");
        // Process data...
        logger->info("Processing completed");
    }
};
```

**Usage:**
```cpp
// Can easily switch between different logging implementations
auto legacyAdapter = std::make_shared<LegacyLoggerAdapter>();
auto modernLogger = std::make_shared<ModernLogger>();

Application app(legacyAdapter); // Uses legacy system
// Application app(modernLogger); // Uses modern system
```

## 9. Template Method Pattern for Algorithm Structure
**Description:** Defines the skeleton of an algorithm in a base class, letting subclasses override specific steps without changing the algorithm's structure.

**Purpose:** Introduced to provide consistent algorithm frameworks while allowing customization for different Linux system configurations and requirements.

**Before (Duplicated Algorithm Structure):**
```cpp
class FileProcessor {
public:
    void processFile(const std::string& filename) {
        // Algorithm steps mixed with specific implementation
        if (validateFile(filename)) {
            auto data = readFile(filename);
            auto processed = processData(data);
            writeFile(filename + ".processed", processed);
            cleanup(filename);
        }
    }
    
private:
    bool validateFile(const std::string& filename) {
        return access(filename.c_str(), R_OK) == 0;
    }
    
    std::string readFile(const std::string& filename) {
        std::ifstream file(filename);
        return std::string(std::istreambuf_iterator<char>(file),
                          std::istreambuf_iterator<char>());
    }
    
    std::string processData(const std::string& data) {
        // Specific processing logic
        return data;
    }
    
    void writeFile(const std::string& filename, const std::string& data) {
        std::ofstream file(filename);
        file << data;
    }
    
    void cleanup(const std::string& filename) {
        // Cleanup logic
    }
};
```

**After (Template Method Pattern):**
```cpp
// Abstract base class with template method
class FileProcessor {
public:
    // Template method - defines algorithm structure
    void processFile(const std::string& filename) {
        if (validateFile(filename)) {
            auto data = readFile(filename);
            auto processed = processData(data);
            writeFile(filename + ".processed", processed);
            cleanup(filename);
        }
    }
    
    virtual ~FileProcessor() = default;
    
protected:
    // Hook methods - can be overridden by subclasses
    virtual bool validateFile(const std::string& filename) {
        return access(filename.c_str(), R_OK) == 0;
    }
    
    virtual std::string readFile(const std::string& filename) {
        std::ifstream file(filename);
        return std::string(std::istreambuf_iterator<char>(file),
                          std::istreambuf_iterator<char>());
    }
    
    virtual std::string processData(const std::string& data) = 0; // Pure virtual
    
    virtual void writeFile(const std::string& filename, const std::string& data) {
        std::ofstream file(filename);
        file << data;
    }
    
    virtual void cleanup(const std::string& filename) {
        // Default cleanup - can be overridden
    }
};

// Concrete implementations
class TextFileProcessor : public FileProcessor {
protected:
    std::string processData(const std::string& data) override {
        // Text-specific processing
        return "Processed: " + data;
    }
};

class BinaryFileProcessor : public FileProcessor {
protected:
    std::string processData(const std::string& data) override {
        // Binary-specific processing
        return "Binary processed: " + data;
    }
    
    void cleanup(const std::string& filename) override {
        // Binary-specific cleanup
        chmod(filename.c_str(), 0644);
    }
};
```

**Usage:**
```cpp
std::unique_ptr<FileProcessor> processor;

if (isTextFile(filename)) {
    processor = std::make_unique<TextFileProcessor>();
} else {
    processor = std::make_unique<BinaryFileProcessor>();
}

processor->processFile(filename); // Same interface, different behavior
```

## 10. Chain of Responsibility Pattern for Request Processing
**Description:** Passes requests along a chain of handlers until one handles it, allowing multiple objects to handle a request without knowing which one will handle it.

**Purpose:** Introduced to create flexible request processing pipelines, especially useful for Linux system event handling, logging, and middleware processing.

**Before (Monolithic Handler):**
```cpp
class RequestHandler {
public:
    void handleRequest(const Request& request) {
        // All handling logic in one place
        if (request.getType() == "auth") {
            // Authentication logic
            if (authenticate(request)) {
                // Authorization logic
                if (authorize(request)) {
                    // Processing logic
                    processRequest(request);
                }
            }
        } else if (request.getType() == "file") {
            // File handling logic
            if (validateFile(request)) {
                processFileRequest(request);
            }
        }
        // Adding new request types requires modifying this class
    }
    
private:
    bool authenticate(const Request& request) { /* ... */ }
    bool authorize(const Request& request) { /* ... */ }
    void processRequest(const Request& request) { /* ... */ }
    bool validateFile(const Request& request) { /* ... */ }
    void processFileRequest(const Request& request) { /* ... */ }
};
```

**After (Chain of Responsibility):**
```cpp
// Handler interface
class IRequestHandler {
public:
    virtual ~IRequestHandler() = default;
    virtual void setNext(std::shared_ptr<IRequestHandler> handler) = 0;
    virtual void handle(const Request& request) = 0;
};

// Base handler with common functionality
class BaseRequestHandler : public IRequestHandler {
protected:
    std::shared_ptr<IRequestHandler> nextHandler;
public:
    void setNext(std::shared_ptr<IRequestHandler> handler) override {
        nextHandler = handler;
    }
    
    void handle(const Request& request) override {
        if (canHandle(request)) {
            processRequest(request);
        } else if (nextHandler) {
            nextHandler->handle(request);
        } else {
            throw std::runtime_error("No handler found for request");
        }
    }
    
protected:
    virtual bool canHandle(const Request& request) = 0;
    virtual void processRequest(const Request& request) = 0;
};

// Concrete handlers
class AuthenticationHandler : public BaseRequestHandler {
protected:
    bool canHandle(const Request& request) override {
        return request.getType() == "auth";
    }
    
    void processRequest(const Request& request) override {
        if (authenticate(request)) {
            std::cout << "Authentication successful" << std::endl;
            if (nextHandler) {
                nextHandler->handle(request);
            }
        } else {
            std::cout << "Authentication failed" << std::endl;
        }
    }
    
private:
    bool authenticate(const Request& request) {
        // Authentication logic
        return true;
    }
};

class AuthorizationHandler : public BaseRequestHandler {
protected:
    bool canHandle(const Request& request) override {
        return request.getType() == "auth";
    }
    
    void processRequest(const Request& request) override {
        if (authorize(request)) {
            std::cout << "Authorization successful" << std::endl;
            if (nextHandler) {
                nextHandler->handle(request);
            }
        } else {
            std::cout << "Authorization failed" << std::endl;
        }
    }
    
private:
    bool authorize(const Request& request) {
        // Authorization logic
        return true;
    }
};

class FileHandler : public BaseRequestHandler {
protected:
    bool canHandle(const Request& request) override {
        return request.getType() == "file";
    }
    
    void processRequest(const Request& request) override {
        if (validateFile(request)) {
            std::cout << "File request processed" << std::endl;
            processFileRequest(request);
        } else {
            std::cout << "File validation failed" << std::endl;
        }
    }
    
private:
    bool validateFile(const Request& request) {
        // File validation logic
        return true;
    }
    
    void processFileRequest(const Request& request) {
        // File processing logic
    }
};
```

**Usage:**
```cpp
// Build the chain
auto authHandler = std::make_shared<AuthenticationHandler>();
auto authzHandler = std::make_shared<AuthorizationHandler>();
auto fileHandler = std::make_shared<FileHandler>();

authHandler->setNext(authzHandler);
authzHandler->setNext(fileHandler);

// Process requests
Request authRequest("auth", "user data");
authHandler->handle(authRequest);

Request fileRequest("file", "file data");
authHandler->handle(fileRequest);
```

## Summary

These top 10 Linux C++11 clean architecture principles provide significant improvements over traditional approaches:

### **Separation of Concerns:**
- Dependency inversion for loose coupling
- Repository pattern for data access abstraction
- Use case pattern for business logic isolation

### **Flexibility and Extensibility:**
- Observer pattern for event-driven systems
- Factory pattern for object creation
- Strategy pattern for algorithm selection

### **Maintainability:**
- Command pattern for operation encapsulation
- Adapter pattern for system integration
- Template method for algorithm structure

### **Scalability:**
- Chain of responsibility for request processing
- Modular design for easy testing
- Interface-based design for component replacement

Remember to compile with `-std=c++11` flag and consider Linux-specific considerations like file permissions, system calls, and process management! 