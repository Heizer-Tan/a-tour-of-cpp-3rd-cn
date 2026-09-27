# 3.3 命名空间

除了函数（[§1.3](../ch01/1-3-functions.md)）、类（[§2.3](../ch02/2-3-class.md)）和枚举（[§2.4](../ch02/2-4-enum.md)）之外，C++ 还提供命名空间（namespace），用来表达“某些声明属于同一组”，并避免它们的名字与其他名字冲突。例如，我可能想试验自己的复数类型（[§5.2.1](../ch05/5-2-concrete-types.md#5.2.1)、[§17.4](../ch17/17-4-complex.md)）：

```cpp
namespace My_code {
    class complex {
        // ...
    };

    complex sqrt(complex);
    // ...

    int main();
}

int My_code::main()
{
    complex z {1,2};
    auto z2 = sqrt(z);
    std::cout << '{' << z2.real() << ',' << z2.imag() << "}\n";
    // ...
}

int main()
{
    return My_code::main();
}
```

把代码放进命名空间 `My_code`，就能确保我的名字不会与命名空间 `std` 中的标准库名字冲突（[§3.3](3-3-namespace.md)）。这种谨慎是明智的，因为标准库本身就提供复数运算支持（[§5.2.1](../ch05/5-2-concrete-types.md#5.2.1)、[§17.4](../ch17/17-4-complex.md)）。

访问另一个命名空间中名字的最简单方式，是用命名空间名加以限定（例如 `std::cout` 和 `My_code::main`）。那个“真正的 `main()`”定义在全局命名空间中，也就是说，它并不局部于某个已定义的命名空间、类或函数。

如果反复限定某个名字变得冗长或干扰阅读，可以用 *using 声明*（using-declaration）把该名字引入某个作用域：

```cpp
void my_code(vector<int>& x, vector<int>& y)
{
    using std::swap;        // 使标准库的 swap 在局部可用
    // ...
    swap(x,y);              // std::swap()
    other::swap(x,y);       // 某个其他的 swap()
    // ...
}
```

using 声明使来自某命名空间的名字，就像在出现该声明的作用域中声明过一样可用。写了 `using std::swap` 之后，效果就完全等同于在 `my_code()` 中声明了 `swap`。

若要获得标准库命名空间中的全部名字，可以使用 *using 指令*（using-directive）：

```cpp
using namespace std;
```

using 指令使来自所指定命名空间的未限定名字，在放置该指令的作用域中可访问。因此，对 `std` 使用 using 指令之后，就可以直接写 `cout`，而不必写 `std::cout`。例如，在前面那个不起眼的 `vector_printer` 模块里，就可以避免反复写 `std::`：

```cpp
export module vector_printer;

import std;
using namespace std;

export
template<typename T>
void print(vector<T>& v)    // 这是用户能看到的（唯一）函数
{
    cout << "{\n";
    for (const T& val : v)
        cout << " " << val << '\n';
    cout << '}';
}
```

重要的是，这种命名空间指令的用法不会影响我们模块的用户；它是模块内部的实现细节。

使用 using 指令后，我们就失去了有选择地使用该命名空间中名字的能力，因此应谨慎使用——通常用于在应用中无处不在的库（例如 `std`），或用于尚未使用命名空间的应用在过渡阶段。

命名空间主要用于组织较大的程序组成部分，例如库。它们让我们更容易把分别开发的部分拼成一个完整程序。
