# 1.6 常量

C++ 支持两种不变性（immutability）概念：

- `const`：大致意思是"我承诺不会更改这个值"。它主要用于定义接口，这样就可以通过指针或引用将数据传递给函数，而无需担心数据会被修改。编译器会确保 `const` 所做出的承诺得到遵守。当然，`const` 的值可以在运行时计算。

- `constexpr`：大致意思是"在编译时求值"。它主要用于指定常量，这样就可以将数据放在只读内存中，从而避免数据被篡改。这可以提升性能。`constexpr` 的值必须由编译器计算得出。

例如：

```cpp
const int dmv = 17;                     // dmv 是一个命名的常量
int var = 17;                           // var 不是常量

constexpr double max1 = 1.4 * square(dmv);  // 如果 square(17) 是常量表达式则 OK
constexpr double max2 = 1.4 * square(var);  // 错误：var 不是常量表达式
const double max3 = 1.4 * square(var);      // OK，可以在运行时求值
```

`constexpr` 函数可以用于处理非常量参数，但这样一来，其返回值就不再是常量表达式了。我们允许在不需要常量表达式的场合中，使用 `constexpr` 函数来处理非常量参数。这样一来，就无需为常量表达式和变量分别定义相同的函数了。当希望某个函数仅在编译时被使用时，应将其声明为`consteval`而非 `constexpr`。例如：

```cpp
constexpr double square(double x) { return x * x; }

constexpr double max1 = 1.4*square2(17);            // OK: 1.4*square(17) is a constant 
const double max3 = 1.4*square2(var);                  // error: var is not a constant
```

被声明为 `constexpr` 或 `consteval` 的函数，其实就是C++中所谓的“纯函数”。这类函数不能产生任何副作用，只能使用作为参数传递给它们的信息来进行操作。特别地，它们不能修改非局部变量，但可以包含循环结构，并使用自己的局部变量。例如：

```cpp
constexpr double nth(double x, int n)
{
    double res = 1;
    int i = 0;
    while (i < n) {     // while-loop: do while the condition is true (§1.7.1) 
        res *= x;
        ++i;
    }
    return res;
}
```
在某些情况下，根据语言规则，必须使用常量表达式（例如，数组的边界值(§1.7)）、case语句的标签(§1.8)、模板参数的值(§7.2)，以及用`constexpr`声明的常量）。而在其他情况下，为了提升性能，编译时计算就显得非常重要。不过，无论是否与性能相关，不可变性这一概念——即对象的状态不可被改变——仍然是一个重要的设计考量因素。
