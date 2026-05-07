
在进行数学计算时，我们需要常见的数学常数，例如 `e`、`pi` 和 `log2e`。标准库提供了这些以及更多。它们有两种形式：一个允许我们指定确切类型的模板（例如 `pi_v<T>`），以及一个用于最常见用途的短名称（例如 `pi` 表示 `pi_v<double>`）。例如：

```cpp
void area(float r)
{
    using namespace std::numbers;   // 这是数学常数所在的地方
    double d = pi * r * r;
    float f = pi_v<float> * r * r;
    // ...
}
```

在这种情况下，差异很小（我们需要打印大约 16 位精度才能看到），但在真正的物理计算中，这种差异很快就会变得显著。常数精度重要的其他领域包括图形和 AI，其中值的较小表示越来越重要。

在 `<numbers>` 中，我们找到 `e`（欧拉数）、`log2e`（以 2 为底的 `e` 的对数）、`log10e`（以 10 为底的 `e` 的对数）、`pi`、`inv_pi`（`1/pi`）、`inv_sqrtpi`（`1/sqrt(pi)`）、`ln2`、`ln10`、`sqrt2`（`sqrt(2)`）、`sqrt3`（`sqrt(3)`）、`inv_sqrt3`（`1/sqrt3`）、`egamma`（欧拉-马斯刻若尼常数）和 `phi`（黄金比例）。

当然，我们可能希望有更多的数学常数和针对不同领域的常数。这很容易做到，因为这样的常数是变量模板，并针对 `double`（或任何对领域最有用的类型）进行了特化：

```cpp
template<typename T>
constexpr T tau_v = 2 * pi_v<T>;

constexpr double tau = tau_v<double>;
```

## 17.10 建议