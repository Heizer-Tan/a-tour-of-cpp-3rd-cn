## 6.3 资源管理

资源管理是 C++ 编程中最重要的话题之一。资源可以是内存、文件句柄、锁、套接字等——任何需要在使用后显式释放的东西。

### 6.3.1 RAII

RAII（Resource Acquisition Is Initialization，资源获取即初始化）是 C++ 资源管理的核心惯用法。其基本思想是：

- 在构造函数中获取资源
- 在析构函数中释放资源
- 将资源的所有权绑定到对象的生命周期

```cpp
class File_handle {
public:
    File_handle(const string& name)
        : file{fopen(name.c_str(), "r")}
    {
        if (!file) throw std::runtime_error{"Cannot open file"};
    }

    ~File_handle() { if (file) fclose(file); }

    // 禁止拷贝，允许移动
    File_handle(const File_handle&) = delete;
    File_handle& operator=(const File_handle&) = delete;
    File_handle(File_handle&& other) noexcept : file{other.file} { other.file = nullptr; }
    File_handle& operator=(File_handle&& other) noexcept { /* ... */ return *this; }

private:
    FILE* file;
};
```

RAII 确保了即使发生异常，资源也能被正确释放。

### 6.3.2 智能指针

标准库提供了智能指针来管理动态分配的对象：

- `std::unique_ptr<T>`：独占所有权，不可拷贝，可移动
- `std::shared_ptr<T>`：共享所有权，引用计数
- `std::weak_ptr<T>`：不参与引用计数的观察者指针

```cpp
std::unique_ptr<Shape> p = std::make_unique<Circle>(Point{0, 0}, 10);
auto q = std::make_shared<Circle>(Point{1, 1}, 5);
```

优先使用 `std::unique_ptr`——它零开销，且清晰地表达了所有权语义。只有在确实需要共享所有权时才使用 `std::shared_ptr`。

### 6.3.3 三/五法则

如果一个类需要自定义析构函数、拷贝构造函数或拷贝赋值运算符中的任何一个，那么它几乎肯定需要自定义所有三个（C++11 之后是五个，加上移动构造函数和移动赋值运算符）。这被称为*三/五法则*（Rule of Three/Five）。

不过，在现代 C++ 中，更好的做法是使用 RAII 封装资源管理，让编译器自动生成这些操作——这被称为*零法则*（Rule of Zero）。
