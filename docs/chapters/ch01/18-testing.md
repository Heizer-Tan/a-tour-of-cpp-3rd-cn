# 1.8 测试

程序必须经过测试才能被信任。测试可以分为多种形式：单元测试（unit test）、集成测试（integration test）、系统测试（system test）和回归测试（regression test）等。

C++ 没有内置的测试框架，但有许多优秀的第三方测试框架可供选择，如 Catch2、Google Test 和 Boost.Test 等。这里我们展示一种简单的测试方法，使用 `assert` 进行基本的检查。

## 1.8.1 断言

C++ 提供了 `assert` 宏，用于在调试时检查条件：

```cpp
#include <cassert>

double square(double x)
{
    return x * x;
}

int main()
{
    assert(square(2) == 4);
    assert(square(-3) == 9);
    assert(square(0) == 0);
}
```

如果 `assert` 的条件为 `false`，程序会终止并输出诊断信息。`assert` 在定义了 `NDEBUG` 宏时会被禁用，因此它主要用于开发和调试阶段，不应依赖它进行运行时的错误处理。

## 1.8.2 静态断言

`static_assert` 在编译时检查条件，如果条件为 `false`，编译会失败并输出指定的错误消息：

```cpp
static_assert(sizeof(int) >= 4, "int 必须至少为 4 字节");
```

`static_assert` 对于检查类型大小、常量表达式和模板参数非常有用。它不会产生任何运行时代码。

## 1.8.3 结构化测试

对于更系统的测试，通常将测试组织为独立的函数：

```cpp
void test_square()
{
    assert(square(2) == 4);
    assert(square(-3) == 9);
    assert(square(0) == 0);
    assert(square(1.5) == 2.25);
}

void test_square_edge_cases()
{
    assert(square(1e100) == 1e200);  // 可能溢出
}

int main()
{
    test_square();
    test_square_edge_cases();
    std::cout << "所有测试通过！\n";
}
```

良好的测试实践包括：
- 测试正常情况（典型输入）
- 测试边界情况（极端值、零、空输入）
- 测试错误情况（无效输入）
- 保持测试简单、独立和可重复

对于大型项目，建议使用专业的测试框架，它们提供了更好的测试组织、报告和自动化支持。
