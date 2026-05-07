
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
