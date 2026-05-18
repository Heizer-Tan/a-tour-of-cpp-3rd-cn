# 16.3 函数适配器

把函数作为实参传递时，实参类型必须与被调用函数声明中所表达的期望完全一致。若打算传入的参数只是「差不多」吻合期望，可用下列替代手段加以调节：

- 使用 lambda（§16.3.1）。
- 使用 `std::mem_fn()` 由成员函数生成函数对象（§16.3.2）。
- 把函数设计成接受 `std::function`（§16.3.3）。

还有很多别的做法，但通常这三种之一效果最好。

## 16.3.1

考虑经典的「绘制全部图形」示例：

```cpp
void draw_all(vector<Shape*>& v)
{
    for_each(v.begin(), v.end(), [](Shape* p) { p->draw(); });
}
```

与所有标准库算法一样，`for_each()` 会用传统的函数调用语法 `f(x)` 调用其实参；但 `Shape::draw()` 使用的是常规的面向对象记法 `x->f()`。lambda 很容易在这两种记法之间斡旋。

## 16.3.2

给定成员函数，函数适配器 `mem_fn(mf)` 会生成一个可按非成员函数那样调用的函数对象。例如：

```cpp
void draw_all(vector<Shape*>& v)
{
    for_each(v.begin(), v.end(), mem_fn(&Shape::draw));
}
```

在 C++11 引入 lambda 之前，`mem_fn()` 及其同类是从面向对象调用风格映射到函数式调用风格的主要途径。

## 16.3.3

标准库的 `function` 是一种类型，能容纳任何可用调用运算符 `()` 调用的对象。也就是说，`function` 类型的对象是一个函数对象（§7.3.2）。例如：

```cpp
int f1(double);
function<int(double)> fct1{f1};               // 初始化为 f1

int f2(string);
function fct2{f2};                           // fct2 的类型为 function<int(string)>

function fct3 = [](Shape* p) { p->draw(); }; // fct3 的类型为 function<void(Shape*)>
```

对 `fct2`，我让函数类型由初始化器 `int(string)` 推导得出。

显然，`function` 适用于回调、把操作作为参数传递、传递函数对象等场合。然而，与直接调用相比，它可能引入一些运行期开销。尤其是，对于无法在编译期确定大小的 `function` 对象，可能发生自由存储分配，并对性能关键应用造成严重不良影响。C++23 将带来一种解决方案：`move_only_function`。

另一个问题是：`function` 作为对象，不参与重载分辨。若需要对函数对象（包括 lambda）进行重载，可考虑 `overloaded`（§15.4.1）。
