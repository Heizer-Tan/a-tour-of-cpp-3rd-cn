# 标准库概览

> **为何要花时间学习？无知可是立竿见影的。**
>
> —— Hobbes

# 9.1 引言

没有哪个严肃项目只依赖裸露的语言本身而活。通常先有一批基础库奠基，再在它们之上堆砌业务逻辑——若拒绝复用成熟的抽象，几乎每个任务都会显得格外琐碎；而一旦配上设计良好的库，同样的工作往往能事半功倍。

接续第 1–8 章，随后的第 9–18 章会对标准库里最常见的设施做一次“高速观光”。行文只会点到为止：`string`、`ostream`、`variant`、`vector`、`map`、`path`、`unique_ptr`、`thread`、`regex`、`system_clock`、`time_zone`、`complex` 等都会露面，但不会在这里展开百科全书式的细节。

与前八章一致，在阅读过程中大可不必因偶尔的陌生感气馁；本章的目标是建立一张“mental map”，让读者知道标准库能解决哪类刚需。

ISO C++ 标准的大部分篇幅其实都是标准库条文。除非你确有理由，否则应优先采纳久经磨炼的既有实现——设计讨论、质量控制与长期维护的投入，都很难在私人项目里复制。

本书涉及的标准库组件属于任何完整实现的必备部分。除此之外，平台往往还提供 GUI、网络、数据库等扩展；商业环境也会叠床架屋地加上“公司标准库”。本文故意只讨论标准本身所定义的那部分，以保证示例可移植。至于更浩瀚的生态，留待读者在真实部署中自行探索。

# 9.2 标准库组件

标准库提供的设施可以分类如下：

- 运行时语言支持（例如，用于分配、异常和运行时类型信息）。

- C 标准库（经过非常小的修改以最小化类型系统违规）。
- 字符串，支持国际字符集、本地化和子字符串的只读视图（[§10.2](../ch10/10-2-strings.md)）。
- 正则表达式匹配支持（[§10.4](../ch10/10-4-regular-expressions.md)）。
- I/O 流是一个可扩展的输入输出框架，用户可以向其中添加自己的类型、流、缓冲策略、本地环境和字符集（第 11 章）。它还提供灵活的输出格式化设施（[§11.6.2](../ch11/11-6-output-formatting.md)）。
- 以可移植方式操作文件系统的库（[§11.9](../ch11/11-9-file-system.md)）。
- 容器（如 `vector` 和 `map`；第 12 章）和算法（如 `find()`、`sort()` 和 `merge()`；第 13 章）的框架。这个框架通常称为 STL [Stepanov,1994]，是可扩展的，因此用户可以添加自己的容器和算法。
- 范围（[§14.1](../ch14/14-1-introduction.md)），包括视图（[§14.2](../ch14/14-2-views.md)）、生成器（[§14.3](../ch14/14-3-generators.md)）和管道（[§14.4](../ch14/14-4-pipes.md)）。
- 用于基本类型和范围的概念（[§14.5](../ch14/14-5-concept-overview.md)）。
- 数值计算支持，例如标准数学函数、复数、带有算术运算的向量、数学常数和随机数生成器（[§5.2.1](../ch05/5-2-concrete-types.md) 和第 16 章）。
- 并发编程支持，包括线程和锁（第 18 章）。并发支持是基础性的，以便用户可以作为库添加对新并发模型的支持。
- 同步和异步协程（[§18.6](../ch18/18-6-coroutines.md)）。
- 大多数 STL 算法和一些数值算法的并行版本，例如 `sort()`（[§13.6](../ch13/13-6-parallel-algorithms.md)）和 `reduce()`（[§17.3.1](../ch17/17-3-numeric-algorithms.md)）。
- 支持元编程（例如类型函数；[§16.4](../ch16/16-4-type-functions.md)）、STL 风格泛型编程（例如 `pair`；[§15.3.3](../ch15/15-3-containers.md)）和通用编程（例如 `variant` 和 `optional`；[§15.4.1](../ch15/15-4-alternatives.md)，[§15.4.2](../ch15/15-4-alternatives.md)）的工具。
- 用于资源管理的“智能指针”（例如 `unique_ptr` 和 `shared_ptr`；[§15.2.1](../ch15/15-2-pointers.md)）。
- 特殊用途容器，例如 `array`（[§15.3.1](../ch15/15-3-containers.md)）、`bitset`（[§15.3.2](../ch15/15-3-containers.md)）和 `tuple`（[§15.3.4](../ch15/15-3-containers.md)）。
- 绝对时间与时长，例如 `time_point` 与 `system_clock`（[§16.2.1](../ch16/16-2-time.md)）。
- 日历与时间区域，例如 `month` 与 `time_zone`（[§16.2.2](../ch16/16-2-time.md)，[§16.2.3](../ch16/16-2-time.md)）。
- 常用单位的后缀，例如 `ms` 表示毫秒，`i` 表示虚数单位（[§6.6](../ch06/6-6-user-defined-literals.md)）。
- 操作元素序列的方式，例如视图（[§14.2](../ch14/14-2-views.md)）、`string_view`（[§10.3](../ch10/10-3-string-view.md)）和 `span`（[§15.2.2](../ch15/15-2-pointers.md)）。

将某个类包含到库中的主要标准是：

- 它对几乎每个 C++ 程序员（无论是新手还是专家）都有帮助。
- 它可以以一种通用形式提供，与相同设施的简单版本相比，不会增加显著的开销。
- 简单使用应该易于学习（相对于其任务的内在复杂性）。

本质上，C++ 标准库提供了最常见的、基本的数据结构以及用于它们的基本算法。

# 9.3 标准库的组织

标准库的各个组件都隶属于命名空间 `std`，并通过模块或带头的翻译单元提供给用户程序。

## 9.3.1 命名空间与头文件

每个标准库设施都通过某个标准头文件提供。例如：

```cpp
#include<string>
#include<list>
```

这使得标准 `string` 和 `list` 可用。

标准库定义在名为 `std` 的命名空间（[§3.3](../ch03/3-3-namespace.md)）中。要使用标准库设施，可以使用 `std::` 前缀：

```cpp
std::string sheep {"Four legs Good; two legs Baaad!"};
std::list<std::string> slogans {"War is Peace", "Freedom is Slavery", "Ignorance is Strength"};
```

为简洁起见，在示例中我很少使用 `std::` 前缀。我也没有显式 `#include` 或 `import` 必要的头文件或模块。要编译和运行这里的程序片段，你必须使标准库的相关部分可用。例如：

```cpp
#include<string>                 // 使标准字符串设施可访问
using namespace std;            // 使 std 名称无需 std:: 前缀即可使用

string s {"C++ is a general-purpose programming language"};   // OK：string 是 std::string
```

将命名空间中的每个名称都转储到全局命名空间中通常是不好的风格。然而，在本书中，我专门使用标准库，知道它提供了什么是很好的。

标准库提供了 `std` 的几个子命名空间，只能通过显式操作访问：

- `std::chrono`：来自 chrono 的所有设施，包括 `std::literals::chrono_literals`（[§16.2](../ch16/16-2-time.md)）。
- `std::literals::chrono_literals`：后缀 `y`（年）、`d`（天）、`h`（小时）、`min`（分钟）、`ms`（毫秒）、`ns`（纳秒）、`s`（秒）、`us`（微秒）（[§16.2](../ch16/16-2-time.md)）。
- `std::literals::complex_literals`：后缀 `i`（虚数 double）、`if`（虚数 float）、`il`（虚数 long double）（[§6.6](../ch06/6-6-user-defined-literals.md)）。
- `std::literals::string_literals`：后缀 `s`（字符串）（[§6.6](../ch06/6-6-user-defined-literals.md)，[§10.2](../ch10/10-2-strings.md)）。
- `std::literals::string_view_literals`：后缀 `sv`（字符串视图）（[§10.3](../ch10/10-3-string-view.md)）。
- `std::numbers`：数学常数（[§17.9](../ch17/17-9-math-constants.md)）。
- `std::pmr`：多态内存资源（[§12.7](../ch12/12-7-allocators.md)）。

要使用子命名空间中的后缀，我们必须将其引入到想要使用它的命名空间中。例如：

```cpp
// 没有提到复数文字
auto z1 = 2 + 3i;   // 错误：没有后缀 i'

using namespace std::literals::complex_literals;   // 使复数文字可见
auto z2 = 2 + 3i;   // OK：z2 是一个 complex<double>
```

对于子命名空间中应该包含什么，没有统一的理念。然而，后缀不能显式限定，因此我们只能将一组后缀引入一个作用域，而不会引起歧义。因此，旨在与其他库（可能定义自己的后缀）一起使用的库的后缀被放置在子命名空间中。

## 9.3.2 ranges 命名空间

标准库以两种版本提供算法，例如 `sort()` 和 `copy()`：

- 传统的序列版本，接受一对迭代器；例如 `sort(begin(v), v.end())`
- 范围版本，接受单个范围；例如 `sort(v)`

理想情况下，这两个版本应该完美重载，无需任何特殊努力。然而，它们并非如此。例如：

```cpp
using namespace std;
using namespace ranges;

void f(vector<int>& v)
{
    sort(v.begin(), v.end());   // 错误：歧义
    sort(v);                    // 错误：歧义
}
```

为了防止使用传统无约束模板时出现歧义，标准要求我们显式地将标准库算法的范围版本引入作用域：

```cpp
using namespace std;
using namespace ranges;

void g(vector<int>& v)
{
    sort(v.begin(), v.end());   // OK
    sort(v);                    // 错误：std 中没有匹配的函数
    ranges::sort(v);            // OK
    using ranges::sort;         // 从此处开始 sort(v) 是 OK 的
    sort(v);                    // OK
}
```

## 9.3.3 模块

目前还没有任何标准库模块。C++23 很可能会弥补这一遗漏（由于委员会时间不足所致）。目前，我使用模块 `std`，它很可能成为标准，提供来自命名空间 `std` 的所有设施。参见附录 A。

## 9.3.4 头文件

以下是标准库头文件的选录，它们都在命名空间 `std` 中提供声明：

**选定的标准库头文件**

| 头文件 | 提供内容 | 参考章节 |
|--------|----------|----------|
| `<algorithm>` | `copy()`, `find()`, `sort()` | 第 13 章 |
| `<array>` | `array` | [§15.3.1](../ch15/15-3-containers.md) |
| `<chrono>` | `duration`, `time_point`, `month`, `time_zone` | [§16.2](../ch16/16-2-time.md) |
| `<cmath>` | `sqrt()`, `pow()` | [§17.2](../ch17/17-2-math-functions.md) |
| `<complex>` | `complex`, `sqrt()`, `pow()` | [§17.4](../ch17/17-4-complex.md) |
| `<concepts>` | `floating_point`, `copyable`, `predicate`, `invocable` | [§14.5](../ch14/14-5-concept-overview.md) |
| `<filesystem>` | `path` | [§11.9](../ch11/11-9-file-system.md) |
| `<format>` | `format()` | [§11.6.2](../ch11/11-6-output-formatting.md) |
| `<fstream>` | `fstream`, `ifstream`, `ofstream` | [§11.7.2](../ch11/11-7-streams.md) |
| `<functional>` | `function`, `greater_equal`, `hash`, `range_value_t` | 第 16 章 |
| `<future>` | `future`, `promise` | [§18.5](../ch18/18-5-inter-task-communication.md) |
| `<ios>` | `hex`, `dec`, `scientific`, `fixed`, `defaultfloat` | [§11.6.2](../ch11/11-6-output-formatting.md) |
| `<iostream>` | `istream`, `ostream`, `cin`, `cout` | 第 11 章 |
| `<map>` | `map`, `multimap` | [§12.6](../ch12/12-6-unordered-map.md) |
| `<memory>` | `unique_ptr`, `shared_ptr`, `allocator` | [§15.2.1](../ch15/15-2-pointers.md) |
| `<random>` | `default_random_engine`, `normal_distribution` | [§17.5](../ch17/17-5-random-numbers.md) |
| `<ranges>` | `sized_range`, `subrange`, `take()`, `split()`, `iterator_t` | [§14.1](../ch14/14-1-introduction.md) |
| `<regex>` | `regex`, `smatch` | [§10.4](../ch10/10-4-regular-expressions.md) |
| `<string>` | `string`, `basic_string` | [§10.2](../ch10/10-2-strings.md) |
| `<string_view>` | `string_view` | [§10.3](../ch10/10-3-string-view.md) |
| `<set>` | `set`, `multiset` | [§12.8](../ch12/12-8-container-overview.md) |
| `<sstream>` | `istringstream`, `ostringstream` | [§11.7.3](../ch11/11-7-streams.md) |
| `<stdexcept>` | `length_error`, `out_of_range`, `runtime_error` | [§4.2](../ch04/4-2-exceptions.md) |
| `<tuple>` | `tuple`, `get<>()`, `tuple_size<>` | [§15.3.4](../ch15/15-3-containers.md) |
| `<thread>` | `thread` | [§18.2](../ch18/18-2-tasks-and-threads.md) |
| `<unordered_map>` | `unordered_map`, `unordered_multimap` | [§12.6](../ch12/12-6-unordered-map.md) |
| `<utility>` | `move()`, `swap()`, `pair` | 第 16 章 |
| `<variant>` | `variant` | [§15.4.1](../ch15/15-4-alternatives.md) |
| `<vector>` | `vector` | [§12.2](../ch12/12-2-vector.md) |

这个列表远非完整。

C 标准库的头文件（例如 `<stdlib.h>`）也被提供。对于每个这样的头文件，还有一个版本，其名称以 `c` 为前缀并去掉 `.h`。这个版本（例如 `<cstdlib>`）将其声明同时放在 `std` 和全局命名空间中。

这些头文件反映了标准库开发的历史。因此，它们并不总是像我们希望的那样合乎逻辑且易于记忆。这就是为什么使用模块（例如 `std`，[§9.3.3](9-3-standard-library-organization.md)）是一个更好的选择。

# 9.4 建议

[1] 不要重复造轮子；使用库；[§9.1](9-1-introduction.md)；[CG: SL.1]。
[2] 当有选择时，优先选择标准库而非其他库；[§9.1](9-1-introduction.md)；[CG: SL.2]。
[3] 不要认为标准库对一切都理想；[§9.1](9-1-introduction.md)。
[4] 如果不使用模块，记得 `#include` 适当的头文件；[§9.3.1](9-3-standard-library-organization.md)。
[5] 记住标准库设施定义在命名空间 `std` 中；[§9.3.1](9-3-standard-library-organization.md)；[CG: SL.3]。
[6] 使用范围时，记得显式限定算法名称；[§9.3.2](9-3-standard-library-organization.md)。
[7] 优先导入模块，而非 `#include` 头文件（[§9.3.3](9-3-standard-library-organization.md)）。
