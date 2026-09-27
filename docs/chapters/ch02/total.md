# 用户定义类型

> **别慌！**
>
> —— Douglas Adams

# 2.1 引言

我们把那些能够由基本类型（§1.4）、常量修饰符（§1.6）以及声明操作符（§1.7）所构成的类型称为内置类型。C++所提供的内置类型和操作非常丰富，但它们都属于低阶层面的功能。

它们能够直接而高效地体现传统计算机硬件的功能。不过，它们无法为程序员提供便捷的手段来编写复杂的应用程序。相反，C++通过一系列复杂的抽象机制来扩展内置的数据类型和运算方式，程序员可以利用这些机制来构建出高级的应用程序。

C++的抽象机制旨在让程序员能够自行设计并实现各种数据类型，这些类型具有合适的表示方式与操作方式，同时程序员也能轻松而优雅地使用这些类型。利用C++的抽象机制从其他类型中派生出来的数据类型，被称为用户定义类型。这类类型包括类和枚举类型。用户定义类型既可以由内置类型构成，也可以由其他用户定义类型构成。本书的大部分内容都致力于介绍用户定义类型的定义、实现及使用方法。通常情况下，人们更倾向于使用用户定义类型，因为它们更易于使用、出错的可能性更低，而且其效率往往与直接使用内置类型相当，有时甚至更高。

本章的其余部分介绍了定义和使用类型时最简单、最基础的方法。第4章到第8章则更详细地阐述了各种抽象机制以及与之相关的编程方式。用户自定义类型是标准库的核心组成部分，因此，第9章到第17章通过实例展示了如何利用第1章到第8章中介绍的语言特性和编程技巧来构建各种程序。

# 2.2 结构体

构建一种新类型的第一步，通常是将其所需的各个元素组织成一个数据结构，也就是所谓的`struct`（结构体）：

```cpp
struct Vector {
    double* elem;   // 指向元素的指针
    int sz;         // 元素个数
};
```

`Vector`的第一个版本中，包含一个 `int` 和一个 `double*`。

`Vector` 类型的变量可以这样定义：

```cpp
Vector v;
```

不过，单独来看的话，这并没有太大用处，因为变量 `v` 的元素指针指向的是空值。要想让它发挥作用，就必须给 `v` 指向的元素指定一些它可以指向的元素。
例如，我们需要一种方式来初始化 `Vector`：

```cpp
void vector_init(Vector& v, int s)
{
    v.elem = new double[s]; // 分配一个包含 s 个 double 的数组
    v.sz = s;
}
```

也就是说，`v`的`elem`成员会得到一个由`new`运算符生成的指针，而`v`的`sz`成员则包含着向量中元素的个数。在`Vector&`中，`&`符号表示我们是以非常量引用[§1.7](../ch01/1-7-pointers-arrays.md)的方式传递`v`的；这样一来，`vector_init()`就可以修改被传递给它的向量了。

`new`操作符从被称为“自由存储区”的地方分配内存。所谓“自由存储区”，其实就是动态内存或堆内存。在自由存储区中分配的对象与其创建时的作用域无关，这些对象会一直存在，直到被`delete`操作符销毁为止（§5.2.2）。

`Vector` 的一个简单用法如下：

```cpp
double read_and_sum(int s)
    // 从 cin 读取 s 个整数，返回它们的和
{
    Vector v;
    vector_init(v, s);          // 为 v 的 s 个元素分配空间

    for (int i = 0; i != s; ++i)
        std::cin >> v.elem[i];  // 读入元素

    double sum = 0;
    for (int i = 0; i != s; ++i)
        sum += v.elem[i];       // 计算元素之和
    return sum;
}
```

要让我们的`Vector`变得像标准库中的向量那样优雅且易于使用，还有很长的路要走。尤其是，使用`Vector`的用户必须了解其与`Vector`相关的所有细节。本章的剩余内容以及接下来的两章，都将通过具体的例子来逐步改进`Vector`的性能，使其更符合语言特性和技术要求。第12章则介绍了标准库中的向量实现方式，其中包含了许多出色的改进之处。

我以`vector`以及其他标准库组件作为示例来进行说明，是为了：
- 说明语言特点与设计技巧，
- 帮助你学习和使用标准库中的各个组件。

对于`vector`和`string`这样的标准库组件，别造轮子，直接使用它们即可。标准库中的类型名称都是小写的，但为了区分那些用于说明设计和实现技巧的类型名称（比如`Vector`和`String`），我将其首字母大写。我们使用点号来通过名称或引用访问结构体成员，而使用箭头符号则可以通过指针来访问结构体成员。例如

```cpp
void f(Vector v, Vector& rv, Vector* pv)
{
    int i1 = v.sz;      // 通过变量名访问
    int i2 = rv.sz;     // 通过引用访问
    int i3 = pv->sz;    // 通过指针访问
}
```

# 2.3 类

将数据与其操作方式分开处理有其优点，比如可以以多种方式来使用这些数据。不过，如果要让用户定义的类型具备“真实类型”应有的所有特性，那么就必须在数据的表示形式与相关操作之间建立更紧密的联系。尤其是，我们通常希望让用户无法直接访问数据的表示形式，这样才能简化使用流程、确保数据的统一性，并便于日后对数据的表示方式进行改进。为此，我们必须区分类型的公共接口（供所有人使用）和其实现细节（只有实现层才能访问那些不可直接访问的数据）。语言本身就提供了实现这一目标的机制。这被称为“类”（class）。一个类包含了一组成员（member），这些成员可以是数据、函数或类型相关的元素。

一个类的接口由其公共成员（`public` member）来定义，而其私有成员（`private` member）则只能通过该类的接口（`interface`）来访问。在类声明中，公共成员和私有成员的排列顺序并不固定，但通常我们会将公共成员的声明放在前面，而将私有成员的声明放在后面。当然，如果我们想特别强调某种结构或关系的话，也可以打破这种惯例。例如：

```cpp
class Vector { 
public: 
        Vector(int s) :elem{new double[s]}, sz{s} { }    // construct a Vector 
        double& operator[](int i) { return elem[i]; }      // element access: subscripting 
        int size() { return sz; } 
private: 
        double* elem;  // pointer to the elements 

        int sz;               // the number of elements 
};
```

鉴于此，我们可以为新定义的“Vector”类型定义一个变量：

```cpp
Vector v(6);
```
我们可以用图形方式来展示`Vector`对象：

![alt text](Vector.png)

基本上，Vector对象可以看作是一种“handle”，它包含着指向元素（elem）的指针，同时还能指示出其中包含的元素数量。在示例中，元素数量为6，但不同`Vector`对象所包含的元素数量可能有所不同。此外，同一个`Vector`对象在不同时间点上所包含的元素数量也可能发生变化（§5.2.3节）。不过，`Vector`对象本身的大小是恒定不变的。这就是C++中处理不同数量数据的基本方式：使用固定大小的指针来引用“其他地方”存储的可变数量的数据。由`new`操作分配的内存空间；§5.2.2节。如何设计和使用这类对象是第5章的主要讨论内容。

在这里，`Vector`对象的相关信息（即其元素`elem`和大小`sz`）只能通过公共成员所提供的接口来获取：`Vector()`、`operator[]()`以及`size()`。而§2.2中的`read_and_sum()`示例可以简化为：

```cpp
double read_and_sum(int s) 
{ 
    Vector v(s);                                    // make a vector of s elements 
    for (int i=0; i!=v.size(); ++i) 
        cin>>v[i];                               // read into elements 

    double sum = 0; 
    for (int i=0; i!=v.size(); ++i) 
        sum+=v[i];                             // take the sum of the elements 

    return sum; 
}
```

和类具有相同名称的成员“函数（function）”叫做构造函数（constructor），也就是说，它是一种用于创建该类对象的函数。因此，构造函数`Vector()`就取代了§2.2中的`vector_init()`函数。与普通函数不同，构造函数一定会被用来初始化其所在类的对象。这样一来，定义构造函数就能避免类中出现变量未初始化的问题。
`Vector(int)`这个构造函数规定了如何创建`Vector`类型的对象。具体来说，它指出在创建对象时需要一个整数作为参数。这个整数用于确定向量中元素的个数。构造函数会利用成员初始化列表来对`Vector`对象的各个成员进行初始化。

```cpp
:elem{new double[s]}, sz{s}
```

也就是说，我们首先用从内存池中获取的、类型为双精度的`s`个元素的指针来初始化`elem`。接着，我们将`sz`初始化为`s`的值。

访问各个元素是通过下标函数来实现的，该函数的运算符为`operator[]`。该函数会返回对相应元素的引用，这个引用是一个双精度浮点型对象的引用(`double&`)，因此可以用来进行读写操作。

函数`size()`的作用是向用户告知元素的数量。

显然，这里完全没有错误处理机制，我们将在第4章中再讨论这个问题。同样，我们也没有提供任何方式来“归还”通过`new`操作创建的双精度浮点数(`double`)数组。§5.2.2章节介绍了如何通过定义析构函数来优雅地实现这一功能。

结构体(struct)与类(class)之间并没有本质上的区别；结构体只不过是一种默认情况下所有成员均为公共属性(`public`)属性的类而已。例如，你也可以为结构体定义构造函数和其他成员函数。

# 2.4 枚举

除了类之外，C++ 还支持一种简单的用户定义类型——*枚举*（enumeration），用于表示一组小范围的整数值：

```cpp
enum class Color { red, blue, green };
enum class Traffic_light { green, yellow, red };

Color col = Color::red; 
Traffic_light light = Traffic_light::red;
```

注意，枚举值（比如`red`）位于其`enum class`的作用域里， 因此可以在不同的`enum class`里重复出现，而且不会相互混淆。 例如：`Color::red`是`Color`里面的`red`，跟`Traffic_light::red`毫无关系。

枚举类型用于表示一组有限的整数值。使用枚举可以让代码更易于理解，也能减少出错的可能性。如果不使用这些有意义的枚举名称，代码就会变得难以理解，出错的概率也会增加。

在枚举类之后定义的规则表明，该枚举属于强类型枚举，其枚举值具有明确的范围。由于枚举值属于独立的类型，因此可以避免对常量的误用。特别是，我们不能将`Traffic_light`和`Color`这两种枚举值混在一起使用。

```cpp
Color x1 = red;                     // error: which red? 
Color y2 = Traffic_light::red;      // error: that red is not a Color 
Color z3 = Color::red;              // OK 
auto x4 = Color::red;               // OK: Color::red is a Color
```

同样，我们也不能将`Color`和整数值混合使用。

```cpp 
int i = Color::red; // 错误：Color::red不是int类型
Color c = 2;        // 初始化错误：2不是Color类型
```

捕获试图将某种类型转换为`enum`的操作，是一种有效的错误预防措施。不过，我们通常希望用该类型对应的底层类型的值来初始化枚举变量（默认情况下，底层类型为整数）。这种做法是允许的，同时，从底层类型进行显式转换也是可行的。

```cpp
Color x = Color{5}; // 可行，但略有些啰嗦
Color y {6};        // 同样可行
```

同样地，我们也可以将枚举值明确转换为相应的底层类型：

```cpp
int x = int(Color::red);
```

默认情况下，枚举类（`enum class`）具有赋值、初始化以及比较操作的功能（比如`==`和`<`运算符；§1.4节）。不过，枚举是一种用户自定义类型，因此我们可以为其定义额外的运算符（参见§6.4节）。

```cpp
Traffic_light& operator++(Traffic_light& t)
    // 前缀递增：++
{
    switch (t) {
    case Traffic_light::green:  return t = Traffic_light::yellow;
    case Traffic_light::yellow: return t = Traffic_light::red;
    case Traffic_light::red:    return t = Traffic_light::green;
    }
    return t;
}

auto signal = Traffic_light::red;
Traffic_light next = ++signal;   // 根据交通灯规则前进到下一个状态
```

如果反复使用`Traffic_light`这个名称显得过于繁琐，我们可以在特定的范围内将其缩写起来。

```cpp
Traffic_light& operator++(Traffic_light& t)  // prefix increment: ++ 
{ 
    using enum Traffic_light;         // here, we are using Traffic_light 

    switch (t) {
        switch (t) { 
        case green:      return t=yellow; 
        case yellow:     return t=red; 
        case red:          return t=green; 
        } 
    }
}
```
如果你不想显式指定枚举量的名称，并且希望枚举量直接以整数的形式存在（无需进行任何转换），那么你可以去掉`enum class`这个限定，从而得到一个“普通”的枚举类型。在这种模式下，枚举量的值会与其名称处于相同的作用域内，并且会自动转换为相应的整数值。例如：

```cpp
enum Color { red, green, blue }; 
int col = green;
```

在这里，`col`的值为`1`。默认情况下，枚举类型的整数值是从`0`开始递增的，每增加一个枚举项，其值就增加1。这种“简单”的枚举方式在C++和C语言中早已被使用。虽然这种枚举方式的性能稍差一些，但它在当前的代码中仍然很常见。

# 2.5 联合体

`union`（联合体）是一种特殊的结构体（`struct`），它的所有成员都分配在同一块内存地址上。因此，联合体所占用的空间大小取决于其中最大的那个成员变量所占用的空间。显然，联合体在任何时候只能保存其中一个成员变量的值。例如，假设有一个符号表条目，它用来存储一个名称和一个值。这个值可以是`Node*`类型的指针，也可以是`int`类型的整数。

```cpp
enum Type { ptr, num }; // 一个 Type 可以是ptr和num(§2.5)

struct Entry {
    string name;    // string是个标准库里的类型
    Type t;
    Node* p;        // 如果t==ptr，用p
    int i;          // 如果t==num，用i
};

void f(Entry* pe)
{
    if (pe->t == num)
        cout << pe->i;
    // ...
}
```

`p`和`i`这两个成员变量永远不会被同时使用，因此这种设计会浪费空间。只要指定它们都应属于同一个联合体，就能轻松解决这个问题，具体做法如下：

```cpp
union Value { 
    Node* p; 
    int i; 
};
```

现在，`Value::p`和`Value::i`被存储在每个`Value`对象相同的内存地址中。
这种空间优化方式对于那些需要处理大量数据的应用程序来说非常重要，因为在这种情况下，采用紧凑的数据存储方式就显得尤为关键。
该语言无法自动判断联合体中存储的是哪种类型的值，因此必须由程序员来负责判断。

```cpp
struct Entry { 
    string name; 
    Type t; 
    Value v;   // use v.p if t==Type::ptr; use v.i if t==Type::num 
}; 

void f(Entry* pe) 
{ 
    if (pe->t == Type::num) 
        cout << pe->v.i; 
    // ... 
}
```

保持类型字段与联合体中所存储的类型之间的对应关系其实相当容易出错。为了避免错误，我们可以将联合体及类型字段封装在一个类中，只有通过那些能够正确使用该联合体的成员函数，才能访问相关数据。在应用层面，这种抽象处理方式有助于避免错误的发生。依赖这种带有标记的联合体是一种常见且有效的做法。而使用“无标记”的联合体则应尽量避免。

标准库有个类型叫`std::variant`，使用它就可以避免绝大多数针对 `union` 的直接应用。`variant`存储一个值，该值的类型可以从一组类型中任选一个(§15.4.1)。 举个例子，`variant<Node*,int>`的值，可以是Node*或者int。

标准库中的`variant`类型可以用来避免大多数对`union`的直接使用。所谓`variant`，其实就是能够存储一组可选类型中某一种类型的值（§15.4.1）。例如，`variant<Node*,int>`这种类型可以存储`Node*`类型的值，也可以存储`int`类型的值。利用`variant`类型，原本需要用`union`来实现的代码，现在就可以用更简单的方式来实现了。比如，上面的`Entry`示例就可以这样改写：

```cpp
struct Entry {
    string name;
    variant<Node*,int> v;
};

void f(Entry* pe)
{
    if (holds_alternative<int>(pe->v))  // *pe的值是int类型吗？（参见§13.5.1）
        cout << get<int>(pe->v);        // 取（get）int值
    // ...
}
```

很多情况下，使用`variant`比`union`更简单也更安全。

# 2.6 建议

[1] 当内置类型过于底层时，优先使用定义良好的用户定义类型；§2.1。
[2] 将相关数据组织到结构（`struct` 或 `class`）中；§2.2；[CG: C.1]。
[3] 使用类表示接口与实现之间的区分；§2.3；[CG: C.3]。
[4] `struct` 只是默认成员为 `public` 的类；§2.3。
[5] 定义构造函数以保证并简化类的初始化；§2.3；[CG: C.2]。
[6] 使用枚举表示具名常量集合；§2.4；[CG: Enum.2]。
[7] 优先使用类枚举（`enum class`）而非“普通”枚举，以减少意外；§2.4；[CG: Enum.3]。
[8] 为枚举定义操作，以便安全、简单地使用；§2.4；[CG: Enum.4]。
[9] 避免“裸露的”联合体；将其与类型字段一起封装在类中；§2.5；[CG: C.181]。
[10] 优先使用 `std::variant`，而非“裸露的”联合体；§2.5。
