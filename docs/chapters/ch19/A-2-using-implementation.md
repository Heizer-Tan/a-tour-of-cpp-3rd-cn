# A.2 使用实现提供的内�?
如果运气好，我们想要使用的实现已经提供了一个模�?`std`。在这种情况下，我们的首要选择应该是使用它。它可能被标记为"实验性的"，使用它可能需要一些设置或一些编译器选项。因此，首先要探索实现是否提供了模块 `std` 或等价物。例如，当前�?022 年春季）Visual Studio 提供了一�?实验�?模块，因此使用该实现，我们可以像这样定义模块 `std`�?
```cpp
export module std;
export import std.regex;          // <regex>
export import std.filesystem;     // <filesystem>
export import std.memory;         // <memory>
export import std.threading;      // <atomic>, <condition_variable>, <future>, <mutex>,
                                  // <shared_mutex>, <thread>
export import std.core;           // 其他所�?```

显然，要做到这一点，我们必须使用 C++20 编译器，并且还需要设置选项以访问实验性模块。请注意，所�?实验�?的东西都会随时间变化�?