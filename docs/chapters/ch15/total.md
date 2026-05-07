===== 第 1 页 =====

15

# 指针与容器

教育是告诉你做什么、何时做以及为什么做。
训练是告诉你如何做。 – 理查德·汉明

- 引言
- 指针
  - unique_ptr 和 shared_ptr；span
- 容器
  - array；bitset；pair；tuple
- 替代方案
  - variant；optional；any

===== 第 2 页 =====

### 15.1 引言

C++ 提供了简单的内置低级类型来保存和引用数据：对象和数组保存数据；指针和数组引用这些数据。然而，我们需要支持更多样化和更通用的方式来保存和使用数据。例如，标准库容器（第 12 章）和迭代器（§13.3）被设计用来支持通用算法。

容器和指针抽象的主要共同点是：它们的正确高效使用需要将数据与一组访问和操作它们的函数封装在一起。例如，指针是对机器地址的非常通用且高效的抽象，但正确使用它们来表示资源的所有权已被证明极其困难。因此，标准库提供了资源管理指针，即封装指针并提供简化其正确使用的操作的类。

这些标准库抽象封装了内置语言类型，并且在时间和空间上的性能要求与正确使用这些类型时相同。

这些类型并没有什么“魔法”。我们可以使用与标准库相同的技术，根据需要设计并实现自己的“智能指针”和专用容器。

## 15.2 指针

指针的一般概念是允许我们引用一个对象并根据其类型访问该对象。内置指针（如 `int*`）是一个例子，但还有更多。

**指针类型**

| 类型 | 描述 |
|------|------|
| `T*` | 内置指针类型：指向一个类型为 `T` 的对象，或指向一个连续分配的 `T` 类型元素序列 |
| `T&` | 内置引用类型：引用一个类型为 `T` 的对象；一种隐式解引用的指针（§1.7） |
| `unique_ptr<T>` | 指向 `T` 的拥有指针 |
| `shared_ptr<T>` | 指向类型 `T` 的对象的指针；所有权由所有指向该 `T` 的 `shared_ptr` 共享 |
| `weak_ptr<T>` | 指向由 `shared_ptr` 拥有的对象的指针；必须转换为 `shared_ptr` 才能访问该对象 |
| `span<T>` | 指向连续 `T` 序列的指针（§15.2.2） |
| `string_view<T>` | 指向常量子串的指针（§10.3） |
| `X_iterator<C>` | 来自 `C` 的元素序列；名称中的 `X` 表示迭代器的种类（§13.3） |

可能有多个指针指向同一个对象。**拥有指针**是指最终负责删除其所指对象的指针。**非拥有指针**（例如 `T*` 或 `span`）可能悬空，即指向一个对象已被删除或已超出作用域的位置。

通过悬空指针进行读写是最严重的错误之一。这样做的结果在技术上是未定义的。在实践中，这通常意味着访问恰好占据该位置的对象。此时，读操作会得到一个任意值，写操作则会扰乱一个不相关的数据结构。我们最好的希望是程序崩溃；这通常比错误的结果更可取。

C++ 核心指南 [CG] 提供了避免这种情况的规则以及静态检查以确保其永不发生的建议。然而，这里有一些避免指针问题的方法：

- 在局部对象超出作用域后，不要保留指向它的指针。特别地，永远不要从函数返回指向局部对象的指针，也不要在长期存在的数据结构中存储来源不确定的指针。系统性地使用容器和算法（第 12 章、第 13 章）常常能让我们避免采用那些难以避免指针问题的编程技术。
- 对在自由存储区上分配的对象使用拥有指针。
- 指向静态对象（例如全局变量）的指针不会悬空。
- 将指针算术留给资源句柄（如 `vector` 和 `unordered_map`）的实现。
- 记住 `string_view` 和 `span` 是非拥有指针的一种。

#### 15.2.1 unique_ptr 和 shared_ptr

任何非平凡程序的关键任务之一是管理资源。资源是必须获取并在之后（显式或隐式）释放的东西。例如内存、锁、套接字、线程句柄和文件句柄。对于长时间运行的程序，未能及时释放资源（“泄漏”）会导致严重的性能下降（§12.7），甚至可能崩溃。即使对于短程序，泄漏也可能成为尴尬的问题，例如导致资源短缺，使运行时间增加几个数量级。

标准库组件被设计为不会泄漏资源。为此，它们依赖基本的语言支持，使用构造函数/析构函数对来确保资源不会超过负责它的对象的寿命。`Vector` 中使用构造函数/析构函数对来管理其元素的生命周期就是一个例子（§5.2.2），所有标准库容器都以类似的方式实现。重要的是，这种方法能正确地与使用异常的出错处理交互。例如，此技术用于标准库的锁类：

```cpp
mutex m;   // 用于保护对共享数据的访问

void f()
{
    scoped_lock lck {m};   // 获取互斥量 m
    // ... 操作共享数据 ...
}   // 隐式释放互斥量
```

===== 第 5 页 =====

线程会阻塞，直到 `lck` 的构造函数获取了互斥量（§18.3）。相应的析构函数释放互斥量。因此，在这个例子中，当控制线程离开 `f()`（通过 `return`、通过“函数结束”或通过抛出异常）时，`scoped_lock` 的析构函数会释放互斥量。

这是 RAII（“资源获取即初始化”技术；§5.2.2）的一个应用。RAII 是 C++ 中惯用的资源处理的基础。容器（如 `vector`、`map`、`string` 和 `iostream`）以类似方式管理它们的资源（如文件句柄和缓冲区）。

到目前为止的例子处理了在作用域内定义的对象，在离开作用域时释放它们所获取的资源，但在自由存储区上分配的对象呢？在 `<memory>` 中，标准库提供了两种“智能指针”来帮助管理自由存储区上的对象：

- `unique_ptr` 表示唯一所有权（其析构函数销毁其对象）
- `shared_ptr` 表示共享所有权（最后一个共享指针的析构函数销毁该对象）

这些“智能指针”最基本的用途是防止因粗心编程导致的内存泄漏。例如：

```cpp
void f(int i, int j)   // X* vs. unique_ptr<X>
{
    X* p = new X;               // 分配一个新的 X
    unique_ptr<X> sp {new X};   // 分配一个新的 X 并将其指针交给 unique_ptr
    // ...
    if (i < 99) throw Z{};      // 可能抛出异常
    if (j < 77) return;         // 可能“提前”返回
    // ... 使用 p 和 sp ..
    delete p;                   // 销毁 *p
}
```

这里，如果 `i<99` 或 `j<77`，我们“忘记”了删除 `p`。另一方面，无论我们以何种方式退出 `f()`（通过抛出异常、执行 `return` 或“函数结束”），`unique_ptr` 都能确保其对象被正确销毁。具有讽刺意味的是，我们本可以通过不使用指针和不使用 `new` 来简单解决问题：

```cpp
void f(int i, int j)   // 使用局部变量
{
    X x;
    // ...
}
```

===== 第 6 页 =====

不幸的是，过度使用 `new`（以及指针和引用）似乎是一个日益严重的问题。

然而，当你确实需要指针的语义时，与正确使用内置指针相比，`unique_ptr` 是一种轻量级机制，没有空间或时间开销。它的进一步用途包括在函数之间传递自由存储区上分配的对象：

```cpp
unique_ptr<X> make_X(int i)   // 创建一个 X 并立即将其交给 unique_ptr
{
    // ... 检查 i 等 ...
    return unique_ptr<X>{new X{i}};
}
```

`unique_ptr` 是单个对象（或数组）的句柄，就像 `vector` 是对象序列的句柄一样。两者都控制其他对象的生命周期（使用 RAII），并且都依赖消除拷贝或使用移动语义来使返回变得简单高效（§6.2.2）。

`shared_ptr` 类似于 `unique_ptr`，区别在于 `shared_ptr` 是被拷贝而不是被移动。指向一个对象的 `shared_ptr` 共享该对象的所有权；当最后一个 `shared_ptr` 被销毁时，该对象被销毁。例如：

```cpp
void f(shared_ptr<fstream>);
void g(shared_ptr<fstream>);

void user(const string& name, ios_base::openmode mode)
{
    shared_ptr<fstream> fp {new fstream(name, mode)};
    if (!*fp)   // 确保文件被正确打开
        throw No_file{};
    f(fp);
    g(fp);
    // ...
}
```

===== 第 7 页 =====

现在，`fp` 的构造函数打开的文件将由最后一个（显式或隐式）销毁 `fp` 副本的函数关闭。注意，`f()` 或 `g()` 可能会创建一个持有 `fp` 副本的任务，或以其他方式存储一个比 `user()` 寿命更长的副本。因此，`shared_ptr` 提供了一种垃圾回收形式，它尊重内存管理对象的基于析构函数的资源管理。这既不是零成本也不是极其昂贵，但它确实使得共享对象的生命周期难以预测。仅在你确实需要共享所有权时才使用 `shared_ptr`。

在自由存储区上创建一个对象，然后将指向它的指针传递给智能指针，这有点冗长。它也容易出错，例如忘记将指针传递给 `unique_ptr`，或者将指向不在自由存储区上的东西的指针传递给 `shared_ptr`。为了避免这些问题，标准库（在 `<memory>` 中）提供了用于构造对象并返回适当智能指针的函数：`make_shared()` 和 `make_unique()`。例如：

```cpp
struct S {
    int i;
    string s;
    double d;
    // ...
};

auto p1 = make_shared<S>(1, "Ankh Morpork", 4.65);   // p1 是一个 shared_ptr<S>
auto p2 = make_unique<S>(2, "Oz", 7.62);             // p2 是一个 unique_ptr<S>
```

现在，`p2` 是一个 `unique_ptr<S>`，指向一个自由存储区上分配的 `S` 类型对象，其值为 `{2, "Oz"s, 7.62}`。

使用 `make_shared()` 不仅比单独使用 `new` 创建对象然后将其传递给 `shared_ptr` 更方便，而且效率也显著更高，因为它不需要为 `shared_ptr` 实现中至关重要的使用计数进行单独分配。

===== 第 8 页 =====

有了 `unique_ptr` 和 `shared_ptr`，我们可以为许多程序实现完全的“无裸露 new”策略（§5.2.2）。然而，这些“智能指针”在概念上仍然是指针，因此仅是我在资源管理中的第二选择——仅次于在更高概念层次上管理其资源的容器和其他类型。特别地，`shared_ptr` 本身并不为哪些所有者可以读和/或写共享对象提供任何规则。仅仅消除资源管理问题并不能解决数据竞争（§18.5）和其他形式的混乱。

我们何时使用“智能指针”（如 `unique_ptr`）而不是具有专门为该资源设计的操作的资源句柄（如 `vector` 或 `thread`）？不出所料，答案是“当我们需要指针语义时”。

- 当我们共享一个对象时，我们需要指针（或引用）来引用该共享对象，因此 `shared_ptr` 成为显而易见的选择（除非有一个明显的单一所有者）。
- 在经典的面向对象代码（§5.5）中引用多态对象时，我们需要指针（或引用），因为我们不知道所指对象的确切类型（甚至不知道其大小），因此 `unique_ptr` 成为显而易见的选择。
- 一个共享的多态对象通常需要 `shared_ptr`。

我们不需要使用指针从函数返回对象集合；作为资源句柄的容器可以通过依赖拷贝省略（§3.4.2）和移动语义（§6.2.2）简单而高效地做到这一点。

## 15.2.2 span

传统上，范围错误一直是 C 和 C++ 程序中严重错误的主要来源，导致错误结果、崩溃和安全问题。容器（第 12 章）、算法（第 13 章）和范围 `for` 的使用已显著减少了这个问题，但还可以做得更多。范围错误的一个关键来源是人们传递指针（原始或智能）然后依赖约定来知道所指元素的数量。对于资源句柄之外的代码，最好的建议是假设最多只有一个对象被指向 [CG: F.22]，但如果没有支持，这条建议是难以管理的。标准库的 `string_view`（§10.3）可以提供帮助，但它是只读的且仅适用于字符。大多数程序员需要更多。例如，在较低级软件中写入和读出缓冲区时，众所周知，要在避免范围错误（“缓冲区溢出”）的同时保持高性能是非常困难的。`<span>` 中的 `span` 基本上是一个（指针，长度）对，表示一个元素序列：

[图片描述：span<int> 示意图]

`span` 提供了对连续元素序列的访问。元素可以以多种方式存储，包括在 `vector` 和内置数组中。像指针一样，`span` 不拥有它指向的字符。在这方面，它类似于 `string_view`（§10.3）和 STL 的迭代器对（§13.3）。

考虑一种常见的接口风格：

```cpp
void fpn(int* p, int n)
{
    for (int i = 0; i < n; ++i)
        p[i] = 0;
}
```

我们假设 `p` 指向 `n` 个整数。不幸的是，这个假设仅仅是一种约定，因此我们不能用它来编写范围 `for` 循环，编译器也无法实现廉价且有效的范围检查。此外，我们的假设可能是错误的：

```cpp
void use(int x)
{
    int a[100];
    fpn(a, 100);      // OK
    fpn(a, 1000);     // 糟糕，我手滑了！（fpn 中的范围错误）
    fpn(a+10, 100);   // fpn 中的范围错误
    fpn(a, x);        // 可疑，但看起来无害
}
```

===== 第 10 页 =====

使用 `span` 我们可以做得更好：

```cpp
void fs(span<int> p)
{
    for (int& x : p)
        x = 0;
}
```

我们可以像这样使用 `fs`：

```cpp
void use(int x)
{
    int a[100];
    fs(a);                     // 隐式创建 span<int>{a,100}
    fs(a, 1000);               // 错误：期望一个 span
    fs({a+10, 100});           // fs 中的范围错误
    fs({a, x});                // 明显可疑
}
```

也就是说，常见的情况——直接从数组创建 `span`——现在是安全的（编译器计算元素个数）且符号简单。在其他情况下，出错的可能性降低了，错误检测也更容易了，因为程序员必须显式地构造一个 `span`。

在函数之间传递 `span` 的常见情况比（指针，计数）接口更简单，并且显然不需要额外的检查：

```cpp
void f1(span<int> p);

void f2(span<int> p)
{
    // ...
    f1(p);
}
```

与容器一样，当 `span` 用于下标（例如 `r[i]`）时，不进行范围检查，越界访问是未定义行为。当然，实现可以将该未定义行为实现为范围检查，但遗憾的是很少有人这样做。来自核心指南支持库 [CG] 的原始 `gsl::span` 确实执行范围检查。

===== 第 11 页 =====

## 15.3 容器

标准库提供了几个不能完美融入 STL 框架（第 12 章，第 13 章）的容器。例如内置数组、`array` 和 `string`。我有时称它们为“准容器”，但这并不完全公平：它们容纳元素，因此是容器，但每个都有一些限制或附加功能，使它们在 STL 的上下文中不太方便。单独描述它们也简化了 STL 的描述。

**容器**

| 类型 | 描述 |
|------|------|
| `T[N]` | 内置数组：固定大小的连续分配序列，包含 `N` 个 `T` 类型元素；隐式转换为 `T*` |
| `array<T,N>` | 固定大小的连续分配序列，包含 `N` 个 `T` 类型元素；类似于内置数组，但解决了大部分问题 |
| `bitset<N>` | 固定大小的 `N` 位序列 |
| `vector<bool>` | 在 `vector` 的特化中紧凑存储的位序列 |
| `pair<T,U>` | 两个元素，类型分别为 `T` 和 `U` |
| `tuple<T...>` | 任意数量、任意类型元素的序列 |
| `basic_string<C>` | 类型 `C` 的字符序列；提供字符串操作 |
| `valarray<T>` | 类型 `T` 的数值数组；提供数值运算 |

为什么标准要提供这么多容器？它们服务于常见但不同（常常重叠）的需求。如果标准库不提供它们，许多人将不得不自己设计并实现它们。例如：

===== 第 12 页 =====

- `pair` 和 `tuple` 是异构的；所有其他容器都是同构的（所有元素类型相同）。
- `array` 和 `tuple` 的元素是连续分配的；`list` 和 `map` 是链接结构。
- `bitset` 和 `vector<bool>` 持有位并通过代理对象访问它们；所有其他标准库容器可以持有各种类型并直接访问元素。
- `basic_string` 要求其元素是某种形式的字符，并提供字符串操作，如连接和本地环境敏感操作。
- `valarray` 要求其元素是数字，并提供数值运算。

所有这些容器都可以被视为提供了大型程序员群体所需的专门服务。没有单一的容器能够满足所有这些需求，因为有些需求是相互矛盾的，例如“能够增长” vs “保证分配在固定位置”，以及“添加元素时元素不移动” vs “连续分配”。

### 15.3.1 array

定义在 `<array>` 中的 `array` 是一个固定大小的元素序列，其中元素个数在编译时指定。因此，`array` 可以将其元素分配在栈上、对象内或静态存储中。元素在 `array` 定义所在的作用域内分配。`array` 最好被理解为一个带有固定大小的内置数组，没有隐式的、可能令人惊讶的到指针类型的转换，并提供了一些便利函数。与使用内置数组相比，使用 `array` 没有（时间或空间上的）开销。`array` 不遵循 STL 容器的“指向元素的句柄”模型。相反，`array` 直接包含其元素。它只不过是内置数组的一个更安全的版本。

这意味着 `array` 可以通过初始化列表进行初始化：

```cpp
array<int,3> a1 = {1, 2, 3};
```

===== 第 13 页 =====

初始化列表中的元素个数必须等于或小于为数组指定的元素个数。

元素个数不是可选的，元素个数必须是常量表达式，元素个数必须为正，并且元素类型必须显式指定：

```cpp
void f(int n)
{
    array<int> a0 = {1,2,3};                // 错误：未指定大小
    array<string, n> a1 = {"John's", "Queens' "}; // 错误：大小不是常量表达式
    array<string, 0> a2;                    // 错误：大小必须为正
    array<2> a3 = {"John's", "Queens' "};   // 错误：未指定元素类型
    // ...
}
```

如果你需要元素个数为变量，请使用 `vector`。

必要时，`array` 可以被显式地传递给期望指针的 C 风格函数。例如：

```cpp
void f(int* p, int sz);   // C 风格接口

void g()
{
    array<int,10> a;
    f(a, a.size());       // 错误：没有转换
    f(a.data(), a.size()); // C 风格使用
    auto p = find(a, 777); // C++/STL 风格使用（传递一个范围）
    // ...
}
```

既然 `vector` 如此灵活，为什么还要使用 `array`？`array` 不那么灵活，因此更简单。偶尔，直接访问分配在栈上的元素与在自由存储区上分配元素、通过 `vector`（一个句柄）间接访问它们然后释放它们相比，有显著的性能优势。另一方面，栈是一种有限的资源（尤其是在某些嵌入式系统上），栈溢出是棘手的。此外，有些应用领域（如安全关键的实时控制）禁止使用自由存储区分配。例如，使用 `delete` 可能导致碎片（§12.7）或内存耗尽（§4.3）。

既然可以使用内置数组，为什么还要使用 `array`？`array` 知道它的大小，因此易于与标准库算法一起使用，并且可以使用 `=` 进行拷贝。例如：

```cpp
array<int,3> a1 = {1, 2, 3};
auto a2 = a1;   // 拷贝
a2[1] = 5;
a1 = a2;        // 赋值
```

然而，我偏爱 `array` 的主要原因是它使我免于令人惊讶且糟糕的到指针的转换。考虑一个涉及类层次结构的例子：

```cpp
void h()
{
    Circle a1[10];
    array<Circle,10> a2;
    // ...
    Shape* p1 = a1;   // OK：等待发生的灾难
    Shape* p2 = a2;   // 错误：没有从 array<Circle,10> 到 Shape* 的转换（好！）
    p1[3].draw();     // 灾难
}
```

“灾难”的注释假设 `sizeof(Shape) < sizeof(Circle)`，因此通过 `Shape*` 对 `Circle` 数组进行下标会得到错误的偏移量。所有标准容器都提供了相对于内置数组的这一优势。

### 15.3.2 bitset

系统的某些方面，例如输入流的状态，通常表示为一组指示二进制条件的标志，如 good/bad、true/false、on/off。C++ 通过整数的按位操作高效地支持小标志集的概念（§1.4）。类 `bitset<N>` 泛化了这一概念，提供了对 `N` 位序列 `[0:N)` 的操作，其中 `N` 在编译时已知。对于不适合 `long long int`（通常为 64 位）的位集，使用 `bitset` 比直接使用整数方便得多。对于较小的集合，`bitset` 通常是被优化的。如果你想要为位命名而不是编号，可以使用 `set`（§12.5）或枚举（§2.4）。

`bitset` 可以用整数或字符串初始化：

```cpp
bitset<9> bs1 {"110001111"};
bitset<9> bs2 {0b1'1000'1111};   // 使用数字分隔符的二进制字面量（§1.4）
```

可以应用通常的按位运算符（§1.4）以及左移和右移运算符（`<<` 和 `>>`）：

```cpp
bitset<9> bs3 = ~bs1;   // 补码：bs3 == "001110000"
bitset<9> bs4 = bs1 & bs3;   // 全零
bitset<9> bs5 = bs1 << 2;    // 左移：bs5 = "000111100"
```

移位运算符（此处为 `<<`）会“移入”零。

操作 `to_ullong()` 和 `to_string()` 提供与构造函数相反的操作。例如，我们可以输出一个 `int` 的二进制表示：

```cpp
void binary(int i)
{
    bitset<8*sizeof(int)> b = i;   // 假设 8 位字节（另见 §17.7）
    cout << b.to_string() << '\n'; // 输出 i 的位
}
```

这将从左到右将位表示为 1 和 0，最高有效位在最左边，因此参数 123 将输出：

```
00000000000000000000000001111011
```

对于这个例子，直接使用 `bitset` 的输出运算符更简单：

```cpp
void binary2(int i)
{
    bitset<8*sizeof(int)> b = i;   // 假设 8 位字节（另见 §17.7）
    cout << b << '\n';             // 输出 i 的位
}
```

===== 第 16 页 =====

`bitset` 提供了许多用于使用和操作位集的函数，例如 `all()`、`any()`、`none()`、`count()`、`flip()`。

### 15.3.3 pair

函数返回两个值相当常见。有很多方法可以做到这一点，最简单且通常最好的是为此定义一个 `struct`。例如，我们可以返回一个值和一个成功指示器：

```cpp
struct My_res {
    Entry* ptr;
    Error_code err;
};

My_res complex_search(vector<Entry>& v, const string& s)
{
    Entry* found = nullptr;
    Error_code err = Error_code::found;
    // ... 在 v 中搜索 s ...
    return {found, err};
}

void user(const string& s)
{
    My_res r = complex_search(entry_table, s);   // 搜索 entry_table
    if (r.err != Error_code::good) {
        // ... 处理错误 ...
    }
    // ... 使用 r.ptr ...
}
```

我们可以争辩说将失败编码为尾后迭代器或 `nullptr` 更优雅，但这只能表达一种失败。通常，我们希望返回两个独立的值。为每一对值定义一个特定的具名 `struct` 通常效果很好，如果这些“值对”结构体及其成员的名称选择得当，可读性也相当高。然而，对于大型代码库，这可能导致名称和约定的激增，并且在需要一致命名的泛型代码中效果不佳。因此，标准库提供了 `pair` 作为对“值对”用例的通用支持。使用 `pair`，我们的简单示例变成：

```cpp
pair<Entry*, Error_code> complex_search(vector<Entry>& v, const string& s)
{
    Entry* found = nullptr;
    Error_code err = Error_code::found;
    // ... 在 v 中搜索 s ...
    return {found, err};
}

void user(const string& s)
{
    auto r = complex_search(entry_table, s);   // 搜索 entry_table
    if (r.second != Error_code::good) {
        // ... 处理错误 ...
    }
    // ... 使用 r.first ...
}
```

`pair` 的成员被命名为 `first` 和 `second`。从实现者的角度来看这是有意义的，但在应用程序代码中，我们可能想使用自己的名称。可以使用结构化绑定（§3.4.5）来处理这个问题：

```cpp
void user(const string& s)
{
    auto [ptr, success] = complex_search(entry_table, s);   // 搜索 entry_table
    if (success != Error_code::good) {
        // ... 处理错误 ...
    }
    // ... 使用 ptr ...
}
```

标准库的 `pair`（来自 `<utility>`）在标准库及其他地方非常频繁地用于“值对”的用例。例如，标准库算法 `equal_range` 返回一对指定子序列的迭代器，该子序列满足谓词：

```cpp
template<typename Forward_iterator, typename T, typename Compare>
pair<Forward_iterator, Forward_iterator>
equal_range(Forward_iterator first, Forward_iterator last, const T& val, Compare cmp);
```

给定已排序序列 `[first:last)`，`equal_range()` 将返回表示匹配谓词 `cmp` 的子序列的 pair。我们可以用它来在已排序的 `Record` 序列中搜索：

```cpp
auto less = [](const Record& r1, const Record& r2) { return r1.name < r2.name; };

void f(const vector<Record>& v)   // 假设 v 已按 "name" 字段排序
{
    auto [first, last] = equal_range(v.begin(), v.end(), Record{"Reg"}, less);
    for (auto p = first; p != last; ++p)   // 打印所有相等的记录
        cout << *p;   // 假设为 Record 定义了 <<
}
```

如果 `pair` 的元素支持，`pair` 会提供诸如 `=`、`==` 和 `<` 等运算符。类型推导使得创建 `pair` 变得容易，无需显式提及它的类型。例如：

```cpp
void f(vector<string>& v)
{
    pair p1 {v.begin(), 2};          // 一种方式
    auto p2 = make_pair(v.begin(), 2); // 另一种方式
    // ...
}
```

`p1` 和 `p2` 的类型都是 `pair<vector<string>::iterator, int>`。

当代码不需要泛型时，带有具名成员的简单 `struct` 通常能产生更易维护的代码。

### 15.3.4 tuple

像数组一样，标准库容器是同构的；也就是说，它们的所有元素都是单一类型的。然而，有时我们想要将不同类型元素的序列视为单个对象；也就是说，我们想要一个异构容器；`pair` 是一个例子，但并非所有此类异构序列都只有两个元素。标准库提供了 `tuple` 作为 `pair` 的推广，可以包含零个或多个元素：

```cpp
tuple t0 {};                                     // 空
tuple<string,int,double> t1 {"Shark", 123, 3.14}; // 显式指定类型
auto t2 = make_tuple(string{"Herring"}, 10, 1.23); // 类型推导为 tuple<string,int,double>
tuple t3 {"Cod"s, 20, 9.99};                     // 类型推导为 tuple<string,int,double>
```

`tuple` 的元素（成员）是独立的；它们之间不维护任何不变式（§4.3）。如果我们想要一个不变式，必须将 `tuple` 封装在一个强制该不变式的类中。

对于单个的、特定的用途，简单的 `struct` 通常是理想的，但在许多泛型用途中，`tuple` 的灵活性使我们不必定义许多 `struct`，代价是成员没有助记名称。`tuple` 的成员通过 `get` 函数模板访问。例如：

```cpp
string fish = get<0>(t1);    // 获取第一个元素："Shark"
int count = get<1>(t1);      // 获取第二个元素：123
double price = get<2>(t1);   // 获取第三个元素：3.14
```

`tuple` 的元素被编号（从零开始），并且传给 `get()` 的索引参数必须是常量。`get` 是一个函数模板，将索引作为模板值参数（§7.2.2）。

通过索引访问 `tuple` 的成员是通用的，但丑陋且有些容易出错。幸运的是，`tuple` 中具有唯一类型的元素可以通过其类型来“命名”：

```cpp
auto fish = get<string>(t1);   // 获取字符串："Shark"
auto count = get<int>(t1);     // 获取 int：123
auto price = get<double>(t1);  // 获取 double：3.14
```

我们也可以使用 `get<>` 进行写入：

```cpp
get<string>(t1) = "Tuna";   // 写入字符串
get<int>(t1) = 7;           // 写入 int
get<double>(t1) = 312;      // 写入 double
```

`tuple` 的大多数用法都隐藏在更高级别构造的实现中。例如，我们可以使用结构化绑定（§3.4.5）访问 `t1` 的成员：

```cpp
auto [fish, count, price] = t1;
cout << fish << ' ' << count << ' ' << price << '\n';   // 读取
fish = "Sea Bass";   // 写入
```

通常，这种绑定及其底层对 `tuple` 的使用用于函数调用：

```cpp
auto [fish, count, price] = todays_catch();
cout << fish << ' ' << count << ' ' << price << '\n';
```

`tuple` 的真正力量在于当您必须将数量未知、类型未知的元素作为对象存储或传递时。

显式地遍历 `tuple` 的元素有点麻烦，需要递归和函数体的编译时求值：

```cpp
template <size_t N = 0, typename... Ts>
constexpr void print(tuple<Ts...> tup)
{
    if constexpr (N < sizeof...(Ts)) {   // 还没到末尾？
        cout << get<N>(tup) << ' ';      // 打印第 N 个元素
        print<N+1>(tup);                 // 打印下一个元素
    }
}
```

这里，`sizeof...(Ts)` 给出 `Ts` 中元素的数量。使用 `print()` 很简单：

```cpp
print(t0);                                         // 无输出
print(t2);                                         // Herring 10 1.23
print(tuple{"Norah", 17, "Gavin", 14, "Anya", 9, "Courtney", 9, "Ada", 0});
```

===== 第 21 页 =====

像 `pair` 一样，如果其元素支持，`tuple` 也提供诸如 `=`、`==` 和 `<` 等运算符。此外，`pair` 与具有两个成员的 `tuple` 之间也存在转换。

## 15.4 替代方案

标准库提供了三种表示替代方案的类型：

**替代方案**

| 类型 | 描述 |
|------|------|
| `union` | 内置类型，持有一组替代方案中的一个值（§2.5） |
| `variant<T...>` | 指定的一组替代方案中的一个（在 `<variant>` 中） |
| `optional<T>` | 类型 `T` 的值或无值（在 `<optional>` 中） |
| `any` | 无界集合的替代类型中的一个值（在 `<any>` 中） |

这些类型为用户提供了相关的功能。不幸的是，它们没有提供统一的接口。

### 15.4.1 variant

`variant<A,B,C>` 通常是显式使用 `union`（§2.5）的更安全、更方便的替代方案。最简单的例子可能是返回一个值或一个错误码：

```cpp
variant<string, Error_code> compose_message(istream& s)
{
    string mess;
    // ... 从 s 读取并组合消息 ...
    if (no_problems)
        return mess;               // 返回一个字符串
    else
        return Error_code{some_problem};   // 返回一个 Error_code
}
```

===== 第 22 页 =====

当你用一个值赋值或初始化 `variant` 时，它会记住该值的类型。之后，我们可以查询 `variant` 持有哪种类型并提取该值。例如：

```cpp
auto m = compose_message(cin);

if (holds_alternative<string>(m)) {
    cout << get<string>(m);
}
else {
    auto err = get<Error_code>(m);
    // ... 处理错误 ...
}
```

这种风格吸引了一些不喜欢异常的人（见 §4.4），但还有更有趣的用途。例如，一个简单的编译器可能需要区分具有不同表示的不同类型的节点：

```cpp
using Node = variant<Expression, Statement, Declaration, Type>;

void check(Node* p)
{
    if (holds_alternative<Expression>(*p)) {
        Expression& e = get<Expression>(*p);
        // ...
    }
    else if (holds_alternative<Statement>(*p)) {
        Statement& s = get<Statement>(*p);
        // ...
    }
    // ... Declaration 和 Type ...
}
```

这种检查替代方案以决定适当行动的模式非常常见，且相对低效，因此它值得直接支持：

```cpp
void check(Node* p)
{
    visit(overloaded{
        [](Expression& e) { /* ... */ },
        [](Statement& s) { /* ... */ },
        // ... Declaration 和 Type ...
    }, *p);
}
```

这基本上等价于虚函数调用，但可能更快。与所有关于性能的声称一样，当性能至关重要时，应通过测量来验证这种“可能更快”。对于大多数用途，性能差异微不足道。

`overloaded` 类是必需的，而且奇怪的是，它不是标准的一部分。它是一个“魔法片段”，从一组参数（通常是 lambda）构建一个重载集：

```cpp
template<class... Ts>
struct overloaded : Ts... {   // 变参模板（§8.4）
    using Ts::operator()...;
};

template<class... Ts>
overloaded(Ts...) -> overloaded<Ts...>;   // 推导指引
```

然后，“访问者” `visit` 将 `()` 应用于 `overloaded` 对象，该对象根据重载规则选择最合适的 lambda 来调用。

推导指引是一种解决微妙歧义的机制，主要用于基础库中类模板的构造函数（§7.2.3）。

如果我们试图访问一个持有与预期不同类型值的 `variant`，会抛出 `bad_variant_access`。

### 15.4.2 optional

`optional<A>` 可以看作一种特殊的 `variant`（类似于 `variant<A, nothing>`），或者看作是“`A*` 要么指向一个对象要么为 `nullptr`”这一概念的泛化。

`optional` 对于可能返回或不返回对象的函数很有用：

```cpp
optional<string> compose_message(istream& s)
{
    string mess;
    // ... 从 s 读取并组合消息 ...
    if (no_problems)
        return mess;
    return {};   // 空的 optional
}
```

===== 第 24 页 =====

有了这个，我们可以写：

```cpp
if (auto m = compose_message(cin))
    cout << *m;   // 注意解引用 (*)
else {
    // ... 处理错误 ...
}
```

这吸引了一些不喜欢异常的人（见 §4.4）。注意 `*` 的不寻常用法。`optional` 被当作指向其对象的指针而不是对象本身来对待。

`optional` 中与 `nullptr` 等价的是空对象 `{}`。例如：

```cpp
int sum(optional<int> a, optional<int> b)
{
    int res = 0;
    if (a) res += *a;
    if (b) res += *b;
    return res;
}

int x = sum(17, 19);   // 36
int y = sum(17, {});   // 17
int z = sum({}, {});   // 0
```

如果我们试图访问一个不持有值的 `optional`，结果是未定义的；不会抛出异常。因此，`optional` 不保证类型安全。不要尝试：

```cpp
int sum2(optional<int> a, optional<int> b)
{
    return *a + *b;   // 自找麻烦
}
```

### 15.4.3 any

`any` 可以持有任意类型，并知道它持有哪种类型（如果有的话）。它基本上是 `variant` 的无约束版本：

```cpp
any compose_message(istream& s)
{
    string mess;
    // ... 从 s 读取并组合消息 ...
    if (no_problems)
        return mess;           // 返回一个字符串
    else
        return error_number;   // 返回一个 int
}
```

当你用一个值赋值或初始化 `any` 时，它会记住该值的类型。之后，我们可以通过断言该值的预期类型来提取 `any` 持有的值。例如：

```cpp
auto m = compose_message(cin);
string& s = any_cast<string>(m);
cout << s;
```

如果我们试图访问一个持有与预期不同类型值的 `any`，会抛出 `bad_any_access`。

## 15.5 建议

[1] 一个库不必庞大或复杂才能有用；§16.1。
[2] 资源是任何必须获取并在之后（显式或隐式）释放的东西；§15.2.1。
[3] 使用资源句柄来管理资源（RAII）；§15.2.1；[CG: R.1]。
[4] `T*` 的问题在于它可以用来表示任何东西，因此我们无法轻易确定“原始”指针的用途；§15.2.1。
[5] 使用 `unique_ptr` 来引用多态类型的对象；§15.2.1；[CG: R.20]。
[6] 使用 `shared_ptr` 来引用共享对象（仅限共享场景）；§15.2.1；[CG: R.20]。
[7] 相对于智能指针，优先选择具有特定语义的资源句柄；§15.2.1。
[8] 在可以使用局部变量的地方不要使用智能指针；§15.2.1。
[9] 相对于 `shared_ptr`，优先选择 `unique_ptr`；§6.3，§15.2.1。
[10] 仅在需要转移所有权责任时，才将 `unique_ptr` 或 `shared_ptr` 用作参数或返回值；§15.2.1；[CG: F.26] [CG: F.27]。
[11] 使用 `make_unique()` 构造 `unique_ptr`；§15.2.1；[CG: R.22]。
[12] 使用 `make_shared()` 构造 `shared_ptr`；§15.2.1；[CG: R.23]。
[13] 相对于垃圾回收，优先选择智能指针；§6.3，§15.2.1。
[14] 相对于指针加计数的接口，优先选择 `span`；§15.2.2；[CG: F.24]。
[15] `span` 支持范围 `for`；§15.2.2。
[16] 当你需要一个具有 `constexpr` 大小的序列时，使用 `array`；§15.3.1。
[17] 相对于内置数组，优先选择 `array`；§15.3.1；[CG: SL.con.2]。
[18] 当你需要 `N` 位，而 `N` 不一定是内置整数类型中的位数时，使用 `bitset`；§15.3.2。
[19] 不要过度使用 `pair` 和 `tuple`；具名的 `struct` 通常能产生更可读的代码；§15.3.3。
[20] 使用 `pair` 时，使用模板参数推导或 `make_pair()` 来避免冗余的类型指定；§15.3.3。
[21] 使用 `tuple` 时，使用模板参数推导或 `make_tuple()` 来避免冗余的类型指定；§15.3.3；[CG: T.44]。
[22] 相对于显式使用 `union`，优先选择 `variant`；§15.4.1；[CG: C.181]。
[23] 当使用 `variant` 在一组替代方案中进行选择时，考虑使用 `visit()` 和 `overloaded()`；§15.4.1。

===== 第 27 页 =====

[24] 如果 `variant`、`optional` 或 `any` 可能有多种替代方案，请在访问前检查标签；§15.4。