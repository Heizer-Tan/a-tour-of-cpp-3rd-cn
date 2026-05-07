# 10.3 字符串视图

字符序列最常见的用途是将其传递给某个函数进行读取。这可以通过按值传递 `string`、传递 `string` 的引用或传递 C 风格字符串来实现。在许多系统中还有更多选择，例如标准未提供的字符串类型。在所有这些情况下，当我们要传递子串时，会遇到额外的复杂性。为了解决这个问题，标准库提供了 `string_view`；`string_view` 基本上是一个（指针，长度）对，表示一个字符序列：

[图片描述]

`string_view` 提供了对连续字符序列的访问。这些字符可以以多种方式存储，包括在 `string` 中和在 C 风格字符串中。`string_view` 像指针或引用一样，它不拥有它指向的字符。在这方面，它类似于 STL 的一对迭代器（§13.3）。

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

这个 `cat()` 相比于接受 `const string&` 参数的 `compose()`（§10.2）有三个优势：

- 它可以用于以多种不同方式管理的字符序列。
- 我们可以轻松传递子串。
- 我们不必创建 `string` 来传递 C 风格字符串参数。

注意 `sv`（“string view”后缀）的使用。要使用它，我们需要使其可见：

```cpp
using namespace std::literals::string_view_literals;   // §6.6
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

`string_view` 的一个显著限制是它对其字符是只读视图。例如，你不能使用 `string_view` 将字符传递给一个将其参数修改为小写的函数。为此，你可以考虑使用 `span`（§15.2.2）。

将 `string_view` 视为一种指针；要使用它，它必须指向某个东西：

```cpp
string_view bad()
{
    string s = "Once upon a time";
    return {&s[5], 4};   // 错误：返回指向局部对象的指针
}
```

这里，返回的 `string_view` 会在我们能够使用其字符之前，字符串 `s` 就被销毁了。

对 `string_view` 进行越界访问的行为是未定义的。如果你需要保证范围检查，请使用 `at()`，它会在尝试越界访问时抛出 `out_of_range`，或者使用 `gsl::string_span`（§15.2.2）。

