===== 第 1 页 =====

16

# 工具

你乐于浪费的时间，算不上虚度光阴。 – 伯特兰·罗素

- 引言
- 时间
  - 时钟；日历；时区
- 函数适配
  - Lambda 作为适配器；mem_fn()；function
- 类型函数
  - 类型谓词；条件属性；类型生成器；关联类型
- source_location
- move() 与 forward()

===== 第 2 页 =====

 - 位操作
 - 程序退出
 - 建议

## 16.1 引言

将某个库组件标记为“工具”并不能提供太多信息。显然，每个库组件都曾在某个时间、某个地方对某人有实用价值。这里选择介绍的设施是因为它们为许多人提供了关键用途，但其描述又不适合放在其他地方。它们通常充当更强大库设施（包括标准库的其他组件）的构建块。

## 16.2 时间

在 `<chrono>` 中，标准库提供了处理时间的设施：

- `clock`、`time_point` 和 `duration`，用于测量某个操作花费的时间，并作为与时间相关的任何事物的基础。
- `day`、`month`、`year` 和 `weekday`，用于将 `time_point` 映射到我们的日常生活中。
- `time_zone` 和 `zoned_time`，用于处理全球时间报告的差异。几乎所有主要系统都会处理这些实体中的一部分。

### 16.2.1 时钟

以下是对某个操作进行计时的基本方法：

```cpp
using namespace std::chrono;   // 在子命名空间 std::chrono 中；见 §3.3

auto t0 = system_clock::now();
do_work();
auto t1 = system_clock::now();

cout << t1 - t0 << "\n";                           // 默认单位
cout << duration_cast<milliseconds>(t1-t0).count() << "ms\n";   // 指定单位
cout << duration_cast<nanoseconds>(t1-t0).count() << "ns\n";    // 指定单位
```

===== 第 3 页 =====

时钟返回一个 `time_point`（时间点）。两个 `time_point` 相减得到一个 `duration`（一段时间）。`duration` 的默认 `<<` 会添加一个表示所用单位的后缀。不同的时钟以不同的时间单位（“时钟滴答”）给出结果（我使用的时钟以百纳秒为单位），因此通常最好将 `duration` 转换为适当的单位。这就是 `duration_cast` 的作用。

时钟对于快速测量很有用。不要在没有进行时间测量的情况下对代码的“效率”做出断言。关于性能的猜测是最不可靠的。快速、简单的测量总比没有测量好，但现代计算机的性能是一个棘手的课题，因此我们必须小心，不要过于重视少数简单的测量。始终重复测量，以降低被罕见事件或缓存效应蒙蔽的机会。

命名空间 `std::chrono_literals` 定义了时间单位后缀（§6.6）。例如：

```cpp
this_thread::sleep_for(10ms + 33us);   // 等待 10 毫秒 33 微秒
```

传统的符号名称极大地提高了可读性，并使代码更易于维护。

#### 16.2.2 日历

在处理日常事件时，我们很少使用毫秒；我们使用年、月、日、小时、秒和星期几。标准库支持这一点。例如：

```cpp
auto spring_day = April/7/2018;
cout << weekday(spring_day) << '\n';                // Sat
cout << format("{:%A}\n", weekday(spring_day));     // Saturday
```

===== 第 4 页 =====

`Sat` 是我电脑上星期六的默认字符表示。我不喜欢那个缩写，因此我使用了 `format`（§11.6.2）来获得更长的名称。出于晦涩的原因，`%A` 表示“写出星期几的全名”。自然，`April` 是一个月份；更准确地说，是 `std::chrono::Month`。我们也可以这样说：

```cpp
auto spring_day = 2018y/April/7;
```

后缀 `y` 用于将年份与普通的 `int`（用于表示 1 到 31 的日期）区分开。

可以表达无效的日期。如果有疑问，可以使用 `ok()` 进行检查：

```cpp
auto bad_day = January/0/2024;
if (!bad_day.ok())
    cout << bad_day << " is not a valid day\n";
```

显然，`ok()` 对从计算得到的日期最有用。

日期通过重载 `operator/`（斜杠）与类型 `year`、`month` 和 `int` 来组合。得到的 `year_month_day` 类型与 `time_point` 之间具有转换关系，以允许涉及日期的精确高效计算。例如：

```cpp
sys_days t = sys_days{February/25/2022};        // 获取一个 time_point，精度为 days
t += days{7};                                   // 2022 年 2 月 25 日之后一周
auto d = year_month_day(t);                     // 将 time_point 转换回日历日期

cout << d << '\n';                              // 2022-03-04
cout << format("{:%B}/{}/{}\n", d.month(), d.day(), d.year()); // March/04/2022
```

这个计算需要月份的变化和关于闰年的知识。默认情况下，实现以 ISO 8601 标准格式给出日期。要拼出“March”这样的月份名称，我们必须分解日期的各个字段并深入了解格式细节（§11.6.2）。出于晦涩的原因，`%B` 表示“写出月份的全名”。

这样的操作通常可以在编译时完成，因此出人意料地快：

```cpp
static_assert(weekday(April/7/2018) == Saturday);   // true
```

===== 第 5 页 =====

日历是复杂而微妙的。这对于为“普通人”设计的“系统”来说是典型且合适的，这些系统经过了几百年的演变，而不是由程序员为了简化编程而设计的。标准库的日历系统可以（并且已经）被扩展以处理儒略历、伊斯兰历、泰历和其他历法。

#### 16.2.3 时区

与时间相关的最棘手的问题之一就是时区。它们是如此随意，以至于难以记忆，并且它们的各种变化方式在全球范围内没有标准化。例如：

```cpp
auto tp = system_clock::now();                  // tp 是一个 time_point
cout << tp << '\n';                             // 2021-11-27 21:36:08.2085095

zoned_time ztp { current_zone(), tp };
cout << ztp << '\n';                            // 2021-11-27 16:36:08.2085095 EST

const time_zone est {"Europe/Copenhagen"};
cout << zoned_time{ &est, tp } << '\n';         // 2021-11-27 22:36:08.2085095 GMT+1
```

`time_zone` 是相对于 `system_clock` 使用的标准时间（称为 GMT 或 UTC）的时间。标准库与一个全局数据库（IANA）同步以得到正确的答案。这种同步可以由操作系统自动完成，或者由系统管理员控制。时区的名称是形式为“洲/主要城市”的 C 风格字符串，例如 `"America/New_York"`、`"Asia/Tokyo"`、`"Africa/Nairobi"`。`zoned_time` 是一个 `time_zone` 加上一个 `time_point`。

像日历一样，时区涉及一系列我们应该交给标准库处理的问题，而不是依赖我们自己手工编写的代码。想一想：2024 年 2 月最后一天，纽约的什么时间，新德里的日期会发生改变？2020 年美国科罗拉多州丹佛市的夏令时何时结束？下一个闰秒何时发生？标准库“知道”。

===== 第 6 页 =====

## 16.3 函数适配

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

## 16.4 类型函数

**类型函数**是在编译时求值的函数，它以类型为参数或返回一个类型。标准库提供了多种类型函数，以帮助库实现者（以及一般程序员）编写能够利用语言、标准库和一般代码各方面特性的代码。

对于数值类型，`<limits>` 中的 `numeric_limits` 提供了各种有用的信息（§17.7）。例如：

```cpp
constexpr float min = numeric_limits<float>::min();   // 最小的正 float
```

类似地，对象大小可以通过内置的 `sizeof` 运算符获得（§1.4）。例如：

```cpp
constexpr int szi = sizeof(int);   // int 的字节数
```

在 `<type_traits>` 中，标准库提供了许多用于查询类型属性的函数。例如：

```cpp
bool b = is_arithmetic_v<X>;        // 如果 X 是（内置）算术类型之一，则为 true
using Res = invoke_result_t<decltype(f)>; // 如果 f 是返回 int 的函数，则 Res 是 int
```

`decltype(f)` 是调用内置类型函数 `decltype()`，它返回其参数声明的类型；这里参数是 `f`。

一些类型函数基于输入创建新的类型。例如：

```cpp
template<typename T>
using Store = conditional_t<(sizeof(T) < max), On_stack<T>, On_heap<T>>;
```

如果 `conditional_t` 的第一个（布尔）参数为真，则结果是第一个备选；否则是第二个。假设 `On_stack` 和 `On_heap` 提供相同的访问 `T` 的函数，它们可以按名字所示的方式分配其 `T`。因此，`Store<X>` 的用户可以根据 `X` 对象的大小进行调优。这种分配选择所启用的性能调优可能非常显著。

===== 第 9 页 =====

这是一个简单的例子，说明我们如何使用标准类型函数或概念来构造自己的类型函数。概念就是类型函数。当在表达式中使用时，它们特指**类型谓词**。例如：

```cpp
template<typename F, typename... Args>
auto call(F f, Args... a, Allocator alloc)
{
    if constexpr (invocable<F, alloc, Args...>)   // 需要分配器吗？
        return f(f, alloc, a...);
    else
        return f(f, a...);
}
```

在许多情况下，概念是最好的类型函数，但大多数标准库是在概念之前编写的，并且必须支持概念之前的代码库。

表示法约定令人困惑。标准库使用 `_v` 表示返回值类型函数，使用 `_t` 表示返回类型类型函数。这是 C 弱类型时代和概念前 C++ 的遗留物。没有标准库类型函数既返回类型又返回值，因此这些后缀是多余的。有了概念，无论是在标准库还是其他地方，都不需要或使用后缀。

类型函数是 C++ 编译时计算机制的一部分，它们允许比没有它们时更严格的类型检查和更好的性能。类型函数和概念（第 8 章，§14.5）的使用通常称为**元编程**，或者在涉及模板时称为**模板元编程**。

## 16.4.1 类型谓词

在 `<type_traits>` 中，标准库提供了几十种简单的类型函数，称为**类型谓词**，它们回答有关类型的基本问题。以下是其中一小部分：

**选定的类型谓词**（`T`、`A` 和 `U` 是类型；所有谓词返回 `bool`）

| 谓词 | 描述 |
|------|------|
| `is_void_v<T>` | `T` 是 `void` 吗？ |
| `is_integral_v<T>` | `T` 是整数类型吗？ |
| `is_floating_point_v<T>` | `T` 是浮点类型吗？ |
| `is_class_v<T>` | `T` 是类（且不是联合）吗？ |
| `is_function_v<T>` | `T` 是函数（而不是函数对象或指向函数的指针）吗？ |
| `is_arithmetic_v<T>` | `T` 是整数或浮点类型吗？ |
| `is_scalar_v<T>` | `T` 是算术、枚举、指针或指向成员的指针类型吗？ |
| `is_constructible_v<T, A...>` | `T` 能否从 `A...` 参数列表构造？ |
| `is_default_constructible_v<T>` | `T` 能否在没有显式参数的情况下构造？ |
| `is_copy_constructible_v<T>` | `T` 能否从另一个 `T` 构造？ |
| `is_move_constructible_v<T>` | `T` 能否被移动或拷贝到另一个 `T` 中？ |
| `is_assignable_v<T, U>` | `U` 能否赋值给 `T`？ |
| `is_trivially_copyable_v<T, U>` | `U` 能否在没有用户定义拷贝操作的情况下赋值给 `T`？ |
| `is_same_v<T, U>` | `T` 与 `U` 是同一类型吗？ |
| `is_base_of_v<T, U>` | `U` 是否派生自 `T`，或者 `U` 与 `T` 是同一类型？ |
| `is_convertible_v<T, U>` | `T` 能否隐式转换为 `U`？ |
| `is_iterator_v<T>` | `T` 是迭代器类型吗？ |
| `is_invocable_v<T, A...>` | `T` 能否用参数列表 `A...` 调用？ |
| `has_virtual_destructor_v<T>` | `T` 是否有虚析构函数？ |

这些谓词的一个传统用途是约束模板参数。例如：

```cpp
template<typename Scalar>
class complex {
    Scalar re, im;
public:
    static_assert(is_arithmetic_v<Scalar>, "Sorry, I support only complex of arithmetic types");
    // ...
};
```

===== 第 11 页 =====

然而，这——像其他传统用法一样——使用概念更简单、更优雅：

```cpp
template<Arithmetic Scalar>
class complex {
    Scalar re, im;
public:
    // ...
};
```

在许多情况下，诸如 `is_arithmetic` 之类的类型谓词会消失在概念的定义中，以更易于使用。例如：

```cpp
template<typename T>
concept Arithmetic = is_arithmetic_v<T>;
```

奇怪的是，标准库中没有 `std::arithmetic` 概念。

通常，我们可以定义比标准库类型谓词更通用的概念。许多标准库类型谓词仅适用于内置类型。我们可以根据所需的操作来定义概念，正如 `Number` 的定义所建议的那样（§8.2.4）：

```cpp
template<typename T, typename U = T>
concept Arithmetic = Number<T, U> && Number<U, T>;
```

大多数情况下，标准库类型谓词的使用出现在基础服务的实现深处，通常是为了区分不同的优化情况。例如，`std::copy(Iter, Iter, Iter2)` 的部分实现可以优化简单类型（如整数）的连续序列这一重要情况：

```cpp
template<class T>
void cpy1(T* first, T* last, T* target)
{
    if constexpr (is_trivially_copyable_v<T>)
        memcpy(target, first, (last - first) * sizeof(T));
    else
        while (first != last)
            *target++ = *first++;
}
```

在某些实现上，这个简单的优化比其非优化版本快了约 50%。除非你已经验证了标准库没有做得更好，否则不要沉迷于这种小聪明。手工优化的代码通常比更简单的替代方案更难维护。

## 16.4.2 条件属性

考虑定义一个“智能指针”：

```cpp
template<typename T>
class Smart_pointer {
    // ...
    T& operator*() const;
    T* operator->() const;   // -> 应该当且仅当 T 是类类型时才有效
};
```

`->` 应该当且仅当 `T` 是类类型时才定义。例如，`Smart_pointer<vector<T>>` 应该有 `->`，但 `Smart_pointer<int>` 不应该。

我们不能使用编译时 `if`，因为我们不在函数内部。相反，我们写：

```cpp
template<typename T>
class Smart_pointer {
    // ...
    T& operator*() const;
    T* operator->() const requires is_class_v<T>;   // -> 当且仅当 T 是类类型时定义
};
```

===== 第 13 页 =====

类型谓词直接表达了对 `operator->()` 的约束。我们也可以使用概念来实现这一点。标准库中没有要求一个类型是类类型（即类是 class、struct 或 union）的概念，但我们可以自己定义一个：

```cpp
template<typename T>
concept Class = is_class_v<T> || is_union_v<T>;   // union 也是类

template<typename T>
class Smart_pointer {
    // ...
    T& operator*() const;
    T* operator->() const requires Class<T>;   // -> 当且仅当 T 是类类型时定义
};
```

通常，概念比直接使用标准库类型谓词更通用，或者仅仅更合适。

## 16.4.3 类型生成器

许多类型函数返回类型，通常是它们计算出的新类型。我称这些函数为**类型生成器**，以区别于类型谓词。标准库提供了几个，例如：

**选定的类型生成器**

| 生成器 | 描述 |
|--------|------|
| `R = remove_const_t<T>` | `R` 是去除了顶层 `const`（如果有）的 `T` |
| `R = add_const_t<T>` | `R` 是 `const T` |
| `R = remove_reference_t<T>` | 如果 `T` 是引用 `U&`，则 `R` 是 `U`，否则 `R` 是 `T` |
| `R = add_lvalue_reference_t<T>` | 如果 `T` 是左值引用，则 `R` 是 `T`，否则 `R` 是 `T&` |
| `R = add_rvalue_reference_t<T>` | 如果 `T` 是右值引用，则 `R` 是 `T`，否则 `R` 是 `T&&` |
| `R = enable_if_t<b, T = void>` | 如果 `b` 为真，则 `R` 是 `T`，否则 `R` 未定义 |
| `R = conditional_t<b, T, U>` | 如果 `b` 为真，则 `R` 是 `T`，否则 `R` 是 `U` |
| `R = common_type_t<T...>` | 如果存在一个所有 `T` 都可以隐式转换到的类型，则 `R` 是该类型；否则 `R` 未定义 |
| `R = underlying_type_t<T>` | 如果 `T` 是枚举，则 `R` 是其底层类型；否则错误 |
| `R = invoke_result_t<T, A...>` | 如果 `T` 可以用参数 `A...` 调用，则 `R` 是其返回类型；否则错误 |

这些类型函数通常用于工具的实现中，而不是直接在应用程序代码中使用。其中，`enable_if` 可能是概念前代码中最常见的。例如，智能指针的条件启用 `->` 传统上是这样实现的：

```cpp
template<typename T>
class Smart_pointer {
    // ...
    T& operator*();
    enable_if<is_class_v<T>, T&> operator->();   // -> 当且仅当 T 是类类型时定义
};
```

我发现这不太容易阅读，更复杂的使用情况则更糟。`enable_if` 的定义依赖于一种称为 SFINAE（“替换失败不是错误”）的微妙语言特性。只有在需要时才去查阅它。

## 16.4.4 关联类型

所有标准容器（§12.8）以及所有遵循其模式设计的容器都有一些关联类型，例如它们的值类型和迭代器类型。在 `<iterator>` 和 `<ranges>` 中，标准库为这些关联类型提供了名称：

**选定的类型生成器**

| 类型生成器 | 描述 |
|--------|------|
| `range_value_t<R>` | 范围 `R` 的元素类型 |
| `iter_value_t<T>` | 迭代器 `T` 指向的元素类型 |
| `iterator_t<R>` | 范围 `R` 的迭代器类型 |

## 16.5 source_location

当输出跟踪消息或错误消息时，我们通常希望将源代码位置作为消息的一部分。库为此提供了 `source_location`：

```cpp
const source_location loc = source_location::current();
```

`current()` 返回一个描述源代码中出现位置的 `source_location`。类 `source_location` 有成员 `file_name()` 和 `function_name()`（返回 C 风格字符串），以及 `line()` 和 `column()`（返回无符号整数）。

将其包装在一个函数中，我们就得到了一个不错的日志记录消息的初步版本：

```cpp
void log(const string& mess = "", const source_location loc = source_location::current())
{
    cout << loc.file_name()
         << '(' << loc.line() << ':' << loc.column() << ") "
         << loc.function_name() << ": " << mess;
}
```

对 `current()` 的调用是一个默认参数，因此我们得到的是 `log()` 的调用者的位置，而不是 `log()` 本身的位置：

```cpp
void foo()
{
    log("Hello");   // myfile.cpp (17,4) foo: Hello
    // ...
}

int bar(const string& label)
{
    log(label);     // myfile.cpp (23,4) bar: <<the value of label>>
    // ...
}
```

===== 第 16 页 =====

在 C++20 之前编写的代码或需要在旧编译器上编译的代码使用宏 `__FILE__` 和 `__LINE__` 来实现此目的。

## 16.6 move() 与 forward()

移动和拷贝之间的选择主要是隐式的（§3.4）。当一个对象即将被销毁时（如在返回语句中），编译器会优先选择移动，因为这被认为是更简单、更高效的操作。然而，有时我们必须显式指定。例如，`unique_ptr` 是其对象的唯一所有者。因此，它不能被拷贝，所以如果你想在其他地方拥有一个 `unique_ptr`，你必须移动它。例如：

```cpp
void f1()
{
    auto p = make_unique<int>(2);
    auto q = p;          // 错误：不能拷贝 unique_ptr
    auto q = move(p);    // p 现在持有 nullptr
    // ...
}
```

令人困惑的是，`std::move()` 实际上并不移动任何东西。相反，它将参数强制转换为右值引用，从而表示该参数将不再被使用，因此可以被移动（§6.2.2）。它本应该被称为类似 `rvalue_cast` 的东西。它的存在是为了服务于几个必要的场景。考虑一个简单的 `swap`：

```cpp
template <typename T>
void swap(T& a, T& b)
{
    T tmp {move(a)};   // T 的构造函数看到右值并移动
    a = move(b);       // T 的赋值运算符看到右值并移动
    b = move(tmp);     // T 的赋值运算符看到右值并移动
}
```

我们不想重复拷贝可能很大的对象，因此使用 `std::move()` 请求移动。

与其他强制转换一样，`std::move()` 也有诱人但危险的使用方式。考虑：

```cpp
string s1 = "Hello";
string s2 = "World";
vector<string> v;
v.push_back(s1);           // 使用 "const string&" 参数；push_back() 将拷贝
v.push_back(move(s2));     // 使用移动构造函数
v.emplace_back(s1);        // 另一种方式；将 s1 的副本放置在新的末尾位置
```

这里 `s1` 被拷贝（通过 `push_back()`），而 `s2` 被移动。这有时（仅有时）会使 `s2` 的 `push_back()` 更便宜。问题在于，被移动的对象被留在了后面。如果我们再次使用 `s2`，我们就会遇到问题：

```cpp
cout << s1[2];   // 输出 'l'
cout << s2[2];   // 崩溃？
```

我认为这种 `std::move()` 的用法太容易出错，不适合广泛使用。除非你能证明其显著且必要的性能提升，否则不要使用它。后续的维护可能会意外地导致对已移动对象的未预期使用。

编译器知道返回值在函数中不会被再次使用，因此使用显式的 `std::move()`（例如 `return std::move(x);`）是多余的，甚至可能抑制优化。

移动后对象的状态通常未指定，但所有标准库类型都会将移动后对象置于一种可以销毁和赋值的状态。不遵循这一先例是不明智的。对于容器（如 `vector` 或 `string`），移动后状态将是“空”的。对于许多类型，默认值是一个很好的空状态：有意义且易于建立。

转发参数是一个需要移动的重要用例（§8.4.2）。有时我们希望将一组参数原封不动地传递给另一个函数（以实现“完美转发”）：

```cpp
template<typename T, typename... Args>
unique_ptr<T> make_unique(Args&&... args)
{
    return unique_ptr<T>{new T{std::forward<Args>(args)...}};   // 转发每个参数
}
```

标准库的 `forward()` 与更简单的 `std::move()` 不同，它正确处理了与左值和右值相关的微妙之处（§6.2.2）。仅将 `std::forward()` 用于转发，并且不要对同一个对象转发两次；一旦你转发了某个对象，它就不再属于你使用了。

## 16.7 位操作

在 `<bit>` 中，我们找到了用于底层位操作的函数。位操作是一项专门但通常必不可少的活动。当我们接近硬件时，我们经常需要查看位、改变字节或字中的位模式，以及将原始内存转换为类型化对象。例如，`bit_cast` 允许我们将一个类型的值转换为另一个相同大小的类型的值：

```cpp
double val = 7.2;
auto x = bit_cast<uint64_t>(val);   // 获取 64 位浮点值的位表示
auto y = bit_cast<uint64_t>(&val);  // 获取 64 位指针的位表示

struct Word { std::byte b[8]; };
std::byte buffer[1024];
// ...
auto p = bit_cast<Word*>(&buffer[i]);   // p 指向 8 个字节
auto i = bit_cast<int64_t>(*p);         // 将这 8 个字节转换为整数
```

标准库类型 `std::byte`（`std::` 是必需的）的存在是为了表示字节，而不是已知表示字符或整数的字节。特别地，`std::byte` 只提供按位逻辑操作，而不提供算术操作。通常，进行位操作的最佳类型是无符号整数或 `std::byte`。所谓最佳，我指的是最快且最不容易出现意外。例如：

```cpp
void use(unsigned int ui)
{
    int x0 = bit_width(ui);          // 表示 ui 所需的最小位数
    unsigned int ui2 = rotl(ui, 8);  // 左旋 8 位（注意：不改变 ui）
    int x1 = popcount(ui);           // ui 中 1 的个数
    // ...
}
```

另请参见 `bitset`（§15.3.2）。

## 16.8 程序退出

偶尔，一段代码会遇到它无法处理的问题：

- 如果问题的类型很常见，并且可以预期直接调用者能够处理它，那么返回某种返回码（§4.4）。
- 如果问题的类型不常见，或者不能预期直接调用者能够处理它，那么抛出异常（§4.4）。
- 如果问题的类型非常严重，以至于不能预期程序的任何普通部分能够处理它，那么退出程序。

标准库提供了处理最后一种情况（“退出程序”）的设施：

- `exit(x)`：调用用 `atexit()` 注册的函数，然后以返回值 `x` 退出程序。如果需要，可以查阅 `atexit()`，它基本上是 C 语言共享的一种原始析构函数机制。
- `abort()`：立即无条件地退出程序，并返回一个表示不成功终止的返回值。一些操作系统提供修改这个简单解释的设施。
- `quick_exit(x)`：调用用 `at_quick_exit()` 注册的函数，然后以返回值 `x` 退出程序。
- `terminate()`：调用 `terminate_handler`。默认的 `terminate_handler` 是 `abort()`。

===== 第 20 页 =====

这些函数用于真正严重的错误。它们不会调用析构函数；也就是说，它们不执行普通且正确的清理。各种处理程序用于在退出前采取行动。这样的操作必须非常简单，因为调用这些退出函数的一个原因是程序状态已损坏。一个合理且相当流行的操作是“在与当前程序无关的明确定义状态下重新启动系统”。另一个稍微冒险但通常并非不合理的操作是“记录错误消息并退出”。写入日志消息可能成为问题的原因是，导致调用退出函数的任何原因可能已经破坏了 I/O 系统。

错误处理是最棘手的编程类型之一；即使是干净地退出程序也可能很难。

没有通用库应该无条件终止。

## 16.9 建议

[1] 一个库不必庞大或复杂才能有用；§16.1。
[2] 在对效率做出断言之前，对你的程序进行计时；§16.2.1。
[3] 使用 `duration_cast` 以适当的单位报告时间测量结果；§16.2.1。
[4] 要在源代码中直接表示日期，请使用符号表示法（例如 `November/28/2021`）；§16.2.2。
[5] 如果日期是计算的结果，请使用 `ok()` 检查其有效性；§16.2.2。
[6] 当处理不同地点的时区时，请使用 `zoned_time`；§16.2.3。
[7] 使用 lambda 来表达调用约定上的微小变化；§16.3.1。
[8] 使用 `mem_fn()` 或 lambda 来创建函数对象，这些函数对象在使用传统函数调用记法调用时可以调用成员函数；§16.3.1，§16.3.2。
[9] 当需要存储可调用对象时，使用 `function`；§16.3.3。
[10] 相对于显式使用类型谓词，优先使用概念；§16.4.1。

===== 第 21 页 =====

[11] 你可以编写明确依赖于类型属性的代码；§16.4.1，§16.4.2。
[12] 只要可能，优先使用概念而不是 traits 和 `enable_if`；§16.4.3。
[13] 使用 `source_location` 将源代码位置嵌入调试和日志消息中；§16.5。
[14] 避免显式使用 `std::move()`；§16.6；[CG: ES.56]。
[15] 仅将 `std::forward()` 用于转发；§16.6。
[16] 在对一个对象执行 `std::move()` 或 `std::forward()` 之后，永远不要读取它；§16.6。
[17] 使用 `std::byte` 表示（尚）没有有意义的类型的数据；§16.7。
[18] 使用无符号整数或 `bitset` 进行位操作；§16.7。
[19] 如果预期直接调用者可以处理问题，则从函数返回错误码；§16.8。
[20] 如果不能预期直接调用者可以处理问题，则从函数抛出异常；§16.8。
[21] 如果尝试从问题中恢复是不合理的，则调用 `exit()`、`quick_exit()` 或 `terminate()` 来退出程序；§16.8。
[22] 没有通用库应该无条件终止；§16.8。