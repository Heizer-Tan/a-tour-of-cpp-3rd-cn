# 5.5 类层次结构

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
