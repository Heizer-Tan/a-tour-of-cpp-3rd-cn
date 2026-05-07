===== 第 1 页 =====

17

## 数值计算

计算的目的是洞察，而不是数字。 – R. W. 汉明

……但对于学生而言，数字往往是通往洞察的最佳途径。 – A. 拉尔斯顿

- 引言
- 数学函数
- 数值算法

===== 第 2 页 =====

 - 并行数值算法
- 复数
- 随机数
- 向量算术
- 数值极限
- 类型别名
- 数学常数
- 建议

## 17.1 引言

C++ 在设计之初并非主要考虑数值计算。然而，数值计算通常发生在其他工作的背景下——例如科学计算、数据库访问、网络、仪器控制、图形、模拟和金融分析——因此 C++ 成为大型系统中计算部分的一个有吸引力的载体。此外，数值方法已经远远超越了简单的浮点数向量循环。当计算中需要更复杂的数据结构时，C++ 的优势就变得重要起来。最终结果是 C++ 被广泛用于科学、工程、金融以及其他涉及复杂数值计算的应用。因此，支持此类计算的设施和技术已经出现。本章描述了标准库中支持数值计算的部分。

## 17.2 数学函数

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

## 17.3 数值算法

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

## 17.4 复数

标准库支持一系列复数类型，遵循 §5.2.1 中描述的 `complex` 类的路线。为了支持标量为单精度浮点数（`float`）、双精度浮点数（`double`）等的复数，标准库的 `complex` 是一个模板：

```cpp
template<typename Scalar>
class complex {
public:
    complex(const Scalar& re = 0, const Scalar& im = 0);   // 默认函数参数
    // ...
};
```

复数支持通常的算术运算和最常见的数学函数。例如：

```cpp
void f(complex<float> fl, complex<double> db)
{
    complex<long double> ld {fl + sqrt(db)};
    db += fl * 3;
    fl = pow(1/fl, 2);
    // ...
}
```

`sqrt()` 和 `pow()`（幂函数）是 `<complex>` 中定义的常见数学函数之一（§17.2）。

## 17.5 随机数

随机数在许多场景中很有用，例如测试、游戏、模拟和安全。应用领域的多样性反映在标准库在 `<random>` 中提供的广泛随机数生成器选择上。一个随机数生成器由两部分组成：

1. **引擎**：生成随机或伪随机值序列。
2. **分布**：将这些值映射到某个范围内的数学分布。

分布的示例包括 `uniform_int_distribution`（所有生成的整数等可能）、`normal_distribution`（“钟形曲线”）以及 `exponential_distribution`（指数增长）；每个分布都针对某个指定范围。例如：

```cpp
using my_engine = default_random_engine;          // 引擎类型
using my_distribution = uniform_int_distribution<>; // 分布类型

my_engine eng {};                                 // 引擎的默认版本
my_distribution dist {1, 6};                      // 映射到 [1,6] 的分布
auto die = [&]() { return dist(eng); };           // 生成一个生成器

int x = die();                                    // 掷骰子：x 成为 1 到 6 之间的值
```

由于其对通用性和性能的不懈关注，一位专家认为标准库的随机数组件是“每个随机数库长大后的理想形态”。然而，它很难被认为是“对新手友好的”。`using` 语句和 lambda 使得所做的事情稍微明显一些。

对于新手（无论背景如何），随机数库的完全通用接口可能是一个严重的障碍。一个简单的均匀随机数生成器通常足以入门。例如：

```cpp
Rand_int rnd {1, 10};   // 创建一个 [1:10] 的随机数生成器
int x = rnd();          // x 是 [1,10] 中的一个数
```

那么，我们如何得到这个？我们必须得到一个像 `die()` 一样的东西，将引擎和分布组合在一个 `Rand_int` 类中：

```cpp
class Rand_int {
public:
    Rand_int(int low, int high) : dist{low, high} { }
    int operator()() { return dist(re); }   // 抽取一个整数
    void seed(int s) { re.seed(s); }        // 选择新的随机引擎种子
private:
    default_random_engine re;
    uniform_int_distribution<> dist;
};
```

这个定义仍然是“专家级别”的，但 `Rand_int()` 的使用在 C++ 新手课程的第一周是可以管理的。例如：

```cpp
int main()
{
    constexpr int max = 9;
    Rand_int rnd {0, max};                      // 创建一个均匀随机数生成器

    vector<int> histogram(max + 1);             // 创建合适大小的向量

    for (int i = 0; i != 200; ++i)
        ++histogram[rnd()];                     // 用频率填充直方图

    for (int i = 0; i != histogram.size(); ++i) { // 输出条形图
        cout << i << '\t';
        for (int j = 0; j != histogram[i]; ++j) cout << '*';
        cout << '\n';
    }
}
```

输出是一个（令人放心的平淡）均匀分布（具有合理的统计变化）：

```
0   *********************
1   ****************
2   *******************
3   ********************
4   ****************
5   ***********************
6   **************************
7   ***********
8   **********************
9   *************************
```

C++ 没有标准的图形库，因此我使用“ASCII 图形”。显然，有很多开源和商业的 C++ 图形和 GUI 库，但在本书中我仅限于 ISO 标准设施。

为了获得重复或不同的值序列，我们**播种**引擎；也就是说，我们给它的内部状态赋一个新值。例如：

```cpp
Rand_int rnd {10, 20};
for (int i = 0; i < 10; ++i) cout << rnd() << ' ';   // 16 13 20 19 14 17 10 16 15 14
cout << '\n';
rnd.seed(999);
for (int i = 0; i < 10; ++i) cout << rnd() << ' ';   // 11 17 14 19 20 13 20 14 16 19
cout << '\n';
rnd.seed(999);
for (int i = 0; i < 10; ++i) cout << rnd() << ' ';   // 11 17 14 19 20 13 20 14 16 19
cout << '\n';
```

重复序列对于确定性调试很重要。当我们不希望重复时，使用不同的值播种很重要。如果你需要真正的随机数，而不是生成的伪随机序列，请查看 `random_device` 在你机器上的实现方式。

## 17.6 向量算术

§12.2 中描述的 `vector` 被设计为一种通用的值存储机制，灵活且适合容器、迭代器和算法的架构。然而，它不支持数学向量运算。将此类操作添加到 `vector` 很容易，但其通用性和灵活性排除了对于严肃数值工作通常被认为是必需的优化。因此，标准库在 `<valarray>` 中提供了一个类似 `vector` 的模板，称为 `valarray`，它不那么通用，但更易于针对数值计算进行优化：

```cpp
template<typename T>
class valarray {
    // ...
};
```

`valarray` 支持通常的算术运算和最常见的数学函数。例如：

```cpp
void f(valarray<double>& a1, valarray<double>& a2)
{
    valarray<double> a = a1 * 3.14 + a2 / a1;   // 数值数组运算符 *, +, /
    a2 += a1 * 3.14;
    a = abs(a);
    double d = a2[7];
    // ...
}
```

这些操作是向量操作；也就是说，它们被应用于所涉及向量的每个元素。

除了算术运算，`valarray` 还提供跨步访问以帮助实现多维计算。

## 17.7 数值极限

在 `<limits>` 中，标准库提供了描述内置类型属性的类——例如 `float` 的最大指数或 `int` 的字节数。例如，我们可以断言 `char` 是有符号的：

```cpp
static_assert(numeric_limits<char>::is_signed, "unsigned characters!");
static_assert(100000 < numeric_limits<int>::max(), "small ints!");
```

第二个断言之所以有效，是因为 `numeric_limits<int>::max()` 是一个 `constexpr` 函数（§1.6）。

我们可以为我们自己的用户定义类型定义 `numeric_limits`。

## 17.8 类型别名

基本类型（如 `int` 和 `long long`）的大小是**实现定义**的；也就是说，它们在不同的 C++ 实现上可能不同。如果需要明确整数的具体大小，我们可以使用 `<cstdint>` 中定义的别名，例如 `int32_t` 和 `uint_least64_t`。后者表示至少 64 位的无符号整数。

奇怪的 `_t` 后缀是 C 语言时代的遗留物，当时认为名称反映其是别名很重要。

其他常见别名，如 `size_t`（`sizeof` 运算符返回的类型）和 `ptrdiff_t`（两个指针相减结果的类型），可以在 `<cstddef>` 中找到。

## 17.9 数学常数

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

[1] 数值问题通常是微妙的。如果你对数值问题的数学方面不是 100% 确定，要么寻求专家建议，要么进行实验，或两者兼做；§17.1。
[2] 不要试图仅使用裸语言进行严肃的数值计算；请使用库；§17.1。
[3] 在编写循环从序列计算值之前，考虑使用 `accumulate()`、`inner_product()`、`partial_sum()` 和 `adjacent_difference()`；§17.3。
[4] 对于大量数据，尝试并行和向量化算法；§17.3.1。
[5] 使用 `std::complex` 进行复数算术；§17.4。
[6] 将引擎与分布绑定以获得随机数生成器；§17.5。
[7] 注意你的随机数对于你的预期用途是否足够随机；§17.5。
[8] 不要使用 C 标准库的 `rand()`；对于实际用途，它的随机性不够；§17.5。
[9] 当运行时效率比对操作和元素类型的灵活性更重要时，使用 `valarray` 进行数值计算；§17.6。
[10] 数值类型的属性可通过 `numeric_limits` 访问；§17.7。
[11] 使用 `numeric_limits` 检查数值类型是否适用于其用途；§17.7。
[12] 如果你需要明确整数类型的大小，请使用别名；§17.8。