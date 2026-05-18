# 16.4 类型函数

类型函数是在编译期求值的函数：以类型为实参，或返回类型。标准库提供多种类型函数，帮助库实现者（以及广大程序员）写出能够利用语言特性、标准库以及一般代码方方面面的程序。

对算术类型，`<limits>` 中的 `numeric_limits` 提供诸多有用信息（§17.7）。例如：

```cpp
constexpr float min = numeric_limits<float>::min();   // 最小正 float
```

对象大小可通过内建运算符 `sizeof`（§1.4）取得。例如：

```cpp
constexpr int szi = sizeof(int);   // 一个 int 占用的字节数
```

在 `<type_traits>` 中，标准库提供了大量用于查询类型性质的函数。例如：

```cpp
bool b = is_arithmetic_v<X>;                        // 若 X 为（内建）算术类型则为 true
using Res = invoke_result_t<decltype(f)>;           // 若 f 是返回 int 的函数，则 Res 为 int
```

`decltype(f)` 是对内建类型函数 `decltype()` 的调用，返回其参数的声明类型；此处为 `f`。

有些类型函数会根据输入构造新类型。例如：

```cpp
template<typename T>
using Store = conditional_t<sizeof(T) < max, On_stack<T>, On_heap<T>>;
```

若 `conditional_t` 的第一个（布尔）实参为 `true`，结果就是第一个备选类型；否则为第二个。假定 `On_stack` 与 `On_heap` 对 `T` 提供相同的访问接口，它们可按名称所示分配其 `T`。于是，`Store<X>` 的用户能依据 `X` 对象的大小调节策略；由此带来的分配策略性能调优可能非常重要。这是我们可以用标准类型函数或借助 concept 构造自有类型函数的简单示例。

concept 也是类型函数。用在表达式里时，它们具体化为类型谓词。例如：

```cpp
template<typename F, typename... Args>
auto call(F f, Args... a, Allocator alloc)
{
    if constexpr (invocable<F, Allocator, Args...>) // 需要分配器吗？
        return f(f, alloc, a...);
    else
        return f(f, a...);
}
```

在许多情形下，concept 是最好的类型函数；但标准库的大多数代码写在 concept 之前，还必须支持尚未使用 concept 的代码库。

记法约定容易令人困惑。标准库用后缀 `_v` 表示返回值的类型函数，用 `_t` 表示返回类型的类型函数。这是 C 以及 concept 之前 C++ 弱类型时代的遗存。没有任何标准库类型函数既返回类型又返回值，因此这些后缀其实是冗余的。有了 concept——无论是在标准库还是别处——都不需要也不使用后缀。

类型函数属于 C++ 编译期计算机制的一部分；相较没有它们的情形，能实现更严格的类型检查和更好的性能。类型函数与 concept（第 8 章，§14.5）的使用常被称为元编程，或（当涉及模板时）模板元编程。

## 16.4.1

在 `<type_traits>` 中，标准库提供数十个简单的类型函数，称为**类型谓词**，回答关于类型的基本问题。下面是一小部分：

**所选类型谓词**（`T`、`A`、`U` 为类型；谓词均返回 `bool`）

| 谓词 | 含义 |
|------|------|
| `is_void_v<T>` | `T` 是否为 `void`？ |
| `is_integral_v<T>` | `T` 是否为整型？ |
| `is_floating_point_v<T>` | `T` 是否为浮点类型？ |
| `is_class_v<T>` | `T` 是否为类（且非联合体）？ |
| `is_function_v<T>` | `T` 是否为函数（而非函数对象或函数指针）？ |
| `is_arithmetic_v<T>` | `T` 是否为整型或浮点类型？ |
| `is_scalar_v<T>` | `T` 是否为算术、枚举、指针或成员指针类型？ |
| `is_constructible_v<T, A...>` | 能否用实参列表 `A...` 构造 `T`？ |
| `is_default_constructible_v<T>` | 能否无显式实参构造 `T`？ |
| `is_copy_constructible_v<T>` | 能否用另一个 `T` 构造 `T`？ |
| `is_move_constructible_v<T>` | 能否将 `T` 移动或拷贝到另一个 `T`？ |
| `is_assignable_v<T, U>` | 能否把 `U` 赋给 `T`？ |
| `is_trivially_copyable_v<T>` | `T` 是否可无用户自定义拷贝操作地平凡可复制？ |
| `is_same_v<T, U>` | `T` 与 `U` 是否同一类型？ |
| `is_base_of_v<T, U>` | `U` 是否派生自 `T`，或 `U` 与 `T` 相同？ |
| `is_convertible_v<T, U>` | `T` 能否隐式转换为 `U`？ |
| `is_iterator_v<T>` | `T` 是否为迭代器类型？ |
| `is_invocable_v<T, A...>` | 能否用实参列表 `A...` 调用 `T`？ |
| `has_virtual_destructor_v<T>` | `T` 是否有虚析构函数？ |

这些谓词的一种传统用途是约束模板实参。例如：

```cpp
template<typename Scalar>
class complex {
    Scalar re, im;
public:
    static_assert(is_arithmetic_v<Scalar>, "Sorry, I support only complex of arithmetic types.");
    // ...
};
```

然而这种做法——与其他许多传统用法一样——用 concept 来做会更轻松、更利落：

```cpp
template<Arithmetic Scalar>
class complex {
    Scalar re, im;
public:
    // ...
};
```

许多场合下，诸如 `is_arithmetic` 的类型谓词会消融在 concept 的定义里，更易使用。例如：

```cpp
template<typename T>
concept Arithmetic = is_arithmetic_v<T>;
```

有趣的是，标准库里并没有名为 `std::arithmetic` 的 concept。

我们还可以定义比标准库类型谓词更一般的 concept。许多标准谓词只适用于内建类型。我们能按所需运算来定义 concept，正如 `Number`（§8.2.4）的定义所示：

```cpp
template<typename T, typename U = T>
concept Arithmetic = Number<T, U> && Number<U, T>;
```

标准库类型谓词的用法最常见于基础服务的实现深处，往往是为了区分情形以做优化。例如，`std::copy(Iter, Iter, Iter2)` 的实现可能对「简单类型的连续序列」（例如整数）这一重要情形专门优化：

```cpp
template<class T>
void cpy1(T* first, T* last, T* target)
{
    if constexpr (is_trivially_copyable_v<T>)
        memcpy(target, first, (last - first) * sizeof(T));   // 注意：示意 memcpy 参数顺序
    else
        while (first != last)
            *target++ = *first++;
}
```

这类简单优化在某些实现上可比未优化版本快大约一半。**除非**你已核实标准库没有做得更好，否则不要沉迷于这类技巧。手工优化的代码通常不如更简单替代品易于维护。

## 16.4.2

考虑定义「智能指针」：

```cpp
template<typename T>
class Smart_pointer {
    // ...
    T& operator*() const;
    T* operator->() const;     // 当且仅当 T 为类类型时才应有 ->
};
```

`operator->` 当且仅当 `T` 为类类型时才应定义。例如，`Smart_pointer<vector<int>>` 应有 `->`，但 `Smart_pointer<int>` 不应有。

我们不能使用编译期 `if`，因为不在函数体内。应写成：

```cpp
template<typename T>
class Smart_pointer {
    // ...
    T& operator*() const;
    T* operator->() const requires is_class_v<T>;   // 当且仅当……时定义 ->
};
```

类型谓词直接表达了对 `operator->()` 的约束。我们也可以用 concept。标准库没有「必须是类类型」的 concept（即类、`struct` 或联合体），但可以自定义：

```cpp
template<typename T>
concept Class = is_class_v<T> || is_union_v<T>;   // 联合体也算一类 class-key

template<typename T>
class Smart_pointer {
    // ...
    T& operator*() const;
    T* operator->() const requires Class<T>;      // 当且仅当 T 为类类型……
};
```

常常 concept 比直接使用标准库类型谓词更一般，或单纯更合适。

## 16.4.3

许多类型函数返回类型——常常是据此计算出的新类型。我把这类函数称为**类型生成器**，以区别于类型谓词。标准库提供的一部分如下：

**所选类型生成器**

| 记号 | 含义 |
|------|------|
| `R = remove_const_t<T>` | `R` 为去掉最外层 `const`（若有）后的 `T` |
| `R = add_const_t<T>` | `R` 为 `const T` |
| `R = remove_reference_t<T>` | 若 `T` 为引用 `U&`，则 `R` 为 `U`，否则为 `T` |
| `R = add_lvalue_reference_t<T>` | 若 `T` 已是左值引用则 `R` 为 `T`，否则为 `T&` |
| `R = add_rvalue_reference_t<T>` | 若 `T` 已是右值引用则 `R` 为 `T`，否则为 `T&&` |
| `R = enable_if_t<b, T = void>` | 若 `b` 为 `true` 则 `R` 为 `T`，否则 `R` 未定义 |
| `R = conditional_t<b, T, U>` | 若 `b` 为 `true` 则 `R` 为 `T`，否则为 `U` |
| `R = common_type_t<T...>` | 若诸 `T` 可隐式转换到同一类型，则 `R` 为该类型；否则未定义 |
| `R = underlying_type_t<T>` | 若 `T` 为枚举，则 `R` 为其底层类型；否则错误 |
| `R = invoke_result_t<T, A...>` | 若可用 `A...` 调用 `T`，则 `R` 为其返回类型；否则错误 |

这些类型函数通常用在工具设施实现里，而不是直接出现在应用代码中。其中 `enable_if` 大概是 concept 之前代码里最常用的。例如，智能指针上有条件启用的 `->` 传统上会像这样实现：

```cpp
template<typename T>
class Smart_pointer {
    // ...
    T& operator*();
    enable_if_t<is_class_v<T>, T&> operator->();   // 当且仅当 T 为类类型……
};
```

我觉得这并不好读，更复杂的用法糟得多。`enable_if` 的定义依赖一门精微的语言特性，称为 SFINAE（替换失败不是错误）。只有在你**确实需要**时再去查阅它。

## 16.4.4

所有标准容器（§12.8）以及遵循其模式的容器都有若干关联类型，例如值类型与迭代器类型。在 `<iterator>` 与 `<ranges>` 中，标准库为这些类型提供了名字：

| 名字 | 含义 |
|------|------|
| `range_value_t<R>` | 范围 `R` 的元素类型 |
| `iter_value_t<T>` | 迭代器 `T` 所指元素的类型 |
| `iterator_t<R>` | 范围 `R` 的迭代器类型 |
