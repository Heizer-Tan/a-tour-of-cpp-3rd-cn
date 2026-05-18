# 14.1 引言

标准库同时提供两类算法：一类用概念（第 8 章）加以约束，另一类保持无约束以兼容旧代码。受约束（基于概念）的版本位于 `<ranges>`，命名空间为 `ranges`。自然，我更偏好使用概念的版本。

**范围**（range）是对 C++98 时代那种由 `{begin(), end()}` 迭代器对所刻画序列的推广：它说明“要成为一串元素序列需要具备什么”。一个范围可以由以下形式给出：

- 一对迭代器 `{begin, end}`；
- `{begin, n}`，其中 `begin` 是迭代器，`n` 是元素个数；
- `{begin, pred}`，其中 `begin` 是迭代器，`pred` 是谓词；若对某个迭代器 `p` 有 `pred(p)` 为真，则表示到达范围末尾。这允许存在无限范围，以及按需生成的范围（§14.3）。

正是 range 概念让我们可以写 `sort(v)`，而不必像自 1994 年以来使用 STL 那样写 `sort(v.begin(), v.end())`。对自己的算法也能做类似处理：

```cpp
template<forward_range R>
    requires sortable<iterator_t<R>>
void my_sort(R& r) // 现代的、带概念约束的 my_sort 版本
{
    my_sort(r.begin(), r.end()); // 委托给（假定存在的）迭代器版本
}
```

Ranges 让我们能更直接地表达大约 99% 的日常算法用法。除了记法上的好处，ranges 还能带来某些优化机会，并消灭一整类低级错误——例如 `sort(v1.begin(), v2.end())`、`sort(v.end(), v.begin())` 这种组合；现实中确实见过。

自然，也存在不同“种类”的范围，对应不同种类的迭代器。尤其地，`input_range`、`forward_range`、`bidirectional_range`、`random_access_range` 与 `contiguous_range` 都以概念的形式给出（§14.5）。
