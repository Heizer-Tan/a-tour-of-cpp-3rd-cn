## 8.4 可变参数模板

可变参数模板（variadic templates）允许模板接受任意数量的参数。这是实现 `std::tuple`、`std::variant` 和 `std::make_shared` 等工具的基础。

### 8.4.1 基本语法

```cpp
template<typename... Args>
void print(Args... args)
{
    (std::cout << ... << args) << '\n';  // C++17 折叠表达式
}

print("Hello", ' ', "World", '!', 2024);
// 输出: Hello World!2024
```

`(std::cout << ... << args)` 是一个*折叠表达式*（fold expression），它将二元运算符应用于参数包中的所有元素。

### 8.4.2 递归展开

在 C++17 之前，可变参数模板通常通过递归来处理：

```cpp
void print() {}  // 递归终止条件

template<typename T, typename... Args>
void print(T first, Args... rest)
{
    std::cout << first;
    if constexpr (sizeof...(rest) > 0)
        std::cout << ' ';
    print(rest...);
}
```

`if constexpr`（C++17）在编译时求值条件，避免了不必要的实例化。

### 8.4.3 `std::tuple`

`std::tuple` 是可变参数模板的经典应用——它是一个可以容纳任意数量和类型元素的异构容器：

```cpp
std::tuple<int, string, double> t {42, "hello", 3.14};
int i = std::get<0>(t);         // 42
string s = std::get<1>(t);      // "hello"
double d = std::get<2>(t);      // 3.14
```
