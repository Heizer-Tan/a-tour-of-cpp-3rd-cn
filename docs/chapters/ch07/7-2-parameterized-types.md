# 7.2 参数化类型

我们可以将 [§5.2.2](../ch05/5-2-concrete-types.md) 中的“元素为 `double` 的向量”类型，推广成“任意元素类型 `T` 的向量”：做法是把它写成模板，并把原先的 `double` 换成类型参数。例如：

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
    const T& operator[](int i) const;      // 用于 const Vector（[§5.2.1](../ch05/5-2-concrete-types.md)）
    int size() const { return sz; }
};
```

`template<typename T>` 前缀使得 `T` 成为其后面声明的类型参数。这是 C++ 对数学中“对于所有 T”或更准确地说“对于所有类型 T”的表示。如果你想要数学意义上的“对所有满足约束 P(T) 的 T”，应使用概念（[§7.2.1](7-2-parameterized-types.md)、[§8.2](../ch08/8-2-concepts.md)）。使用 `class` 来引入类型参数等价于使用 `typename`，在旧代码中我们经常看到 `template<class T>` 作为前缀。

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
Vector<list<int>> vli(45);                // 45 个整数链表组成的向量
```

`Vector<list<int>>` 中的 `>>` 结束嵌套的模板实参列表，而不是误贴的输入运算符 `>>`。

我们可以这样使用 `Vector`：

```cpp
void write(const Vector<string>& vs)      // 一些字符串的 Vector
{
    for (int i = 0; i != vs.size(); ++i)
        cout << vs[i] << '\n';
}
```

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

类似地，我们可以把 `list`、`vector`、`map`（关联数组）、`unordered_map`（哈希表）等都实现为模板（[第十二章](../ch12/index.md)）。

模板是一种编译时机制，因此与手工编写的代码相比，使用模板不会产生运行时开销。事实上，为 `Vector<double>` 生成的代码与为第 5 章中的 `Vector` 版本生成的代码完全相同。此外，为标准库 `vector<double>` 生成的代码可能更好（因为在其实现上投入了更多的精力）。

模板加上一组模板实参构成一次**实例化**（或称**特化**）。这一过程发生在编译较晚阶段：对每个实际用到的模板组合生成对应代码（[§8.5](../ch08/8-5-template-compilation.md)）。

## 7.2.1 受约束的模板参数

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

这个 `template<Element T>` 前缀就是 C++ 里“对所有满足 Element(T) 的类型 T”的写法；换言之，`Element` 是一个语法谓词，检查 `T` 是否具备向量元素应有的性质。这一类谓词称为**概念**（[§8.2](../ch08/8-2-concepts.md)）。带上概念限制的模板形参叫作**受约束实参**，而含有这种形参的模板叫作**受约束模板**。

标准库对元素类型的要求略复杂（[§12.2](../ch12/12-2-vector.md)）；对我们这个简单的 `Vector` 而言，`Element` 可以比照标准库概念 `copyable`（[§14.5](../ch14/14-5-concept-overview.md)）。

尝试使用不满足其要求的类型实例化模板会在编译期失败。例如：

```cpp
Vector<int> v1;        // OK：可以拷贝 int
Vector<thread> v2;     // 错误：`thread` 不可拷贝（[§18.2](../ch18/18-2-tasks-and-threads.md)）
```

因此概念让编译器在**调用点**完成类型核对，能比无约束模板更早给出清晰的诊断。在 C++20 之前语言尚未标准化“概念”，老代码只能靠无约束的 `typename` 参数并把约定写在注释里；但模板仍会生成并进行类型检查，因此无约束代码在语义上并不比手写函数更不安全。只不过对这一类参数的最后一轮检查要等到所有实体类型都确定后才能做，往往拖到**模板实例化**阶段（[§8.5](../ch08/8-5-template-compilation.md)），报错信息也往往令人抓狂。概念核对纯属于编译期，生成代码的效率与手写特化相差无几。

## 7.2.2 值模板参数

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

## 7.2.3 模板参数推导

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

显然，这简化了符号，并能消除因打错冗余模板参数类型而带来的烦恼。然而，它并非万能药。像所有其他强大的机制一样，推导也可能带来意外。考虑：

```cpp
Vector<string> vs {"Hello", "World"};        // OK：Vector<string>
Vector vs1 {"Hello", "World"};               // OK：推导为 Vector<const char*>（奇怪！）
Vector vs2 {"Hello"s, "World"s};             // OK：推导为 Vector<string>
Vector vs3 {"Hello"s, "World"};              // 错误：初始化列表的类型不统一
Vector<string> vs4 {"Hello"s, "World"};      // OK：元素类型是显式指定的
```

C 风格字符串字面量的类型是 `const char*`（[§1.7.1](../ch01/1-7-pointers-arrays.md)）。若这并非 `vs1` 的设计意图，要么显式写出元素类型，要么使用 `s` 后缀得到真正的 `string`（[§10.2](../ch10/10-2-strings.md)）。

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

我们可以借助概念（[§8.2](../ch08/8-2-concepts.md)）描述这种区别，但标准库与大量基础库早在概念进入语言之前就已写成。对它们而言，需要一种说法来表达“若两个值类型相同，就把它们当作迭代器对”。在类模板声明之后追加**推导指引**即可做到：

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

当不希望命中 `initializer_list` 构造函数时，更惯用的是圆括号初始化（[§12.2](../ch12/12-2-vector.md)）。

推导指引的效果通常很微妙，因此最好将类模板设计为不需要推导指引。

喜欢首字母缩略词的人将“类模板参数推导”称为 **CTAD**。
