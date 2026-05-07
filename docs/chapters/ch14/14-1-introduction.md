# 14.1 引言

标准库提供的算法既有使用概念约束的版本（第 8 章），也有无约束的版本（出于兼容性考虑）。受约束的（使用概念的）版本位于 `<ranges>` 中，在命名空间 `ranges` 下。自然，我更喜欢使用概念的版本。**范围**（range）是对 C++98 中由 `{begin(), end()}` 对定义的序列的泛化；它规定了成为一个元素序列需要满足的条件。一个范围可以由以下方式定义：

- 一对迭代器 `{begin, end}`
- 一对 `{begin, n}`，其中 `begin` 是迭代器，`n` 是元素个数
- 一对 `{begin, pred}`，其中 `begin` 是迭代器，`pred` 是一个谓词；如果对迭代器 `p` 有 `pred(p)` 为真，则表示已到达范围的末尾。这允许我们拥有无限范围以及“按需生成”的范围（§14.3）。

正是这个 range 概念让我们可以写 `sort(v)` 而不是自 1994 年以来使用 STL 时必须写的 `sort(v.begin(), v.end())`。对于自己的算法，我们也可以类似地处理：

```cpp
template<forward_range R>
    requires sortable<iterator_t<R>>
void my_sort(R& r)   // 现代的、使用概念约束的 my_sort 版本
{
    return my_sort(r.begin(), end());   // 使用 1994 风格的 sort
}
```

Ranges 使我们能够更直接地表达大约 99% 的常见算法使用场景。除了符号上的优势外，ranges 还提供了一些优化机会，并消除了一类“愚蠢的错误”，例如 `sort(v1.begin(), v2.end())` 和 `sort(v.end(), v.begin())`。是的，这类错误在“现实中”确实出现过。

===== 第 3 页 =====

自然，存在不同类型的范围对应于不同类型的迭代器。特别地，`input_range`、`forward_range`、`bidirectional_range`、`random_access_range` 和 `contiguous_range` 都以概念的形式表示（§14.5）。
