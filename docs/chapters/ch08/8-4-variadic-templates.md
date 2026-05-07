# 8.4 变参模板

模板可以定义为接受任意数量的任意类型的参数。这样的模板称为**变参模板**。考虑一个简单的函数，用于写出任何具有 `<<` 运算符的类型的值：

```cpp
void user()
{
    print("first: ", 1, 2.2, "hello\n");   // first: 1 2.2 hello
    print("\nsecond: ", 0.2, 'c', "yuck!"s, 0, 1, 2, '\n'); // second: 0.2 c yuck! 0 1 2
}
```

传统上，实现变参模板的方法是将第一个参数与其余参数分开，然后对参数包的尾部递归调用变参模板：

```cpp
template<typename T>
concept Printable = requires(T t) { std::cout << t; };   // 仅一个操作！

void print() { }   // 对于没有参数的情况：什么都不做

template<Printable T, Printable... Tail>
void print(T head, Tail... tail)
{
    cout << head << ' ';
    print(tail...);
}
```

`Printable...` 表示 `Tail` 是一个类型序列。`Tail...` 表示 `tail` 是一个值序列，这些值的类型在 `Tail` 中。用 `...` 声明的形参称为**参数包**。这里，`tail` 是一个（函数参数）参数包，其中的元素类型在（模板参数）参数包 `Tail` 中找到。因此，`print()` 可以接受任意数量的任意类型的参数。

调用 `print()` 将参数分为头部（第一个）和尾部（其余）。打印头部，然后对尾部调用 `print()`。最终，尾部当然会变空，因此我们需要无参数的 `print()` 版本来处理这种情况。如果我们不允许零参数的情况，可以使用编译时 `if` 来消除那个 `print()`：

```cpp
template<Printable T, Printable... Tail>
void print(T head, Tail... tail)
{
    cout << head << ' ';
    if constexpr (sizeof...(tail) > 0)
        print(tail...);
}
```

我使用了编译时 `if`（§7.4.3）而不是普通的运行时 `if`，以避免生成最终的 `print()` 调用。这样，“空”的 `print()` 就不必定义了。

变参模板的优势在于它们可以接受你愿意给它们的任何参数。缺点包括：
- 递归实现可能很难正确。
- 接口的类型检查可能是一个复杂的模板程序。
- 类型检查代码是临时的，而不是在标准中定义的。
- 递归实现可能在编译时间和编译器内存需求方面出奇地昂贵。

由于它们的灵活性，变参模板在标准库中被广泛使用，偶尔也被过度使用。

===== 第 20 页 =====

#### 8.4.1 折叠表达式

为了简化简单变参模板的实现，C++ 提供了对参数包元素进行有限形式的迭代。例如：

```cpp
template<Number... T>
int sum(T... v)
{
    return (v + ... + 0);   // 从 0 开始添加 v 的所有元素
}
```

这个 `sum()` 可以接受任意数量的任意类型的参数：

```cpp
int x = sum(1, 2, 3, 4, 5);   // x 变成 15
int y = sum('a', 2.4, x);     // y 变成 114（2.4 被截断，'a' 的值是 97）
```

`sum` 的主体使用了一个**折叠表达式**：

```cpp
return (v + ... + 0);   // 将 v 的所有元素加到 0 上
```

这里，`(v + ... + 0)` 表示从初始值 0 开始，添加 `v` 的所有元素。第一个被添加的元素是“最右边”的（索引最高的）：`(v[0] + (v[1] + (v[2] + (v[3] + (v[4] + 0)))))`。也就是说，从 0 所在的右边开始。这称为**右折叠**。或者，我们可以使用左折叠：

```cpp
template<Number... T>
int sum2(T... v)
{
    return (0 + ... + v);   // 将 v 的所有元素加到 0 上
}
```

现在，第一个被添加的元素是“最左边”的（索引最低的）：`((((0+v[0])+v[1])+v[2])+v[3])+v[4]`。也就是说，从 0 所在的左边开始。

折叠是一个非常强大的抽象，显然与标准库的 `accumulate()` 相关，在不同的语言和社区中有不同的名称。在 C++ 中，折叠表达式目前仅限于简化变参模板的实现。折叠不必执行数值计算。考虑一个著名的例子：

```cpp
template<Printable... T>
void print(T&... args)
{
    (std::cout << ... << args) << '\n';   // 打印所有参数
}

print("Hello!"s, ' ', "World ", 2017);   // (((std::cout << "Hello!"s) << ' ') << "World ") << 2017
```

为什么是 2017？因为 `fold()` 是在 2017 年添加到 C++ 中的（§19.2.3）。

#### 8.4.2 转发参数

通过接口原封不动地传递参数是变参模板的一个重要用途。考虑一个网络输入通道的概念，其中实际的值传递方法是参数化的。不同的传输机制有不同的构造函数参数集：

```cpp
template<concepts::InputTransport Transport>
class InputChannel {
public:
    // ...
    InputChannel(Transport::Args&&... transportArgs)
        : _transport(std::forward<TransportArgs>(transportArgs)...)
    { }
private:
    Transport _transport;
};
```

标准库函数 `forward()`（§16.6）用于将参数从 `InputChannel` 构造函数原封不动地移动到 `Transport` 构造函数。

这里的要点是，`InputChannel` 的编写者可以在不知道构造特定 `Transport` 需要什么参数的情况下构造一个 `Transport` 类型的对象。`InputChannel` 的实现者只需要知道所有 `Transport` 对象的共同用户接口。

转发在基础库中非常常见，在这些库中，通用性和低运行时开销是必需的，并且非常通用的接口很常见。
