## 17.4 数值数组

`std::valarray` 是为数值计算优化的数组类型，支持逐元素运算：

```cpp
#include <valarray>

std::valarray<double> a = {1, 2, 3, 4};
std::valarray<double> b = {5, 6, 7, 8};

auto sum = a + b;                   // {6, 8, 10, 12}
auto prod = a * b;                  // {5, 12, 21, 32}
auto scaled = a * 2.0;              // {2, 4, 6, 8}

double total = a.sum();             // 10
double avg = a.sum() / a.size();    // 2.5
auto sq = std::sqrt(a);             // 逐元素平方根

// 切片操作
std::slice s(0, 2, 2);             // 从 0 开始，2 个元素，步长 2
auto sub = a[s];                    // {1, 3}
```

`valarray` 的设计目标是支持向量化操作，适合科学计算场景。不过，对于大多数通用编程任务，`std::vector` 是更好的选择。
