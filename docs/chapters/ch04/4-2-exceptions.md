## 4.2 异常

异常（exception）是 C++ 中处理运行时错误的主要机制。它的核心思想是将错误的*检测*（detection）与错误的*处理*（handling）分离开来。

### 4.2.1 抛出和捕获

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

### 4.2.2 标准异常层次结构

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

### 4.2.3 资源管理

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
