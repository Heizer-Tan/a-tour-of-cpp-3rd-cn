
通常，范围需要按需生成。标准库为此提供了几个简单的生成器（也称为工厂）：

**范围工厂 `<ranges>`**（`v` 是视图；`x` 是元素类型 `T` 的值；`is` 是一个 `istream`）

| 生成器 | 描述 |
|--------|------|
| `v = empty_view<T>{}` | `v` 是一个空的 `T` 类型元素的范围（如果它有元素的话） |
| `v = single_view{x}` | `v` 是包含单个元素 `x` 的范围 |
| `v = iota_view{x}` | `v` 是一个无限范围：`x, x+1, x+2, ...`（使用 `++` 递增） |
| `v = iota_view{x, y}` | `v` 是一个包含 `n` 个元素的范围：`x, x+1, ..., y-1`（使用 `++` 递增） |
| `v = istream_view<T>{is}` | `v` 是通过对 `is` 调用 `>>` 获取 `T` 值所得到的范围 |

`iota_view` 对于生成简单序列非常有用。例如：

```cpp
for (int x : iota_view(42, 52))   // 42 43 44 45 46 47 48 49 50 51
    cout << x << ' ';
```

`istream_view` 为我们提供了一种在 range-for 循环中使用 `istream` 的简单方法：

```cpp
for (auto x : istream_view<complex<double>>(cin))
    cout << x << '\n';
```

像其他视图一样，`istream_view` 可以与其他视图组合：

```cpp
auto cplx = istream_view<complex<double>>(cin);
for (auto x : transform_view(cplx, [](auto z) { return z * z; }))
    cout << x << '\n';
```

输入 `1 2 3` 会产生 `1 4 9`。

## 14.4 管道