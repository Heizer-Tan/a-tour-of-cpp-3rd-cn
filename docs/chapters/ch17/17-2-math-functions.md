# 17.2 数学函数

在 `<cmath>` 中，我们找到了标准数学函数，例如 `sqrt()`、`log()` 和 `sin()`，它们的参数类型可以是 `float`、`double` 和 `long double`：

**选定的标准数学函数**

| 函数 | 描述 |
|------|------|
| `abs(x)` | 绝对值 |
| `ceil(x)` | 大于等于 `x` 的最小整数 |
| `floor(x)` | 小于等于 `x` 的最大整数 |
| `sqrt(x)` | 平方根；`x` 必须非负 |
| `cos(x)` | 余弦 |
| `sin(x)` | 正弦 |
| `tan(x)` | 正切 |
| `acos(x)` | 反余弦；结果非负 |
| `asin(x)` | 反正弦；返回最接近 0 的结果 |
| `atan(x)` | 反正切 |
| `sinh(x)` | 双曲正弦 |
| `cosh(x)` | 双曲余弦 |
| `tanh(x)` | 双曲正切 |
| `exp(x)` | 以 `e` 为底的指数 |
| `exp2(x)` | 以 2 为底的指数 |
| `log(x)` | 自然对数（以 `e` 为底）；`x` 必须为正 |
| `log2(x)` | 以 2 为底的对数；`x` 必须为正 |
| `log10(x)` | 以 10 为底的对数；`x` 必须为正 |

这些函数针对复数的版本（§17.4）位于 `<complex>` 中。对于每个函数，返回类型与参数类型相同。

通过将 `<cerrno>` 中的 `errno` 设置为 `EDOM`（定义域错误）或 `ERANGE`（值域错误）来报告错误。例如：

```cpp
errno = 0;                     // 清除旧的错误状态
double d = sqrt(-1);
if (errno == EDOM)
    cerr << "sqrt() not defined for negative argument\n";

errno = 0;                     // 清除旧的错误状态
double dd = pow(numeric_limits<double>::max(), 2);
if (errno == ERANGE)
    cerr << "result of pow() too large to represent as a double\n";
```

===== 第 4 页 =====

在 `<cmath>` 和 `<cstdlib>` 中可以找到更多数学函数。所谓的特殊数学函数（例如 `beta()`、`rieman_zeta()` 和 `sph_bessel()`）也在 `<cmath>` 中。
