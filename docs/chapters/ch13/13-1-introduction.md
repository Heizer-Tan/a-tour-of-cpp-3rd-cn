# 13.1 引言

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
