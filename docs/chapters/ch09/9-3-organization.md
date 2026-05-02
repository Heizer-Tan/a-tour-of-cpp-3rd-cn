## 9.3 标准库组织

标准库的头文件按功能组织：

| 头文件 | 功能 |
|--------|------|
| `<vector>` | 动态数组 |
| `<list>` | 双向链表 |
| `<map>` | 关联数组 |
| `<string>` | 字符串 |
| `<iostream>` | 流 I/O |
| `<algorithm>` | 通用算法 |
| `<numeric>` | 数值算法 |
| `<memory>` | 智能指针和内存管理 |
| `<thread>` | 线程 |
| `<chrono>` | 时间工具 |
| `<regex>` | 正则表达式 |
| `<random>` | 随机数 |
| `<filesystem>` | 文件系统操作（C++17） |
| `<format>` | 格式化输出（C++20） |
| `<ranges>` | 范围库（C++20） |
| `<concepts>` | 概念定义（C++20） |

### 命名空间

标准库中的所有组件都定义在 `std` 命名空间中。使用时需要加上 `std::` 前缀，或使用 `using` 声明：

```cpp
using std::vector;
using std::string;
using std::cout;
```

通常不建议在头文件中使用 `using namespace std;`，以避免名称污染。
