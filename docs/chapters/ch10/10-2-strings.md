# 10.2 字符串

标准库提供了一种字符串类型作为字符串字面量（§1.2.1）的补充；`string` 是一个常规类型（§8.2，§14.5），用于拥有和操作各种字符类型的字符序列。`string` 类型提供了多种有用的字符串操作，例如连接。例如：

```cpp
string compose(const string& name, const string& domain)
{
    return name + '@' + domain;
}

auto addr = compose("dmr", "bell-labs.com");
```

这里，`addr` 被初始化为字符序列 `dmr@bell-labs.com`。字符串的“加法”意味着连接。你可以将字符串、字符串字面量、C 风格字符串或字符连接到字符串。标准 `string` 具有移动构造函数，因此即使是返回长字符串也是高效的（§6.2.2）。

在许多应用程序中，最常见的连接形式是在字符串末尾添加内容。这直接由 `+=` 操作支持。例如：

```cpp
void m2(string& s1, string& s2)
{
    s1 = s1 + '\n';   // 追加换行符
    s2 += '\n';       // 追加换行符
}
```

===== 第 3 页 =====

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

许多有用的字符串操作包括：赋值（使用 `=`）、下标（使用 `[]` 或 `at()`，类似于 `vector`；§12.2.2）、比较（使用 `==` 和 `!=`）、字典序比较（使用 `<`, `<=`, `>`, `>=`）、迭代（使用迭代器、`begin()` 和 `end()`，类似于 `vector`；§13.2）、输入（§11.3）和流式输出（§11.7.3）。

自然地，字符串可以相互比较，也可以与 C 风格字符串（§1.7.1）以及字符串字面量比较。例如：

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

===== 第 4 页 =====

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

要使用 `s` 后缀，你需要使用命名空间 `std::literals::string_literals`（§6.6）。

#### 10.2.1 string 的实现

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
