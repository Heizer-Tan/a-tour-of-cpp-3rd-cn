# 错误处理

> **程序不仅要能处理正确的情况，更要能优雅地应对错误。**
>
> —— Bjarne Stroustrup

## 4.1 引言

错误处理是编写健壮软件的关键部分。C++ 提供了多种机制来处理错误，从简单的断言到结构化的异常处理。选择哪种机制取决于错误的性质以及程序需要如何响应。

常见的错误处理策略包括：

- **异常**（exceptions）：用于处理运行时错误，将错误检测与错误处理分离
- **不变式**（invariants）：在设计和实现中确保对象始终处于有效状态
- **断言**（assertions）：在开发和调试阶段捕获逻辑错误
- **错误码**（error codes）：通过返回值来指示操作是否成功

本章将逐一介绍这些机制，并讨论何时使用哪种策略。

# 4.2 异常

异常（exception）是 C++ 中处理运行时错误的主要机制。它的核心思想是将错误的*检测*（detection）与错误的*处理*（handling）分离开来。

## 4.2.1 抛出和捕获

当函数检测到一个无法自行处理的问题时，它可以*抛出*（throw）一个异常：

```cpp
class Vector {
public:
    Vector(int s) {
        if (s < 0)
            throw std::length_error{"Vector size must be non-negative"};
        elem = new double[s];
        sz = s;
    }
    // ...
private:
    double* elem;
    int sz;
};
```

调用方可以使用 `try`-`catch` 块来*捕获*（catch）并处理异常：

```cpp
void test()
{
    try {
        Vector v(-27);
    }
    catch (std::length_error& e) {
        std::cerr << "Error: " << e.what() << '\n';
        // 处理负大小的问题
    }
    catch (std::bad_alloc& e) {
        std::cerr << "Memory exhausted\n";
        // 处理内存耗尽的问题
    }
}
```

异常处理机制会沿着调用栈向上查找匹配的 `catch` 子句。这个过程称为*栈展开*（stack unwinding）——在展开过程中，局部对象的析构函数会被正确调用，确保资源得到释放。

## 4.2.2 标准异常层次结构

标准库定义了一套异常层次结构，以 `std::exception` 为根：

```
std::exception
├── std::logic_error          // 逻辑错误（可预防的）
│   ├── std::length_error
│   ├── std::domain_error
│   ├── std::invalid_argument
│   └── std::out_of_range
└── std::runtime_error        // 运行时错误（难以预防的）
    ├── std::range_error
    ├── std::overflow_error
    └── std::underflow_error
```

- `logic_error` 及其子类用于表示可以通过更仔细的编程来避免的错误
- `runtime_error` 及其子类用于表示依赖于运行时条件的错误

## 4.2.3 资源管理

异常安全（exception safety）是 C++ 资源管理的核心概念。基本保证是：当异常被抛出时，程序不会泄漏资源。这通常通过 RAII（Resource Acquisition Is Initialization，资源获取即初始化）技术来实现（[§6.3](../ch06/6-3-resource-mgmt.md)）。

```cpp
void f(const string& name)
{
    std::ifstream file {name};  // 打开文件
    // 使用文件...
    // 当 file 离开作用域时，文件自动关闭——即使发生异常
}
```

`std::ifstream` 的析构函数会自动关闭文件，因此无论函数是正常返回还是因异常退出，资源都会被正确释放。

## 4.3 不变式

*不变式*（invariant）是一个关于对象状态的逻辑条件，在对象的整个生命周期中（除了短暂的内部操作期间），该条件必须始终为真。不变式是设计健壮类的基础。

例如，对于我们的 `Vector` 类，不变式包括：

- `elem` 指向一个包含 `sz` 个 `double` 的数组（当 `sz > 0` 时）
- `sz >= 0`

构造函数负责建立不变式：

```cpp
Vector::Vector(int s)
{
    if (s < 0)
        throw std::length_error{"Vector size must be non-negative"};
    elem = new double[s];
    sz = s;
}
```

每个成员函数在进入时假定不变式成立，在退出时必须确保不变式仍然成立：

```cpp
double& Vector::operator[](int i)
{
    if (i < 0 || size() <= i)
        throw std::out_of_range{"Vector::operator[]"};
    return elem[i];
}
```

通过维护不变式，我们可以大大简化代码的推理过程——我们总是知道对象处于有效状态。

RAII（[§6.3](../ch06/6-3-resource-mgmt.md)）是维护不变式的关键技术：构造函数获取资源并建立不变式，析构函数释放资源。这样，即使发生异常，不变式也不会被破坏。

## 4.4 错误处理替代方案

异常并非处理错误的唯一方式。在某些场景下，其他策略可能更合适。

### 4.4.1 错误码

传统的错误处理方式是通过返回值来指示成功或失败：

```cpp
enum class Error_code { success, negative_size, out_of_memory };

Error_code vector_init(Vector& v, int s)
{
    if (s < 0)
        return Error_code::negative_size;
    v.elem = new(std::nothrow) double[s];
    if (!v.elem)
        return Error_code::out_of_memory;
    v.sz = s;
    return Error_code::success;
}
```

错误码的优点是显式且可预测，但缺点是容易忽略检查，且会使代码中充斥着错误检查逻辑。

### 4.4.2 `std::optional`

C++17 引入了 `std::optional<T>`，用于表示"可能存在也可能不存在"的值：

```cpp
std::optional<int> to_int(const string& s)
{
    try {
        return std::stoi(s);
    }
    catch (...) {
        return {};  // 返回空的 optional
    }
}

if (auto i = to_int("123")) {
    std::cout << "Got integer: " << *i << '\n';
} else {
    std::cout << "Not an integer\n";
}
```

### 4.4.3 `std::expected`（C++23）

`std::expected<T, E>` 是 C++23 中引入的类型，它要么包含一个期望的值 `T`，要么包含一个错误 `E`。这结合了异常和错误码的优点：

```cpp
std::expected<double, string> compute(double x)
{
    if (x < 0)
        return std::unexpected{"negative input"};
    return std::sqrt(x);
}
```

### 4.4.4 何时使用异常

一般来说：

- 当错误是"异常"的（不常发生）且调用者通常无法立即处理时，使用异常
- 当错误是"预期"的（经常发生）且调用者需要立即处理时，使用错误码或 `optional`
- 在构造函数和运算符重载中，异常通常是唯一可行的错误报告方式

## 4.5 断言

*断言*（assertion）是一种在开发和调试阶段捕获逻辑错误的机制。C++ 提供了多种断言方式。

### 4.5.1 `assert` 宏

传统的 C 风格断言：

```cpp
#include <cassert>

void f(int* p)
{
    assert(p != nullptr);   // 如果 p 为 null，程序终止
    // ...
}
```

`assert` 在 `NDEBUG` 宏被定义时（通常在发布构建中）会被禁用，因此不应依赖它来进行运行时错误检查。

### 4.5.2 `static_assert`

`static_assert` 在编译时检查条件：

```cpp
static_assert(sizeof(int) >= 4, "int must be at least 4 bytes");
```

如果条件为假，编译器会产生一条包含指定消息的错误。`static_assert` 是泛型编程中非常有用的工具（[§8.2](../ch08/8-2-concepts.md)）。

### 4.5.3 `noexcept`

`noexcept` 说明符用于声明一个函数不会抛出异常：

```cpp
void f() noexcept;   // f 保证不抛出异常
```

如果 `noexcept` 函数实际抛出了异常，程序将调用 `std::terminate()`。`noexcept` 对于性能优化和编写异常安全的代码非常重要（[§6.2.2](../ch06/6-2-copy-move.md)）。

# 4.6 建议

[1] 抛出异常以表明你无法完成所分配的任务；§4.4；[CG: E.2]。
[2] 仅将异常用于错误处理；§4.4；[CG: E.3]。
[3] 无法打开文件或到达迭代末尾是预期事件，而非异常；§4.4。
[4] 当预期直接调用者会处理错误时，使用错误码；§4.4。
[5] 对于预期会向上传播穿过许多函数调用的错误，抛出异常；§4.4。
[6] 如果不确定使用异常还是错误码，优先选择异常；§4.4。
[7] 在设计早期制定错误处理策略；§4.4；[CG: E.12]。
[8] 使用专门设计的用户定义类型作为异常（而非内置类型）；§4.2。
[9] 不要试图在每个函数中捕获每个异常；§4.4；[CG: E.7]。
[10] 你不必使用标准库异常类层次结构；§4.3。
[11] 优先使用 RAII 而非显式 try 块；§4.2，§4.3；[CG: E.6]。
[12] 让构造函数建立不变式，如果无法建立则抛出异常；§4.3；[CG: E.5]。
[13] 围绕不变式设计你的错误处理策略；§4.3；[CG: E.4]。
[14] 能在编译时检查的内容，通常最好在编译时检查；§4.5.2；[CG: P.4] [CG: P.5]。
[15] 使用断言机制为失败的含义提供单一控制点；§4.5。
[16] 概念（§8.2）是编译时谓词，因此在断言中常常有用；§4.5.2。
[17] 如果函数可能不会抛出异常，将其声明为 `noexcept`；§4.4；[CG: E.12]。
[18] 不要盲目使用 `noexcept`；§4.5.3。
