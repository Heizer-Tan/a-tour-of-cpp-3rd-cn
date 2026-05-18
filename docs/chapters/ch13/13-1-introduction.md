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
