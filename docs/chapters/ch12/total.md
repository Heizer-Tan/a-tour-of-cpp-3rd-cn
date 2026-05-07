===== 第 1 页 =====

12

# 容器

它是崭新的。它是独特的。它是简单的。

它必须成功！ – H. Nelson

- 引言
- vector
  - 元素；范围检查
- list
- forward_list
- map
- unordered_map
- 分配器
- 容器概览

===== 第 2 页 =====

### 12.1 引言

大多数计算都涉及创建值的集合，然后操作这些集合。将字符读入字符串并打印出来就是一个简单的例子。主要目的是容纳对象的类通常称为**容器**。为给定任务提供合适的容器，并用有用的基本操作来支持它们，是构建任何程序的重要步骤。

为了说明标准库容器，考虑一个用于保存姓名和电话号码的简单程序。对于这种程序，不同背景的人会认为不同的方法“简单而显而易见”。§11.5 中的 `Entry` 类可用于保存一个简单的电话簿条目。在这里，我们刻意忽略了许多现实世界的复杂性，例如许多电话号码不能用 32 位 `int` 简单表示。

## 12.2 vector

最有用的标准库容器是 `vector`。`vector` 是给定类型元素的一个序列。元素在内存中连续存储。`vector` 的一个典型实现（§5.2.2，§6.2）将包含一个句柄，其中持有指向第一个元素、最后一个元素之后位置以及最后分配空间之后位置的指针（§13.1）（或表示为指针加偏移量的等效信息）：

[图片描述：vector 结构示意图]

===== 第 3 页 =====

此外，它还持有一个分配器（此处为 `alloc`），`vector` 可以从该分配器获取其元素所需的内存。默认分配器使用 `new` 和 `delete` 来获取和释放内存（§12.7）。使用稍微高级的实现技术，我们可以避免在 `vector` 对象中为简单的分配器存储任何数据。

我们可以用一组其元素类型的值来初始化 `vector`：

```cpp
vector<Entry> phone_book = {
    {"David Hume", 123456},
    {"Karl Popper", 234567},
    {"Bertrand Arthur William Russell", 345678}
};
```

可以通过下标来访问元素。因此，假设我们已经为 `Entry` 定义了 `<<`，我们可以写：

```cpp
void print_book(const vector<Entry>& book)
{
    for (int i = 0; i != book.size(); ++i)
        cout << book[i] << '\n';
}
```

和往常一样，索引从 0 开始，因此 `book[0]` 保存了 David Hume 的条目。`vector` 的成员函数 `size()` 返回元素的数量。

`vector` 的元素构成一个范围，因此我们可以使用范围 `for` 循环（§1.7）：

```cpp
void print_book(const vector<Entry>& book)
{
    for (const auto& x : book)   // 关于 "auto" 见 §1.4
        cout << x << '\n';
}
```

当我们定义 `vector` 时，我们给它一个初始大小（初始元素个数）：

```cpp
vector<int> v1 = {1, 2, 3, 4};        // 大小是 4
vector<string> v2;                    // 大小是 0
```

===== 第 4 页 =====

```cpp
vector<Shape*> v3(23);                // 大小是 23；初始元素值：nullptr
vector<double> v4(32, 9.9);           // 大小是 32；初始元素值：9.9
```

显式的大小用普通括号括起来，例如 `(23)`，默认情况下，元素被初始化为元素类型的默认值（例如指针为 `nullptr`，数字为 `0`）。如果你不想要默认值，可以指定一个值作为第二个参数（例如 `v4` 的 32 个元素指定为 `9.9`）。

初始大小可以改变。`vector` 最有用的操作之一是 `push_back()`，它在 `vector` 的末尾添加一个新元素，将其大小增加一。例如，假设我们已经为 `Entry` 定义了 `>>`，我们可以写：

```cpp
void input()
{
    for (Entry e; cin >> e; )
        phone_book.push_back(e);
}
```

这从标准输入读取 `Entry` 到 `phone_book` 中，直到到达输入末尾（例如文件结束）或输入操作遇到格式错误。

标准库 `vector` 的实现使得通过重复 `push_back()` 来增长 `vector` 是高效的。为了说明这一点，考虑使用上面图中指示的表示法对第 5 章和第 7 章中的简单 `Vector` 进行细化：

```cpp
template<typename T>
class Vector {
    allocator<T> alloc;   // 标准库为 T 分配空间的分配器
    T* elem;              // 指向第一个元素的指针
    T* space;             // 指向第一个未使用（且未初始化）的槽位
    T* last;              // 指向最后一个槽位
public:
    // ...
    int size() const { return space - elem; }      // 元素个数
    int capacity() const { return last - elem; }   // 可用于元素的槽位数
    // ...
    void reserve(int newsz);                       // 将 capacity() 增加到 newsz
    // ...
    void push_back(const T& t);   // 将 t 拷贝到 Vector
    void push_back(T&& t);        // 将 t 移动到 Vector
};
```

===== 第 5 页 =====

标准库 `vector` 有成员 `capacity()`、`reserve()` 和 `push_back()`。`reserve()` 被 `vector` 的用户和其他 `vector` 成员用来为更多元素腾出空间。它可能需要分配新的内存，并且当它这样做时，它会将元素移动到新的分配中。当 `reserve()` 将元素移动到一个新的更大的分配时，指向这些元素的任何指针现在都将指向错误的位置；它们已经失效，不应再使用。

有了 `capacity()` 和 `reserve()`，实现 `push_back()` 就很简单了：

```cpp
template<typename T>
void Vector<T>::push_back(const T& t)
{
    if (capacity() <= size())               // 确保有空间容纳 t
        reserve(size() == 0 ? 8 : 2 * size());  // 容量加倍
    construct_at(space, t);                // 将 *space 初始化为 t（“将 t 置于 space”）
    ++space;
}
```

现在，元素的分配和重定位只偶尔发生。我曾经使用 `reserve()` 来尝试提高性能，但结果证明是白费力气：`vector` 使用的启发式规则平均而言比我猜的要好，所以现在我仅在希望使用指向元素的指针时，为了避免元素的重分配才显式使用 `reserve()`。

`vector` 可以在赋值和初始化中被拷贝。例如：

```cpp
vector<Entry> book2 = phone_book;
```

拷贝和移动 `vector` 是通过构造函数和赋值运算符实现的，如 §6.2 所述。赋值一个 `vector` 涉及拷贝其元素。因此，在 `book2` 初始化之后，`book2` 和 `phone_book` 各自持有电话簿中每个 `Entry` 的独立副本。当一个 `vector` 持有许多元素时，这种看起来无害的赋值和初始化可能代价高昂。在拷贝不可取的情况下，应使用引用或指针（§1.7）或移动操作（§6.2.2）。

标准库 `vector` 非常灵活和高效。将它作为你的默认容器；也就是说，除非你有充分的理由使用其他容器，否则就使用它。如果你因为模糊的“效率”担忧而避免使用 `vector`，请进行测量。在容器使用的性能方面，我们的直觉最容易出错。

#### 12.2.1 元素

与所有标准库容器一样，`vector` 是某种类型 `T` 的元素的容器，即 `vector<T>`。几乎任何类型都可以作为元素类型：内置数值类型（如 `char`、`int` 和 `double`）、用户定义类型（如 `string`、`Entry`、`list<int>` 和 `Matrix<double,2>`）以及指针（如 `const char*`、`Shape*` 和 `double*`）。当你插入一个新元素时，它的值会被拷贝到容器中。例如，当你将一个值为 `7` 的整数放入容器时，结果元素的值就是 `7`。该元素不是对某个包含 `7` 的对象的引用或指针。这使得容器很好、很紧凑，并且访问速度快。对于那些关心内存大小和运行时性能的人来说，这一点至关重要。

如果你有一个依赖于虚函数来实现多态行为的类层次结构（§5.5），不要直接将对象存储在容器中。而是存储指针（或智能指针；§15.2.1）。例如：

```cpp
vector<Shape> vs;                     // 不，不要这样做 - 没有空间存放 Circle 或 Smiley
vector<Shape*> vps;                   // 更好，但见 §5.5.3（不要泄漏）
vector<unique_ptr<Shape>> vups;       // OK
```

#### 12.2.2 范围检查

标准库 `vector` 不保证范围检查。例如：

```cpp
void silly(vector<Entry>& book)
{
    int i = book[book.size()].number;   // book.size() 超出范围
    // ...
}
```

===== 第 7 页 =====

那个初始化很可能将某个随机值放入 `i` 而不是给出错误。这是不可取的，并且越界错误是一个常见问题。因此，我经常使用 `vector` 的一个简单的范围检查适配版本：

```cpp
template<typename T>
struct Vec : std::vector<T> {
    using vector<T>::vector;   // 使用来自 vector 的构造函数

    T& operator[](int i) { return vector<T>::at(i); }               // 范围检查
    const T& operator[](int i) const { return vector<T>::at(i); }   // 范围检查（const）

    auto begin() { return Checked_iter<vector<T>>{*this}; }         // 见 §13.1
    auto end()   { return Checked_iter<vector<T>>{*this, vector<T>::end()}; }
};
```

`Vec` 从 `vector` 继承了所有内容，除了它重新定义了下标操作以进行范围检查。`at()` 操作是一个 `vector` 下标操作，如果其参数超出 `vector` 的范围，它会抛出 `out_of_range` 类型的异常（§4.2）。

对于 `Vec`，越界访问将抛出一个用户可以捕获的异常。例如：

```cpp
void checked(Vec<Entry>& book)
{
    try {
        book[book.size()] = {"Joe", 999999};   // 将抛出一个异常
        // ...
    }
    catch (out_of_range&) {
        cerr << "range error\n";
    }
}
```

异常将被抛出，然后被捕获（§4.2）。如果用户没有捕获异常，程序将以定义良好的方式终止，而不是以未定义的方式继续或失败。最小化未捕获异常带来的意外的一种方法是使用带有 `try` 块作为其主体的 `main()`。例如：

```cpp
int main()
try {
    // 你的代码
}
catch (out_of_range&) {
    cerr << "range error\n";
}
catch (...) {
    cerr << "unknown exception thrown\n";
}
```

这提供了默认的异常处理程序，因此如果我们未能捕获某个异常，会在标准错误诊断输出流 `cerr` 上打印错误信息（§11.2）。

为什么标准不保证范围检查？许多性能关键的应用程序使用 `vector`，检查所有下标意味着大约 10% 的成本。显然，该成本可能因硬件、优化器和应用程序对下标的使用而有很大差异。然而，经验表明，这种开销可能导致人们更喜欢更不安全的内置数组。甚至仅仅是对这种开销的恐惧也可能导致弃用。至少 `vector` 在调试时很容易进行范围检查，并且我们可以在未检查的默认版本之上构建检查版本。

范围 `for` 通过隐式访问范围内的所有元素，以零成本避免了范围错误。只要它们的参数有效，标准库算法也会这样做，以确保没有范围错误。

如果你直接在代码中使用 `vector::at()`，你就不需要我的 `Vec` 变通方法。此外，一些标准库有范围检查的 `vector` 实现，提供比 `Vec` 更完整的检查。

===== 第 9 页 =====

## 12.3 list

标准库提供了一个称为 `list` 的双向链表：

[图片描述：list 结构示意图]

当我们希望在不移动其他元素的情况下插入和删除元素时，我们使用 `list`。电话簿条目的插入和删除可能很常见，因此 `list` 可能适合表示一个简单的电话簿。例如：

```cpp
list<Entry> phone_book = {
    {"David Hume", 123456},
    {"Karl Popper", 234567},
    {"Bertrand Arthur William Russell", 345678}
};
```

当我们使用链表时，我们通常不像对 `vector` 那样使用下标来访问元素。相反，我们可能会搜索链表以查找具有给定值的元素。为此，我们利用 `list` 是一个序列的事实，如第 13 章所述：

```cpp
int get_number(const string& s)
{
    for (const auto& x : phone_book)
        if (x.name == s)
            return x.number;
    return 0;   // 使用 0 表示“未找到号码”
}
```

对 `s` 的搜索从链表的开头开始，直到找到 `s` 或到达 `phone_book` 的末尾。

===== 第 10 页 =====

有时，我们需要标识 `list` 中的一个元素。例如，我们可能想要删除一个元素或在其之前插入一个新元素。为此，我们使用**迭代器**：`list` 迭代器标识 `list` 的一个元素，并可用于遍历 `list`（因此得名）。每个标准库容器都提供函数 `begin()` 和 `end()`，它们分别返回指向第一个元素和最后一个元素之后位置的迭代器（§13.1）。显式使用迭代器，我们可以（不那么优雅地）这样写 `get_number()` 函数：

```cpp
int get_number(const string& s)
{
    for (auto p = phone_book.begin(); p != phone_book.end(); ++p)
        if (p->name == s)
            return p->number;
    return 0;
}
```

事实上，这大致就是编译器实现更简洁、更不易出错的范围 `for` 循环的方式。给定一个迭代器 `p`，`*p` 是它所指的元素，`++p` 将 `p` 前进到下一个元素，并且当 `p` 指向一个带有成员 `m` 的类时，`p->m` 等价于 `(*p).m`。

向 `list` 添加元素和从 `list` 中删除元素很容易：

```cpp
void f(const Entry& ee, list<Entry>::iterator p, list<Entry>::iterator q)
{
    phone_book.insert(p, ee);   // 在 p 所指的元素之前添加 ee
    phone_book.erase(q);        // 删除 q 所指的元素
}
```

对于 `list`，`insert(p, elem)` 在 `p` 指向的元素之前插入一个值为 `elem` 的副本。这里，`p` 可以是一个指向 `list` 末尾之后位置的迭代器。相反，`erase(p)` 删除 `p` 指向的元素并销毁它。

这些 `list` 的例子可以完全类似地用 `vector` 编写，并且（除非你了解计算机体系结构，否则会令人惊讶地）通常使用 `vector` 比使用 `list` 性能更好。当我们只需要一个元素序列时，我们可以在 `vector` 和 `list` 之间选择。除非你有理由不这样做，否则请使用 `vector`。`vector` 在遍历（例如 `find()` 和 `count()`）以及排序和搜索（例如 `sort()` 和 `equal_range()`；§13.5，§15.3.3）方面表现更好。

===== 第 11 页 =====

## 12.4 forward_list

标准库还提供了一个称为 `forward_list` 的单向链表：

[图片描述：forward_list 结构示意图]

`forward_list` 与（双向）`list` 的不同之处在于它只允许前向迭代。这样做的目的是节省空间。不需要在每个链接中保留前驱指针，并且一个空的 `forward_list` 的大小只有一个指针。`forward_list` 甚至不保存其元素个数。如果你需要元素个数，请自行计数。如果你负担不起计数的代价，那么你可能不应该使用 `forward_list`。

## 12.5 map

在（姓名，号码）对列表中查找姓名的代码编写起来相当繁琐。此外，除了最短的列表外，线性搜索效率低下。标准库提供了一个平衡二叉搜索树（通常是红黑树），称为 `map`：

[图片描述：map 结构示意图]

===== 第 12 页 =====

在其他上下文中，`map` 被称为关联数组或字典。

标准库 `map` 是一个键值对的容器，针对查找和插入进行了优化。我们可以使用与 `vector` 和 `list` 相同的初始化器（§12.2，§12.3）：

```cpp
map<string, int> phone_book {
    {"David Hume", 123456},
    {"Karl Popper", 234567},
    {"Bertrand Arthur William Russell", 345678}
};
```

当用其第一个类型的值（称为**键**）进行索引时，`map` 返回对应的第二个类型的值（称为**值**或**映射类型**）。例如：

```cpp
int get_number(const string& s)
{
    return phone_book[s];
}
```

换句话说，对 `map` 的下标操作本质上就是我们之前称为 `get_number()` 的查找。如果找不到某个键，它会被以默认值作为其值插入到 `map` 中。整数类型的默认值是 `0`，而这恰好是表示无效电话号码的一个合理值。

===== 第 13 页 =====

如果我们想避免将无效号码输入电话簿，可以使用 `find()` 和 `insert()`（§12.8）而不是 `[]`。

## 12.6 unordered_map

`map` 查找的成本是 O(log(n))，其中 `n` 是 `map` 中元素的数量。这相当不错。例如，对于一个包含 1,000,000 个元素的 `map`，我们只需大约 20 次比较和间接寻址就能找到一个元素。然而，在许多情况下，我们可以使用哈希查找而不是使用排序函数（如 `<`）进行比较，从而做得更好。标准库的哈希容器被称为“无序的”，因为它们不需要排序函数：

[图片描述：unordered_map 结构示意图]

例如，我们可以使用 `<unordered_map>` 中的 `unordered_map` 来实现我们的电话簿：

```cpp
unordered_map<string, int> phone_book {
    {"David Hume", 123456},
    {"Karl Popper", 234567},
    {"Bertrand Arthur William Russell", 345678}
};
```

与 `map` 类似，我们可以对 `unordered_map` 使用下标操作：

```cpp
int get_number(const string& s)
{
    return phone_book[s];
}
```

===== 第 14 页 =====

标准库为字符串以及其他内置类型和标准库类型提供了默认的哈希函数。如有必要，我们可以提供自己的哈希函数。可能最常见的需要自定义哈希函数的情况是当我们需要一个包含我们自己类型的无序容器时。哈希函数通常实现为函数对象（§7.3.2）。例如：

```cpp
struct Record {
    string name;
    int product_code;
    // ...
};

struct Rhash {   // 为 Record 定义的哈希函数
    size_t operator()(const Record& r) const
    {
        return hash<string>{}(r.name) ^ hash<int>{}(r.product_code);
    }
};

unordered_set<Record, Rhash> my_set;   // 使用 Rhash 进行查找的 Record 集合
```

设计好的哈希函数是一门艺术，通常需要了解将应用于哪些数据。通过使用异或（`^`）组合现有哈希函数来创建新的哈希函数，既简单又通常非常有效。然而，要小心确保参与哈希的每个值确实有助于区分不同的值。例如，除非你可以为同一个产品代码有多个名称（或为同一个名称有多个产品代码），否则组合两个哈希值不会带来任何好处。

我们可以通过将哈希操作定义为标准库 `hash` 的特化来避免显式传递它：

```cpp
namespace std {   // 为 Record 创建一个哈希函数
    template<> struct hash<Record> {
        using argument_type = Record;
        using result_type = size_t;

        result_type operator()(const Record& r) const
        {
            return hash<string>{}(r.name) ^ hash<int>{}(r.product_code);
        }
    };
}
```

===== 第 15 页 =====

注意 `map` 和 `unordered_map` 之间的区别：

- `map` 需要一个排序函数（默认是 `<`），并产生一个有序序列。
- `unordered_map` 需要一个相等函数（默认是 `==`）；它不维护其元素之间的顺序。

给定一个好的哈希函数，`unordered_map` 对于大型容器比 `map` 快得多。然而，如果哈希函数不佳，`unordered_map` 的最坏情况行为远比 `map` 糟糕。

## 12.7 分配器

默认情况下，标准库容器使用 `new` 分配空间。运算符 `new` 和 `delete` 提供了一个通用的自由存储区（也称为动态内存或堆），可以容纳任意大小的对象，并具有用户控制的生命周期。这意味着时间和空间开销，在许多特殊情况下可以消除。因此，标准库容器提供了在需要时安装具有特定语义的分配器的机会。这已被用于解决各种与性能相关的问题（例如池分配器）、安全性（在删除时清理内存的分配器）、每线程分配以及非统一内存架构（在特定内存中分配并匹配指针类型）。这里不是讨论这些重要但非常专业且通常是高级技术的地方。然而，我将给出一个由真实世界问题驱动的例子，其中池分配器是解决方案。

一个重要的长期运行系统使用了一个事件队列（见 §18.4），将向量作为事件，以 `shared_ptr` 传递。这样，事件的最后一个使用者会隐式删除它：

```cpp
struct Event {
    vector<int> data = vector<int>(512);
};

list<shared_ptr<Event>> q;

void producer()
{
    for (int n = 0; n != LOTS; ++n) {
        lock_guard lk {m};               // m 是一个互斥量；见 §18.3
        q.push_back(make_shared<Event>());
        cv.notify_one();                 // cv 是一个条件变量；见 §18.4
    }
}
```

从逻辑角度来看，这工作得很好。逻辑上简单，因此代码健壮且可维护。不幸的是，这导致了大量的内存碎片。在 16 个生产者和 4 个消费者之间传递了 100,000 个事件之后，消耗了超过 6GB 的内存。

解决碎片问题的传统方法是重写代码以使用池分配器。池分配器是一种管理单个固定大小对象的分配器，它一次为许多对象分配空间，而不是使用单独的分配。幸运的是，C++ 直接支持这一点。池分配器定义在 `std` 的 `pmr`（“多态内存资源”）子命名空间中：

```cpp
pmr::synchronized_pool_resource pool;   // 创建一个池

struct Event {
    vector<int> data = vector<int>{512, &pool};   // 让 Events 使用池
};

list<shared_ptr<Event>> q {&pool};                // 让 q 使用池

void producer()
{
    for (int n = 0; n != LOTS; ++n) {
        scoped_lock lk {m};                       // m 是一个互斥量（§18.3）
        q.push_back(allocate_shared<Event, pmr::polymorphic_allocator<Event>>(pmr::polymorphic_allocator<Event>{&pool}));
        cv.notify_one();
    }
}
```

===== 第 17 页 =====

现在，在 16 个生产者和 4 个消费者之间传递了 100,000 个事件之后，消耗的内存不到 3MB。这大约提升了 2000 倍！当然，实际使用的内存量（相对于浪费在碎片上的内存）没有变化。消除碎片后，内存使用随时间保持稳定，因此系统可以运行数月。

这类技术从 C++ 的早期就被应用并取得了良好的效果，但通常需要重写代码以使用专门的容器。现在，标准容器可以选择性地接受分配器参数。默认情况下，容器使用 `new` 和 `delete`。其他多态内存资源包括：

- `unsynchronized_polymorphic_resource`：类似于 `polymorphic_resource`，但只能由一个线程使用。
- `monotonic_polymorphic_resource`：一个快速的分配器，仅在销毁时释放其内存，并且只能由一个线程使用。

多态资源必须派生自 `memory_resource` 并定义成员 `allocate()`、`deallocate()` 和 `is_equal()`。其理念是让用户构建自己的资源来调整代码。

## 12.8 容器概览

标准库提供了一些最通用和最有用的容器类型，允许程序员选择最能满足应用需求的容器：

**标准容器摘要**

| 容器 | 描述 |
|------|------|
| `vector<T>` | 可变大小的向量（§12.2） |
| `list<T>` | 双向链表（§12.3） |
| `forward_list<T>` | 单向链表 |
| `deque<T>` | 双端队列 |
| `map<K,V>` | 关联数组（§12.5） |
| `multimap<K,V>` | 一个键可以出现多次的 map |
| `unordered_map<K,V>` | 使用哈希查找的 map（§12.6） |
| `unordered_multimap<K,V>` | 使用哈希查找的 multimap |
| `set<T>` | 集合（只有键没有值的 map） |
| `multiset<T>` | 一个值可以出现多次的 set |
| `unordered_set<T>` | 使用哈希查找的 set |
| `unordered_multiset<T>` | 使用哈希查找的 multiset |

无序容器针对用键（通常是字符串）进行查找进行了优化；换句话说，它们是哈希表。

这些容器定义在命名空间 `std` 中，并在头文件 `<vector>`、`<list>`、`<map>` 等中提供（§9.3.4）。此外，标准库还提供了容器适配器 `queue<T>`、`stack<T>` 和 `priority_queue<T>`。如果你需要它们，请查阅相关文档。标准库还提供了更专门的类似容器的类型，例如 `array<T,N>`（§15.3.1）和 `bitset<N>`（§15.3.2）。

标准容器及其基本操作在表示法上设计得相似。此外，对于不同的容器，操作的含义是等价的。基本操作适用于每一种有意义的、可以有效实现的容器：

**标准容器操作（部分）**

| 操作 | 描述 |
|------|------|
| `value_type` | 元素的类型 |
| `p = c.begin()` | `p` 指向 `c` 的第一个元素；也有 `cbegin()` 返回指向 const 的迭代器 |
| `p = c.end()` | `p` 指向 `c` 的最后一个元素之后的位置；也有 `cend()` 返回指向 const 的迭代器 |
| `k = c.size()` | `k` 是 `c` 中元素的数量 |
| `c.empty()` | `c` 是否为空？ |
| `k = c.capacity()` | `k` 是 `c` 在不进行新分配的情况下能容纳的元素数量 |
| `c.reserve(k)` | 将容量增加到 `k`；如果 `k <= c.capacity()`，则 `c.reserve(k)` 什么也不做 |
| `c.resize(k)` | 将元素数量设为 `k`；添加的元素具有默认值 `value_type{}` |
| `c[k]` | `c` 的第 `k` 个元素；从 0 开始；不保证范围检查 |
| `c.at(k)` | `c` 的第 `k` 个元素；如果超出范围，抛出 `out_of_range` |
| `c.push_back(x)` | 在 `c` 的末尾添加 `x`；使 `c` 的大小增加一 |
| `c.emplace_back(a)` | 在 `c` 的末尾添加 `value_type{a}`；使 `c` 的大小增加一 |
| `q = c.insert(p, x)` | 在 `c` 中的 `p` 之前添加 `x` |
| `q = c.erase(p)` | 删除 `c` 中位置 `p` 处的元素 |
| `c = c2` | 赋值：拷贝 `c2` 的所有元素，使得 `c == c2` |
| `b = (c == c2)` | `c` 和 `c2` 的所有元素相等；若相等则 `b == true` |
| `x = (c <=> c2)` | `c` 和 `c2` 的字典序比较：若 `c` 小于 `c2` 则 `x < 0`，相等则 `x == 0`，大于则 `x > 0`。<br>`!=`、`<`、`<=`、`>` 和 `>=` 由 `<=>` 生成 |

这种表示法和语义的统一性使得程序员能够提供与标准容器使用方式非常相似的新容器类型。范围检查的 `Vector`（§4.3，第 5 章）就是这样一个例子。容器接口的统一性使我们能够独立于单个容器类型来指定算法。然而，每种容器都有自己的优缺点。例如，对 `vector` 进行下标和遍历是廉价且容易的。另一方面，当我们插入或删除元素时，`vector` 的元素会被移动到不同的位置；`list` 则正好有相反的特性。请注意，对于包含小元素的短序列（即使对于 `insert()` 和 `erase()`），`vector` 通常比 `list` 更高效。我推荐使用标准库 `vector` 作为元素序列的默认类型：你需要一个理由才会选择其他容器。

===== 第 20 页 =====

考虑单向链表 `forward_list`，这是一个针对空序列进行优化的容器（§12.3）。一个空的 `forward_list` 只占用一个单词，而一个空的 `vector` 占用三个。空序列以及仅有一两个元素的序列出人意料地常见且有用。

**安置操作**（如 `emplace_back()`）接受元素构造函数的参数，并在容器中新分配的空间中构造对象，而不是将对象拷贝到容器中。例如，对于 `vector<pair<int,string>>`，我们可以写：

```cpp
v.push_back(pair{1, "copy or move"});   // 创建一个 pair 并将其移动到 v 中
v.emplace_back(1, "build in place");    // 在 v 中就地构建一个 pair
```

对于像这样的简单例子，优化可以使两次调用的性能相当。

## 12.9 建议

[1] STL 容器定义了一个序列；§12.2。
[2] STL 容器是资源句柄；§12.2，§12.3，§12.5，§12.6。
[3] 使用 `vector` 作为你的默认容器；§12.2，§12.8；[CG: SL.con.2]。
[4] 对于简单的容器遍历，使用范围 `for` 循环或一对 `begin/end` 迭代器；§12.2，§12.3。
[5] 使用 `reserve()` 避免使指向元素的指针和迭代器失效；§12.2。
[6] 不要在没有测量的情况下假设 `reserve()` 能带来性能提升；§12.2。
[7] 在容器上使用 `push_back()` 或 `resize()`，而不是在数组上使用 `realloc()`；§12.2。
[8] 不要使用指向已调整大小的 `vector` 的迭代器；§12.2；[CG: ES.65]。
[9] 不要假设 `[]` 会进行范围检查；§12.2。
[10] 当你需要保证范围检查时，使用 `at()`；§12.2；[CG: SL.con.3]。

===== 第 21 页 =====

[11] 使用范围 `for` 和标准库算法以零成本避免范围错误；§12.2.2。
[12] 元素被拷贝到容器中；§12.2.1。
[13] 为了保留元素的多态行为，存储指针（内置指针或用户定义指针）；§12.2.1。
[14] 插入操作，例如 `insert()` 和 `push_back()`，在 `vector` 上通常出人意料地高效；§12.3。
[15] 对于通常为空的序列，使用 `forward_list`；§12.8。
[16] 在性能方面，不要相信你的直觉：进行测量；§12.2。
[17] `map` 通常实现为红黑树；§12.5。
[18] `unordered_map` 是一个哈希表；§12.6。
[19] 通过引用传递容器，通过值返回容器；§12.2。
[20] 对于容器，使用 `()` 初始化语法表示大小，使用 `{}` 初始化语法表示元素序列；§5.2.3，§12.2。
[21] 优先选择紧凑且连续的数据结构；§12.3。
[22] `list` 的遍历相对昂贵；§12.3。
[23] 如果你需要对大量数据进行快速查找，请使用无序容器；§12.6。
[24] 如果你需要按顺序遍历元素，请使用有序容器（例如 `map` 和 `set`）；§12.5。
[25] 对于没有自然顺序（即没有合理的 `<`）的元素类型，使用无序容器（例如 `unordered_map`）；§12.5。
[26] 当你需要指向元素的指针在容器大小改变时保持稳定时，使用关联容器（例如 `map` 和 `list`）；§12.8。
[27] 通过实验检查你是否有一个可接受的哈希函数；§12.6。
[28] 通过使用异或运算符（`^`）组合元素的标准哈希函数得到的哈希函数通常是不错的；§12.6。
[29] 了解你的标准库容器，并优先使用它们而不是手工制作的数据结构；§12.8。
[30] 如果你的应用程序遇到与内存相关的性能问题，尽量减少自由存储区的使用，并/或考虑使用专门的分配器；§12.7。