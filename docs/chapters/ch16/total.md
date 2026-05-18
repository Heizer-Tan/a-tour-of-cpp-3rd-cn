# 实用工具

> **你乐于“浪费”的时间并非浪费时间。**
>
> —— Bertrand Russell

# 16.1 引言

把某个库组件贴上「工具」标签并没有太大信息量。显然，每个库组件都曾在某个时刻、某个地方对某些人有用。这里介绍的设施之所以被选中，是因为它们对许多人至关重要，但其说明又不适合放在别处。它们常常充当更强大库设施（包括标准库的其他组件）的构造单元。

# 16.2 时间

在 `<chrono>` 中，标准库提供了与时间有关的设施：时钟、`time_point` 与 `duration`，用于度量某项操作耗时，并作为一切与时间相关计算的基础；`day`、`month`、`year` 以及工作日，用于把 `time_point` 映射到日常生活；`time_zone` 与 `zoned_time`，用于处理全球各地在时间报告上的差异。几乎所有主要系统都会涉及这些实体中的若干种。

## 16.2.1

度量某项操作耗时的基本写法如下：

```cpp
using namespace std::chrono;         // 位于 std::chrono 子命名空间；见 §3.3

auto t0 = system_clock::now();
do_work();
auto t1 = system_clock::now();
```

时钟返回的是 `time_point`（时间点）。两个 `time_point` 相减得到 `duration`（时间段）。`duration` 默认的 `<<` 会以后缀形式附带所用单位的提示。各种时钟以不同的时间单位给出结果——「时钟滴答」（我所用的时钟以百分之一纳秒为单位度量），因此通常最好把 `duration` 转换成合适的单位，这正是 `duration_cast` 的作用。

时钟便于做快速测量。在没有先做计时测量之前，不要对代码的「效率」下结论。对性能的猜测大多不可靠。快速而简单的测量总比没有测量好，但现代计算机的性能是个棘手话题，因此要小心，别把寥寥几次简单测量看得过重。应反复测量，以降低被罕见事件或缓存效应蒙蔽的概率。

命名空间 `std::chrono_literals` 定义了时间单位后缀（§6.6）。例如可以写出 `200ms`、`24h` 一类字面量。相较无名魔术常量，这类符号名字大大提高可读性，也使代码更易维护。

## 16.2.2

处理日常事件时，我们很少用毫秒；我们用年、月、日、小时、秒以及星期几。标准库对此提供支持。例如：

```cpp
auto spring_day = April/7/2018;
cout << weekday(spring_day) << '\n';                            // Sat
cout << format("{:%A}\n", weekday(spring_day));                 // Saturday

cout << t1 - t0 << "\n";                                        // 默认单位：……
cout << duration_cast<milliseconds>(t1 - t0).count() << "ms\n"; // 指定单位
cout << duration_cast<nanoseconds>(t1 - t0).count() << "ns\n"; // 指定单位
this_thread::sleep_for(10ms + 33us);                            // 等待 10 毫秒加 33 微秒
```

`Sat` 是我这台计算机上星期六的默认字符表示。我不喜欢这种缩写，于是用 `format`（§11.6.2）得到更长名称。出于晦涩的历史原因，`%A` 表示「写出星期几的全名」。自然，`April` 是一个月份；更确切地说，它是 `std::chrono::month`。我们也可以写成：

```cpp
auto spring_day = 2018y/April/7;
```

后缀 `y` 用来把年份与普通 `int`（表示月中从 1 到 31 的日序）区分开。

有可能写出非法日期。若有疑虑，用 `ok()` 检查：

```cpp
auto bad_day = January/0/2024;
if (!bad_day.ok())
    cout << bad_day << " is not a valid day\n";
```

显然，对于通过计算得到的日期，`ok()` 最有用。

日期通过对类型 `year`、`month` 与 `int` 重载运算符 `/`（斜杠）拼合而成。得到的 `year_month_day` 类型可与 `time_point` 相互转换，从而能对日期进行准确而高效的计算；这其中往往需要跨月并正确处理闰年。默认情况下，实现会以 ISO 8601 标准格式给出日期。若要把月份拼写成「March」，就必须拆出日期的各个字段并涉足格式化细节（§11.6.2）。出于晦涩原因，`%B` 表示「写出月份全名」。

此类操作常常可在编译期完成，因而快得出奇：

```cpp
sys_days t = sys_days{February/25/2022};     // 取得某日期的 time_point（精度为……）
t += days{7};                                   // 2022 年 2 月 25 日之后一周
auto d = year_month_day(t);                   // 再把时间点换回日历日期

cout << d << '\n';                            // 2022-03-04
cout << format("{:%B}/{}/{}\n", d.month(), d.day(), d.year()); // March/04/2022

static_assert(weekday(April/7/2018) == Saturday);   // 成立
```

日历复杂而微妙。这对专为「普通人」历经数百年演化而成的「体系」而言很典型，也很恰当——并非程序员为了简化编程而设计。标准库的日历系统可以（并且已经）扩展以应对儒略历、伊斯兰历、泰国历等其他历法。

## 16.2.3

与时间相关的议题里，最难做对之一是时区。它们任意性强，难以记忆，而且会以种种方式在各地并非标准化的时刻发生变化。

`time_zone` 表示相对于某种标准（常称为 GMT 或 UTC）的时间偏移，供 `system_clock` 使用。标准库与全球数据库（IANA）同步以给出正确结果；同步可以由操作系统自动完成，也可由系统管理员控制。时区名字是 C 风格字符串，形如「洲／主要城市」，例如 `"America/New_York"`、`"Asia/Tokyo"`、`"Africa/Nairobi"`。`zoned_time` 则是 `time_zone` 与一个 `time_point` 的组合。

与日历一样，时区涉及我们应交给标准库、而非靠自己手写代码去对付的一组关注点。想一想：2024 年 2 月最后一天纽约当地的什么时间，新德里的日期会更替？2020 年美国科罗拉多州丹佛的夏令时何时结束？下一次闰秒何时发生？标准库「知道」这些答案。

```cpp
auto tp = system_clock::now();              // tp 为 time_point
cout << tp << '\n';                         // 例如：2021-11-27 21:36:08.2085095

zoned_time ztp{current_zone(), tp};        // 例如：2021-11-27 16:36:08.2085095 EST
cout << ztp << '\n';

const time_zone est{"Europe/Copenhagen"};
cout << zoned_time{&est, tp} << '\n';      // 例如：2021-11-27 22:36:08.2085095 GMT+……
```

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

# 16.4 类型函数

类型函数是在编译期求值的函数：以类型为实参，或返回类型。标准库提供多种类型函数，帮助库实现者（以及广大程序员）写出能够利用语言特性、标准库以及一般代码方方面面的程序。

对算术类型，`<limits>` 中的 `numeric_limits` 提供诸多有用信息（§17.7）。例如：

```cpp
constexpr float min = numeric_limits<float>::min();   // 最小正 float
```

对象大小可通过内建运算符 `sizeof`（§1.4）取得。例如：

```cpp
constexpr int szi = sizeof(int);   // 一个 int 占用的字节数
```

在 `<type_traits>` 中，标准库提供了大量用于查询类型性质的函数。例如：

```cpp
bool b = is_arithmetic_v<X>;                        // 若 X 为（内建）算术类型则为 true
using Res = invoke_result_t<decltype(f)>;           // 若 f 是返回 int 的函数，则 Res 为 int
```

`decltype(f)` 是对内建类型函数 `decltype()` 的调用，返回其参数的声明类型；此处为 `f`。

有些类型函数会根据输入构造新类型。例如：

```cpp
template<typename T>
using Store = conditional_t<sizeof(T) < max, On_stack<T>, On_heap<T>>;
```

若 `conditional_t` 的第一个（布尔）实参为 `true`，结果就是第一个备选类型；否则为第二个。假定 `On_stack` 与 `On_heap` 对 `T` 提供相同的访问接口，它们可按名称所示分配其 `T`。于是，`Store<X>` 的用户能依据 `X` 对象的大小调节策略；由此带来的分配策略性能调优可能非常重要。这是我们可以用标准类型函数或借助 concept 构造自有类型函数的简单示例。

concept 也是类型函数。用在表达式里时，它们具体化为类型谓词。例如：

```cpp
template<typename F, typename... Args>
auto call(F f, Args... a, Allocator alloc)
{
    if constexpr (invocable<F, Allocator, Args...>) // 需要分配器吗？
        return f(f, alloc, a...);
    else
        return f(f, a...);
}
```

在许多情形下，concept 是最好的类型函数；但标准库的大多数代码写在 concept 之前，还必须支持尚未使用 concept 的代码库。

记法约定容易令人困惑。标准库用后缀 `_v` 表示返回值的类型函数，用 `_t` 表示返回类型的类型函数。这是 C 以及 concept 之前 C++ 弱类型时代的遗存。没有任何标准库类型函数既返回类型又返回值，因此这些后缀其实是冗余的。有了 concept——无论是在标准库还是别处——都不需要也不使用后缀。

类型函数属于 C++ 编译期计算机制的一部分；相较没有它们的情形，能实现更严格的类型检查和更好的性能。类型函数与 concept（第 8 章，§14.5）的使用常被称为元编程，或（当涉及模板时）模板元编程。

## 16.4.1

在 `<type_traits>` 中，标准库提供数十个简单的类型函数，称为**类型谓词**，回答关于类型的基本问题。下面是一小部分：

**所选类型谓词**（`T`、`A`、`U` 为类型；谓词均返回 `bool`）

| 谓词 | 含义 |
|------|------|
| `is_void_v<T>` | `T` 是否为 `void`？ |
| `is_integral_v<T>` | `T` 是否为整型？ |
| `is_floating_point_v<T>` | `T` 是否为浮点类型？ |
| `is_class_v<T>` | `T` 是否为类（且非联合体）？ |
| `is_function_v<T>` | `T` 是否为函数（而非函数对象或函数指针）？ |
| `is_arithmetic_v<T>` | `T` 是否为整型或浮点类型？ |
| `is_scalar_v<T>` | `T` 是否为算术、枚举、指针或成员指针类型？ |
| `is_constructible_v<T, A...>` | 能否用实参列表 `A...` 构造 `T`？ |
| `is_default_constructible_v<T>` | 能否无显式实参构造 `T`？ |
| `is_copy_constructible_v<T>` | 能否用另一个 `T` 构造 `T`？ |
| `is_move_constructible_v<T>` | 能否将 `T` 移动或拷贝到另一个 `T`？ |
| `is_assignable_v<T, U>` | 能否把 `U` 赋给 `T`？ |
| `is_trivially_copyable_v<T>` | `T` 是否可无用户自定义拷贝操作地平凡可复制？ |
| `is_same_v<T, U>` | `T` 与 `U` 是否同一类型？ |
| `is_base_of_v<T, U>` | `U` 是否派生自 `T`，或 `U` 与 `T` 相同？ |
| `is_convertible_v<T, U>` | `T` 能否隐式转换为 `U`？ |
| `is_iterator_v<T>` | `T` 是否为迭代器类型？ |
| `is_invocable_v<T, A...>` | 能否用实参列表 `A...` 调用 `T`？ |
| `has_virtual_destructor_v<T>` | `T` 是否有虚析构函数？ |

这些谓词的一种传统用途是约束模板实参。例如：

```cpp
template<typename Scalar>
class complex {
    Scalar re, im;
public:
    static_assert(is_arithmetic_v<Scalar>, "Sorry, I support only complex of arithmetic types.");
    // ...
};
```

然而这种做法——与其他许多传统用法一样——用 concept 来做会更轻松、更利落：

```cpp
template<Arithmetic Scalar>
class complex {
    Scalar re, im;
public:
    // ...
};
```

许多场合下，诸如 `is_arithmetic` 的类型谓词会消融在 concept 的定义里，更易使用。例如：

```cpp
template<typename T>
concept Arithmetic = is_arithmetic_v<T>;
```

有趣的是，标准库里并没有名为 `std::arithmetic` 的 concept。

我们还可以定义比标准库类型谓词更一般的 concept。许多标准谓词只适用于内建类型。我们能按所需运算来定义 concept，正如 `Number`（§8.2.4）的定义所示：

```cpp
template<typename T, typename U = T>
concept Arithmetic = Number<T, U> && Number<U, T>;
```

标准库类型谓词的用法最常见于基础服务的实现深处，往往是为了区分情形以做优化。例如，`std::copy(Iter, Iter, Iter2)` 的实现可能对「简单类型的连续序列」（例如整数）这一重要情形专门优化：

```cpp
template<class T>
void cpy1(T* first, T* last, T* target)
{
    if constexpr (is_trivially_copyable_v<T>)
        memcpy(target, first, (last - first) * sizeof(T));   // 注意：示意 memcpy 参数顺序
    else
        while (first != last)
            *target++ = *first++;
}
```

这类简单优化在某些实现上可比未优化版本快大约一半。**除非**你已核实标准库没有做得更好，否则不要沉迷于这类技巧。手工优化的代码通常不如更简单替代品易于维护。

## 16.4.2

考虑定义「智能指针」：

```cpp
template<typename T>
class Smart_pointer {
    // ...
    T& operator*() const;
    T* operator->() const;     // 当且仅当 T 为类类型时才应有 ->
};
```

`operator->` 当且仅当 `T` 为类类型时才应定义。例如，`Smart_pointer<vector<int>>` 应有 `->`，但 `Smart_pointer<int>` 不应有。

我们不能使用编译期 `if`，因为不在函数体内。应写成：

```cpp
template<typename T>
class Smart_pointer {
    // ...
    T& operator*() const;
    T* operator->() const requires is_class_v<T>;   // 当且仅当……时定义 ->
};
```

类型谓词直接表达了对 `operator->()` 的约束。我们也可以用 concept。标准库没有「必须是类类型」的 concept（即类、`struct` 或联合体），但可以自定义：

```cpp
template<typename T>
concept Class = is_class_v<T> || is_union_v<T>;   // 联合体也算一类 class-key

template<typename T>
class Smart_pointer {
    // ...
    T& operator*() const;
    T* operator->() const requires Class<T>;      // 当且仅当 T 为类类型……
};
```

常常 concept 比直接使用标准库类型谓词更一般，或单纯更合适。

## 16.4.3

许多类型函数返回类型——常常是据此计算出的新类型。我把这类函数称为**类型生成器**，以区别于类型谓词。标准库提供的一部分如下：

**所选类型生成器**

| 记号 | 含义 |
|------|------|
| `R = remove_const_t<T>` | `R` 为去掉最外层 `const`（若有）后的 `T` |
| `R = add_const_t<T>` | `R` 为 `const T` |
| `R = remove_reference_t<T>` | 若 `T` 为引用 `U&`，则 `R` 为 `U`，否则为 `T` |
| `R = add_lvalue_reference_t<T>` | 若 `T` 已是左值引用则 `R` 为 `T`，否则为 `T&` |
| `R = add_rvalue_reference_t<T>` | 若 `T` 已是右值引用则 `R` 为 `T`，否则为 `T&&` |
| `R = enable_if_t<b, T = void>` | 若 `b` 为 `true` 则 `R` 为 `T`，否则 `R` 未定义 |
| `R = conditional_t<b, T, U>` | 若 `b` 为 `true` 则 `R` 为 `T`，否则为 `U` |
| `R = common_type_t<T...>` | 若诸 `T` 可隐式转换到同一类型，则 `R` 为该类型；否则未定义 |
| `R = underlying_type_t<T>` | 若 `T` 为枚举，则 `R` 为其底层类型；否则错误 |
| `R = invoke_result_t<T, A...>` | 若可用 `A...` 调用 `T`，则 `R` 为其返回类型；否则错误 |

这些类型函数通常用在工具设施实现里，而不是直接出现在应用代码中。其中 `enable_if` 大概是 concept 之前代码里最常用的。例如，智能指针上有条件启用的 `->` 传统上会像这样实现：

```cpp
template<typename T>
class Smart_pointer {
    // ...
    T& operator*();
    enable_if_t<is_class_v<T>, T&> operator->();   // 当且仅当 T 为类类型……
};
```

我觉得这并不好读，更复杂的用法糟得多。`enable_if` 的定义依赖一门精微的语言特性，称为 SFINAE（替换失败不是错误）。只有在你**确实需要**时再去查阅它。

## 16.4.4

所有标准容器（§12.8）以及遵循其模式的容器都有若干关联类型，例如值类型与迭代器类型。在 `<iterator>` 与 `<ranges>` 中，标准库为这些类型提供了名字：

| 名字 | 含义 |
|------|------|
| `range_value_t<R>` | 范围 `R` 的元素类型 |
| `iter_value_t<T>` | 迭代器 `T` 所指元素的类型 |
| `iterator_t<R>` | 范围 `R` 的迭代器类型 |

# 16.5 `source_location`

写出跟踪信息或错误信息时，常常希望把源码位置一并写入消息。库为此提供了 `source_location`：

```cpp
const source_location loc = source_location::current();
```

`current()` 返回描述其在源码中出现位置的 `source_location`。类 `source_location` 提供成员 `file()` 与 `function_name()`，返回 C 风格字符串；`line()` 与 `column()` 返回无符号整数。

把它包进函数里，就得到日志消息的初步可用版本：

```cpp
void log(const string& mess = "", const source_location loc = source_location::current())
{
    cout << loc.file_name()
         << '(' << loc.line() << ':' << loc.column() << ") "
         << loc.function_name() << ": "
         << mess;
}

void foo()
{
    log("Hello");               // myfile.cpp(17,4) foo: Hello
    // ...
}

int bar(const string& label)
{
    log(label);                 // myfile.cpp(23,4) bar: <label 的值>
    // ...
}
```

对 `current()` 的调用位于默认实参中，于是我们得到的是 **`log()` 调用方** 的位置，而不是 `log()` 自身的位置。

在 C++20 之前编写、或需在旧编译器上编译的代码，对此通常使用宏 `__FILE__` 与 `__LINE__`。

# 16.6 `move` 与 `forward`

在移动与拷贝之间的选择大多是隐式的（§3.4）。编译器在对象即将销毁时（例如在 `return` 中）往往更倾向于移动，因为那假定是更简单也更高效的操作。然而有时我们必须显式指明。例如，`unique_ptr` 是对象的唯一拥有者，因而不能拷贝；若想把 `unique_ptr` 交给别处，就必须移动它。例如：

```cpp
void f1()
{
    auto p = make_unique<int>(2);
    auto q = p;            // 错误：不能拷贝 unique_ptr
    auto q = move(p);      // p 现为 nullptr
    // ...
}
```

容易混淆的是：`std::move()` **并不移动**任何东西。它只是把实参转型为右值引用，表示「该实参不会再被使用」，因而**可以**被移动（§6.2.2）。它本该叫 `rvalue_cast` 之类。它服务于少数关键情形。

考虑一个简单的 `swap`：

```cpp
template<typename T>
void swap(T& a, T& b)
{
    T tmp{move(a)};        // T 的构造函数看到右值，执行移动
    a = move(b);           // T 的赋值看到右值，执行移动
    b = move(tmp);         // T 的赋值看到右值，执行移动
}
```

我们不想一再拷贝潜在的大对象，于是用 `std::move()` 请求移动。

与其它转型一样，`std::move()` 也有诱人却危险的用法。考虑：

```cpp
string s1 = "Hello";
string s2 = "World";
vector<string> v;
v.push_back(s1);              // 以 const string& 传参；push_back 会拷贝
v.push_back(move(s2));        // 使用移动构造函数
v.emplace_back(s1);           // 另一种做法：原位构造一份 s1 拷贝
```

这里 `s1` 被 `push_back()` **拷贝**，而 `s2` 被**移动**。这有时（也只是有时）会让 `s2` 的 `push_back()` 更便宜。问题在于：移动之后会留下一个「被移空」的对象。若再次使用 `s2`，就可能出问题：

```cpp
cout << s1[2];    // 输出 'l'
cout << s2[2];    // 崩溃？
```

我认为这种 `std::move()` 用法太容易出错，不宜广泛使用。**除非**你能证明存在显著且必要的性能收益，否则不要用。后续维护可能无意中再次使用已移动的对象。

编译器知道返回值在函数中不会被再用，因此显式写 `return std::move(x)` 多余，甚至可能妨碍优化。

被移走对象的状态一般不规定，但所有标准库类型都会把它留在「仍可销毁、仍可赋值」的状态。明智的做法是效仿这一约定。对容器（例如 `vector` 或 `string`），被移走后通常处于「空」状态。对许多类型，默认值就是一种合适又廉价的「空」状态。

转发实参是需要移动的又一重要场景（§8.4.2）。有时我们希望把一整组实参原封不动传给另一个函数（以实现「完美转发」）。标准库的 `forward()` 与更简单的 `std::move()` 不同，它能正确处理左值与右值的细微差别（§6.2.2）。请**只**用 `std::forward()` 做转发；并且不要对同一对象 `forward()` 两次——一旦转发出去，对象就不再归你使用。

# 16.7 按位运算

在 `<bit>` 中可以找到用于底层按位操作的函数。按位操作很专门化，却又常常必不可少。当我们贴近硬件时，往往必须观察比特、改变字节或字中的位模式，并把原始内存解释为带类型的对象。例如，`bit_cast` 允许我们把一种类型的值转换为另一种**大小相同**类型的值：

```cpp
double val = 7.2;
auto x = bit_cast<uint64_t>(val);       // 取得 64 位浮点数的位模式
auto y = bit_cast<uint64_t>(&val);      // 取得 64 位指针的位模式

struct Word { std::byte b[8]; };
std::byte buffer[1024];
// ...
auto p = bit_cast<Word*>(&buffer[i]);    // p 指向 8 个字节
auto i = bit_cast<int64_t>(*p);          // 把这 8 个字节解释为整数
```

标准库类型 `std::byte`（必须写 `std::` 前缀）用来表示**字节**，而不是已知表示字符或整数的字节。特别是，`std::byte` 只提供按位逻辑运算，不提供算术运算。通常，执行位操作的最佳类型是无符号整数或 `std::byte`。所谓最佳，指的是最快、也最不易出人意料。

另见 `bitset`（§15.3.2）。

对整数做常用位操作时，还会用到诸如 `bit_width`、`rotl`、`popcount` 等函数。例如：

```cpp
void use(unsigned int ui)
{
    int x0 = bit_width(ui);           // 表示 ui 所需的最少位数
    unsigned int ui2 = rotl(ui, 8);    // 循环左移 8 位（注意：不改变 ui 本身）
    int x1 = popcount(ui);            // ui 中 1 的个数
    // ...
}
```

# 16.8 程序终止

有时代码遇到的问题是自己无法处理的：

- 若问题**很常见**，且**直接调用方**理应能处理，返回某种返回码（§4.4）。
- 若问题**罕见**，或调用方**不能**指望处理它，抛出异常（§4.4）。
- 若问题**严重**到程序的任何寻常部分都不该指望处理它，就**终止程序**。

标准库为最后一种情况（「退出程序」）提供了设施：

- **`exit(x)`**：调用通过 `atexit()` 注册的函数，然后以返回值 `x` 退出程序。如需细节可查 `atexit()`——它基本上是与 C 语言共享的一种初级析构机制。
- **`abort()`**：立即且无附带条件地退出程序，返回值表示未成功终止。某些操作系统会提供修改这一行为的设施。
- **`quick_exit(x)`**：调用通过 `at_quick_exit()` 注册的函数，然后以返回值 `x` 退出程序。
- **`terminate()`**：调用 `terminate_handler`。默认的 `terminate_handler` 是 `abort()`。

这些函数面向**真正严重**的错误。**它们不会调用析构函数**；也就是说，不会做寻常而体面的清理。各种 handler 用于在退出前采取行动；此类行动必须非常简单，因为调用这些退出函数的原因之一可能就是程序状态已经损坏。一种合理且相当流行的做法是「在不依赖当前程序任何状态的前提下，把系统重启到一个定义良好的状态」。另一种略冒险但常常还不算不合理的做法是「记录错误日志并退出」。日志可能成为问题的原因在于：触发终止的根源可能已经破坏了 I/O 子系统。

错误处理是最棘手的编程门类之一；即便干净地退出程序也可能很难。

**通用目的库不应无条件终止程序。**

# 16.9 建议

[1] 库未必又大又复杂才有用；§16.1。

[2] 在对效率宣称看法之前，先给你的程序计时；§16.2.1。

[3] 使用时间度量时，用 `duration_cast` 以合适单位报告结果；§16.2.1。

[4] 要在源码里直接表示日期，使用符号记法（例如 `November/28/2021`）；§16.2.2。

[5] 若日期来自计算结果，用 `ok()` 检查合法性；§16.2.2。

[6] 在不同时区地点处理时间时，使用 `zoned_time`；§16.2.3。

[7] 用 lambda 表达调用约定上的细小差别；§16.3.1。

[8] 用 `mem_fn()` 或 lambda 创建能以传统函数调用记法调用成员函数的函数对象；§16.3.1，§16.3.2。

[9] 当你需要保存「可被调用」的东西时，使用 `function`；§16.3.3。

[10] 相较于显式使用类型谓词，更偏向 concept；§16.4.1。

[11] 你可以编写显式依赖类型性质的代码；§16.4.1，§16.4.2。

[12] 只要办得到，就更偏向 concept，而不是 trait 与 `enable_if`；§16.4.3。

[13] 用 `source_location` 把源码位置嵌入调试与日志消息；§16.5。

[14] 避免显式使用 `std::move()`；§16.6；[CG: ES.56]。

[15] **只**用 `std::forward()` 做转发；§16.6。

[16] 在对对象执行 `std::move()` 或 `std::forward()` 之后，永远不要再读取该对象；§16.6。

[17] 用 `std::byte` 表示尚不具备有意义类型的原始数据；§16.7。

[18] 做按位操作时，使用无符号整数或 `bitset`；§16.7。

[19] 若直接调用方理应能处理问题，从函数返回错误码；§16.8。

[20] 若直接调用方不能指望处理问题，从函数抛出异常；§16.8。

[21] 若试图从问题中恢复并不合理，调用 `exit()`、`quick_exit()` 或 `terminate()` 终止程序；§16.8。

[22] 通用目的库不应无条件终止；§16.8。
