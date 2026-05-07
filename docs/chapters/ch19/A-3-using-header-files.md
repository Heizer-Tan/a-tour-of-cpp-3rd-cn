# A.3 使用头文�?
如果一个实现尚未支持模块，或者尚未提供模�?`std` 或等价的模块，我们可以回退到使用传统头文件。它们是标准且普遍可用的。问题在于，要使示例工作，我们需要弄清楚需要哪些头文件�?`#include` 它们。第 9 章可以提供帮助，我们可以�?[Cppreference] 上查找我们想要使用的特性的名称，以查看它属于哪个头文件。如果这变得乏味，我们可以将常用的头文件收集到一�?`std.h` 头文件中�?
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

这里的问题在于，`#include` 如此多的内容可能会导致编译非常慢 [Stroustrup,2021b]�?