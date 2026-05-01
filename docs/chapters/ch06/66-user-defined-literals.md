## 6.6 用户定义字面量

C++ 支持*用户定义字面量*（user-defined literals，UDL），允许为自定义类型提供自然的字面量语法：

```cpp
constexpr complex operator""i(long double d)
{
    return complex{0, static_cast<double>(d)};
}

auto z = 2.0 + 3.0i;    // complex{2, 3}
```

标准库为常用类型提供了字面量后缀：

```cpp
using namespace std::literals;

auto s = "hello"s;          // std::string
auto sv = "hello"sv;        // std::string_view
auto d = 1s + 500ms;        // std::chrono::duration
```

用户定义字面量让代码更加简洁和可读，但应谨慎使用——字面量后缀应该直观且不会引起歧义。
