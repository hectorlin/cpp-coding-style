# Complete List of C++11 Keywords

## New Keywords Introduced in C++11

### 1. `auto`
**Purpose:** Automatic type deduction
```cpp
auto x = 42;           // int
auto str = "hello";    // const char*
auto vec = std::vector<int>{1, 2, 3}; // std::vector<int>
```

### 2. `decltype`
**Purpose:** Type deduction from expression
```cpp
int x = 42;
decltype(x) y = 10;           // y is int
decltype(x + y) result = 0;   // result is int
```

### 3. `nullptr`
**Purpose:** Null pointer literal
```cpp
int* ptr = nullptr;           // Instead of NULL or 0
void* vptr = nullptr;
```

### 4. `constexpr`
**Purpose:** Compile-time evaluation
```cpp
constexpr int factorial(int n) {
    return (n <= 1) ? 1 : n * factorial(n - 1);
}
constexpr int result = factorial(10);
```

### 5. `static_assert`
**Purpose:** Compile-time assertions
```cpp
static_assert(sizeof(int) == 4, "int must be 4 bytes");
static_assert(sizeof(long) >= sizeof(int), "long must be at least as large as int");
```

### 6. `noexcept`
**Purpose:** Exception specification
```cpp
void function() noexcept {
    // This function will not throw exceptions
}

void another() noexcept(true) {
    // Explicitly noexcept
}

void third() noexcept(false) {
    // May throw exceptions
}
```

### 7. `override`
**Purpose:** Explicit override specification
```cpp
class Base {
public:
    virtual void process() = 0;
    virtual void cleanup() {}
};

class Derived : public Base {
public:
    void process() override { /* ... */ }  // Compiler checks
    void cleanup() override { /* ... */ }  // Must match base signature
};
```

### 8. `final`
**Purpose:** Prevent inheritance or overriding
```cpp
class Base {
public:
    virtual void process() final { /* ... */ }  // Cannot override
};

class Derived final : public Base {  // Cannot inherit from Derived
    // void process() { }  // Error: cannot override final function
};

// class Further : public Derived { };  // Error: cannot inherit from final class
```

### 9. `char16_t`
**Purpose:** 16-bit character type
```cpp
char16_t c16 = u'字';
const char16_t* str16 = u"Unicode string";
```

### 10. `char32_t`
**Purpose:** 32-bit character type
```cpp
char32_t c32 = U'字';
const char32_t* str32 = U"Unicode string";
```

### 11. `thread_local`
**Purpose:** Thread-local storage
```cpp
thread_local int thread_id = 0;
thread_local std::string thread_name = "unknown";
```

## Existing Keywords with New Meanings/Usage in C++11

### 12. `using` (Type Aliases)
**Purpose:** Type alias declaration (alternative to typedef)
```cpp
using IntPtr = int*;
using StringVector = std::vector<std::string>;
using FunctionType = void(*)(int, double);

template<typename T>
using Vector = std::vector<T>;
Vector<int> numbers;  // std::vector<int>
```

### 13. `extern` (Template Instantiation)
**Purpose:** Explicit template instantiation declaration
```cpp
extern template class std::vector<int>;
extern template void std::sort<int*>(int*, int*);
```

### 14. `inline` (Namespace)
**Purpose:** Inline namespace
```cpp
inline namespace v1 {
    void function() { /* ... */ }
}

namespace v2 {
    void function() { /* ... */ }
}

// v1::function() is accessible as ::function()
```

## All C++ Keywords (Including C++11)

### Storage Class Specifiers
- `auto` (C++11: type deduction)
- `register` (deprecated in C++17)
- `static`
- `extern`
- `mutable`
- `thread_local` (C++11)

### Type Specifiers
- `void`
- `char`
- `wchar_t`
- `char16_t` (C++11)
- `char32_t` (C++11)
- `bool`
- `short`
- `int`
- `long`
- `float`
- `double`
- `signed`
- `unsigned`

### Type Qualifiers
- `const`
- `volatile`
- `constexpr` (C++11)

### Function Specifiers
- `inline`
- `virtual`
- `explicit`
- `override` (C++11)
- `final` (C++11)
- `noexcept` (C++11)

### Access Specifiers
- `public`
- `protected`
- `private`

### Class Specifiers
- `class`
- `struct`
- `union`
- `enum`

### Control Flow
- `if`
- `else`
- `switch`
- `case`
- `default`
- `for`
- `while`
- `do`
- `break`
- `continue`
- `goto`
- `return`

### Exception Handling
- `try`
- `catch`
- `throw`

### Namespace
- `namespace`
- `using`

### Template
- `template`
- `typename`
- `export` (deprecated)

### Other
- `typedef`
- `using` (C++11: type aliases)
- `friend`
- `operator`
- `this`
- `new`
- `delete`
- `true`
- `false`
- `nullptr` (C++11)
- `sizeof`
- `alignof` (C++11)
- `typeid`
- `decltype` (C++11)
- `static_assert` (C++11)
- `asm`
- `and`
- `and_eq`
- `bitand`
- `bitor`
- `compl`
- `not`
- `not_eq`
- `or`
- `or_eq`
- `xor`
- `xor_eq`

## C++11 Contextual Keywords

### 14. `= default`
**Purpose:** Request default implementation
```cpp
class Widget {
public:
    Widget() = default;
    Widget(const Widget&) = default;
    Widget& operator=(const Widget&) = default;
    ~Widget() = default;
};
```

### 15. `= delete`
**Purpose:** Delete function
```cpp
class Widget {
public:
    Widget() = default;
    Widget(const Widget&) = delete;  // Prevent copying
    Widget& operator=(const Widget&) = delete;
    void process(int) = delete;  // Prevent specific overload
};
```

### 16. `= 0` (Pure Virtual)
**Purpose:** Pure virtual function
```cpp
class Abstract {
public:
    virtual void process() = 0;  // Pure virtual
    virtual ~Abstract() = default;
};
```

## C++11 Literals

### 17. Raw String Literals
```cpp
const char* raw = R"(C:\path\to\file)";  // No escaping needed
const char* multiline = R"(
    Line 1
    Line 2
    Line 3
)";
```

### 18. Unicode String Literals
```cpp
const char* utf8 = u8"UTF-8 string";
const char16_t* utf16 = u"UTF-16 string";
const char32_t* utf32 = U"UTF-32 string";
const wchar_t* wide = L"Wide string";
```

### 19. User-Defined Literals
```cpp
constexpr long double operator"" _deg(long double deg) {
    return deg * 3.141592 / 180;
}

double angle = 90.0_deg;  // Convert degrees to radians
```

## C++11 Attributes

### 20. `[[noreturn]]`
**Purpose:** Function never returns
```cpp
[[noreturn]] void exit_program() {
    std::exit(1);
}
```

### 21. `[[carries_dependency]]`
**Purpose:** Dependency carrying
```cpp
[[carries_dependency]] int* get_pointer();
```

### 22. `[[deprecated]]`
**Purpose:** Mark as deprecated
```cpp
[[deprecated("Use new_function instead")]]
void old_function() { /* ... */ }
```

## Summary

### **New C++11 Keywords:**
1. `auto` - Type deduction
2. `decltype` - Type deduction from expression
3. `nullptr` - Null pointer literal
4. `constexpr` - Compile-time evaluation
5. `static_assert` - Compile-time assertions
6. `noexcept` - Exception specification
7. `override` - Explicit override
8. `final` - Prevent inheritance/overriding
9. `char16_t` - 16-bit character
10. `char32_t` - 32-bit character
11. `thread_local` - Thread-local storage
12. `alignof` - Alignment requirement
13. `using` - Type aliases (new meaning)

### **New C++11 Syntax:**
- `= default` - Request default implementation
- `= delete` - Delete function
- `= 0` - Pure virtual function
- Raw string literals `R"(...)"`
- Unicode string literals `u8"..."`, `u"..."`, `U"..."`
- User-defined literals
- Attributes `[[...]]`

### **Total Count:**
- **New Keywords:** 13
- **New Syntax Elements:** 8
- **Total C++11 Additions:** 21

This comprehensive list covers all keywords and syntax elements introduced or modified in C++11, providing a complete reference for modern C++ development! 