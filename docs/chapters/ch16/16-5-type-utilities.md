## 16.5 类型工具

### 16.5.1 类型特征（Type Traits）

`<type_traits>` 提供了编译时类型查询和变换：

```cpp
#include <type_traits>

static_assert(std::is_integral_v<int>);             // int 是整数类型
static_assert(!std::is_integral_v<double>);         // double 不是
static_assert(std::is_same_v<int, signed int>);     // int 和 signed int 相同

using U = std::remove_const_t<const int>;           // U 是 int
using V = std::add_pointer_t<int>;                  // V 是 int*
```

### 16.5.2 `std::pair` 和 `std::tuple`

```cpp
std::pair<string, int> p {"Alice", 30};
cout << p.first << ": " << p.second << '\n';

auto [name, age] = p;                               // C++17 结构化绑定

std::tuple<int, string, double> t {42, "hello", 3.14};
auto [i, s, d] = t;                                 // 结构化绑定
```

### 16.5.3 `std::move` 和 `std::forward`

```cpp
template<typename T>
void wrapper(T&& arg) {
    // std::forward 保持 arg 的值类别
    process(std::forward<T>(arg));
}
```

`std::move` 将左值转换为右值引用，`std::forward` 用于完美转发（perfect forwarding）。
