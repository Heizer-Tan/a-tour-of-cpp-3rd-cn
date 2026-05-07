# 7.3 参数化操作

模板的用途远不止用元素类型参数化容器。特别是，它们广泛用于标准库中类型和算法的参数化（§12.8，§13.5）。

有三种表达由类型或值参数化的操作的方式：

===== 第 10 页 =====

- **函数模板**
- **函数对象**：一个可以携带数据并像函数一样被调用的对象
- **Lambda 表达式**：函数对象的简写符号

#### 7.3.1 函数模板

我们可以编写一个函数，计算 range-for 可以遍历的任何序列（例如容器）的元素值的和，如下所示：

```cpp
template<typename Sequence, typename Value>
Value sum(const Sequence& s, Value v)
{
    for (auto x : s)
        v += x;
    return v;
}
```

`Value` 模板参数和函数参数 `v` 的作用是允许调用者指定累加器的类型和初始值（用于累加和的变量）：

```cpp
void user(Vector<int>& vi, list<double>& ld, vector<complex<double>>& vc)
{
    int x = sum(vi, 0);                       // int 型 vector 的和（加 int）
    double d = sum(vi, 0.0);                  // int 型 vector 的和（加 double）
    double d = sum(ld, 0.0);                  // double 型 list 的和
    auto z = sum(vc, complex{0.0, 0.0});      // complex<double> 型 vector 的和
}
```

在 `double` 中累加 int 的意义在于优雅地处理大于最大 int 的和。注意 `sum<Sequence, Value>` 的模板参数类型是如何从函数参数推导出来的。幸运的是，我们不需要显式指定这些类型。

这个 `sum()` 是标准库 `accumulate()`（§17.3）的简化版本。

===== 第 11 页 =====

函数模板可以是成员函数，但不能是虚成员。编译器无法知道程序中此类模板的所有实例化，因此无法生成 vtable（§5.4）。

#### 7.3.2 函数对象

一种特别有用的模板是**函数对象**（有时称为 functor），用于定义可以像函数一样被调用的对象。例如：

```cpp
template<typename T>
class Less_than {
    const T val;   // 要比较的值
public:
    Less_than(const T& v) : val(v) { }
    bool operator()(const T& x) const { return x < val; }   // 调用运算符
};
```

名为 `operator()` 的函数实现了应用运算符 `()`，也称为“函数调用”或简称为“调用”。

我们可以为某些参数类型定义 `Less_than` 类型的命名变量：

```cpp
Less_than lti {42};             // lti(i) 将使用 < 比较 i 和 42 (i<42)
Less_than lts {"Backus"s};      // lts(s) 将使用 < 比较 s 和 "Backus" (s<"Backus")
Less_than<string> lts2 {"Naur"}; // "Naur" 是 C 风格字符串，所以需要 <string>
```

我们可以像调用函数一样调用这样的对象：

```cpp
void fct(int n, const string& s)
{
    bool b1 = lti(n);    // 如果 n<42 则为 true
    bool b2 = lts(s);    // 如果 s<"Backus" 则为 true
    // ...
}
```

===== 第 12 页 =====

函数对象被广泛用作算法的参数。例如，我们可以计算谓词返回 true 的值的出现次数：

```cpp
template<typename C, typename P>
int count(const C& c, P pred)   // 假设 C 是容器，P 是谓词
{
    int cnt = 0;
    for (const auto& x : c)
        if (pred(x))
            ++cnt;
    return cnt;
}
```

这是标准库 `count_if` 算法（§13.5）的简化版本。

有了概念（§8.2），我们可以形式化 `count()` 对其参数的假设，并在编译时检查它们。

**谓词**是我们可以调用并返回 true 或 false 的东西。例如：

```cpp
void f(const Vector<int>& vec, const list<string>& lst, int x, const string& s)
{
    cout << "number of values less than " << x << ": "
         << count(vec, Less_than{x}) << '\n';
    cout << "number of values less than " << s << ": "
         << count(lst, Less_than{s}) << '\n';
}
```

这里，`Less_than{x}` 构造了一个 `Less_than<int>` 类型的对象，其调用运算符与 int 类型的 `x` 的值进行比较；`Less_than{s}` 构造了一个与字符串 `s` 的值进行比较的对象。

函数对象的美妙之处在于它们携带着要与之比较的值。我们不需要为每个值（和每种类型）编写单独的函数，也不需要引入糟糕的全局变量来保存值。此外，对于像 `Less_than` 这样的简单函数对象，内联很简单，因此调用 `Less_than` 远比间接函数调用高效。携带数据的能力加上它们的效率使得函数对象作为算法的参数特别有用。

===== 第 13 页 =====

用于指定通用算法关键操作含义的函数对象（例如用于 `count()` 的 `Less_than`）有时被称为**策略对象**。

#### 7.3.3 Lambda 表达式

在 §7.3.2 中，我们将 `Less_than` 与其使用分开定义。这可能不方便。因此，有一种用于隐式生成函数对象的表示法：

```cpp
void f(const Vector<int>& vec, const list<string>& lst, int x, const string& s)
{
    cout << "number of values less than " << x << ": "
         << count(vec, [&](int a) { return a < x; }) << '\n';
    cout << "number of values less than " << s << ": "
         << count(lst, [&](const string& a) { return a < s; }) << '\n';
}
```

表示法 `[&](int a){ return a<x; }` 称为 **lambda 表达式**。它生成一个类似于 `Less_than<int>(x)` 的函数对象。`[&]` 是一个**捕获列表**，指定 lambda 体中使用的所有局部名称（如 `x`）将通过引用访问。如果我们只想“捕获” `x`，可以说 `[&x]`。如果我们想给生成的对象一个 `x` 的副本，可以说 `[x]`。不捕获任何东西是 `[]`，通过引用捕获所有使用的局部名称是 `[&]`，通过值捕获所有使用的局部名称是 `[=]`。

对于在成员函数内定义的 lambda，`[this]` 通过引用捕获当前对象，以便我们可以引用类成员。如果我们想要当前对象的副本，可以说 `[*this]`。

如果我们想捕获几个特定的对象，可以列出它们。在 `expect()` 的使用中（§4.5）使用了 `[i,this]`，这就是一个例子。

#### 7.3.3.1 Lambda 作为函数参数

使用 lambda 可以方便且简洁，但也可能晦涩难懂。对于非平凡的操作（比如超过一个简单表达式），我更喜欢命名该操作，以便更清楚地说明其目的，并使其在程序中的多个地方可用。

===== 第 14 页 =====

在 §5.5.3 中，我们注意到必须编写许多函数（如 `draw_all()` 和 `rotate_all()`）来对指针和 `unique_ptr` 向量的元素执行操作，这很烦人。函数对象（特别是 lambda）可以通过允许我们将容器的遍历与对每个元素执行的操作 specification 分离开来提供帮助。

首先，我们需要一个函数，将操作应用于容器元素所指向的每个对象：

```cpp
template<typename C, typename Oper>
void for_each(C& c, Oper op)   // 假设 C 是指针容器
{
    for (auto& x : c)
        op(x);   // 将 op() 传递给每个所指对象的引用
}
```

这是标准库 `for_each` 算法（§13.5）的简化版本。

现在，我们可以编写 §5.5 中的 `user()` 版本，而无需编写全套函数：

```cpp
void user()
{
    vector<unique_ptr<Shape>> v;
    while (cin)
        v.push_back(read_shape(cin));
    for_each(v, [](unique_ptr<Shape>& ps) { ps->draw(); });      // draw_all()
    for_each(v, [](unique_ptr<Shape>& ps) { ps->rotate(45); });   // rotate_all(45)
}
```

我通过引用将 `unique_ptr<Shape>` 传递给 lambda。这样 `for_each()` 就不必处理生命周期问题。

像函数一样，lambda 也可以是泛型的。例如：

```cpp
template<class S>
void rotate_and_draw(vector<S>& v, int r)
{
    for_each(v, [](auto& s) { s->rotate(r); s->draw(); });
}
```

这里，就像在变量声明中一样，`auto` 表示接受任何类型的值作为初始化器（参数被视为在调用中初始化形式参数）。这使得带有 `auto` 参数的 lambda 成为一个模板，即**泛型 lambda**。需要时，我们可以用概念约束参数（§8.1）。例如，我们可以定义 `Pointer_to_class` 来要求 `*` 和 `->`，并写：

```cpp
for_each(v, [](Pointer_to_class auto& s) { s->rotate(r); s->draw(); });
```

我们可以用任何可以 `draw()` 和 `rotate()` 的对象容器来调用这个泛型的 `rotate_and_draw()`。例如：

```cpp
void user()
{
    vector<unique_ptr<Shape>> v1;
    vector<Shape*> v2;
    // ...
    rotate_and_draw(v1, 45);
    rotate_and_draw(v2, 90);
}
```

为了更严格的检查，我们可以定义一个 `Pointer_to_Shape` 概念，指定一个类型要能够作为 shape 所需的属性。这将允许我们使用并非派生自 `Shape` 类的 shape。

#### 7.3.3.2 用于初始化的 Lambda

使用 lambda，我们可以将任何语句转换为表达式。这主要用于提供一个操作来计算一个值作为参数值，但这种能力是通用的。考虑一个复杂的初始化：

```cpp
enum class Init_mode { zero, seq, cpy, patrn };   // 初始化选项
void user(Init_mode m, int n, vector<int>& arg, Iterator p, Iterator q)
{
    vector<int> v;
    // 混乱的初始化代码：
    switch (m) {
    case zero:
        v = vector<int>(n);   // n 个元素初始化为 0
        break;
    case cpy:
        v = arg;
        break;
    }
    // ...
    if (m == seq)
        v.assign(p, q);       // 从序列 [p:q) 拷贝
    // ...
}
```

这是一个风格化的例子，但不幸的是并非不典型。我们需要在一组初始化数据结构的选项中进行选择（此处是 `v`），并且需要为不同的选项执行不同的计算。这样的代码通常很混乱，被认为是“为了效率”所必需的，并且是 bug 的来源：

- 变量可能在获得其预期值之前就被使用。
- “初始化代码”可能与其他代码混合，使其难以理解。
- 当“初始化代码”与其他代码混合时，更容易忘记一种情况。
- 这不是初始化，而是赋值（§1.9.2）。

相反，我们可以将其转换为用作初始化的 lambda：

```cpp
void user(Init_mode m, int n, vector<int>& arg, Iterator p, Iterator q)
{
    vector<int> v = [&] {
        switch (m) {
        case zero: return vector<int>(n);      // n 个元素初始化为 0
        case seq:  return vector<int>(p, q);   // 从序列 [p:q) 拷贝
        case cpy:  return arg;
        }
    }();
    // ...
}
```

我仍然“忘记”了一种情况，但现在更容易发现。在许多情况下，编译器会发现该问题并发出警告。

#### 7.3.3.3 Finally

析构函数提供了一种在对象使用后进行清理的通用隐式机制（RAII；§6.3），但如果我们需要进行一些与单个对象无关、或与没有析构函数的对象（例如，因为它是一个与 C 程序共享的类型）相关联的清理，该怎么办？我们可以定义一个 `finally()` 函数，它接受一个在作用域退出时执行的动作：

```cpp
void old_style(int n)
{
    void* p = malloc(n * sizeof(int));   // C 风格
    auto act = finally([&] { free(p); }); // 在作用域退出时调用 lambda
    // ... p 在作用域退出时被隐式释放
}
```

这是临时的，但比试图在函数的所有退出点上正确且一致地调用 `free(p)` 要好得多。

`finally()` 函数很简单：

```cpp
template <class F>
[[nodiscard]] auto finally(F f)
{
    return Final_action(f);
}
```

我使用了属性 `[[nodiscard]]` 以确保用户不会忘记将生成的 `Final_action` 复制到其动作意图所在的作用域。

提供必要析构函数的 `Final_action` 类可以如下所示：

```cpp
template <class F>
struct Final_action {
    explicit Final_action(F f) : act(f) { }
    ~Final_action() { act(); }
    F act;
};
```

在核心指南支持库（GSL）中有一个 `finally()`，并且有一个为标准库提供更精细的 `scope_exit` 机制的提案。
