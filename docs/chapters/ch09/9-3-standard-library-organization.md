# 9.3 标准库的组织

标准库的设施被放置在命名空间 `std` 中，并通过模块或头文件提供给用户。

#### 9.3.1 命名空间

每个标准库设施都通过某个标准头文件提供。例如：

```cpp
#include<string>
#include<list>
```

这使得标准 `string` 和 `list` 可用。

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
