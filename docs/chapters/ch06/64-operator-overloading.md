## 6.4 运算符重载

C++ 允许为自定义类型定义运算符的行为。这使得用户定义类型可以像内置类型一样自然地使用。

### 6.4.1 基本运算符

常见的可重载运算符包括：

```cpp
class complex {
public:
    complex(double r = 0, double i = 0) : re{r}, im{i} {}

    complex& operator+=(complex z) { re += z.re; im += z.im; return *this; }
    complex& operator-=(complex z) { re -= z.re; im -= z.im; return *this; }

    double real() const { return re; }
    double imag() const { return im; }
private:
    double re, im;
};

// 二元运算符通常定义为非成员函数
complex operator+(complex a, complex b) { return a += b; }
complex operator-(complex a, complex b) { return a -= b; }
bool operator==(complex a, complex b) { return a.real() == b.real() && a.imag() == b.imag(); }
```

### 6.4.2 运算符重载的原则

- 保持运算符的常规语义——不要用 `+` 来做减法
- 成员函数用于需要修改左操作数的运算符（如 `+=`、`=`、`[]`）
- 非成员函数用于对称的二元运算符（如 `+`、`-`、`==`）
- 不要过度使用运算符重载——如果某个操作没有自然的运算符对应，使用命名函数

### 6.4.3 函数对象（函子）

重载 `operator()` 可以创建*函数对象*（function object，也叫*函子* functor）：

```cpp
struct Less_than {
    double d;
    Less_than(double dd) : d{dd} {}
    bool operator()(double x) const { return x < d; }
};

Less_than lt{42};
bool b = lt(10);    // true: 10 < 42
```

函数对象是泛型编程中的重要工具（[§7.3](../ch07/73-parameterized-operations.md)）。
