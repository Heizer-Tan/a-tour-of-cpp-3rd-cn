# 11.6.2 printf() 风格的格式化

有可靠论证说 `printf()` 是 C 中最流行的函数，是其成功的重要因素。例如：
```cpp
printf("an int %g and a string '%s'\n", 123, "Hello!");
```

这种“格式字符串后跟参数”的风格被 BCPL 采纳到 C 中，并被许多语言效仿。自然地，`printf()` 一直是 C++ 标准库的一部分，但它缺乏类型安全性和处理用户定义类型的可扩展性。

在 `<format>` 中，标准库提供了一个类型安全（尽管不可扩展）的 `printf()` 风格格式化机制。基本函数 `format()` 产生一个字符串：

```cpp
string s = format("Hello, {}\n", val);
```

格式字符串中的“普通字符”只是被放入输出字符串中。由 `{` 和 `}` 分隔的格式指定了格式字符串后面的参数如何被插入到字符串中。最简单的格式字符串是空字符串 `{}`，它从参数列表中取出下一个参数并根据其 `<<` 默认（如果有）打印它。因此，如果 `val` 是 `"World"`，我们得到标志性的 `"Hello, World\n"`。如果 `val` 是 `127`，我们得到 `"Hello, 127\n"`。

`format()` 最常见的用途是输出其结果：

```cpp
cout << format("Hello, {}\n", val);
```

为了看到这是如何工作的，让我们首先重复 §11.6.1 中的例子：

```cpp
cout << format("{} {:x} {:o} {:d} {:b}\n", 1234, 1234, 1234, 1234, 1234);
```

这给出了与 §11.6.1 中的整数示例相同的输出，除了我添加了二进制 `b`（`ostream` 不直接支持）：
```
1234 4d2 2322 1234 10011010010
```

格式化指令前面有一个冒号。整数格式化选项是：`x` 表示十六进制，`o` 表示八进制，`d` 表示十进制，`b` 表示二进制。

默认情况下，`format()` 按顺序获取其参数。然而，我们可以指定任意顺序。例如：

```cpp
cout << format("{3:} {1:x} {2:o} {0:b}\n", 000, 111, 222, 333);
```
打印 `333 6f 336 0`。冒号前的数字是要格式化的参数的编号。以最佳的 C++ 风格，编号从零开始。这允许我们多次格式化同一个参数：

```cpp
cout << format("{0:} {0:x} {0:o} {0:d} {0:b}\n", 1234);
```

浮点格式与 `ostream` 相同：`e` 表示科学计数法，`a` 表示十六进制浮点，`f` 表示定点，`g` 表示默认。例如：

```cpp
cout << format("{0:}; {0:e}; {0:a}; {0:f}; {0:g}\n", 123.456);
```

结果与 `ostream` 相同，除了十六进制数字前没有 `0x`：
```
123.456; 1.234560e+002; 1.edd2f2p+6; 123.456000; 123.456
```

点号后跟精度说明符：

```cpp
cout << format("precision(8): {:.8} {} {}\n", 1234.56789, 1234.56789, 123456);
cout << format("precision(4): {:.4} {} {}\n", 1234.56789, 1234.56789, 123456);
cout << format("{}\n", 1234.56789);
```

与流不同，说明符不是“粘性的”，因此我们得到：
```
precision(8): 1234.5679 1234.56789 123456
precision(4): 1235 1234.56789 123456
1234.56789
```

与流格式化器一样，`format()` 也可以处理时间和日期（§16.2.2）。例如：

```cpp
cout << format("birthday: {}\n", November/28/2021);
cout << format("zt: {}", zoned_time{current_zone(), system_clock::now()});
```

像往常一样，值的默认格式化与默认流输出格式化相同。然而，`format()` 提供了大约 60 个格式说明符的微型语言，允许对数字和日期的格式化进行非常详细的控制。例如：

```cpp
auto ymd = 2021y/March/30;
cout << format("ymd: {3:%A}, {1:} {2:%B}, {0:}\n", ymd.year(), ymd.month(), ymd.day(), ymd);
```

这产生了：
```
ymd: Tuesday, March 30, 2021
```

所有时间和日期格式字符串都以 `%` 开头。

众多格式说明符提供的灵活性可能很重要，但也伴随着许多出错的机会。有些说明符带有可选或依赖于本地环境的语义。如果在运行时捕获到格式化错误，则会抛出 `format_error` 异常。例如：

```cpp
string ss = format("{:%F}", 2);   // 错误：参数不匹配；可能被捕获
string sss = format("{%F}", 2);   // 错误：格式错误；可能在编译时被捕获
```

到目前为止的例子都有可以在编译时检查的常量格式。对应的函数 `vformat()` 接受一个变量作为格式，以显著增加灵活性和运行时错误的机会：

```cpp
string fmt = "{}";
cout << vformat(fmt, make_format_args(2));   // OK
fmt = "{:%F}";
cout << vformat(fmt, make_format_args(2));   // 错误：格式与参数不匹配
```

最后，格式化器也可以直接写入由迭代器定义的缓冲区。例如：

```cpp
string buf;
format_to(back_inserter(buf), "iterator: {} {}\n", "Hi! ", 2022);
cout << buf;   // iterator: Hi! 2022
```

如果我们直接使用流的缓冲区或其他输出设备的缓冲区，这对于性能来说就变得有趣了。

## 11.7 流

标准库直接支持：

- **标准流**：附加到系统标准 I/O 流的流（§11.7.1）
- **文件流**：附加到文件的流（§11.7.2）
- **字符串流**：附加到字符串的流（§11.7.3）
- **内存流**：附加到特定内存区域的流（§11.7.4）
- **同步流**：可以在没有数据竞争的情况下从多个线程使用的流（§11.7.5）

此外，我们可以定义自己的流，例如附加到通信通道的流。

流不能被拷贝；总是通过引用传递它们。

所有标准库流都是以字符类型为参数的模板。这里我使用的名称的版本接受 `char`。例如，`ostream` 是 `basic_ostream<char>`。对于每个这样的流，标准库还提供了 `wchar_t` 的版本。例如，`wostream` 是 `basic_ostream<wchar_t>`。宽字符流可用于 unicode 字符。

## 11.7.1 标准流

===== 第 16 页 =====

标准流是：

- `cout`：用于“普通输出”
- `cerr`：用于无缓冲的“错误输出”
- `clog`：用于缓冲的“日志输出”
- `cin`：用于标准输入。

#### 11.7.2 文件流

在 `<fstream>` 中，标准库提供了与文件之间的流：

- `ifstream`：用于从文件读取
- `ofstream`：用于写入文件
- `fstream`：用于读写文件

例如：

```cpp
ofstream ofs {"target"};   // “o” 代表 “output”
if (!ofs)
    error("couldn't open 'target' for writing");

ifstream ifs {"source"};   // “i” 代表 “input”
if (!ifs)
    error("couldn't open 'source' for reading");
```

假设测试成功，`ofs` 可以用作普通的 `ostream`（就像 `cout` 一样），`ifs` 可以用作普通的 `istream`（就像 `cin` 一样）。

文件定位和更详细的文件打开控制是可能的，但超出了本书的范围。

有关文件名的组合和文件系统操作，请参见 §11.9。

## 11.7.3 字符串流

===== 第 17 页 =====

在 `<sstream>` 中，标准库提供了与字符串之间的流：

- `istringstream`：用于从字符串读取
- `ostringstream`：用于写入字符串
- `stringstream`：用于读写字符串。

例如：

```cpp
void test()
{
    ostringstream oss;

    oss << "{temperature," << scientific << 123.4567890 << "}";
    cout << oss.view() << '\n';
}
```

`ostringstream` 的内容可以使用 `str()`（内容的一个字符串副本）或 `view()`（内容的一个 `string_view`）读取。`ostringstream` 的一个常见用途是在将结果字符串交给 GUI 之前进行格式化。类似地，从 GUI 接收的字符串可以通过放入 `istringstream` 并使用格式化输入操作（§11.3）来读取。

`stringstream` 可以同时用于读取和写入。例如，我们可以定义一个操作，将任何具有字符串表示的类型转换为另一种也可以表示为字符串的类型：

```cpp
template<typename Target = string, typename Source = string>
Target to(Source arg)   // 将 Source 转换为 Target
{
    stringstream buf;
    Target result;

    if (!(buf << arg)          // 将 arg 写入流
        || !(buf >> result)    // 从流中读取结果
        || !(buf >> std::ws).eof())   // 流中是否还有东西？
        throw runtime_error{"to<>() failed"};

    return result;
}
```

函数模板参数只有在无法推导或没有默认值时才需要显式提及（§8.2.4），因此我们可以写：

```cpp
auto x1 = to<string, double>(1.2);   // 非常显式（冗长）
auto x2 = to<string>(1.2);           // Source 推导为 double
auto x3 = to<>(1.2);                 // Target 默认为 string；Source 推导为 double
auto x4 = to(1.2);                   // <> 是多余的；Target 默认为 string；Source 推导为 double
```

如果所有函数模板参数都有默认值，则可以省略 `<>`。

我认为这是一个很好的例子，展示了通过语言特性和标准库设施的组合可以达到的通用性和易用性。

## 11.7.4 内存流

从 C++ 的早期开始，就有附加到用户指定内存区域的流，以便我们可以直接读取/写入它们。最古老的此类流 `strstream` 已被弃用数十年，但它们的替代品 `spanstream`、`ispanstream` 和 `ospanstream` 要到 C++23 才会正式成为标准。然而，它们已经广泛可用；可以尝试你的实现或在 GitHub 上搜索。

`ospanstream` 的行为类似于 `ostringstream`（§11.7.3），其初始化方式也类似，只是 `ospanstream` 接受一个 `span` 而不是一个字符串作为参数。例如：

```cpp
void user(int arg)
{
    array<char, 128> buf;
    ospanstream ss(buf);
    ss << "write " << arg << " to memory\n";
    // ...
}
```

尝试溢出目标缓冲区会将流状态设置为失败（§11.4）。类似地，`ispanstream` 类似于 `istringstream`。

## 11.7.5 同步流

在多线程系统中，除非满足以下条件之一，否则 I/O 会变得不可靠：

- 只有一个线程使用该流。
- 对流的访问是同步的，以便一次只有一个线程获得访问。

`osyncstream` 保证即使其他线程试图写入，一系列输出操作也将完成，并且其结果在输出缓冲区中符合预期。例如：

```cpp
void unsafe(int x, string& s)
{
    cout << x;
    cout << s;
}
```

另一个线程可能会引入数据竞争（§18.2）并导致令人惊讶的输出。可以使用 `osyncstream` 来避免这种情况：

```cpp
void safer(int x, string& s)
{
    osyncstream oss(cout);
    oss << x;
    oss << s;
}
```

其他也使用 `osyncstream` 的线程不会干扰。另一个直接使用 `cout` 的线程可能会干扰，因此要么一致地使用 `osyncstream`，要么确保只有一个线程向特定的输出流产生输出。

并发可能很棘手，所以要小心（第 18 章）。尽可能避免线程之间的数据共享。
