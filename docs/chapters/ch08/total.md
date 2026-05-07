===== 第 1 页 =====

8

# 概念与泛型编程

编程：你必须从有趣的算法开始。 – Alex Stepanov

- 引言
- 概念
  - 概念的使用；基于概念的重载；有效代码；概念的定义；概念与 auto；概念与类型
- 泛型编程
  - 概念的使用；使用模板进行抽象
- 变参模板

===== 第 2 页 =====

  - 折叠表达式；转发参数
- 模板编译模型
- 建议

### 8.1 引言

模板是用来做什么的？换句话说，模板使哪些编程技术变得有效？模板提供了：

- 将类型（以及值和模板）作为参数传递而不丢失信息的能力。这意味着在可表达的内容上具有极大的灵活性，并且有极好的内联机会，当前的实现充分利用了这一点。
- 在实例化时将来自不同上下文的信息编织在一起的机会。这意味着优化机会。
- 将值作为模板参数传递的能力。这意味着编译时计算的机会。

换句话说，模板提供了一种强大的编译时计算和类型操作机制，可以产生非常紧凑高效的代码。请记住，类型（类）可以同时包含代码（§7.3.2）和值（§7.2.2）。

模板的第一个也是最常见的用途是支持**泛型编程**，即专注于设计、实现和使用通用算法的编程。这里，“通用”意味着算法可以被设计为接受各种各样的类型，只要它们满足算法对其参数的要求。与概念一起，模板是 C++ 对泛型编程的主要支持。模板提供（编译时）参数多态。

### 8.2 概念

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

### 8.3 泛型编程

===== 第 15 页 =====

C++ 直接支持的泛型编程形式围绕着从具体、高效的算法中抽象以获得通用算法的思想，这些通用算法可以与不同的数据表示相结合，产生各种有用的软件 [Stepanov,2009]。表示基本操作和数据结构的抽象称为**概念**。

#### 8.3.1 概念的使用

好的、有用的概念是基本的，更多是被发现而不是被设计的。例子包括整数和浮点数（甚至在经典 C [Kernighan,1978] 中定义的）、序列，以及更一般的数学概念，如环和向量空间。它们代表了一个应用领域的基本概念。这就是为什么它们被称为“概念”。将概念识别和形式化到有效泛型编程所需的程度可能是一个挑战。

对于基本使用，考虑概念 `regular`（§14.5）。当一个类型的行为很像 `int` 或 `vector` 时，它就是 regular 的。一个 regular 类型的对象：
- 可以默认构造。
- 可以使用构造函数或赋值进行拷贝（具有拷贝的常规语义，产生两个独立且比较相等的对象）。
- 可以使用 `==` 和 `!=` 进行比较。
- 不会因过于聪明的编程技巧而遭受技术问题。

`string` 是 regular 类型的另一个例子。与 `int` 一样，`string` 也是 `totally_ordered`（§14.5）。也就是说，可以使用 `<`、`<=`、`>`、`>=` 和 `<=>` 以适当的语义比较两个字符串。

概念不仅仅是句法上的概念，它本质上是关于语义的。例如，不要定义 `+` 来做除法；那不符合任何合理数字的要求。不幸的是，我们还没有任何语言支持来表达语义，因此我们必须依靠专家知识和常识来获得语义上有意义的概念。不要定义语义上无意义的概念，例如 `Addable` 和 `Subtractable`。

===== 第 16 页 =====

#### 8.3.2 使用模板进行抽象

好的抽象是从具体的例子中仔细培养出来的。试图通过为每一个可以想到的需求和技术做准备来进行“抽象”并不是一个好主意；那会导致不优雅和代码膨胀。相反，从一个——最好多个——实际使用中的具体例子开始，并尝试消除无关紧要的细节。考虑：

```cpp
double sum(const vector<int>& v)
{
    double res = 0;
    for (auto x : v)
        res += x;
    return res;
}
```

这显然是计算数字序列之和的众多方法之一。

考虑是什么使得这段代码比它本应有的更不通用：

- 为什么只是 `int`？
- 为什么只是 `vector`？
- 为什么累加在 `double` 中？
- 为什么从 0 开始？
- 为什么是加法？

通过将具体类型变为模板参数来回答前四个问题，我们得到了标准库 `accumulate` 算法的最简单形式：

```cpp
template<forward_iterator Iter, Arithmetic<iter_value_t<Iter>> Val>
Val accumulate(Iter first, Iter last, Val res)
{
    for (auto p = first; p != last; ++p)
        res += *p;
    return res;
}
```

这里：
- 要遍历的数据结构已被抽象为一对表示序列的迭代器（§8.2.4，§13.1）。
- 累加器的类型已被设为参数。
- 累加器的类型必须是算术类型。
- 累加器的类型必须与迭代器的值类型（序列的元素类型）一起工作。
- 初始值现在是一个输入；累加器的类型就是这个初始值的类型。

快速检查或者——甚至更好——测量将表明，为各种数据结构调用生成的代码与手工编码示例得到的代码相同。考虑：

```cpp
void use(const vector<int>& vec, const list<double>& lst)
{
    auto sum = accumulate(begin(vec), end(vec), 0.0);   // 在 double 中累加
    auto sum2 = accumulate(begin(lst), end(lst), sum);  // ...
}
```

从具体代码片段（最好从多个）中泛化同时保持性能的过程称为**提升**。反过来，开发模板的最佳方法通常是：
1.  首先，编写一个具体版本
2.  然后，调试、测试和测量它
3.  最后，用模板参数替换具体类型。

当然，重复 `begin()` 和 `end()` 很繁琐，因此我们可以稍微简化用户界面：

```cpp
template<forward_range R, Arithmetic<value_type_t<R>> Val>
Val accumulate(const R& r, Val res = 0)
{
    for (auto x : r)
        res += x;
    return res;
}
```

`range` 是一个标准库概念，表示具有 `begin()` 和 `end()` 的序列（§13.1）。为了完全通用，我们也可以抽象 `+=` 操作；见 §17.3。

`accumulate()` 的迭代器对版本和 range 版本都很有用：迭代器对版本用于通用性，range 版本用于常见用法的简单性。

### 8.4 变参模板

模板可以定义为接受任意数量的任意类型的参数。这样的模板称为**变参模板**。考虑一个简单的函数，用于写出任何具有 `<<` 运算符的类型的值：

```cpp
void user()
{
    print("first: ", 1, 2.2, "hello\n");   // first: 1 2.2 hello
    print("\nsecond: ", 0.2, 'c', "yuck!"s, 0, 1, 2, '\n'); // second: 0.2 c yuck! 0 1 2
}
```

传统上，实现变参模板的方法是将第一个参数与其余参数分开，然后对参数包的尾部递归调用变参模板：

```cpp
template<typename T>
concept Printable = requires(T t) { std::cout << t; };   // 仅一个操作！

void print() { }   // 对于没有参数的情况：什么都不做

template<Printable T, Printable... Tail>
void print(T head, Tail... tail)
{
    cout << head << ' ';
    print(tail...);
}
```

`Printable...` 表示 `Tail` 是一个类型序列。`Tail...` 表示 `tail` 是一个值序列，这些值的类型在 `Tail` 中。用 `...` 声明的形参称为**参数包**。这里，`tail` 是一个（函数参数）参数包，其中的元素类型在（模板参数）参数包 `Tail` 中找到。因此，`print()` 可以接受任意数量的任意类型的参数。

调用 `print()` 将参数分为头部（第一个）和尾部（其余）。打印头部，然后对尾部调用 `print()`。最终，尾部当然会变空，因此我们需要无参数的 `print()` 版本来处理这种情况。如果我们不允许零参数的情况，可以使用编译时 `if` 来消除那个 `print()`：

```cpp
template<Printable T, Printable... Tail>
void print(T head, Tail... tail)
{
    cout << head << ' ';
    if constexpr (sizeof...(tail) > 0)
        print(tail...);
}
```

我使用了编译时 `if`（§7.4.3）而不是普通的运行时 `if`，以避免生成最终的 `print()` 调用。这样，“空”的 `print()` 就不必定义了。

变参模板的优势在于它们可以接受你愿意给它们的任何参数。缺点包括：
- 递归实现可能很难正确。
- 接口的类型检查可能是一个复杂的模板程序。
- 类型检查代码是临时的，而不是在标准中定义的。
- 递归实现可能在编译时间和编译器内存需求方面出奇地昂贵。

由于它们的灵活性，变参模板在标准库中被广泛使用，偶尔也被过度使用。

===== 第 20 页 =====

#### 8.4.1 折叠表达式

为了简化简单变参模板的实现，C++ 提供了对参数包元素进行有限形式的迭代。例如：

```cpp
template<Number... T>
int sum(T... v)
{
    return (v + ... + 0);   // 从 0 开始添加 v 的所有元素
}
```

这个 `sum()` 可以接受任意数量的任意类型的参数：

```cpp
int x = sum(1, 2, 3, 4, 5);   // x 变成 15
int y = sum('a', 2.4, x);     // y 变成 114（2.4 被截断，'a' 的值是 97）
```

`sum` 的主体使用了一个**折叠表达式**：

```cpp
return (v + ... + 0);   // 将 v 的所有元素加到 0 上
```

这里，`(v + ... + 0)` 表示从初始值 0 开始，添加 `v` 的所有元素。第一个被添加的元素是“最右边”的（索引最高的）：`(v[0] + (v[1] + (v[2] + (v[3] + (v[4] + 0)))))`。也就是说，从 0 所在的右边开始。这称为**右折叠**。或者，我们可以使用左折叠：

```cpp
template<Number... T>
int sum2(T... v)
{
    return (0 + ... + v);   // 将 v 的所有元素加到 0 上
}
```

现在，第一个被添加的元素是“最左边”的（索引最低的）：`((((0+v[0])+v[1])+v[2])+v[3])+v[4]`。也就是说，从 0 所在的左边开始。

折叠是一个非常强大的抽象，显然与标准库的 `accumulate()` 相关，在不同的语言和社区中有不同的名称。在 C++ 中，折叠表达式目前仅限于简化变参模板的实现。折叠不必执行数值计算。考虑一个著名的例子：

```cpp
template<Printable... T>
void print(T&... args)
{
    (std::cout << ... << args) << '\n';   // 打印所有参数
}

print("Hello!"s, ' ', "World ", 2017);   // (((std::cout << "Hello!"s) << ' ') << "World ") << 2017
```

为什么是 2017？因为 `fold()` 是在 2017 年添加到 C++ 中的（§19.2.3）。

#### 8.4.2 转发参数

通过接口原封不动地传递参数是变参模板的一个重要用途。考虑一个网络输入通道的概念，其中实际的值传递方法是参数化的。不同的传输机制有不同的构造函数参数集：

```cpp
template<concepts::InputTransport Transport>
class InputChannel {
public:
    // ...
    InputChannel(Transport::Args&&... transportArgs)
        : _transport(std::forward<TransportArgs>(transportArgs)...)
    { }
private:
    Transport _transport;
};
```

标准库函数 `forward()`（§16.6）用于将参数从 `InputChannel` 构造函数原封不动地移动到 `Transport` 构造函数。

这里的要点是，`InputChannel` 的编写者可以在不知道构造特定 `Transport` 需要什么参数的情况下构造一个 `Transport` 类型的对象。`InputChannel` 的实现者只需要知道所有 `Transport` 对象的共同用户接口。

转发在基础库中非常常见，在这些库中，通用性和低运行时开销是必需的，并且非常通用的接口很常见。

### 8.5 模板编译模型

在使用点，模板的参数会根据其概念进行检查。此处发现的错误将立即报告。此时无法检查的内容（例如无约束模板参数的参数）将推迟到为模板生成代码时，即“模板实例化时”。

实例化时类型检查的一个不幸的副作用是，类型错误可能被检测得过晚（§8.2.4.1）。此外，延迟检查通常会导致非常糟糕的错误消息，因为编译器没有类型信息来提示程序员的意图，并且通常在组合了程序中多个地方的信息后才检测到问题。

为模板提供的实例化时类型检查会检查模板定义中参数的使用。这提供了通常称为**鸭子类型**的编译时变体（“如果它像鸭子一样走路，像鸭子一样叫，那它就是鸭子”）。或者——使用更技术的术语——我们操作的是值，操作的存在和意义完全取决于其操作数值。这与另一种观点不同，即对象具有类型，类型决定了操作的存在和意义。值“存在于”对象中。这就是对象（例如变量）在 C++ 中的工作方式，只有满足对象要求的值才能放入其中。使用模板在编译时完成的工作大多不涉及对象，只涉及值。例外是 `constexpr` 函数（§1.6）中的局部变量，它们在编译器内部用作对象。

要使用无约束模板，其定义（不仅仅是声明）必须在其使用点处于作用域内。当使用头文件和 `#include` 时，这意味着模板定义位于头文件中，而不是 `.cpp` 文件中。例如，标准头文件 `<vector>` 包含了 `vector` 的定义。

===== 第 23 页 =====

当我们开始使用模块时，这种情况会改变（§3.2.2）。使用模块时，源代码可以以与普通函数和模板函数相同的方式组织。模块被半编译成一种表示形式，使其能够快速导入和使用。可以将这种表示形式视为一个易于遍历的图，包含所有可用的作用域和类型信息，并由符号表支持，允许快速访问各个实体。

### 8.6 建议

[1] 模板提供了一种用于编译时编程的通用机制；§8.1。
[2] 在设计模板时，仔细考虑为其模板参数假定的概念（要求）；§8.3.2。
[3] 在设计模板时，使用具体版本进行初始实现、调试和测量；§8.3.2。
[4] 将概念用作设计工具；§8.2.1。
[5] 为所有模板参数指定概念；§8.2；[CG: T.10]。
[6] 尽可能使用命名概念（例如标准库概念）；§8.2.4，§14.5；[CG: T.11]。
[7] 如果你只需要在一个地方使用简单的函数对象，请使用 lambda；§7.3.2。
[8] 使用模板来表达容器和范围；§8.3.2；[CG: T.3]。
[9] 避免没有有意义语义的“概念”；§8.2；[CG: T.20]。
[10] 要求概念拥有完整的操作集；§8.2；[CG: T.21]。
[11] 使用命名概念；§8.2.3。
[12] 避免 `requires requires`；§8.2.3。
[13] `auto` 是约束最少的概念；§8.2.5。
[14] 当你需要一个接受可变数量各种类型参数的函数时，使用变参模板；§8.4。
[15] 模板提供编译时的“鸭子类型”；§8.5。
[16] 当使用头文件时，在使用模板的每个翻译单元中 `#include` 模板定义（而不仅仅是声明）；§8.5。

===== 第 24 页 =====

[17] 要使用模板，请确保其定义（而不仅仅是声明）在作用域内；§8.5。
[18] 无约束模板提供编译时的“鸭子类型”；§8.5。