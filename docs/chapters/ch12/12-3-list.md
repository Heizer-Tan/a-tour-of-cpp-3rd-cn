# 12.3 list

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