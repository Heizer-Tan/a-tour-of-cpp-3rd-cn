## 10.4 正则表达式

`std::regex` 提供了正则表达式匹配、搜索和替换功能。

### 10.4.1 基本匹配

```cpp
#include <regex>

string s = "The price is $12.99";
std::regex pattern {R"(\$\d+\.\d{2})"};  // 匹配 $xx.xx

if (std::regex_search(s, pattern)) {
    cout << "Found a price\n";
}
```

`R"(...)"` 是*原始字符串字面量*（raw string literal），其中的反斜杠不需要转义。

### 10.4.2 提取匹配

```cpp
std::smatch matches;
if (std::regex_search(s, matches, pattern)) {
    cout << "Found: " << matches[0] << '\n';  // "$12.99"
}
```

### 10.4.3 替换

```cpp
string result = std::regex_replace(s, pattern, "***");
// "The price is ***"
```

### 10.4.4 迭代匹配

```cpp
string text = "Prices: $1.99, $12.50, $100.00";
std::regex price_pattern {R"(\$(\d+)\.(\d{2}))"};
std::sregex_iterator it(text.begin(), text.end(), price_pattern);
std::sregex_iterator end;

for (; it != end; ++it) {
    cout << "Full: " << (*it)[0] << '\n';   // 完整匹配
    cout << "Dollars: " << (*it)[1] << '\n'; // 捕获组 1
    cout << "Cents: " << (*it)[2] << '\n';   // 捕获组 2
}
```

正则表达式功能强大，但编译和匹配可能代价较高。对于简单的字符串操作，优先使用 `string` 的成员函数（如 `find`、`substr`）。
