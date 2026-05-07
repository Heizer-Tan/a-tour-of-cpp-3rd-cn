===== 第 1 页 =====

11

# 输入与输出

所见即所得。 – Brian W. Kernighan

- 引言
- 输出
- 输入
- I/O 状态
- 用户定义类型的 I/O
- 格式化
  - 流格式化；printf() 风格的格式化
- 流
  - 标准流；文件流；字符串流；内存流；同步流
- C 风格 I/O

===== 第 2 页 =====

 - 文件系统
  - 路径；文件与目录
- 建议

## 11.1 引言

I/O 流库提供文本和数值的格式化和非格式化缓冲 I/O。它是可扩展的，能够像支持内置类型一样支持用户定义类型，并且是类型安全的。

文件系统库提供了操作文件和目录的基本设施。

`ostream` 将类型化对象转换为字符（字节）流：

[图片描述]

`istream` 将字符（字节）流转换为类型化对象：

[图片描述]

`istream` 和 `ostream` 的操作在 §11.2 和 §11.3 中描述。这些操作是类型安全、类型敏感且可扩展的，能够处理用户定义类型（§11.5）。

===== 第 3 页 =====

其他形式的用户交互，例如图形 I/O，通过不属于 ISO 标准的库来处理，因此这里不描述。

这些流可用于二进制 I/O，可用于多种字符类型，可针对特定本地环境，并使用高级缓冲策略，但这些主题超出了本书的范围。

流可用于字符串的输入和输出（§11.3）、格式化到字符串缓冲区（§11.7.3）、格式化到内存区域（§11.7.4）以及文件 I/O（§11.9）。

所有 I/O 流类都有析构函数，用于释放所拥有的所有资源（例如缓冲区和文件句柄）。也就是说，它们是“资源获取即初始化”（RAII；§6.3）的实例。

## 11.2 输出

在 `<ostream>` 中，I/O 流库为每个内置类型定义了输出。此外，定义用户定义类型的输出也很容易（§11.5）。运算符 `<<`（“放入”）用作 `ostream` 类型对象上的输出运算符；`cout` 是标准输出流，`cerr` 是用于报告错误的标准流。默认情况下，写入 `cout` 的值被转换为字符序列。例如，要输出十进制数 10，我们可以写：

```cpp
cout << 10;
```

这会将字符 `1` 后跟字符 `0` 放入标准输出流。

等价地，我们可以写：

```cpp
int x {10};
cout << x;
```

不同类型的输出可以按显而易见的方式组合：

```cpp
void h(int i)
{
    cout << "the value of i is ";
    cout << i;
    cout << '\n';
}
```

===== 第 4 页 =====

对于 `h(10)`，输出将是：
```
the value of i is 10
```

当输出几个相关的项时，人们很快就会厌倦重复输出流的名称。幸运的是，输出表达式的结果本身可以用于进一步的输出。例如：

```cpp
void h2(int i)
{
    cout << "the value of i is " << i << '\n';
}
```

这个 `h2()` 产生与 `h()` 相同的输出。

字符常量是用单引号括起来的字符。注意，字符是作为字符输出，而不是作为数值输出。例如：

```cpp
int b = 'b';   // 注意：char 隐式转换为 int
char c = 'c';
cout << 'a' << b << c;
```

字符 `'b'` 的整数值是 98（在我使用的 C++ 实现的 ASCII 编码中），因此这将输出 `a98c`。

## 11.3 输入

在 `<istream>` 中，标准库提供了用于输入的 `istream`。与 `ostream` 一样，`istream` 处理内置类型的字符串表示，并且可以轻松扩展以处理用户定义类型。

运算符 `>>`（“从...获取”）用作输入运算符；`cin` 是标准输入流。`>>` 的右操作数的类型决定了接受什么输入以及输入操作的目标是什么。例如：

```cpp
int i;
cin >> i;      // 将整数读入 i

double d;
cin >> d;      // 将双精度浮点数读入 d
```

这从标准输入读取一个数字（例如 1234）到整数变量 `i`，并读取一个浮点数（例如 12.34e5）到双精度浮点变量 `d`。

与输出操作一样，输入操作可以链式连接，因此我可以等价地写：

```cpp
int i;
double d;
cin >> i >> d;   // 读入 i 和 d
```

在这两种情况下，整数的读取由任何非数字字符终止。默认情况下，`>>` 会跳过开头的空白，因此一个合适的完整输入序列可以是：
```
1234 12.34e5
```

通常，我们希望读取一个字符序列。一种方便的方法是读入一个 `string`。例如：

```cpp
cout << "Please enter your name\n";
string str;
cin >> str;
cout << "Hello, " << str << "!\n";
```

如果你输入 `Eric`，响应是：
```
Hello, Eric!
```

默认情况下，空白字符（如空格或换行符）会终止读取，因此如果你输入 `Eric Bloodaxe`（假扮那位不幸的约克国王），响应仍然是：
```
Hello, Eric!
```

你可以使用 `getline()` 函数读取整行。例如：

===== 第 6 页 =====

```cpp
cout << "Please enter your name\n";
string str;
getline(cin, str);
cout << "Hello, " << str << "!\n";
```

使用这个程序，输入 `Eric Bloodaxe` 会产生期望的输出：
```
Hello, Eric Bloodaxe!
```

终止该行的换行符被丢弃，因此 `cin` 已为下一行输入做好准备。

使用格式化 I/O 操作通常比逐个操作字符更不容易出错、更高效且代码更少。特别是，`istream` 负责内存管理和范围检查。我们可以使用 `stringstream`（§11.7.3）或内存流（§11.7.4）在内存中进行格式化。

标准字符串具有很好的属性：它们会扩展以容纳放入其中的内容；你不必预先计算最大大小。因此，如果你输入几兆字节的分号，`hello_line()` 会回显给你很多页的分号。

## 11.4 I/O 状态

`iostream` 有一个状态，我们可以检查它以确定操作是否成功。最常见的用法是读取一个值序列：

```cpp
vector<int> read_ints(istream& is)
{
    vector<int> res;
    for (int i; is >> i; )
        res.push_back(i);
    return res;
}
```

这从 `is` 读取，直到遇到不是整数的东西。那个东西通常是输入的结尾。这里发生的是：操作 `is >> i` 返回对 `is` 的引用，而测试 `iostream` 在流准备好进行下一个操作时返回 `true`。

===== 第 7 页 =====

一般来说，I/O 状态保存了读取或写入所需的所有信息，例如格式化信息（§11.6.2）、错误状态（例如是否已到达输入结尾？）以及使用何种缓冲。特别地，用户可以设置状态以反映发生了错误（§11.5），并且如果错误不严重，可以清除状态。例如，我们可以想象一个 `read_ints()` 的版本，它接受一个终止字符串：

```cpp
vector<int> read_ints(istream& is, const string& terminator)
{
    vector<int> res;
    for (int i; is >> i; )
        res.push_back(i);

    if (is.eof())               // 没问题：文件结束
        return res;
    if (is.fail()) {            // 读取整数失败；是否是终止字符串？
        is.clear();             // 将状态重置为 good()
        string s;
        if (is >> s && s == terminator)
            return res;
        is.setstate(ios_base::failbit);   // 向 is 的状态添加 fail()
    }
    return res;
}

auto v = read_ints(cin, "stop");
```

## 11.5 用户定义类型的 I/O

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

## 11.6 输出格式化

`iostream` 和 `format` 库提供了控制输入和输出格式的操作。`iostream` 设施与 C++ 一样古老，专注于数字流的格式化。`format` 设施（§11.6.2）是较新的（C++20），专注于 `printf()` 风格（§11.8）的值组合的格式化规范。

输出格式化还提供对 unicode 的支持，但这超出了本书的范围。

## 11.6.1 流格式化

最简单的格式化控件称为**操纵符**，位于 `<ios>`、`<istream>`、`<ostream>` 和 `<iomanip>`（用于接受参数的操纵符）中。例如，我们可以将整数输出为十进制（默认）、八进制或十六进制数：

```cpp
cout << 1234 << ' ' << hex << 1234 << ' ' << oct << 1234 << ' ' << dec << 1234 << '\n';
// 输出：1234 4d2 2322 1234
```

我们可以显式设置浮点数的输出格式：

```cpp
constexpr double d = 123.456;

cout << d << "; "                // 使用 d 的默认格式
     << scientific << d << "; "  // 使用 1.123e2 风格格式
     << hexfloat << d << "; "    // 使用十六进制表示法
     << fixed << d << "; "       // 使用 123.456 风格格式
     << defaultfloat << d << '\n'; // 使用默认格式
```

这会产生：
```
123.456; 1.234560e+002; 0x1.edd2f2p+6; 123.456000; 123.456
```

**精度**是一个整数，决定用于显示浮点数的数字位数：

- 通用格式（`defaultfloat`）让实现选择一种格式，以在可用空间中最佳地保留值。精度指定最大数字位数。
- 科学格式（`scientific`）以小数点前一位数字和指数形式呈现值。精度指定小数点后的最大数字位数。
- 定点格式（`fixed`）以整数部分后跟小数点和小数部分的形式呈现值。精度指定小数点后的最大数字位数。

浮点值被四舍五入而不是简单地截断，并且 `precision()` 不会影响整数输出。例如：

```cpp
cout.precision(8);
cout << "precision(8): " << 1234.56789 << ' ' << 1234.56789 << ' ' << 123456 << '\n';
cout.precision(4);
cout << "precision(4): " << 1234.56789 << ' ' << 1234.56789 << ' ' << 123456 << '\n';
cout << 1234.56789 << '\n';
```

这会产生：
```
precision(8): 1234.5679 1234.5679 123456
precision(4): 1235 1235 123456
1235
```

这些浮点数操纵符是“粘性的”；也就是说，它们的效果会持续到后续的浮点数操作。也就是说，它们主要是为格式化值流而设计的。

我们还可以指定数字要放置的字段大小及其在字段中的对齐方式。

除基本数字外，`<<` 还可以处理时间和日期：`duration`、`time_point`、`year_month_day`、`weekday`、`month` 和 `zoned_time`（§16.2）。例如：

```cpp
cout << "birthday: " << November/28/2021 << '\n';
cout << "zt: " << zoned_time{current_zone(), system_clock::now()} << '\n';
```

这会产生：
```
birthday: 2021-11-28
zt: 2021-12-05 11:03:13.5945638 EST
```

标准还为复数、`bitset`（§15.3.2）、错误码和指针定义了 `<<`。流 I/O 是可扩展的，因此我们可以为我们自己的（用户定义的）类型定义 `<<`（§11.5）。

## 11.6.2 printf() 风格的格式化

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

## 11.8 C 风格 I/O

C++ 标准库也支持 C 标准库的 I/O，包括 `printf()` 和 `scanf()`。从类型和安全性角度来看，这个库的许多用法是不安全的，因此我不推荐使用它。特别是，它很难用于安全且方便的输入。它不支持用户定义类型。如果你不使用 C 风格 I/O 但关心 I/O 性能，请调用：

```cpp
ios_base::sync_with_stdio(false);   // 避免显著开销
```

如果没有这个调用，标准 `iostream`（例如 `cin` 和 `cout`）可能会因与 C 风格 I/O 兼容而显著变慢。

如果你喜欢 `printf()` 风格的格式化输出，请使用 `format()`（§11.6.2）；它是类型安全的、更易用的、同样灵活且同样快速。

## 11.9 文件系统

大多数系统都有文件系统的概念，提供对存储为文件的持久信息的访问。不幸的是，文件系统的属性和操作它们的方式差异很大。为了解决这个问题，`<filesystem>` 中的文件系统库为大多数文件系统的大多数设施提供了统一的接口。使用 `<filesystem>`，我们可以可移植地：

- 表达文件系统路径并在文件系统中导航
- 检查文件类型及其关联的权限

文件系统库可以处理 unicode，但解释如何做超出了本书的范围。我推荐 cppreference [Cppreference] 和 Boost 文件系统文档 [Boost] 以获取详细信息。

## 11.9.1 路径

考虑一个例子：

```cpp
path f = "dir/hypothetical.cpp";   // 命名一个文件

assert(exists(f));                 // f 必须存在

if (is_regular_file(f))            // f 是普通文件吗？
    cout << f << " is a file; its size is " << file_size(f) << '\n';
```

===== 第 21 页 =====

注意，操作文件系统的程序通常与其他程序一起在计算机上运行。因此，文件系统的内容可能在两个命令之间发生变化。例如，即使我们首先仔细断言了 `f` 存在，当我们在下一行询问 `f` 是否是普通文件时，这可能不再为真。

`path` 是一个相当复杂的类，能够处理许多操作系统的不同字符集和约定。特别地，它可以处理来自 `main()` 呈现的命令行中的文件名；例如：

```cpp
int main(int argc, char* argv[])
{
    if (argc < 2) {
        cerr << "arguments expected\n";
        return 1;
    }

    path p {argv[1]};                // 从命令行创建一个路径
    cout << p << " " << exists(p) << '\n';   // 注意：path 可以像字符串一样打印
    // ...
}
```

路径在被使用之前不会检查其有效性。即使在那时，其有效性也取决于程序运行所在系统的约定。

自然地，路径可用于打开文件：

```cpp
void use(path p)
{
    ofstream f {p};
    if (!f) error("bad file name: ", p);
    f << "Hello, file!";
}
```

除了 `path`，`<filesystem>` 还提供了用于遍历目录和查询所找到文件属性的类型：

[表：文件系统类型（部分）]

考虑一个简单但并非完全不现实的例子：

```cpp
void print_directory(path p)   // 打印 p 中所有文件的名称
try {
    if (is_directory(p)) {
        cout << p << ":\n";
        for (const directory_entry& x : directory_iterator{p})
            cout << "  " << x.path() << '\n';
    }
}
catch (const filesystem_error& ex) {
    cerr << ex.what() << '\n';
}
```

字符串可以隐式转换为 `path`，因此我们可以这样运行 `print_directory`：

```cpp
void use()
{
    print_directory(".");   // 当前目录
    print_directory("..");  // 父目录
    print_directory("/");   // Unix 根目录
    print_directory("c:");  // Windows 卷 C

    for (string s; cin >> s; )
        print_directory(s);
}
```

如果我还想列出子目录，我会使用 `recursive_directory_iterator(p)`。如果我想按字典顺序打印条目，我会将路径复制到一个 `vector` 中，并在打印前对其进行排序。

类 `path` 提供了许多常见且有用的操作：

[表：路径操作（部分）]

例如：

```cpp
void test(path p)
{
    if (is_directory(p)) {
        cout << p << ":\n";
        for (const directory_entry& x : directory_iterator(p)) {
            const path& f = x;   // 引用目录条目的路径部分
            if (f.extension() == ".exe")
                cout << f.stem() << " is a Windows executable\n";
            else {
                string n = f.extension().string();
                if (n == ".cpp" || n == ".C" || n == ".cxx")
                    cout << f.stem() << " is a C++ source file\n";
            }
        }
    }
}
```

我们将 `path` 用作字符串（例如 `f.extension()`），并且可以从 `path` 中提取各种类型的字符串（例如 `f.extension().string()`）。

命名约定、自然语言和字符串编码非常复杂。标准库的文件系统抽象提供了可移植性和极大的简化。

## 11.9.2 文件与目录

自然地，文件系统提供许多操作，并且不同的操作系统自然提供不同的操作集。标准库提供了少数可以在多种系统上合理实现的操作。

[表：文件系统操作（部分）]

与 `copy()` 一样，所有操作都有两个版本：

- 表中列出的基本版本，例如 `exists(p)`。如果操作失败，该函数将抛出 `filesystem_error`。
- 一个带有额外 `error_code` 参数的版本，例如 `exists(p, e)`。检查 `e` 以查看操作是否成功。

当操作在正常使用中预期频繁失败时，我们使用错误码版本；当错误被认为是异常情况时，我们使用抛出异常的版本。

通常，使用查询函数是检查文件属性的最简单直接的方法。`<filesystem>` 库知道几种常见的文件类型，并将其余分类为“其他”：

[表：文件类型]

## 11.10 建议

[1] `iostream` 是类型安全、类型敏感且可扩展的；§11.1。
[2] 仅在必要时使用字符级输入；§11.3；[CG: SL.io.1]。
[3] 读取时，始终考虑格式错误的输入；§11.3；[CG: SL.io.2]。
[4] 避免 `endl`（如果你不知道 `endl` 是什么，那你就没有错过任何东西）；[CG: SL.io.50]。
[5] 为具有有意义文本表示的值定义 `<<` 和 `>>` 用于用户定义类型；§11.1, §11.2, §11.3。
[6] 使用 `cout` 进行普通输出，使用 `cerr` 进行错误输出；§11.1。
[7] 有用于普通字符和宽字符的 `iostream`，你可以为任何类型的字符定义 `iostream`；§11.1。
[8] 支持二进制 I/O；§11.1。
[9] 有用于标准 I/O 流、文件和字符串的标准 `iostream`；§11.2, §11.3, §11.7.2, §11.7.3。
[10] 链式使用 `<<` 操作以获得更简洁的表示法；§11.2。
[11] 链式使用 `>>` 操作以获得更简洁的表示法；§11.3。
[12] 输入到字符串不会溢出；§11.3。
[13] 默认情况下 `>>` 会跳过开头的空白；§11.3。

===== 第 27 页 =====

[14] 使用流状态 `fail` 处理可能可恢复的 I/O 错误；§11.4。
[15] 我们可以为自己的类型定义 `<<` 和 `>>` 运算符；§11.5。
[16] 我们不需要修改 `istream` 或 `ostream` 来添加新的 `<<` 和 `>>` 运算符；§11.5。
[17] 使用操纵符或 `format()` 来控制格式化；§11.6.1, §11.6.2。
[18] `precision()` 规范适用于所有后续的浮点数输出操作；§11.6.1。
[19] 浮点数格式规范（例如 `scientific`）适用于所有后续的浮点数输出操作；§11.6.1。
[20] 使用标准操纵符时，`#include <ios>` 或 `<iostream>`；§11.6。
[21] 流格式化操纵符是“粘性的”，适用于流中的多个值；§11.6.1。
[22] 使用带参数的标准操纵符时，`#include <iomanip>`；§11.6。
[23] 我们可以以标准格式输出时间、日期等；§11.6.1, §11.6.2。
[24] 不要试图拷贝流：流只能移动；§11.7。
[25] 记住在使用文件流之前检查它是否已附加到文件；§11.7.2。
[26] 使用 `stringstream` 或内存流进行内存中格式化；§11.7.3; §11.7.4。
[27] 我们可以定义任何两个都具有字符串表示的类型之间的转换；§11.7.3。
[28] C 风格 I/O 不是类型安全的；§11.8。
[29] 除非你使用 `printf` 系列函数，否则调用 `ios_base::sync_with_stdio(false)`；§11.8；[CG: SL.io.10]。
[30] 优先使用 `<filesystem>` 而不是直接使用平台特定的接口；§11.9。