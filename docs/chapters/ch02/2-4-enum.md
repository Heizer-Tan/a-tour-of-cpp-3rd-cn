## 2.4 枚举

除了类之外，C++ 还支持一种简单的用户定义类型——*枚举*（enumeration），用于表示一组小范围的整数值：

```cpp
enum class Color { red, blue, green };
enum class Traffic_light { green, yellow, red };
```

`enum class`（限定作用域的枚举）中的枚举值位于其所属枚举的作用域内，使用时需要加上限定符：

```cpp
Color col = Color::red;
Traffic_light light = Traffic_light::red;
```

注意，`Color::red` 和 `Traffic_light::red` 是两个完全不同的值，不会混淆。

枚举常用于表示一组固定的选项。例如：

```cpp
Traffic_light& operator++(Traffic_light& t)
    // 前缀递增：++
{
    switch (t) {
    case Traffic_light::green:  return t = Traffic_light::yellow;
    case Traffic_light::yellow: return t = Traffic_light::red;
    case Traffic_light::red:    return t = Traffic_light::green;
    }
    return t;
}

Traffic_light next = ++light;   // 根据交通灯规则前进到下一个状态
```

C++ 也支持普通的 `enum`（不限作用域的枚举），但 `enum class` 更受推荐——它不会隐式转换为整数，也不会污染外层作用域。

默认情况下，`enum class` 的底层类型是 `int`，但你也可以显式指定：

```cpp
enum class Warning : int { green, yellow, orange, red }; // sizeof(Warning) == sizeof(int)
enum class Small : char { a, b, c };                      // sizeof(Small) == 1
```
