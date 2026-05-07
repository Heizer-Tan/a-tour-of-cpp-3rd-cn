# 5.2 具体类型

具体类的基本思想是：它们的行为“就像内置类型一样”。例如，复数类型和任意精度整数非常类似于内置的 `int`，当然它们有自己的语义和操作集。同样，`vector` 和 `string` 也非常类似于内置数组，只是它们更灵活、行为更良好（§10.2, §11.3, §12.2）。

具体类型的定义性特征是：**其表示是其定义的一部分**。在许多重要情况下（例如 `vector`），该表示仅仅是一个或多个指向存储在别处的数据的指针，但该表示存在于具体类的每个对象中。这使得实现能够在时间和空间上达到最优效率。特别地，它允许我们：

- 将具体类型的对象放在栈上、静态分配的内存中以及其他对象中（§1.5）。
- 直接引用对象（而不仅仅通过指针或引用）。
- 立即且完整地初始化对象（例如使用构造函数；§2.3）。
- 拷贝和移动对象（§6.2）。

表示可以是私有的，并且只能通过成员函数访问（如 `vector` 那样；§2.3），但它确实存在。因此，如果表示发生任何重大变化，用户必须重新编译。这是让具体类型的行为完全像内置类型一样所付出的代价。对于那些不常更改的类型，以及局部变量能提供急需的清晰度和效率的情况，这是可以接受的，而且通常是理想的。为了增加灵活性，具体类型可以将其表示的主要部分保存在自由存储区（动态内存、堆）中，并通过存储在类对象本身中的部分来访问它们。这就是 `vector` 和 `string` 的实现方式；它们可以被视为具有精心设计的接口的资源句柄。

#### 5.2.1 算术类型

“经典的用户定义算术类型”是 `complex`：

```cpp
class complex {
    double re, im;   // 表示：两个 double
public:
    complex(double r, double i) : re{r}, im{i} {}       // 从两个标量构造复数
    complex(double r) : re{r}, im{0} {}                 // 从一个标量构造复数
    complex() : re{0}, im{0} {}                         // 默认复数：{0,0}
    complex(complex z) : re{z.re}, im{z.im} {}          // 拷贝构造函数

    double real() const { return re; }
    void real(double d) { re = d; }
    double imag() const { return im; }
    void imag(double d) { im = d; }

    complex& operator+=(complex z)
    {
        re += z.re;        // 加到 re 和 im
        im += z.im;
        return *this;      // 返回结果
    }

    complex& operator-=(complex z)
    {
        re -= z.re;
        im -= z.im;
        return *this;
    }

    complex& operator*=(complex);   // 在别处定义
    complex& operator/=(complex);   // 在别处定义
};
```

这是标准库 `complex`（§17.4）的简化版本。类定义本身只包含需要访问表示的操作。表示简单且传统。出于实际原因，它必须与 60 年前 Fortran 提供的内容兼容，并且我们需要一组传统的操作符。除了逻辑上的要求，`complex` 必须高效，否则它将无人使用。这意味着简单的操作必须被内联。也就是说，简单的操作（如构造函数、`+=` 和 `imag()`）必须在生成的机器码中不产生函数调用。在类中定义的函数默认是内联的。也可以通过在前缀加上关键字 `inline` 来显式请求内联。工业强度的 `complex`（如标准库中的那个）经过精心实现以进行适当的内联。此外，标准库的 `complex` 将这里展示的函数声明为 `constexpr`，以便我们可以在编译时进行复数算术。

拷贝赋值和拷贝初始化是隐式定义的（§6.2）。可以不带参数调用的构造函数称为**默认构造函数**。因此，`complex()` 是 `complex` 的默认构造函数。通过定义默认构造函数，你消除了该类类型的变量未初始化的可能性。

在返回实部和虚部的函数上的 `const` 限定符表明这些函数不会修改调用它们的对象。`const` 成员函数可以用于 `const` 对象和 `non-const` 对象，但非 `const` 成员函数只能用于 `non-const` 对象。例如：

```cpp
complex z = {1, 0};
const complex cz {1, 3};
z = cz;           // OK：赋值给非 const 变量
cz = z;           // 错误：赋值给 const
double x = z.real();   // OK：complex::real() 是 const
```

许多有用的操作不需要直接访问 `complex` 的表示，因此它们可以在类定义之外单独定义：

```cpp
complex operator+(complex a, complex b) { return a += b; }
complex operator-(complex a, complex b) { return a -= b; }
complex operator-(complex a) { return {-a.real(), -a.imag()}; }   // 一元负号
complex operator*(complex a, complex b) { return a *= b; }
complex operator/(complex a, complex b) { return a /= b; }
```

这里，我利用了按值传递的参数会被拷贝的事实，这样我就可以修改参数而不影响调用者的副本，并将结果用作返回值。

`==` 和 `!=` 的定义很直接：

```cpp
bool operator==(complex a, complex b) { return a.real() == b.real() && a.imag() == b.imag(); }
bool operator!=(complex a, complex b) { return !(a == b); }
```

`complex` 类可以这样使用：

```cpp
void f(complex z)
{
    complex a {2.3};                 // 从 2.3 构造 {2.3,0.0}
    complex b {1 / a};
    complex c {a + z * complex{1, 2.3}};
    if (c != b)
        c = -(b / a) + 2 * b;
}
```

编译器将涉及复数的操作符转换为适当的函数调用。例如，`c != b` 意味着 `operator!=(c, b)`，`1/a` 意味着 `operator/(complex{1}, a)`。

用户定义的操作符（“重载操作符”）应谨慎且符合惯例地使用（§6.4）。语法由语言固定，因此不能定义一元 `/`。此外，不能更改内置类型操作符的含义，因此不能重新定义 `+` 来使 `int` 相减。

#### 5.2.2 容器

**容器**是持有元素集合的对象。我们称 `Vector` 类为容器，因为 `Vector` 类型的对象就是容器。如 §2.3 中所定义的，`Vector` 作为一个 `double` 的容器并非不合理：它易于理解，建立了一个有用的不变式（§4.3），提供了范围检查的访问（§4.2），并提供了 `size()` 以允许我们遍历其元素。然而，它有一个致命缺陷：它使用 `new` 分配元素，但从不释放它们。这不是一个好主意，因为 C++ 不提供垃圾回收器来使未使用的内存对新对象可用。在某些环境中，你不能使用回收器，或者出于逻辑或性能原因，你更希望对销毁进行更精确的控制。我们需要一种机制来确保构造函数分配的内存被释放；这种机制就是**析构函数**：

```cpp
class Vector {
public:
    Vector(int s) : elem{new double[s]}, sz{s}   // 构造函数：获取资源
    {
        for (int i = 0; i != s; ++i)             // 初始化元素
            elem[i] = 0;
    }

    ~Vector() { delete[] elem; }                 // 析构函数：释放资源

    double& operator[](int i);
    int size() const;
private:
    double* elem;   // elem 指向一个包含 sz 个 double 的数组
    int sz;
};
```

析构函数的名称是补运算符 `~` 后跟类名；它是构造函数的补。

`Vector` 的构造函数使用 `new` 运算符在自由存储区（也称为堆或动态内存）上分配一些内存。析构函数通过使用 `delete[]` 运算符释放该内存来清理。普通的 `delete` 删除单个对象；`delete[]` 删除数组。

这一切都是在 `Vector` 的用户不干预的情况下完成的。用户就像使用内置类型的变量一样创建和使用 `Vector`。例如：

```cpp
Vector gv(10);                      // 全局变量；gv 在程序结束时被销毁

Vector* gp = new Vector(100);       // 自由存储区上的 Vector；永远不会隐式销毁

void fct(int n)
{
    Vector v(n);
    // ... 使用 v ...
    {
        Vector v2(2 * n);
        // ... 使用 v 和 v2 ...
    }   // v2 在此处被销毁
    // ... 使用 v ...
}   // v 在此处被销毁
```

`Vector` 遵循与内置类型（如 `int` 和 `char`）相同的命名、作用域、分配、生命周期等规则（§1.5）。这个 `Vector` 为了简化而省略了错误处理；参见 §4.4。

构造函数/析构函数组合是许多优雅技术的基础。特别是，它是大多数 C++ 通用资源管理技术的基础（§6.3, §15.2.1）。考虑 `Vector` 的图形示意：

构造函数分配元素并适当地初始化 `Vector` 成员。析构函数释放元素。这种**句柄到数据**的模型非常常用于管理在对象生命周期内大小可能变化的数据。在构造函数中获取资源并在析构函数中释放资源的技术称为**资源获取即初始化**（RAII）。它使我们能够消除“裸露的 `new` 操作”，即避免在通用代码中进行分配，而将它们隐藏在行为良好的抽象的实现内部。同样，应避免“裸露的 `delete` 操作”。避免裸露的 `new` 和裸露的 `delete` 使代码更不容易出错，并且更容易保持无资源泄漏（§15.2.1）。

#### 5.2.3 初始化容器

容器存在的目的是容纳元素，因此显然我们需要方便的方法将元素放入容器。我们可以创建一个具有适当数量元素的 `Vector`，然后再赋值，但通常其他方式更优雅。这里我只提及两种最喜欢的方式：

- **初始化列表构造函数**：用元素列表初始化。
- **`push_back()`**：在序列的末尾（后面）添加一个新元素。

可以这样声明：

```cpp
class Vector {
public:
    Vector();                                               // 默认初始化为“空”
    Vector(std::initializer_list<double>);                  // 用 double 列表初始化
    // ...
    void push_back(double);                                 // 在末尾添加元素，增加大小
    // ...
};
```

`push_back()` 对于输入任意数量的元素非常有用。例如：

```cpp
Vector read(istream& is)
{
    Vector v;
    for (double d; is >> d; )      // 读取浮点值到 d
        v.push_back(d);            // 将 d 添加到 v
    return v;
}
```

输入循环由文件结束或格式错误终止。在此之前，每个读取的数字都被添加到 `Vector` 中，因此最后 `v` 的大小就是读取的元素数量。我使用了 `for` 语句而不是更传统的 `while` 语句，以将 `d` 的作用域限制在循环内。

从 `read()` 返回潜在大量的数据可能会很昂贵。保证返回 `Vector` 廉价的方式是为其提供**移动构造函数**（§6.2.2）：

```cpp
Vector v = read(cin);   // 这里没有拷贝 Vector 的元素
```

`std::vector` 的表示方式以及使 `push_back()` 和其他改变 `vector` 大小的操作高效的方法在 §12.2 中介绍。

用于定义初始化列表构造函数的 `std::initializer_list` 是编译器知道的标准库类型：当我们使用 `{}` 列表（如 `{1,2,3,4}`）时，编译器将创建一个 `initializer_list` 类型的对象传递给程序。因此，我们可以这样写：

```cpp
Vector v1 = {1, 2, 3, 4, 5};          // v1 有 5 个元素
Vector v2 = {1.23, 3.45, 6.7, 8};     // v2 有 4 个元素
```

`Vector` 的初始化列表构造函数可以这样定义：

```cpp
Vector::Vector(std::initializer_list<double> lst)   // 用列表初始化
    : elem{new double[lst.size()]}, sz{static_cast<int>(lst.size())}
{
    copy(lst.begin(), lst.end(), elem);   // 从 lst 拷贝到 elem（§13.5）
}
```

不幸的是，标准库对大小和下标使用无符号整数，因此我们需要使用丑陋的 `static_cast` 来显式地将初始化列表的大小转换为 `int`。这是迂腐的，因为手写列表中的元素个数大于最大整数（16 位整数为 32,767，32 位整数为 2,147,483,647）的可能性相当低。然而，类型系统没有常识。它知道变量的可能值，而不是实际值，因此它可能会在没有实际违反的情况下发出警告。这种警告偶尔可以拯救程序员免于严重错误。

`static_cast` **不检查**它正在转换的值；程序员被认为会正确使用它。这并不总是一个好的假设，因此如果有疑问，请检查该值。最好避免显式类型转换（通常称为“强制转换”以提醒你它们用于支撑某些破损的东西）。尽量仅在系统的最底层使用未经检查的强制转换。它们容易出错。

其他强制转换包括 `reinterpret_cast` 和 `bit_cast`（§16.7）（用于将对象视为简单的字节序列）以及 `const_cast`（用于“去掉 const”）。明智地使用类型系统和精心设计的库，我们可以消除高层软件中的未经检查的强制转换。
