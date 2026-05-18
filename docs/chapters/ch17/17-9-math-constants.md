# 17.9 数学常数

在进行数学计算时，我们需要常见的数学常数，例如 `e`、`pi` 和 `log2e`。标准库提供了这些以及更多。它们有两种形式：其一为模板，可指定确切类型（例如 `pi_v<T>`）；其二为最常见用法准备的短名字（例如 `pi` 即 `pi_v<double>`）。例如：

```cpp
void area(float r)
{
    using namespace std::numbers;   // 数学常数所在命名空间
    double d = pi * r * r;
    float f = pi_v<float> * r * r;
    // ...
}
```

此处的数值差别很小（往往要打印到十几位小数才能看出来），但在严肃的物理计算中很快就会放大。高精度常数在图形学、人工智能等领域也很重要——更小数值表示在这些场景日益常见。

在 `<numbers>` 中可以找到 `e`（欧拉数）、`log2e`、`log10e`、`pi`、`inv_pi`（`1/pi`）、`inv_sqrtpi`（`1/sqrt(pi)`）、`ln2`、`ln10`、`sqrt2`（`sqrt(2)`）、`sqrt3`（`sqrt(3)`）、`inv_sqrt3`、`egamma`（欧拉–马斯刻罗尼常数）与 `phi`（黄金比例）。

当然我们可能想要更多数学常数，或面向不同领域的常数。这很容易实现：它们是变量模板，并（默认）针对 `double`（或对给定领域最合适的类型）特化：

```cpp
template<typename T>
constexpr T tau_v = 2 * pi_v<T>;

constexpr double tau = tau_v<double>;
```
