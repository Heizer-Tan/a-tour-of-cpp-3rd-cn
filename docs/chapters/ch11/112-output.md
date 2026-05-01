## 11.2 输出

### 11.2.1 基本输出

```cpp
cout << "Hello, World!\n";
cout << "Value: " << 42 << ", Pi: " << 3.14159 << '\n';
```

`<<` 运算符将数据写入输出流。它可以链式调用，因为 `<<` 返回流的引用。

### 11.2.2 格式化输出

可以使用 I/O 操纵器（manipulators）来控制输出格式：

```cpp
#include <iomanip>

cout << std::fixed << std::setprecision(2);  // 固定小数点，2 位精度
cout << 3.14159 << '\n';                      // 输出: 3.14

cout << std::hex << 255 << '\n';              // 输出: ff (十六进制)
cout << std::oct << 255 << '\n';              // 输出: 377 (八进制)
cout << std::dec << 255 << '\n';              // 输出: 255 (十进制)

cout << std::setw(10) << std::left << "Hello" << '\n';  // 左对齐，宽度 10
```

### 11.2.3 `std::format`（C++20）

`std::format` 提供了一种更安全、更直观的格式化方式：

```cpp
#include <format>

string s = std::format("Hello, {}!", "World");          // "Hello, World!"
cout << std::format("Pi is approximately {:.2f}", 3.14159);  // "Pi is approximately 3.14"
cout << std::format("{:>10}", "right");                  // 右对齐，宽度 10
cout << std::format("{:#x}", 255);                       // "0xff" (带前缀的十六进制)
```

`std::format` 的优势：

- 类型安全：格式说明符与参数类型在编译时检查
- 可读性：格式字符串直观易懂
- 性能：通常比 `iostream` 更快
