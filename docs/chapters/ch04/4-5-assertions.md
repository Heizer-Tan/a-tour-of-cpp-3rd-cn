# 4.5 断言

目前还没有通用且标准的方式，来编写对不变式、前置条件等的可选运行时测试。然而，对许多大型程序来说，需要支持这样的用户：他们在测试时希望依赖大量运行时检查，但部署时又希望代码带有最少检查。

眼下，我们只能依赖各种特设机制。这类机制很多。它们需要灵活、通用，且在未启用时不带来开销。这意味着概念上要简单，实现上要精巧。下面是我用过的一种方案：

```cpp
enum class Error_action { ignore, throwing, terminating, logging }; // 错误处理备选

constexpr Error_action default_Error_action = Error_action::throwing; // 默认

enum class Error_code { range_error, length_error }; // 指示

string error_code_name[] { "range error", "length error" }; // 名字

template<Error_action action = default_Error_action, class C>
constexpr void expect(C cond, Error_code x) // 若期望的条件 "cond" 不成立则采取 "action"
{
    if constexpr (action == Error_action::logging)
        if (!cond())
            std::cerr << "expect() failure: " << int(x) << ' '
                      << error_code_name[int(x)] << '\n';
    if constexpr (action == Error_action::throwing)
        if (!cond()) throw x;
    if constexpr (action == Error_action::terminating)
        if (!cond()) terminate();
    // 或者不采取任何行动
}
```

乍一看这可能令人目眩，因为用到的许多语言特性尚未介绍。但正如所要求的，它既非常灵活，用起来又很简单。例如：

```cpp
double& Vector::operator[](int i)
{
    expect([i,this] { return 0<=i && i<size(); }, Error_code::range_error);
    return elem[i];
}
```

这检查下标是否在范围内，若不在则采取默认动作——抛出异常。期望成立的条件 `0<=i&&i<size()` 作为 lambda 传给 `expect()`：`[i,this]{return 0<=i&&i<size();}`（[§7.3.3](../ch07/7-3-parameterized-operations.md#7.3.3)）。`if constexpr` 测试在编译时完成（[§7.4.3](../ch07/7-4-template-mechanisms.md#7.4.3)），因此每次调用 `expect()` 至多执行一次运行时测试。把 `action` 设为 `Error_action::ignore`，则不采取任何动作，也不为 `expect()` 生成代码。

通过设置 `default_Error_action`，用户可以为程序的某次特定部署选择合适的动作，例如终止或记录日志。为支持日志记录，需要定义 `error_code_name` 表。日志信息还可以通过使用 `source_location`（[§16.5](../ch16/16-5-source-location.md)）加以改进。

在许多系统中，重要的是断言机制（如 `expect()`）为断言失败的含义提供单一控制点。在大型代码库中搜索那些其实是在检查假定的 `if` 语句，通常并不现实。

## 4.5.1 assert()

标准库提供调试宏 `assert()`，用于断言某个条件在运行时必须成立。例如：

```cpp
void f(const char* p)
{
    assert(p!=nullptr);    // p 不得为 nullptr
    // ...
}
```

若 `assert()` 的条件在“调试模式”下失败，程序终止。若不在调试模式，则不检查该 `assert()`。这相当粗糙且不灵活，但往往总比什么都没有好。

## 4.5.2 静态断言

异常报告的是在运行时发现的错误。若错误能在编译时发现，通常更可取。类型系统以及规定用户定义类型接口的许多设施，正是为此服务的。不过，我们也可以对大多数在编译时已知的性质做简单检查，并把未能满足期望的情况报告为编译器错误消息。例如：

```cpp
static_assert(4<=sizeof(int), "integers are too small");    // 检查整数大小
```

若 `4<=sizeof(int)` 不成立——也就是说，若本系统上的 `int` 没有至少 4 个字节——就会输出 `integers are too small`。我们把这类期望陈述称为断言。

`static_assert` 机制可用于任何能用常量表达式表达的性质（[§1.6](../ch01/1-6-constants.md)）。例如：

```cpp
constexpr double C = 299792.458;                        // km/s

void f(double speed)
{
    constexpr double local_max = 160.0/(60*60);         // 160 km/h == 160.0/(60*60) km/s

    static_assert(speed<C,"can't go that fast");        // 错误：speed 必须是常量
    static_assert(local_max<C,"can't go that fast");    // OK

    // ...
}
```

一般而言，若 `A` 不为真，`static_assert(A,S)` 会把 `S` 作为编译器错误消息打印出来。若不想打印特定消息，可省略 `S`，编译器会提供默认消息：

```cpp
static_assert(4<=sizeof(int));            // 使用默认消息
```

默认消息通常是该 `static_assert` 的源码位置，外加所断言谓词的字符表示。

`static_assert` 的一个重要用途，是对泛型编程中用作参数的类型作出断言（[§8.2](../ch08/8-2-concepts.md)、[§16.4](../ch16/16-4-type-functions.md)）。

## 4.5.3 noexcept

永不应当抛出异常的函数可以声明为 `noexcept`。例如：

```cpp
void user(int sz) noexcept
{
    Vector v(sz);
    iota(&v[0],&v[sz],1);           // 用 1,2,3,4... 填充 v（见 §17.3）
    // ...
}
```

这里的 `iota` 见 [§17.3](../ch17/17-3-numeric-algorithms.md)。若一切良好意图和规划都失败了，以至于 `user()` 仍然抛出异常，则会调用 `std::terminate()` 立即终止程序。

不动脑筋地在函数上洒满 `noexcept` 是危险的。若一个 `noexcept` 函数调用了某个会抛出异常、并期望该异常被捕获和处理的函数，`noexcept` 会把它变成致命错误。此外，`noexcept` 迫使编写者通过某种可能复杂、易错且昂贵的错误码形式来处理错误（[§4.4](4-4-alternatives.md)）。像其他强大的语言特性一样，`noexcept` 应当在理解并谨慎的前提下使用。
