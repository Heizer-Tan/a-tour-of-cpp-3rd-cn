## 4.3 不变式

*不变式*（invariant）是一个关于对象状态的逻辑条件，在对象的整个生命周期中（除了短暂的内部操作期间），该条件必须始终为真。不变式是设计健壮类的基础。

例如，对于我们的 `Vector` 类，不变式包括：

- `elem` 指向一个包含 `sz` 个 `double` 的数组（当 `sz > 0` 时）
- `sz >= 0`

构造函数负责建立不变式：

```cpp
Vector::Vector(int s)
{
    if (s < 0)
        throw std::length_error{"Vector size must be non-negative"};
    elem = new double[s];
    sz = s;
}
```

每个成员函数在进入时假定不变式成立，在退出时必须确保不变式仍然成立：

```cpp
double& Vector::operator[](int i)
{
    if (i < 0 || size() <= i)
        throw std::out_of_range{"Vector::operator[]"};
    return elem[i];
}
```

通过维护不变式，我们可以大大简化代码的推理过程——我们总是知道对象处于有效状态。

RAII（[§6.3](../ch06/6-3-resource-mgmt.md)）是维护不变式的关键技术：构造函数获取资源并建立不变式，析构函数释放资源。这样，即使发生异常，不变式也不会被破坏。
