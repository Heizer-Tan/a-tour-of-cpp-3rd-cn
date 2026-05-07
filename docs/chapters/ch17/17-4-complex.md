# 17.4 复数

标准库支持一系列复数类型，遵循 §5.2.1 中描述的 `complex` 类的路线。为了支持标量为单精度浮点数（`float`）、双精度浮点数（`double`）等的复数，标准库的 `complex` 是一个模板：

```cpp
template<typename Scalar>
class complex {
public:
    complex(const Scalar& re = 0, const Scalar& im = 0);   // 默认函数参数
    // ...
};
```

复数支持通常的算术运算和最常见的数学函数。例如：

```cpp
void f(complex<float> fl, complex<double> db)
{
    complex<long double> ld {fl + sqrt(db)};
    db += fl * 3;
    fl = pow(1/fl, 2);
    // ...
}
```

`sqrt()` 和 `pow()`（幂函数）是 `<complex>` 中定义的常见数学函数之一（§17.2）。

## 17.5 随机数