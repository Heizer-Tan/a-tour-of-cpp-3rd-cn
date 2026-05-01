## 8.3 泛型编程

泛型编程的核心思想是将算法从具体的数据类型中抽象出来。C++ 标准库是泛型编程的典范——算法（如 `sort`、`find`）与容器（如 `vector`、`list`）通过迭代器（iterators）解耦。

### 8.3.1 迭代器

迭代器是泛型编程的关键抽象。它提供了一种统一的方式来遍历不同类型的容器：

```cpp
template<typename Iter, typename Value>
Iter find(Iter first, Iter last, const Value& val)
{
    while (first != last && *first != val)
        ++first;
    return first;
}
```

这个 `find` 函数适用于任何支持迭代器的容器——`vector`、`list`、`set`，甚至数组。

### 8.3.2 算法和容器

标准库算法通过迭代器与容器交互，实现了算法与容器的完全解耦：

```cpp
vector<int> v = {5, 3, 1, 4, 2};
std::sort(v.begin(), v.end());          // 排序
auto p = std::find(v.begin(), v.end(), 3); // 查找
int count = std::count_if(v.begin(), v.end(),
    [](int x) { return x % 2 == 0; });  // 计数偶数
```

### 8.3.3 范围（Ranges）

C++20 引入了*范围*（ranges）库，进一步简化了泛型编程：

```cpp
#include <ranges>

vector<int> v = {5, 3, 1, 4, 2};
std::ranges::sort(v);                   // 直接对容器排序

auto even = v | std::views::filter([](int x) { return x % 2 == 0; });
// even 是一个"视图"，惰性地过滤出偶数
```

范围库使用管道操作符 `|` 来组合操作，使代码更加简洁和可读。
