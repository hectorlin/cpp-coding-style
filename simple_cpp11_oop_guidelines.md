# Simple C++11 OOP Guidelines

## 1. Use Smart Pointers for Memory Management
**Description:** Smart pointers automatically manage memory allocation and deallocation, preventing memory leaks and ensuring exception safety.

**Purpose:** Eliminates manual memory management, prevents memory leaks, and provides clear ownership semantics.

**Usage:**
```cpp
#include <memory>

class Widget {
private:
    std::unique_ptr<Resource> resource;
public:
    Widget() : resource(std::make_unique<Resource>()) {}
    
    // No destructor needed - unique_ptr handles cleanup
    ~Widget() = default;
    
    // Move semantics work automatically
    Widget(Widget&&) = default;
    Widget& operator=(Widget&&) = default;
    
    // No copying allowed (unique_ptr is move-only)
    Widget(const Widget&) = delete;
    Widget& operator=(const Widget&) = delete;
};

// Usage
auto widget = std::make_unique<Widget>();
auto sharedWidget = std::make_shared<Widget>();
```

## 2. Use RAII for Resource Management
**Description:** Resource Acquisition Is Initialization - tie resource lifetime to object lifetime for automatic cleanup.

**Purpose:** Ensures resources are properly released even when exceptions occur, preventing resource leaks.

**Usage:**
```cpp
class FileHandler {
private:
    std::unique_ptr<FILE, decltype(&fclose)> file;
public:
    FileHandler(const char* filename) 
        : file(fopen(filename, "r"), fclose) {
        if (!file) {
            throw std::runtime_error("Failed to open file");
        }
    }
    
    void processFile() {
        // File automatically closed when object goes out of scope
        // even if exception is thrown
    }
};

// Usage
{
    FileHandler handler("data.txt");
    handler.processFile();
} // File automatically closed here
```

## 3. Use Override and Final Keywords
**Description:** Explicitly indicate function overrides and prevent further inheritance or overriding.

**Purpose:** Catch errors at compile time and make code intent clearer.

**Usage:**
```cpp
class Base {
public:
    virtual void process() = 0;
    virtual void cleanup() { /* ... */ }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void process() override { /* ... */ } // Compile-time check
    void cleanup() override { /* ... */ } // Compile-time check
};

class FinalClass final : public Base {
public:
    void process() override { /* ... */ }
    void cleanup() override { /* ... */ }
};

// class FurtherDerived : public FinalClass { }; // Error - can't inherit from final
```

## 4. Use Auto for Type Deduction
**Description:** Let the compiler automatically determine variable types from initializers.

**Purpose:** Reduces verbosity, prevents type mismatches, and makes code more maintainable.

**Usage:**
```cpp
// Automatic type deduction
auto number = 42;                                    // int
auto text = std::string("Hello");                   // std::string
auto ptr = std::make_unique<Widget>();              // std::unique_ptr<Widget>
auto it = std::vector<int>{1, 2, 3}.begin();        // std::vector<int>::iterator

// In templates
template<typename Container>
void processContainer(const Container& c) {
    auto it = c.begin();  // Works with any container type
    auto value = *it;     // Deduces element type
}

// With lambdas
auto lambda = [](int x) { return x * 2; };
auto result = lambda(5);
```

## 5. Use Range-based For Loops
**Description:** Clean syntax for iterating over containers without explicit iterator management.

**Purpose:** Simplifies iteration code and makes it more readable and less error-prone.

**Usage:**
```cpp
std::vector<int> numbers = {1, 2, 3, 4, 5};

// Simple iteration
for (const auto& number : numbers) {
    std::cout << number << std::endl;
}

// Iterating over map
std::map<std::string, int> scores{{"Alice", 100}, {"Bob", 95}};
for (const auto& pair : scores) {
    std::cout << pair.first << ": " << pair.second << std::endl;
}

// Custom containers
class Container {
public:
    auto begin() const { return data.begin(); }
    auto end() const { return data.end(); }
private:
    std::vector<int> data;
};

Container container;
for (const auto& item : container) {
    // Works with any container that has begin()/end()
}
```

## 6. Use Lambda Expressions
**Description:** Create anonymous function objects inline for concise and functional code.

**Purpose:** Simplifies callback functions and reduces boilerplate code.

**Usage:**
```cpp
std::vector<std::string> words = {"hello", "world", "cpp"};

// Simple lambda
std::sort(words.begin(), words.end(), 
    [](const std::string& a, const std::string& b) {
        return a.length() < b.length();
    });

// Lambda with capture
int minLength = 3;
auto filtered = std::find_if(words.begin(), words.end(),
    [minLength](const std::string& word) {
        return word.length() >= minLength;
    });

// Lambda with member function access
class Widget {
public:
    void processItems(const std::vector<int>& items) {
        std::for_each(items.begin(), items.end(),
            [this](int item) {
                this->processItem(item);
            });
    }
private:
    void processItem(int item) { /* ... */ }
};
```

## 7. Use Deleted and Defaulted Functions
**Description:** Explicitly control which functions are generated or prevented.

**Purpose:** Make intent explicit and prevent unwanted function calls.

**Usage:**
```cpp
class NonCopyable {
public:
    NonCopyable() = default;
    ~NonCopyable() = default;
    
    // Prevent copying
    NonCopyable(const NonCopyable&) = delete;
    NonCopyable& operator=(const NonCopyable&) = delete;
    
    // Allow moving
    NonCopyable(NonCopyable&&) = default;
    NonCopyable& operator=(NonCopyable&&) = default;
};

class Widget {
public:
    Widget() { /* custom constructor */ }
    
    // Explicitly request default copy operations
    Widget(const Widget&) = default;
    Widget& operator=(const Widget&) = default;
    
    // Delete unwanted operations
    void* operator new(size_t) = delete; // Prevent heap allocation
};

// Singleton pattern
class Singleton {
public:
    static Singleton& getInstance() {
        static Singleton instance;
        return instance;
    }
    
    void doSomething() { /* ... */ }
    
private:
    Singleton() = default;
    ~Singleton() = default;
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
};
```

## 8. Use Move Semantics
**Description:** Transfer ownership of resources efficiently without copying.

**Purpose:** Eliminates unnecessary copying and improves performance for expensive-to-copy objects.

**Usage:**
```cpp
class String {
private:
    char* data;
    size_t size;
public:
    String(const char* str) {
        size = strlen(str);
        data = new char[size + 1];
        strcpy(data, str);
    }
    
    // Move constructor
    String(String&& other) noexcept 
        : data(other.data), size(other.size) {
        other.data = nullptr;
        other.size = 0;
    }
    
    // Move assignment operator
    String& operator=(String&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            size = other.size;
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }
    
    ~String() { delete[] data; }
};

// Usage
std::vector<String> strings;
strings.push_back(String("Hello")); // Move constructor used

String s1("World");
String s2 = std::move(s1); // Explicit move
```

## 9. Use Uniform Initialization
**Description:** Consistent syntax for initializing objects using braces.

**Purpose:** Provides consistent initialization syntax and prevents narrowing conversions.

**Usage:**
```cpp
// Basic types
int x{5};
double y{3.14};
std::string text{"Hello"};

// Containers
std::vector<int> numbers{1, 2, 3, 4, 5};
std::map<std::string, int> scores{{"Alice", 100}, {"Bob", 95}};

// Objects
class Point {
public:
    Point(int x, int y) : x_(x), y_(y) {}
private:
    int x_, y_;
};

Point p{10, 20};
std::vector<Point> points{{1, 2}, {3, 4}, {5, 6}};

// Smart pointers
auto widget = std::make_unique<Widget>(Widget{param1, param2});
```

## 10. Use Constexpr for Compile-time Computations
**Description:** Evaluate functions and variables at compile time for better performance.

**Purpose:** Move computations to compile time and provide compile-time constants.

**Usage:**
```cpp
// Compile-time functions
constexpr int factorial(int n) {
    return (n <= 1) ? 1 : n * factorial(n - 1);
}

constexpr int power(int base, int exp) {
    return (exp == 0) ? 1 : base * power(base, exp - 1);
}

// Compile-time constants
constexpr int MAX_SIZE = 100;
constexpr double PI = 3.14159;

// Compile-time evaluation
constexpr int result = factorial(10); // Computed at compile time
constexpr int square = power(5, 2);   // 25

// Runtime evaluation still works
int runtime_result = factorial(user_input);

// Use in templates
template<int N>
class Array {
    int data[N];
};

Array<factorial(5)> arr; // Array<120>
```

## Summary

These simple C++11 OOP guidelines focus on:

### **Memory Safety:**
- Smart pointers for automatic memory management
- RAII for resource cleanup
- Move semantics for efficient transfers

### **Code Clarity:**
- Auto for reduced verbosity
- Override/final for explicit intent
- Range-based for loops for cleaner iteration

### **Performance:**
- Move semantics eliminate unnecessary copying
- Constexpr enables compile-time computations
- Smart pointers provide zero-overhead abstractions

### **Maintainability:**
- Lambda expressions for inline functionality
- Deleted/defaulted functions for explicit control
- Uniform initialization for consistency

Remember to compile with `-std=c++11` flag to enable these features! 