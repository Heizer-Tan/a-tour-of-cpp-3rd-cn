# 16.5 source_location

当输出跟踪消息或错误消息时，我们通常希望将源代码位置作为消息的一部分。库为此提供了 `source_location`：

```cpp
const source_location loc = source_location::current();
```

`current()` 返回一个描述源代码中出现位置的 `source_location`。类 `source_location` 有成员 `file_name()` 和 `function_name()`（返回 C 风格字符串），以及 `line()` 和 `column()`（返回无符号整数）。

将其包装在一个函数中，我们就得到了一个不错的日志记录消息的初步版本：

```cpp
void log(const string& mess = "", const source_location loc = source_location::current())
{
    cout << loc.file_name()
         << '(' << loc.line() << ':' << loc.column() << ") "
         << loc.function_name() << ": " << mess;
}
```

对 `current()` 的调用是一个默认参数，因此我们得到的是 `log()` 的调用者的位置，而不是 `log()` 本身的位置：

```cpp
void foo()
{
    log("Hello");   // myfile.cpp (17,4) foo: Hello
    // ...
}

int bar(const string& label)
{
    log(label);     // myfile.cpp (23,4) bar: <<the value of label>>
    // ...
}
```

===== 第 16 页 =====

在 C++20 之前编写的代码或需要在旧编译器上编译的代码使用宏 `__FILE__` 和 `__LINE__` 来实现此目的。

## 16.6 move() 与 forward()