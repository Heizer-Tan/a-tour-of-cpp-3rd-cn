## 11.3 输入

### 11.3.1 基本输入

```cpp
int i;
double d;
string s;

cin >> i;       // 读取整数
cin >> d;       // 读取浮点数
cin >> s;       // 读取单词（以空白字符分隔）
```

`>>` 运算符默认跳过前导空白字符，读取到下一个空白字符为止。

### 11.3.2 读取整行

```cpp
string line;
while (std::getline(cin, line)) {
    if (line.empty()) break;
    cout << "Read: " << line << '\n';
}
```

`std::getline` 读取一整行（包括空格），直到遇到换行符。

### 11.3.3 输入状态

流会跟踪其状态，可以通过以下方法检查：

```cpp
if (cin) { /* 流处于良好状态 */ }
if (cin.good()) { /* 没有错误 */ }
if (cin.eof()) { /* 到达文件末尾 */ }
if (cin.fail()) { /* 读取操作失败 */ }
if (cin.bad()) { /* 严重错误 */ }
```

读取循环的惯用写法：

```cpp
for (int i; cin >> i; ) {
    cout << "Read: " << i << '\n';
}
```

当读取失败或到达文件末尾时，循环自动终止。
