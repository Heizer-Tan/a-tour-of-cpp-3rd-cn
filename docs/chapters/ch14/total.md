===== 第 1 页 =====

14

# 范围

再有力的论证，只要结论未经经验验证，就毫无说服力。 – 罗杰·培根

- 引言
- 视图
- 生成器
- 管道
- 概念概览

===== 第 2 页 =====

### 14.1 引言

标准库提供的算法既有使用概念约束的版本（第 8 章），也有无约束的版本（出于兼容性考虑）。受约束的（使用概念的）版本位于 `<ranges>` 中，在命名空间 `ranges` 下。自然，我更喜欢使用概念的版本。**范围**（range）是对 C++98 中由 `{begin(), end()}` 对定义的序列的泛化；它规定了成为一个元素序列需要满足的条件。一个范围可以由以下方式定义：

- 一对迭代器 `{begin, end}`
- 一对 `{begin, n}`，其中 `begin` 是迭代器，`n` 是元素个数
- 一对 `{begin, pred}`，其中 `begin` 是迭代器，`pred` 是一个谓词；如果对迭代器 `p` 有 `pred(p)` 为真，则表示已到达范围的末尾。这允许我们拥有无限范围以及“按需生成”的范围（§14.3）。

正是这个 range 概念让我们可以写 `sort(v)` 而不是自 1994 年以来使用 STL 时必须写的 `sort(v.begin(), v.end())`。对于自己的算法，我们也可以类似地处理：

```cpp
template<forward_range R>
    requires sortable<iterator_t<R>>
void my_sort(R& r)   // 现代的、使用概念约束的 my_sort 版本
{
    return my_sort(r.begin(), end());   // 使用 1994 风格的 sort
}
```

Ranges 使我们能够更直接地表达大约 99% 的常见算法使用场景。除了符号上的优势外，ranges 还提供了一些优化机会，并消除了一类“愚蠢的错误”，例如 `sort(v1.begin(), v2.end())` 和 `sort(v.end(), v.begin())`。是的，这类错误在“现实中”确实出现过。

===== 第 3 页 =====

自然，存在不同类型的范围对应于不同类型的迭代器。特别地，`input_range`、`forward_range`、`bidirectional_range`、`random_access_range` 和 `contiguous_range` 都以概念的形式表示（§14.5）。

## 14.2 视图

**视图**（view）是一种查看范围的方式。例如：

```cpp
void user(forward_range auto& r)
{
    filter_view v {r, [](int x) { return x % 2; }};   // 只查看 r 中的奇数
    cout << "odd numbers: ";
    for (int x : v)
        cout << x << ' ';
}
```

当从 `filter_view` 读取时，我们从它的底层范围读取。如果读取的值匹配谓词，则返回它；否则，`filter_view` 会尝试从范围中获取下一个元素。

许多范围是无限的。此外，我们通常只想要少数几个值。因此，有一些视图只从范围中取少量值：

```cpp
void user(forward_range auto& r)
{
    filter_view v {r, [](int x) { return x % 2; }};   // 只查看 r 中的奇数
    take_view tv {v, 100};                            // 最多从 v 中取 100 个元素
    cout << "odd numbers: ";
    for (int x : tv)
        cout << x << ' ';
}
```

我们可以直接使用 `take_view` 而不用给它命名：

```cpp
for (int x : take_view{v, 3})
    cout << x << ' ';
```

类似地对于 `filter_view`：

```cpp
for (int x : take_view{ filter_view { r, [](int x) { return x % 2; } }, 3 })
    cout << x << ' ';
```

这种视图的嵌套很快就会变得有些晦涩，因此有一种替代方案：管道（§14.4）。

标准库提供了许多视图，也称为范围适配器：

**标准库视图（范围适配器）`<ranges>`**（`v` 是视图；`r` 是范围；`p` 是谓词；`n` 是整数）

| 视图 | 描述 |
|------|------|
| `v = all_view{r}` | `v` 是 `r` 中的所有元素 |
| `v = filter_view{r, p}` | `v` 是 `r` 中满足 `p` 的元素 |
| `v = transform_view{r, f}` | `v` 是对 `r` 中每个元素调用 `f` 的结果 |
| `v = take_view{r, n}` | `v` 是 `r` 中最多 `n` 个元素 |
| `v = take_while_view{r, p}` | `v` 是 `r` 中直到（但不包括）第一个不满足 `p` 的元素为止的元素 |
| `v = drop_view{r, n}` | `v` 是 `r` 中从第 `n+1` 个元素开始的元素 |
| `v = drop_while_view{r, p}` | `v` 是 `r` 中从第一个不满足 `p` 的元素开始的元素 |
| `v = join_view{r}` | `v` 是 `r` 的扁平化版本；`r` 的元素本身必须是范围 |
| `v = split_view(r, d)` | `v` 是由分隔符 `d` 确定的 `r` 的子范围的范围；`d` 必须是一个元素或一个范围 |
| `v = common_view(r)` | `v` 是由 `{begin, end}` 对描述的 `r` |
| `v = reverse_view{r}` | `v` 是 `r` 中元素的反向顺序；`r` 必须支持双向访问 |
| `v = views::elements<n>(r)` | `v` 是 `r` 中元组元素的第 `n` 个元素的范围 |
| `v = keys_view{r}` | `v` 是 `r` 中 pair 元素的第一个元素的范围 |
| `v = values_view{r}` | `v` 是 `r` 中 pair 元素的第二个元素的范围 |
| `v = ref_view{r}` | `v` 是 `r` 的元素的引用范围 |

视图提供的接口与范围非常相似，因此在大多数情况下，我们可以在任何可以使用范围的地方以相同的方式使用视图。关键区别在于视图不拥有其元素；它不负责删除其底层范围的元素——那是范围的责任。另一方面，视图不能超出其范围的生命周期：

```cpp
auto bad()
{
    vector v = {1, 2, 3, 4};
    return filter_view{v, odd};   // v 将在视图之前被销毁
}
```

视图应该廉价拷贝，因此我们按值传递它们。

我使用了简单的标准类型来保持示例简单，但当然，我们也可以拥有我们自己用户定义类型的视图。例如：

```cpp
struct Reading {
    int location {};
    int temperature {};
    int humidity {};
    int air_pressure {};
    //...
};

int average_temp(vector<Reading> readings)
{
    if (readings.size() == 0) throw No_readings{};
    double s = 0;
    for (int x : views::elements<1>(readings))   // 只查看温度字段
        s += x;
    return s / readings.size();
}
```

## 14.3 生成器

通常，范围需要按需生成。标准库为此提供了几个简单的生成器（也称为工厂）：

**范围工厂 `<ranges>`**（`v` 是视图；`x` 是元素类型 `T` 的值；`is` 是一个 `istream`）

| 生成器 | 描述 |
|--------|------|
| `v = empty_view<T>{}` | `v` 是一个空的 `T` 类型元素的范围（如果它有元素的话） |
| `v = single_view{x}` | `v` 是包含单个元素 `x` 的范围 |
| `v = iota_view{x}` | `v` 是一个无限范围：`x, x+1, x+2, ...`（使用 `++` 递增） |
| `v = iota_view{x, y}` | `v` 是一个包含 `n` 个元素的范围：`x, x+1, ..., y-1`（使用 `++` 递增） |
| `v = istream_view<T>{is}` | `v` 是通过对 `is` 调用 `>>` 获取 `T` 值所得到的范围 |

`iota_view` 对于生成简单序列非常有用。例如：

```cpp
for (int x : iota_view(42, 52))   // 42 43 44 45 46 47 48 49 50 51
    cout << x << ' ';
```

`istream_view` 为我们提供了一种在 range-for 循环中使用 `istream` 的简单方法：

```cpp
for (auto x : istream_view<complex<double>>(cin))
    cout << x << '\n';
```

像其他视图一样，`istream_view` 可以与其他视图组合：

```cpp
auto cplx = istream_view<complex<double>>(cin);
for (auto x : transform_view(cplx, [](auto z) { return z * z; }))
    cout << x << '\n';
```

输入 `1 2 3` 会产生 `1 4 9`。

## 14.4 管道

对于每个标准库视图（§14.2），标准库还提供了一个生成该视图的函数；也就是说，一个可以作为 `|` 运算符参数的对象。例如，`filter()` 产生一个 `filter_view`。这允许我们按顺序组合这些过滤器，而不是将它们呈现为一组嵌套的函数调用。

```cpp
void user(forward_range auto& r)
{
    auto odd = [](int x) { return x % 2; };
    for (int x : r | views::filter(odd) | views::take(3))
        cout << x << ' ';
}
```

输入范围 `1 2 3 4 5 6 7 8 9 10` 会产生 `1 2 3`。

管道风格（使用 Unix 管道运算符 `|`）被广泛认为比嵌套函数调用更具可读性。管道从左到右工作；即 `f | g` 将 `f` 的结果传递给 `g`，因此 `r | f | g` 意味着 `(g_filter(f_filter(r)))`。初始的 `r` 必须是一个范围或生成器。

这些过滤器函数位于命名空间 `ranges::views` 中：

```cpp
void user(forward_range auto& r)
{
    for (int x : r | views::filter([](int x) { return x % 2; }) | views::take(3))
        cout << x << ' ';
}
```

我发现显式使用 `views::` 使代码相当可读，但当然我们可以进一步缩短代码：

```cpp
void user(forward_range auto& r)
{
    using namespace views;
    auto odd = [](int x) { return x % 2; };
    for (int x : r | filter(odd) | take(3))
        cout << x << ' ';
}
```

视图和管道的实现涉及一些相当棘手的模板元编程，因此如果你担心性能，请务必测量你的实现是否满足你的需求。如果不满足，总有一个传统的变通方法：

```cpp
void user(forward_range auto& r)
{
    int count = 0;
    for (int x : r)
        if (x % 2) {
            cout << x << ' ';
            if (++count == 3) return;
        }
}
```

然而，在这里，其中的逻辑被掩盖了。

## 14.5 概念概览

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

## 14.6 建议

[1] 当迭代器对的风格变得繁琐时，使用 range 算法；§13.1；§14.1。
[2] 在使用 range 算法时，记得显式引入其名称；§13.3.1。
[3] 对范围的操作管道可以使用视图、生成器和过滤器来表达；§14.2，§14.3，§14.4。
[4] 要使用谓词结束一个范围，需要定义一个哨兵；§14.5。
[5] 使用 `static_assert`，我们可以检查特定类型是否满足概念的要求；§8.2.4。
[6] 如果你需要一个 range 算法而标准中没有，那就自己写一个；§13.6。
[7] 类型的理想状态是 regular；§14.5。
[8] 在适用的情况下，优先使用标准库概念；§14.5。
[9] 当请求并行执行时，务必避免数据竞争（§18.2）和死锁（§18.3）；§13.6。