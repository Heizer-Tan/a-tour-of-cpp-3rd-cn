## 3.3 命名空间

随着程序规模的增长，名称冲突的风险也随之增加。不同库可能定义了同名的函数或类。C++ 通过*命名空间*（namespace）来解决这个问题。

命名空间提供了一种将相关名称组织在一起的作用域机制：

```cpp
namespace My_code {
    class complex { /* ... */ };
    complex sqrt(complex);
    // ...
    int main();
}

int My_code::main()
{
    complex z {1, 2};
    auto z2 = sqrt(z);
    std::cout << '{' << z2.real() << ',' << z2.imag() << "}\n";
    // ...
}

int main()
{
    return My_code::main();
}
```

通过将代码放入命名空间 `My_code`，我们可以确保其中的名称不会与其他命名空间中的同名实体发生冲突。

要访问命名空间中的名称，可以使用作用域解析运算符 `::`：

```cpp
My_code::complex c {1, 2};
My_code::sqrt(c);
```

如果某个名称被频繁使用，可以通过 `using` 声明将其引入当前作用域：

```cpp
using My_code::complex;
complex c {1, 2};   // 现在可以直接使用 complex
```

也可以使用 `using namespace` 指令将整个命名空间中的所有名称引入：

```cpp
using namespace My_code;
complex c {1, 2};   // 所有 My_code 中的名称都可见
```

不过，`using namespace` 应谨慎使用——尤其是在头文件中——因为它会破坏命名空间提供的隔离效果。在 `.cpp` 文件中使用 `using namespace` 通常是安全的。

标准库中的所有名称都位于 `std` 命名空间中。这就是为什么我们写 `std::cout` 而不是 `cout` 的原因。
