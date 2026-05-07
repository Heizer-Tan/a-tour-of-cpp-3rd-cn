===== 第 1 页 =====

13

# 算法

如无必要，勿增实体。 – 奥卡姆的威廉

- 引言
- 迭代器的使用
- 迭代器类型
- 流迭代器
- 谓词的使用
- 算法概览
- 并行算法
- 建议

===== 第 2 页 =====

### 13.1 引言

像 `list` 或 `vector` 这样的数据结构本身并不是很有用。要使用它，我们需要基本的访问操作，例如添加和删除元素（就像 `list` 和 `vector` 提供的那样）。此外，我们很少仅仅在容器中存储对象。我们会对它们排序、打印、提取子集、删除元素、搜索对象等。因此，标准库除了提供最常见的容器类型外，还提供了最常见的容器算法。例如，我们可以简单高效地对一个 `Entry` 的 `vector` 进行排序，并将每个唯一的 `vector` 元素的副本放入一个 `list` 中：

```cpp
void f(vector<Entry>& vec, list<Entry>& lst)
{
    sort(vec.begin(), vec.end());                     // 使用 < 作为排序准则
    unique_copy(vec.begin(), vec.end(), lst.begin()); // 不拷贝相邻的相等元素
}
```

为此，必须为 `Entry` 定义小于（`<`）和等于（`==`）。例如：

```cpp
bool operator<(const Entry& x, const Entry& y)   // 小于
{
    return x.name < y.name;                      // 按姓名对 Entry 排序
}
```

标准算法是用（半开）元素序列来表达的。一个序列由一对迭代器表示，指定第一个元素和最后一个元素之后的位置：

[图片描述：序列 [begin:end) 示意图]

===== 第 3 页 =====

在示例中，`sort()` 对由迭代器对 `vec.begin()` 和 `vec.end()` 定义的序列进行排序，这恰好是 `vector` 的所有元素。对于写入（输出），我们只需要指定要写入的第一个元素。如果写入多个元素，则初始元素之后的元素将被覆盖。因此，为避免错误，`lst` 必须至少与 `vec` 中唯一值的数量一样多的元素。

不幸的是，标准库没有提供一个抽象来支持写入容器时的范围检查。然而，我们可以自己定义一个：

```cpp
template<typename C>
class Checked_iter {
public:
    using value_type = typename C::value_type;
    using difference_type = int;

    Checked_iter() { throw Missing_container{}; }   // 概念 forward_iterator 要求
    Checked_iter(C& cc) : pc{ &cc } {}
    Checked_iter(C& cc, typename C::iterator pp) : pc{ &cc }, p{ pp } {}

    Checked_iter& operator++() { check_end(); ++p; return *this; }
    Checked_iter operator++(int) { check_end(); auto t{ *this }; ++p; return t; }
    value_type& operator*() const { check_end(); return *p; }

    bool operator==(const Checked_iter& a) const { return p == a.p; }
    bool operator!=(const Checked_iter& a) const { return p != a.p; }
private:
    void check_end() const { if (p == pc->end()) throw Overflow{}; }
    C* pc {};                     // 默认初始化为 nullptr
    typename C::iterator p = pc->begin();
};
```

显然，这不是标准库的质量，但它展示了这个想法：

```cpp
vector<int> v1 {1, 2, 3};        // 三个元素
vector<int> v2(2);               // 两个元素

copy(v1, v2.begin());            // 将会溢出
copy(v1, Checked_iter{v2});      // 将会抛出异常
```

===== 第 4 页 =====

在读取并排序的例子中，如果我们想把唯一的元素放到一个新的 `list` 中，我们可以这样写：

```cpp
list<Entry> f(vector<Entry>& vec)
{
    list<Entry> res;
    sort(vec.begin(), vec.end());
    unique_copy(vec.begin(), vec.end(), back_inserter(res));   // 追加到 res 中
    return res;
}
```

调用 `back_inserter(res)` 为 `res` 构造一个迭代器，该迭代器在容器的末尾添加元素，并扩展容器以腾出空间。这使我们不必先分配固定大小的空间再填充它。因此，标准容器加上 `back_inserter()` 消除了使用容易出错的、显式的 C 风格 `realloc()` 内存管理的需要。标准库 `list` 有一个移动构造函数（§6.2.2），这使得按值返回 `res` 是高效的（即使对于包含数千个元素的 `list` 也是如此）。

当我们发现像 `sort(vec.begin(), vec.end())` 这样的迭代器对风格的代码变得繁琐时，我们可以使用算法的 range 版本并写成 `sort(vec)`（§13.5）。这两个版本是等价的。类似地，范围 `for` 循环大致等价于直接使用迭代器的 C 风格循环：

```cpp
for (auto& x : v) cout << x;                 // 输出 v 的所有元素
for (auto p = v.begin(); p != v.end(); ++p) cout << *p;   // 输出 v 的所有元素
```

除了更简单和更不易出错之外，范围 `for` 版本通常也更高效。

## 13.2 迭代器的使用

对于一个容器，可以获得几个指向有用元素的迭代器；`begin()` 和 `end()` 是最好的例子。此外，许多算法返回迭代器。例如，标准算法 `find` 在一个序列中查找一个值，并返回指向找到的元素的迭代器：

```cpp
bool has_c(const string& s, char c)   // s 是否包含字符 c？
{
    auto p = find(s.begin(), s.end(), c);
    if (p != s.end())
        return true;
    else
        return false;
}
```

===== 第 5 页 =====

像许多标准库搜索算法一样，`find` 返回 `end()` 表示“未找到”。一个等价但更短的 `has_c()` 定义是：

```cpp
bool has_c(const string& s, char c)   // s 是否包含字符 c？
{
    return find(s, c) != s.end();
}
```

一个更有趣的练习是找出字符串中某个字符的所有出现位置。我们可以将出现位置的集合作为 `vector<char*>` 返回。返回 `vector` 是高效的，因为 `vector` 提供了移动语义（§6.2.1）。假设我们希望修改找到的位置，我们传递一个非 `const` 的 `string`：

```cpp
vector<string::iterator> find_all(string& s, char c)   // 找出 s 中所有 c 的位置
{
    vector<string::iterator> res;
    for (auto p = s.begin(); p != s.end(); ++p)
        if (*p == c)
            res.push_back(p);
    return res;
}
```

我们使用传统的循环遍历字符串，通过 `++` 每次将迭代器 `p` 向前移动一个元素，并通过解引用运算符 `*` 查看元素。我们可以这样测试 `find_all()`：

```cpp
void test()
{
    string m {"Mary had a little lamb"};
    for (auto p : find_all(m, 'a'))
        if (*p != 'a')
            cerr << "a bug!\n";
}
```

那个 `find_all()` 的调用可以图示如下：

[图片描述：find_all 返回的迭代器向量]

迭代器和标准算法在每个有意义的标准容器上都能等效地工作。因此，我们可以泛化 `find_all()`：

```cpp
template<typename C, typename V>
vector<typename C::iterator> find_all(C& c, V v)   // 找出 c 中所有 v 的位置
{
    vector<typename C::iterator> res;
    for (auto p = c.begin(); p != c.end(); ++p)
        if (*p == v)
            res.push_back(p);
    return res;
}
```

这里需要 `typename` 来告知编译器 `C` 的 `iterator` 应该是一个类型，而不是某个类型的值（比如整数 7）。

或者，我们可以返回一个指向元素的普通指针的向量：

```cpp
template<typename C, typename V>
auto find_all(C& c, V v)   // 找出 c 中所有 v 的位置
{
    vector<range_value_t<C>*> res;
    for (auto& x : c)
        if (x == v)
            res.push_back(&x);
    return res;
}
```

===== 第 7 页 =====

在我做这件事的同时，我还使用范围 `for` 循环和标准库的 `range_value_t`（§16.4.4）简化了代码，以命名元素的类型。`range_value_t` 的一个简化版本可以这样定义：

```cpp
template<typename T>
using range_value_type_t = T::value_type;
```

使用任一版本的 `find_all()`，我们可以写：

```cpp
void test()
{
    string m {"Mary had a little lamb"};

    for (auto p : find_all(m, 'a'))   // p 是 string::iterator
        if (*p != 'a')
            cerr << "string bug!\n";

    list<int> ld {1, 2, 3, 1, -11, 2};
    for (auto p : find_all(ld, 1))    // p 是 list<int>::iterator
        if (*p != 1)
            cerr << "list bug!\n";

    vector<string> vs {"red", "blue", "green", "green", "orange", "green"};
    for (auto p : find_all(vs, "red"))   // p 是 vector<string>::iterator
        if (*p != "red")
            cerr << "vector bug!\n";

    for (auto p : find_all(vs, "green"))
        *p = "vert";
}
```

迭代器用于将算法与容器分离。算法通过迭代器操作其数据，对存储元素的容器一无所知。

===== 第 8 页 =====

相反，容器对其元素上运行的算法一无所知；它所做的只是在请求时提供迭代器（例如 `begin()` 和 `end()`）。这种数据存储与算法分离的模型产生了非常通用和灵活的软件。

[图片描述：算法通过迭代器操作容器]

## 13.3 迭代器类型

迭代器到底是什么？任何特定的迭代器都是某个类型的对象。然而，有许多不同的迭代器类型——迭代器需要持有为特定容器类型执行其工作所需的信息。这些迭代器类型可以和容器以及它们所服务的专门需求一样不同。例如，`vector` 的迭代器可以是一个普通的指针，因为指针是指向 `vector` 元素的一种相当合理的方式：

[图片描述：vector 的迭代器作为指针]

或者，`vector` 迭代器可以实现为指向 `vector` 的指针加上一个索引：

[图片描述：vector 迭代器实现为指针+索引]

使用这样的迭代器可以进行范围检查。

`list` 迭代器必须比指向元素的简单指针更复杂，因为 `list` 的元素通常不知道其下一个元素的位置。因此，`list` 迭代器可能是一个指向链接的指针：

[图片描述：list 迭代器指向链接节点]

所有迭代器的共同点是它们的语义和操作的命名。例如，对任何迭代器应用 `++` 都会产生一个指向下一个元素的迭代器。类似地，`*` 产生迭代器所指的元素。事实上，任何遵守这样几条简单规则的对象都是一个迭代器。迭代器是一个通用的概念（§8.2），不同类型的迭代器作为标准库概念提供，例如 `forward_iterator` 和 `random_access_iterator`（§14.5）。此外，用户很少需要知道特定迭代器的类型；每个容器“知道”其迭代器类型，并在常规名称 `iterator` 和 `const_iterator` 下提供它们。例如，`list<Entry>::iterator` 是 `list<Entry>` 的通用迭代器类型。我们很少需要担心该类型是如何定义的细节。

在某些情况下，迭代器不是成员类型，因此标准库提供了 `iterator_t<X>`，它在定义了 `X` 的迭代器的任何地方都能工作。

### 13.3.1 流迭代器

===== 第 10 页 =====

迭代器是处理容器中元素序列的通用且有用的概念。然而，容器并不是唯一存在元素序列的地方。例如，输入流产生一个值序列，我们将一个值序列写入输出流。因此，迭代器的概念可以有效地应用于输入和输出。

要创建 `ostream_iterator`，我们需要指定将使用哪个流以及写入该流的对象类型。例如：

```cpp
ostream_iterator<string> oo {cout};   // 将字符串写入 cout
```

对 `*oo` 赋值的效果是将该值写入 `cout`。例如：

```cpp
int main()
{
    *oo = "Hello, ";   // 相当于 cout << "Hello, "
    ++oo;
    *oo = "world!\n";  // 相当于 cout << "world!\n"
}
```

这是向标准输出写入规范消息的另一种方式。`++oo` 是为了模拟通过指针写入数组。这样，我们就可以在流上使用算法。例如：

```cpp
vector<string> v{"Hello", ", ", "World!\n"};
copy(v, oo);
```

类似地，`istream_iterator` 允许我们将输入流视为只读容器。同样，我们必须指定要使用的流和期望的值类型：

```cpp
istream_iterator<string> ii {cin};
```

输入迭代器成对使用以表示一个序列，因此我们必须提供一个 `istream_iterator` 来指示输入的结束。这是默认的 `istream_iterator`：

```cpp
istream_iterator<string> eos {};
```

===== 第 11 页 =====

通常，`istream_iterator` 和 `ostream_iterator` 不直接使用。相反，它们作为参数提供给算法。例如，我们可以编写一个简单的程序来读取一个文件，对读取的单词进行排序，消除重复项，并将结果写入另一个文件：

```cpp
int main()
{
    string from, to;
    cin >> from >> to;                     // 获取源文件和目标文件名

    ifstream is {from};                    // 文件 "from" 的输入流
    istream_iterator<string> ii {is};      // 流的输入迭代器
    istream_iterator<string> eos {};       // 输入哨兵

    ofstream os {to};                      // 文件 "to" 的输出流
    ostream_iterator<string> oo {os, "\n"}; // 输出迭代器，并带有分隔符

    vector<string> b {ii, eos};            // b 是从输入初始化的 vector
    sort(b);                               // 对缓冲区进行排序

    unique_copy(b, oo);                    // 将缓冲区复制到输出，丢弃重复项

    return !is.eof() || !os;               // 返回错误状态（§1.2.1, §11.4）
}
```

这里使用了 `sort()` 和 `unique_copy()` 的 range 版本。我也可以直接使用迭代器，例如 `sort(b.begin(), b.end())`，这在旧代码中很常见。

请记住，要同时使用标准库算法的传统迭代器版本和其范围版本，我们需要显式限定范围版本的调用，或者使用 `using` 声明（§9.3.2）：

```cpp
copy(v, oo);                    // 可能有歧义
ranges::copy(v, oo);            // OK
using ranges::copy;             // 从此处开始 copy(v, oo) 是 OK 的
copy(v, oo);                    // OK
```

===== 第 12 页 =====

`ifstream` 是可以附加到文件的 `istream`（§11.7.2），而 `ofstream` 是可以附加到文件的 `ostream`。`ostream_iterator` 的第二个参数用于分隔输出值。

实际上，这个程序比需要的要长。我们将字符串读入一个 `vector`，然后对它们进行 `sort()`，然后将它们写出，并消除重复项。一个更优雅的解决方案是根本不存储重复项。这可以通过将字符串保存在一个 `set` 中来实现，因为 `set` 不会保留重复项，并且会保持其元素有序（§12.5）。这样，我们可以用一行使用 `set` 的代码替换两行使用 `vector` 的代码，并用更简单的 `copy()` 替换 `unique_copy()`：

```cpp
set<string> b {ii, eos};   // 从输入收集字符串
copy(b, oo);               // 将缓冲区复制到输出
```

我们只使用了一次名称 `ii`、`eos` 和 `oo`，因此我们可以进一步缩减程序的大小：

```cpp
int main()
{
    string from, to;
    cin >> from >> to;   // 获取源文件和目标文件名

    ifstream is {from};  // 文件 "from" 的输入流
    ofstream os {to};    // 文件 "to" 的输出流

    set<string> b {istream_iterator<string>{is}, istream_iterator<string>{}};   // 从输入收集字符串
    copy(b, ostream_iterator<string>{os, "\n"});   // 将缓冲区复制到输出

    return !is.eof() || !os;   // 返回错误状态（§1.2.1, §11.4）
}
```

这种最后的简化是否提高了可读性，取决于个人品味和经验。

## 13.4 谓词的使用

===== 第 13 页 =====

到目前为止的例子中，算法只是“内置”了对序列每个元素要执行的操作。然而，我们经常希望将该操作作为算法的参数。例如，`find` 算法（§13.2，§13.5）提供了一种查找特定值的便捷方法。一个更通用的变体会查找满足指定要求的元素，即**谓词**。例如，我们可能想要搜索一个 `map`，找到第一个大于 42 的值。`map` 允许我们将其元素作为（键，值）对序列来访问，因此我们可以搜索 `map<string,int>` 的序列，寻找一个 `pair<const string,int>`，其中 `int` 大于 42：

```cpp
void f(map<string,int>& m)
{
    auto p = find_if(m, Greater_than{42});
    // ...
}
```

这里，`Greater_than` 是一个函数对象（§7.3.2），持有一个值（42），用于与类型为 `pair<string,int>` 的 `map` 条目进行比较：

```cpp
struct Greater_than {
    int val;
    Greater_than(int v) : val{v} { }
    bool operator()(const pair<string,int>& r) const { return r.second > val; }
};
```

或者，等价地，我们可以使用 lambda 表达式（§7.3.2）：

```cpp
auto p = find_if(m, [](const auto& r) { return r.second > 42; });
```

谓词不应修改它所应用的元素。

## 13.5 算法概览

算法的一般定义是“一个有限规则集，它给出了解决特定问题集的操作序列[并且]具有五个重要特征：有限性……确定性……输入……输出……有效性”[Knuth,1968, §1.1]。在 C++ 标准库的上下文中，

===== 第 14 页 =====

算法是在元素序列上操作的函数模板。

标准库提供了几十种算法。算法定义在命名空间 `std` 中，并在 `<algorithm>` 和 `<numeric>` 头文件中提供。这些标准库算法都以序列作为输入。从 `b` 到 `e` 的半开序列记为 `[b:e)`。以下是一些示例：

**选定的标准算法（`<algorithm>`）**

| 算法 | 描述 |
|------|------|
| `f = for_each(b,e,f)` | 对 `[b:e)` 中的每个元素 `x` 执行 `f(x)` |
| `p = find(b,e,x)` | `p` 是 `[b:e)` 中第一个满足 `*p == x` 的位置 |
| `p = find_if(b,e,f)` | `p` 是 `[b:e)` 中第一个满足 `f(*p)` 的位置 |
| `n = count(b,e,x)` | `n` 是 `[b:e)` 中满足 `*q == x` 的元素 `*q` 的数量 |
| `n = count_if(b,e,f)` | `n` 是 `[b:e)` 中满足 `f(*q)` 的元素 `*q` 的数量 |
| `replace(b,e,v,v2)` | 将 `[b:e)` 中满足 `*q == v` 的元素 `*q` 替换为 `v2` |
| `replace_if(b,e,f,v2)` | 将 `[b:e)` 中满足 `f(*q)` 的元素 `*q` 替换为 `v2` |
| `p = copy(b,e,out)` | 将 `[b:e)` 复制到 `[out:p)` |
| `p = copy_if(b,e,out,f)` | 将 `[b:e)` 中满足 `f(*q)` 的元素 `*q` 复制到 `[out:p)` |
| `p = move(b,e,out)` | 将 `[b:e)` 移动到 `[out:p)` |
| `p = unique_copy(b,e,out)` | 将 `[b:e)` 复制到 `[out:p)`；不复制相邻的重复项 |
| `sort(b,e)` | 使用 `<` 作为排序准则对 `[b:e)` 的元素进行排序 |
| `sort(b,e,f)` | 使用 `f` 作为排序准则对 `[b:e)` 的元素进行排序 |
| `(p1,p2) = equal_range(b,e,v)` | `[p1:p2)` 是已排序序列 `[b:e)` 中值为 `v` 的子序列；基本上是 `v` 的二分查找 |
| `p = merge(b,e,b2,e2,out)` | 将两个已排序序列 `[b:e)` 和 `[b2:e2)` 合并到 `[out:p)` |
| `p = merge(b,e,b2,e2,out,f)` | 使用 `f` 作为比较，将两个已排序序列 `[b:e)` 和 `[b2:e2)` 合并到 `[out:p)` |

对于每个接受 `[b:e)` 范围的算法，`<ranges>` 提供了一个接受范围的版本。请记住（§9.3.2），要同时使用标准库算法的传统迭代器版本及其范围对应版本，你需要显式限定调用或使用 `using` 声明。

这些算法以及更多（例如 §17.3）可以应用于容器、字符串和内置数组的元素。

一些算法（如 `replace()` 和 `sort()`）会修改元素的值，但没有算法会添加或删除容器的元素。原因是序列并不标识持有该序列元素的容器。要添加或删除元素，你需要一些了解容器的东西（例如 `back_inserter`；§13.1）或直接引用容器本身（例如 `push_back()` 或 `erase()`；§12.2）。

Lambda 表达式作为操作参数传递非常常见。例如：

```cpp
vector<int> v = {0, 1, 2, 3, 4, 5};
for_each(v, [](int& x) { x = x * x; });                // v == {0,1,4,9,16,25}
for_each(v.begin(), v.begin() + 3, [](int& x) { x = sqrt(x); }); // v == {0,1,2,9,16,25}
```

标准库算法往往比一般的手工循环设计、规范和实现得更仔细。了解它们并优先使用它们，而不是用裸语言编写的代码。

## 13.6 并行算法

当需要对许多数据项执行相同的任务时，如果不同数据项上的计算是独立的，我们可以对每个数据项并行执行：

===== 第 16 页 =====

- **并行执行**：任务在多个线程上执行（通常运行在多个处理器核心上）
- **向量化执行**：任务在单个线程上使用向量化（也称为 SIMD，“单指令多数据”）执行

标准库对两者都提供支持，我们可以明确指定顺序执行；在 `<execution>` 的命名空间 `execution` 中，我们有：

- `seq`：顺序执行
- `par`：并行执行（如果可行）
- `unseq`：无序（向量化）执行（如果可行）
- `par_unseq`：并行和/或无序（向量化）执行（如果可行）

考虑 `std::sort()`：

```cpp
sort(v.begin(), v.end());               // 顺序
sort(seq, v.begin(), v.end());          // 顺序（与默认相同）
sort(par, v.begin(), v.end());          // 并行
sort(par_unseq, v.begin(), v.end());    // 并行和/或向量化
```

是否值得并行化和/或向量化取决于算法、序列中元素的数量、硬件以及其上运行的程序对该硬件的利用率。因此，执行策略指示符只是提示。编译器和/或运行时调度器将决定使用多少并发。这绝非易事，并且在不进行测量的情况下对效率做出判断的规则在这里非常重要。

遗憾的是，并行算法的 range 版本尚未纳入标准，但如果我们需要它们，很容易定义：

```cpp
void sort(auto pol, random_access_range auto& r)
{
    sort(pol, r.begin(), r.end());
}
```

大多数标准库算法，包括 §13.5 表中除 `equal_range` 之外的所有算法，都可以像 `sort()` 一样使用 `par` 和 `par_unseq` 请求并行化和向量化。为什么不包括 `equal_range()`？因为到目前为止还没有人提出一个有价值的并行算法。

许多并行算法主要用于数值数据；见 §17.3.1。

当请求并行执行时，务必避免数据竞争（§18.2）和死锁（§18.3）。

## 13.7 建议

[1] STL 算法操作一个或多个序列；§13.1。
[2] 输入序列是半开的，并由一对迭代器定义；§13.1。
[3] 你可以定义自己的迭代器来满足特殊需求；§13.1。
[4] 许多算法可以应用于 I/O 流；§13.3.1。
[5] 搜索时，算法通常返回输入序列的末尾以表示“未找到”；§13.2。
[6] 算法不会直接向其参数序列添加或删除元素；§13.2，§13.5。
[7] 编写循环时，考虑它是否可以表达为通用算法；§13.2。
[8] 使用类型别名来清理混乱的表示法；§13.2。
[9] 使用谓词和其他函数对象为标准算法提供更广泛的意义；§13.4，§13.5。
[10] 谓词不得修改其参数；§13.4。
[11] 了解你的标准库算法，并优先使用它们而不是手工制作的循环；§13.5。