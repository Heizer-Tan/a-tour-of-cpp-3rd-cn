## 14.2 范围算法

范围版本的算法直接接受范围（容器或视图）作为参数：

```cpp
#include <algorithm>
#include <ranges>

vector<int> v = {5, 3, 1, 4, 2};

std::ranges::sort(v);                               // 直接对容器排序
auto it = std::ranges::find(v, 3);                  // 查找元素
int n = std::ranges::count_if(v,
    [](int x) { return x % 2 == 0; });              // 计数偶数
```

与传统算法相比，范围算法：

- 不需要写 `.begin()` 和 `.end()`
- 返回更丰富的类型（如 `std::ranges::borrowed_iterator_t`）
- 通过概念约束参数，提供更好的错误消息
