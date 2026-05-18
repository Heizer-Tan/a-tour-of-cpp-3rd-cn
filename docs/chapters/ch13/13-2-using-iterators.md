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
