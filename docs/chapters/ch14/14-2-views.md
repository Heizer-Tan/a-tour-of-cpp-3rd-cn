# 14.2 视图

**视图**（view）是一种查看范围的方式。例如：

```cpp
void user(forward_range auto& r)
{
    filter_view v {r, [](int x) { return x % 2; }};   // 只查看 r 中的奇数
    cout << "odd numbers: ";
    for (int x : v)
        cout << x << ' ';
}
```

当从 `filter_view` 读取时，我们从它的底层范围读取。如果读取的值匹配谓词，则返回它；否则，`filter_view` 会尝试从范围中获取下一个元素。

许多范围是无限的。此外，我们通常只想要少数几个值。因此，有一些视图只从范围中取少量值：

```cpp
void user(forward_range auto& r)
{
    filter_view v {r, [](int x) { return x % 2; }};   // 只查看 r 中的奇数
    take_view tv {v, 100};                            // 最多从 v 中取 100 个元素
    cout << "odd numbers: ";
    for (int x : tv)
        cout << x << ' ';
}
```

我们可以直接使用 `take_view` 而不用给它命名：

```cpp
for (int x : take_view{v, 3})
    cout << x << ' ';
```

类似地对于 `filter_view`：

```cpp
for (int x : take_view{ filter_view { r, [](int x) { return x % 2; } }, 3 })
    cout << x << ' ';
```

这种视图的嵌套很快就会变得有些晦涩，因此有一种替代方案：管道（§14.4）。

标准库提供了许多视图，也称为范围适配器：

**标准库视图（范围适配器）`<ranges>`**（`v` 是视图；`r` 是范围；`p` 是谓词；`n` 是整数）

| 视图 | 描述 |
|------|------|
| `v = all_view{r}` | `v` 是 `r` 中的所有元素 |
| `v = filter_view{r, p}` | `v` 是 `r` 中满足 `p` 的元素 |
| `v = transform_view{r, f}` | `v` 是对 `r` 中每个元素调用 `f` 的结果 |
| `v = take_view{r, n}` | `v` 是 `r` 中最多 `n` 个元素 |
| `v = take_while_view{r, p}` | `v` 是 `r` 中直到（但不包括）第一个不满足 `p` 的元素为止的元素 |
| `v = drop_view{r, n}` | `v` 是 `r` 中从第 `n+1` 个元素开始的元素 |
| `v = drop_while_view{r, p}` | `v` 是 `r` 中从第一个不满足 `p` 的元素开始的元素 |
| `v = join_view{r}` | `v` 是 `r` 的扁平化版本；`r` 的元素本身必须是范围 |
| `v = split_view(r, d)` | `v` 是由分隔符 `d` 确定的 `r` 的子范围的范围；`d` 必须是一个元素或一个范围 |
| `v = common_view(r)` | `v` 是由 `{begin, end}` 对描述的 `r` |
| `v = reverse_view{r}` | `v` 是 `r` 中元素的反向顺序；`r` 必须支持双向访问 |
| `v = views::elements<n>(r)` | `v` 是 `r` 中元组元素的第 `n` 个元素的范围 |
| `v = keys_view{r}` | `v` 是 `r` 中 pair 元素的第一个元素的范围 |
| `v = values_view{r}` | `v` 是 `r` 中 pair 元素的第二个元素的范围 |
| `v = ref_view{r}` | `v` 是 `r` 的元素的引用范围 |

视图提供的接口与范围非常相似，因此在大多数情况下，我们可以在任何可以使用范围的地方以相同的方式使用视图。关键区别在于视图不拥有其元素；它不负责删除其底层范围的元素——那是范围的责任。另一方面，视图不能超出其范围的生命周期：

```cpp
auto bad()
{
    vector v = {1, 2, 3, 4};
    return filter_view{v, odd};   // v 将在视图之前被销毁
}
```

视图应该廉价拷贝，因此我们按值传递它们。

我使用了简单的标准类型来保持示例简单，但当然，我们也可以拥有我们自己用户定义类型的视图。例如：

```cpp
struct Reading {
    int location {};
    int temperature {};
    int humidity {};
    int air_pressure {};
    //...
};

int average_temp(vector<Reading> readings)
{
    if (readings.size() == 0) throw No_readings{};
    double s = 0;
    for (int x : views::elements<1>(readings))   // 只查看温度字段
        s += x;
    return s / readings.size();
}
```

## 14.3 生成器