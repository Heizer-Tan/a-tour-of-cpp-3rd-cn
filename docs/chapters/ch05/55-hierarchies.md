## 5.5 类层次结构

类层次结构（class hierarchy）是通过继承（inheritance）组织起来的一组类。它表达了概念之间的"是一个"（is-a）关系。

### 5.5.1 继承

C++ 支持多种继承方式：

```cpp
class Circle : public Shape { /* ... */ };  // public 继承：is-a 关系
class My_list : private List { /* ... */ }; // private 继承：实现继承
```

`public` 继承是最常见的形式，表示派生类"是一个"基类。`private` 继承用于实现细节的复用，而非接口继承。

### 5.5.2 显式禁止拷贝

对于类层次结构中的类，拷贝操作通常需要被禁止，以避免*切片*（slicing）问题——当派生类对象被拷贝到基类对象时，派生类特有的数据会被"切掉"：

```cpp
class Shape {
public:
    Shape(const Shape&) = delete;           // 禁止拷贝构造
    Shape& operator=(const Shape&) = delete; // 禁止拷贝赋值
    // ...
};
```

### 5.5.3 多重继承

C++ 支持多重继承（multiple inheritance），即一个类可以从多个基类派生：

```cpp
class Satellite : public Shape, public Orbiter {
    // ...
};
```

多重继承应谨慎使用——大多数情况下，单继承配合接口（纯抽象类）就足够了。

### 5.5.4 类型转换

在类层次结构中，可以使用 `dynamic_cast` 进行安全的向下转型：

```cpp
void rotate(Shape* s, int angle)
{
    if (auto* c = dynamic_cast<Circle*>(s)) {
        // s 实际上是一个 Circle
    }
    s->rotate(angle);
}
```

`dynamic_cast` 在运行时检查转换是否有效，如果失败则返回 `nullptr`（对指针）或抛出 `std::bad_cast`（对引用）。
