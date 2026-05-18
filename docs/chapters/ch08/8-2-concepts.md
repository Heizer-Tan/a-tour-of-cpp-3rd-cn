# 8.2 概念

考虑 [§7.3.1](../ch07/7-3-parameterized-operations.md) 中的 `sum()`：

```cpp
template<typename Seq, typename Value>
Value sum(Seq s, Value v)
{
    for (const auto& x : s)
        v += x;
    return v;
}
```

这个 `sum()` 要求：

- 它的第一个模板参数是某种元素序列，并且
- 它的第二个模板参数是某种数字。

更具体地说，`sum()` 可以为一对参数调用：

- 序列 `Seq` 能够提供 `begin()` / `end()`，使区间 `for` 可用（[§1.7](../ch01/1-7-pointers-arrays.md)；[§14.1](../ch14/14-1-introduction.md)）。
- 一个算术类型 `Value`，支持 `+=` 以便序列的元素可以被相加。

我们称这样的要求为**概念**。

满足这个简化要求（以及更多）的类型示例，作为序列（也称为 range）的标准库 `vector`、`list` 和 `map`。满足这个简化要求（以及更多）的算术类型示例包括 `int`、`double` 和 `Matrix`（对于任何合理的 `Matrix` 定义）。我们可以说 `sum()` 算法在两个维度上是通用的：用于存储元素的数据结构类型（“序列”）和元素的类型。

## 8.2.1 概念的使用

大多数模板实参都必须满足一整套前提，生成的代码才可期待正确行为。这也意味着绝大多数模板应当是**带约束模板**（[§7.2.1](../ch07/7-2-parameterized-types.md)）。仅仅写下 `typename` 是最宽松的约束——它只说明“此地需要某种类型”，往往还能更进一步。再看一次 `sum()`：

```cpp
template<Sequence Seq, Number Num>
Num sum(Seq s, Num v)
{
    for (const auto& x : s)
        v += x;
    return v;
}
```

这清晰得多。一旦我们定义了概念 `Sequence` 和 `Number` 的含义，编译器就可以仅通过查看 `sum()` 的接口（而不是其实现）来拒绝错误的调用。这改进了错误报告。

然而，`sum()` 的接口规范并不完整：我“忘记”说明应该能够将 `Sequence` 的元素与 `Number` 相加。我们可以做到这一点：

```cpp
template<Sequence Seq, Number Num>
    requires Arithmetic<range_value_t<Seq>, Num>
Num sum(Seq s, Num n);
```

`range_value_t` 给出了序列元素的类型——标准库里正是用它描述 range 值的类别（[§14.1](../ch14/14-1-introduction.md)，亦见 [§16.4.4](../ch16/16-4-type-functions.md)）。`Arithmetic<X, Y>` 则声明：能够以 `X`、`Y` 参与常规算术而不会撞墙。有了这层约束，就不会误把 `vector<string>`、`vector<int*>` 当作可求和的序列；`vector<int>`、`vector<complex<double>>` 之类却仍然畅通无阻。一般说来，当一个算法同时使用多种类型参数时，最好明确指出它们之间的关系。

在这个例子中，我们只需要 `+=`，但为了简单和灵活，我们不应该过于严格地约束模板参数。特别地，有一天我们可能希望用 `+` 和 `=` 而不是 `+=` 来表达 `sum()`，那时我们会很高兴我们使用了一个通用的概念（此处为 `Arithmetic`），而不是一个狭隘的“拥有 `+=`”的要求。

部分规范，如第一个使用概念的 `sum()`，可能非常有用。除非规范完整，否则某些错误要到实例化时才会被发现。然而，即使是部分规范也表达了意图，对于我们在最初并未认识到所有必要需求的平滑增量开发至关重要。有了成熟的概念库，初始规范将接近完美。

不出所料，`requires Arithmetic<range_value_t<Seq>,Num>` 被称为**需求子句**。`template<Sequence Seq>` 表示法只是显式使用 `requires Sequence<Seq>` 的简写。如果我喜欢冗长，我可以等价地写成：

```cpp
template<typename Seq, typename Num>
    requires Sequence<Seq> && Number<Num> && Arithmetic<range_value_t<Seq>, Num>
Num sum(Seq s, Num n);
```

另一方面，我们也可以使用两种表示法之间的等价性来写：

```cpp
template<Sequence Seq, Arithmetic<range_value_t<Seq>> Num>
Num sum(Seq s, Num n);
```

在还不能使用概念的代码库中，我们只能使用命名约定和注释，例如：

```cpp
template<typename Sequence, typename Number>
    // requires Arithmetic<range_value_t<Sequence>, Number>
Number sum(Sequence s, Number n);
```

无论采用哪种记号，请务必让模板参数的语义约束清晰可追踪（参见 [§8.2.4](8-2-concepts.md)）。

## 8.2.2 基于概念的重载

只要把模板的接口用概念描述清楚，就可以像函数重载那样，根据性质挑选不同的模板实例。考虑精简版标准函数 `advance()`，它会把迭代器沿序列推进（参见 [§13.3](../ch13/13-3-iterator-types.md)）：

```cpp
template<forward_iterator Iter>
void advance(Iter p, int n)   // 将 p 向前移动 n 个元素
{
    while (n--)
        ++p;   // 前向迭代器有 ++，但没有 + 或 +=
}

template<random_access_iterator Iter>
void advance(Iter p, int n)   // 将 p 向前移动 n 个元素
{
    p += n;   // 随机访问迭代器有 +=
}
```

编译器将选择满足参数的最强要求的模板。在这种情况下，`list` 只提供前向迭代器，但 `vector` 提供随机访问迭代器，因此我们得到：

```cpp
void user(vector<int>::iterator vip, list<string>::iterator lsp)
{
    advance(vip, 10);   // 使用快速的 advance()
    advance(lsp, 10);   // 使用慢速的 advance()
}
```

与常规的重载决议一样，这完全发生在编译期，没有运行时税；若没有唯一最佳候选，就会在编译时报歧义。**基于概念的候选挑选**比一般意义上的重载（[§1.3](../ch01/1-3-functions.md)）简单得多——先只看单参数的一组备选：

- 如果参数不匹配概念，则无法选择该备选。
- 如果参数仅匹配一个备选的概念，则选择该备选。
- 如果来自两个备选的参数都匹配某个概念，并且其中一个比另一个更严格（满足另一个的所有要求以及更多），则选择该备选。
- 如果来自两个备选的参数在概念上匹配得同样好，则存在歧义。

要选择某个备选，它必须：
- 匹配其所有参数，并且
- 对于所有参数，至少与其它备选匹配得一样好，并且

- 对于至少一个参数匹配得更好。

## 8.2.3 有效代码

一组模板实参是否提供模板对其模板形参的要求，最终归结为某些表达式是否有效。

使用 **requires 表达式**，我们可以检查一组表达式是否有效。例如，我们可以尝试在不使用标准库概念 `random_access_iterator` 的情况下编写 `advance()`：

```cpp
template<forward_iterator Iter>
    requires requires(Iter p, int i) { p[i]; p+i; }   // 迭代器具有下标和整数加法
void advance(Iter p, int n)   // 将 p 向前移动 n 个元素
{
    p += n;
}
```

不，那个 `requires requires` 不是拼写错误。第一个 `requires` 开始需求子句，第二个 `requires` 开始 requires 表达式：

```cpp
requires(Iter p, int i) { p[i]; p+i; }
```

requires 表达式是一个谓词，如果其中的语句是有效代码则为 `true`，否则为 `false`。

我认为 requires 表达式是泛型编程的汇编代码。像普通的汇编代码一样，requires 表达式非常灵活，并且不施加任何编程规范。以某种形式或其他形式，它们位于大多数有趣的泛型代码的最底层，就像汇编代码位于大多数有趣的普通代码的最底层一样。像汇编代码一样，requires 表达式不应该出现在普通代码中。它们属于抽象的实现。如果你在你的代码中看到 `requires requires`，它可能太底层了，最终会成为一个问题。

在 `advance()` 中使用 `requires requires` 是故意不优雅和拼凑的。请注意，我“忘记”指定 `+=` 以及操作的所需返回类型。因此，这个版本的 `advance()` 的某些用法会通过概念检查但仍然无法编译。警告过你了！适当的随机访问版本的 `advance()` 更简单、更可读：

```cpp
template<random_access_iterator Iter>
void advance(Iter p, int n)   // 将 p 向前移动 n 个元素
{
    p += n;   // 随机访问迭代器有 +=
}
```

应优先采纳语义完备的命名概念（[§8.2.4](8-2-concepts.md)），而把 requires 表达式藏在概念定义的内部。

## 8.2.4 概念的定义

标准库和社区库已经整理了诸如 `forward_iterator` 等在实践里摸爬滚打过的概念（[§14.5](../ch14/14-5-concept-overview.md)）。与现成的类、函数相仿，善用成熟库往往是上策；不过只要需求明确，小规模自定义也不难。一般而言，源自标准库的标识符（如 `random_access_iterator`、`vector`）都采用小写字母组合；本节中的 `Sequence`、`Vector` 等则是作者自行约定的概念名称，用大写开头以示区分。

**概念**是一个编译时谓词，指定一个或多个类型可以如何使用。首先考虑一个最简单的例子：

```cpp
template<typename T>
concept Equality_comparable = requires (T a, T b) {
    { a == b } -> Boolean;   // 使用 == 比较 T
    { a != b } -> Boolean;   // 使用 != 比较 T
};
```

`Equality_comparable` 是我们用来确保可以比较类型值的相等和不等的概念。我们简单地说，给定该类型的两个值，它们必须可以使用 `==` 和 `!=` 进行比较，并且这些操作的结果必须是 `Boolean`。例如：

```cpp
static_assert(Equality_comparable<int>);   // 成功

struct S { int a; };
static_assert(Equality_comparable<S>);     // 失败，因为结构体不会自动获得 ==
```

概念 `Equality_comparable` 的定义与英文描述完全等价，并不更长。概念的值总是 `bool`。

在 `->` 之后指定的 `{ ... }` 的结果必须是一个概念。

标准库里没有名为 `boolean` 的基础概念时，需要自己补一个极小版本（见 [§14.5](../ch14/14-5-concept-overview.md)）。下文中的 `Boolean` 仅表示“能出现在条件表达式里的那种结果类型”。

定义 `Equality_comparable` 来处理非同质比较几乎同样容易：

```cpp
template<typename T, typename T2 = T>
concept Equality_comparable = requires (T a, T2 b) {
    { a == b } -> Boolean;   // 使用 == 比较 T 与 T2
    { a != b } -> Boolean;   // 使用 != 比较 T 与 T2
    { b == a } -> Boolean;   // 使用 == 比较 T2 与 T
    { b != a } -> Boolean;   // 使用 != 比较 T2 与 T
};
```

`typename T2 = T` 表示如果我们不指定第二个模板参数，`T2` 将与 `T` 相同；`T` 是一个默认模板参数。

我们可以这样测试 `Equality_comparable`：

```cpp
static_assert(Equality_comparable<int, double>);   // 成功
static_assert(Equality_comparable<int>);            // 成功（T2 默认为 int）
static_assert(Equality_comparable<int, string>);    // 失败
```

它几乎就是标准概念的 `equality_comparable`（[§14.5](../ch14/14-5-concept-overview.md)）。

我们现在可以定义一个要求数字之间算术运算有效的概念。首先我们需要定义 `Number`：

```cpp
template<typename T, typename U = T>
concept Number = requires(T x, U y) {
    // 一些算术运算和一个零
    x + y;
    x - y;
    x * y;
    x / y;
    x += y;
    x -= y;
    x *= y;
    x /= y;
    x = x;      // 拷贝
    x = 0;
};
```

有了 `Number`，就能组合出本节开头用到的 `Arithmetic`（[§8.2.1](8-2-concepts.md)）：

```cpp
template<typename T, typename U = T>
concept Arithmetic = Number<T, U> && Number<U, T>;
```

对于一个更复杂的例子，考虑一个序列：

```cpp
template<typename S>
concept Sequence = requires(S a) {
    typename range_value_t<S>;        // S 必须有一个值类型
    typename iterator_t<S>;           // S 必须有一个迭代器类型
    { a.begin() } -> same_as<iterator_t<S>>;   // S 必须有一个 begin() 返回其迭代器
    { a.end() }   -> same_as<iterator_t<S>>;   // S 必须有一个 end() 返回其迭代器
    requires input_iterator<iterator_t<S>>;    // S 的迭代器必须是一个 input_iterator
    requires same_as<range_value_t<S>, iter_value_t<S>>; // 值类型与迭代器的值类型相同
};
```

若想让 `S` 表示 Sequence，它需要公布元素类型（值类型）与迭代器类型；标准库分别以 `range_value_t<S>`、`iterator_t<S>` 来描述（参见 [§16.4.4](../ch16/16-4-type-functions.md)，亦与 [§13.1](../ch13/13-1-introduction.md) 的讨论呼应）。同时还要存在返回上述迭代器的 `begin()`、`end()`，这也是容器惯用法（[§12.3](../ch12/12-3-list.md)）。最后，`S` 自带的迭代器至少应是 **input**，且迭代器解引用得到的值类型必须与 range 的元素类型一致。

最棘手的概念是那些试图刻画语言基本面的概念——与其闭门造车，不如直接复用库里的套件（可参考 [§14.5](../ch14/14-5-concept-overview.md)）。若想跳过上一段那样逐条罗列细节，可采用标准库的 `input_range` 一步到位：

```cpp
template<typename S>
concept Sequence = input_range<S>;   // 简单易写且通用
```

如果我将“S 的值类型”限制为 `S::value_type`，我可以使用一个简单的 `Value_type`：

```cpp
template<class S>
using Value_type = typename S::value_type;
```

这是一种简洁地表达简单概念并隐藏复杂性的有用技术。标准 `value_type_t` 的定义本质上是相似的，但稍微复杂一些，因为它处理没有名为 `value_type` 的成员的序列（例如内置数组）。

## 8.2.4.1 定义检查

为模板指定的概念用于在模板的使用点检查参数。它们不用于检查模板定义中参数的使用。例如：

```cpp
template<equality_comparable T>
bool cmp(T a, T b) { return a < b; }
```

这里，概念保证了 `==` 的存在，但没有保证 `<`。

```cpp
bool b0 = cmp(cout, cerr);   // 错误：ostream 不支持 ==
bool b1 = cmp(2, 3);         // OK：返回 true
bool b2 = cmp(2+3i, 3+4i);   // 错误：complex<double> 不支持 <
```

概念检查捕获了传递 ostream 的尝试，但接受了 `int` 和 `complex<double>`，因为这两种类型支持 `==`。然而，`int` 支持 `<` 所以 `cmp(2,3)` 编译，而 `cmp(2+3i,3+4i)` 在 `cmp()` 的主体被检查并为不支持 `<` 的 `complex<double>` 实例化时被拒绝。

将模板定义的最终检查推迟到实例化时间有两个好处：

- 我们可以在开发期间使用不完整的概念。这允许我们在开发概念、类型和算法的过程中积累经验，并逐步改进检查。
- 我们可以在模板中插入调试、跟踪、遥测等代码，而不影响其接口。更改接口可能导致大规模重新编译。

代价则是在模板真正实例化之前的最后时刻才能逮住某些自相矛盾用法（参阅 [§8.5](8-5-template-compilation.md)）。

## 8.2.5 概念与 auto

`auto` 告诉编译器：“此处类型请向初始化式看齐”（[§1.4.2](../ch01/1-4-types-variables.md)）。

```cpp
auto x = 1;                   // x 是 int
auto z = complex<double>{1,2}; // z 是 complex<double>
```

然而，初始化并不仅仅发生在简单的变量定义中：

```cpp
auto g() { return 99; }       // g() 返回 int
int f(auto x) { /* ... */ }   // 接受任意类型的参数
int x = f(1);                 // 这个 f() 接受 int
int z = f(complex<double>{1,2}); // 这个 f() 接受 complex<double>
```

关键字 `auto` 表示对值约束最少的概念：它仅仅要求它必须是某个类型的值。采用 `auto` 参数使得函数成为函数模板。

有了概念，我们可以通过在 `auto` 前面加上概念来加强所有这些初始化的要求。例如：

```cpp
auto twice(Arithmetic auto x) { return x + x; }   // 仅用于数字
auto thrice(auto x) { return x + x + x; }         // 用于任何有 + 的

auto x1 = twice(7);           // OK：x1==14
string s = "Hello";
auto x2 = twice(s);           // 错误：string 不是 Arithmetic
auto x3 = thrice(s);          // OK：x3=="HelloHelloHello"
```

除了用于约束函数参数，概念还可以约束变量的初始化：

```cpp
auto ch1 = open_channel("foo");              // 无论 open_channel 返回什么都行
Arithmetic auto ch2 = open_channel("foo");   // 错误：channel 不是 Arithmetic
Channel auto ch3 = open_channel("foo");      // OK：假设 Channel 是 API 概念
                                             // 并且 open_channel() 返回一个 Channel
```

这对于对抗过度使用 `auto` 以及记录使用泛型函数的代码的要求非常方便。

为了可读性和调试，通常重要的是类型错误在尽可能接近其起源的地方被捕获。约束返回类型可以有所帮助：

```cpp
Number auto some_function(int x)
{
    // return fct(x);   // 除非 fct(x) 返回 Number，否则错误
    // ...
}
```

当然，我们可以通过引入一个局部变量来实现这一点：

```cpp
auto some_function(int x)
{
    Number auto y = fct(x);   // 除非 fct(x) 返回 Number，否则错误
    return y;
}
```

然而，这有点冗长，并且并非所有类型都可以廉价拷贝。

## 8.2.6 概念与类型

**类型**
- 指定可以应用于对象的操作集（隐式和显式）
- 依赖于函数声明和语言规则
- 指定对象在内存中的布局

**单参数概念**
- 指定可以应用于对象的操作集（隐式和显式）
- 依赖于反映函数声明和语言规则的使用模式
- 对对象的布局只字不提
- 启用一组类型的使用

因此，用概念约束代码比用类型约束更灵活。此外，概念可以定义多个参数之间的关系。我的理想是，最终大多数函数将被定义为模板函数，其参数由概念约束。不幸的是，对此的符号支持尚不完美：我们必须将概念用作形容词，而不是名词。例如：

```cpp
void sort(Sortable auto&);   // 需要 'auto'
void sort(Sortable&);        // 错误：概念名称后需要 'auto'
```
