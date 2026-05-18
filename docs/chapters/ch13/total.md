# 算法

> **如无必要，勿增实体。**
>
> —— William Occam

# 13.1 引言

像 `list` 或 `vector` 这样的数据结构，单凭自身并不太够用。要用它，就需要诸如添加、删除元素之类的基础访问操作（`list` 与 `vector` 提供了这类操作）。此外，我们也极少只是把对象塞进容器就了事：我们会排序、打印、抽取子集、删除元素、查找对象，等等。因此，标准库在提供最常见容器类型的同时，也提供了最常见的一批容器算法。举例来说，我们可以很简单也很高效地对 `Entry` 构成的 `vector` 排序，并把每个互不相同的元素的副本依次放进 `list`：

```cpp
void f(vector<Entry>& vec, list<Entry>& lst)
{
    sort(vec.begin(), vec.end());                     // 用 < 定义顺序
    unique_copy(vec.begin(), vec.end(), lst.begin()); // 不把相邻相等元素重复拷贝出去
}
```

要做到这一点，必须为 `Entry` 定义小于（`<`）与相等（`==`）。例如：

```cpp
bool operator<(const Entry& x, const Entry& y)   // 小于
{
    return x.name < y.name;                       // 按姓名排序 Entry
}
```

标准算法用（半开）元素序列来描述问题：序列由一对迭代器表示，分别指向首元素与尾后位置：

[图片描述：序列 `[begin:end)` 示意图]

在上面的例子里，`sort()` 对迭代器对 `vec.begin()`、`vec.end()` 所限制的序列排序——该序列恰好覆盖整个 `vector`。对于写出（输出），只需要指明写入的起点；若要写出多个元素，则会从起点开始向后覆盖。因而为避免错误，`lst` 至少要有与 `vec` 中互不相同元素个数一样多的元素。

遗憾的是，标准库并没有提供一个抽象来保证“写入容器”时的范围检查；不过我们可以自行定义：

```cpp
template<typename C>
class Checked_iter {
public:
    using value_type = typename C::value_type;
    using difference_type = int;

    Checked_iter() { throw Missing_container{}; }   // 概念 forward_iterator 的要求
    Checked_iter(C& cc) : pc{ &cc } {}
    Checked_iter(C& cc, typename C::iterator pp) : pc{ &cc }, p{ pp } {}

    Checked_iter& operator++() { check_end(); ++p; return *this; }
    Checked_iter operator++(int) { check_end(); auto t{ *this }; ++p; return t; }
    value_type& operator*() const { check_end(); return *p; }

    bool operator==(const Checked_iter& a) const { return p == a.p; }
    bool operator!=(const Checked_iter& a) const { return p != a.p; }
private:
    void check_end() const { if (p == pc->end()) throw Overflow{}; }
    C* pc {};                                      // 默认初始化为 nullptr
    typename C::iterator p = pc->begin();
};
```

这当然达不到标准库的品质，但足以说明思路：

```cpp
vector<int> v1 {1, 2, 3};     // 三个元素
vector<int> v2(2);            // 两个元素

copy(v1, v2.begin());       // 将会越界写入
copy(v1, Checked_iter{v2}); // 将会抛出异常
```

若在“读完再排序”的例子中想把互异元素放进一个新的 `list`，也可以写成：

```cpp
list<Entry> f(vector<Entry>& vec)
{
    list<Entry> res;
    sort(vec.begin(), vec.end());
    unique_copy(vec.begin(), vec.end(), back_inserter(res)); // 追加到 res
    return res;
}
```

`back_inserter(res)` 会构造一个指向 `res` 的迭代器：写入时在容器末尾添加元素，并扩展容器以容纳它们。这样就不必先分配一块固定空间再填入。于是标准容器配合 `back_inserter()`，可以避免那种容易出错、手工管理的 C 风格 `realloc()` 用法。标准库的 `list` 带有移动构造函数（§6.2.2），因此即便元素很多，按值返回 `res` 也仍然高效。

当我们觉得 `sort(vec.begin(), vec.end())` 这种“迭代器对”写法累赘时，可以使用算法的范围（range）版本写成 `sort(vec)`（§13.5）；两者等价。类似地，范围 `for` 大致等价于手写的使用迭代器的循环：

```cpp
for (auto& x : v)
    cout << x;                                      // 输出 v 中所有元素
for (auto p = v.begin(); p != v.end(); ++p)
    cout << *p;                                     // 输出 v 中所有元素
```

除了更简单、更不易出错之外，范围 `for` 往往也更高效。

# 13.2 使用迭代器

对容器而言，我们能拿到若干指向有用元素的迭代器；`begin()` 与 `end()` 就是最典型的例子。此外，许多算法也会返回迭代器。例如标准算法 `find` 在一个序列里查找某个值，并返回指向找到的元素的迭代器：

```cpp
list<Entry> f(vector<Entry>& vec)
{
    list<Entry> res;
    sort(vec.begin(), vec.end());
    unique_copy(vec.begin(), vec.end(), back_inserter(res)); // 追加到 res
    return res;
}

bool has_c(const string& s, char c) // s 是否包含字符 c？
{
    auto p = find(s.begin(), s.end(), c);
    if (p != s.end())
        return true;
    else
        return false;
}
```

与许多标准库查找算法一样，`find` 用返回 `end()` 来表示“未找到”。下面是等价但更简短的 `has_c()`：

```cpp
bool has_c(const string& s, char c) // s 是否包含字符 c？
{
    return find(s, c) != s.end();
}
```

更有趣的练习是在字符串中找到某个字符出现的所有位置。可以把每次出现的位置作为一组指针返回；由于 `vector` 提供了移动语义（§6.2.1），返回 `vector` 本身是高效的。若希望对找到的位置进行修改，则传入非常量的 `string`：

```cpp
vector<string::iterator> find_all(string& s, char c) // 找出 s 中所有为 c 的位置
{
    vector<string::iterator> res;
    for (auto p = s.begin(); p != s.end(); ++p)
        if (*p == c)
            res.push_back(p);
    return res;
}
```

我们用常规的循环遍历字符串：`++` 每次向前移动迭代器，`*` 读取字符。可以这样测试 `find_all()`：

```cpp
void test()
{
    string m {"Mary had a little lamb"};
    for (auto p : find_all(m, 'a'))
        if (*p != 'a')
            cerr << "a bug!\n";
}
```

对这次 `find_all()` 调用，也可以画图示意。

迭代器与标准算法对所有适用它们的容器能起到同样的效果。因此可以把 `find_all()` 推广：

```cpp
template<typename C, typename V>
vector<typename C::iterator> find_all(C& c, V v) // 找出容器 c 中所有值为 v 的位置
{
    vector<typename C::iterator> res;
    for (auto p = c.begin(); p != c.end(); ++p)
        if (*p == v)
            res.push_back(p);
    return res;
}
```

这里的 `typename` 用来告诉编译器：`C::iterator` 应当被视为类型，而不是某个类型的某个具体取值。

等价的做法是返回一组指向元素的普通指针：

```cpp
template<typename C, typename V>
auto find_all(C& c, V v)
{
    vector<range_value_t<C>*> res;
    for (auto& x : c)
        if (x == v)
            res.push_back(&x);
    return res;
}
```

这里顺带把遍历改成了范围 `for`，并用标准库的 `range_value_t`（§16.4.4）命名元素的类型。`range_value_t` 的一种极简等价写法可以是：

```cpp
template<typename T>
using range_value_type_t = T::value_type;
```

不管采用哪一种 `find_all()`，都可以这样写：

```cpp
void test()
{
    string m {"Mary had a little lamb"};

    for (auto p : find_all(m, 'a'))               // p 的类型为 string::iterator
        if (*p != 'a')
            cerr << "string bug!\n";

    list<int> ld {1, 2, 3, 1, -11, 2};
    for (auto p : find_all(ld, 1))               // p 的类型为 list<int>::iterator
        if (*p != 1)
            cerr << "list bug!\n";

    vector<string> vs {"red", "blue", "green", "green", "orange", "green"};
    for (auto p : find_all(vs, "red"))           // p 的类型为 vector<string>::iterator
        if (*p != "red")
            cerr << "vector bug!\n";

    for (auto p : find_all(vs, "green"))
        *p = "vert";
}
```

迭代器用来把算法与容器隔开：算法只通过迭代器操作数据，不知道元素存放在何种容器中；反过来容器也不知道有哪些算法作用于元素——它只是按需给出迭代器（例如 `begin()` 与 `end()`）。这种把存储与计算分离的模型非常通用且灵活。

# 13.3 迭代器类型

迭代器究竟是什么？任何一个迭代器都是某个类型的对象。但迭代器的类型很多：它需要携带在某个容器上做本职工作所需的足够信息。这些迭代器类型之间的差别可以像容器及其特定用途的差异那么大。

举例来说，`vector` 的迭代器完全可以是普通指针——指针恰好是一种很自然地指向 `vector` 元素的引用：

[图片描述：把 vector 的迭代器实现成指针]

或者，`vector` 迭代器也可以实现成「指向 `vector` 的指针 + 下标」：

[图片描述：把 vector 的迭代器实现成指针加索引]

使用此类迭代器就便于做范围检查。

相较之下，`list` 迭代器往往比一个指向节点的裸指针更复杂：链表中的一个节点通常并不知道表中下一个节点在哪儿，于是 `list` 迭代器可能表现为指向某个链接（link）的指针：

[图片描述：list 迭代器指向链表结点链接]

对所有迭代器共同的，是其语义以及操作的命名。例如对任一迭代器做 `++` 都得到指向下一元素的迭代器；`*` 则给出迭代器所指的元素。实际上，只要遵循几条诸如此类的规则的对象都可视作迭代器。迭代器是一种宽泛的观念（概念，§8.2）；不同类别的迭代器在标准库里也以概念的形式给出，例如 `forward_iterator`、`random_access_iterator`（§14.5）。

此外，使用者也很少真的关心某个迭代器的具体类型；每种容器都“知道”自己的迭代器类型，并用常规的别名把它们提供给外部：`iterator` 与 `const_iterator`。例如 `list<Entry>::iterator` 就是 `list<Entry>` 的一般迭代器类型；我们通常无须深究它的底层定义。

有些情形下迭代器并不是嵌套的成员类型，此时标准库提供 `iterator_t<X>`：凡是能为 `X` 定义迭代器的地方，`iterator_t<X>` 都好用。

## 13.3.1 输入迭代器与输出迭代器

迭代器是把容器中元素序列当作序列来处理时一个非常通用而有用的抽象；然而序列不只存在于容器里。输入流会产生一串值，而我们会向输出流写出一串值。因而迭代器的观念同样可以应用于输入与输出。

若要构造 `ostream_iterator`，需要指明要写往哪一个流，以及写入对象值的类型。例如：

```cpp
ostream_iterator<string> oo {cout}; // 把字符串写到 cout
```

对 `*oo` 赋值的效果就是把相应值写入 `cout`。例如：

```cpp
int main()
{
    *oo = "Hello, ";   // 相当于 cout << "Hello, "
    ++oo;
    *oo = "world!\n"; // 相当于 cout << "world!\n"
}
```

这又是写出那句经典问候语的另一种写法。这里的 `++oo` 是为了模仿通过指针向数组写入，从而让算法也能施加于流。例如：

```cpp
vector<string> v {"Hello", ", ", "World!\n"};
copy(v, oo);
```

类似地，`istream_iterator` 让我们把输入流当成只读容器来面对；同样需要指明输入源以及读取的值类型：

```cpp
istream_iterator<string> ii {cin};
```

输入迭代器总是成对出现来表示序列，因此还需要第二个迭代器标记输入结束；默认构造的 `istream_iterator` 就扮演这个角色：

```cpp
istream_iterator<string> eos {};
```

实践中，`istream_iterator` / `ostream_iterator` 通常并非手写直接用，而是作为参数传给算法。例如可以先读取文件，对读到的单词排序、去掉相邻重复，再写入另一个文件：

```cpp
int main()
{
    string from, to;
    cin >> from >> to;                     // 源文件名与目标文件名

    ifstream is {from};                    // 来自文件 `from` 的输入流
    istream_iterator<string> ii {is};      // 针对流的输入迭代器
    istream_iterator<string> eos {};       // 输入哨兵

    ofstream os {to};                      // 写入文件 `to` 的输出流
    ostream_iterator<string> oo {os, "\n"}; // 输出迭代器，带分隔串

    vector<string> b {ii, eos};            // 用输入初始化缓冲区 b
    sort(b);                               // 排序缓冲区
    unique_copy(b, oo);                    // 写到输出并去掉相邻重复

    return !is.eof() || !os;               // 返回错误状态（§1.2.1，§11.4）
}
```

这里用的是 `sort()` 与 `unique_copy()` 的范围版本；也可以写成 `sort(b.begin(), b.end())`——在老代码里更常见。

请记住（§9.3.2）：若要在同一作用域里同时使用传统迭代器版本与其范围版本，要么显式限定命名空间（例如 `ranges::copy`），要么通过 `using` 声明消除歧义：

```cpp
copy(v, oo);           // 可能产生歧义
ranges::copy(v, oo);   // OK

using std::ranges::copy;
copy(v, oo);           // OK（在本作用域中指范围版本）
```

`ifstream` 是可附着到文件的输入流（§11.7.2），`ofstream` 是可附着到文件的输出流。`ostream_iterator` 的第二个参数用来分隔输出的各个值。

坦诚地说，这个示例还可以写得更短：我们先读入 `vector`、排序，再写出并去掉重复。更优雅的思路是根本不必保存重复——把它们放进 `set` 即可：`set` 既不含重复元素，又自动保持有序（§12.5）。于是可以把原先两行基于 `vector` 的逻辑换成一行基于 `set`，并把 `unique_copy()` 换成更直接的 `copy()`：

```cpp
set<string> b {ii, eos}; // 从输入收集字符串
copy(b, oo);             // 写出缓冲区
```

`ii`、`eos`、`oo` 若只用一次，还可以进一步压缩：

```cpp
int main()
{
    string from, to;
    cin >> from >> to;

    ifstream is {from};
    ofstream os {to};

    set<string> b {istream_iterator<string>{is}, istream_iterator<string>{}};
    copy(b, ostream_iterator<string>{os, "\n"});

    return !is.eof() || !os;
}
```

是否要把程序压到这么短，归根结底关乎品味与经验。

# 13.4 谓词

```cpp
int main()
{
    string from, to;
    cin >> from >> to;

    ifstream is {from};
    ofstream os {to};

    set<string> b {istream_iterator<string>{is}, istream_iterator<string>{}};
    copy(b, ostream_iterator<string>{os, "\n"});

    return !is.eof() || !os;
}
```

此前的例子中，算法对每个元素要做什么往往是“内置固定”的；但我们常常希望把这项动作参数化。譬如 `find`（§13.2，§13.5）提供了查找特定值的便捷途径；更一般的变体则是查找满足某个要求的元素——这类要求称为**谓词**（predicate）。

例如我们可能想在 `map` 里寻找第一个值大于 `42` 的项。`map` 的元素序列可以视作一串 `(key, value)` 对，于是能在 `map<string,int>` 上搜索第一个满足“`int` 部分大于 `42`”的 `pair<const string,int>`：

```cpp
void f(map<string, int>& m)
{
    auto p = find_if(m, Greater_than{42});
    // ...
}
```

这里的 `Greater_than` 是函数对象（§7.3.2），内部保存要与 `map` 元素（类型为 `pair<string,int>`）比较的阈值：

```cpp
struct Greater_than {
    int val;
    Greater_than(int v) : val{v} {}
    bool operator()(const pair<string, int>& r) const { return r.second > val; }
};
```

等价地，也可以用 lambda（§7.3.3）：

```cpp
auto p = find_if(m, [](const auto& r) { return r.second > 42; });
```

谓词不应修改它所作用的元素。

# 13.5 算法概览

算法的一般定义是：“有限的若干规则，给出求解一类具体问题的一组运算序列”，并具有五个关键特性——有限性、确定性、输入、输出与有效性等。[Knuth,1968, §1.1] 就强调了这一点。

就 C++ 标准库的语境而言，算法是指在元素序列上工作的函数模板。

标准库里有几十个算法；它们定义在命名空间 `std`，并通过 `<algorithm>`、`<numeric>` 提供。标准算法都把序列当作输入：半开区间 `[b:e)` 表示从 `b` 到 `e`（不包含 `e`）。下面是少量代表性的算法：

**选定的标准算法（`<algorithm>`）**

| 算法 | 说明 |
|------|------|
| `f = for_each(b, e, f)` | 对 `[b:e)` 中的每个元素 `x` 调用 `f(x)` |
| `p = find(b, e, x)` | `p` 是 `[b:e)` 中首个满足 `*p == x` 的位置 |
| `p = find_if(b, e, f)` | `p` 是 `[b:e)` 中首个满足 `f(*p)` 的位置 |
| `n = count(b, e, x)` | `n` 统计 `[b:e)` 中满足 `*q == x` 的元素个数 |
| `n = count_if(b, e, f)` | `n` 统计 `[b:e)` 中满足 `f(*q)` 的元素个数 |
| `replace(b, e, v, v2)` | 把 `[b:e)` 中所有满足 `*q == v` 的 `*q` 替换为 `v2` |
| `replace_if(b, e, f, v2)` | 把 `[b:e)` 中所有满足 `f(*q)` 的 `*q` 替换为 `v2` |
| `p = copy(b, e, out)` | 复制 `[b:e)` 到 `[out:p)` |
| `p = copy_if(b, e, out, f)` | 复制 `[b:e)` 中满足 `f(*q)` 的元素到 `[out:p)` |
| `p = move(b, e, out)` | 移动 `[b:e)` 到 `[out:p)` |
| `p = unique_copy(b, e, out)` | 复制 `[b:e)` 到 `[out:p)`，跳过相邻重复 |
| `sort(b, e)` | 按 `<` 排序 `[b:e)` |
| `sort(b, e, f)` | 按比较函数 `f` 排序 `[b:e)` |
| `(p1, p2) = equal_range(b, e, v)` | `[p1:p2)` 是在有序序列 `[b:e)` 中值等于 `v` 的子区间（本质是二分查找 `v`） |
| `p = merge(b, e, b2, e2, out)` | 合并两个有序序列 `[b:e)` 与 `[b2:e2)` 到 `[out:p)` |
| `p = merge(b, e, b2, e2, out, f)` | 用比较函数 `f` 合并两个有序序列 |

凡是形如 `[b:e)` 的传统迭代器接口，`<ranges>` 几乎都提供一个直接接收范围的版本。请记住（§9.3.2）：若想同时使用同名算法的迭代器版本与范围版本，需要显式限定调用或通过 `using` 引入其中之一。

这些（再加上 §17.3 等章节提到的算法）都可施加于容器、字符串乃至内置数组中的元素。

部分算法（例如 `replace()`、`sort()`）会改写元素的值，但不会凭空增减容器元素个数——原因很简单：序列并不指明背后的容器是谁。若要增删元素，需要知晓容器本身的某种抽象（例如 `back_inserter`；§13.1），或直接调用容器提供的接口（如 `push_back()`、`erase()`；§12.2）。

Lambda 很适合充当传给算法的运算参数：

```cpp
vector<int> v = {0, 1, 2, 3, 4, 5};
for_each(v, [](int& x) { x = x * x; });                              // v == {0,1,4,9,16,25}
for_each(v.begin(), v.begin() + 3, [](int& x) { x = sqrt(x); });    // v == {0,1,2,9,16,25}
```

标准库算法通常比一般手写循环在设计规范度与实现质量上更可靠；熟悉它们并在能用之处优先使用。

# 13.6 并行算法

若要对大量数据项重复同一任务，只要彼此之间的计算互不依赖，就可以并行地在多条数据上执行：

- **并行执行**：任务分布在多个线程（通常对应多颗处理器核心）上完成。
- **向量化执行**：在单线程上用 SIMD（“单指令多数据”）做向量化的并行。

标准库对这两种方式都有支持；也可以在 `<execution>` 里明确要求顺序执行。命名空间 `execution`（通过 `<execution>`）提供下列执行策略标签：

- `seq`：顺序执行；
- `par`：并行执行（若可行）；
- `unseq`：非顺序的向量化执行（若可行）；
- `par_unseq`：并行与/或向量化的执行（若可行）。

以 `sort()` 为例：

```cpp
sort(v.begin(), v.end());              // 顺序
sort(seq, v.begin(), v.end());        // 顺序（与默认相同）
sort(par, v.begin(), v.end());        // 并行
sort(par_unseq, v.begin(), v.end());  // 并行与/或向量化
```

并行化或向量化是否值得，取决于算法本身、序列长度、硬件以及当时的负载等因素；因此这些策略只是提示（hint）。编译器与/或运行时调度器会决定并发程度。这个问题并不平凡——此处再次强调：**不做测量就不要断言效率**。

遗憾的是，并行算法的范围版本尚未进入标准；但若确实需要，很容易自行封装：

```cpp
void sort(auto pol, random_access_range auto& r)
{
    sort(pol, r.begin(), r.end());
}
```

绝大多数标准库算法（§13.5 表格中的算法除 `equal_range` 之外）都可以像上面这样对 `sort()` 那样指定 `par` / `par_unseq`。为何暂时没有并行版 `equal_range()`？因为迄今仍未找到足够有意义的并行算法。

许多并行算法主要服务于数值计算场景（§17.3.1）。

请求并行执行时，务必避免数据竞争（§18.2）与死锁（§18.3）。

# 13.7 建议

[1] STL 算法对一个或多个序列进行操作；§13.1。

[2] 输入序列是半开区间，由一对迭代器界定；§13.1。

[3] 你可以为满足特定需求自定义迭代器；§13.1。

[4] 许多算法也能作用于 I/O 流；§13.3.1。

[5] 查找类算法通常用返回输入序列尾迭代器表示“未找到”；§13.2。

[6] 算法不会直接对其参数序列增删元素；§13.2，§13.5。

[7] 编写循环时，想一想是否能改写为标准算法；§13.2。

[8] 使用类型别名整理累赘的类型记号；§13.2。

[9] 借助谓词与其他函数对象，可以让标准算法表达更丰富的含义；§13.4，§13.5。

[10] 谓词不得修改其参数；§13.4。

[11] 熟悉标准库算法并优先于手写循环使用它们；§13.5。
