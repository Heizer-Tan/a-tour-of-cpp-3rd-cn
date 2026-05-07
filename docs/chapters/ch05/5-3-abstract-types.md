# 5.3 抽象类型

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
