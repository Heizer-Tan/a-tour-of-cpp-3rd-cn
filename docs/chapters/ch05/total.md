好的，我将把您提供的《A Tour of C++》第5章 **Classes** 的完整英文内容翻译为中文。保留原文的章节结构、代码块、图片标记和引用标记，只翻译叙述性文字。下面是翻译结果：

---

===== 第 1 页 =====

5

# 类

这些类型并非“抽象”——它们和 int、float 一样真实。 – Doug McIlroy

- 引言
- 类
- 具体类型
  - 算术类型；容器；初始化容器
- 抽象类型
- 虚函数
- 类层次结构

===== 第 2 页 =====

  - 层次结构带来的好处；层次结构导航；避免资源泄漏
- 建议

### 5.1 引言

本章及接下来的三章旨在让您了解 C++ 对抽象和资源管理的支持，而不会深入过多细节：

- 本章非正式地介绍定义和使用新类型（用户定义类型）的方式。特别地，它展示了用于具体类、抽象类以及类层次结构的基本属性、实现技术和语言设施。
- 第 6 章介绍在 C++ 中具有定义意义的操作，例如构造函数、析构函数和赋值操作。它概述了将这些操作组合使用的规则，以控制对象的生命周期，并支持简单、高效且完整的资源管理。
- 第 7 章引入模板，作为一种用其他类型和算法来参数化类型和算法的机制。对用户定义类型和内置类型的计算被表示为函数，有时会泛化为函数模板和函数对象。
- 第 8 章概述了构成泛型编程基础的概念、技术和语言特性。重点在于定义和使用概念，以精确指定模板的接口并指导设计。同时引入变参模板，用于指定最通用、最灵活的接口。

这些是支持面向对象编程和泛型编程这两种编程风格的语言设施。第 9 至 18 章通过展示标准库设施及其使用示例来进一步阐述。

#### 5.1.1 类

C++ 的核心语言特性是**类**。类是一种用户定义类型，用于表示程序代码中的某个实体。每当我们的程序设计中有一个有用的概念、实体、数据集合等时，我们都会尝试将其表示为程序中的一个类，以便这个概念存在于代码中，而不仅仅存在于我们的头脑、设计文档或某些注释中。一个由精心挑选的一组类构建而成的程序，远比一个完全直接使用内置类型构建的程序更容易理解、更容易正确实现。特别地，类通常是库所提供的内容。

实际上，所有超越基本类型、运算符和语句的语言设施的存在，都是为了帮助定义更好的类，或者更方便地使用它们。所谓“更好”，我指的是更正确、更易于维护、更高效、更优雅、更易于使用、更易于阅读以及更易于推理。大多数编程技术都依赖于特定种类的类的设计和实现。程序员的需求和品味千差万别，因此对类的支持非常广泛。在这里，我们将考虑三种重要类的基本支持：

- **具体类**（§5.2）
- **抽象类**（§5.3）
- **类层次结构中的类**（§5.5）

数量惊人的有用类实际上都属于这三种类型之一。更多的类可以被视为这些类型的简单变体，或者是通过组合使用这些类型所采用的技术来实现的。

### 5.2 具体类型

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

### 5.3 抽象类型

像 `complex` 和 `Vector` 这样的类型被称为**具体类型**，因为它们的表示是其定义的一部分。在这方面，它们类似于内置类型。相比之下，**抽象类型**是一种将用户与实现细节完全隔离的类型。为此，我们将接口与表示分离，并放弃了真正的局部变量。由于我们对抽象类型的表示一无所知（甚至不知道它的大小），我们必须将对象分配在自由存储区（§5.2.2）上，并通过引用或指针（§1.7, §15.2.1）来访问它们。

首先，我们定义一个类 `Container` 的接口，我们将其设计为 `Vector` 的一个更抽象的版本：

```cpp
class Container {
public:
    virtual double& operator[](int) = 0;   // 纯虚函数
    virtual int size() const = 0;          // const 成员函数（§5.2.1）
    virtual ~Container() {}                // 析构函数（§5.2.2）
};
```

这个类是一个纯接口，供之后定义的具体容器使用。关键字 `virtual` 的意思是“可以在从此类派生的类中重新定义”。不出所料，被声明为 `virtual` 的函数称为**虚函数**。从 `Container` 派生的类为 `Container` 接口提供实现。奇怪的 `=0` 语法表示该函数是**纯虚函数**；也就是说，某个从 `Container` 派生的类必须定义该函数。因此，不可能定义一个仅仅是 `Container` 的对象。例如：

```cpp
Container c;                // 错误：不能有抽象类的对象
Container* p = new Vector_container(10);   // OK：Container 是 Vector_container 的接口
```

`Container` 只能作为实现了其 `operator[]()` 和 `size()` 函数的类的接口。包含纯虚函数的类称为**抽象类**。

这个 `Container` 可以这样使用：

```cpp
void use(Container& c)
{
    const int sz = c.size();
    for (int i = 0; i != sz; ++i)
        cout << c[i] << '\n';
}
```

注意 `use()` 如何使用 `Container` 接口，而完全不知道实现细节。它使用 `size()` 和 `[]`，却不知道具体是哪个类型提供了它们的实现。一个为多种其他类提供接口的类通常被称为**多态类型**。

与抽象类的常见情况一样，`Container` 没有构造函数。毕竟，它没有任何需要初始化的数据。另一方面，`Container` 确实有一个析构函数，并且该析构函数是虚函数，以便从 `Container` 派生的类可以提供实现。同样，这对抽象类是常见的，因为它们往往通过引用或指针来操作，而通过指针销毁 `Container` 的人不知道其实现拥有哪些资源；另见 §5.5。

抽象类 `Container` 只定义了一个接口，没有实现。要使 `Container` 有用，我们必须实现一个容器，提供其接口所要求的函数。为此，我们可以使用具体类 `Vector`：

```cpp
class Vector_container : public Container {   // Vector_container 实现了 Container 接口
public:
    Vector_container(int s) : v(s) { }        // 包含 s 个元素的 Vector
    ~Vector_container() {}

    double& operator[](int i) override { return v[i]; }
    int size() const override { return v.size(); }
private:
    Vector v;
};
```

`: public` 可以读作“派生自”或“是……的子类型”。`Vector_container` 类被称为**派生自** `Container` 类，而 `Container` 类被称为 `Vector_container` 的**基类**。另一种术语分别称 `Vector_container` 和 `Container` 为**子类**和**超类**。派生类被说成从它的基类**继承**成员，因此基类和派生类的使用通常被称为**继承**。

成员 `operator[]()` 和 `size()` 被称为**覆盖**（override）了基类 `Container` 中的对应成员。我使用了显式的 `override` 来明确表达意图。使用 `override` 是可选的，但显式使用可以让编译器捕获错误，例如函数名的拼写错误或虚函数类型与其预期覆盖者之间的微小差异。显式使用 `override` 在较大的类层次结构中尤其有用，否则可能很难知道哪个函数应该覆盖哪个。

析构函数（`~Vector_container()`）覆盖了基类析构函数（`~Container()`）。注意，成员析构函数（`~Vector()`）由其类的析构函数（`~Vector_container()`）隐式调用。

对于一个像 `use(Container&)` 这样完全不知道实现细节就能使用 `Container` 的函数，必须有另一个函数来创建一个它能够操作的对象。例如：

```cpp
void g()
{
    Vector_container vc(10);   // 包含十个元素的 Vector
    // ... 填充 vc ...
    use(vc);
}
```

由于 `use()` 不知道 `Vector_container`，只知道 `Container` 接口，因此对于 `Container` 的不同实现，它同样能正常工作。例如：

```cpp
class List_container : public Container {        // List_container 实现了 Container 接口
public:
    List_container() { }                         // 空 List
    List_container(initializer_list<double> il) : ld{il} { }
    ~List_container() {}

    double& operator[](int i) override;
    int size() const override { return ld.size(); }

private:
    std::list<double> ld;   // 标准库的 double 列表（§12.3）
};

double& List_container::operator[](int i)
{
    for (auto& x : ld) {
        if (i == 0)
            return x;
        --i;
    }
    throw out_of_range{"List container"};
}
```

这里，表示是一个标准库的 `list<double>`。通常，我不会用 `list` 来实现带下标操作的容器，因为相对于 `vector` 的下标操作，`list` 的下标性能非常差。但这里我只是想展示一个与常规实现截然不同的实现。

一个函数可以创建一个 `List_container` 并让 `use()` 使用它：

```cpp
void h()
{
    List_container lc = {1, 2, 3, 4, 5, 6, 7, 8, 9};
    use(lc);
}
```

关键在于，`use(Container&)` 不知道它的参数是 `Vector_container`、`List_container` 还是其他某种容器；它也不需要知道。它可以**使用任何类型的 `Container`**。它只知道 `Container` 定义的接口。因此，如果 `List_container` 的实现发生了变化，或者使用了一个全新的从 `Container` 派生的类，`use(Container&)` 也不需要重新编译。

这种灵活性的另一面是，对象必须通过指针或引用来操作（§6.2, §15.2.1）。

### 5.4 虚函数

再次考虑 `Container` 的使用：

```cpp
void use(Container& c)
{
    const int sz = c.size();
    for (int i = 0; i != sz; ++i)
        cout << c[i] << '\n';
}
```

`use()` 中的调用 `c[i]` 是如何解析到正确的 `operator[]()` 的？当 `h()` 调用 `use()` 时，必须调用 `List_container` 的 `operator[]()`。当 `g()` 调用 `use()` 时，必须调用 `Vector_container` 的 `operator[]()`。为了实现这种解析，`Container` 对象必须包含信息，以便在运行时选择正确的函数来调用。通常的实现技术是让编译器将虚函数的名字转换为一个指向函数指针表的索引。该表通常称为**虚函数表**或简称为 **vtbl**。每个包含虚函数的类都有自己的 vtbl，标识其虚函数。这可以用图形表示如下：

[图片描述：Vector_container 及其 vtbl 图示]
[图片描述：Container 对象指向 vtbl 的指针]

vtbl 中的函数使得即使调用者不知道对象的大小和数据布局，也能正确使用对象。调用者的实现只需要知道 `Container` 中指向 vtbl 的指针的位置以及每个虚函数使用的索引。这种虚调用机制可以做到几乎与“普通函数调用”机制一样高效（在 25% 以内，并且对于同一对象的重复调用远远更便宜）。它的空间开销是每个具有虚函数的类对象中一个指针，再加上每个这样的类一个 vtbl。

### 5.5 类层次结构

`Container` 示例是一个非常简单的类层次结构示例。类层次结构是一组通过派生（例如 `: public`）排序的类构成的网格。我们使用类层次结构来表示具有层次关系的概念，例如“消防车是一种卡车，卡车是一种车辆”以及“笑脸是一种圆形，圆形是一种形状”。包含数百个类的、既深又广的大型层次结构很常见。作为一个半真实的经典例子，让我们考虑屏幕上的形状：

[图片描述：形状层次结构图，Shape 派生出 Circle、Triangle 等，Circle 派生出 Smiley]

箭头表示继承关系。例如，`Circle` 类派生自 `Shape` 类。类层次结构通常从最基本的类（根）向下（朝向后面定义的派生类）绘制。为了在代码中表示这个简单的图，我们首先必须定义一个指定所有形状通用属性的类：

```cpp
class Shape {
public:
    virtual Point center() const = 0;   // 纯虚函数
    virtual void move(Point to) = 0;
    virtual void draw() const = 0;      // 在当前“画布”上绘制
    virtual void rotate(int angle) = 0;
    virtual ~Shape() {}                 // 析构函数
    // ...
};
```

自然，这个接口是一个抽象类：就表示而言，没有什么是所有 `Shape` 共有的（除了指向 vtbl 的指针的位置）。给定这个定义，我们可以编写操作指向形状的指针向量的通用函数：

```cpp
void rotate_all(vector<Shape*>& v, int angle)   // 将 v 的每个元素旋转 angle 度
{
    for (auto p : v)
        p->rotate(angle);
}
```

要定义一个特定的形状，我们必须说明它是一个 `Shape` 并指定其特定属性（包括其虚函数）：

```cpp
class Circle : public Shape {
public:
    Circle(Point p, int rad) : x{p}, r{rad} { }   // 构造函数

    Point center() const override { return x; }
    void move(Point to) override { x = to; }
    void draw() const override;
    void rotate(int) override {}   // 简单且漂亮的算法

private:
    Point x;   // 圆心
    int r;     // 半径
};
```

到目前为止，`Shape` 和 `Circle` 的示例与 `Container` 和 `Vector_container` 的示例相比并没有提供新内容，但我们可以进一步构建：

```cpp
class Smiley : public Circle {   // 以圆形作为脸的基础
public:
    Smiley(Point p, int rad) : Circle{p, rad}, mouth{nullptr} { }

    ~Smiley()
    {
        delete mouth;
        for (auto p : eyes)
            delete p;
    }

    void move(Point to) override;
    void draw() const override;
    void rotate(int) override;

    void add_eye(Shape* s) { eyes.push_back(s); }
    void set_mouth(Shape* s);
    virtual void wink(int i);   // 眨眼第 i 只眼睛

private:
    vector<Shape*> eyes;   // 通常两只眼睛
    Shape* mouth;
};
```

`vector` 的 `push_back()` 成员将其参数复制到向量中（此处为 `eyes`）作为最后一个元素，并将该向量的大小增加一。

我们现在可以使用对 `Smiley` 的基类和成员 `draw()` 的调用来定义 `Smiley::draw()`：

```cpp
void Smiley::draw() const
{
    Circle::draw();
    for (auto p : eyes)
        p->draw();
    mouth->draw();
}
```

注意 `Smiley` 将其眼睛保存在标准库的 `vector` 中，并在其析构函数中删除它们。`Shape` 的析构函数是虚函数，`Smiley` 的析构函数覆盖了它。对于抽象类来说，虚析构函数至关重要，因为派生类的对象通常通过其抽象基类提供的接口来操作。特别地，它可能通过指向基类的指针被删除。然后，虚函数调用机制确保调用正确的析构函数。该析构函数随后隐式调用其基类和成员的析构函数。

在这个简化的示例中，程序员的任务是将眼睛和嘴巴适当地放置在代表脸部的圆内。

当我们通过派生定义新类时，可以添加数据成员、操作或两者。这带来了极大的灵活性，同时也带来了混淆和糟糕设计的机会。

#### 5.5.1 层次结构带来的好处

类层次结构提供两种好处：

- **接口继承**：派生类的对象可以在任何需要基类对象的地方使用。也就是说，基类充当派生类的接口。`Container` 和 `Shape` 类就是例子。此类类通常是抽象类。
- **实现继承**：基类提供简化派生类实现的函数或数据。`Smiley` 使用 `Circle` 的构造函数和 `Circle::draw()` 就是例子。此类基类通常有数据成员和构造函数。

具体类——尤其是表示较小的类——非常像内置类型：我们将其定义为局部变量，通过名称访问它们，拷贝它们等。类层次结构中的类则不同：我们倾向于使用 `new` 将它们分配在自由存储区上，并通过指针或引用访问它们。例如，考虑一个从输入流读取形状描述并构造相应 `Shape` 对象的函数：

```cpp
enum class Kind { circle, triangle, smiley };

Shape* read_shape(istream& is)   // 从输入流 is 读取形状描述
{
    // ... 从 is 读取形状头部并找到其 Kind k ...
    switch (k) {
    case Kind::circle:
        // ... 将圆数据 {Point,int} 读入 p 和 r ...
        return new Circle{p, r};
    case Kind::triangle:
        // ... 将三角形数据 {Point,Point,Point} 读入 p1, p2, p3 ...
        return new Triangle{p1, p2, p3};
    case Kind::smiley:
        // ... 将笑脸数据 {Point,int,Shape,Shape,Shape} 读入 p, r, e1, e2, m ...
        Smiley* ps = new Smiley{p, r};
        ps->add_eye(e1);
        ps->add_eye(e2);
        ps->set_mouth(m);
        return ps;
    }
}
```

一个程序可以这样使用这个形状读取器：

```cpp
void user()
{
    std::vector<Shape*> v;
    while (cin)
        v.push_back(read_shape(cin));
    draw_all(v);        // 对每个元素调用 draw()
    rotate_all(v, 45);  // 对每个元素调用 rotate(45)
    for (auto p : v)    // 记得删除元素
        delete p;
}
```

显然，这个例子被简化了——尤其是在错误处理方面——但它生动地说明了 `user()` 完全不知道它操作的是哪种形状。`user()` 代码可以编译一次，然后以后用于添加到程序中的新 `Shape`。注意，在 `user()` 之外没有指向这些形状的指针，因此 `user()` 负责释放它们。这是通过 `delete` 运算符完成的，并且关键依赖于 `Shape` 的虚析构函数。因为该析构函数是虚函数，`delete` 会调用最派生类的析构函数。这一点至关重要，因为派生类可能已经获取了各种资源（如文件句柄、锁和输出流），这些资源需要被释放。在这种情况下，`Smiley` 会删除它的眼睛和嘴巴对象。完成后，它会调用 `Circle` 的析构函数。对象由构造函数“自底向上”（先基类）构造，由析构函数“自顶向下”（先派生类）销毁。

#### 5.5.2 层次结构导航

`read_shape()` 函数返回 `Shape*`，以便我们可以统一对待所有 `Shape`。但是，如果我们想使用仅由特定派生类（如 `Smiley` 的 `wink()`）提供的成员函数，该怎么办？我们可以使用 `dynamic_cast` 运算符询问“这个 `Shape` 是一种 `Smiley` 吗？”：

```cpp
Shape* ps {read_shape(cin)};
if (Smiley* p = dynamic_cast<Smiley*>(ps)) {   // ps 指向一个 Smiley 吗？
    // ... 是一个 Smiley，使用它 ...
}
else {
    // ... 不是 Smiley，尝试其他 ...
}
```

如果在运行时 `dynamic_cast` 的参数（此处为 `ps`）指向的对象不是预期类型（此处为 `Smiley`）或不是从预期类型派生的类，则 `dynamic_cast` 返回 `nullptr`。

当指向不同派生类的对象的指针是有效参数时，我们使用 `dynamic_cast` 到指针类型。然后我们测试结果是否为 `nullptr`。这个测试通常可以方便地放在条件的变量初始化中。

当不同类型不可接受时，我们可以简单地 `dynamic_cast` 到引用类型。如果对象不是预期类型，`dynamic_cast` 会抛出 `bad_cast` 异常：

```cpp
Shape* ps {read_shape(cin)};
Smiley& r {dynamic_cast<Smiley&>(ps)};   // 在某个地方捕获 std::bad_cast
```

谨慎使用 `dynamic_cast` 时代码更清晰。如果我们能避免在运行时测试类型信息，就可以编写更简单、更高效的代码，但偶尔类型信息会丢失且必须恢复。这通常发生在我们将对象传递给某个接受基类指定接口的系统时。当该系统稍后将对象传递回给我们时，我们可能必须恢复原始类型。类似于 `dynamic_cast` 的操作被称为“is kind of”和“is instance of”操作。

#### 5.5.3 避免资源泄漏

**泄漏**是获取资源后未能释放资源的传统术语。必须避免泄漏资源，因为泄漏会使资源对系统不可用。因此，泄漏最终可能导致系统因所需资源耗尽而变慢甚至崩溃。

有经验的程序员会注意到，我在 `Smiley` 示例中留下了三个可能出错的地方：

- `Smiley` 的实现者可能未能删除指向 `mouth` 的指针。
- `read_shape()` 的用户可能未能删除返回的指针。
- 持有 `Shape` 指针的容器的所有者可能未能删除指向的对象。

从这个意义上说，指向自由存储区分配的对象的指针是危险的：“普通的旧指针”不应被用来表示所有权。例如：

```cpp
void user(int x)
{
    Shape* p = new Circle{Point{0,0}, 10};
    // ...
    if (x < 0) throw Bad_x{};   // 可能泄漏
    if (x == 0) return;         // 可能泄漏
    // ...
    delete p;
}
```

除非 `x` 是正数，否则这将泄漏。将 `new` 的结果赋给“裸露的指针”是在自找麻烦。

解决此类问题的一个简单方案是：在需要删除时使用标准库的 `unique_ptr`（§15.2.1），而不是“裸露的指针”：

```cpp
class Smiley : public Circle {
    // ...
private:
    vector<unique_ptr<Shape>> eyes;   // 通常两只眼睛
    unique_ptr<Shape> mouth;
};
```

这是一个简单、通用且高效的资源管理技术示例（§6.3）。

作为这一改变的一个令人愉快的副作用，我们不再需要为 `Smiley` 定义析构函数。编译器将隐式生成一个析构函数，它会在 `vector` 中执行所需的 `unique_ptr` 销毁（§6.3）。使用 `unique_ptr` 的代码将与正确使用原始指针的代码一样高效。

现在考虑 `read_shape()` 的用户：

```cpp
unique_ptr<Shape> read_shape(istream& is)   // 从输入流 is 读取形状描述
{
    // ... 从 is 读取形状头部并找到其 Kind k ...
    switch (k) {
    case Kind::circle:
        // ... 将圆数据 {Point,int} 读入 p 和 r ...
        return unique_ptr<Shape>(new Circle{p, r});   // §15.2.1
    // ...
    }
}

void user()
{
    vector<unique_ptr<Shape>> v;
    while (cin)
        v.push_back(read_shape(cin));

    draw_all(v);        // 对每个元素调用 draw()
    rotate_all(v, 45);  // 对每个元素调用 rotate(45)
    // 所有 Shape 都被隐式销毁
}
```

现在每个对象都由一个 `unique_ptr` 拥有，当不再需要时（即当其 `unique_ptr` 超出作用域时），`unique_ptr` 会删除该对象。

为了使 `user()` 的 `unique_ptr` 版本工作，我们需要接受 `vector<unique_ptr<Shape>>` 的 `draw_all()` 和 `rotate_all()` 版本。编写许多这样的 `_all()` 函数可能会变得乏味，因此 §7.3.2 展示了另一种方法。

## 5.6 建议

[1] 直接在代码中表达想法；§5.1；[CG: P.1]。
[2] 具体类是最简单的类。在适用的情况下，优先选择具体类而非更复杂的类以及普通数据结构；§5.2；[CG: C.10]。
[3] 使用具体类表示简单概念；§5.2。
[4] 对于性能关键的组件，优先选择具体类而非类层次结构；§5.2。
[5] 定义构造函数来处理对象的初始化；§5.2.1，§6.1.1；[CG: C.40] [CG: C.41]。
[6] 仅当函数需要直接访问类的表示时才将其设为成员；§5.2.1；[CG: C.4]。
[7] 定义运算符主要是为了模仿常规用法；§5.2.1；[CG: C.160]。
[8] 对对称运算符使用非成员函数；§5.2.1；[CG: C.161]。
[9] 将不修改对象状态的成员函数声明为 `const`；§5.2.1。
[10] 如果构造函数获取了资源，则其类需要析构函数来释放资源；§5.2.2；[CG: C.20]。
[11] 避免“裸露的” `new` 和 `delete` 操作；§5.2.2；[CG: R.11]。
[12] 使用资源句柄和 RAII 来管理资源；§5.2.2；[CG: R.1]。
[13] 如果一个类是容器，则为其提供初始化列表构造函数；§5.2.3；[CG: C.103]。
[14] 当需要完全分离接口和实现时，使用抽象类作为接口；§5.3；[CG: C.122]。
[15] 通过指针和引用访问多态对象；§5.3。
[16] 抽象类通常不需要构造函数；§5.3；[CG: C.126]。
[17] 使用类层次结构来表示具有固有层次结构的概念；§5.5。
[18] 包含虚函数的类应具有虚析构函数；§5.5；[CG: C.127]。
[19] 在大型类层次结构中使用 `override` 来明确覆盖关系；§5.3；[CG: C.128]。
[20] 设计类层次结构时，区分实现继承和接口继承；§5.5.1；[CG: C.129]。
[21] 在无法避免类层次结构导航时使用 `dynamic_cast`；§5.5.2；[CG: C.146]。
[22] 当未能找到所需类被视为失败时，使用 `dynamic_cast` 到引用类型；§5.5.2；[CG: C.147]。
[23] 当未能找到所需类被视为有效替代时，使用 `dynamic_cast` 到指针类型；§5.5.2；[CG: C.148]。
[24] 使用 `unique_ptr` 或 `shared_ptr` 避免忘记删除使用 `new` 创建的对象；§5.5.3；[CG: C.149]。

---

上面是第 5 章《Classes》的完整中文翻译。如果您还需要翻译其他章节，请提供相应的文本内容，我可以继续为您翻译。