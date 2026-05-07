# 12.2 vector

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
