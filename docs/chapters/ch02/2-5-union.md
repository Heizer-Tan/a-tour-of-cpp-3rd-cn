## 2.5 联合体

`union`（联合体）是一种特殊的结构体，它的所有成员都分配在同一块内存地址上。因此，一个 `union` 对象在同一时刻只能持有其中一个成员的值。`union` 的大小等于其最大成员的大小：

```cpp
enum class Type { ptr, num };   // Type 可以是 ptr 或 num

union Value {
    char* p;    // 如果 Type 是 ptr，则使用 p
    int i;      // 如果 Type 是 num，则使用 i
};
```

使用时，程序员需要自行跟踪当前 `union` 中实际存储的是哪个成员：

```cpp
struct Entry {
    Type t;
    Value v;    // 当 t == Type::ptr 时使用 v.p，当 t == Type::num 时使用 v.i
};

void f(Entry* pe)
{
    if (pe->t == Type::num)
        std::cout << pe->v.i;
    // ...
}
```

这种自行维护类型标签的方式容易出错。C++17 引入了 `std::variant`（[§15.4.1](../ch15/15-4-alternatives.md)），它提供了一种类型安全的 `union` 替代方案，能够自动跟踪当前存储的类型。

标准库中有一个特殊的 `union` 叫做 `std::optional`，用于表示"可能存在也可能不存在"的值。不过，`std::optional` 实际上是用 `variant` 实现的，而非裸 `union`。

在底层编程中，`union` 仍然有其用武之地——例如，用于查看内存中某个对象的原始字节表示，或者用于实现紧凑的数据结构。但大多数应用层代码应优先使用 `std::variant` 或 `std::optional`。
