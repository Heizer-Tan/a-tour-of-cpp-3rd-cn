# 13.2 迭代器的使用

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
