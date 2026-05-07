
在 `<limits>` 中，标准库提供了描述内置类型属性的类——例如 `float` 的最大指数或 `int` 的字节数。例如，我们可以断言 `char` 是有符号的：

```cpp
static_assert(numeric_limits<char>::is_signed, "unsigned characters!");
static_assert(100000 < numeric_limits<int>::max(), "small ints!");
```

第二个断言之所以有效，是因为 `numeric_limits<int>::max()` 是一个 `constexpr` 函数（§1.6）。

我们可以为我们自己的用户定义类型定义 `numeric_limits`。

## 17.8 类型别名