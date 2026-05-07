# 11.5 用户定义类型的 I/O

除了内置类型和标准字符串的 I/O，`iostream` 库还允许我们为自己的类型定义 I/O。例如，考虑一个简单的类型 `Entry`，我们可能用它来表示电话簿中的条目：

```cpp
struct Entry {
    string name;
    int number;
};
```

我们可以定义一个简单的输出运算符，使用类似于我们在代码中用于初始化的 `{"name",number}` 格式来写入 `Entry`：

```cpp
ostream& operator<<(ostream& os, const Entry& e)
{
    return os << "{\"" << e.name << "\", " << e.number << "}";
}
```

用户定义的输出运算符将其输出流（通过引用）作为第一个参数，并将其作为结果返回。

相应的输入运算符更复杂，因为它必须检查正确的格式并处理错误：

```cpp
istream& operator>>(istream& is, Entry& e)
// 读取 {"name", number} 对。注意：格式为 { " " , 和 }
{
    char c, c2;
    if (is >> c && c == '{' && is >> c2 && c2 == '"') {   // 以 { 后跟 " 开头
        string name;   // 字符串的默认值是空字符串
        while (is.get(c) && c != '"')   // 在 " 之前的任何字符都是名字的一部分
            name += c;

        if (is >> c && c == ',') {
            int number = 0;
            if (is >> number >> c && c == '}') {   // 读取数字和一个 }
                e = {name, number};               // 赋值给条目
                return is;
            }
        }
    }
    is.setstate(ios_base::failbit);   // 在流中注册失败
    return is;
}
```

输入操作返回对其 `istream` 的引用，可用于测试操作是否成功。例如，当用作条件时，`is >> c` 的意思是“我们成功地从 `is` 读取一个字符到 `c` 了吗？”

`is >> c` 默认会跳过空白，但 `is.get(c)` 不会，因此这个 `Entry` 输入运算符会忽略（跳过）名字字符串外部的空白，但不会忽略内部的空白。例如：

```cpp
{ "John Marwood Cleese", 123456 }
{"Michael Edward Palin", 987654}
```

我们可以像这样从输入中将这样一对值读入 `Entry`：

```cpp
for (Entry ee; cin >> ee; )   // 从 cin 读入 ee
    cout << ee << '\n';       // 将 ee 写入 cout
```

输出是：
```cpp
{"John Marwood Cleese", 123456}
{"Michael Edward Palin", 987654}
```

有关在字符流中识别模式的更系统技术（正则表达式匹配），请参见 §10.4。
