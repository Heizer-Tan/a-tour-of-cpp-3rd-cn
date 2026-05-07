# 7.2 参数化类型

我们可以将 §5.2.2 中的 `vector-of-doubles` 类型推广为 `vector-of-anything` 类型，方法是将其变为模板，并将具体的 `double` 类型替换为类型参数。例如：

```cpp
template<typename T>
class Vector {
private:
    T* elem;   // elem 指向一个包含 sz 个 T 类型元素的数组
    int sz;
public:
    explicit Vector(int s);                // 构造函数：建立不变式，获取资源
    ~Vector() { delete[] elem; }           // 析构函数：释放资源
    // ... 拷贝和移动操作 ...
    T& operator[](int i);                  // 用于非 const Vector
    const T& operator[](int i) const;      // 用于 const Vector（§5.2.1）
    int size() const { return sz; }
};
```

`template<typename T>` 前缀使得 `T` 成为其后面声明的类型参数。这是 C++ 对数学中“对于所有 T”或更准确地说“对于所有类型 T”的表示。如果你想要数学上的“对于所有满足 P(T) 的 T”，你需要使用概念（§7.2.1，§8.2）。使用 `class` 来引入类型参数等价于使用 `typename`，在旧代码中我们经常看到 `template<class T>` 作为前缀。

成员函数可以类似地定义：

```cpp
template<typename T>
Vector<T>::Vector(int s)
{
    if (s < 0)
        throw length_error("Vector constructor: negative size");
    elem = new T[s];
    sz = s;
}

template<typename T>
const T& Vector<T>::operator[](int i) const
{
    if (i < 0 || size() <= i)
        throw out_of_range("Vector::operator[]");
    return elem[i];
}
```

给定这些定义，我们可以像这样定义 `Vector`：

```cpp
Vector<char> vc(200);                     // 200 个字符的 vector
Vector<string> vs(17);                    // 17 个字符串的 vector
Vector<List<int>> vli(45);                // 45 个整数列表的 vector
```

`Vector<List<int>>` 中的 `>>` 终止了嵌套的模板参数；它不是一个放错位置的输入运算符。

我们可以这样使用 `Vector`：

```cpp
void write(const Vector<string>& vs)      // 一些字符串的 Vector
{
    for (int i = 0; i != vs.size(); ++i)
        cout << vs[i] << '\n';
}
```

===== 第 4 页 =====

为了支持我们的 `Vector` 使用 range-for 循环，我们必须定义合适的 `begin()` 和 `end()` 函数：

```cpp
template<typename T>
T* begin(Vector<T>& x)
{
    return &x[0];   // 指向第一个元素或最后一个元素之后的位置
}

template<typename T>
T* end(Vector<T>& x)
{
    return &x[0] + x.size();   // 指向最后一个元素之后的位置
}
```

有了这些，我们可以写：

```cpp
void write2(Vector<string>& vs)           // 一些字符串的 Vector
{
    for (auto& s : vs)
        cout << s << '\n';
}
```

类似地，我们可以将 list、vector、map（即关联数组）、unordered map（即哈希表）等定义为模板（第 12 章）。

模板是一种编译时机制，因此与手工编写的代码相比，使用模板不会产生运行时开销。事实上，为 `Vector<double>` 生成的代码与为第 5 章中的 `Vector` 版本生成的代码完全相同。此外，为标准库 `vector<double>` 生成的代码可能更好（因为在其实现上投入了更多的精力）。

模板加上一组模板实参称为**实例化**或**特化**。在编译过程的后期，在实例化时，会为程序中使用的每个实例化生成代码（§8.5）。

#### 7.2.1 受约束的模板参数

===== 第 5 页 =====

大多数情况下，模板只对满足特定条件的模板实参有意义。例如，`Vector` 通常提供拷贝操作，如果它提供拷贝操作，则必须要求其元素是可拷贝的。也就是说，我们必须要求 `Vector` 的模板参数不仅仅是一个 `typename`，而是一个 `Element`，其中“Element”指定了可以作为元素的类型的要求：

```cpp
template<Element T>
class Vector {
private:
    T* elem;   // elem 指向一个包含 sz 个 T 类型元素的数组
    int sz;
    // ...
};
```

这个 `template<Element T>` 前缀是 C++ 对数学中“对于所有满足 Element(T) 的 T”的表示；也就是说，`Element` 是一个谓词，用于检查 `T` 是否具有 `Vector` 所需的所有属性。这样的谓词称为**概念**（§8.2）。为模板实参指定了概念的实参称为**受约束实参**，而实参受约束的模板称为**受约束模板**。

标准库元素类型的要求有点复杂（§12.2），但对于我们简单的 `Vector` 来说，`Element` 可以是类似标准库概念 `copyable`（§14.5）的东西。

尝试使用不满足其要求的类型来实例化模板是编译时错误。例如：

```cpp
Vector<int> v1;        // OK：可以拷贝 int
Vector<thread> v2;     // 错误：不能拷贝标准 thread（§18.2）
```

因此，概念使得编译器可以在使用点进行类型检查，提供比无约束模板实参更早、更好的错误信息。C++ 在 C++20 之前没有正式支持概念，因此旧代码使用无约束模板实参，并将要求留给了文档。然而，从模板生成的代码会进行类型检查，因此即使是无约束模板代码也与手写代码一样类型安全。对于无约束参数，这种类型检查必须等到所有相关实体的类型都可用时才能进行，因此它可能在编译过程的后期（实例化时，§8.5）发生，而且错误信息通常非常糟糕。概念检查是一种纯粹的编译时机制，生成的代码与无约束模板一样好。

#### 7.2.2 值模板参数

除了类型参数，模板还可以接受值参数。例如：

```cpp
template<typename T, int N>
struct Buffer {
    constexpr int size() { return N; }
    T elem[N];
    // ...
};
```

值参数在许多上下文中都很有用。例如，`Buffer` 允许我们创建任意大小的缓冲区，而不需要使用自由存储区（动态内存）：

```cpp
Buffer<char, 1024> glob;          // 全局字符缓冲区（静态分配）

void fct()
{
    Buffer<int, 10> buf;          // 局部整数缓冲区（在栈上）
    // ...
}
```

遗憾的是，由于晦涩的技术原因，字符串字面量目前还不能作为模板值参数。然而，在某些上下文中，用字符串值进行参数化的能力至关重要。幸运的是，我们可以使用一个保存字符串字符的数组：

```cpp
template<char* s>
void outs() { cout << s; }

char arr[] = "Weird workaround!";
void use()
{
    outs<"straightforward use">();    // 错误（目前）
    outs<arr>();                      // 输出：Weird workaround!
}
```

在 C++ 中，通常都有变通办法；我们不需要为每个用例都提供直接支持。

#### 7.2.3 模板参数推导

在将一个类型定义为模板的实例化时，我们必须指定其模板参数。考虑使用标准库模板 `pair`：

```cpp
pair<int, double> p = {1, 5.2};
```

必须指定模板参数类型可能会很繁琐。幸运的是，在许多上下文中，我们可以直接让 `pair` 的构造函数从初始化器推导模板参数：

```cpp
pair p = {1, 5.2};   // p 是一个 pair<int, double>
```

容器提供了另一个例子：

```cpp
template<typename T>
class Vector {
public:
    Vector(int);
    Vector(initializer_list<T>);   // 初始化列表构造函数
    // ...
};

Vector v1 {1, 2, 3};               // 从初始化器元素类型推导 v1 的元素类型：int
Vector v2 = v1;                    // 从 v1 的元素类型推导 v2 的元素类型：int
auto p = new Vector{1, 2, 3};      // p 是一个 Vector<int>*
```

===== 第 8 页 =====

显然，这简化了符号，并能消除因打错冗余模板参数类型而带来的烦恼。然而，它并非万能药。像所有其他强大的机制一样，推导也可能带来意外。考虑：

```cpp
Vector<string> vs {"Hello", "World"};        // OK：Vector<string>
Vector vs1 {"Hello", "World"};               // OK：推导为 Vector<const char*>（奇怪！）
Vector vs2 {"Hello"s, "World"s};             // OK：推导为 Vector<string>
Vector vs3 {"Hello"s, "World"};              // 错误：初始化列表的类型不统一
Vector<string> vs4 {"Hello"s, "World"};      // OK：元素类型是显式指定的
```

C 风格字符串字面量的类型是 `const char*`（§1.7.1）。如果这不是 `vs1` 所期望的，我们必须显式指定元素类型，或使用 `s` 后缀将其转换为真正的 `string`（§10.2）。

如果初始化列表中的元素具有不同的类型，我们无法推导出唯一的元素类型，因此会出现歧义错误。

有时，我们需要解决歧义。例如，标准库 `vector` 有一个接受一对界定序列的迭代器的构造函数，还有一个可以接受一对值的初始化列表构造函数。考虑：

```cpp
template<typename T>
class Vector {
public:
    Vector(initializer_list<T>);                 // 初始化列表构造函数
    template<typename Iter>
    Vector(Iter b, Iter e);                      // [b:e) 迭代器对构造函数
    struct iterator {
        using value_type = T;
        // ...
    };
    iterator begin();
    // ...
};
```

===== 第 9 页 =====

我们可以使用概念（§8.2）来解决这个问题，但标准库和许多其他重要的代码库是在我们拥有语言支持概念之前几十年编写的。对于这些代码，我们需要一种方式来表达“一对相同类型的值应被视为迭代器”。在 `Vector` 的声明之后添加一个**推导指引**正是实现了这一点：

```cpp
template<typename Iter>
Vector(Iter, Iter) -> Vector<typename Iter::value_type>;
```

现在我们有：

```cpp
Vector v1 {1, 2, 3, 4, 5};                     // 元素类型是 int
Vector v2(v1.begin(), v1.begin() + 2);         // 迭代器对：元素类型是 int
Vector v3 {v1.begin(), v1.begin() + 2};        // 元素类型是 Vector<int>::iterator
```

`{}` 初始化语法总是优先选择 `initializer_list` 构造函数（如果存在），因此 `v3` 是一个迭代器的 vector：`Vector<Vector<int>::iterator>`。

当不希望使用 `initializer_list` 时，通常使用 `()` 初始化语法（§12.2）。

推导指引的效果通常很微妙，因此最好将类模板设计为不需要推导指引。

喜欢首字母缩略词的人将“类模板参数推导”称为 **CTAD**。
