## 2.2 结构体

构建新类型的第一步，通常是把所需的元素组织成一个数据结构——也就是一个 `struct`（结构体）：

```cpp
struct Vector {
    int sz;         // 元素个数
    double* elem;   // 指向元素的指针
};
```

这个 `Vector` 类型的第一个版本包含一个 `int` 和一个 `double*`。

`Vector` 类型的变量可以这样定义：

```cpp
Vector v;
```

但光有数据本身往往不够用——我们还需要能对这些数据执行操作。例如，我们需要一种方式来初始化 `Vector`：

```cpp
void vector_init(Vector& v, int s)
{
    v.elem = new double[s]; // 分配一个包含 s 个 double 的数组
    v.sz = s;
}
```

也就是说，`v` 的 `elem` 成员被赋予了一个由 `new` 运算符生成的指针，而 `sz` 成员则记录了元素个数。这里的 `Vector&` 表示我们通过*引用*（reference，[§1.7](../ch01/1-7-pointers-arrays.md)）来传递 `v`，这样 `vector_init()` 就能直接修改传入的 `v` 对象。

`new` 运算符从一块名为*自由存储区*（free store，也叫*动态内存*或*堆*）的内存区域中分配空间。分配在自由存储区上的对象独立于其创建时所处的作用域，会一直"存活"到被 `delete` 运算符销毁为止（[§6.2](../ch06/6-2-copy-move.md)）。

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

然而，我们的 `Vector` 和 `vector_init()` 之间的这种分离并不优雅——我们需要把 `Vector` 的各个部分推给用户自己去操作。有没有办法让 `Vector` 自己"知道"如何初始化呢？
