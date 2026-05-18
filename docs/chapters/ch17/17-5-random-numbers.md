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
