# Top 10 C++11 OOP Style Guidelines

## 1. Use RAII (Resource Acquisition Is Initialization)
**Description:** RAII is a programming idiom where resource management is tied to object lifetime. Resources are acquired during object construction and automatically released during destruction.

**Purpose:** Introduced to prevent resource leaks, ensure exception safety, and provide deterministic resource cleanup without manual memory management.

**Before (C++98):**
```cpp
class FileHandler {
private:
    FILE* file;
public:
    FileHandler(const char* filename) {
        file = fopen(filename, "r");
        if (!file) {
            throw std::runtime_error("Failed to open file");
        }
    }
    
    ~FileHandler() {
        if (file) {
            fclose(file);
        }
    }
    
    // Manual resource management - error prone
    void processFile() {
        // What if exception is thrown here?
        // File might not be closed properly
    }
};
```

**After (C++11):**
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
    
    // Destructor not needed - unique_ptr handles cleanup
    ~FileHandler() = default;
    
    void processFile() {
        // Exception safe - file will be closed automatically
        // even if exception is thrown
    }
};
```

**Usage:**
```cpp
// Automatic cleanup when object goes out of scope
{
    FileHandler handler("data.txt");
    handler.processFile();
} // File automatically closed here, even if exception occurred
```

## 2. Use Smart Pointers Instead of Raw Pointers
**Description:** Smart pointers (unique_ptr, shared_ptr, weak_ptr) provide automatic memory management with ownership semantics and exception safety.

**Purpose:** Eliminates manual memory management, prevents memory leaks, and provides clear ownership semantics while maintaining exception safety.

**Before (C++98):**
```cpp
class Widget {
private:
    Resource* resource;
public:
    Widget() {
        resource = new Resource();
    }
    
    ~Widget() {
        delete resource; // Manual cleanup
    }
    
    // Need copy constructor and assignment operator
    Widget(const Widget& other) {
        resource = new Resource(*other.resource);
    }
    
    Widget& operator=(const Widget& other) {
        if (this != &other) {
            delete resource;
            resource = new Resource(*other.resource);
        }
        return *this;
    }
};
```

**After (C++11):**
```cpp
class Widget {
private:
    std::unique_ptr<Resource> resource;
public:
    Widget() : resource(std::make_unique<Resource>()) {}
    
    // No destructor needed - unique_ptr handles cleanup
    ~Widget() = default;
    
    // No copy constructor/assignment needed - unique_ptr is move-only
    Widget(const Widget&) = delete;
    Widget& operator=(const Widget&) = delete;
    
    // Move semantics work automatically
    Widget(Widget&&) = default;
    Widget& operator=(Widget&&) = default;
};
```

**Usage:**
```cpp
// Unique ownership
auto widget1 = std::make_unique<Widget>();

// Shared ownership
auto widget2 = std::make_shared<Widget>();
auto widget3 = widget2; // Reference count increased

// Weak reference (prevents circular references)
std::weak_ptr<Widget> weak_widget = widget2;
```

## 3. Use Move Semantics for Efficient Resource Transfer
**Description:** Move semantics allow transferring ownership of resources from one object to another without copying, improving performance for expensive-to-copy resources.

**Purpose:** Introduced to eliminate unnecessary copying of large objects, improve performance, and provide clear ownership transfer semantics.

**Before (C++98):**
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
    
    // Expensive copy constructor
    String(const String& other) {
        size = other.size;
        data = new char[size + 1];
        strcpy(data, other.data);
    }
    
    ~String() {
        delete[] data;
    }
};

// Usage - expensive copying
String createString() {
    String temp("Hello World");
    return temp; // Expensive copy here
}
```

**After (C++11):**
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
    
    // Move constructor - efficient transfer
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
    
    ~String() {
        delete[] data;
    }
};

// Usage - efficient move
String createString() {
    String temp("Hello World");
    return std::move(temp); // Efficient move here
}
```

**Usage:**
```cpp
std::vector<String> strings;
strings.push_back(String("Hello")); // Move constructor used

String s1("World");
String s2 = std::move(s1); // Explicit move
```

## 4. Use Override and Final Keywords
**Description:** The `override` keyword explicitly indicates that a function is intended to override a virtual function from a base class. The `final` keyword prevents further inheritance or overriding.

**Purpose:** Introduced to catch errors at compile time, make code intent clearer, and prevent accidental hiding of virtual functions.

**Before (C++98):**
```cpp
class Base {
public:
    virtual void process() = 0;
    virtual void cleanup() { /* ... */ }
};

class Derived : public Base {
public:
    // Easy to make mistakes - no compile-time checking
    void process() { /* ... */ } // Correct
    void Process() { /* ... */ } // Wrong - doesn't override
    void cleanup(int param) { /* ... */ } // Wrong - doesn't override
};
```

**After (C++11):**
```cpp
class Base {
public:
    virtual void process() = 0;
    virtual void cleanup() { /* ... */ }
};

class Derived : public Base {
public:
    void process() override { /* ... */ } // Compile-time check
    void cleanup() override { /* ... */ } // Compile-time check
    
    // Compiler error if signature doesn't match
    // void Process() override { /* ... */ } // Error!
    // void cleanup(int param) override { /* ... */ } // Error!
};

class FinalClass final : public Base {
public:
    void process() override { /* ... */ }
    void cleanup() override { /* ... */ }
};

// class FurtherDerived : public FinalClass { }; // Error - can't inherit from final class
```

**Usage:**
```cpp
class Shape {
public:
    virtual double area() const = 0;
    virtual void draw() const = 0;
    virtual ~Shape() = default;
};

class Circle final : public Shape {
public:
    double area() const override { return 3.14159 * radius * radius; }
    void draw() const override { /* draw circle */ }
private:
    double radius;
};
```

## 5. Use Deleted Functions and Defaulted Functions
**Description:** `= delete` prevents function calls, while `= default` explicitly requests the compiler to generate default implementations.

**Purpose:** Introduced to prevent unwanted function calls, make intent explicit, and provide better control over class behavior.

**Before (C++98):**
```cpp
class NonCopyable {
private:
    // Hide copy constructor and assignment operator
    NonCopyable(const NonCopyable&);
    NonCopyable& operator=(const NonCopyable&);
public:
    NonCopyable() { /* ... */ }
    ~NonCopyable() { /* ... */ }
};

class Widget {
public:
    Widget() { /* custom constructor */ }
    // Compiler generates default copy constructor and assignment
    // Might not be what we want
};
```

**After (C++11):**
```cpp
class NonCopyable {
public:
    NonCopyable() = default;
    ~NonCopyable() = default;
    
    // Explicitly delete copy operations
    NonCopyable(const NonCopyable&) = delete;
    NonCopyable& operator=(const NonCopyable&) = delete;
    
    // Allow move operations
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
```

**Usage:**
```cpp
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
    
    // Prevent copying and assignment
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
};
```

## 6. Use Range-based For Loops
**Description:** Range-based for loops provide a cleaner syntax for iterating over containers and ranges without explicit iterator management.

**Purpose:** Introduced to simplify iteration code, reduce boilerplate, and make code more readable and less error-prone.

**Before (C++98):**
```cpp
std::vector<int> numbers = {1, 2, 3, 4, 5};

// Manual iteration with iterators
for (std::vector<int>::iterator it = numbers.begin(); 
     it != numbers.end(); ++it) {
    std::cout << *it << std::endl;
}

// Or with indices
for (size_t i = 0; i < numbers.size(); ++i) {
    std::cout << numbers[i] << std::endl;
}

// Iterating over map
std::map<std::string, int> scores;
for (std::map<std::string, int>::iterator it = scores.begin(); 
     it != scores.end(); ++it) {
    std::cout << it->first << ": " << it->second << std::endl;
}
```

**After (C++11):**
```cpp
std::vector<int> numbers = {1, 2, 3, 4, 5};

// Clean range-based iteration
for (const auto& number : numbers) {
    std::cout << number << std::endl;
}

// Iterating over map
std::map<std::string, int> scores;
for (const auto& pair : scores) {
    std::cout << pair.first << ": " << pair.second << std::endl;
}

// With structured bindings (C++17)
for (const auto& [key, value] : scores) {
    std::cout << key << ": " << value << std::endl;
}
```

**Usage:**
```cpp
class Container {
public:
    // Custom iterator support for range-based for
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

## 7. Use Lambda Expressions
**Description:** Lambda expressions provide a way to create anonymous function objects inline, making code more concise and functional.

**Purpose:** Introduced to simplify callback functions, reduce boilerplate code, and enable functional programming patterns in C++.

**Before (C++98):**
```cpp
// Need to define separate function or functor
struct CompareByLength {
    bool operator()(const std::string& a, const std::string& b) const {
        return a.length() < b.length();
    }
};

std::vector<std::string> words = {"hello", "world", "cpp"};
std::sort(words.begin(), words.end(), CompareByLength());

// Or with function pointer
bool compareByLength(const std::string& a, const std::string& b) {
    return a.length() < b.length();
}

std::sort(words.begin(), words.end(), compareByLength);
```

**After (C++11):**
```cpp
std::vector<std::string> words = {"hello", "world", "cpp"};

// Inline lambda expression
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
```

**Usage:**
```cpp
class Widget {
public:
    void processItems(const std::vector<int>& items) {
        // Lambda with member function access
        std::for_each(items.begin(), items.end(),
            [this](int item) {
                this->processItem(item);
            });
    }
    
private:
    void processItem(int item) { /* ... */ }
};

// Lambda for async operations
auto future = std::async(std::launch::async, []() {
    return std::string("Async result");
});
```

## 8. Use Auto for Type Deduction
**Description:** The `auto` keyword allows the compiler to automatically deduce the type of a variable from its initializer.

**Purpose:** Introduced to reduce verbosity, prevent type mismatches, and make code more maintainable by letting the compiler handle type inference.

**Before (C++98):**
```cpp
// Verbose type declarations
std::vector<int>::iterator it = numbers.begin();
std::map<std::string, int>::const_iterator map_it = scores.begin();
std::unique_ptr<Widget> widget = std::make_unique<Widget>();

// Easy to make mistakes
std::vector<int>::iterator wrong_it = numbers.cbegin(); // Wrong type!
```

**After (C++11):**
```cpp
// Automatic type deduction
auto it = numbers.begin();
auto map_it = scores.begin();
auto widget = std::make_unique<Widget>();

// Compiler deduces correct type
auto const_it = numbers.cbegin(); // Correctly deduced as const_iterator
```

**Usage:**
```cpp
// With complex types
auto result = std::make_shared<ComplexType>(param1, param2);

// In templates
template<typename Container>
void processContainer(const Container& c) {
    auto it = c.begin(); // Works with any container type
    auto value = *it;    // Deduces element type
}

// With lambdas
auto lambda = [](int x) { return x * 2; };
auto result = lambda(5);
```

## 9. Use Uniform Initialization
**Description:** Uniform initialization provides a consistent syntax for initializing objects using braces `{}`, applicable to all types.

**Purpose:** Introduced to provide a consistent initialization syntax, prevent narrowing conversions, and eliminate the "most vexing parse" problem.

**Before (C++98):**
```cpp
// Inconsistent initialization syntax
int x = 5;
int y(10);
std::vector<int> v;
v.push_back(1);
v.push_back(2);
v.push_back(3);

// Most vexing parse
class Widget {
public:
    Widget(int x) { /* ... */ }
};

Widget w(); // Declares a function, not a Widget object!
```

**After (C++11):**
```cpp
// Uniform initialization syntax
int x{5};
int y{10};
std::vector<int> v{1, 2, 3};

// No most vexing parse
Widget w{}; // Creates a Widget object

// Works with any type
std::map<std::string, int> scores{{"Alice", 100}, {"Bob", 95}};
std::pair<int, std::string> pair{42, "answer"};
```

**Usage:**
```cpp
class Point {
public:
    Point(int x, int y) : x_(x), y_(y) {}
    
private:
    int x_, y_;
};

// Uniform initialization
Point p{10, 20};

// With containers
std::vector<Point> points{{1, 2}, {3, 4}, {5, 6}};

// With smart pointers
auto widget = std::make_unique<Widget>(Widget{param1, param2});
```

## 10. Use Constexpr for Compile-time Computations
**Description:** `constexpr` allows functions and variables to be evaluated at compile time, enabling compile-time computations and optimizations.

**Purpose:** Introduced to enable compile-time evaluation, improve performance by moving computations to compile time, and provide compile-time constants.

**Before (C++98):**
```cpp
// Runtime computation
int factorial(int n) {
    return (n <= 1) ? 1 : n * factorial(n - 1);
}

// Compile-time constants limited to simple expressions
const int MAX_SIZE = 100;
const double PI = 3.14159;

// No compile-time function evaluation
int result = factorial(10); // Computed at runtime
```

**After (C++11):**
```cpp
// Compile-time computation
constexpr int factorial(int n) {
    return (n <= 1) ? 1 : n * factorial(n - 1);
}

// Compile-time constants
constexpr int MAX_SIZE = 100;
constexpr double PI = 3.14159;

// Compile-time evaluation
constexpr int result = factorial(10); // Computed at compile time

// Runtime evaluation still works
int runtime_result = factorial(user_input);
```

**Usage:**
```cpp
class MathUtils {
public:
    static constexpr double PI = 3.14159265359;
    
    constexpr static int power(int base, int exp) {
        return (exp == 0) ? 1 : base * power(base, exp - 1);
    }
    
    constexpr static int gcd(int a, int b) {
        return (b == 0) ? a : gcd(b, a % b);
    }
};

// Compile-time computations
constexpr int square = MathUtils::power(5, 2); // 25
constexpr int gcd_result = MathUtils::gcd(12, 18); // 6

// Use in template parameters
template<int N>
class Array {
    int data[N];
};

Array<MathUtils::power(2, 3)> arr; // Array<8>
```

## Summary

These top 10 C++11 OOP style guidelines provide significant improvements over previous approaches:

### **Resource Management:**
- RAII for automatic resource cleanup
- Smart pointers for memory safety
- Move semantics for efficient transfers

### **Code Safety:**
- Override/final keywords for compile-time checking
- Deleted functions to prevent unwanted operations
- Constexpr for compile-time validation

### **Code Clarity:**
- Range-based for loops for cleaner iteration
- Lambda expressions for inline functionality
- Auto for reduced verbosity
- Uniform initialization for consistency

### **Performance:**
- Move semantics eliminate unnecessary copying
- Constexpr enables compile-time computations
- Smart pointers provide zero-overhead abstractions

Remember to compile with `-std=c++11` flag to enable these features! 