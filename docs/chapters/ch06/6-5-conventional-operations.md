## 6.5 常规操作

除了基本的构造、析构、拷贝和移动之外，类通常还需要一些常规操作。

### 6.5.1 比较运算符

C++20 引入了*三路比较运算符*（three-way comparison operator）`<=>`，也称为"太空船运算符"（spaceship operator）：

```cpp
class Vector {
public:
    auto operator<=>(const Vector&) const = default;   // 自动生成所有比较运算符
    // ...
};
```

`= default` 让编译器自动生成 `==`、`!=`、`<`、`<=`、`>`、`>=` 等所有比较运算符。

### 6.5.2 输入输出

可以通过重载 `<<` 和 `>>` 来支持自定义类型的 I/O：

```cpp
std::ostream& operator<<(std::ostream& os, const complex& z)
{
    return os << '(' << z.real() << ',' << z.imag() << ')';
}

std::istream& operator>>(std::istream& is, complex& z)
{
    double re, im;
    char c;
    is >> c >> re >> c >> im >> c;  // 读取格式: (re,im)
    z = complex{re, im};
    return is;
}
```

### 6.5.3 `swap`

`swap` 是一个重要的操作，常用于实现拷贝赋值和许多算法：

```cpp
void swap(Vector& a, Vector& b) noexcept
{
    std::swap(a.elem, b.elem);
    std::swap(a.sz, b.sz);
}
```

高效的 `swap` 实现通常是移动语义的直接应用。
