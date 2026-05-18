# 数值

> **计算的目的是洞察，而非数字；但对学生而言，数字往往是通往洞察的最佳途径。**
>
> —— R. W. Hamming / A. Ralston

# 17.1 引言

C++ 在设计之初并非主要针对数值计算。然而数值计算通常发生在其它工作之中——例如科学计算、数据库访问、网络、仪器控制、图形、仿真与金融分析——于是 C++ 成为更大系统中组成部分的计算任务的诱人载体。再者，数值方法早已不再是简单地对浮点向量做循环；当计算需要更复杂的数据结构时，C++ 的长处便显现出来。其净效果是：C++ 广泛用于科学、工程、金融以及涉及复杂数值的其它计算；支持此类计算的设施与技术也随之出现。本章描述标准库中支持数值计算的各个部分。

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

# 17.4 复数

标准库支持一族复数类型，其思路与 §5.2.1 描述的 `complex` 类一脉相承。为了支持标量为单精度浮点数（`float`）、双精度浮点数（`double`）等情形的复数，标准库的 `complex` 是一个模板：

```cpp
template<typename Scalar>
class complex {
public:
    complex(const Scalar& re = {}, const Scalar& im = {});
    // ...
};
```

普通的算术运算与最常见的数学函数对复数均有支持。例如：

```cpp
void f(complex<float> fl, complex<double> db)
{
    complex<long double> ld{fl + sqrt(db)};
    db += fl * 3;
    fl = pow(1 / fl, 2);
    // ...
}
```

`<complex>` 中也定义了常见的数学函数，`sqrt()` 与 `pow()`（幂运算）即在其中（§17.2）。

# 17.5 随机数

随机数在许多语境下有用：测试、游戏、仿真、安全等。应用领域的高度多样性，反映在标准库 `<random>` 所提供的随机数发生器种类之丰富上。随机数发生器由两部分组成：

1. **引擎**：产生一串随机或伪随机值；
2. **分布**：把这些值映射到某个区间上的数学分布。

分布的例子包括：`uniform_int_distribution`（生成的整数等概率）、`normal_distribution`（「钟形曲线」）、以及 `exponential_distribution`（指数衰减）；各自对应某个指定区间。

例如：

```cpp
using my_engine = default_random_engine;
using my_distribution = uniform_int_distribution<>;

my_engine eng{};
my_distribution dist{1, 6};
auto die = [&]() { return dist(eng); };   // 构造一个发生器

int x = die();                              // 掷骰子：x 成为某个值
```

多亏它对泛化与性能的不妥协，曾有专家评价标准库的随机数组件是「每个随机数库长大后想成为的样子」。但它也很难被称为「新手友好」。`using` 与 lambda 能让正在发生的事稍微直白一点。

对新手（不论背景）而言，随机数库的完全通用接口可能成为严重障碍。一个简单的均匀随机数发生器往往足以入门。例如：

```cpp
Rand_int rnd{1, 10};    // 构造 [1:10] 上的均匀随机整数发生器
int x = rnd();          // x 取自 [1:10]
```

怎么才能做到？我们需要某种像 `die()` 一样把引擎与分布封装在类里的东西，例如 `Rand_int`：

```cpp
class Rand_int {
public:
    Rand_int(int low, int high) : dist{low, high} {}
    int operator()() { return dist(re); }    // 抽取一个 int
    void seed(int s) { re.seed(s); }          // 设置新的引擎种子
private:
    default_random_engine re;
    uniform_int_distribution<> dist;
};
```

这一定义仍旧偏「专家级」，但 `Rand_int()` 的用法在面向新手的 C++ 课程第一周还算可控。例如：

```cpp
int main()
{
    constexpr int max = 9;
    Rand_int rnd{0, max};

    vector<int> histogram(max + 1);
    for (int i = 0; i != 200; ++i)
        ++histogram[rnd()];

    for (int i = 0; i != histogram.size(); ++i) {
        cout << i << '\t';
        for (int j = 0; j != histogram[i]; ++j)
            cout << '*';
        cout << '\n';
    }
}
```

输出是（统计上略有起伏、表面看来单调乏味的）均匀分布：

```
0      *********************
1      ****************
...
```

C++ 没有标准图形库，所以我用「ASCII 图形」。显然有大量开源与商业图形/GUI 库可用，但本书只使用 ISO 标准设施。

若要得到重复或可区分的序列，我们为引擎**播种**——让其内部状态取新值。例如：

```cpp
Rand_int rnd{10, 20};
for (int i = 0; i < 10; ++i)
    cout << rnd() << ' ';
cout << '\n';

rnd.seed(999);
for (int i = 0; i < 10; ++i)
    cout << rnd() << ' ';
cout << '\n';

rnd.seed(999);
for (int i = 0; i < 10; ++i)
    cout << rnd() << ' ';
cout << '\n';
```

重复序列对可重复的调试很重要；用不同值播种则在不想重复时很重要。若你需要真正的随机数而非生成的伪随机序列，请查看你的机器上 `random_device` 如何实现。

# 17.6 向量算术

§12.2 描述的 `vector` 旨在成为容纳值的一般机制，灵活并嵌入容器、迭代器与算法的体系之中。然而它并不支持数学意义上的向量运算。把此类运算加到 `vector` 上并不容易违背其通用性；而其通用与灵活又排除了数值工作中常常认为至关重要的优化。因此，标准库在 `<valarray>` 中提供了另一种更像数组的模板，名为 `valarray`：它不那么通用，却更容易为数值计算优化：

```cpp
template<typename T>
class valarray {
    // ...
};
```

通常的算术运算与最常见的数学函数对 `valarray` 均有支持。例如：

```cpp
void f(valarray<double>& a1, valarray<double>& a2)
{
    valarray<double> a = a1 * 3.14 + a2 / a1;   // 逐元素 *, +, /
    a2 += a1 * 3.14;

    a = abs(a);
    double d = a2[7];
    // ...
}
```

这些运算是**向量运算**；也就是说，它们施加到所涉及向量的每个元素上。

除算术运算外，`valarray` 还提供按步幅访问，以帮助实现多维计算。

# 17.7 数值极限

在 `<limits>` 中，标准库提供了描述内建类型性质的类——例如 `float` 的最大指数，或 `int` 的字节数。例如，我们可以断言 `char` 是有符号的：

```cpp
static_assert(numeric_limits<char>::is_signed, "unsigned characters!");
static_assert(100000 < numeric_limits<int>::max(), "small ints!");
```

第二个断言之所以可行，是因为 `numeric_limits<int>::max()` 是 `constexpr` 函数（§1.6）。

我们也可以为自己的用户定义类型定义 `numeric_limits`。

# 17.8 类型别名

`int`、`long long` 等基本类型的大小由实现定义；也就是说，不同 C++ 实现上可能不同。若我们需要明确整数宽度，可以使用 `<cstdint>` 中定义的别名，例如 `int32_t` 与 `uint_least64_t`。后者表示「至少 64 位的无符号整数」。

古怪的 `_t` 后缀是 C 时代的遗迹：那时人们认为名字应当表明它是别名。

其它常见别名，例如 `size_t`（`sizeof` 运算符结果的类型）与 `ptrdiff_t`（两指针相减结果的类型），见 `<cstddef>`。

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

# 17.10 建议

[1] 数值问题往往很微妙。若你对某个数值问题的数学方面并非百分之百确定，要么请教专家，要么做实验，或两者并行；§17.1。

[2] 不要试图只用裸语言做严肃的数值计算；请使用库；§17.1。

[3] 在动手写循环、从一个序列计算某个值之前，先考虑 `accumulate()`、`inner_product()`、`partial_sum()` 和 `adjacent_difference()`；§17.3。

[4] 对更大数据量，尝试并行与向量化算法；§17.3.1。

[5] 用 `std::complex` 做复数算术；§17.4。

[6] 把引擎与分布绑在一起，得到随机数发生器；§17.5。

[7] 当心：对你的用途而言，随机数是否足够随机；§17.5。

[8] 不要使用 C 标准库的 `rand()`；它对真实用途而言随机性不足；§17.5。

[9] 当运行期效率比操作与元素类型方面的灵活性更重要时，对数值计算使用 `valarray`；§17.6。

[10] 数值类型的性质可通过 `numeric_limits` 取得；§17.7。

[11] 用 `numeric_limits` 检查数值类型是否足以胜任其用途；§17.7。

[12] 若想明确整数的大小，使用整数类型别名；§17.8。
