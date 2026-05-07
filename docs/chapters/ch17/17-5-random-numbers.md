
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