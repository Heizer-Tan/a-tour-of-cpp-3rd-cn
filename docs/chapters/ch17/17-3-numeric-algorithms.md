# 17.3 数值算法

在 `<numeric>` 中，我们找到了一小组泛化数值算法，例如 `accumulate()`。

**数值算法**

| 算法 | 描述 |
|------|------|
| `x = accumulate(b, e, i)` | `x` 是 `i` 与 `[b:e)` 中元素的和 |
| `x = accumulate(b, e, i, f)` | 使用 `f` 代替 `+` 的 `accumulate` |
| `x = inner_product(b, e, b2, i)` | `x` 是 `[b:e)` 与 `[b2:b2+(e-b))` 的内积，即 `i` 加上 `(*p1)*(*p2)` 的和 |
| `x = inner_product(b, e, b2, i, f, f2)` | 使用 `f` 和 `f2` 代替 `+` 和 `*` 的 `inner_product` |
| `p = partial_sum(b, e, out)` | `[out:p)` 的第 `i` 个元素是 `[b:b+i]` 元素的和 |
| `p = partial_sum(b, e, out, f)` | 使用 `f` 代替 `+` 的 `partial_sum` |
| `p = adjacent_difference(b, e, out)` | 对于 `i>0`，`[out:p)` 的第 `i` 个元素是 `*(b+i) - *(b+i-1)`；如果 `e-b>0`，则 `*out` 是 `*b` |
| `p = adjacent_difference(b, e, out, f)` | 使用 `f` 代替 `-` 的 `adjacent_difference` |
| `iota(b, e, v)` | 为 `[b:e)` 中的每个元素依次赋值 `v`、`v+1`、`v+2`…… |
| `x = gcd(n, m)` | `x` 是整数 `n` 和 `m` 的最大公约数 |
| `x = lcm(n, m)` | `x` 是整数 `n` 和 `m` 的最小公倍数 |
| `x = midpoint(n, m)` | `x` 是 `n` 和 `m` 的中点 |

这些算法将常见操作（如求和）泛化，使其适用于各种类型的序列。它们还将应用于序列元素的操作参数化。对于每个算法，通用版本都配有一个使用该算法最常见运算符的版本。例如：

```cpp
list<double> lst {1, 2, 3, 4, 5, 9999.99999};
auto s = accumulate(lst.begin(), lst.end(), 0.0);   // 计算和：10014.9999
```

这些算法适用于每个标准库序列，并且可以将操作作为参数提供（§17.3）。

### 17.3.1 并行数值算法

在 `<numeric>` 中，数值算法（§17.3）具有并行版本，它们与顺序版本略有不同。特别是，并行版本允许以未指定的顺序对元素进行操作。并行数值算法可以接受执行策略参数（§13.6）：`seq`、`unseq`、`par` 和 `par_unseq`。

**并行数值算法**

| 算法 | 描述 |
|------|------|
| `x = reduce(b, e, v)` | 与 `accumulate(b, e, v)` 相同，但无序 |
| `x = reduce(b, e)` | `reduce(b, e, V{})`，其中 `V` 是 `b` 的值类型 |
| `x = reduce(pol, b, e, v)` | 使用执行策略 `pol` 的 `reduce(b, e, v)` |
| `x = reduce(pol, b, e)` | `reduce(pol, b, e, V{})`，其中 `V` 是 `b` 的值类型 |
| `p = exclusive_scan(pol, b, e, out)` | 根据 `pol` 的 `partial_sum(b, e, out)`，从第 `i` 个和中排除第 `i` 个输入元素 |
| `p = inclusive_scan(pol, b, e, out)` | 根据 `pol` 的 `partial_sum(b, e, out)`，第 `i` 个和包含第 `i` 个输入元素 |
| `p = transform_reduce(pol, b, e, f, v)` | 先对 `[b:e)` 中每个 `x` 计算 `f(x)`，然后 `reduce` |
| `p = transform_exclusive_scan(pol, b, e, out, f, v)` | 先对 `[b:e)` 中每个 `x` 计算 `f(x)`，然后 `exclusive_scan` |
| `p = transform_inclusive_scan(pol, b, e, out, f, v)` | 先对 `[b:e)` 中每个 `x` 计算 `f(x)`，然后 `inclusive_scan` |

为简单起见，我省略了那些接受操作作为参数（而不是仅仅使用 `+` 和 `=`）的算法版本。除了 `reduce()` 之外，我还省略了带有默认策略（顺序）和默认值的版本。

就像 `<algorithm>` 中的并行算法（§13.6）一样，我们可以指定执行策略：

```cpp
vector<double> v {1, 2, 3, 4, 5, 9999.99999};
auto s = reduce(v.begin(), v.end());   // 使用 double 作为累加器计算和

vector<double> large;
// ... 用大量值填充 large ...
auto s2 = reduce(par_unseq, large.begin(), large.end());   // 使用并行和/或向量化计算和
```

执行策略 `par`、`seq`、`unseq` 和 `par_unseq` 隐藏在 `<execution>` 的命名空间 `std::execution` 中。

通过测量验证使用并行或向量化算法是否值得。
