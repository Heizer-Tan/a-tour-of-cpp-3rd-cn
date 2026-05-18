# 17.2 数学函数

在 `<cmath>` 中可以找到标准数学函数，例如面向 `float`、`double` 与 `long double` 实参的 `sqrt()`、`log()`、`sin()` 等：

**所选标准数学函数**

| 函数 | 含义 |
|------|------|
| `abs(x)` | 绝对值 |
| `ceil(x)` | 不小于 `x` 的最小整数 |
| `floor(x)` | 不大于 `x` 的最大整数 |
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
| `log(x)` | 自然对数（底 `e`）；`x` 必须为正 |
| `log2(x)` | 以 2 为底的对数；`x` 必须为正 |
| `log10(x)` | 常用对数（底 10）；`x` 必须为正 |

复数版本（§17.4）位于 `<complex>`。

对每个函数，返回类型与实参类型相同。

错误通过把 `<cerrno>` 中的 `errno` 设为 `EDOM`（定义域错误）或 `ERANGE`（值域错误）来报告。例如：

```cpp
errno = 0;              // 清除旧状态
double d = sqrt(-1);    // 示意：非法实参
if (errno == EDOM)
    cerr << "sqrt() not defined for negative argument\n";

errno = 0;
double dd = pow(numeric_limits<double>::max(), 2);
if (errno == ERANGE)
    cerr << "result of pow() too large to represent as a double\n";
```

更多数学函数见 `<cmath>` 与 `<cstdlib`。所谓**特殊数学函数**，例如 `beta()`、`riemann_zeta()`、`sph_bessel()` 等，也在 `<cmath>` 中。
