## 2.3 类

将数据与其上的操作分离有诸多弊端——比如，用户不得不自己组合使用这两者，而且由于无法对数据施加有效的访问控制，数据很容易被意外篡改。因此，更好的做法是将数据的*表示*（representation）与一组操作封装在一起，形成一个完整的类型。这正是*类*（class）的用武之地。

类拥有一组*成员*（members），这些成员可以是数据、函数，也可以是类型成员。类的对外接口由它的 `public` 成员定义；而 `private` 成员则只能通过接口来间接访问。例如：

```cpp
class Vector {
public:
    Vector(int s) : elem{new double[s]}, sz{s} {} // 构造函数：获取资源
    double& operator[](int i) { return elem[i]; }  // 元素访问：下标运算符
    int size() { return sz; }                       // 返回元素个数
private:
    double* elem; // 指向元素的指针
    int sz;       // 元素个数
};
```

有了这个定义，我们就可以定义一个新的 `Vector` 类型变量：

```cpp
Vector v(6);    // 包含 6 个元素的 Vector
```

下图对此做了示意：

![Vector illustration](../../assets/images/ch02/vector-illustration.png)

简单来说，`Vector` 对象就是一个"句柄"（handle），它持有指向实际元素的指针以及元素个数。这里的 `Vector` 对象在大小上固定为两个 `int` 加一个指针，而 `new` 分配的数组则可以拥有任意数量的元素。不同 `Vector` 对象可以包含不同数量的元素。这种句柄与数据分离的设计是 C++ 中管理变化量信息的常用技巧。

这里的关键点在于，我们通过接口来访问数据——具体来说，就是通过下标运算符 `operator[]` 和 `size()` 函数。`Vector` 的构造函数（constructor）负责初始化数据成员，并确保不变式（invariant）得以成立。构造函数与类同名，例如 `Vector` 的构造函数就是 `Vector()`。

`operator[]` 返回一个 `double&`（对 `double` 的引用），这使得我们既可以读取也可以写入 `elem[i]`：

```cpp
double read_and_sum(int s)
{
    Vector v(s);                    // 创建一个包含 s 个元素的 vector
    for (int i = 0; i != v.size(); ++i)
        std::cin >> v[i];           // 通过下标运算符读入元素

    double sum = 0;
    for (int i = 0; i != v.size(); ++i)
        sum += v[i];                // 通过下标运算符计算元素之和
    return sum;
}
```

与结构体版本相比，类的使用者不再需要直接操作数据成员——所有访问都通过清晰的接口完成。此外，成员函数的定义可以放在类声明内部（隐式内联），也可以放在外部。例如：

```cpp
class Vector {
public:
    Vector(int s);
    double& operator[](int i);
    int size();
private:
    double* elem;
    int sz;
};

Vector::Vector(int s)
    : elem{new double[s]}, sz{s}
{
}

double& Vector::operator[](int i)
{
    return elem[i];
}

int Vector::size()
{
    return sz;
}
```

这种分离使得我们可以将接口（类声明）与实现（成员函数定义）放在不同的文件中，这正是大型程序组织的关键所在（[§3.2](../ch03/3-2-separate-compilation.md)）。

值得指出的是，我们的 `Vector` 目前还存在一个严重缺陷：它通过 `new` 分配了内存，但从未释放。这会导致*内存泄漏*（memory leak）。我们将在[§6.2](../ch06/6-2-copy-move.md)中讨论如何正确地管理资源。
