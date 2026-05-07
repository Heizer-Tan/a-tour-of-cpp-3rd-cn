===== 第 1 页 =====

7

# 模板

请在此处引用您的话。 – B. Stroustrup

- 引言
- 参数化类型
  - 受约束的模板参数；值模板参数；模板参数推导
- 参数化操作
  - 函数模板；函数对象；Lambda 表达式
- 模板机制
  - 变量模板；别名；编译时 if
- 建议

### 7.1 引言

===== 第 2 页 =====

想要 `vector` 的人不太可能总是想要一个 `double` 的 vector。vector 是一个通用概念，与浮点数的概念无关。因此，vector 的元素类型应该独立表示。**模板**是我们用一组类型或值进行参数化的类或函数。我们使用模板来表示那些最好被理解为通用的思想——通过指定参数（例如用 `double` 作为 vector 的元素类型）可以从这些通用思想生成具体的类型和函数。

本章重点介绍语言机制。第 8 章将介绍编程技巧，而库章节（第 10-18 章）则提供了许多示例。

### 7.2 参数化类型

我们可以将 §5.2.2 中的 `vector-of-doubles` 类型推广为 `vector-of-anything` 类型，方法是将其变为模板，并将具体的 `double` 类型替换为类型参数。例如：

```cpp
template<typename T>
class Vector {
private:
    T* elem;   // elem 指向一个包含 sz 个 T 类型元素的数组
    int sz;
public:
    explicit Vector(int s);                // 构造函数：建立不变式，获取资源
    ~Vector() { delete[] elem; }           // 析构函数：释放资源
    // ... 拷贝和移动操作 ...
    T& operator[](int i);                  // 用于非 const Vector
    const T& operator[](int i) const;      // 用于 const Vector（§5.2.1）
    int size() const { return sz; }
};
```

`template<typename T>` 前缀使得 `T` 成为其后面声明的类型参数。这是 C++ 对数学中“对于所有 T”或更准确地说“对于所有类型 T”的表示。如果你想要数学上的“对于所有满足 P(T) 的 T”，你需要使用概念（§7.2.1，§8.2）。使用 `class` 来引入类型参数等价于使用 `typename`，在旧代码中我们经常看到 `template<class T>` 作为前缀。

成员函数可以类似地定义：

```cpp
template<typename T>
Vector<T>::Vector(int s)
{
    if (s < 0)
        throw length_error("Vector constructor: negative size");
    elem = new T[s];
    sz = s;
}

template<typename T>
const T& Vector<T>::operator[](int i) const
{
    if (i < 0 || size() <= i)
        throw out_of_range("Vector::operator[]");
    return elem[i];
}
```

给定这些定义，我们可以像这样定义 `Vector`：

```cpp
Vector<char> vc(200);                     // 200 个字符的 vector
Vector<string> vs(17);                    // 17 个字符串的 vector
Vector<List<int>> vli(45);                // 45 个整数列表的 vector
```

`Vector<List<int>>` 中的 `>>` 终止了嵌套的模板参数；它不是一个放错位置的输入运算符。

我们可以这样使用 `Vector`：

```cpp
void write(const Vector<string>& vs)      // 一些字符串的 Vector
{
    for (int i = 0; i != vs.size(); ++i)
        cout << vs[i] << '\n';
}
```

===== 第 4 页 =====

为了支持我们的 `Vector` 使用 range-for 循环，我们必须定义合适的 `begin()` 和 `end()` 函数：

```cpp
template<typename T>
T* begin(Vector<T>& x)
{
    return &x[0];   // 指向第一个元素或最后一个元素之后的位置
}

template<typename T>
T* end(Vector<T>& x)
{
    return &x[0] + x.size();   // 指向最后一个元素之后的位置
}
```

有了这些，我们可以写：

```cpp
void write2(Vector<string>& vs)           // 一些字符串的 Vector
{
    for (auto& s : vs)
        cout << s << '\n';
}
```

类似地，我们可以将 list、vector、map（即关联数组）、unordered map（即哈希表）等定义为模板（第 12 章）。

模板是一种编译时机制，因此与手工编写的代码相比，使用模板不会产生运行时开销。事实上，为 `Vector<double>` 生成的代码与为第 5 章中的 `Vector` 版本生成的代码完全相同。此外，为标准库 `vector<double>` 生成的代码可能更好（因为在其实现上投入了更多的精力）。

模板加上一组模板实参称为**实例化**或**特化**。在编译过程的后期，在实例化时，会为程序中使用的每个实例化生成代码（§8.5）。

#### 7.2.1 受约束的模板参数

===== 第 5 页 =====

大多数情况下，模板只对满足特定条件的模板实参有意义。例如，`Vector` 通常提供拷贝操作，如果它提供拷贝操作，则必须要求其元素是可拷贝的。也就是说，我们必须要求 `Vector` 的模板参数不仅仅是一个 `typename`，而是一个 `Element`，其中“Element”指定了可以作为元素的类型的要求：

```cpp
template<Element T>
class Vector {
private:
    T* elem;   // elem 指向一个包含 sz 个 T 类型元素的数组
    int sz;
    // ...
};
```

这个 `template<Element T>` 前缀是 C++ 对数学中“对于所有满足 Element(T) 的 T”的表示；也就是说，`Element` 是一个谓词，用于检查 `T` 是否具有 `Vector` 所需的所有属性。这样的谓词称为**概念**（§8.2）。为模板实参指定了概念的实参称为**受约束实参**，而实参受约束的模板称为**受约束模板**。

标准库元素类型的要求有点复杂（§12.2），但对于我们简单的 `Vector` 来说，`Element` 可以是类似标准库概念 `copyable`（§14.5）的东西。

尝试使用不满足其要求的类型来实例化模板是编译时错误。例如：

```cpp
Vector<int> v1;        // OK：可以拷贝 int
Vector<thread> v2;     // 错误：不能拷贝标准 thread（§18.2）
```

因此，概念使得编译器可以在使用点进行类型检查，提供比无约束模板实参更早、更好的错误信息。C++ 在 C++20 之前没有正式支持概念，因此旧代码使用无约束模板实参，并将要求留给了文档。然而，从模板生成的代码会进行类型检查，因此即使是无约束模板代码也与手写代码一样类型安全。对于无约束参数，这种类型检查必须等到所有相关实体的类型都可用时才能进行，因此它可能在编译过程的后期（实例化时，§8.5）发生，而且错误信息通常非常糟糕。概念检查是一种纯粹的编译时机制，生成的代码与无约束模板一样好。

#### 7.2.2 值模板参数

除了类型参数，模板还可以接受值参数。例如：

```cpp
template<typename T, int N>
struct Buffer {
    constexpr int size() { return N; }
    T elem[N];
    // ...
};
```

值参数在许多上下文中都很有用。例如，`Buffer` 允许我们创建任意大小的缓冲区，而不需要使用自由存储区（动态内存）：

```cpp
Buffer<char, 1024> glob;          // 全局字符缓冲区（静态分配）

void fct()
{
    Buffer<int, 10> buf;          // 局部整数缓冲区（在栈上）
    // ...
}
```

遗憾的是，由于晦涩的技术原因，字符串字面量目前还不能作为模板值参数。然而，在某些上下文中，用字符串值进行参数化的能力至关重要。幸运的是，我们可以使用一个保存字符串字符的数组：

```cpp
template<char* s>
void outs() { cout << s; }

char arr[] = "Weird workaround!";
void use()
{
    outs<"straightforward use">();    // 错误（目前）
    outs<arr>();                      // 输出：Weird workaround!
}
```

在 C++ 中，通常都有变通办法；我们不需要为每个用例都提供直接支持。

#### 7.2.3 模板参数推导

在将一个类型定义为模板的实例化时，我们必须指定其模板参数。考虑使用标准库模板 `pair`：

```cpp
pair<int, double> p = {1, 5.2};
```

必须指定模板参数类型可能会很繁琐。幸运的是，在许多上下文中，我们可以直接让 `pair` 的构造函数从初始化器推导模板参数：

```cpp
pair p = {1, 5.2};   // p 是一个 pair<int, double>
```

容器提供了另一个例子：

```cpp
template<typename T>
class Vector {
public:
    Vector(int);
    Vector(initializer_list<T>);   // 初始化列表构造函数
    // ...
};

Vector v1 {1, 2, 3};               // 从初始化器元素类型推导 v1 的元素类型：int
Vector v2 = v1;                    // 从 v1 的元素类型推导 v2 的元素类型：int
auto p = new Vector{1, 2, 3};      // p 是一个 Vector<int>*
```

===== 第 8 页 =====

显然，这简化了符号，并能消除因打错冗余模板参数类型而带来的烦恼。然而，它并非万能药。像所有其他强大的机制一样，推导也可能带来意外。考虑：

```cpp
Vector<string> vs {"Hello", "World"};        // OK：Vector<string>
Vector vs1 {"Hello", "World"};               // OK：推导为 Vector<const char*>（奇怪！）
Vector vs2 {"Hello"s, "World"s};             // OK：推导为 Vector<string>
Vector vs3 {"Hello"s, "World"};              // 错误：初始化列表的类型不统一
Vector<string> vs4 {"Hello"s, "World"};      // OK：元素类型是显式指定的
```

C 风格字符串字面量的类型是 `const char*`（§1.7.1）。如果这不是 `vs1` 所期望的，我们必须显式指定元素类型，或使用 `s` 后缀将其转换为真正的 `string`（§10.2）。

如果初始化列表中的元素具有不同的类型，我们无法推导出唯一的元素类型，因此会出现歧义错误。

有时，我们需要解决歧义。例如，标准库 `vector` 有一个接受一对界定序列的迭代器的构造函数，还有一个可以接受一对值的初始化列表构造函数。考虑：

```cpp
template<typename T>
class Vector {
public:
    Vector(initializer_list<T>);                 // 初始化列表构造函数
    template<typename Iter>
    Vector(Iter b, Iter e);                      // [b:e) 迭代器对构造函数
    struct iterator {
        using value_type = T;
        // ...
    };
    iterator begin();
    // ...
};
```

===== 第 9 页 =====

我们可以使用概念（§8.2）来解决这个问题，但标准库和许多其他重要的代码库是在我们拥有语言支持概念之前几十年编写的。对于这些代码，我们需要一种方式来表达“一对相同类型的值应被视为迭代器”。在 `Vector` 的声明之后添加一个**推导指引**正是实现了这一点：

```cpp
template<typename Iter>
Vector(Iter, Iter) -> Vector<typename Iter::value_type>;
```

现在我们有：

```cpp
Vector v1 {1, 2, 3, 4, 5};                     // 元素类型是 int
Vector v2(v1.begin(), v1.begin() + 2);         // 迭代器对：元素类型是 int
Vector v3 {v1.begin(), v1.begin() + 2};        // 元素类型是 Vector<int>::iterator
```

`{}` 初始化语法总是优先选择 `initializer_list` 构造函数（如果存在），因此 `v3` 是一个迭代器的 vector：`Vector<Vector<int>::iterator>`。

当不希望使用 `initializer_list` 时，通常使用 `()` 初始化语法（§12.2）。

推导指引的效果通常很微妙，因此最好将类模板设计为不需要推导指引。

喜欢首字母缩略词的人将“类模板参数推导”称为 **CTAD**。

### 7.3 参数化操作

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

### 7.4 模板机制

为了定义良好的模板，我们需要一些支持性的语言设施：

- 依赖于类型的值：**变量模板**（§7.4.1）
- 类型和模板的别名：**别名模板**（§7.4.2）
- 编译时选择机制：**`if constexpr`**（§7.4.3）
- 查询类型和表达式属性的编译时机制：**requires 表达式**（§8.2.3）

此外，`constexpr` 函数（§1.6）和 `static_assert`（§4.5.2）也经常参与模板的设计和使用。

这些基本机制主要是用于构建通用、基础抽象的工具。

#### 7.4.1 变量模板

当我们使用一个类型时，我们通常需要该类型的常量和值。当我们使用类模板时当然也是如此：当我们定义 `C<T>` 时，我们通常需要类型 `C<T>` 和其他依赖于 `T` 的类型的常量和变量。这里有一个来自流体动力学仿真的例子 [Garcia,2015]：

```cpp
template <class T>
constexpr T viscosity = 0.4;

template <class T>
constexpr space_vector<T> external_acceleration = { T(), T{-9.8}, T() };

auto vis2 = 2 * viscosity<double>;
auto acc = external_acceleration<float>;
```

这里，`space_vector` 是一个三维向量。

奇怪的是，大多数变量模板似乎是常量。但话又说回来，许多变量也是如此。术语没有跟上我们关于不可变性的概念。

当然，我们可以使用合适类型的任意表达式作为初始化器。考虑：

```cpp
template<typename T, typename T2>
constexpr bool Assignable = is_assignable<T, T2>::value;   // is_assignable 是一个类型谓词

template<typename T>
void testing()
{
    static_assert(Assignable<T&, double>, "can't assign a double to a T");
    static_assert(Assignable<T&, string>, "can't assign a string to a T");
}
```

经过一些重要的演变后，这个想法成为概念定义的核心（§8.2）。

标准库使用变量模板来提供数学常数，例如 `pi` 和 `log2e`（§17.9）。

#### 7.4.2 别名

令人惊讶的是，为类型或模板引入同义词常常很有用。例如，标准头文件 `<cstddef>` 包含了别名 `size_t` 的定义，可能如下：

```cpp
using size_t = unsigned int;
```

名为 `size_t` 的实际类型是实现相关的，因此在另一种实现中 `size_t` 可能是 `unsigned long`。拥有别名 `size_t` 使得程序员能够编写可移植的代码。

===== 第 20 页 =====

参数化类型为其模板参数相关的类型提供别名非常常见。例如：

```cpp
template<typename T>
class Vector {
public:
    using value_type = T;
    // ...
};
```

事实上，每个标准库容器都提供 `value_type` 作为其元素类型的名称（第 12 章）。这使得我们能够编写适用于遵循此约定的任何容器的代码。例如：

```cpp
template<typename C>
using Value_type = typename C::value_type;   // C 的元素类型

template<typename Container>
void algo(Container& c)
{
    Vector<Value_type<Container>> vec;       // 在此保存结果
    // ...
}
```

这个 `Value_type` 是标准库 `range_value_t`（§16.4.4）的简化版本。别名机制可用于通过绑定部分或全部模板参数来定义新的模板。例如：

```cpp
template<typename Key, typename Value>
class Map {
    // ...
};

template<typename Value>
using String_map = Map<string, Value>;

String_map<int> m;   // m 是一个 Map<string, int>
```

#### 7.4.3 编译时 if

===== 第 21 页 =====

考虑编写一个可以使用两个函数 `slow_and_safe(T)` 或 `simple_and_fast(T)` 之一实现的操作。这类问题在基础代码中比比皆是，在这些代码中，通用性和最佳性能至关重要。如果涉及类层次结构，基类可以提供 `slow_and_safe` 通用操作，派生类可以用 `simple_and_fast` 实现覆盖它。

或者，我们可以使用编译时 `if`：

```cpp
template<typename T>
void update(T& target)
{
    if constexpr (is_trivially_copyable_v<T>)
        simple_and_fast(target);   // 用于“普通旧数据”
    else
        slow_and_safe(target);     // 用于更复杂的类型
}
```

`is_trivially_copyable_v<T>` 是一个类型谓词（§16.4.1），告诉我们一个类型是否可以平凡拷贝。

编译器只检查 `if constexpr` 中被选中的分支。这个解决方案提供了最佳性能以及优化的局部性。

重要的是，`if constexpr` **不是**文本操作机制，也不能用于破坏语法、类型和作用域的常规规则。例如，这是一个试图有条件地将调用包装在 `try` 块中的幼稚且失败的尝试：

```cpp
template<typename T>
void bad(T arg)
{
    if constexpr (!is_trivially_copyable_v<T>)
        try {
            g(arg);
    if constexpr (!is_trivially_copyable_v<T>)
        catch(...) { /* ... */ }   // 语法错误
}
```

===== 第 22 页 =====

允许这样的文本操作会严重损害代码的可读性，并为依赖现代程序表示技术（如“抽象语法树”）的工具带来问题。

许多此类尝试的黑客技巧也是不必要的，因为存在不违反作用域规则的更干净的解决方案。例如：

```cpp
template<typename T>
void good(T arg)
{
    if constexpr (is_trivially_copyable_v<T>)
        g(arg);
    else
        try {
            g(arg);
        } catch (...) { /* ... */ }
}
```

### 7.5 建议

[1] 使用模板来表达适用于多种参数类型的算法；§7.1；[CG: T.2]。
[2] 使用模板来表达容器；§7.2；[CG: T.3]。
[3] 使用模板来提高代码的抽象层次；§7.2；[CG: T.1]。
[4] 模板是类型安全的，但对于无约束模板，检查发生得太晚；§7.2。
[5] 让构造函数或函数模板推导类模板参数类型；§7.2.3。
[6] 使用函数对象作为算法的参数；§7.3.2；[CG: T.40]。
[7] 如果你只需要在一个地方使用简单的函数对象，请使用 lambda；§7.3.2。
[8] 虚函数成员不能是模板成员函数；§7.3.1。
[9] 使用 `finally()` 为没有析构函数但需要“清理操作”的类型提供 RAII；§7.3.3.3。

===== 第 23 页 =====

[10] 使用模板别名来简化符号并隐藏实现细节；§7.4.2。
[11] 使用 `if constexpr` 来提供无运行时开销的替代实现；§7.4.3。