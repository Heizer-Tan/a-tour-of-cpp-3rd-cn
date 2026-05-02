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
