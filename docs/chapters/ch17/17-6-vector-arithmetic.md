# 17.6 向量算术

§12.2 描述的 `vector` 旨在成为容纳值的一般机制，灵活并嵌入容器、迭代器与算法的体系之中。然而它并不支持数学意义上的向量运算。把此类运算加到 `vector` 上并不容易违背其通用性；而其通用与灵活又排除了数值工作中常常认为至关重要的优化。因此，标准库在 `<valarray>` 中提供了另一种更像数组的模板，名为 `valarray`：它不那么通用，却更容易为数值计算优化：

```cpp
template<typename T>
class valarray {
    // ...
};
```

通常的算术运算与最常见的数学函数对 `valarray` 均有支持。例如：

```cpp
void f(valarray<double>& a1, valarray<double>& a2)
{
    valarray<double> a = a1 * 3.14 + a2 / a1;   // 逐元素 *, +, /
    a2 += a1 * 3.14;

    a = abs(a);
    double d = a2[7];
    // ...
}
```

这些运算是**向量运算**；也就是说，它们施加到所涉及向量的每个元素上。

除算术运算外，`valarray` 还提供按步幅访问，以帮助实现多维计算。
