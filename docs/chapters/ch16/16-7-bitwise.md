# 16.7 按位运算

在 `<bit>` 中可以找到用于底层按位操作的函数。按位操作很专门化，却又常常必不可少。当我们贴近硬件时，往往必须观察比特、改变字节或字中的位模式，并把原始内存解释为带类型的对象。例如，`bit_cast` 允许我们把一种类型的值转换为另一种**大小相同**类型的值：

```cpp
double val = 7.2;
auto x = bit_cast<uint64_t>(val);       // 取得 64 位浮点数的位模式
auto y = bit_cast<uint64_t>(&val);      // 取得 64 位指针的位模式

struct Word { std::byte b[8]; };
std::byte buffer[1024];
// ...
auto p = bit_cast<Word*>(&buffer[i]);    // p 指向 8 个字节
auto i = bit_cast<int64_t>(*p);          // 把这 8 个字节解释为整数
```

标准库类型 `std::byte`（必须写 `std::` 前缀）用来表示**字节**，而不是已知表示字符或整数的字节。特别是，`std::byte` 只提供按位逻辑运算，不提供算术运算。通常，执行位操作的最佳类型是无符号整数或 `std::byte`。所谓最佳，指的是最快、也最不易出人意料。

另见 `bitset`（§15.3.2）。

对整数做常用位操作时，还会用到诸如 `bit_width`、`rotl`、`popcount` 等函数。例如：

```cpp
void use(unsigned int ui)
{
    int x0 = bit_width(ui);           // 表示 ui 所需的最少位数
    unsigned int ui2 = rotl(ui, 8);    // 循环左移 8 位（注意：不改变 ui 本身）
    int x1 = popcount(ui);            // ui 中 1 的个数
    // ...
}
```
