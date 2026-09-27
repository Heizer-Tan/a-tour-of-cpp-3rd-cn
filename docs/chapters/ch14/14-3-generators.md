# 14.3 生成器

很多时候范围需要“临时生成”。标准库为此提供了少量简单的生成器（也称为工厂）：

**范围工厂 `<ranges>`**（`v` 是视图；`x` 为元素类型 `T` 的值；`is` 为输入流）

| 工厂 | 说明 |
|------|------|
| `v = empty_view<T>{}` | `v` 是元素类型为 `T` 的空范围（即便原本“会有”元素） |
| `v = single_view{x}` | `v` 只包含元素 `x` |
| `v = iota_view{x}` | `v` 是无限序列：`x, x+1, x+2, ...`（用 `++` 递增） |
| `v = iota_view{x, y}` | `v` 是含 `n` 个元素的序列：`x, x+1, ..., y-1`（用 `++` 递增） |
| `v = istream_view<T>{is}` | `v` 通过对 `is` 反复执行 `>>`（读取 `T`）得到 |

`iota_view` 很适合构造简单序列：

```cpp
for (int x : iota_view(42, 52)) // 42 43 44 45 46 47 48 49 50 51
    cout << x << ' ';
```

`istream_view` 让我们能在范围 `for` 里方便地使用输入流：

```cpp
for (auto x : istream_view<complex<double>>(cin))
    cout << x << '\n';
```

与其他视图一样，`istream_view` 也能与其他视图组合：

```cpp
auto cplx = istream_view<complex<double>>(cin);

for (auto x : transform_view(cplx, [](auto z) { return z * z; }))
    cout << x << '\n';
```

若输入为 `1 2 3`，输出将是 `1 4 9`。
