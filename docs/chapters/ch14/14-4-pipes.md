# 14.4 管道

对每个标准库视图（[§14.2](14-2-views.md)），标准库还提供一个产出“过滤器”的函数；也就是说，产出的对象可作为过滤器运算符 `|` 的操作数。例如，`filter()` 会给出一个 `filter_view`。于是可以把过滤器串成序列，而不必写成层层嵌套的函数调用：

```cpp
void user(forward_range auto& r)
{
    auto odd = [](int x) { return x % 2; };

    for (int x : r | views::filter(odd) | views::take(3))
        cout << x << ' ';
}
```

管道风格（沿用 Unix 管道运算符 `|`）普遍被认为比嵌套调用更易读。管道从左向右工作：对 `f|g` 而言，`f` 的结果被传给 `g`，因此 `r|f|g` 表示 `(g_filter(f_filter(r)))`。最初的 `r` 必须是一个范围或生成器。

这些过滤器函数位于命名空间 `ranges::views`：

```cpp
void user(forward_range auto& r)
{
    for (int x : r | views::filter([](int x) { return x % 2; }) | views::take(3))
        cout << x << ' ';
}
```

我发现显式写出 `views::` 往往更清楚；当然也可以进一步缩短：

```cpp
void user(forward_range auto& r)
{
    using namespace views;

    auto odd = [](int x) { return x % 2; };

    for (int x : r | filter(odd) | take(3))
        cout << x << ' ';
}
```

视图与管道的实现涉及相当令人咋舌的模板元编程；若你关心性能，请务必实测你的实现是否满足预期。否则总还有传统写法兜底：

```cpp
void user(forward_range auto& r)
{
    int count = 0;
    for (int x : r)
        if (x % 2) {
            cout << x << ' ';
            if (++count == 3)
                return;
        }
}
```

只不过在这种写法里，“到底在做什么”这层意图更容易被细节淹没。
