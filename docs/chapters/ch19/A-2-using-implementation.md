# A.2 使用实现所提供的内容

如果运气好，我们想要使用的实现已经提供了一个 module `std`。在这种情况下，我们的首要选择应该是使用它。它可能被标记为“实验性的”，使用它可能需要一些设置或一些编译器选项。因此，首先要探查该实现是否提供了 module `std` 或等价物。例如，当前（2022 年春季）Visual Studio 提供了若干“实验性”模块，因此使用该实现，我们可以像这样定义 module `std`：

```cpp
export module std;
export import std.regex;          // <regex>
export import std.filesystem;     // <filesystem>
export import std.memory;         // <memory>
export import std.threading;      // <atomic>, <condition_variable>, <future>, <mutex>,
                                  // <shared_mutex>, <thread>
export import std.core;           // 其余全部
```

显然，要做到这一点，我们必须使用 C++20 编译器，并且还需要设置选项以访问实验性模块。请注意，所有“实验性”的东西都会随时间变化。
