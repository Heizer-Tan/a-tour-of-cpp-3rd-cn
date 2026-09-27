# 范围

> **最强的论证在结论未经经验验证时什么也证明不了。**
>
> —— Roger Bacon

# 14.1 引言

标准库同时提供两类算法：一类用概念（[第 8 章](../ch08/index.md)）加以约束，另一类保持无约束以兼容旧代码。受约束（基于概念）的版本位于 `<ranges>`，命名空间为 `ranges`。自然，我更偏好使用概念的版本。

**范围**（range）是对 C++98 时代那种由 `{begin(), end()}` 迭代器对所刻画序列的推广：它说明“要成为一串元素序列需要具备什么”。一个范围可以由以下形式给出：

- 一对迭代器 `{begin, end}`；
- `{begin, n}`，其中 `begin` 是迭代器，`n` 是元素个数；
- `{begin, pred}`，其中 `begin` 是迭代器，`pred` 是谓词；若对某个迭代器 `p` 有 `pred(p)` 为真，则表示到达范围末尾。这允许存在无限范围，以及按需生成的范围（[§14.3](14-3-generators.md)）。

正是 range 概念让我们可以写 `sort(v)`，而不必像自 1994 年以来使用 STL 那样写 `sort(v.begin(), v.end())`。对自己的算法也能做类似处理：

```cpp
template<forward_range R>
    requires sortable<iterator_t<R>>
void my_sort(R& r) // 现代的、带概念约束的 my_sort 版本
{
    return my_sort(r.begin(), r.end()); // 使用 1994 风格的 sort
}
```

Ranges 让我们能更直接地表达大约 99% 的日常算法用法。除了记法上的好处，ranges 还能带来某些优化机会，并消灭一整类低级错误——例如 `sort(v1.begin(), v2.end())`、`sort(v.end(), v.begin())` 这种组合；现实中确实见过。

自然，也存在不同“种类”的范围，对应不同种类的迭代器。尤其地，`input_range`、`forward_range`、`bidirectional_range`、`random_access_range` 与 `contiguous_range` 都以概念的形式给出（[§14.5](14-5-concept-overview.md)）。

# 14.2 视图

**视图**（view）是一种“如何去看某个范围”的方式。例如：

```cpp
void user(forward_range auto& r)
{
    filter_view v {r, [](int x) { return x % 2; }}; // 只看 r 中的奇数
    cout << "odd numbers: ";
    for (int x : v)
        cout << x << ' ';
}
```

从 `filter_view` 读取时，实际上是从其底层范围读取：若读到的值满足谓词就返回；否则 `filter_view` 会继续尝试下一个元素。

许多范围是无限的；而我们往往只需要很少的值。因此还有只“截取前若干个元素”的视图：

```cpp
void user(forward_range auto& r)
{
    filter_view v {r, [](int x) { return x % 2; }}; // r 中的奇数
    take_view tv {v, 100};                       // 从 v 中最多取 100 个元素

    cout << "odd numbers: ";
    for (int x : tv)
        cout << x << ' ';
}
```

也可以不为 `take_view` 单独命名，直接写：

```cpp
for (int x : take_view{v, 3})
    cout << x << ' ';
```

对 `filter_view` 也类似：

```cpp
for (int x : take_view{filter_view{r, [](int x) { return x % 2; }}, 3})
    cout << x << ' ';
```

这种层层嵌套的视图很快就会晦涩难懂，因此还有另一条路可走：**管道**（[§14.4](14-4-pipes.md)）。

标准库提供了很多视图（也常被称为范围适配器）：

**标准库视图（范围适配器）`<ranges>`**（`v` 是视图；`r` 是范围；`p` 是谓词；`n` 是整数）

| 视图 | 说明 |
|------|------|
| `v = all_view{r}` | `v` 包含 `r` 的全部元素 |
| `v = filter_view{r, p}` | `v` 包含 `r` 中满足 `p` 的元素 |
| `v = transform_view{r, f}` | `v` 包含对 `r` 中各元素调用 `f` 的结果 |
| `v = take_view{r, n}` | `v` 至多包含 `r` 的前 `n` 个元素 |
| `v = take_while_view{r, p}` | `v` 包含 `r` 的开头一段连续元素，直到某个元素不满足 `p` |
| `v = drop_view{r, n}` | `v` 从 `r` 的第 `n+1` 个元素开始 |
| `v = drop_while_view{r, p}` | `v` 从 `r` 中首个不满足 `p` 的元素开始 |
| `v = join_view{r}` | `v` 是“压平”后的范围；`r` 的元素本身必须也是范围 |
| `v = split_view(r, d)` | `v` 是由分隔符 `d` 切分 `r` 得到的子范围序列；`d` 可以是单个元素或一个范围 |
| `v = common_view(r)` | `v` 用普通的 `{begin, end}` 迭代器对描述 `r` |
| `v = reverse_view{r}` | `v` 按逆序遍历 `r`；要求 `r` 支持双向迭代 |
| `v = views::elements<n>(r)` | `v` 取 `r` 中每个元组元素的第 `n` 个分量 |
| `v = keys_view{r}` | `v` 取 `r` 中每个 `pair` 的第一个分量 |
| `v = values_view{r}` | `v` 取 `r` 中每个 `pair` 的第二个分量 |
| `v = ref_view{r}` | `v` 提供对 `r` 元素的引用视图 |

视图对外呈现的接口与范围极为相似，因此在绝大多数场合可以把视图当作范围那样使用。关键区别在于：**视图不拥有元素**，也不负责释放底层范围里的对象——那是底层范围的责任。另一方面，视图的生命周期不能长于它所观测的范围：

```cpp
auto bad()
{
    vector v = {1, 2, 3, 4};
    return filter_view{v, odd}; // `v` 会在视图之前被销毁
}
```

视图应当廉价拷贝，因此通常按值传递。

示例里我用了最简单的标准类型来保持短小；当然也能为用户自定义类型建立视图。例如：

```cpp
struct Reading {
    int location {};
    int temperature {};
    int humidity {};
    int air_pressure {};
    // ...
};

int average_temp(vector<Reading> readings)
{
    if (readings.size() == 0)
        throw No_readings{};
    double s = 0;
    for (int x : views::elements<1>(readings)) // 只看温度字段
        s += x;
    return s / readings.size();
}
```

# 14.3 生成器

很多时候范围需要“临时生成”。标准库为此提供了少量简单的生成器（也称为工厂）：

**范围工厂 `<ranges>`**（`v` 是视图；`x` 为元素类型 `T` 的值；`is` 为输入流）

| 工厂 | 说明 |
|------|------|
| `v = empty_view<T>{}` | `v` 是元素类型为 `T` 的空范围（即便原本“会有”元素） |
| `v = single_view{x}` | `v` 只包含元素 `x` |
| `v = iota_view{x}` | `v` 是无限序列：`x, x+1, x+2, ...`（用 `++` 递增） |
| `v = iota_view{x, y}` | `v` 是含 `n` 个元素的序列：`x, x+1, ..., y-1`（用 `++` 递增） |
| `v = istream_view<T>{is}` | `v` 通过对 `is` 反复执行 `>>`（读取 `T`）得到 |

`iota_view` 很适合构造简单序列：

```cpp
for (int x : iota_view(42, 52)) // 42 43 44 45 46 47 48 49 50 51
    cout << x << ' ';
```

`istream_view` 让我们能在范围 `for` 里方便地使用输入流：

```cpp
for (auto x : istream_view<complex<double>>(cin))
    cout << x << '\n';
```

与其他视图一样，`istream_view` 也能与其他视图组合：

```cpp
auto cplx = istream_view<complex<double>>(cin);

for (auto x : transform_view(cplx, [](auto z) { return z * z; }))
    cout << x << '\n';
```

若输入为 `1 2 3`，输出将是 `1 4 9`。

# 14.4 管道

对每个标准库视图（[§14.2](14-2-views.md)），标准库还提供一个产出“过滤器”的函数；也就是说，产出的对象可作为过滤器运算符 `|` 的操作数。例如，`filter()` 会给出一个 `filter_view`。于是可以把过滤器串成序列，而不必写成层层嵌套的函数调用：

```cpp
void user(forward_range auto& r)
{
    auto odd = [](int x) { return x % 2; };

    for (int x : r | views::filter(odd) | views::take(3))
        cout << x << ' ';
}
```

管道风格（沿用 Unix 管道运算符 `|`）普遍被认为比嵌套调用更易读。管道从左向右工作：对 `f|g` 而言，`f` 的结果被传给 `g`，因此 `r|f|g` 表示 `(g_filter(f_filter(r)))`。最初的 `r` 必须是一个范围或生成器。

这些过滤器函数位于命名空间 `ranges::views`：

```cpp
void user(forward_range auto& r)
{
    for (int x : r | views::filter([](int x) { return x % 2; }) | views::take(3))
        cout << x << ' ';
}
```

我发现显式写出 `views::` 往往更清楚；当然也可以进一步缩短：

```cpp
void user(forward_range auto& r)
{
    using namespace views;

    auto odd = [](int x) { return x % 2; };

    for (int x : r | filter(odd) | take(3))
        cout << x << ' ';
}
```

视图与管道的实现涉及相当令人咋舌的模板元编程；若你关心性能，请务必实测你的实现是否满足预期。否则总还有传统写法兜底：

```cpp
void user(forward_range auto& r)
{
    int count = 0;
    for (int x : r)
        if (x % 2) {
            cout << x << ' ';
            if (++count == 3)
                return;
        }
}
```

只不过在这种写法里，“到底在做什么”这层意图更容易被细节淹没。

# 14.5 概念概览

标准库提供了大量有用的概念，大体可以分为三类：

- 描述类型性质的概念（[§14.5.1](14-5-concept-overview.md#14.5.1)）
- 描述迭代器的概念（[§14.5.2](14-5-concept-overview.md#14.5.2)）
- 描述范围的概念（[§14.5.3](14-5-concept-overview.md#14.5.3)）

## 14.5.1 描述类型性质的概念

与类型性质及类型之间关系相关的概念反映了类型的多样性；它们能帮助我们把多数模板写得更简单。

**核心语言概念 `<concepts>`**（`T`、`U` 为类型）

| 概念 | 含义 |
|------|------|
| `same_as<T, U>` | `T` 与 `U` 相同 |
| `derived_from<T, U>` | `T` 派生自 `U` |
| `convertible_to<T, U>` | `T` 可转换为 `U` |
| `common_reference_with<T, U>` | `T` 与 `U` 拥有共同的引用类型 |
| `common_with<T, U>` | `T` 与 `U` 拥有共同类型 |
| `integral<T>` | `T` 为整数类型 |
| `signed_integral<T>` | `T` 为有符号整数类型 |
| `unsigned_integral<T>` | `T` 为无符号整数类型 |
| `floating_point<T>` | `T` 为浮点类型 |
| `assignable_from<T, U>` | 可以把 `U` 赋给 `T` |
| `swappable_with<T, U>` | `T` 与 `U` 可互换 |
| `swappable<T>` | `swappable_with<T, T>` |

许多算法需要在一组彼此关联的类型上工作——例如表达式里混用 `int` 与 `double`。我们用 `common_with` 描述这种混合是否在语义上合理。若 `common_with<X, Y>` 成立，就可以先把 `X` 与 `Y` 都提升到 `common_type_t<X, Y>` 再比较。例如：

```cpp
common_type_t<string, const char*> s1 = some_fct();
common_type_t<string, const char*> s2 = some_other_fct();

if (s1 < s2) {
    // ...
}
```

要为一对类型指定公共类型，可以为 `common_type`（进而影响 `common_type_t`）做特化。例如：

```cpp
using common_type_t<Bigint, long> = Bigint; // 假定已有合适的 Bigint 定义
```

幸运的是，除非想在某些尚未由库覆盖的类型组合上使用混合运算，我们通常并不需要自定义这种特化。

比较相关的概念深受 [Stepanov, 2009] 的影响。

**比较概念 `<concepts>`**

| 概念 | 含义 |
|------|------|
| `equality_comparable_with<T, U>` | `T` 与 `U` 可用 `==` 比较是否等价 |
| `equality_comparable<T>` | `equality_comparable_with<T, T>` |
| `totally_ordered_with<T, U>` | `T` 与 `U` 可用 `<`、`<=`、`>`、`>=` 构成全序 |
| `totally_ordered<T>` | `totally_ordered_with<T, T>` |
| `three_way_comparable_with<T, U>` | `T` 与 `U` 可用 `<=>` 给出一致的结果 |
| `three_way_comparable<T>` | `three_way_comparable_with<T, T>` |

同时存在 `equality_comparable_with` 与 `equality_comparable` 反映了概念设计上尚未充分挖掘的重载机会。

有趣的是标准库里没有一个通用的布尔概念；我常在别处需要一个，下面是示意版本：

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

写模板时，还经常需要对类型做归类。

**对象概念 `<concepts>`**

| 概念 | 含义 |
|------|------|
| `destructible<T>` | `T` 可被销毁，并可对其取地址（一元 `&`） |
| `constructible_from<T, Args...>` | 可用类型列表 `Args...` 构造 `T` |
| `default_initializable<T>` | `T` 可默认构造 |
| `move_constructible<T>` | `T` 可移动构造 |
| `copy_constructible<T>` | `T` 可复制构造且可移动构造 |
| `movable<T>` | 移动构造、`assignable_from<T&, T>` 且可交换 |
| `copyable<T>` | 可复制构造、`movable<T>`，且 `assignable_from<T&, const T&>` |
| `semiregular<T>` | `copyable<T>` 且 `default_initializable<T>` |
| `regular<T>` | `semiregular<T>` 且 `equality_comparable<T>` |

类型的理想形态是 **regular**：行为大致像 `int`，能显著简化我们对用法的一切推理（[§8.2](../ch08/8-2-concepts.md)）。由于类默认不提供 `==`，多数类一开始往往只是 `semiregular`，尽管其中许多完全可以、也应当演进成 `regular`。

每当我们把某个“运算”作为带约束的模板实参传入时，都需要说明它如何被调用，有时还要说明我们对其语义所做假设。

**可调用概念 `<concepts>`**

| 概念 | 含义 |
|------|------|
| `invocable<F, Args...>` | `F` 可用参数列表 `Args...` 调用 |
| `regular_invocable<F, Args...>` | `invocable<F, Args...>`，并且在等价输入上保持等价输出 |
| `predicate<F, Args...>` | `regular_invocable<F, Args...>` 且返回 `bool` |
| `relation<F, T, U>` | `predicate<F, T, U>` |
| `equivalence_relation<F, T, U>` | 提供等价关系的 `relation<F, T, U>` |
| `strict_weak_order<F, T, U>` | 提供严格弱序的 `relation<F, T, U>` |

若 `x == y` 蕴含 `f(x) == f(y)`，则称 `f` **保持等价**。`invocable` 与 `regular_invocable` 的差别仅在语义层面——目前还无法在代码里完全刻画这种差别，因此命名主要表达意图。类似地，`relation` 与 `equivalence_relation` 的差别也在语义上：等价关系满足自反、对称与传递。`relation` 与 `strict_weak_order` 的差别同样在语义上；严格弱序大致就是标准库对 `<` 一类比较的常规假设。

## 14.5.2 迭代器概念

经典的标准算法通过迭代器访问数据，因此需要一组概念来刻画迭代器类型的性质。

**迭代器概念 `<iterator>`**

| 概念 | 含义 |
|------|------|
| `input_or_output_iterator<I>` | `I` 可自增（`++`）并可解引用（`*`） |
| `sentinel_for<S, I>` | `S` 是迭代器 `I` 的哨兵：可理解为作用在 `I` 的值类型上的结束条件 |
| `sized_sentinel_for<S, I>` | 哨兵 `S` 使得可对 `I` 使用 `-` |
| `input_iterator<I>` | `I` 为输入迭代器：`*` 主要用于读取 |
| `output_iterator<I>` | `I` 为输出迭代器：`*` 主要用于写入 |
| `forward_iterator<I>` | 前向迭代器：可多趟遍历并支持 `==` |
| `bidirectional_iterator<I>` | 在 `forward_iterator` 基础上支持 `--` |
| `random_access_iterator<I>` | 在双向迭代器基础上支持 `+`、`-`、`+=`、`-=`、`[]` |
| `contiguous_iterator<I>` | 随机访问迭代器且元素存放在连续内存 |
| `permutable<I>` | 前向迭代器且支持移动与交换 |
| `mergeable<I1, I2, R, O>` | 能否把 `I1`、`I2` 所指有序序列按关系 `R` 合并到 `O` |
| `sortable<I>` | 能否用 `<` 对 `I` 所指序列排序 |
| `sortable<I, R>` | 能否用关系 `R` 对 `I` 所指序列排序 |

这里的 `mergeable`、`sortable` 相对 C++20 正式定义做了简化。

不同类别的迭代器用于为给定实参选择最合适的算法实现；参见 [§8.2.2](../ch08/8-2-concepts.md#8.2.2)、[§16.4.1](../ch16/16-4-type-functions.md#16.4.1)。输入迭代器的例子可见 [§13.3.1](../ch13/13-3-iterator-types.md#13.3.1)。

哨兵的基本思想是：从某个迭代器出发前进，直到谓词对当前元素成立为止，因此迭代器 `p` 与哨兵 `s` 刻画范围 `[p:s(*p))`。举例来说，可以用指针作迭代器，为遍历 C 风格字符串定义哨兵谓词。遗憾的是，这需要一些样板代码：意图是把谓词呈现为“不会与普通迭代器混淆、却又能与用于遍历该范围的迭代器相比较”的东西：

```cpp
template<class Iter>
class Sentinel {
public:
    Sentinel(iter_value_t<Iter> ee) : end(ee) { }
    Sentinel() : end{} {} // sentinel_for 通常要求可默认构造

    friend bool operator==(const Iter& p, Sentinel s) { return *p == s.end; }
    friend bool operator!=(const Iter& p, Sentinel s) { return !(p == s); }
private:
    iter_value_t<Iter> end; // 哨兵所代表的终止值
};
```

在类作用域内用友元声明，就能定义用于比较迭代器与哨兵的二元函数 `==`、`!=`。

可以静态断言该哨兵满足要求：

```cpp
static_assert(sentinel_for<Sentinel<const char*>, const char*>); // 检查 Sentinel
```

最后可以得到一个颇为特别的 “Hello, World!” 程序：

```cpp
const char aa[] = "Hello, World!\nBye for now\n";

ranges::for_each(aa, Sentinel<const char*>('\n'), [](const char x) { cout << x; });
```

是的，它只会写出 `Hello, World!`，后面不会紧跟换行。

## 14.5.3 范围概念

范围概念刻画“何谓范围”。

**范围概念 `<ranges>`**

| 概念 | 含义 |
|------|------|
| `range<R>` | `R` 拥有 `begin` 迭代器与哨兵 |
| `sized_range<R>` | `R` 能在常数时间内获知大小 |
| `view<R>` | `R` 是范围，且拷贝、移动、赋值都具有摊销常数时间 |
| `common_range<R>` | `R` 的迭代器类型与哨兵类型相同 |
| `input_range<R>` | `R` 的迭代器满足 `input_iterator` |
| `output_range<R>` | `R` 的迭代器满足 `output_iterator` |
| `forward_range<R>` | `R` 的迭代器满足 `forward_iterator` |
| `bidirectional_range<R>` | `R` 的迭代器满足 `bidirectional_iterator` |
| `random_access_range<R>` | `R` 的迭代器满足 `random_access_iterator` |
| `contiguous_range<R>` | `R` 的迭代器满足 `contiguous_iterator` |

`<ranges>` 里还有更多概念，但上面这组已经足够入门。它们最主要的用途，是让实现能够依据输入类型的性质来选择重载（[§8.2.2](../ch08/8-2-concepts.md#8.2.2)）。

# 14.6 建议

[1] 当“迭代器对”风格显得累赘时，改用范围算法；[§13.1](../ch13/13-1-introduction.md)；[§14.1](14-1-introduction.md)。

[2] 使用范围算法时，记得显式引入所需的名字；[§13.3.1](../ch13/13-3-iterator-types.md#13.3.1)。

[3] 对范围施加的一连串操作可以用视图、生成器与过滤器组成的管道来表达；[§14.2](14-2-views.md)，[§14.3](14-3-generators.md)，[§14.4](14-4-pipes.md)。

[4] 若要用谓词结束一个范围，需要定义哨兵；[§14.5](14-5-concept-overview.md)。

[5] 借助 `static_assert`，可以检查某个具体类型是否满足概念的语法要求；[§8.2.4](../ch08/8-2-concepts.md#8.2.4)。

[6] 若你需要某个范围算法而标准库尚未提供，不妨自己写一个；[§13.6](../ch13/13-6-parallel-algorithms.md)。

[7] 类型的理想目标是 regular；[§14.5](14-5-concept-overview.md)。

[8] 在适用之处优先使用标准库概念；[§14.5](14-5-concept-overview.md)。

[9] 请求并行执行时务必避免数据竞争（[§18.2](../ch18/18-2-tasks-and-threads.md)）与死锁（[§18.3](../ch18/18-3-shared-data.md)）；[§13.6](../ch13/13-6-parallel-algorithms.md)。
