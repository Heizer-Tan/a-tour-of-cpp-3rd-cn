## 5.2 具体类型

具体类型（concrete type）的基本特征是：它的表示（representation）是其定义的一部分。这使得具体类型可以：

- 将对象直接放在栈上、其他对象内部，或者静态分配
- 直接引用对象（而非通过指针或引用）
- 在创建时立即初始化（通常通过构造函数）
- 在销毁时自动清理（通过析构函数）

典型的具体类型包括 `std::vector`、`std::string`、`std::complex` 等。

### 5.2.1 容器和资源句柄

许多具体类型是*资源句柄*（resource handles）——它们管理着对其他对象的访问。例如，`std::vector` 管理一个动态分配的数组：

```cpp
class Vector {
public:
    Vector(int s) : elem{new double[s]}, sz{s} {}
    ~Vector() { delete[] elem; }    // 析构函数：释放资源

    double& operator[](int i) { return elem[i]; }
    int size() const { return sz; }
private:
    double* elem;
    int sz;
};
```

析构函数（destructor）在对象生命周期结束时被自动调用，负责释放构造函数获取的资源。这种"构造函数获取资源，析构函数释放资源"的模式被称为 RAII（[§6.3](../ch06/63-resource-mgmt.md)）。

### 5.2.2 初始化容器

容器可以通过多种方式初始化。C++ 提供了*初始化列表*（initializer lists）：

```cpp
Vector v1 = {1, 2, 3, 4, 5};    // 使用初始化列表
Vector v2 = {1.1, 2.2, 3.3};    // 自动推断大小
```

要支持初始化列表，需要添加一个接受 `std::initializer_list` 的构造函数：

```cpp
class Vector {
public:
    Vector(std::initializer_list<double> lst)
        : elem{new double[lst.size()]}, sz{static_cast<int>(lst.size())}
    {
        std::copy(lst.begin(), lst.end(), elem);
    }
    // ...
};
```

### 5.2.3 `const` 成员函数

声明为 `const` 的成员函数承诺不修改对象的状态：

```cpp
int size() const { return sz; }     // 不会修改 Vector
```

`const` 成员函数可以被 `const` 对象调用，这对于编写正确的代码至关重要。
