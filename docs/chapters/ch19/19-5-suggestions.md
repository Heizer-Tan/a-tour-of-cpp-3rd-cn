
[1] ISO C++ 标准 [C++,2020] 定义了 C++。
[2] 当为新项目选择风格或现代化代码库时，依赖 C++ 核心指南；§19.1.4。
[3] 学习 C++ 时，不要孤立地关注语言特性；§19.2.1。
[4] 不要固守几十年前的语言特性集和设计技术；§19.1.4。
[5] 在生产代码中使用新特性之前，通过编写小程序测试你计划使用的实现的标准符合性和性能。
[6] 为了学习 C++，使用你能获得的最新的、最完整的标准 C++ 实现。
[7] C 和 C++ 的公共子集不是学习 C++ 的最佳初始子集；§19.3.2.1。
[8] 避免强制转换；§19.3.2.1；[CG: ES.48]。
[9] 优先选择命名强制转换（如 `static_cast`）而不是 C 风格强制转换；§5.2.3；[CG: ES.49]。
[10] 将 C 程序转换为 C++ 时，重命名作为 C++ 关键字的变量；§19.3.2。
[11] 为了可移植性和类型安全，如果必须使用 C，请用 C 和 C++ 的公共子集编写；§19.3.2.1；[CG: CPL.2]。
[12] 将 C 程序转换为 C++ 时，将 `malloc()` 的结果强制转换为正确的类型，或将所有 `malloc()` 的使用改为使用 `new`；§19.3.2.2。
[13] 从 `malloc()` 和 `free()` 转换到 `new` 和 `delete` 时，考虑使用 `vector`、`push_back()` 和 `reserve()` 而不是 `realloc()`；§19.3.2.1。
[14] 在 C++ 中，没有从 `int` 到枚举的隐式转换；必要时使用显式类型转换。
[15] 对于每个将名称放在全局命名空间中的标准 C 头文件 `<X.h>`，头文件 `<cX>` 将名称放在命名空间 `std` 中。
[16] 在声明 C 函数时使用 `extern "C"`；§19.3.2.3。
[17] 相对于 C 风格字符串（直接操作零结尾的 `char` 数组），优先选择 `string`；[CG: SL.str.1]。
[18] 相对于 `stdio`，优先选择 `iostream`；[CG: SL.io.3]。
[19] 相对于内置数组，优先选择容器（例如 `vector`）。

===== 第 21 页 =====

A

# 模块 std

发明中的大事是：你必须有一个能工作的完整系统。 – J. Presper Eckert

- 引言
- 使用实现提供的内容
- 使用头文件
- 制作你自己的模块 std
- 建议

===== 第 22 页 =====

## A.1 引言

在撰写本文时，模块 std [Stroustrup,2021b] 不幸地尚未成为标准的一部分。我有合理的希望它会成为 C++23 的一部分。本附录提供了一些目前如何应对的想法。

模块 `std` 的理念是使用单个 `import std;` 语句就可以简单且廉价地获得标准库的所有组件。我在全书各章节中都依赖于此。提到头文件并命名它们主要是因为它们是传统的且普遍可用的，部分原因是它们反映了标准库（不完美的）历史组织。

少数标准库组件将名称（例如 `<cmath>` 中的 `sqrt()`）转储到全局命名空间中。模块 `std` 不会这样做，但当我们需要获得这些全局名称时，可以导入 `std.compat`。导入 `std.compat` 而非 `std` 的唯一真正好的理由是为了避免弄乱旧代码库，同时仍然获得模块带来的编译速度提升的部分好处。

请注意，模块特意不导出宏。如果你需要宏，请使用 `#include`。模块和头文件共存；也就是说，如果你同时 `#include` 和 `import` 一组相同的声明，你将得到一个一致的程序。这对于大型代码库从依赖头文件演进到使用模块至关重要。

## A.2 使用实现提供的内容

如果运气好，我们想要使用的实现已经提供了一个模块 `std`。在这种情况下，我们的首要选择应该是使用它。它可能被标记为“实验性的”，使用它可能需要一些设置或一些编译器选项。因此，首先要探索实现是否提供了模块 `std` 或等价物。例如，当前（2022 年春季）Visual Studio 提供了一些“实验性”模块，因此使用该实现，我们可以像这样定义模块 `std`：

```cpp
export module std;
export import std.regex;          // <regex>
export import std.filesystem;     // <filesystem>
export import std.memory;         // <memory>
export import std.threading;      // <atomic>, <condition_variable>, <future>, <mutex>,
                                  // <shared_mutex>, <thread>
export import std.core;           // 其他所有
```

===== 第 23 页 =====

显然，要做到这一点，我们必须使用 C++20 编译器，并且还需要设置选项以访问实验性模块。请注意，所有“实验性”的东西都会随时间变化。

## A.3 使用头文件

如果一个实现尚未支持模块，或者尚未提供模块 `std` 或等价的模块，我们可以回退到使用传统头文件。它们是标准且普遍可用的。问题在于，要使示例工作，我们需要弄清楚需要哪些头文件并 `#include` 它们。第 9 章可以提供帮助，我们可以在 [Cppreference] 上查找我们想要使用的特性的名称，以查看它属于哪个头文件。如果这变得乏味，我们可以将常用的头文件收集到一个 `std.h` 头文件中：

```cpp
// std.h

#include <iostream>
#include <string>
#include <vector>
#include <list>
#include <memory>
#include <algorithm>
// ...
```

然后

```cpp
#include "std.h"
```

这里的问题在于，`#include` 如此多的内容可能会导致编译非常慢 [Stroustrup,2021b]。

## A.4 制作你自己的模块 std

这是最不吸引人的替代方案，因为它可能是最费力的工作，但一旦有人完成了，就可以共享：

```cpp
module;
#include <iostream>
#include <string>
#include <vector>
#include <list>
#include <memory>
#include <algorithm>
// ...

export module std;
export import <iostream>;
export import <string>;
export import <vector>;
export import <list>;
export import <memory>;
export import <algorithm>;
// ...
```

有一种快捷方式：

```cpp
export module std;

export import "iostream";
export import "string";
export import "vector";
export import "list";
export import "memory";
export import "algorithms";
// ...
```

构造 `import "iostream";`

===== 第 25 页 =====

导入头文件单元是模块和头文件之间的一个中间地带。它接受一个头文件并将其变成类似于模块的东西，但它也可能将名称注入全局命名空间（如 `#include`）并泄露宏。

这不像 `#include` 那样编译得那么慢，但也不像一个正确构造的命名模块那样快。

## A.5 建议

[1] 优先使用实现提供的模块；§A.2。
[2] 使用模块；§A.3。
[3] 优先使用命名模块而不是头文件单元；§A.4。
[4] 要使用 C 标准中的宏和全局名称，请导入 `std.compat`；§A.1。
[5] 避免使用宏；§A.1；[CG: ES.30] [CG: ES.31]。

---

以上是第 19 章和附录 A 的完整翻译。由于原始文本中第 19 章之后包含大量索引条目（从第 26 页开始的索引），这些索引条目是术语表，为节省篇幅，此处不予翻译。如需翻译索引，请告知。