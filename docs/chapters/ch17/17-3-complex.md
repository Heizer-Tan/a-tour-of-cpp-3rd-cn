## 17.3 复数

`std::complex` 提供了复数运算：

```cpp
#include <complex>

std::complex<double> z1 {2, 3};     // 2 + 3i
std::complex<double> z2 {1, -1};    // 1 - i

auto sum = z1 + z2;                 // 3 + 2i
auto prod = z1 * z2;                // (2+3i)(1-i) = 5 + i
auto conj = std::conj(z1);          // 共轭: 2 - 3i
double mag = std::abs(z1);          // 模: sqrt(13)
double arg = std::arg(z1);          // 辐角

// 使用字面量（C++14）
using namespace std::complex_literals;
auto z3 = 2.0 + 3.0i;              // complex<double>{2, 3}
```
