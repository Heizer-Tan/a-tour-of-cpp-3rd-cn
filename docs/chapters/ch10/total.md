# 字符串和正则表达式

> **优先标准，而非标新立异。**
>
> —— Strunk & White

# 10.1 引言

文本操作是大多数程序的主要组成部分。C++ 标准库提供了 `string` 类型，使大多数用户不必再通过指针进行 C 风格的字符数组操作。`string_view` 类型允许我们操作字符序列，无论它们以何种方式存储（例如在 `std::string` 或 `char[]` 中）。此外，标准库还提供了正则表达式匹配，以帮助在文本中查找模式。正则表达式的形式与大多数现代语言中常见的类似。字符串和正则表达式对象都可以使用多种字符类型（例如 Unicode）。

# 10.2 字符串

标准库提供了一种字符串类型作为字符串字面量（[§1.2.1](../ch01/1-2-program.md#121-hello-world)）的补充；`string` 是一个常规类型（[§8.2](../ch08/8-2-concepts.md)，[§14.5](../ch14/14-5-concept-overview.md)），用于拥有和操作各种字符类型的字符序列。`string` 类型提供了多种有用的字符串操作，例如连接。例如：

```cpp
string compose(const string& name, const string& domain)
{
    return name + '@' + domain;
}

auto addr = compose("dmr", "bell-labs.com");
```

这里，`addr` 被初始化为字符序列 `dmr@bell-labs.com`。字符串的“加法”意味着连接。你可以将字符串、字符串字面量、C 风格字符串或字符连接到字符串。标准 `string` 具有移动构造函数，因此即使是返回长字符串也是高效的（[§6.2.2](../ch06/6-2-copy-move.md#622-移动容器)）。

在许多应用程序中，最常见的连接形式是在字符串末尾添加内容。这直接由 `+=` 操作支持。例如：

```cpp
void m2(string& s1, string& s2)
{
    s1 = s1 + '\n';   // 追加换行符
    s2 += '\n';       // 追加换行符
}
```

这两种在字符串末尾添加内容的方式在语义上是等价的，但我更喜欢后者，因为它更明确地表达了意图，更简洁，并且可能更高效。

`string` 是可变的。除了 `=` 和 `+=`，还支持下标操作（使用 `[]`）和子串操作。例如：

```cpp
string name = "Niels Stroustrup";

void m3()
{
    string s = name.substr(6, 10);      // s = "Stroustrup"
    name.replace(0, 5, "nicholas");     // name 变为 "nicholas Stroustrup"
    name[0] = toupper(name[0]);         // name 变为 "Nicholas Stroustrup"
}
```

`substr()` 操作返回一个字符串，它是其参数所指示子串的一份拷贝。第一个参数是字符串中的索引（位置），第二个是所需子串的长度。由于索引从 0 开始，`s` 得到的值是 `Stroustrup`。

`replace()` 操作用一个值替换子串。在此例中，从 0 开始长度为 5 的子串是 `Niels`；它被替换为 `nicholas`。最后，我将首字符替换为其大写形式。因此，`name` 的最终值是 `Nicholas Stroustrup`。注意，替换字符串不必与被替换的子串大小相同。

许多有用的字符串操作包括：赋值（使用 `=`）、下标（使用 `[]` 或 `at()`，类似于 `vector`；[§12.2.2](../ch12/12-2-vector.md#1222-范围检查)）、比较（使用 `==` 和 `!=`）、字典序比较（使用 `<`, `<=`, `>`, `>=`）、迭代（使用迭代器、`begin()` 和 `end()`，类似于 `vector`；[§13.2](../ch13/13-2-using-iterators.md)）、输入（[§11.3](../ch11/11-3-input.md)）和流式输出（[§11.7.3](../ch11/11-7-streams.md#1173-字符串流)）。

自然地，字符串可以相互比较，也可以与 C 风格字符串（[§1.7.1](../ch01/1-7-pointers-arrays.md)）以及字符串字面量比较。例如：

```cpp
string incantation;

void respond(const string& answer)
{
    if (answer == incantation) {
        // ... 执行魔法 ...
    }
    else if (answer == "yes") {
        // ...
    }
    // ...
}
```

如果你需要 C 风格字符串（一个以零结尾的 `char` 数组），`string` 提供了对其所含字符的只读访问（`c_str()` 和 `data()`）。例如：

```cpp
void print(const string& s)
{
    printf("For people who like printf: %s\n", s.c_str());
    cout << "For people who like streams: " << s << '\n';
}
```

根据定义，字符串字面量是 `const char*`。要获得 `std::string` 类型的字面量，请使用 `s` 后缀。例如：

```cpp
auto cat = "Cat"s;   // 一个 std::string
auto dog = "Dog";    // 一个 C 风格字符串：const char*
```

要使用 `s` 后缀，你需要使用命名空间 `std::literals::string_literals`（[§6.6](../ch06/6-6-user-defined-literals.md)）。

## 10.2.1 string 的实现

实现一个 `string` 类是一个流行且有用的练习。然而，对于通用用途，我们精心设计的初次尝试很少能在便利性或性能上与标准 `string` 匹敌。如今，`string` 通常使用短字符串优化来实现。也就是说，短的字符串值直接保存在 `string` 对象本身中，只有较长的字符串才放置在自由存储区。考虑：

```cpp
string s1 {"Annemarie"};              // 短字符串
string s2 {"Annemarie Stroustrup"};   // 长字符串
```

内存布局大致如下：

[图片描述]

当字符串的值从短变为长（反之亦然）时，其表示会相应地调整。一个“短”字符串能有多少个字符？这是实现定义的，但“大约 14 个字符”是一个不错的猜测。

字符串的实际性能可能严重依赖于运行时环境。特别是在多线程实现中，内存分配可能相对昂贵。此外，当大量使用不同长度的字符串时，可能导致内存碎片。这些是短字符串优化变得无处不在的主要原因。

为了处理多种字符集，`string` 实际上是通用模板 `basic_string` 以字符类型 `char` 实例化的别名：

```cpp
template<typename Char>
class basic_string {
    // ... Char 的字符串 ...
};

using string = basic_string<char>;
```

用户可以定义任意字符类型的字符串。例如，假设我们有一个日语字符类型 `Jchar`，我们可以写：

```cpp
using Jstring = basic_string<Jchar>;
```

现在我们可以对 `Jstring`（一个日语字符的字符串）执行所有常规的字符串操作。

# 10.3 字符串视图

字符序列最常见的用途是将其传递给某个函数进行读取。这可以通过按值传递 `string`、传递 `string` 的引用或传递 C 风格字符串来实现。在许多系统中还有更多选择，例如标准未提供的字符串类型。在所有这些情况下，当我们要传递子串时，会遇到额外的复杂性。为了解决这个问题，标准库提供了 `string_view`；`string_view` 基本上是一个（指针，长度）对，表示一个字符序列：

[图片描述]

`string_view` 提供了对连续字符序列的访问。这些字符可以以多种方式存储，包括在 `string` 中和在 C 风格字符串中。`string_view` 像指针或引用一样，它不拥有它指向的字符。在这方面，它类似于 STL 的一对迭代器（[§13.3](../ch13/13-3-iterator-types.md)）。

考虑一个简单的函数：

```cpp
string cat(string_view sv1, string_view sv2)
{
    string res {sv1};        // 用 sv1 初始化字符串
    return res += sv2;       // 追加 sv2 并返回
}
```

我们可以这样调用 `cat()`：

```cpp
string king = "Harold";
auto s1 = cat(king, "William");            // HaroldWilliam
auto s2 = cat(king, king);                 // HaroldHarold
auto s3 = cat("Edward", "Stephen"sv);      // EdwardStephen
auto s4 = cat("Canute"sv, king);           // CanuteHarold
auto s5 = cat({&king[0], 2}, "Henry"sv);   // HaHenry
auto s6 = cat({&king[0], 2}, {&king[2], 4}); // Harold
```

这个 `cat()` 相比于接受 `const string&` 参数的 `compose()`（[§10.2](10-2-strings.md)）有三个优势：

- 它可以用于以多种不同方式管理的字符序列。
- 我们可以轻松传递子串。
- 我们不必创建 `string` 来传递 C 风格字符串参数。

注意 `sv`（“string view”后缀）的使用。要使用它，我们需要使其可见：

```cpp
using namespace std::literals::string_view_literals;   // [§6.6](../ch06/6-6-user-defined-literals.md)
```

为什么需要后缀？原因是当我们传递 `"Edward"` 时，需要从 `const char*` 构造一个 `string_view`，这需要计算字符个数。而对于 `"Stephen"sv`，长度是在编译时计算的。

`string_view` 定义了一个范围，因此我们可以遍历其字符。例如：

```cpp
void print_lower(string_view sv1)
{
    for (char ch : sv1)
        cout << tolower(ch);
}
```

`string_view` 的一个显著限制是它对其字符是只读视图。例如，你不能使用 `string_view` 将字符传递给一个将其参数修改为小写的函数。为此，你可以考虑使用 `span`（[§15.2.2](../ch15/15-2-pointers.md#1522-span)）。

将 `string_view` 视为一种指针；要使用它，它必须指向某个东西：

```cpp
string_view bad()
{
    string s = "Once upon a time";
    return {&s[5], 4};   // 错误：返回指向局部对象的指针
}
```

这里，返回的 `string_view` 会在我们能够使用其字符之前，字符串 `s` 就被销毁了。

对 `string_view` 进行越界访问的行为是未定义的。如果你需要保证范围检查，请使用 `at()`，它会在尝试越界访问时抛出 `out_of_range`，或者使用 `gsl::string_span`（[§15.2.2](../ch15/15-2-pointers.md#1522-span)）。

# 10.4 正则表达式

正则表达式是文本处理的强大工具。它们提供了一种简洁地描述文本模式（例如美国邮政编码，如 TX 77845，或 ISO 风格的日期，如 2009-06-07）并高效查找这些模式的方法。在 `<regex>` 中，标准库以 `std::regex` 类及其支持函数的形式提供了对正则表达式的支持。为了感受一下正则表达式库的风格，让我们定义并打印一个模式：

```cpp
regex pat {R"(\w{2}\s*\d{5}(-\d{4})?)"};   // 美国邮政编码模式：XXddddd-dddd 等
```

使用过任何语言中的正则表达式的人都会对 `\w{2}\s*\d{5}(-\d{4})?` 感到熟悉。它指定了一个模式：以两个字母 `\w{2}` 开头，可选地后跟一些空格 `\s*`，然后跟五个数字 `\d{5}`，最后可选地后跟一个短横线和四个数字 `-\d{4}`。如果你不熟悉正则表达式，现在可能是学习它们的好时机（[Stroustrup,2009]，[Maddock,2009]，[Friedl,1997]）。

为了表达这个模式，我使用了以 `R"(` 开头、以 `)"` 结尾的原始字符串字面量。这允许在字符串中直接使用反斜杠和引号。原始字符串特别适合正则表达式，因为它们往往包含许多反斜杠。如果我使用常规字符串，模式定义将是：

```cpp
regex pat {"\\w{2}\\s*\\d{5}(-\\d{4})?"};   // 美国邮政编码模式
```

在 `<regex>` 中，标准库提供了对正则表达式的支持：

- `regex_match()`：将正则表达式与一个字符串（已知大小）进行匹配（[§10.4.2](#1042-正则表达式记法)）。
- `regex_search()`：在（任意长的）数据流中搜索与正则表达式匹配的字符串（[§10.4.1](#1041-搜索)）。
- `regex_replace()`：在（任意长的）数据流中搜索与正则表达式匹配的字符串并替换它们。
- `regex_iterator`：遍历匹配项和子匹配项（[§10.4.3](#1043-迭代器regex_iterator)）。
- `regex_token_iterator`：遍历非匹配项。

## 10.4.1 搜索

使用模式的最简单方法是在流中搜索它：

```cpp
int lineno = 0;
for (string line; getline(cin, line); ) {   // 读入行缓冲区
    ++lineno;
    smatch matches;                         // 匹配的字符串放在这里
    if (regex_search(line, matches, pat))   // 在 line 中搜索 pat
        cout << lineno << ": " << matches[0] << '\n';
}
```

`regex_search(line, matches, pat)` 在 `line` 中搜索与存储在 `pat` 中的正则表达式匹配的任何内容，如果找到任何匹配项，则将其存储在 `matches` 中。如果没有找到匹配项，`regex_search(line, matches, pat)` 返回 `false`。变量 `matches` 的类型是 `smatch`。“s”代表“子”或“字符串”，`smatch` 是一个类型为 `string` 的子匹配项向量。第一个元素（这里 `matches[0]`）是完整的匹配项。`regex_search()` 的结果是一个匹配项的集合，通常表示为 `smatch`：

```cpp
void use()
{
    ifstream in("file.txt");                // 输入文件
    if (!in) {                              // 检查文件是否成功打开
        cerr << "no file\n";
        return;
    }

    regex pat {R"(\w{2}\s*\d{5}(-\d{4})?)"}; // 美国邮政编码模式

    int lineno = 0;
    for (string line; getline(in, line); ) {
        ++lineno;
        smatch matches;                     // 匹配的字符串放在这里
        if (regex_search(line, matches, pat)) {
            cout << lineno << ": " << matches[0] << '\n';   // 完整匹配
            if (1 < matches.size() && matches[1].matched)   // 如果有子模式且匹配
                cout << "\t: " << matches[1] << '\n';       // 子匹配
        }
    }
}
```

这个函数读取一个文件，寻找美国邮政编码，例如 TX 77845 和 DC 20500-0001。`smatch` 类型是一个正则表达式结果的容器。这里，`matches[0]` 是整个模式，`matches[1]` 是可选的四位数字子模式 `(-\d{4})?`。

换行符 `\n` 可以是模式的一部分，因此我们可以搜索多行模式。显然，如果要这样做，我们不应该一次读取一行。

正则表达式的语法和语义设计使得正则表达式可以被编译成状态机以实现高效执行 [Cox,2007]。`regex` 类型在运行时执行此编译。

## 10.4.2 正则表达式记法

`regex` 库可以识别正则表达式记法的多种变体。这里，我使用默认记法，即 ECMAScript（通常称为 JavaScript）所采用的 ECMA 标准的一种变体。正则表达式的语法基于特殊字符：

| 正则表达式特殊字符 | 说明 |
|-------------------|------|
| `.` | 任意单个字符（“通配符”） |
| `\` | 下一个字符具有特殊含义 |
| `[` | 开始字符类 |
| `*` | 零次或多次（后缀操作） |
| `]` | 结束字符类 |
| `+` | 一次或多次（后缀操作） |
| `{` | 开始计数 |
| `?` | 可选（零次或一次）（后缀操作） |
| `}` | 结束计数 |
| `\|` | 替代（或） |
| `(` | 开始分组 |
| `^` | 行首；否定 |
| `)` | 结束分组 |
| `$` | 行尾 |

例如，我们可以如下指定一个以零个或多个 `A` 开头、后跟一个或多个 `B`、再后跟一个可选的 `C` 的行：
`^A*B+C?$`

匹配的例子：
- `AAAAAAAAAAAABBBBBBBBBC`
- `BC`
- `B`

不匹配的例子：
- `AAAAA`（没有 B）
- ` AAAABC`（开头的空格）
- `AABBCC`（C 太多）

如果模式的一部分被括在括号中，则被视为子模式（可以从 `smatch` 中单独提取）。例如：
- `\d+-\d+`（无子模式）
- `\d+(-\d+)`（一个子模式）
- `(\d+)(-\d+)`（两个子模式）

可以通过添加后缀使模式变为可选或重复（默认是恰好一次）：

| 重复 | 含义 |
|------|------|
| `{n}` | 恰好 n 次 |
| `{n,}` | n 次或更多次 |
| `{n,m}` | 至少 n 次，至多 m 次 |
| `*` | 零次或多次，即 `{0,}` |
| `+` | 一次或多次，即 `{1,}` |
| `?` | 可选（零次或一次），即 `{0,1}` |

例如：
`A{3}B{2,4}C*`

匹配的例子：
- `AAABBC`
- `AAABBB`

不匹配的例子：
- `AABBC`（A 太少）
- `AAABC`（B 太少）
- `AAABBBBBCCC`（B 太多）

在任何重复记法（`?`, `*`, `+`, `{}`）之后添加 `?` 后缀会使模式匹配器变得“懒惰”或“非贪婪”。也就是说，在查找模式时，它会寻找最短匹配而不是最长匹配。默认情况下，模式匹配器总是寻找最长匹配；这称为最大咀嚼规则。考虑：
`ababab`
模式 `(ab)+` 匹配整个 `ababab`。但是 `(ab)+?` 只匹配第一个 `ab`。

最常见的字符类有名称：

| 字符类 | 说明 |
|--------|------|
| `alnum` | 任何字母数字字符 |
| `alpha` | 任何字母字符 |
| `blank` | 任何不是行分隔符的空白字符 |
| `cntrl` | 任何控制字符 |
| `d` | 任何十进制数字 |
| `digit` | 任何十进制数字 |
| `graph` | 任何图形字符 |
| `lower` | 任何小写字母 |
| `print` | 任何可打印字符 |
| `punct` | 任何标点字符 |
| `s` | 任何空白字符 |
| `space` | 任何空白字符 |
| `upper` | 任何大写字母 |
| `w` | 任何单词字符（字母数字字符加下划线） |
| `xdigit` | 任何十六进制数字字符 |

在正则表达式中，字符类名称必须用 `[: :]` 括起来。例如，`[:digit:]` 匹配一个十进制数字。此外，它们必须在定义字符类的 `[ ]` 对内使用。

几种字符类有简写符号：

| 简写 | 含义 | 等价 |
|------|------|------|
| `\d` | 十进制数字 | `[[:digit:]]` |
| `\s` | 空格（空格、制表符等） | `[[:space:]]` |
| `\w` | 字母 (a-z) 或数字 (0-9) 或下划线 (_) | `[_[:alnum:]]` |
| `\D` | 不是 `\d` | `[^[:digit:]]` |
| `\S` | 不是 `\s` | `[^[:space:]]` |
| `\W` | 不是 `\w` | `[^_[:alnum:]]` |

此外，支持正则表达式的语言通常还提供：

| 非标准（但常见）简写 | 含义 | 等价 |
|----------------------|------|------|
| `\l` | 小写字母 | `[[:lower:]]` |
| `\u` | 大写字母 | `[[:upper:]]` |
| `\L` | 不是 `\l` | `[^[:lower:]]` |
| `\U` | 不是 `\u` | `[^[:upper:]]` |

为了完全可移植性，请使用字符类名称而不是这些缩写。

作为示例，考虑编写一个描述 C++ 标识符的模式：下划线或字母后跟可能为空的字母、数字或下划线序列。为了说明其中的微妙之处，我包含了几次错误的尝试：

```cpp
[:alpha:][:alnum:]*               // 错误：来自集合 ":alpha" 的字符后跟...
[[:alpha:]][[:alnum:]]*           // 错误：不接受下划线（_ 不是 alpha）
([[:alpha:]]|_)[[:alnum:]]*       // 错误：下划线也不是 alnum 的一部分
([[:alpha:]]|_)([[:alnum:]]|_)*   // 可以，但笨拙
[[:alpha:]_][[:alnum:]_]*         // 可以：将下划线包含在字符类中
[_[:alpha:]][_[:alnum:]]*         // 也可以
[_[:alpha:]]\w*                   // \w 等价于 [_[:alnum:]]
```

最后，这里有一个使用最简单版本的 `regex_match()`（见 [§10.4.1](#1041-搜索) 中的用法）来测试字符串是否为标识符的函数：

```cpp
bool is_identifier(const string& s)
{
    regex pat {"[_[:alpha:]]\\w*"};   // 下划线或字母，后跟零个或多个下划线、字母或数字
    return regex_match(s, pat);
}
```

注意在普通字符串字面量中为了包含反斜杠需要将其写成 `\\`。使用原始字符串字面量（[§10.4](#104-正则表达式)）可以减轻特殊字符带来的麻烦。例如：

```cpp
bool is_identifier(const string& s)
{
    regex pat {R"([_[:alpha:]]\w*)"};
    return regex_match(s, pat);
}
```

以下是一些模式的示例：

```cpp
Ax*           // A, Ax, Axxxx
Ax+           // Ax, Axxx 不包括 A
\d-?\d        // 1-2, 12 不包括 1--2
\w{2}-\d{4,5} // Ab-1234, XX-54321, 22-5432（数字属于 \w）
(\d*:)?(\d+)  // 12:3, 1:23, 123, :123 不包括 123:
(bs|BS)       // bs, BS 不包括 bS
[aeiouy]      // a, o, u 英语元音，不是 x
[^aeiouy]     // x, k 不是英语元音，不是 e
[a^eiouy]     // a, ^, o, u 英语元音或 ^
```

可能由 `sub_match` 表示的组（子模式）由括号分隔。如果你需要括号但不应定义子模式，请使用 `(?:` 而不是普通的 `(`。例如：

```cpp
(\s|:|,)*(\d*)   // 可选的空格、冒号和/或逗号，后跟一个可选的数字
```

假设我们对数字之前的字符（可能是分隔符）不感兴趣，我们可以写：

```cpp
(?:\s|:|,)*(\d*)   // 可选的空格、冒号和/或逗号，后跟一个可选的数字
```

这将使正则表达式引擎不必存储第一个字符：`(?:` 变体只有一个子模式。

正则表达式分组示例：

| 模式 | 说明 |
|------|------|
| `\d*\s\w+` | 无分组（子模式） |
| `(\d*)\s(\w+)` | 两个分组 |
| `(\d*)(\s(\w+))+` | 两个分组（分组不嵌套） |
| `(\s*\w*)+` | 一个分组；一个或多个子模式；只有最后一个子模式保存为 sub_match |
| `<(.*?)>(.*?)</\1>` | 三个分组；`\1` 表示“与第 1 组相同” |

最后一个模式对于解析 XML 很有用。它可以找到标签/结束标签标记。注意，我在标签和结束标签之间的子模式使用了非贪婪匹配（懒惰匹配）`.*?`。如果我使用了普通的 `.*`，以下输入会引起问题：

```cpp
Always look on the <b>bright</b> side of <b>life</b>.
```

第一个子模式的贪婪匹配会匹配第一个 `<` 和最后一个 `>`。这行为是正确的，但不太可能是程序员想要的。

有关正则表达式的更详尽介绍，请参阅 [Friedl,1997]。

## 10.4.3 迭代器（regex_iterator）

我们可以定义 `regex_iterator`，用于在字符序列中迭代查找与模式匹配的子串。例如，可以使用 `sregex_iterator`（即 `regex_iterator<string>`）输出字符串中所有由空白分隔的单词：

```cpp
void test()
{
    string input = "aa as; asd ++e^asdf asdfg";
    regex pat {R"(\s+(\w+))"};
    for (sregex_iterator p(input.begin(), input.end(), pat); p != sregex_iterator{}; ++p)
        cout << (*p)[1] << '\n';
}
```

输出为：

```
as
asd
asdfg
```

我们遗漏了第一个单词 `aa`，因为它前面没有空白。若将模式简化为 `R"((\w+))"`，则输出为：

```
aa
as
asd
e
asdf
asdfg
```

`regex_iterator` 是双向迭代器，因此不能直接对 `istream`（只提供输入迭代器）进行迭代。此外，不能通过 `regex_iterator` 写入，并且默认的 `regex_iterator`（`regex_iterator{}`）是唯一可能的序列尾迭代器。

# 10.5 建议

[1] 使用 `std::string` 拥有字符序列；[§10.2](10-2-strings.md)；[CG: SL.str.1]。
[2] 优先使用字符串操作，而非 C 风格字符串函数；[§10.1](10-1-introduction.md)。
[3] 使用 `string` 声明变量和成员，而非作为基类；[§10.2](10-2-strings.md)。
[4] 按值返回字符串（依赖移动语义和拷贝省略）；[§10.2](10-2-strings.md)，[§10.2.1](10-2-strings.md#1021-string-的实现)；[CG: F.15]。
[5] 直接或间接使用 `substr()` 读取子串，使用 `replace()` 写入子串；[§10.2](10-2-strings.md)。
[6] 字符串可以根据需要增长和收缩；[§10.2](10-2-strings.md)。
[7] 当你需要范围检查时，使用 `at()` 而非迭代器或 `[]`；[§10.2](10-2-strings.md)，[§10.3](10-3-string-view.md)。
[8] 当你需要优化速度时，使用迭代器和 `[]` 而非 `at()`；[§10.2](10-2-strings.md)，[§10.3](10-3-string-view.md)。
[9] 使用范围 `for` 以安全地减少范围检查；[§10.2](10-2-strings.md)，[§10.3](10-3-string-view.md)。
[10] 读入 `string` 不会溢出；[§10.2](10-2-strings.md)，[§11.3](../ch11/11-3-input.md)。
[11] 仅在必须时，使用 `c_str()` 或 `data()` 生成字符串的 C 风格表示；[§10.2](10-2-strings.md)。
[12] 使用 `stringstream` 或通用值提取函数（如 `to<X>`）进行字符串的数值转换；[§11.7.3](../ch11/11-7-streams.md#1173-字符串流)。
[13] `basic_string` 可用于构造任意字符类型的字符串；[§10.2.1](10-2-strings.md#1021-string-的实现)。
[14] 对表示标准库字符串的字符串字面量使用 `s` 后缀；[§10.2](10-2-strings.md)；[CG: SL.str.12]。（亦见 [§6.6](../ch06/6-6-user-defined-literals.md)。）
[15] 将 `string_view` 作为需要以多种方式**读取**字符序列的函数的参数；[§10.3](10-3-string-view.md)；[CG: SL.str.2]。
[16] 将 `string_span<char>`（或等价的可写字符区间类型）作为需要以多种方式**写入**字符序列的函数的参数；[§10.3](10-3-string-view.md)；[CG: SL.str.2] [CG: SL.str.11]。
[17] 将 `string_view` 视为一种带长度的指针；它不拥有其字符；[§10.3](10-3-string-view.md)。
[18] 对表示标准库 `string_view` 的字符串字面量使用 `sv` 后缀；[§10.3](10-3-string-view.md)。
[19] 对大多数常规正则表达式用途使用 `regex`；[§10.4](10-4-regular-expressions.md)。
[20] 除最简单的模式外，优先用原始字符串字面量来表达模式；[§10.4](10-4-regular-expressions.md)。
[21] 使用 `regex_match()` 匹配完整输入；[§10.4](10-4-regular-expressions.md)，[§10.4.2](10-4-regular-expressions.md#1042-正则表达式记法)。
[22] 使用 `regex_search()` 在输入流中搜索模式；[§10.4.1](10-4-regular-expressions.md#1041-搜索)。
[23] 正则表达式记号可调整以匹配各种标准；[§10.4.2](10-4-regular-expressions.md#1042-正则表达式记法)。
[24] 默认正则表达式记号是 ECMAScript（JavaScript）所基于的 ECMA 变体；[§10.4.2](10-4-regular-expressions.md#1042-正则表达式记法)。
[25] 有所节制；正则表达式很容易变成只写语言；[§10.4.2](10-4-regular-expressions.md#1042-正则表达式记法)。
[26] 注意反向引用 `\1`、`\2` 等可用来引用先前的子模式；[§10.4.2](10-4-regular-expressions.md#1042-正则表达式记法)。
[27] 使用后缀 `?` 使重复为“惰性”（最短匹配）；[§10.4.2](10-4-regular-expressions.md#1042-正则表达式记法)。
[28] 使用 `regex_iterator` 在字符序列上迭代查找模式；[§10.4.3](10-4-regular-expressions.md#1043-迭代器regex_iterator)。
