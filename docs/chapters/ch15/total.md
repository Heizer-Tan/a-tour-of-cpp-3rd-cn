# 指针和容器


> **教育是关于做什么、何时做以及为何做；训练是关于如何做。**
>
> —— Richard Hamming


# 15.1 引言

C++ 提供简单的内置低级类型来保存并引用数据：对象与数组保存数据；指针与数组引用这些数据。然而，我们还需要支持更专门以及更一般的持有与使用数据的方式。举例来说，标准库容器（[第 12 章](../ch12/index.md)）与迭代器（[§13.3](../ch13/13-3-iterator-types.md)）就是为通用算法而设计的。

容器与指针抽象的主要共同点在于：正确且高效地使用它们，需要把数据连同访问与操纵它的一组函数一起封装起来。举例来说，指针是极其通用且高效的机器地址抽象，但要用它来表示资源所有权已被证明极其困难。因此标准库提供了资源管理指针；也就是封装指针并提供简化正确使用的操作的那些类。

这些标准库抽象封装内置语言类型，并且在时间与空间上被要求与正确使用那些类型时同样高效。

它们也没有什么“魔法”。我们也可以按需、沿用标准库所用的技法设计与实现自己的“智能指针”与专用容器。


# 15.2 指针

指针这一笼统观念指的是：允许我们引用对象并按其类型去访问它的某种手段。内置指针例如 `int*` 只是一种特例；此外还有许许多多形态。

**指针**

| 记号 | 含义 |
|------|------|
| `T*` | 内置指针：指向类型 `T` 的对象，或指向一段连续分配的 `T` 元素序列 |
| `T&` | 内置引用：引用类型 `T` 的对象；相当于隐式解引用的指针（[§1.7](../ch01/1-7-pointers-arrays.md)） |
| `unique_ptr<T>` | 指向 `T` 的拥有型指针 |
| `shared_ptr<T>` | 指向类型 `T` 对象的指针；所有权由所有指向该 `T` 的 `shared_ptr` 共享 |
| `weak_ptr<T>` | 指向由 `shared_ptr` 拥有的对象；必须先转成 `shared_ptr` 才能访问 |
| `span<T>` | 指向一段连续的 `T` 序列（[§15.2.2](15-2-pointers.md#15.2.2)） |
| `string_view` | 指向常量子串（[§10.3](../ch10/10-3-string-view.md)） |
| `X_iterator<C>` | 来自 `C` 的元素序列；名称中的 `X` 指明迭代器类别（[§13.3](../ch13/13-3-iterator-types.md)） |

同一个对象可以同时被多个指针指到。**拥有型指针**负责最终销毁所指对象；**非拥有指针**（如 `T*`、`span`）则可能悬空——指向早已销毁或离开作用域之处。

经由悬空指针读写是最棘手的一类错误之一：语义上是未定义行为。实践中常常表现为碰巧读写到了别处占用的一块内存：读出任意比特，写入破坏无关数据结构。最体面的下场往往是崩溃——而这常常好过静默的错误。

《C++ 核心准则》[CG] 给出了如何避免悬空指针的规则与可做静态检查的建议。下面是几条实操层面的防线：

- 不要在局部对象离开作用域后仍保留指向它的指针。尤其不要从函数返回指向局部对象的指针，也不要把来路不明的指针存进长寿数据结构。系统化地使用容器与算法（[第 12 章](../ch12/index.md)、[第 13 章](../ch13/index.md)）常常使我们免于采用那些难以避免指针问题的编程技法。
- 对在自由存储上分配的对象使用拥有型指针。
- 指向静态对象（例如全局变量）的指针不会悬空。
- 把指针算术留给资源句柄的实现（例如 `vector` 与 `unordered_map`）。
- 记住 `string_view` 与 `span` 是非拥有指针的种类。

## 15.2.1 `unique_ptr` 与 `shared_ptr`

任何一个非平凡程序的关键任务之一，就是管理资源。**资源**指必须先获取、之后再（显式或隐式）释放的东西。例子包括内存、锁、套接字、线程句柄、文件句柄等。对长跑程序而言，未能及时释放资源（“泄漏”）可能造成严重的性能退化（[§12.7](../ch12/12-7-allocators.md)），甚至更糟糕的崩溃。即便是短程序，泄漏也可能令人尴尬——例如因资源短缺把运行时间拖垮好几个数量级。

标准库组件被设计成不泄漏资源。为此，它们依赖语言对资源管理的基本支持：用构造函数/析构函数成对，确保资源不会比负责它的对象活得更久。`Vector` 用构造函数/析构函数管理元素生命周期的示例（[§5.2.2](../ch05/5-2-concrete-types.md#5.2.2)）就是一例；所有标准库容器都以类似方式实现。重要的是，这一做法与基于异常的错误处理正确协作。例如，标准库的锁类使用了该技术：

```cpp
mutex m; // 用于保护共享数据

void f()
{
    scoped_lock lck {m}; // 构造阶段占有互斥量 m
    // ... 操纵共享数据 ...
}
```

线程在 `lck` 的构造函数获取互斥量之前不会继续前进（[§18.3](../ch18/18-3-shared-data.md)）。对应的析构函数释放互斥量。因此在该例中，当控制流离开 `f()`（通过 `return`、从函数末尾“坠落”，或通过抛出异常）时，`scoped_lock` 的析构函数会释放互斥量。

这便是 **RAII**（“资源获取即初始化”技术；[§5.2.2](../ch05/5-2-concrete-types.md#5.2.2)）的应用。RAII 是 C++ 中惯用资源处理的基础。容器（例如 `vector` 与 `map`）、`string` 以及 `iostream` 也以类似方式管理各自的资源（例如文件句柄与缓冲区）。

到目前为止的例子处理的是在作用域内定义的对象，在离开作用域时释放它们获取的资源；那自由存储上分配的对象呢？在 `<memory>` 中，标准库提供两个“智能指针”来帮助管理自由存储上的对象：

- `unique_ptr` 表示独占所有权（其析构函数销毁对象）
- `shared_ptr` 表示共享所有权（最后一个共享指针的析构函数销毁对象）

这些“智能指针”最基本的用途，是防止因粗心编程导致的内存泄漏。例如：

```cpp
void f(int i, int j) // X* 对比 unique_ptr<X>
{
    X* p = new X;
    unique_ptr<X> sp {new X};
    // ...

    if (i < 99)
        throw Z{};      // 可能抛异常
    if (j < 77)
        return;         // 可能提前返回
    // ... 使用 p 与 sp ...
    delete p;           // 销毁 *p
}
```

这里，若 `i<99` 或 `j<77`，我们“忘记”了 `delete p`。另一方面，`unique_ptr` 确保不论我们以何种方式退出 `f()`（抛出异常、执行 `return`，或从函数末尾“坠落”），其对象都会被妥善销毁。颇具讽刺意味的是，我们本可以干脆不用指针、也不用 `new` 来解决问题：

```cpp
void f(int i, int j)      // 使用局部变量
{
    X x;
    // ...
}
```

不幸的是，`new`（以及指针与引用）的过度使用似乎正在成为一个日益严重的问题。

不过，当你确实需要指针语义时，`unique_ptr` 是一种轻量机制，相对正确使用内置指针而言没有额外的空间或时间开销。它的进一步用途包括把自由存储分配的对象传入和传出函数：

```cpp
unique_ptr<X> make_X(int i)
    // 创建 X 并立刻交给 unique_ptr
{
    // ... 校验 i 等等 ...
    return unique_ptr<X>{new X{i}};
}
```

`unique_ptr` 是单个对象（或数组）的句柄，很大程度上就像 `vector` 是对象序列的句柄。二者都（用 RAII）控制其他对象的生命周期，并都依赖消除拷贝或依赖移动语义，使返回简单且高效（[§6.2.2](../ch06/6-2-copy-move.md#6.2.2)）。

`shared_ptr` 与 `unique_ptr` 类似，区别在于 `shared_ptr` 是拷贝而非移动。同一对象的各个 `shared_ptr` 共享该对象的所有权；当最后一个 `shared_ptr` 被销毁时，该对象才被销毁。例如：

```cpp
void f(shared_ptr<fstream>);
void g(shared_ptr<fstream>);

void user(const string& name, ios_base::openmode mode)
{
    shared_ptr<fstream> fp {new fstream(name, mode)};
    if (!*fp)                   // 确认文件确实打开成功
        throw No_file{};

    f(fp);
    g(fp);
    // ...
}
```

现在，由 `fp` 的构造函数打开的文件，会由最后一个（显式或隐式）销毁 `fp` 副本的函数关闭。注意 `f()` 或 `g()` 可能派生一个持有 `fp` 副本的任务，或以其他方式保存一个比 `user()` 活得更久的副本。因此，`shared_ptr` 提供了一种尊重基于析构函数的资源管理的垃圾回收形式。这既不是零成本，也谈不上极其昂贵，但确实使共享对象的寿命难以预测。**只在你确实需要共享所有权时才使用 `shared_ptr`。**

先在自由存储上创建对象，再把指向它的指针交给智能指针，写法略显啰嗦。这也容易出错，例如忘记把指针交给 `unique_ptr`，或把并非自由存储上对象的指针交给 `shared_ptr`。为避免这类问题，标准库（在 `<memory>` 中）提供了构造对象并返回相应智能指针的函数：`make_shared()` 与 `make_unique()`。例如：

```cpp
struct S {
    int i;
    string s;
    double d;
    // ...
};

auto p1 = make_shared<S>(1, "Ankh Morpork", 4.65); // shared_ptr<S>
auto p2 = make_unique<S>(2, "Oz", 7.62);         // unique_ptr<S>
```

现在，`p2` 是一个 `unique_ptr<S>`，指向自由存储上分配的、值为 `{2,"Oz"s,7.62}` 的 `S` 对象。

使用 `make_shared()` 不仅比分别用 `new` 创建对象再交给 `shared_ptr` 更方便——它也明显更高效，因为它不必为 `shared_ptr` 实现所必需的使用计数再做一次单独分配。

有了 `unique_ptr` 与 `shared_ptr`，对许多程序我们可以贯彻完整的“禁止裸 `new`”策略（[§5.2.2](../ch05/5-2-concrete-types.md#5.2.2)）。然而，这些“智能指针”在概念上仍是指针，因此在资源管理上只是我的第二选择——排在容器以及其他在更高概念层次上管理资源的类型之后。尤其是，`shared_ptr` 本身并不提供关于哪些所有者可以读和/或写共享对象的任何规则。仅仅消除资源管理问题，并不能解决数据竞争（[§18.5](../ch18/18-5-inter-task-communication.md)）以及其他形式的混淆。

我们何时使用“智能指针”（如 `unique_ptr`），而非为资源专门设计操作的资源句柄（如 `vector` 或 `thread`）？毫不意外，答案是“当我们需要指针语义时”。

- 当我们共享一个对象时，需要指针（或引用）来指称该共享对象，因此 `shared_ptr` 成为显然的选择（除非存在显然的单一所有者）。
- 当我们在经典面向对象代码（[§5.5](../ch05/5-5-hierarchies.md)）中引用多态对象时，需要指针（或引用），因为我们不知道所指对象的确切类型（甚至不知道其大小），因此 `unique_ptr` 成为显然的选择。
- 共享的多态对象通常需要 `shared_ptr`。
- 我们不必用指针从函数返回一组对象；作为资源句柄的容器可以依靠拷贝省略（[§3.4.2](../ch03/3-4-parameters.md#3.4.2)）与移动语义（[§6.2.2](../ch06/6-2-copy-move.md#6.2.2)）简洁高效地完成。

## 15.2.2 `span`

长久以来，范围错误一直是 C 与 C++ 程序中严重错误的主要来源，导致错误结果、崩溃与安全问题。使用容器（[第 12 章](../ch12/index.md)）、算法（[第 13 章](../ch13/index.md)）与范围 `for` 已显著减轻这一问题，但还能做得更多。范围错误的一个关键来源是：人们传入指针（裸的或智能的），然后依赖约定来知道所指元素的个数。对资源句柄之外的代码，最佳建议是假定至多指向一个对象 [CG: F.22]，但若没有支撑，这条建议难以落实。

标准库的 `string_view`（[§10.3](../ch10/10-3-string-view.md)）可以帮忙，但它只读且仅面向字符。多数程序员需要更多。例如，在较低层软件中向缓冲区写入、从缓冲区读出时，既要保持高性能又要避免范围错误（“缓冲区溢出”）出了名地困难。来自 `<span>` 的 `span` 本质上就是表示元素序列的（指针，长度）对：

![span 为（begin()，size()）对](../../assets/images/ch15/span-layout.png)

`span` 提供对连续元素序列的访问。这些元素可以多种方式存储，包括在 `vector` 与内置数组中。与指针一样，`span` 并不拥有它指向的字符。在这一点上，它类似于 `string_view`（[§10.3](../ch10/10-3-string-view.md)）以及 STL 的一对迭代器（[§13.3](../ch13/13-3-iterator-types.md)）。

考虑一种常见的接口风格：

```cpp
void fpn(int* p, int n)
{
    for (int i = 0; i < n; ++i)
        p[i] = 0;
}
```

我们假定 `p` 指向 `n` 个整数。不幸的是，这一假定只是约定，因此我们不能据此写出范围 `for` 循环，编译器也无法实现廉价而有效的范围检查。此外，我们的假定还可能是错的：

```cpp
void use(int x)
{
    int a[100];
    fpn(a, 100);      // OK
    fpn(a, 1000);     // 糟糕，手指滑了！（fpn 中的范围错误）
    fpn(a + 10, 100); // fpn 中的范围错误
    fpn(a, x);        // 可疑，但看起来无害
}
```

使用 `span` 可以做得更好：

```cpp
void fs(span<int> p)
{
    for (int& x : p)
        x = 0;
}
```

我们可以这样使用 `fs`：

```cpp
void use(int x)
{
    int a[100];
    fs(a);             // 隐式创建 span<int>{a,100}
    fs(a, 1000);       // 错误：期望 span
    fs({a + 10, 100}); // fs 中的范围错误
    fs({a, x});        // 明显可疑
}
```

也就是说，常见情形——直接从数组创建 `span`——现在既安全（由编译器计算元素个数）又记法简单。在其他情形下，出错概率降低，错误也更容易察觉，因为程序员必须显式拼出 `span`。

把 `span` 从函数传到函数的常见情形，也比（指针，计数）接口更简单，而且显然不需要额外检查：

```cpp
void f1(span<int> p);

void f2(span<int> p)
{
    // ...
    f1(p);
}
```

与容器类似：`span` 的下标 `r[i]` 不做范围检查，越界访问仍是未定义行为。实现当然可以把这种未定义行为实现成抛出异常，但遗憾的是鲜有实现这么做。《C++ 核心准则》支撑库中的原始 `gsl::span` 则会进行范围检查（参见 [CG]）。


# 15.3 容器

标准库还提供了若干并不能完美嵌入 STL 框架（[第 12 章](../ch12/index.md)、[第 13 章](../ch13/index.md)）的容器。例子包括内置数组、`array` 与 `string`。我有时把它们叫作“准容器”，但这并不完全公平：它们保存元素，因而确实是容器，只是各自带有限制或额外设施，使它们在 STL 语境中显得别扭。把它们单独描述也有助于简化对 STL 本身的说明。

**容器**

| 类型 | 说明 |
|------|------|
| `T[N]` | 内置数组：固定大小、连续分配的 `N` 个类型为 `T` 的元素序列；隐式转换为 `T*` |
| `array<T,N>` | 固定大小、连续分配的 `N` 个类型为 `T` 的元素序列；像内置数组，但多数问题已解决 |
| `bitset<N>` | 固定大小的 `N` 个比特的序列 |
| `vector<bool>` | 在 `vector` 的特化中紧凑存放的比特序列 |
| `pair<T,U>` | 类型分别为 `T` 与 `U` 的两个元素 |
| `tuple<T...>` | 任意个数、任意类型的元素序列 |
| `basic_string<C>` | 类型为 `C` 的字符序列；提供字符串操作 |
| `valarray<T>` | 类型为 `T` 的数值数组；提供数值运算 |

为什么标准库要提供这么多容器？它们服务于常见但不同（往往重叠）的需求。若标准库不提供它们，许多人就不得不自行设计与实现。例如：

- `pair` 与 `tuple` 是异质的；所有其他容器都是同质的（所有元素类型相同）。
- `array` 与 `tuple` 的元素是连续分配的；`list` 与 `map` 是链接结构。
- `bitset` 与 `vector<bool>` 保存比特并通过代理对象访问；所有其他标准库容器可保存多种类型并直接访问元素。
- `basic_string` 要求其元素是某种形式的字符，并提供字符串操纵，例如拼接与 locale 敏感操作。
- `valarray` 要求其元素是数，并提供数值运算。

所有这些容器都可以看作在为大型程序员社群提供所需的专门服务。没有单一容器能满足所有这些需求，因为有些需求相互矛盾，例如“能够增长”对“保证分配在固定位置”，以及“添加元素时元素不移动”对“连续分配”。

## 15.3.1 `array`

`<array>` 中定义的 `array` 是给定类型元素的固定大小序列，元素个数在编译期指定。因此，`array` 可以连同其元素一起分配在栈上、对象中或静态存储中。元素分配在定义该 `array` 的作用域中。

把 `array` 理解成“尺寸牢牢附在身上的内置数组”最贴切：没有隐式的、可能令人吃惊的向指针类型的转换，并提供少量便利函数。相对使用内置数组，使用 `array` 没有时间或空间开销。`array` 并不遵循 STL 容器那种“指向元素的句柄”模型；相反，`array` 直接包含其元素。它不多不少，就是更安全的内置数组。

这意味着 `array` 可以且必须用初始化列表初始化：

```cpp
array<int,3> a1 = {1,2,3};
```

初始化器中的元素个数必须等于或少于为该 `array` 指定的元素个数。

元素个数不是可选的；元素个数必须是常量表达式；元素个数必须为正；元素类型必须显式写明：

```cpp
void f(int n)
{
    array<int> a0 = {1,2,3};                          // 错误：未指定大小
    array<string,n> a1 = {"John's", "Queens' "};      // 错误：大小不是常量表达式
    array<string,0> a2;                               // 错误：大小必须为正
    array<2> a3 = {"John's", "Queens' "};             // 错误：未写明元素类型
    // ...
}
```

若需要元素个数是变量，请使用 `vector`。

必要时，可以把 `array` 显式传给期望指针的 C 风格函数。例如：

```cpp
void f(int* p, int sz);         // C 风格接口

void g()
{
    array<int,10> a;

    f(a, a.size());             // 错误：无法转换
    f(a.data(), a.size());      // C 风格用法

    auto p = find(a, 777);      // C++/STL 风格用法（传入一个范围）
    // ...
}
```

既然 `vector` 灵活得多，我们为什么还要用 `array`？`array` 不那么灵活，因而更简单。偶尔，直接访问分配在栈上的元素，会比把元素分配在自由存储上、经 `vector`（句柄）间接访问、然后再释放它们，带来显著的性能优势。另一方面，栈是有限资源（尤其在某些嵌入式系统上），栈溢出也很讨厌。此外，在某些应用领域——例如安全攸关的实时控制——自由存储分配是被禁止的。例如，使用 `delete` 可能导致碎片化（[§12.7](../ch12/12-7-allocators.md)）或内存耗尽（[§4.3](../ch04/4-3-invariants.md)）。

既然可以用内置数组，我们为什么还要用 `array`？`array` 知道自己的大小，因而易于与标准库算法一起使用，并且可以用 `=` 拷贝。例如：

```cpp
array<int,3> a1 = {1, 2, 3};
auto a2 = a1;     // 拷贝
a2[1] = 5;
a1 = a2;          // 赋值
```

不过，我偏爱 `array` 的主要原因是：它使我免于令人吃惊且讨厌的向指针的转换。考虑一个涉及类层次结构的例子：

```cpp
void h()
{
    Circle a1[10];
    array<Circle,10> a2;
    // ...
    Shape* p1 = a1;       // OK：灾难等待发生
    Shape* p2 = a2;       // 错误：array<Circle,10> 不能转换为 Shape*（很好！）
    p1[3].draw();         // 灾难
}
```

“灾难”这一注释假定 `sizeof(Shape)<sizeof(Circle)`，因此通过 `Shape*` 对 `Circle[]` 做下标会得到错误的偏移。所有标准容器相对内置数组都提供这一优势。

## 15.3.2 `bitset`

系统的某些方面——例如输入流的状态——常常表示为一组标志，指示 good/bad、true/false、on/off 一类二元条件。C++ 通过对整数的按位运算高效支持小规模标志集合的观念（[§1.4](../ch01/1-4-types-variables.md)）。类 `bitset<N>` 把这一观念推广为对 `N` 个比特的序列 `[0:N)` 的操作，其中 `N` 在编译期已知。对装不进 `long long int`（常常是 64 位）的比特集合，使用 `bitset` 比直接使用整数方便得多。对较小的集合，`bitset` 通常也会被优化。若你想按名字而非编号指称这些比特，可以使用 `set`（[§12.5](../ch12/12-5-map.md)）或枚举（[§2.4](../ch02/2-4-enum.md)）。

`bitset` 可以用整数或字符串初始化：

```cpp
bitset<9> bs1 {"110001111"};
bitset<9> bs2 {0b1'1000'1111};       // 使用数字分隔符的二进制字面量（[§1.4](../ch01/1-4-types-variables.md)）
```

通常的按位运算符（[§1.4](../ch01/1-4-types-variables.md)）以及左移、右移运算符（`<<` 与 `>>`）都可以使用：

```cpp
bitset<9> bs3 = ~bs1;               // 取反：bs3=="001110000"
bitset<9> bs4 = bs1&bs3;            // 全零
bitset<9> bs5 = bs1<<2;             // 左移：bs5 = "000111100"
```

移位运算符（这里是 `<<`）会“移入”零。

操作 `to_ullong()` 与 `to_string()` 提供与构造函数相反的操作。例如，我们可以写出一个 `int` 的二进制表示：

```cpp
void binary(int i)
{
    bitset<8*sizeof(int)> b = i;             // 假定 8 位字节（亦见 [§17.7](../ch17/17-7-numeric-limits.md)）
    cout << b.to_string() << '\n';           // 写出 i 的各个比特
}
```

这会把比特表示为从左到右的 1 与 0，最高有效位在最左，因此实参 `123` 会给出输出

```
00000000000000000000000001111011
```

对本例而言，直接使用 `bitset` 的输出运算符更简单：

```cpp
void binary2(int i)
{
    bitset<8*sizeof(int)> b = i;      // 假定 8 位字节（亦见 [§17.7](../ch17/17-7-numeric-limits.md)）
    cout << b << '\n';                // 写出 i 的各个比特
}
```

`bitset` 还提供许多用于使用与操纵比特集合的函数，例如 `all()`、`any()`、`none()`、`count()`、`flip()`。

## 15.3.3 `pair`

函数返回两个值相当常见。做法很多，最简单且往往最好的是为此定义一个 `struct`。例如，我们可以返回一个值与一个成功指示：

```cpp
struct My_res {
    Entry* ptr;
    Error_code err;
};

My_res complex_search(vector<Entry>& v, const string& s)
{
    Entry* found = nullptr;
    Error_code err = Error_code::found;
    // ... 在 v 中查找 s ...
    return {found, err};
}

void user(const string& s)
{
    My_res r = complex_search(entry_table, s);      // 搜索 entry_table
    if (r.err != Error_code::good) {
        // ... 处理错误 ...
    }
    // ... 使用 r.ptr ...
}
```

我们可以争辩：把失败编码为尾迭代器或 `nullptr` 更优雅，但那只能表达一种失败。我们常常希望返回两个独立的值。为每一对值定义特定的具名 `struct` 往往效果很好，而且只要“值对”结构体及其成员的名字选得好，就相当可读。然而，在大型代码库中这可能导致名字与约定激增，并且对需要一致命名的泛型代码并不适用。因此，标准库提供 `pair`，作为“值对”用例的通用支持。使用 `pair`，我们的简单例子变成：

```cpp
pair<Entry*,Error_code> complex_search(vector<Entry>& v, const string& s)
{
    Entry* found = nullptr;
    Error_code err = Error_code::found;
    // ... 在 v 中查找 s ...
    return {found, err};
}

void user(const string& s)
{
    auto r = complex_search(entry_table, s);           // 搜索 entry_table
    if (r.second != Error_code::good) {
        // ... 处理错误 ...
    }
    // ... 使用 r.first ...
}
```

`pair` 的成员名为 `first` 与 `second`。从实现者角度看这说得通，但在应用代码中我们可能想用自己的名字。结构化绑定（[§3.4.5](../ch03/3-4-parameters.md#3.4.5)）可以用来处理这一点：

```cpp
void user(const string& s)
{
    auto [ptr, success] = complex_search(entry_table, s);      // 搜索 entry_table
    if (success != Error_code::good) {
        // ... 处理错误 ...
    }
    // ... 使用 ptr ...
}
```

标准库的 `pair`（来自 `<utility>`）在标准库及其他地方的“值对”用例中相当常见。例如，标准库算法 `equal_range` 返回一对迭代器，指明满足谓词的子序列。给定有序序列 `[first:last)`，`equal_range()` 将返回表示匹配谓词 `cmp` 的那个子序列的 `pair`。我们可以用它在已排序的 `Record` 序列中搜索：

```cpp
template<typename Forward_iterator, typename T, typename Compare>
pair<Forward_iterator, Forward_iterator>
equal_range(Forward_iterator first, Forward_iterator last, const T& val, Compare cmp);

auto less = [](const Record& r1, const Record& r2) { return r1.name < r2.name; };

void f(const vector<Record>& v)            // 假定 v 已按 name 字段排序
{
    auto [first, last] = equal_range(v.begin(), v.end(), Record{"Reg"}, less);

    for (auto p = first; p != last; ++p)                // 打印所有相等记录
        cout << *p;                                     // 假定已为 Record 定义 <<
}
```

若其元素支持，`pair` 提供诸如 `=`、`==` 与 `<` 的运算符。类型推导使我们无需显式提及类型即可轻松创建 `pair`。例如：

```cpp
void f(vector<string>& v)
{
    pair p1 {v.begin(), 2};                         // 一种写法
    auto p2 = make_pair(v.begin(), 2);              // 另一种写法
    // ...
}
```

`p1` 与 `p2` 的类型都是 `pair<vector<string>::iterator,int>`。

当代码不需要泛型时，带有具名成员的简单 `struct` 往往带来更易维护的代码。

## 15.3.4 `tuple`

与数组一样，标准库容器是同质的；也就是说，它们的所有元素都是单一类型。然而，有时我们希望把不同类型元素的序列当作单一对象来处理；也就是说，我们想要异质容器。`pair` 是一例，但并非所有这类异质序列都恰好只有两个元素。标准库提供 `tuple`，作为带有零个或更多元素的 `pair` 的推广：

```cpp
tuple t0 {};                                                      // 空
tuple<string,int,double> t1 {"Shark",123,3.14};                   // 类型显式指定
auto t2 = make_tuple(string{"Herring"},10,1.23);                  // 类型推导为 tuple<string,int,double>
tuple t3 {"Cod"s,20,9.99};                                        // 类型推导为 tuple<string,int,double>
```

`tuple` 的元素（成员）彼此独立；它们之间不维护不变式（[§4.3](../ch04/4-3-invariants.md)）。若我们想要不变式，必须把 `tuple` 封装进强制该不变式的类中。

对单一、具体的用途，简单的 `struct` 往往最理想，但在许多泛型用途中，`tuple` 的灵活性使我们免于定义许多 `struct`，代价是没有助记的成员名。`tuple` 的成员通过 `get` 函数模板访问。例如：

```cpp
string fish = get<0>(t1);              // 取得第一个元素："Shark"
int count = get<1>(t1);                // 取得第二个元素：123
double price = get<2>(t1);             // 取得第三个元素：3.14
```

`tuple` 的元素从零开始编号，传给 `get()` 的下标实参必须是常量。函数 `get` 是以该下标为模板值实参的函数模板（[§7.2.2](../ch07/7-2-parameterized-types.md#7.2.2)）。

通过下标访问 `tuple` 成员通用、难看，且有些容易出错。幸运的是，若 `tuple` 中某个元素的类型在该 `tuple` 中唯一，就可以用其类型“命名”它：

```cpp
auto fish = get<string>(t1);                // 取得 string："Shark"
auto count = get<int>(t1);                  // 取得 int：123
auto price = get<double>(t1);               // 取得 double：3.14
```

我们也可以用 `get<>` 写入：

```cpp
get<string>(t1) = "Tuna";       // 写入 string
get<int>(t1) = 7;               // 写入 int
get<double>(t1) = 312;          // 写入 double
```

多数对 `tuple` 的使用隐藏在更高层构造的实现中。例如，我们可以用结构化绑定（[§3.4.5](../ch03/3-4-parameters.md#3.4.5)）访问 `t1` 的成员：

```cpp
auto [fish, count, price] = t1;
cout << fish << ' ' << count << ' ' << price << '\n';      // 读
fish = "Sea Bass";                                                       // 写
```

通常，这种绑定及其底层对 `tuple` 的使用出现在函数调用中：

```cpp
auto [fish, count, price] = todays_catch();
cout << fish << ' ' << count << ' ' << price << '\n';
```

`tuple` 的真正力量在于：当你必须把未知个数、未知类型的元素作为对象存储或传来传去时。

显式遍历 `tuple` 的元素有些凌乱，需要递归以及对函数体的编译期求值：

```cpp
template <size_t N = 0, typename... Ts>
constexpr void print(tuple<Ts...> tup)
{
    if constexpr (N < sizeof...(Ts)) {         // 尚未到末尾？
        cout << get<N>(tup) << ' ';            // 打印第 N 个元素
        print<N + 1>(tup);                     // 打印下一个元素
    }
}
```

这里，`sizeof...(Ts)` 给出 `Ts` 中的元素个数。

使用 `print()` 很直接：

```cpp
print(t0);           // 无输出
print(t2);           // Herring 10 1.23
print(tuple{ "Norah", 17, "Gavin", 14, "Anya", 9, "Courtney", 9, "Ada", 0 });
```

与 `pair` 一样，若其元素支持，`tuple` 提供诸如 `=`、`==` 与 `<` 的运算符。在 `pair` 与带有两个成员的 `tuple` 之间也有转换。


# 15.4 替代方案

标准库提供三种类型来表达替代：

**替代方案**

| 类型 | 说明 |
|------|------|
| `union` | 内置类型，保存一组替代中的一个（[§2.5](../ch02/2-5-union.md)） |
| `variant<T...>` | 指定的一组替代中的一个（在 `<variant>` 中） |
| `optional<T>` | 类型为 `T` 的值，或没有值（在 `<optional>` 中） |
| `any` | 无界替代类型集合中某一类型的值（在 `<any>` 中） |

这些类型为用户提供相关功能。遗憾的是，它们并不提供统一的接口。

## 15.4.1 `variant`

`variant<A,B,C>` 常常是显式使用 `union`（[§2.5](../ch02/2-5-union.md)）的更安全、更方便的替代。可能最简单的例子是返回一个值或一个错误码：

```cpp
variant<string,Error_code> compose_message(istream& s)
{
    string mess;
    // ... 从 s 读取并拼装消息 ...
    if (no_problems)
        return mess;                                               // 返回 string
    else
        return Error_code{some_problem};        // 返回 Error_code
}
```

当你用一个值赋值或初始化 `variant` 时，它会记住该值的类型。之后，我们可以查询 `variant` 持有何种类型并取出该值。例如：

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

这种风格对某些不喜欢异常的人有吸引力（见 [§4.4](../ch04/4-4-alternatives.md)），但还有更有趣的用途。例如，一个简单的编译器可能需要区分具有不同表示的不同种类的节点：

```cpp
using Node = variant<Expression,Statement,Declaration,Type>;

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
    // ... Declaration 与 Type ...
}
```

这种检查各个替代以决定适当动作的模式如此常见，又相对低效，因而值得直接支持：

```cpp
void check(Node* p)
{
    visit(overloaded {
        [](Expression& e) { /* ... */ },
        [](Statement& s) { /* ... */ },
        // ... Declaration 与 Type ...
    }, *p);
}
```

这基本上等价于虚函数调用，但可能更快。与所有关于性能的断言一样，当性能关键时，这一“可能更快”应当用测量来核实。对多数用途，性能差异并不显著。

`overloaded` 类是必要的，而且奇怪的是，它并非标准的。它是一块“魔法”，从一组实参（通常是 lambda）构造一个重载集：

```cpp
template<class... Ts>
struct overloaded : Ts... {            // 可变参数模板（[§8.4](../ch08/8-4-variadic-templates.md)）
    using Ts::operator()...;
};

template<class... Ts>
overloaded(Ts...) -> overloaded<Ts...>;    // 推导指引
```

然后“访问者”`visit` 把 `()` 作用到该重载对象上，后者按重载规则选出最合适的 lambda 来调用。

推导指引是一种消解微妙歧义的机制，主要用于基础库中类模板的构造函数（[§7.2.3](../ch07/7-2-parameterized-types.md#7.2.3)）。

若我们试图访问持有与期望类型不同的 `variant`，会抛出 `bad_variant_access`。

## 15.4.2 `optional`

可以把 `optional<A>` 看成一种特殊的 `variant`（类似 `variant<A,nothing>`），或看成“`A*` 要么指向对象要么为 `nullptr`”这一观念的推广。

`optional` 对那些可能返回对象也可能不返回对象的函数很有用：

```cpp
optional<string> compose_message(istream& s)
{
    string mess;

    // ... 从 s 读取并拼装消息 ...

    if (no_problems)
        return mess;
    return {};          // 空 optional
}
```

有了它，我们可以写：

```cpp
if (auto m = compose_message(cin))
    cout << *m;               // 注意解引用（*）
else {
    // ... 处理错误 ...
}
```

这对某些不喜欢异常的人有吸引力（见 [§4.4](../ch04/4-4-alternatives.md)）。注意对 `*` 的奇特用法。`optional` 被当作指向其对象的指针，而非对象本身。

与 `nullptr` 对应的 `optional` 是空对象 `{}`。例如：

```cpp
int sum(optional<int> a, optional<int> b)
{
    int res = 0;
    if (a) res += *a;
    if (b) res += *b;
    return res;
}

int x = sum(17, 19);         // 36
int y = sum(17, {});          // 17
int z = sum({}, {});            // 0
```

若我们试图访问不持有值的 `optional`，结果是未定义的；不会抛出异常。因此，`optional` 并不保证类型安全。不要尝试：

```cpp
int sum2(optional<int> a, optional<int> b)
{
    return *a + *b;     // 自讨苦吃
}
```

## 15.4.3 `any`

`any` 可以持有任意类型，并知道它持有何种类型（如果有的话）。它基本上是不受约束的 `variant` 版本：

```cpp
any compose_message(istream& s)
{
    string mess;

    // ... 从 s 读取并拼装消息 ...

    if (no_problems)
        return mess;                        // 返回 string
    else
        return error_number;          // 返回 int
}
```

当你用一个值赋值或初始化 `any` 时，它会记住该值的类型。之后，我们可以通过断言该值的期望类型来取出 `any` 所持有的值。例如：

```cpp
auto m = compose_message(cin);
string& s = any_cast<string>(m);
cout << s;
```

若我们试图访问持有与期望类型不同的 `any`，会抛出 `bad_any_access`。


﻿# 15.5 建议

[1] 库不见得庞大或复杂才有用；[§16.1](../ch16/16-1-introduction.md)。

[2] 资源是任何必须先获取再（显式或隐式）释放的东西；[§15.2.1](15-2-pointers.md#15.2.1)。

[3] 用资源句柄管理资源（RAII）；[§15.2.1](15-2-pointers.md#15.2.1)；[CG: R.1]。

[4] `T*` 的问题是它可以被用来表示任何东西，因而我们难以确定一个“裸”指针的用途；[§15.2.1](15-2-pointers.md#15.2.1)。

[5] 用 `unique_ptr` 指称多态类型的对象；[§15.2.1](15-2-pointers.md#15.2.1)；[CG: R.20]。

[6] （仅）用 `shared_ptr` 指称共享对象；[§15.2.1](15-2-pointers.md#15.2.1)；[CG: R.20]。

[7] 偏爱具有特定语义的资源句柄，而非智能指针；[§15.2.1](15-2-pointers.md#15.2.1)。

[8] 局部变量就够用时，不要用智能指针；[§15.2.1](15-2-pointers.md#15.2.1)。

[9] 偏爱 `unique_ptr` 而非 `shared_ptr`；[§6.3](../ch06/6-3-resource-mgmt.md)，[§15.2.1](15-2-pointers.md#15.2.1)。

[10] 仅在需要转移所有权职责时，才把 `unique_ptr` 或 `shared_ptr` 用作实参或返回值；[§15.2.1](15-2-pointers.md#15.2.1)；[CG: F.26] [CG: F.27]。

[11] 用 `make_unique()` 构造 `unique_ptr`；[§15.2.1](15-2-pointers.md#15.2.1)；[CG: R.22]。

[12] 用 `make_shared()` 构造 `shared_ptr`；[§15.2.1](15-2-pointers.md#15.2.1)；[CG: R.23]。

[13] 偏爱智能指针而非垃圾回收；[§6.3](../ch06/6-3-resource-mgmt.md)，[§15.2.1](15-2-pointers.md#15.2.1)。

[14] 偏爱 `span`，而非指针加计数接口；[§15.2.2](15-2-pointers.md#15.2.2)；[CG: F.24]。

[15] `span` 支持范围 `for`；[§15.2.2](15-2-pointers.md#15.2.2)。

[16] 需要具有 `constexpr` 大小的序列时，使用 `array`；[§15.3.1](15-3-containers.md#15.3.1)。

[17] 偏爱 `array` 而非内置数组；[§15.3.1](15-3-containers.md#15.3.1)；[CG: SL.con.2]。

[18] 若你需要 `N` 个比特，且 `N` 未必等于某个内置整数类型的比特数，使用 `bitset`；[§15.3.2](15-3-containers.md#15.3.2)。

[19] 不要过度使用 `pair` 与 `tuple`；具名 `struct` 往往带来更可读的代码；[§15.3.3](15-3-containers.md#15.3.3)。

[20] 使用 `pair` 时，用模板实参推导或 `make_pair()` 避免多余的类型说明；[§15.3.3](15-3-containers.md#15.3.3)。

[21] 使用 `tuple` 时，用模板实参推导或 `make_tuple()` 避免多余的类型说明；[§15.3.3](15-3-containers.md#15.3.3)；[CG: T.44]。

[22] 偏爱 `variant`，而非显式使用 `union`；[§15.4.1](15-4-alternatives.md#15.4.1)；[CG: C.181]。

[23] 用 `variant` 在一组替代中做选择时，考虑使用 `visit()` 与 `overloaded()`；[§15.4.1](15-4-alternatives.md#15.4.1)。

[24] 若 `variant`、`optional` 或 `any` 可能有多个替代，访问前先检查标签；[§15.4](15-4-alternatives.md)。
