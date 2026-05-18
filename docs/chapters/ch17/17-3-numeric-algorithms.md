# 17.3 数值算法

在 `<numeric>` 中可以找到一组泛化的数值算法，例如 `accumulate()`。

**数值算法**

| 运算 | 含义 |
|------|------|
| `x = accumulate(b, e, i)` | `x` 为 `i` 与 `[b:e)` 中元素之和 |
| `x = accumulate(b, e, i, f)` | 用 `f` 取代 `+` 做 accumulate |
| `x = inner_product(b, e, b2, i)` | `x` 为 `[b:e)` 与 `[b2:b2+(e-b))` 的内积；即 `i` 加上每个 `p1 ∈ [b:e)` 与对应 `p2` 的 `(*p1)*(*p2)` 之和 |
| `x = inner_product(b, e, b2, i, f, f2)` | 用 `f`、`f2` 取代 `+`、`*` |
| `p = partial_sum(b, e, out)` | `[out:p)` 的第 `i` 个元素为 `[b:b+i]` 的元素之和 |
| `p = partial_sum(b, e, out, f)` | 用 `f` 取代 `+` |
| `p = adjacent_difference(b, e, out)` | `[out:p)` 的第 `i` 个元素：若 `i>0` 为 `*(b+i)-*(b+i-1)`；若 `e-b>0` 则 `*out` 为 `*b` |
| `p = adjacent_difference(b, e, out, f)` | 用 `f` 取代减法 |
| `iota(b, e, v)` | 对 `[b:e)` 中每个元素赋值 `v` 并执行 `++v`，从而序列变为 `v, v+1, v+2, …` |
| `x = gcd(n, m)` | `x` 为整数 `n`、`m` 的最大公约数 |
| `x = lcm(n, m)` | `x` 为最小公倍数 |
| `x = midpoint(n, m)` | `x` 为 `n` 与 `m` 的中点 |

这些算法通过允许把运算作用于任意序列，并把施加在元素上的操作参数化，从而推广了诸如求和之类的常见操作。对每个算法，泛化版本都辅以使用最常见运算符的版本。例如：

```cpp
list<double> lst{1, 2, 3, 4, 5, 9999.99999};
auto s = accumulate(lst.begin(), lst.end(), 0.0);   // 求和：约 10014.9999
```

这些算法适用于每一种标准库序列，并且能把运算作为实参传入（§17.3）。

## 17.3.1

在 `<numeric>` 中，数值算法（§17.3）还有并行版本，与顺序版本略有不同。特别是，并行版本允许按未指明顺序对元素进行操作。并行数值算法可以接受执行策略实参（§13.6）：`seq`、`unseq`、`par`、`par_unseq`。

**并行数值算法**

| 运算 | 含义 |
|------|------|
| `x = reduce(b, e, v)` | 类似 `accumulate(b, e, v)`，但顺序未定 |
| `x = reduce(b, e)` | `x = reduce(b, e, V{})`，`V` 为 `b` 的值类型 |
| `x = reduce(pol, b, e, v)` | 带执行策略 `pol` 的 `reduce(b, e, v)` |
| `x = reduce(pol, b, e)` | 带策略 `pol` 的 `reduce(b, e, V{})` |
| `p = exclusive_scan(pol, b, e, out)` | 按策略 `pol` 做 `partial_sum`，第 `i` 个和**不包含**第 `i` 个输入 |
| `p = inclusive_scan(pol, b, e, out)` | 按策略 `pol` 做 `partial_sum`，第 `i` 个和**包含**第 `i` 个输入 |
| `p = transform_reduce(pol, b, e, f, v)` | 对 `[b:e)` 中每个 `x` 计算 `f(x)`，再 reduce |
| `p = transform_exclusive_scan(pol, b, e, out, f, v)` | 先 `f(x)`，再 `exclusive_scan` |
| `p = transform_inclusive_scan(pol, b, e, out, f, v)` | 先 `f(x)`，再 `inclusive_scan` |

为简洁起见，我省略了这些算法中带自定义运算（而非只用 `+` 与 `=`）的版本。除 `reduce()` 外，也省略了带默认策略（顺序）与默认值的版本。

与 `<algorithm>` 中的并行算法（§13.6）一样，我们可以指定执行策略：

```cpp
vector<double> v{1, 2, 3, 4, 5, 9999.99999};
auto s = reduce(v.begin(), v.end());              // 用 double 累加求和

vector<double> large;
// ... 向 large 填入大量数据 ...
auto s2 = reduce(par_unseq, large.begin(), large.end()); // 并行、可向量化求和
```

执行策略 `par`、`seq`、`unseq`、`par_unseq` 定义在 `<execution>` 的命名空间 `std::execution` 中。

请通过**测量**验证使用并行或向量化算法是否划算。
