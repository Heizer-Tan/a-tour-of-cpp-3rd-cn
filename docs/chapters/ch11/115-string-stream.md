## 11.5 字符串流

字符串流允许在内存中像操作文件一样操作字符串：

```cpp
#include <sstream>

std::ostringstream oss;
oss << "Name: " << "Alice" << ", Age: " << 30;
string result = oss.str();          // "Name: Alice, Age: 30"

std::istringstream iss {"42 3.14 hello"};
int i; double d; string s;
iss >> i >> d >> s;                 // i=42, d=3.14, s="hello"
```

字符串流在以下场景中非常有用：

- 构建复杂的字符串
- 解析格式化的字符串
- 类型转换
