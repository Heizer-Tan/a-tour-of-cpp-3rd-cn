# 14.2 视图

**视图**（view）是一种“如何去看某个范围”的方式。例如：

```cpp
void user(forward_range auto& r)
{
    filter_view v {r, [](int x) { return x % 2; }}; // 只看 r 中的奇数
    cout << "odd numbers: ";
    for (int x : v)
        cout << x << ' ';
}
```

从 `filter_view` 读取时，实际上是从其底层范围读取：若读到的值满足谓词就返回；否则 `filter_view` 会继续尝试下一个元素。

许多范围是无限的；而我们往往只需要很少的值。因此还有只“截取前若干个元素”的视图：

```cpp
void user(forward_range auto& r)
{
    filter_view v {r, [](int x) { return x % 2; }}; // r 中的奇数
    take_view tv {v, 100};                       // 从 v 中最多取 100 个元素

    cout << "odd numbers: ";
    for (int x : tv)
        cout << x << ' ';
}
```

也可以不为 `take_view` 单独命名，直接写：

```cpp
for (int x : take_view{v, 3})
    cout << x << ' ';
```

对 `filter_view` 也类似：

```cpp
for (int x : take_view{filter_view{r, [](int x) { return x % 2; }}, 3})
    cout << x << ' ';
```

这种层层嵌套的视图很快就会晦涩难懂，因此还有另一条路可走：**管道**（§14.4）。

标准库提供了很多视图（也常被称为范围适配器）：

**标准库视图（范围适配器）`<ranges>`**（`v` 是视图；`r` 是范围；`p` 是谓词；`n` 是整数）

| 视图 | 说明 |
|------|------|
| `v = all_view{r}` | `v` 包含 `r` 的全部元素 |
| `v = filter_view{r, p}` | `v` 包含 `r` 中满足 `p` 的元素 |
| `v = transform_view{r, f}` | `v` 包含对 `r` 中各元素调用 `f` 的结果 |
| `v = take_view{r, n}` | `v` 至多包含 `r` 的前 `n` 个元素 |
| `v = take_while_view{r, p}` | `v` 包含 `r` 的开头一段连续元素，直到某个元素不满足 `p` |
| `v = drop_view{r, n}` | `v` 从 `r` 的第 `n+1` 个元素开始 |
| `v = drop_while_view{r, p}` | `v` 从 `r` 中首个不满足 `p` 的元素开始 |
| `v = join_view{r}` | `v` 是“压平”后的范围；`r` 的元素本身必须也是范围 |
| `v = split_view(r, d)` | `v` 是由分隔符 `d` 切分 `r` 得到的子范围序列；`d` 可以是单个元素或一个范围 |
| `v = common_view(r)` | `v` 用普通的 `{begin, end}` 迭代器对描述 `r` |
| `v = reverse_view{r}` | `v` 按逆序遍历 `r`；要求 `r` 支持双向迭代 |
| `v = views::elements<n>(r)` | `v` 取 `r` 中每个元组元素的第 `n` 个分量 |
| `v = keys_view{r}` | `v` 取 `r` 中每个 `pair` 的第一个分量 |
| `v = values_view{r}` | `v` 取 `r` 中每个 `pair` 的第二个分量 |
| `v = ref_view{r}` | `v` 提供对 `r` 元素的引用视图 |

视图对外呈现的接口与范围极为相似，因此在绝大多数场合可以把视图当作范围那样使用。关键区别在于：**视图不拥有元素**，也不负责释放底层范围里的对象——那是底层范围的责任。另一方面，视图的生命周期不能长于它所观测的范围：

```cpp
auto bad()
{
    vector v = {1, 2, 3, 4};
    return filter_view{v, [](int x) { return x % 2; }}; // 危险：`v` 会先被销毁
}
```

视图应当廉价拷贝，因此通常按值传递。

示例里我用了最简单的标准类型来保持短小；当然也能为用户自定义类型建立视图。例如：

```cpp
struct Reading {
    int location {};
    int temperature {};
    int humidity {};
    int air_pressure {};
    // ...
};

int average_temp(vector<Reading> readings)
{
    if (readings.size() == 0)
        throw No_readings{};
    double s = 0;
    for (int x : views::elements<1>(readings)) // 只看温度字段
        s += x;
    return s / readings.size();
}
```
