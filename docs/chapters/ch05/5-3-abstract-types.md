## 5.3 抽象类型

与具体类型不同，抽象类型（abstract type）将使用者与实现细节完全隔离开来。这是通过*虚函数*（virtual functions）实现的。

抽象类型的典型例子是图形系统中的 `Shape`：

```cpp
class Shape {
public:
    virtual void draw() const = 0;  // 纯虚函数
    virtual void rotate(int angle) = 0;
    virtual ~Shape() {}             // 虚析构函数
};
```

`= 0` 表示该函数是*纯虚函数*（pure virtual），这意味着 `Shape` 是一个*抽象类*（abstract class）——不能直接创建 `Shape` 对象，只能创建其派生类的对象。

```cpp
class Circle : public Shape {
public:
    Circle(Point p, int r) : center{p}, radius{r} {}
    void draw() const override;
    void rotate(int) override {}    // 圆旋转后不变
private:
    Point center;
    int radius;
};
```

`override` 关键字明确表示该函数覆盖了基类中的虚函数，编译器会检查签名是否匹配。

抽象类型的使用者通过基类的指针或引用来操作对象：

```cpp
void draw_all(const vector<unique_ptr<Shape>>& shapes)
{
    for (auto& s : shapes)
        s->draw();
}
```

这里使用了 `std::unique_ptr`（[§15.2.1](../ch15/15-2-pointers.md)）来管理 `Shape` 对象的生命周期。
