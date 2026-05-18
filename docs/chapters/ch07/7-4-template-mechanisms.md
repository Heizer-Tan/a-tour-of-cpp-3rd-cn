# 7.4 模板机制

为了定义良好的模板，我们需要一些语言层面的配套机制：

- 依赖类型的取值：**变量模板**（[§7.4.1](7-4-template-mechanisms.md)）
- 为类型与模板起别名：**别名模板**（[§7.4.2](7-4-template-mechanisms.md)）
- 编译期分支：**`if constexpr`**（[§7.4.3](7-4-template-mechanisms.md)）
- 在编译期探查类型与表达式是否成立：**requires 表达式**（[§8.2.3](../ch08/8-2-concepts.md)）

此外，`constexpr` 函数（[§1.6](../ch01/1-6-constants.md)）和 `static_assert`（[§4.5.2](../ch04/4-5-assertions.md)）也常常参与模板的设计与使用。

这些基本机制主要是用于构建通用、基础抽象的工具。

## 7.4.1 变量模板

当我们使用一个类型时，我们通常需要该类型的常量和值。当我们使用类模板时当然也是如此：当我们定义 `C<T>` 时，我们通常需要类型 `C<T>` 和其他依赖于 `T` 的类型的常量和变量。这里有一个来自流体动力学仿真的例子 [Garcia,2015]：

```cpp
template <class T>
constexpr T viscosity = 0.4;

template <class T>
constexpr space_vector<T> external_acceleration = { T(), T{-9.8}, T() };

auto vis2 = 2 * viscosity<double>;
auto acc = external_acceleration<float>;
```

这里，`space_vector` 是一个三维向量。

奇怪的是，大多数变量模板似乎是常量。但话又说回来，许多变量也是如此。术语没有跟上我们关于不可变性的概念。

当然，我们可以使用合适类型的任意表达式作为初始化器。考虑：

```cpp
template<typename T, typename T2>
constexpr bool Assignable = is_assignable_v<T&, T2>;   // is_assignable_v 是标准类型谓词

template<typename T>
void testing()
{
    static_assert(Assignable<T&, double>, "can't assign a double to a T");
    static_assert(Assignable<T&, string>, "can't assign a string to a T");
}
```

几经演化，这一思路成为形式化**概念**的核心（[§8.2](../ch08/8-2-concepts.md)）。

标准库也通过变量模板提供诸如 `pi`、`log2e` 之类的数学常数（[§17.9](../ch17/17-9-math-constants.md)）。

## 7.4.2 别名

令人惊讶的是，为类型或模板引入同义词常常很有用。例如，标准头文件 `<cstddef>` 包含了别名 `size_t` 的定义，可能如下：

```cpp
using size_t = unsigned int;
```

名为 `size_t` 的实际类型是实现相关的，因此在另一种实现中 `size_t` 可能是 `unsigned long`。拥有别名 `size_t` 使得程序员能够编写可移植的代码。

参数化类型为其模板参数相关的类型提供别名非常常见。例如：

```cpp
template<typename T>
class Vector {
public:
    using value_type = T;
    // ...
};
```

事实上，每一种标准序列容器都把 `value_type` 用作元素类型的别名（参见[第十二章](../ch12/index.md)）。于是可以编写对所有遵循这一约定的容器都能工作的代码：

```cpp
template<typename C>
using Value_type = typename C::value_type;   // C 的元素类型

template<typename Container>
void algo(Container& c)
{
    Vector<Value_type<Container>> vec;       // 在此保存结果
    // ...
}
```

这个 `Value_type` 只是标准库中 `range_value_t`（[§16.4.4](../ch16/16-4-type-functions.md)）的极简示意；别名语法还能用来通过绑定若干模板实参得出“缩写模板”。

```cpp
template<typename Key, typename Value>
class Map {
    // ...
};

template<typename Value>
using String_map = Map<string, Value>;

String_map<int> m;   // m 是一个 Map<string, int>
```

## 7.4.3 编译时 if

考虑编写一个可以使用两个函数 `slow_and_safe(T)` 或 `simple_and_fast(T)` 之一实现的操作。这类问题在基础代码中比比皆是，在这些代码中，通用性和最佳性能至关重要。如果涉及类层次结构，基类可以提供 `slow_and_safe` 通用操作，派生类可以用 `simple_and_fast` 实现覆盖它。

或者，我们可以使用编译时 `if`：

```cpp
template<typename T>
void update(T& target)
{
    if constexpr (is_trivially_copyable_v<T>)
        simple_and_fast(target);   // 用于“普通旧数据”
    else
        slow_and_safe(target);     // 用于更复杂的类型
}
```

`is_trivially_copyable_v<T>` 是一条类型层面的谓词（[§16.4.1](../ch16/16-4-type-functions.md)），用来说明 `T` 是否可以按“平凡复制”的规则搬运。

编译器只检查 `if constexpr` 中被选中的分支。这个解决方案提供了最佳性能以及优化的局部性。

重要的是，`if constexpr` **不是**文本操作机制，也不能用于破坏语法、类型和作用域的常规规则。例如，这是一个试图有条件地将调用包装在 `try` 块中的幼稚且失败的尝试：

```cpp
template<typename T>
void bad(T arg)
{
    if constexpr (!is_trivially_copyable_v<T>)
        try {                         // 问题：`if constexpr` 无法覆盖整个 try/catch 结构体

            g(arg);

    if constexpr (!is_trivially_copyable_v<T>)
        catch (...) { /* ... */ }       // 语法错误
}
```

允许这样的文本操作会严重损害代码的可读性，并为依赖现代程序表示技术（如“抽象语法树”）的工具带来问题。

许多此类尝试的黑客技巧也是不必要的，因为存在不违反作用域规则的更干净的解决方案。例如：

```cpp
template<typename T>
void good(T arg)
{
    if constexpr (is_trivially_copyable_v<T>)
        g(arg);
    else
        try {
            g(arg);
        } catch (...) { /* ... */ }
}
```
