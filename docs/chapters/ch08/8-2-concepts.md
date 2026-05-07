# 8.2 概念

考虑 §7.3.1 中的 `sum()`：

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

- 一个序列 `Seq`，支持 `begin()` 和 `end()` 以便 range-for 能够工作（§1.7；§14.1）。
- 一个算术类型 `Value`，支持 `+=` 以便序列的元素可以被相加。

我们称这样的要求为**概念**。

满足这个简化要求（以及更多）的类型示例，作为序列（也称为 range）的标准库 `vector`、`list` 和 `map`。满足这个简化要求（以及更多）的算术类型示例包括 `int`、`double` 和 `Matrix`（对于任何合理的 `Matrix` 定义）。我们可以说 `sum()` 算法在两个维度上是通用的：用于存储元素的数据结构类型（“序列”）和元素的类型。

#### 8.2.1 概念的使用

大多数模板参数必须满足特定的要求，模板才能正确编译，生成的代码才能正确工作。也就是说，大多数模板应该是**受约束模板**（§7.2.1）。类型名引入符 `typename` 是约束最少的，只要求参数是一个类型。通常，我们可以做得比这更好。再次考虑 `sum()`：

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

序列的 `range_value_t`（§16.4.4）是该序列中元素的类型；它来自标准库，其中它命名了 range 中元素的类型（§14.1）。`Arithmetic<X, Y>` 是一个概念，指定我们可以使用 `X` 和 `Y` 类型的数字进行算术运算。这使我们免于意外尝试计算 `vector<string>` 或 `vector<int>` 的和，同时仍然接受 `vector<int>` 和 `vector<complex<double>>`。通常，当一个算法需要不同类型的参数时，这些类型之间存在某种关系，最好明确这一点。

在这个例子中，我们只需要 `+=`，但为了简单和灵活，我们不应该过于严格地约束模板参数。特别地，有一天我们可能希望用 `+` 和 `=` 而不是 `+=` 来表达 `sum()`，那时我们会很高兴我们使用了一个通用的概念（此处为 `Arithmetic`），而不是一个狭隘的“拥有 `+=`”的要求。

部分规范，如第一个使用概念的 `sum()`，可能非常有用。除非规范完整，否则某些错误要到实例化时才会被发现。然而，即使是部分规范也表达了意图，对于我们在最初并未认识到所有必要需求的平滑增量开发至关重要。有了成熟的概念库，初始规范将接近完美。

===== 第 5 页 =====

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

无论选择什么表示法，设计一个对其参数具有语义上有意义的约束的模板非常重要（§8.2.4）。

#### 8.2.2 基于概念的重载

一旦我们正确地使用接口指定了模板，就可以像对函数一样，根据它们的属性进行重载。考虑一个稍微简化的标准库函数 `advance()`，它用于推进迭代器（§13.3）：

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

像其他重载一样，这是一种编译时机制，意味着没有运行时成本，当编译器找不到最佳选择时，它会给出歧义错误。基于概念的重载的规则比一般重载的规则（§1.3）简单得多。首先考虑单个参数在几个备选函数中的情况：

- 如果参数不匹配概念，则无法选择该备选。
- 如果参数仅匹配一个备选的概念，则选择该备选。
- 如果来自两个备选的参数都匹配某个概念，并且其中一个比另一个更严格（满足另一个的所有要求以及更多），则选择该备选。
- 如果来自两个备选的参数在概念上匹配得同样好，则存在歧义。

要选择某个备选，它必须：
- 匹配其所有参数，并且
- 对于所有参数，至少与其它备选匹配得一样好，并且

===== 第 7 页 =====

- 对于至少一个参数匹配得更好。

#### 8.2.3 有效代码

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

优先使用具有良好规范语义的命名概念（§8.2.4），并且主要在定义这些概念时使用 requires 表达式。

#### 8.2.4 概念的定义

我们在库（包括标准库）中找到了有用的概念，例如 `forward_iterator`（§14.5）。与类和函数一样，使用好库中的概念通常比编写新概念更容易，但简单的概念并不难定义。来自标准库的名称，如 `random_access_iterator` 和 `vector`，是小写的。这里，我使用大写字母来命名我自己定义的概念，例如 `Sequence` 和 `Vector`。

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

不幸的是，标准库中没有 `boolean` 概念，所以我定义了一个（§14.5）。`Boolean` 只是指可以用作条件的类型。

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

这个 `Equality_comparable` 与标准库的 `equality_comparable`（§14.5）几乎相同。

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

这没有对结果类型做出假设，但对于简单用途来说已经足够了。给定一个参数类型，`Number<X>` 检查 `X` 是否具有 `Number` 所需的属性。给定两个参数，`Number<X,Y>` 检查两种类型是否可以一起使用所需的操作。由此，我们可以定义我们的 `Arithmetic` 概念（§8.2.1）：

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

对于一个类型 `S` 要成为 `Sequence`，它必须提供一个值类型（其元素的类型；见 §13.1）和一个迭代器类型（其迭代器的类型）。这里，我使用了标准库的关联类型 `range_value_t<S>` 和 `iterator_t<S>`（§16.4.4）来表达这一点。它还必须确保存在返回 `S` 的迭代器的 `begin()` 和 `end()` 函数，这是标准库容器的惯用法（§12.3）。最后，`S` 的迭代器类型必须至少是一个 `input_iterator`，并且元素的值类型和迭代器的值类型必须相同。

===== 第 11 页 =====

最难定义的概念是那些代表基本语言概念的概念。因此，最好使用来自成熟库的集合。有关有用的集合，请参见 §14.5。特别地，有一个标准库概念允许我们绕过 `Sequence` 定义的复杂性：

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

#### 8.2.4.1 定义检查

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

在开发和维护大型代码库时，这两点都很重要。我们为这一重要好处付出的代价是，某些错误（例如在只保证 `==` 的地方使用了 `<`）会在编译过程的后期才被捕获（§8.5）。

#### 8.2.5 概念与 auto

关键字 `auto` 可以用来表示对象应该具有其初始化器的类型（§1.4.2）：

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

===== 第 13 页 =====

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

#### 8.2.6 概念与类型

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
