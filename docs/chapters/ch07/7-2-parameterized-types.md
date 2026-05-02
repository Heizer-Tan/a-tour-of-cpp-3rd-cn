## 7.2 参数化类型

类模板允许我们定义可以适用于多种类型的类。最经典的例子是 `std::vector`：

```cpp
template<typename T>
class Vector {
public:
    Vector(int s);
    ~Vector() { delete[] elem; }

    T& operator[](int i);
    const T& operator[](int i) const;
    int size() const { return sz; }
private:
    T* elem;
    int sz;
};

template<typename T>
Vector<T>::Vector(int s)
    : elem{new T[s]}, sz{s}
{
}

template<typename T>
T& Vector<T>::operator[](int i)
{
    if (i < 0 || size() <= i)
        throw std::out_of_range{"Vector::operator[]"};
    return elem[i];
}
```

使用类模板时，需要指定类型参数：

```cpp
Vector<char> vc(200);       // 200 个字符的向量
Vector<string> vs(17);      // 17 个字符串的向量
Vector<list<int>> vli(45);  // 45 个整数列表的向量
```

模板支持任何满足要求的类型——只要该类型支持模板中使用的操作。例如，`Vector<T>` 要求 `T` 支持默认构造和析构。

### 模板参数推导（CTAD）

C++17 引入了*类模板参数推导*（Class Template Argument Deduction，CTAD），允许在某些情况下省略模板参数：

```cpp
std::pair p {1, 3.14};      // std::pair<int, double>
std::vector v {1, 2, 3};    // std::vector<int>
```
