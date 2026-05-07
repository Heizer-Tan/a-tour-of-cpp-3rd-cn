# 16.3 函数适配

当将一个函数作为函数参数传递时，参数的类型必须完全匹配被调用函数声明中表达的期望。如果一个预期的参数只是“几乎符合期望”，我们有以下几种方法进行调整：

- 使用 lambda（§16.3.1）。
- 使用 `std::mem_fn()` 从一个成员函数生成函数对象（§16.3.2）。
- 将函数定义为接受 `std::function`（§16.3.3）。

还有许多其他方法，但通常这三种之一效果最好。

### 16.3.1 Lambda 作为适配器

考虑经典的“绘制所有形状”示例：

```cpp
void draw_all(vector<Shape*>& v)
{
    for_each(v.begin(), v.end(), [](Shape* p) { p->draw(); });
}
```

与所有标准库算法一样，`for_each()` 使用传统的函数调用语法 `f(x)` 调用其参数，但 `Shape` 的 `draw()` 使用传统的面向对象记法 `x->f()`。Lambda 可以轻松地在这两种记法之间进行中介。

### 16.3.2 mem_fn()

给定一个成员函数，函数适配器 `mem_fn(mf)` 会产生一个可以作为非成员函数调用的函数对象。例如：

```cpp
void draw_all(vector<Shape*>& v)
{
    for_each(v.begin(), v.end(), mem_fn(&Shape::draw));
}
```

在 C++11 引入 lambda 之前，`mem_fn()` 及其等价物是将面向对象调用风格映射到函数式风格的主要方式。

===== 第 7 页 =====

### 16.3.3 function

标准库的 `function` 是一种可以容纳任何可以使用调用运算符 `()` 调用的对象的类型。也就是说，`function` 类型的对象就是一个函数对象（§7.3.2）。例如：

```cpp
int f1(double);
function<int(double)> fct1 {f1};            // 初始化为 f1

int f2(string);
function fct2 {f2};                         // fct2 的类型是 function<int(string)>

function fct3 = [](Shape* p) { p->draw(); }; // fct3 的类型是 function<void(Shape*)>
```

对于 `fct2`，我让函数的类型从初始化器推导：`int(string)`。

显然，`function` 对于回调、将操作作为参数传递、传递函数对象等很有用。然而，与直接调用相比，它可能会引入一些运行时开销。特别地，对于一个大小在编译时无法计算的函数对象，可能会发生自由存储区分配，这对性能关键的应用程序有严重的不良影响。C++23 将提供一个解决方案：`move_only_function`。

另一个问题是，`function` 作为一个对象，不参与重载。如果你需要重载函数对象（包括 lambda），请考虑使用 `overloaded`（§15.4.1）。
