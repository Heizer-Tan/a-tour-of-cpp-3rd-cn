
对于每个标准库视图（§14.2），标准库还提供了一个生成该视图的函数；也就是说，一个可以作为 `|` 运算符参数的对象。例如，`filter()` 产生一个 `filter_view`。这允许我们按顺序组合这些过滤器，而不是将它们呈现为一组嵌套的函数调用。

```cpp
void user(forward_range auto& r)
{
    auto odd = [](int x) { return x % 2; };
    for (int x : r | views::filter(odd) | views::take(3))
        cout << x << ' ';
}
```

输入范围 `1 2 3 4 5 6 7 8 9 10` 会产生 `1 2 3`。

管道风格（使用 Unix 管道运算符 `|`）被广泛认为比嵌套函数调用更具可读性。管道从左到右工作；即 `f | g` 将 `f` 的结果传递给 `g`，因此 `r | f | g` 意味着 `(g_filter(f_filter(r)))`。初始的 `r` 必须是一个范围或生成器。

这些过滤器函数位于命名空间 `ranges::views` 中：

```cpp
void user(forward_range auto& r)
{
    for (int x : r | views::filter([](int x) { return x % 2; }) | views::take(3))
        cout << x << ' ';
}
```

我发现显式使用 `views::` 使代码相当可读，但当然我们可以进一步缩短代码：

```cpp
void user(forward_range auto& r)
{
    using namespace views;
    auto odd = [](int x) { return x % 2; };
    for (int x : r | filter(odd) | take(3))
        cout << x << ' ';
}
```

视图和管道的实现涉及一些相当棘手的模板元编程，因此如果你担心性能，请务必测量你的实现是否满足你的需求。如果不满足，总有一个传统的变通方法：

```cpp
void user(forward_range auto& r)
{
    int count = 0;
    for (int x : r)
        if (x % 2) {
            cout << x << ' ';
            if (++count == 3) return;
        }
}
```

然而，在这里，其中的逻辑被掩盖了。

## 14.5 概念概览