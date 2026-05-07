# 16.4 类型函数

**类型函数**是在编译时求值的函数，它以类型为参数或返回一个类型。标准库提供了多种类型函数，以帮助库实现者（以及一般程序员）编写能够利用语言、标准库和一般代码各方面特性的代码。

对于数值类型，`<limits>` 中的 `numeric_limits` 提供了各种有用的信息（§17.7）。例如：

```cpp
constexpr float min = numeric_limits<float>::min();   // 最小的正 float
```

类似地，对象大小可以通过内置的 `sizeof` 运算符获得（§1.4）。例如：

```cpp
constexpr int szi = sizeof(int);   // int 的字节数
```

在 `<type_traits>` 中，标准库提供了许多用于查询类型属性的函数。例如：

```cpp
bool b = is_arithmetic_v<X>;        // 如果 X 是（内置）算术类型之一，则为 true
using Res = invoke_result_t<decltype(f)>; // 如果 f 是返回 int 的函数，则 Res 是 int
```

`decltype(f)` 是调用内置类型函数 `decltype()`，它返回其参数声明的类型；这里参数是 `f`。

一些类型函数基于输入创建新的类型。例如：

```cpp
template<typename T>
using Store = conditional_t<(sizeof(T) < max), On_stack<T>, On_heap<T>>;
```

如果 `conditional_t` 的第一个（布尔）参数为真，则结果是第一个备选；否则是第二个。假设 `On_stack` 和 `On_heap` 提供相同的访问 `T` 的函数，它们可以按名字所示的方式分配其 `T`。因此，`Store<X>` 的用户可以根据 `X` 对象的大小进行调优。这种分配选择所启用的性能调优可能非常显著。

===== 第 9 页 =====

这是一个简单的例子，说明我们如何使用标准类型函数或概念来构造自己的类型函数。概念就是类型函数。当在表达式中使用时，它们特指**类型谓词**。例如：

```cpp
template<typename F, typename... Args>
auto call(F f, Args... a, Allocator alloc)
{
    if constexpr (invocable<F, alloc, Args...>)   // 需要分配器吗？
        return f(f, alloc, a...);
    else
        return f(f, a...);
}
```

在许多情况下，概念是最好的类型函数，但大多数标准库是在概念之前编写的，并且必须支持概念之前的代码库。

表示法约定令人困惑。标准库使用 `_v` 表示返回值类型函数，使用 `_t` 表示返回类型类型函数。这是 C 弱类型时代和概念前 C++ 的遗留物。没有标准库类型函数既返回类型又返回值，因此这些后缀是多余的。有了概念，无论是在标准库还是其他地方，都不需要或使用后缀。

类型函数是 C++ 编译时计算机制的一部分，它们允许比没有它们时更严格的类型检查和更好的性能。类型函数和概念（第 8 章，§14.5）的使用通常称为**元编程**，或者在涉及模板时称为**模板元编程**。

## 16.4.1 类型谓词

在 `<type_traits>` 中，标准库提供了几十种简单的类型函数，称为**类型谓词**，它们回答有关类型的基本问题。以下是其中一小部分：

**选定的类型谓词**（`T`、`A` 和 `U` 是类型；所有谓词返回 `bool`）

| 谓词 | 描述 |
|------|------|
| `is_void_v<T>` | `T` 是 `void` 吗？ |
| `is_integral_v<T>` | `T` 是整数类型吗？ |
| `is_floating_point_v<T>` | `T` 是浮点类型吗？ |
| `is_class_v<T>` | `T` 是类（且不是联合）吗？ |
| `is_function_v<T>` | `T` 是函数（而不是函数对象或指向函数的指针）吗？ |
| `is_arithmetic_v<T>` | `T` 是整数或浮点类型吗？ |
| `is_scalar_v<T>` | `T` 是算术、枚举、指针或指向成员的指针类型吗？ |
| `is_constructible_v<T, A...>` | `T` 能否从 `A...` 参数列表构造？ |
| `is_default_constructible_v<T>` | `T` 能否在没有显式参数的情况下构造？ |
| `is_copy_constructible_v<T>` | `T` 能否从另一个 `T` 构造？ |
| `is_move_constructible_v<T>` | `T` 能否被移动或拷贝到另一个 `T` 中？ |
| `is_assignable_v<T, U>` | `U` 能否赋值给 `T`？ |
| `is_trivially_copyable_v<T, U>` | `U` 能否在没有用户定义拷贝操作的情况下赋值给 `T`？ |
| `is_same_v<T, U>` | `T` 与 `U` 是同一类型吗？ |
| `is_base_of_v<T, U>` | `U` 是否派生自 `T`，或者 `U` 与 `T` 是同一类型？ |
| `is_convertible_v<T, U>` | `T` 能否隐式转换为 `U`？ |
| `is_iterator_v<T>` | `T` 是迭代器类型吗？ |
| `is_invocable_v<T, A...>` | `T` 能否用参数列表 `A...` 调用？ |
| `has_virtual_destructor_v<T>` | `T` 是否有虚析构函数？ |

这些谓词的一个传统用途是约束模板参数。例如：

```cpp
template<typename Scalar>
class complex {
    Scalar re, im;
public:
    static_assert(is_arithmetic_v<Scalar>, "Sorry, I support only complex of arithmetic types");
    // ...
};
```

===== 第 11 页 =====

然而，这——像其他传统用法一样——使用概念更简单、更优雅：

```cpp
template<Arithmetic Scalar>
class complex {
    Scalar re, im;
public:
    // ...
};
```

在许多情况下，诸如 `is_arithmetic` 之类的类型谓词会消失在概念的定义中，以更易于使用。例如：

```cpp
template<typename T>
concept Arithmetic = is_arithmetic_v<T>;
```

奇怪的是，标准库中没有 `std::arithmetic` 概念。

通常，我们可以定义比标准库类型谓词更通用的概念。许多标准库类型谓词仅适用于内置类型。我们可以根据所需的操作来定义概念，正如 `Number` 的定义所建议的那样（§8.2.4）：

```cpp
template<typename T, typename U = T>
concept Arithmetic = Number<T, U> && Number<U, T>;
```

大多数情况下，标准库类型谓词的使用出现在基础服务的实现深处，通常是为了区分不同的优化情况。例如，`std::copy(Iter, Iter, Iter2)` 的部分实现可以优化简单类型（如整数）的连续序列这一重要情况：

```cpp
template<class T>
void cpy1(T* first, T* last, T* target)
{
    if constexpr (is_trivially_copyable_v<T>)
        memcpy(target, first, (last - first) * sizeof(T));
    else
        while (first != last)
            *target++ = *first++;
}
```

在某些实现上，这个简单的优化比其非优化版本快了约 50%。除非你已经验证了标准库没有做得更好，否则不要沉迷于这种小聪明。手工优化的代码通常比更简单的替代方案更难维护。

## 16.4.2 条件属性

考虑定义一个“智能指针”：

```cpp
template<typename T>
class Smart_pointer {
    // ...
    T& operator*() const;
    T* operator->() const;   // -> 应该当且仅当 T 是类类型时才有效
};
```

`->` 应该当且仅当 `T` 是类类型时才定义。例如，`Smart_pointer<vector<T>>` 应该有 `->`，但 `Smart_pointer<int>` 不应该。

我们不能使用编译时 `if`，因为我们不在函数内部。相反，我们写：

```cpp
template<typename T>
class Smart_pointer {
    // ...
    T& operator*() const;
    T* operator->() const requires is_class_v<T>;   // -> 当且仅当 T 是类类型时定义
};
```

===== 第 13 页 =====

类型谓词直接表达了对 `operator->()` 的约束。我们也可以使用概念来实现这一点。标准库中没有要求一个类型是类类型（即类是 class、struct 或 union）的概念，但我们可以自己定义一个：

```cpp
template<typename T>
concept Class = is_class_v<T> || is_union_v<T>;   // union 也是类

template<typename T>
class Smart_pointer {
    // ...
    T& operator*() const;
    T* operator->() const requires Class<T>;   // -> 当且仅当 T 是类类型时定义
};
```

通常，概念比直接使用标准库类型谓词更通用，或者仅仅更合适。

## 16.4.3 类型生成器

许多类型函数返回类型，通常是它们计算出的新类型。我称这些函数为**类型生成器**，以区别于类型谓词。标准库提供了几个，例如：

**选定的类型生成器**

| 生成器 | 描述 |
|--------|------|
| `R = remove_const_t<T>` | `R` 是去除了顶层 `const`（如果有）的 `T` |
| `R = add_const_t<T>` | `R` 是 `const T` |
| `R = remove_reference_t<T>` | 如果 `T` 是引用 `U&`，则 `R` 是 `U`，否则 `R` 是 `T` |
| `R = add_lvalue_reference_t<T>` | 如果 `T` 是左值引用，则 `R` 是 `T`，否则 `R` 是 `T&` |
| `R = add_rvalue_reference_t<T>` | 如果 `T` 是右值引用，则 `R` 是 `T`，否则 `R` 是 `T&&` |
| `R = enable_if_t<b, T = void>` | 如果 `b` 为真，则 `R` 是 `T`，否则 `R` 未定义 |
| `R = conditional_t<b, T, U>` | 如果 `b` 为真，则 `R` 是 `T`，否则 `R` 是 `U` |
| `R = common_type_t<T...>` | 如果存在一个所有 `T` 都可以隐式转换到的类型，则 `R` 是该类型；否则 `R` 未定义 |
| `R = underlying_type_t<T>` | 如果 `T` 是枚举，则 `R` 是其底层类型；否则错误 |
| `R = invoke_result_t<T, A...>` | 如果 `T` 可以用参数 `A...` 调用，则 `R` 是其返回类型；否则错误 |

这些类型函数通常用于工具的实现中，而不是直接在应用程序代码中使用。其中，`enable_if` 可能是概念前代码中最常见的。例如，智能指针的条件启用 `->` 传统上是这样实现的：

```cpp
template<typename T>
class Smart_pointer {
    // ...
    T& operator*();
    enable_if<is_class_v<T>, T&> operator->();   // -> 当且仅当 T 是类类型时定义
};
```

我发现这不太容易阅读，更复杂的使用情况则更糟。`enable_if` 的定义依赖于一种称为 SFINAE（“替换失败不是错误”）的微妙语言特性。只有在需要时才去查阅它。
