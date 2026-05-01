## 6.2 复制和移动

### 6.2.1 拷贝

默认情况下，对象可以通过拷贝来复制——无论是通过赋值还是通过初始化。例如：

```cpp
Vector v1 = {1, 2, 3};
Vector v2 = v1;     // 拷贝构造
Vector v3;
v3 = v1;            // 拷贝赋值
```

当类管理着资源（如动态内存）时，默认的逐成员拷贝会导致*浅拷贝*（shallow copy）——两个对象共享同一块内存。这通常不是我们想要的。我们需要*深拷贝*（deep copy）：

```cpp
class Vector {
public:
    Vector(const Vector& a)             // 拷贝构造函数
        : elem{new double[a.sz]}, sz{a.sz}
    {
        std::copy(a.elem, a.elem + sz, elem);
    }

    Vector& operator=(const Vector& a)  // 拷贝赋值运算符
    {
        double* p = new double[a.sz];
        std::copy(a.elem, a.elem + a.sz, p);
        delete[] elem;                  // 释放旧资源
        elem = p;
        sz = a.sz;
        return *this;
    }
    // ...
};
```

### 6.2.2 移动

拷贝可能代价高昂——特别是对于大型容器。C++11 引入了*移动语义*（move semantics），允许将资源从一个对象"窃取"到另一个对象，而不需要拷贝：

```cpp
class Vector {
public:
    Vector(Vector&& a) noexcept         // 移动构造函数
        : elem{a.elem}, sz{a.sz}
    {
        a.elem = nullptr;               // 让 a 变为空状态
        a.sz = 0;
    }

    Vector& operator=(Vector&& a) noexcept  // 移动赋值运算符
    {
        delete[] elem;                  // 释放旧资源
        elem = a.elem;
        sz = a.sz;
        a.elem = nullptr;
        a.sz = 0;
        return *this;
    }
    // ...
};
```

`&&` 表示*右值引用*（rvalue reference），`noexcept` 保证移动操作不抛出异常。

要触发移动而非拷贝，可以使用 `std::move()`：

```cpp
Vector v1 = {1, 2, 3};
Vector v2 = std::move(v1);  // 移动 v1 到 v2；v1 变为空状态
```

移动语义是现代 C++ 性能优化的关键——它使得按值返回大型对象变得高效。
