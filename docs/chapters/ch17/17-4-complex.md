# 17.4 复数

标准库支持一族复数类型，其思路与 §5.2.1 描述的 `complex` 类一脉相承。为了支持标量为单精度浮点数（`float`）、双精度浮点数（`double`）等情形的复数，标准库的 `complex` 是一个模板：

```cpp
template<typename Scalar>
class complex {
public:
    complex(const Scalar& re = {}, const Scalar& im = {});
    // ...
};
```

普通的算术运算与最常见的数学函数对复数均有支持。例如：

```cpp
void f(complex<float> fl, complex<double> db)
{
    complex<long double> ld{fl + sqrt(db)};
    db += fl * 3;
    fl = pow(1 / fl, 2);
    // ...
}
```

`<complex>` 中也定义了常见的数学函数，`sqrt()` 与 `pow()`（幂运算）即在其中（§17.2）。
