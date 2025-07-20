# Top 50 C++11 Q&A (Easy to Hard)

## Beginner Level (1-15)

### Q1: What is `auto` and why use it?
**A:** `auto` lets the compiler automatically deduce variable types from initializers.

**Why use it:** Reduces verbosity, prevents type mismatches, makes code more maintainable.

```cpp
// Before C++11
std::vector<int>::iterator it = vec.begin();
std::map<std::string, int>::const_iterator map_it = scores.begin();

// With C++11 auto
auto it = vec.begin();
auto map_it = scores.begin();
```

### Q2: What are smart pointers and why use them?
**A:** Smart pointers (`unique_ptr`, `shared_ptr`, `weak_ptr`) automatically manage memory.

**Why use them:** Prevents memory leaks, provides exception safety, clear ownership semantics.

```cpp
// Before C++11
int* ptr = new int(42);
// ... use ptr
delete ptr; // Easy to forget

// With C++11
auto ptr = std::make_unique<int>(42);
// Automatically deleted when ptr goes out of scope
```

### Q3: What is `nullptr` and why use it?
**A:** `nullptr` is a keyword representing a null pointer constant.

**Why use it:** Type-safe, distinguishes from integer 0, clearer intent.

```cpp
// Before C++11
void* ptr = NULL; // or 0
if (ptr == 0) { /* ... */ }

// With C++11
void* ptr = nullptr;
if (ptr == nullptr) { /* ... */ }
```

### Q4: What are range-based for loops?
**A:** Clean syntax for iterating over containers without explicit iterators.

**Why use them:** Simpler, less error-prone, more readable.

```cpp
// Before C++11
for (std::vector<int>::iterator it = vec.begin(); it != vec.end(); ++it) {
    std::cout << *it << std::endl;
}

// With C++11
for (const auto& item : vec) {
    std::cout << item << std::endl;
}
```

### Q5: What is uniform initialization?
**A:** Consistent brace `{}` syntax for initializing any type.

**Why use it:** Prevents narrowing conversions, eliminates "most vexing parse".

```cpp
// Before C++11
int x = 5;
std::vector<int> v;
v.push_back(1); v.push_back(2); v.push_back(3);

// With C++11
int x{5};
std::vector<int> v{1, 2, 3};
```

### Q6: What is `override` keyword?
**A:** Explicitly indicates a function overrides a virtual function.

**Why use it:** Compile-time checking, clearer intent, prevents accidental hiding.

```cpp
class Base {
public:
    virtual void process() = 0;
};

class Derived : public Base {
public:
    void process() override { /* ... */ } // Compiler checks signature
};
```

### Q7: What is `final` keyword?
**A:** Prevents further inheritance or overriding of a class/function.

**Why use it:** Design intent, performance optimization, prevents misuse.

```cpp
class Base {
public:
    virtual void process() final { /* ... */ } // Can't override
};

class Derived final : public Base { }; // Can't inherit from Derived
```

### Q8: What are `= delete` and `= default`?
**A:** Explicitly delete or request default implementations of special functions.

**Why use them:** Clear intent, prevent unwanted operations, control class behavior.

```cpp
class Widget {
public:
    Widget() = default;
    Widget(const Widget&) = delete; // Prevent copying
    Widget& operator=(const Widget&) = delete;
};
```

### Q9: What are lambda expressions?
**A:** Anonymous function objects created inline.

**Why use them:** Reduce boilerplate, functional programming, local function definitions.

```cpp
// Before C++11
struct Compare {
    bool operator()(int a, int b) const { return a < b; }
};
std::sort(vec.begin(), vec.end(), Compare());

// With C++11
std::sort(vec.begin(), vec.end(), [](int a, int b) { return a < b; });
```

### Q10: What is `constexpr`?
**A:** Allows functions and variables to be evaluated at compile time.

**Why use it:** Performance improvement, compile-time validation, template metaprogramming.

```cpp
constexpr int factorial(int n) {
    return (n <= 1) ? 1 : n * factorial(n - 1);
}

constexpr int result = factorial(10); // Computed at compile time
```

### Q11: What are `std::array`?
**A:** Fixed-size array container with STL interface.

**Why use it:** Type safety, STL compatibility, no decay to pointer.

```cpp
// Before C++11
int arr[10];
// No size() method, no iterators

// With C++11
std::array<int, 10> arr;
auto size = arr.size(); // 10
for (const auto& item : arr) { /* ... */ }
```

### Q12: What is `std::tuple`?
**A:** Fixed-size collection of heterogeneous values.

**Why use it:** Return multiple values, structured data, template metaprogramming.

```cpp
auto getPerson() {
    return std::make_tuple("John", 30, "Engineer");
}

auto [name, age, job] = getPerson(); // C++17 structured bindings
```

### Q13: What is `std::function`?
**A:** Polymorphic function wrapper for callable objects.

**Why use it:** Store any callable, callback functions, type erasure.

```cpp
std::function<int(int, int)> operation = [](int a, int b) { return a + b; };
int result = operation(3, 4); // 7
```

### Q14: What is `std::bind`?
**A:** Creates function objects with bound arguments.

**Why use it:** Partial function application, callback adaptation, argument reordering.

```cpp
auto add = [](int a, int b) { return a + b; };
auto addFive = std::bind(add, 5, std::placeholders::_1);
int result = addFive(3); // 8
```

### Q15: What is `std::ref` and `std::cref`?
**A:** Create reference wrappers for passing references to templates.

**Why use them:** Pass references to templates that expect values, avoid copying.

```cpp
int value = 42;
auto ref_wrapper = std::ref(value);
auto const_ref_wrapper = std::cref(value);
```

## Intermediate Level (16-35)

### Q16: What are move semantics?
**A:** Transfer ownership of resources without copying.

**Why use them:** Performance improvement, eliminate unnecessary copying.

```cpp
class String {
private:
    char* data;
public:
    // Move constructor
    String(String&& other) noexcept : data(other.data) {
        other.data = nullptr;
    }
    
    // Move assignment
    String& operator=(String&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            other.data = nullptr;
        }
        return *this;
    }
};
```

### Q17: What is `std::move`?
**A:** Converts lvalue to rvalue reference for move semantics.

**Why use it:** Enable move semantics, transfer ownership, performance optimization.

```cpp
std::vector<std::string> vec;
std::string str = "Hello";
vec.push_back(std::move(str)); // Move instead of copy
// str is now in valid but unspecified state
```

### Q18: What is `std::forward`?
**A:** Perfect forwarding of arguments preserving value category.

**Why use it:** Template forwarding, preserve lvalue/rvalue nature of arguments.

```cpp
template<typename T>
void wrapper(T&& arg) {
    foo(std::forward<T>(arg)); // Preserves lvalue/rvalue
}
```

### Q19: What are rvalue references?
**A:** References that can bind to temporary objects.

**Why use them:** Move semantics, perfect forwarding, performance optimization.

```cpp
void process(String&& str) { // Only accepts rvalues
    // Can safely move from str
}

String s = "Hello";
process(std::move(s)); // Explicit move
process(String("World")); // Temporary object
```

### Q20: What is `noexcept`?
**A:** Specifies that a function does not throw exceptions.

**Why use it:** Performance optimization, compile-time checking, interface contracts.

```cpp
void process() noexcept {
    // Guaranteed not to throw
}

// Compile-time checking
static_assert(noexcept(process()), "process should not throw");
```

### Q21: What are variadic templates?
**A:** Templates that accept variable number of template arguments.

**Why use them:** Generic programming, type-safe variable argument functions.

```cpp
template<typename... Args>
void print(Args&&... args) {
    (std::cout << ... << args) << std::endl; // C++17 fold expression
}

print("Hello", 42, 3.14); // Works with any number of arguments
```

### Q22: What is `std::initializer_list`?
**A:** Lightweight proxy for array of const elements.

**Why use it:** Support brace initialization, variable argument constructors.

```cpp
class Container {
public:
    Container(std::initializer_list<int> list) {
        for (int item : list) {
            data.push_back(item);
        }
    }
private:
    std::vector<int> data;
};

Container c{1, 2, 3, 4, 5};
```

### Q23: What is `std::chrono`?
**A:** Time utilities for measuring time and durations.

**Why use it:** Type-safe time operations, high-resolution timing, portable code.

```cpp
auto start = std::chrono::high_resolution_clock::now();
// ... do work
auto end = std::chrono::high_resolution_clock::now();
auto duration = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);
```

### Q24: What are `std::thread`?
**A:** Thread management and synchronization.

**Why use them:** Concurrent programming, parallel execution, modern threading.

```cpp
void worker(int id) {
    std::cout << "Worker " << id << " started" << std::endl;
}

std::thread t1(worker, 1);
std::thread t2(worker, 2);
t1.join();
t2.join();
```

### Q25: What is `std::mutex`?
**A:** Mutual exclusion for thread synchronization.

**Why use it:** Thread safety, protect shared resources, prevent race conditions.

```cpp
std::mutex mtx;
int shared_data = 0;

void increment() {
    std::lock_guard<std::mutex> lock(mtx);
    ++shared_data;
}
```

### Q26: What is `std::condition_variable`?
**A:** Synchronization primitive for waiting on conditions.

**Why use it:** Producer-consumer patterns, thread coordination, efficient waiting.

```cpp
std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void producer() {
    std::lock_guard<std::mutex> lock(mtx);
    ready = true;
    cv.notify_one();
}

void consumer() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, []{ return ready; });
}
```

### Q27: What is `std::future` and `std::async`?
**A:** Asynchronous task execution and result retrieval.

**Why use them:** Asynchronous programming, parallel execution, future values.

```cpp
auto future = std::async(std::launch::async, []() {
    return std::string("Async result");
});

std::string result = future.get(); // Wait for result
```

### Q28: What is `std::atomic`?
**A:** Atomic operations for lock-free programming.

**Why use it:** Performance, lock-free algorithms, hardware-level synchronization.

```cpp
std::atomic<int> counter{0};

void increment() {
    ++counter; // Atomic operation
}

int value = counter.load(); // Atomic read
counter.store(42); // Atomic write
```

### Q29: What is `std::regex`?
**A:** Regular expression support in standard library.

**Why use it:** Pattern matching, text processing, validation.

```cpp
std::regex email_pattern(R"([a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,})");
std::string email = "user@example.com";

if (std::regex_match(email, email_pattern)) {
    std::cout << "Valid email" << std::endl;
}
```

### Q30: What is `std::random`?
**A:** Modern random number generation facilities.

**Why use it:** Better quality random numbers, multiple distributions, thread safety.

```cpp
std::random_device rd;
std::mt19937 gen(rd());
std::uniform_int_distribution<> dis(1, 100);

int random_number = dis(gen);
```

### Q31: What is `std::unordered_map`?
**A:** Hash table container for key-value pairs.

**Why use it:** O(1) average lookup, when order doesn't matter.

```cpp
std::unordered_map<std::string, int> scores;
scores["Alice"] = 100;
scores["Bob"] = 95;

auto it = scores.find("Alice");
if (it != scores.end()) {
    std::cout << it->second << std::endl;
}
```

### Q32: What is `std::unordered_set`?
**A:** Hash table container for unique elements.

**Why use it:** O(1) average lookup, when order doesn't matter.

```cpp
std::unordered_set<std::string> names;
names.insert("Alice");
names.insert("Bob");

if (names.find("Alice") != names.end()) {
    std::cout << "Found Alice" << std::endl;
}
```

### Q33: What is `std::forward_list`?
**A:** Singly-linked list container.

**Why use it:** Memory efficiency, when you only need forward iteration.

```cpp
std::forward_list<int> list;
list.push_front(1);
list.push_front(2);

for (const auto& item : list) {
    std::cout << item << std::endl;
}
```

### Q34: What is `std::array` with custom allocators?
**A:** Fixed-size array with custom memory allocation.

**Why use it:** Memory pool allocation, custom memory management.

```cpp
std::array<int, 10> arr;
// Custom allocator example would be more complex
```

### Q35: What is `std::bitset`?
**A:** Fixed-size sequence of bits.

**Why use it:** Bit manipulation, flags, compact storage.

```cpp
std::bitset<8> bits(42);
std::cout << bits << std::endl; // 00101010
std::cout << bits.count() << std::endl; // Number of set bits
```

## Advanced Level (36-50)

### Q36: What is SFINAE?
**A:** Substitution Failure Is Not An Error - template metaprogramming technique.

**Why use it:** Compile-time type checking, conditional compilation, template specialization.

```cpp
template<typename T>
auto hasSize(T&& t) -> decltype(t.size(), std::true_type{}) {
    return std::true_type{};
}

template<typename T>
auto hasSize(...) -> std::false_type {
    return std::false_type{};
}
```

### Q37: What is `std::enable_if`?
**A:** Template metaprogramming utility for conditional compilation.

**Why use it:** SFINAE, template specialization, compile-time type checking.

```cpp
template<typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
add(T a, T b) {
    return a + b;
}

// Only works with integral types
```

### Q38: What is `std::is_same`?
**A:** Type trait to check if two types are the same.

**Why use it:** Template metaprogramming, compile-time type checking.

```cpp
template<typename T>
void process() {
    if constexpr (std::is_same_v<T, int>) {
        // T is int
    } else if constexpr (std::is_same_v<T, std::string>) {
        // T is string
    }
}
```

### Q39: What is `std::declval`?
**A:** Creates a reference to a type without constructing an object.

**Why use it:** Template metaprogramming, decltype expressions.

```cpp
template<typename T>
auto getSize(T&& t) -> decltype(t.size()) {
    return t.size();
}

// Works with declval for incomplete types
template<typename T>
auto hasSize() -> decltype(std::declval<T>().size(), std::true_type{}) {
    return std::true_type{};
}
```

### Q40: What is `std::result_of`?
**A:** Determines the return type of a callable.

**Why use it:** Template metaprogramming, generic programming.

```cpp
template<typename F, typename... Args>
typename std::result_of<F(Args...)>::type
call(F&& f, Args&&... args) {
    return std::forward<F>(f)(std::forward<Args>(args)...);
}
```

### Q41: What is `std::tuple_element`?
**A:** Access tuple elements by index at compile time.

**Why use it:** Template metaprogramming, generic tuple processing.

```cpp
auto tuple = std::make_tuple(1, 2.0, "three");
using FirstType = std::tuple_element_t<0, decltype(tuple)>; // int
using SecondType = std::tuple_element_t<1, decltype(tuple)>; // double
```

### Q42: What is `std::tuple_size`?
**A:** Get the size of a tuple at compile time.

**Why use it:** Template metaprogramming, generic tuple processing.

```cpp
auto tuple = std::make_tuple(1, 2.0, "three");
constexpr size_t size = std::tuple_size_v<decltype(tuple)>; // 3
```

### Q43: What is `std::index_sequence`?
**A:** Compile-time integer sequence for template metaprogramming.

**Why use it:** Unpacking tuples, variadic template processing.

```cpp
template<typename Tuple, size_t... I>
auto tupleToVector(Tuple&& t, std::index_sequence<I...>) {
    return std::vector{std::get<I>(std::forward<Tuple>(t))...};
}

template<typename Tuple>
auto tupleToVector(Tuple&& t) {
    return tupleToVector(std::forward<Tuple>(t),
                        std::make_index_sequence<std::tuple_size_v<std::decay_t<Tuple>>>{});
}
```

### Q44: What is `std::void_t`?
**A:** Utility for SFINAE and template metaprogramming.

**Why use it:** Type checking, conditional compilation.

```cpp
template<typename T, typename = void>
struct hasSize : std::false_type {};

template<typename T>
struct hasSize<T, std::void_t<decltype(std::declval<T>().size())>> 
    : std::true_type {};
```

### Q45: What is `std::conjunction`?
**A:** Logical AND of type traits.

**Why use it:** Template metaprogramming, complex type checking.

```cpp
template<typename T>
constexpr bool isNumeric = std::conjunction_v<
    std::is_arithmetic<T>,
    std::negation<std::is_same<T, bool>>
>;
```

### Q46: What is `std::disjunction`?
**A:** Logical OR of type traits.

**Why use it:** Template metaprogramming, complex type checking.

```cpp
template<typename T>
constexpr bool isStringLike = std::disjunction_v<
    std::is_same<T, std::string>,
    std::is_same<T, const char*>
>;
```

### Q47: What is `std::negation`?
**A:** Logical NOT of a type trait.

**Why use it:** Template metaprogramming, type trait composition.

```cpp
template<typename T>
constexpr bool isNotVoid = std::negation_v<std::is_void<T>>;
```

### Q48: What is `std::is_invocable`?
**A:** Checks if a type can be called with given arguments.

**Why use it:** Template metaprogramming, compile-time interface checking.

```cpp
template<typename F, typename... Args>
constexpr bool canCall = std::is_invocable_v<F, Args...>;
```

### Q49: What is `std::is_invocable_r`?
**A:** Checks if a type can be called and returns a specific type.

**Why use it:** Template metaprogramming, return type checking.

```cpp
template<typename F, typename R, typename... Args>
constexpr bool returnsType = std::is_invocable_r_v<R, F, Args...>;
```

### Q50: What is `std::remove_reference`?
**A:** Removes reference from a type.

**Why use it:** Template metaprogramming, perfect forwarding.

```cpp
template<typename T>
void process(T&& value) {
    using ValueType = std::remove_reference_t<T>;
    // ValueType is T without reference
}
```

## Summary

This Q&A guide covers C++11 features from basic to advanced:

### **Beginner (1-15):**
- Basic language features (auto, smart pointers, nullptr)
- Simple syntax improvements (range-based for, uniform initialization)
- Basic keywords (override, final, delete, default)

### **Intermediate (16-35):**
- Move semantics and performance features
- Concurrency and threading
- Advanced containers and utilities
- Template basics

### **Advanced (36-50):**
- Template metaprogramming
- SFINAE and type traits
- Advanced template utilities
- Compile-time programming

Each feature includes practical examples and explanations of why it's useful in modern C++ development! 