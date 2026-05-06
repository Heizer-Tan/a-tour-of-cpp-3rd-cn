# 2.4 枚举

除了类之外，C++ 还支持一种简单的用户定义类型——*枚举*（enumeration），用于表示一组小范围的整数值：

```cpp
enum class Color { red, blue, green };
enum class Traffic_light { green, yellow, red };

Color col = Color::red; 
Traffic_light light = Traffic_light::red;
```

注意，枚举值（比如`red`）位于其`enum class`的作用域里， 因此可以在不同的`enum class`里重复出现，而且不会相互混淆。 例如：`Color::red`是`Color`里面的`red`，跟`Traffic_light::red`毫无关系。

枚举类型用于表示一组有限的整数值。使用枚举可以让代码更易于理解，也能减少出错的可能性。如果不使用这些有意义的枚举名称，代码就会变得难以理解，出错的概率也会增加。

在枚举类之后定义的规则表明，该枚举属于强类型枚举，其枚举值具有明确的范围。由于枚举值属于独立的类型，因此可以避免对常量的误用。特别是，我们不能将`Traffic_light`和`Color`这两种枚举值混在一起使用。

```cpp
Color x1 = red;                     // error: which red? 
Color y2 = Traffic_light::red;      // error: that red is not a Color 
Color z3 = Color::red;              // OK 
auto x4 = Color::red;               // OK: Color::red is a Color
```

同样，我们也不能将`Color`和整数值混合使用。

```cpp 
int i = Color::red; // 错误：Color::red不是int类型
Color c = 2;        // 初始化错误：2不是Color类型
```

捕获试图将某种类型转换为`enum`的操作，是一种有效的错误预防措施。不过，我们通常希望用该类型对应的底层类型的值来初始化枚举变量（默认情况下，底层类型为整数）。这种做法是允许的，同时，从底层类型进行显式转换也是可行的。

```cpp
Color x = Color{5}; // 可行，但略有些啰嗦
Color y {6};        // 同样可行
```

同样地，我们也可以将枚举值明确转换为相应的底层类型：

```cpp
int x = int(Color::red);
```

默认情况下，枚举类（`enum class`）具有赋值、初始化以及比较操作的功能（比如`==`和`<`运算符；§1.4节）。不过，枚举是一种用户自定义类型，因此我们可以为其定义额外的运算符（参见§6.4节）。

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

auto signal = Traffic_light::red;
Traffic_light next = ++signal;   // 根据交通灯规则前进到下一个状态
```

如果反复使用`Traffic_light`这个名称显得过于繁琐，我们可以在特定的范围内将其缩写起来。

```cpp
Traffic_light& operator++(Traffic_light& t)  // prefix increment: ++ 
{ 
    using enum Traffic_light;         // here, we are using Traffic_light 

    switch (t) {
        switch (t) { 
        case green:      return t=yellow; 
        case yellow:     return t=red; 
        case red:          return t=green; 
        } 
    }
}
```
如果你不想显式指定枚举量的名称，并且希望枚举量直接以整数的形式存在（无需进行任何转换），那么你可以去掉`enum class`这个限定，从而得到一个“普通”的枚举类型。在这种模式下，枚举量的值会与其名称处于相同的作用域内，并且会自动转换为相应的整数值。例如：

```cpp
enum Color { red, green, blue }; 
int col = green;
```

在这里，`col`的值为`1`。默认情况下，枚举类型的整数值是从`0`开始递增的，每增加一个枚举项，其值就增加1。这种“简单”的枚举方式在C++和C语言中早已被使用。虽然这种枚举方式的性能稍差一些，但它在当前的代码中仍然很常见。
