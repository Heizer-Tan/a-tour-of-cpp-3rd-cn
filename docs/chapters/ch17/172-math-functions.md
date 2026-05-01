## 17.2 数学函数

`<cmath>` 提供了标准数学函数：

```cpp
#include <cmath>

double x = 2.0;
double y = std::sqrt(x);            // 平方根: 1.41421
double z = std::pow(x, 3);          // 幂: 8.0
double s = std::sin(3.14159 / 2);   // 正弦: ~1.0
double c = std::cos(0);             // 余弦: 1.0
double a = std::abs(-42);           // 绝对值: 42
double r = std::round(3.7);         // 四舍五入: 4.0
double f = std::floor(3.7);         // 向下取整: 3.0
double ce = std::ceil(3.2);         // 向上取整: 4.0
```

C++17 还引入了许多特殊数学函数：

```cpp
double b = std::beta(2.0, 3.0);     // Beta 函数
double g = std::tgamma(5.0);        // Gamma 函数 (4! = 24)
double e = std::erf(1.0);           // 误差函数
```
