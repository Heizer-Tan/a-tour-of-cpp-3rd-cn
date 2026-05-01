## 10.2 字符串

`std::string` 是 C++ 中最常用的字符串类型。它管理一个动态分配的字符数组，支持丰富的操作。

### 10.2.1 基本操作

```cpp
string s = "Hello";
string t = "World";

string u = s + " " + t;     // 拼接: "Hello World"
size_t len = s.length();    // 长度: 5
char c = s[1];              // 索引: 'e'

if (s == t) { /* ... */ }   // 比较
if (s < t) { /* ... */ }    // 字典序比较
```

### 10.2.2 子串和搜索

```cpp
string s = "Hello, World!";
string sub = s.substr(7, 5);        // "World"
size_t pos = s.find("World");       // 7
size_t pos2 = s.find('o');          // 4 (第一个 'o')
size_t pos3 = s.rfind('o');         // 8 (最后一个 'o')
```

### 10.2.3 修改

```cpp
string s = "Hello";
s += " World";                      // 追加
s.insert(5, ",");                   // 插入: "Hello, World"
s.replace(7, 5, "C++");            // 替换: "Hello, C++"
s.erase(5, 2);                      // 删除: "HelloC++"
```

### 10.2.4 数值转换

```cpp
string s = "123.45";
double d = std::stod(s);            // 字符串转 double
int i = std::stoi("42");            // 字符串转 int

string s2 = std::to_string(3.14);   // double 转字符串
string s3 = std::to_string(42);     // int 转字符串
```
