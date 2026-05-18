# 14.5 概念概览

标准库提供了大量有用的概念，大体可以分为三类：

- 描述类型性质的概念（§14.5.1）
- 描述迭代器的概念（§14.5.2）
- 描述范围的概念（§14.5.3）

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

要为一对类型指定公共类型，可以为 `common_type`（进而影响 `common_type_t`）做特化。幸运的是，除非想在某些尚未由库覆盖的类型组合上使用混合运算，我们通常并不需要自定义这种特化。

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

类型的理想形态是 **regular**：行为大致像 `int`，能显著简化我们对用法的一切推理（§8.2）。由于类默认不提供 `==`，多数类一开始往往只是 `semiregular`，尽管其中许多完全可以、也应当演进成 `regular`。

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

不同类别的迭代器用于为给定实参选择最合适的算法实现；参见 §8.2.2、§16.4.1。输入迭代器的例子可见 §13.3.1。

哨兵的基本思想是：从某个迭代器出发前进，直到谓词对当前元素成立为止，因此迭代器 `p` 与哨兵 `s` 刻画范围 `[p:s(*p))`。举例来说，可以定义遍历以 `\n` 结尾的 C 风格字符串段的哨兵。为避免与普通迭代器混淆，这需要少量样板代码：

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

在类作用域内用友元声明 `==`、`!=`，就能把“迭代器与哨兵比较”的二元运算符定义清楚。

可以静态断言该哨兵满足要求：

```cpp
static_assert(sentinel_for<Sentinel<const char*>, const char*>);
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

`<ranges>` 里还有更多概念，但上面这组已经足够入门。它们最主要的用途，是让实现能够依据输入类型的性质来选择重载（§8.2.2）。
