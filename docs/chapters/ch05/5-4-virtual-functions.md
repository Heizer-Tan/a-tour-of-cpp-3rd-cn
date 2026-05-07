# 5.4 虚函数

再次考虑 `Container` 的使用：

```cpp
void use(Container& c)
{
    const int sz = c.size();
    for (int i = 0; i != sz; ++i)
        cout << c[i] << '\n';
}
```

`use()` 中的调用 `c[i]` 是如何解析到正确的 `operator[]()` 的？当 `h()` 调用 `use()` 时，必须调用 `List_container` 的 `operator[]()`。当 `g()` 调用 `use()` 时，必须调用 `Vector_container` 的 `operator[]()`。为了实现这种解析，`Container` 对象必须包含信息，以便在运行时选择正确的函数来调用。通常的实现技术是让编译器将虚函数的名字转换为一个指向函数指针表的索引。该表通常称为**虚函数表**或简称为 **vtbl**。每个包含虚函数的类都有自己的 vtbl，标识其虚函数。这可以用图形表示如下：

[图片描述：Vector_container 及其 vtbl 图示]
[图片描述：Container 对象指向 vtbl 的指针]

vtbl 中的函数使得即使调用者不知道对象的大小和数据布局，也能正确使用对象。调用者的实现只需要知道 `Container` 中指向 vtbl 的指针的位置以及每个虚函数使用的索引。这种虚调用机制可以做到几乎与“普通函数调用”机制一样高效（在 25% 以内，并且对于同一对象的重复调用远远更便宜）。它的空间开销是每个具有虚函数的类对象中一个指针，再加上每个这样的类一个 vtbl。
