# 6.6 用户定义字面量

类的目的之一是使程序员能够设计并实现与内置类型行为高度相似的类型。构造函数提供了与内置类型初始化相当或更灵活高效的初始化，但内置类型拥有字面量：
- `123` 是一个 `int`
- `0xFF00u` 是一个 `unsigned int`
- `123.456` 是一个 `double`
- `"Surprise!"` 是一个 `const char[10]`

为用户定义类型也提供这样的字面量可能很有用。这是通过为字面量定义合适的后缀含义来实现的，从而我们可以得到：
- `"Surprise!"s` 是一个 `std::string`
- `123s` 是秒
- `12.7i` 是虚数，所以 `12.7i + 47` 是一个复数（即 `{47, 12.7}`）

特别地，我们可以通过使用合适的头文件和命名空间从标准库中获得这些示例：

**标准库字面量后缀**

| 头文件 | 命名空间 | 后缀 |
|--------|----------|------|
| `<chrono>` | `std::literals::chrono_literals` | `h`, `min`, `s`, `ms`, `us`, `ns` |
| `<string>` | `std::literals::string_literals` | `s` |
| `<string_view>` | `std::literals::string_literals` | `sv` |
| `<complex>` | `std::literals::complex_literals` | `i`, `il`, `if` |

带有用户定义后缀的字面量称为**用户定义字面量**（UDL）。这类字面量通过字面量运算符定义。字面量运算符将其参数类型的字面量（后跟后缀）转换为其返回类型。例如，虚数后缀 `i` 可以这样实现：

```cpp
constexpr complex<double> operator""i(long double arg)   // 虚数字面量
{
    return {0, arg};
}
```

这里：
- `operator""` 表示我们正在定义一个字面量运算符。
- 字面量指示符 `""` 后面的 `i` 是我们赋予含义的后缀。
- 参数类型 `long double` 表示后缀 `i` 是为浮点字面量定义的。
- 返回类型 `complex<double>` 指定了结果字面量的类型。

有了这个，我们可以写：

```cpp
complex<double> z = 2.7182818 + 6.2831851i;
```

后缀 `i` 的实现和 `+` 都是 `constexpr`，因此 `z` 的计算在编译时完成。
