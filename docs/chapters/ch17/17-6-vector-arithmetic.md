
§12.2 中描述的 `vector` 被设计为一种通用的值存储机制，灵活且适合容器、迭代器和算法的架构。然而，它不支持数学向量运算。将此类操作添加到 `vector` 很容易，但其通用性和灵活性排除了对于严肃数值工作通常被认为是必需的优化。因此，标准库在 `<valarray>` 中提供了一个类似 `vector` 的模板，称为 `valarray`，它不那么通用，但更易于针对数值计算进行优化：

```cpp
template<typename T>
class valarray {
    // ...
};
```

`valarray` 支持通常的算术运算和最常见的数学函数。例如：

```cpp
void f(valarray<double>& a1, valarray<double>& a2)
{
    valarray<double> a = a1 * 3.14 + a2 / a1;   // 数值数组运算符 *, +, /
    a2 += a1 * 3.14;
    a = abs(a);
    double d = a2[7];
    // ...
}
```

这些操作是向量操作；也就是说，它们被应用于所涉及向量的每个元素。

除了算术运算，`valarray` 还提供跨步访问以帮助实现多维计算。

## 17.7 数值极限