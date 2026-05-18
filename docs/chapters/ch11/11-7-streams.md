# 11.7 流

标准库直接支持：

- **标准流**：附加到系统标准 I/O 流的流（[§11.7.1](#1171-标准流)）
- **文件流**：附加到文件的流（[§11.7.2](#1172-文件流)）
- **字符串流**：附加到字符串的流（[§11.7.3](#1173-字符串流)）
- **内存流**：附加到特定内存区域的流（[§11.7.4](#1174-内存流)）
- **同步流**：可以在没有数据竞争的情况下从多个线程使用的流（[§11.7.5](#1175-同步流)）

此外，我们可以定义自己的流，例如附加到通信通道的流。

流不能被拷贝；总是通过引用传递它们。

所有标准库流都是以字符类型为参数的模板。这里我使用的名称的版本接受 `char`。例如，`ostream` 是 `basic_ostream<char>`。对于每个这样的流，标准库还提供了 `wchar_t` 的版本。例如，`wostream` 是 `basic_ostream<wchar_t>`。宽字符流可用于 Unicode 字符。

## 11.7.1 标准流

标准流是：

- `cout`：用于“普通输出”
- `cerr`：用于无缓冲的“错误输出”
- `clog`：用于缓冲的“日志输出”
- `cin`：用于标准输入。

## 11.7.2 文件流

在 `<fstream>` 中，标准库提供了与文件之间的流：

- `ifstream`：用于从文件读取
- `ofstream`：用于写入文件
- `fstream`：用于读写文件。

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

有关文件名的组合和文件系统操作，请参见 [§11.9](11-9-file-system.md)。

## 11.7.3 字符串流

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

`ostringstream` 的内容可以使用 `str()`（内容的一个字符串副本）或 `view()`（内容的一个 `string_view`）读取。`ostringstream` 的一个常见用途是在将结果字符串交给 GUI 之前进行格式化。类似地，从 GUI 接收的字符串可以通过放入 `istringstream` 并使用格式化输入操作（[§11.3](11-3-input.md)）来读取。

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

函数模板参数只有在无法推导或没有默认值时才需要显式提及（[§8.2.4](../ch08/8-2-concepts.md#824-概念的定义)），因此我们可以写：

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

`ospanstream` 的行为类似于 `ostringstream`（[§11.7.3](#1173-字符串流)），其初始化方式也类似，只是 `ospanstream` 接受一个 `span` 而不是一个字符串作为参数。例如：

```cpp
void user(int arg)
{
    array<char, 128> buf;
    ospanstream ss(buf);
    ss << "write " << arg << " to memory\n";
    // ...
}
```

尝试溢出目标缓冲区会将流状态设置为失败（[§11.4](11-4-io-status.md)）。类似地，`ispanstream` 类似于 `istringstream`。

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

另一个线程可能会引入数据竞争（[§18.2](../ch18/18-2-tasks-and-threads.md)）并导致令人惊讶的输出。可以使用 `osyncstream` 来避免这种情况：

```cpp
void safer(int x, string& s)
{
    osyncstream oss(cout);
    oss << x;
    oss << s;
}
```

其他也使用 `osyncstream` 的线程不会干扰。另一个直接使用 `cout` 的线程可能会干扰，因此要么一致地使用 `osyncstream`，要么确保只有一个线程向特定的输出流产生输出。

并发可能很棘手，所以要小心（[第 18 章](../ch18/index.md)）。尽可能避免线程之间的数据共享。
