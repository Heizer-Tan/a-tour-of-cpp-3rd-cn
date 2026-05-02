# 1.6 常量

C++ 支持两种不变性（immutability）概念：

- `const`：大致意思是"我承诺不修改这个值"。这主要用于说明接口，使得数据可以通过函数参数传递而不必担心被修改。编译器会强制执行 `const` 的承诺。`const` 的值可以在运行时计算。

- `constexpr`：大致意思是"在编译时求值"。这主要用于指定常量，以便将数据放在只读内存中（从而保护数据不受破坏），并提升性能。`constexpr` 的值必须由编译器计算。

例如：

```cpp
const int dmv = 17;                     // dmv 是一个命名的常量
int var = 17;                           // var 不是常量

constexpr double max1 = 1.4 * square(dmv);  // 如果 square(17) 是常量表达式则 OK
constexpr double max2 = 1.4 * square(var);  // 错误：var 不是常量表达式
const double max3 = 1.4 * square(var);      // OK，可以在运行时求值
```

`constexpr` 函数可以在编译时求值，也可以在运行时求值。如果参数是常量表达式，且结果在要求常量表达式的地方使用，则在编译时求值；否则在运行时求值。

```cpp
constexpr double square(double x) { return x * x; }
```

要成为 `constexpr` 函数，其函数体必须足够简单：只包含一条 `return` 语句，不包含循环和局部变量。在 C++14 中，这个限制放宽了，允许更复杂的函数体。在 C++20 中，甚至允许在 `constexpr` 函数中使用 `new` 和 `delete`（但必须在编译时释放）。

```cpp
constexpr double nth(double x, int n)
{
    double res = 1;
    int i = 0;
    while (i < n) {    // C++14 起允许循环
        res *= x;
        ++i;
    }
    return res;
}
```

`constexpr` 可以用于数据成员、函数、构造函数等。它使得我们能够在编译时进行更多计算，从而减少运行时的开销。

对于在编译时求值的函数调用，如果参数是常量表达式，则结果也是常量表达式。这允许我们在需要常量表达式的地方（如数组边界、模板参数等）使用函数调用。

```cpp
constexpr int max_elements = 100;
int buffer[max_elements];               // OK：常量表达式
int buffer2[nth(2, 10)];               // OK：nth(2,10) 是 1024，是常量表达式
```
