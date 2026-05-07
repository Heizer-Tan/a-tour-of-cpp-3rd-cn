
标准库提供了许多有用的概念：

- 定义类型属性的概念（§14.5.1）
- 定义迭代器的概念（§14.5.2）
- 定义范围的概念（§14.5.3）

### 14.5.1 类型概念

与类型属性和类型间关系相关的概念反映了类型的多样性。这些概念有助于简化大多数模板。

**核心语言概念 `<concepts>`**（`T` 和 `U` 是类型）

| 概念 | 描述 |
|------|------|
| `same_as<T, U>` | `T` 与 `U` 相同 |
| `derived_from<T, U>` | `T` 派生自 `U` |
| `convertible_to<T, U>` | `T` 可以转换为 `U` |
| `common_reference_with<T, U>` | `T` 和 `U` 共享一个公共引用类型 |
| `common_with<T, U>` | `T` 和 `U` 共享一个公共类型 |
| `integral<T>` | `T` 是整数类型 |
| `signed_integral<T>` | `T` 是有符号整数类型 |
| `unsigned_integral<T>` | `T` 是无符号整数类型 |
| `floating_point<T>` | `T` 是浮点类型 |
| `assignable_from<T, U>` | `U` 可以赋值给 `T` |
| `swappable_with<T, U>` | `T` 可以与 `U` 交换 |
| `swappable<T>` | `swappable_with<T, T>` |

许多算法应该能够处理相关类型的组合，例如混合了 `int` 和 `double` 的表达式。我们使用 `common_with` 来判断这样的混合在数学上是否合理。如果 `common_with<X, Y>` 为真，我们可以使用 `common_type_t<X, Y>` 来比较 `X` 和 `Y`，方法是先将两者都转换为 `common_type_t<X, Y>`。例如：

```cpp
common_type<string, const char*> s1 = some_fct();
common_type<string, const char*> s2 = some_other_fct();

if (s1 < s2) {
    // ...
}
```

要为一对类型指定公共类型，我们需要特化在 `common` 定义中使用的 `common_type_t`。例如：

```cpp
using common_type_t<Bigint, long> = Bigint;   // 为合适的 Bigint 定义
```

幸运的是，除非我们希望对库（尚）没有合适定义的类型混合进行操作，否则我们不需要定义 `common_type_t` 特化。

与比较相关的概念受到 [Stepanov,2009] 的强烈影响。

**比较概念 `<concepts>`**

| 概念 | 描述 |
|------|------|
| `equality_comparable_with<T, U>` | 可以使用 `==` 比较 `T` 和 `U` 是否等价 |
| `equality_comparable<T>` | `equality_comparable_with<T, T>` |
| `totally_ordered_with<T, U>` | 可以使用 `<`, `<=`, `>`, `>=` 比较 `T` 和 `U`，产生全序 |
| `totally_ordered<T>` | `totally_ordered_with<T, T>` |
| `three_way_comparable_with<T, U>` | 可以使用 `<=>` 比较 `T` 和 `U`，产生一致的结果 |
| `three_way_comparable<T>` | `three_way_comparable_with<T, T>` |

同时使用 `equality_comparable_with` 和 `equality_comparable` 显示了一个（迄今为止）错失的重载概念的机会。

奇怪的是，标准中没有 `boolean` 概念。我经常需要它，因此这里是一个版本：

```cpp
template<typename B>
concept Boolean =
    requires(B x, B y) {
        { x = true };
        { x = false };
        { x = (x == y) };
        { x = (x != y) };
        { x = !x };
        { x = (x = y) };
    };
```

在编写模板时，我们经常需要对类型进行分类。

**对象概念 `<concepts>`**

| 概念 | 描述 |
|------|------|
| `destructible<T>` | `T` 可以被销毁，并且可以使用一元 `&` 取其地址 |
| `constructible_from<T, Args>` | `T` 可以从 `Args` 类型的参数列表构造 |
| `default_initializable<T>` | `T` 可以默认构造 |
| `move_constructible<T>` | `T` 可以移动构造 |
| `copy_constructible<T>` | `T` 可以拷贝构造和移动构造 |
| `movable<T>` | `move_constructible<T>`, `assignable_from<T&, T>`, 和 `swappable<T>` |
| `copyable<T>` | `copy_constructible<T>`, `movable<T>`, 和 `assignable_from<T&, const T&>` |
| `semiregular<T>` | `copyable<T>` 和 `default_initializable<T>` |
| `regular<T>` | `semiregular<T>` 和 `equality_comparable<T>` |

类型的理想状态是 `regular`。一个 regular 类型的行为大致像 `int`，简化了我们关于如何使用一个类型的许多思考（§8.2）。类缺少默认的 `==` 意味着大多数类开始时是 `semiregular`，尽管大多数类可以而且应该是 regular 的。

每当我们传递一个操作作为受约束的模板参数时，我们需要指定它如何被调用，有时还需要指定我们对其语义的假设。

**可调用概念 `<concepts>`**

| 概念 | 描述 |
|------|------|
| `invocable<F, Args>` | `F` 可以使用 `Args` 类型的参数列表调用 |
| `regular_invocable<F, Args>` | `invocable<F, Args>` 并且是保等性的 |
| `predicate<F, Args>` | 返回 `bool` 的 `regular_invocable<F, Args>` |
| `relation<F, T, U>` | `predicate<F, T, U>` |
| `equivalence_relation<F, T, U>` | 提供等价关系的 `relation<F, T, U>` |
| `strict_weak_order<F, T, U>` | 提供严格弱序的 `relation<F, T, U>` |

如果 `x == y` 蕴含 `f(x) == f(y)`，则函数 `f()` 是保等性的。`invocable` 和 `regular_invocable` 仅在语义上不同。（目前）我们无法在代码中表示这一点，因此这些名称仅仅表达了我们的意图。类似地，`relation` 和 `equivalence_relation` 也仅在语义上不同。等价关系是自反、对称和传递的。`relation` 和 `strict_weak_order` 也仅在语义上不同。严格弱序是标准库通常对比较（如 `<`）所假设的。

### 14.5.2 迭代器概念

传统的标准算法通过迭代器访问其数据，因此我们需要概念来对迭代器类型的属性进行分类。

**迭代器概念 `<iterator>`**

| 概念 | 描述 |
|------|------|
| `input_or_output_iterator<I>` | `I` 可以递增（`++`）和解引用（`*`） |
| `sentinel_for<S, I>` | `S` 是迭代器类型 `I` 的一个哨兵；即 `S` 是 `I` 的值类型上的一个谓词 |
| `sized_sentinel_for<S, I>` | 可以应用 `-` 运算符的哨兵 `S` |
| `input_iterator<I>` | `I` 是一个输入迭代器；`*` 只能用于读取 |
| `output_iterator<I>` | `I` 是一个输出迭代器；`*` 只能用于写入 |
| `forward_iterator<I>` | `I` 是一个前向迭代器，支持多趟遍历和 `==` |
| `bidirectional_iterator<I>` | 支持 `--` 的 `forward_iterator<I>` |
| `random_access_iterator<I>` | 支持 `+`, `-`, `+=`, `-=` 和 `[]` 的 `bidirectional_iterator<I>` |
| `contiguous_iterator<I>` | 用于连续内存中元素的 `random_access_iterator<I>` |
| `permutable<I>` | 支持移动和交换的 `forward_iterator<I>` |
| `mergeable<I1, I2, R, O>` | 能否使用 `relation<R>` 将 `I1` 和 `I2` 定义的已排序序列合并到 `O`？ |
| `sortable<I>` | 能否使用 `less<>` 对 `I` 定义的序列进行排序？ |
| `sortable<I, R>` | 能否使用 `relation<R>` 对 `I` 定义的序列进行排序？ |

这里对 `mergeable` 和 `sortable` 进行了简化，相对于它们在 C++20 中的定义。

不同类型的（类别）迭代器用于为给定的参数集选择最佳算法；见 §8.2.2 和 §16.4.1。有关 `input_iterator` 的示例，请参见 §13.3.1。

哨兵的基本思想是，我们可以从迭代器开始遍历一个范围，直到谓词对某个元素变为真。这样，迭代器 `p` 和哨兵 `s` 定义了一个范围 `[p : s(*p))`。例如，我们可以定义一个哨兵的谓词，使用指针作为迭代器来遍历 C 风格字符串。不幸的是，这需要一些样板代码，因为这个想法是将谓词呈现为不能被误认为是普通迭代器的东西，但你可以将用于遍历范围的迭代器与它进行比较：

```cpp
template<class Iter>
class Sentinel {
public:
    Sentinel(int ee) : end(ee) { }
    Sentinel() : end(0) {}   // 概念 sentinel_for 要求一个默认构造函数

    friend bool operator==(const Iter& p, Sentinel s) { return *p == s.end; }
    friend bool operator!=(const Iter& p, Sentinel s) { return !(p == s); }
private:
    iter_value_t<const char*> end;   // 哨兵值
};
```

`friend` 声明允许我们在类的作用域内定义用于比较迭代器和哨兵的 `==` 和 `!=` 二元函数。

我们可以检查这个 `Sentinel` 是否满足 `sentinel_for` 对 `const char*` 的要求：

```cpp
static_assert(sentinel_for<Sentinel<const char*>, const char*>);   // 检查 Sentinel
```

最后，我们可以编写一个相当奇特的“Hello, World!”程序版本：

```cpp
const char aa[] = "Hello, World!\nBye for now\n";

ranges::for_each(aa, Sentinel<const char*>('\n'), [](const char x) { cout << x; });
```

是的，这确实写出了 `Hello, World!`，但没有跟随换行符。

### 14.5.3 范围概念

范围概念定义了范围的属性。

**范围概念 `<ranges>`**

| 概念 | 描述 |
|------|------|
| `range<R>` | `R` 是一个具有开始迭代器和哨兵的范围 |
| `sized_range<R>` | `R` 是一个能在常数时间内知道其大小的范围 |
| `view<R>` | `R` 是一个具有常数时间拷贝、移动和赋值操作的范围 |
| `common_range<R>` | `R` 是一个迭代器和哨兵类型相同的范围 |
| `input_range<R>` | `R` 是一个迭代器类型满足 `input_iterator` 的范围 |
| `output_range<R>` | `R` 是一个迭代器类型满足 `output_iterator` 的范围 |
| `forward_range<R>` | `R` 是一个迭代器类型满足 `forward_iterator` 的范围 |
| `bidirectional_range<R>` | `R` 是一个迭代器类型满足 `bidirectional_iterator` 的范围 |
| `random_access_range<R>` | `R` 是一个迭代器类型满足 `random_access_iterator` 的范围 |
| `contiguous_range<R>` | `R` 是一个迭代器类型满足 `contiguous_iterator` 的范围 |

`<ranges>` 中还有几个概念，但这一组是一个很好的开始。这些概念的主要用途是根据输入的类型的属性来启用实现的重载（§8.2.2）。
