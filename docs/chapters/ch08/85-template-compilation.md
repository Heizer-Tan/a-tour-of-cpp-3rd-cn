## 8.5 模板编译模型

模板的编译模型与传统代码有所不同。理解这些差异对于高效使用模板至关重要。

### 8.5.1 包含模型

传统上，模板定义必须放在头文件中（或通过 `#include` 包含），因为编译器在实例化模板时需要看到完整的定义。这被称为*包含模型*（inclusion model）。

### 8.5.2 模块（C++20）

C++20 的模块提供了一种更好的模板组织方式：

```cpp
// Vector.cppm
export module Vector;

export template<typename T>
class Vector {
    // ...
};
```

使用模块时，模板定义可以放在模块实现文件中，编译器在编译模块时会生成所需的实例化信息。

### 8.5.3 显式实例化

可以通过*显式实例化*（explicit instantiation）来减少编译时间：

```cpp
template class Vector<int>;     // 显式实例化 Vector<int>
template class Vector<string>;  // 显式实例化 Vector<string>
```

这告诉编译器在此处生成模板的完整代码，避免在多个翻译单元中重复实例化。
