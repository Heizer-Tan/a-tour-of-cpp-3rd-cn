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
