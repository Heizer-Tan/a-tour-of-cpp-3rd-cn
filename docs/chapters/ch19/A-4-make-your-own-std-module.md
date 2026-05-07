# A.4 制作你自己的模块 std

这是最不吸引人的替代方案，因为它可能是最费力的工作，但一旦有人完成了，就可以共享�?
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

构�?`import "iostream";` 导入头文件单元是模块和头文件之间的一个中间地带。它接受一个头文件并将其变成类似于模块的东西，但它也可能将名称注入全局命名空间（如 `#include`）并泄露宏�?
这不�?`#include` 那样编译得那么慢，但也不像一个正确构造的命名模块那样快�?