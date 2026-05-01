## 14.3 视图

视图（view）是范围库最强大的特性之一。视图是范围的轻量级包装器，它不拥有数据，而是提供一种新的"观察"方式。视图是惰性求值的——操作只在需要时才执行。

### 14.3.1 常用视图

```cpp
#include <ranges>
namespace views = std::views;

vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

// filter: 过滤元素
auto evens = v | views::filter([](int x) { return x % 2 == 0; });
// evens 惰性地产生: 2, 4, 6, 8, 10

// transform: 变换元素
auto squares = v | views::transform([](int x) { return x * x; });
// squares 惰性地产生: 1, 4, 9, 16, 25, ...

// take: 取前 n 个
auto first5 = v | views::take(5);
// first5: 1, 2, 3, 4, 5

// drop: 跳过前 n 个
auto after5 = v | views::drop(5);
// after5: 6, 7, 8, 9, 10

// reverse: 反转
auto reversed = v | views::reverse;
// reversed: 10, 9, 8, 7, 6, 5, 4, 3, 2, 1
```

### 14.3.2 组合视图

视图可以通过管道运算符 `|` 组合：

```cpp
auto result = v
    | views::filter([](int x) { return x % 2 == 0; })  // 取偶数
    | views::transform([](int x) { return x * x; })     // 平方
    | views::take(3);                                    // 取前 3 个

for (int x : result)
    cout << x << ' ';  // 输出: 4 16 36
```

这种管道语法使得数据处理流水线清晰易读，且由于惰性求值，性能与手写循环相当。

### 14.3.3 生成视图

```cpp
// iota: 生成序列
for (int i : views::iota(1, 6))
    cout << i << ' ';  // 1 2 3 4 5

// iota 无上限
auto infinite = views::iota(0)
    | views::transform([](int x) { return x * x; })
    | views::take(5);
// 0, 1, 4, 9, 16
```
