# 16.7 位操�?
�?`<bit>` 中，我们找到了用于底层位操作的函数。位操作是一项专门但通常必不可少的活动。当我们接近硬件时，我们经常需要查看位、改变字节或字中的位模式，以及将原始内存转换为类型化对象。例如，`bit_cast` 允许我们将一个类型的值转换为另一个相同大小的类型的值：

```cpp
double val = 7.2;
auto x = bit_cast<uint64_t>(val);   // 获取 64 位浮点值的位表�?auto y = bit_cast<uint64_t>(&val);  // 获取 64 位指针的位表�?
struct Word { std::byte b[8]; };
std::byte buffer[1024];
// ...
auto p = bit_cast<Word*>(&buffer[i]);   // p 指向 8 个字�?auto i = bit_cast<int64_t>(*p);         // 将这 8 个字节转换为整数
```

标准库类�?`std::byte`（`std::` 是必需的）的存在是为了表示字节，而不是已知表示字符或整数的字节。特别地，`std::byte` 只提供按位逻辑操作，而不提供算术操作。通常，进行位操作的最佳类型是无符号整数或 `std::byte`。所谓最佳，我指的是最快且最不容易出现意外。例如：

```cpp
void use(unsigned int ui)
{
    int x0 = bit_width(ui);          // 表示 ui 所需的最小位�?    unsigned int ui2 = rotl(ui, 8);  // 左旋 8 位（注意：不改变 ui�?    int x1 = popcount(ui);           // ui �?1 的个�?    // ...
}
```

另请参见 `bitset`（�?5.3.2）�?