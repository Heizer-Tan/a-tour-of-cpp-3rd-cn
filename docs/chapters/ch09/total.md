===== 第 1 页 =====

9

# 库概览

既然无知是瞬间的，何必浪费时间学习？ – Hobbes

- 引言
- 标准库组件
- 标准库的组织
  - 命名空间；ranges 命名空间；模块；头文件
- 建议

### 9.1 引言

===== 第 2 页 =====

没有任何重要的程序是仅仅用裸编程语言写成的。首先，要开发一组库。这些库随后构成进一步工作的基础。大多数程序用裸语言编写都很繁琐，而几乎任何任务都可以通过使用好的库变得简单。

接续第 1–8 章，第 9–18 章将快速介绍关键的标准库设施。我将非常简要地介绍有用的标准库类型，例如 `string`、`ostream`、`variant`、`vector`、`map`、`path`、`unique_ptr`、`thread`、`regex`、`system_clock`、`time_zone` 和 `complex`，以及它们最常见的使用方式。

与第 1–8 章一样，强烈建议您不要因细节理解不完整而分心或气馁。本章的目的是让您对最有用的库设施有一个基本的了解。

标准库的规范占 ISO C++ 标准的三分之二以上。请探索它，并优先选择它而不是自己造轮子。其设计投入了大量思考，其实现投入了更多，其维护和扩展也将投入大量努力。

本书中描述的标准库设施是每个完整 C++ 实现的一部分。除了标准库组件，大多数实现还提供“图形用户界面”系统（GUI）、Web 接口、数据库接口等。类似地，大多数应用程序开发环境为“标准”的公司或工业开发/执行环境提供“基础库”。除此之外，还有成千上万的库支持专门的应用领域。这里，我不描述标准库之外的库、系统或环境。目的是提供 C++ 标准所定义的 C++ 的自包含描述 [C++,2020]，并保持示例的可移植性。当然，鼓励程序员探索大多数系统上可用的更广泛的设施。

### 9.2 标准库组件

标准库提供的设施可以分类如下：

- 运行时语言支持（例如，用于分配、异常和运行时类型信息）。

===== 第 3 页 =====

- C 标准库（经过非常小的修改以最小化类型系统违规）。
- 字符串，支持国际字符集、本地化和子字符串的只读视图（§10.2）。
- 正则表达式匹配支持（§10.4）。
- I/O 流是一个可扩展的输入输出框架，用户可以向其中添加自己的类型、流、缓冲策略、本地环境和字符集（第 11 章）。它还提供灵活的输出格式化设施（§11.6.2）。
- 以可移植方式操作文件系统的库（§11.9）。
- 容器（如 `vector` 和 `map`；第 12 章）和算法（如 `find()`、`sort()` 和 `merge()`；第 13 章）的框架。这个框架通常称为 STL [Stepanov,1994]，是可扩展的，因此用户可以添加自己的容器和算法。
- 范围（§14.1），包括视图（§14.2）、生成器（§14.3）和管道（§14.4）。
- 用于基本类型和范围的概念（§14.5）。
- 数值计算支持，例如标准数学函数、复数、带有算术运算的向量、数学常数和随机数生成器（§5.2.1 和第 16 章）。
- 并发编程支持，包括线程和锁（第 18 章）。并发支持是基础性的，以便用户可以作为库添加对新并发模型的支持。
- 同步和异步协程（§18.6）。
- 大多数 STL 算法和一些数值算法的并行版本，例如 `sort()`（§13.6）和 `reduce()`（§17.3.1）。
- 支持元编程（例如类型函数；§16.4）、STL 风格泛型编程（例如 `pair`；§15.3.3）和通用编程（例如 `variant` 和 `optional`；§15.4.1，§15.4.2）的工具。
- 用于资源管理的“智能指针”（例如 `unique_ptr` 和 `shared_ptr`；§15.2.1）。

===== 第 4 页 =====

- 特殊用途容器，例如 `array`（§15.3.1）、`bitset`（§15.3.2）和 `tuple`（§15.3.3）。
- 绝对时间和持续时间支持，例如 `time_point` 和 `system_clock`（§16.2.1）。
- 日历支持，例如 `month` 和 `time_zone`（§16.2.2，§16.2.3）。
- 常用单位的后缀，例如 `ms` 表示毫秒，`i` 表示虚数单位（§6.6）。
- 操作元素序列的方式，例如视图（§14.2）、`string_view`（§10.3）和 `span`（§15.2.2）。

将某个类包含到库中的主要标准是：

- 它对几乎每个 C++ 程序员（无论是新手还是专家）都有帮助。
- 它可以以一种通用形式提供，与相同设施的简单版本相比，不会增加显著的开销。
- 简单使用应该易于学习（相对于其任务的内在复杂性）。

本质上，C++ 标准库提供了最常见的、基本的数据结构以及用于它们的基本算法。

### 9.3 标准库的组织

标准库的设施被放置在命名空间 `std` 中，并通过模块或头文件提供给用户。

#### 9.3.1 命名空间

每个标准库设施都通过某个标准头文件提供。例如：

```cpp
#include<string>
#include<list>
```

这使得标准 `string` 和 `list` 可用。

===== 第 5 页 =====

标准库定义在名为 `std` 的命名空间（§3.3）中。要使用标准库设施，可以使用 `std::` 前缀：

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

- `std::chrono`：来自 chrono 的所有设施，包括 `std::literals::chrono_literals`（§16.2）。
- `std::literals::chrono_literals`：后缀 `y`（年）、`d`（天）、`h`（小时）、`min`（分钟）、`ms`（毫秒）、`ns`（纳秒）、`s`（秒）、`us`（微秒）（§16.2）。
- `std::literals::complex_literals`：后缀 `i`（虚数 double）、`if`（虚数 float）、`il`（虚数 long double）（§6.6）。
- `std::literals::string_literals`：后缀 `s`（字符串）（§6.6，§10.2）。
- `std::literals::string_view_literals`：后缀 `sv`（字符串视图）（§10.3）。
- `std::numbers`：数学常数（§17.9）。
- `std::pmr`：多态内存资源（§12.7）。

要使用子命名空间中的后缀，我们必须将其引入到想要使用它的命名空间中。例如：

===== 第 6 页 =====

```cpp
// 没有提到复数文字
auto z1 = 2 + 3i;   // 错误：没有后缀 i'

using namespace std::literals::complex_literals;   // 使复数文字可见
auto z2 = 2 + 3i;   // OK：z2 是一个 complex<double>
```

对于子命名空间中应该包含什么，没有统一的理念。然而，后缀不能显式限定，因此我们只能将一组后缀引入一个作用域，而不会引起歧义。因此，旨在与其他库（可能定义自己的后缀）一起使用的库的后缀被放置在子命名空间中。

#### 9.3.2 ranges 命名空间

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

===== 第 7 页 =====

#### 9.3.3 模块

目前还没有任何标准库模块。C++23 很可能会弥补这一遗漏（由于委员会时间不足所致）。目前，我使用模块 `std`，它很可能成为标准，提供来自命名空间 `std` 的所有设施。参见附录 A。

#### 9.3.4 头文件

以下是标准库头文件的选录，它们都在命名空间 `std` 中提供声明：

**选定的标准库头文件**

| 头文件 | 提供内容 | 参考章节 |
|--------|----------|----------|
| `<algorithm>` | `copy()`, `find()`, `sort()` | 第 13 章 |
| `<array>` | `array` | §15.3.1 |
| `<chrono>` | `duration`, `time_point`, `month`, `time_zone` | §16.2 |
| `<cmath>` | `sqrt()`, `pow()` | §17.2 |
| `<complex>` | `complex`, `sqrt()`, `pow()` | §17.4 |
| `<concepts>` | `floating_point`, `copyable`, `predicate`, `invocable` | §14.5 |
| `<filesystem>` | `path` | §11.9 |
| `<format>` | `format()` | §11.6.2 |
| `<fstream>` | `fstream`, `ifstream`, `ofstream` | §11.7.2 |
| `<functional>` | `function`, `greater_equal`, `hash`, `range_value_t` | 第 16 章 |
| `<future>` | `future`, `promise` | §18.5 |
| `<ios>` | `hex`, `dec`, `scientific`, `fixed`, `defaultfloat` | §11.6.2 |
| `<iostream>` | `istream`, `ostream`, `cin`, `cout` | 第 11 章 |
| `<map>` | `map`, `multimap` | §12.6 |
| `<memory>` | `unique_ptr`, `shared_ptr`, `allocator` | §15.2.1 |
| `<random>` | `default_random_engine`, `normal_distribution` | §17.5 |
| `<ranges>` | `sized_range`, `subrange`, `take()`, `split()`, `iterator_t` | §14.1 |
| `<regex>` | `regex`, `smatch` | §10.4 |
| `<string>` | `string`, `basic_string` | §10.2 |
| `<string_view>` | `string_view` | §10.3 |
| `<set>` | `set`, `multiset` | §12.8 |
| `<sstream>` | `istringstream`, `ostringstream` | §11.7.3 |
| `<stdexcept>` | `length_error`, `out_of_range`, `runtime_error` | §4.2 |
| `<tuple>` | `tuple`, `get<>()`, `tuple_size<>` | §15.3.4 |
| `<thread>` | `thread` | §18.2 |
| `<unordered_map>` | `unordered_map`, `unordered_multimap` | §12.6 |
| `<utility>` | `move()`, `swap()`, `pair` | 第 16 章 |
| `<variant>` | `variant` | §15.4.1 |
| `<vector>` | `vector` | §12.2 |

这个列表远非完整。

C 标准库的头文件（例如 `<stdlib.h>`）也被提供。对于每个这样的头文件，还有一个版本，其名称以 `c` 为前缀并去掉 `.h`。这个版本（例如 `<cstdlib>`）将其声明同时放在 `std` 和全局命名空间中。

这些头文件反映了标准库开发的历史。因此，它们并不总是像我们希望的那样合乎逻辑且易于记忆。这就是为什么使用模块（例如 `std`，§9.3.3）是一个更好的选择。

===== 第 9 页 =====

### 9.4 建议

[1] 不要重新发明轮子；使用库；§9.1；[CG: SL.1]。
[2] 当有选择时，优先选择标准库而不是其他库；§9.1；[CG: SL.2]。
[3] 不要认为标准库对一切都是理想的；§9.1。
[4] 如果不使用模块，记得 `#include` 适当的头文件；§9.3.1。
[5] 记住标准库设施定义在命名空间 `std` 中；§9.3.1；[CG: SL.3]。
[6] 使用范围时，记得显式限定算法名称；§9.3.2。
[7] 优先导入模块而不是 `#include` 头文件（§9.3.3）。