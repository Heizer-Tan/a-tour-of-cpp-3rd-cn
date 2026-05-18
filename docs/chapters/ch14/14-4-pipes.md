# 14.4 管道

对每个标准库视图（§14.2），标准库还提供相应的“过滤器工厂函数”，产出的对象可作为 `|`（管道）运算符的操作数。例如 `views::filter` 最终会给出类似 `filter_view` 的行为。于是可以把一连串过滤器横向串起来，而不必写成层层嵌套的函数调用：

```cpp
void user(forward_range auto& r)
{
    auto odd = [](int x) { return x % 2; };

    for (int x : r | views::filter(odd) | views::take(3))
        cout << x << ' ';
}
```

管道风格（沿用 Unix shell 里熟悉的 `|`）普遍被认为比嵌套调用更易读。管道从左向右结合：`r | f | g` 表示先把 `r` 送入 `f`，再把结果送入 `g`。

这些过滤器函数位于 `ranges::views`：

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

视图与管道的实现依赖相当精巧的模板元编程；若你关心性能，请务必实测你的实现是否满足预期。否则总还有传统写法兜底：

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
