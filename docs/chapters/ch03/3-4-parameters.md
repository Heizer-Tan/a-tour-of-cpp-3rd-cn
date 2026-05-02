## 3.4 函数参数和返回值

函数是代码模块化的基本单元。理解参数如何传递以及返回值如何返回，对于编写正确且高效的 C++ 代码至关重要。

### 3.4.1 参数传递

默认情况下，C++ 使用*按值传递*（pass by value）——函数获得的是实参的一份副本：

```cpp
void f(int x) { x = 42; }   // 修改的是局部副本，不影响调用者

int a = 7;
f(a);                       // a 仍然是 7
```

对于大型对象，按值传递可能代价高昂。此时可以使用*引用传递*（pass by reference）：

```cpp
void g(int& x) { x = 42; }  // 修改的是原始对象

int a = 7;
g(a);                       // a 现在是 42
```

如果不想让函数修改实参，可以使用 `const` 引用：

```cpp
void h(const int& x) { /* 可以读取 x，但不能修改 */ }
```

对于小对象（如 `int`、`double`、指针），按值传递通常是最佳选择。对于大型对象，使用 `const` 引用可以避免不必要的拷贝。

### 3.4.2 返回值

函数可以通过返回值将结果传回调用者。与参数传递类似，返回值默认也是按值返回的：

```cpp
string compose(const string& name, const string& domain)
{
    return name + '@' + domain;
}

string addr = compose("greeting", "example.com");
```

现代 C++ 编译器会通过*返回值优化*（Return Value Optimization，RVO）和*移动语义*（move semantics，[§6.2.2](../ch06/6-2-copy-move.md)）来避免不必要的拷贝，因此按值返回通常非常高效。

### 3.4.3 结构化绑定

C++17 引入了*结构化绑定*（structured bindings），允许从函数返回多个值：

```cpp
struct Entry {
    string name;
    int value;
};

Entry read_entry(istream& is)
{
    string s;
    int i;
    is >> s >> i;
    return {s, i};
}

auto [name, value] = read_entry(cin);
cout << name << " = " << value << '\n';
```

结构化绑定使得处理多返回值变得简洁而优雅。
