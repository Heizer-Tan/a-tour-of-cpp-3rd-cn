# 16.5 `source_location`

写出跟踪信息或错误信息时，常常希望把源码位置一并写入消息。库为此提供了 `source_location`：

```cpp
const source_location loc = source_location::current();
```

`current()` 返回描述其在源码中出现位置的 `source_location`。类 `source_location` 提供成员 `file()` 与 `function_name()`，返回 C 风格字符串；`line()` 与 `column()` 返回无符号整数。

把它包进函数里，就得到日志消息的初步可用版本：

```cpp
void log(const string& mess = "", const source_location loc = source_location::current())
{
    cout << loc.file_name()
         << '(' << loc.line() << ':' << loc.column() << ") "
         << loc.function_name() << ": "
         << mess;
}

void foo()
{
    log("Hello");               // myfile.cpp(17,4) foo: Hello
    // ...
}

int bar(const string& label)
{
    log(label);                 // myfile.cpp(23,4) bar: <label 的值>
    // ...
}
```

对 `current()` 的调用位于默认实参中，于是我们得到的是 **`log()` 调用方** 的位置，而不是 `log()` 自身的位置。

在 C++20 之前编写、或需在旧编译器上编译的代码，对此通常使用宏 `__FILE__` 与 `__LINE__`。
