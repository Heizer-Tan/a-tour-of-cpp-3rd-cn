## 7.4 模板机制

### 7.4.1 实例化

模板不是直接的代码——它是生成代码的*蓝图*。当模板被使用时，编译器会根据模板参数*实例化*（instantiate）出具体的代码：

```cpp
Vector<int> vi(10);     // 编译器生成 Vector<int> 的完整代码
Vector<string> vs(10);  // 编译器生成 Vector<string> 的完整代码
```

每个不同的模板参数组合都会生成一份独立的代码。这意味着模板可能导致*代码膨胀*（code bloat），但通常可以通过精心设计来缓解。

### 7.4.2 模板参数

模板参数不仅可以是类型，还可以是值、模板，甚至其他模板：

```cpp
template<typename T, int N>
struct Buffer {
    T buf[N];
    // ...
};

Buffer<char, 1024> glob;    // 1024 个字符的缓冲区
```

### 7.4.3 可变参数模板

C++11 引入了*可变参数模板*（variadic templates），允许模板接受任意数量的参数：

```cpp
template<typename T, typename... Args>
std::unique_ptr<T> make_unique(Args&&... args)
{
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}
```

`Args...` 表示任意数量的类型参数，`args...` 表示对应的值参数。这是实现 `std::make_unique`、`std::tuple` 等工具的基础。

### 7.4.4 模板特化

可以为特定的模板参数提供*特化*（specialization）版本：

```cpp
template<>
class Vector<void*> {
    // void* 指针向量的特殊实现
};
```

特化允许为特定类型提供优化的实现，但应谨慎使用——过度特化会使代码难以维护。
