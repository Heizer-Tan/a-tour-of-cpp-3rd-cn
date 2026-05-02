## 11.4 文件 I/O

### 11.4.1 文件输出

```cpp
#include <fstream>

std::ofstream file {"output.txt"};
if (!file) {
    std::cerr << "Cannot open file\n";
    return 1;
}
file << "Hello, File!\n";
// 文件在 file 离开作用域时自动关闭
```

### 11.4.2 文件输入

```cpp
std::ifstream file {"input.txt"};
if (!file) {
    std::cerr << "Cannot open file\n";
    return 1;
}

string line;
while (std::getline(file, line)) {
    cout << line << '\n';
}
```

### 11.4.3 文件打开模式

```cpp
std::ofstream file {"data.txt", std::ios::app};  // 追加模式
std::ifstream file {"data.bin", std::ios::binary}; // 二进制模式
```

常用模式：
- `std::ios::in`：读取
- `std::ios::out`：写入
- `std::ios::app`：追加
- `std::ios::binary`：二进制模式
- `std::ios::ate`：打开后定位到文件末尾
