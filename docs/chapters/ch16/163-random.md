## 16.3 随机数

`std::random` 库提供了比 `rand()` 更好的随机数生成。

### 16.3.1 随机数引擎和分布

```cpp
#include <random>

std::random_device rd;                      // 真随机数源（用于种子）
std::mt19937 gen(rd());                     // 梅森旋转引擎

std::uniform_int_distribution<int> dist(1, 6);  // 均匀分布 [1, 6]
int dice = dist(gen);                       // 掷骰子

std::normal_distribution<double> ndist(0.0, 1.0); // 正态分布 (均值 0, 标准差 1)
double val = ndist(gen);                    // 正态分布随机数
```

### 16.3.2 常用分布

```cpp
std::uniform_real_distribution<double> udist(0.0, 1.0);  // [0, 1) 均匀分布
std::bernoulli_distribution bdist(0.7);                   // 70% true
std::exponential_distribution<double> edist(1.0);         // 指数分布
```
