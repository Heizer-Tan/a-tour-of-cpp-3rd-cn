## 19.3 C 兼容性

C++ 在很大程度上兼容 C 语言，但有一些重要的区别：

### 19.3.1 兼容的方面

- 大多数 C 程序可以作为 C++ 程序编译
- C 标准库在 C++ 中可用（通过 `<c...>` 头文件）
- C 和 C++ 代码可以通过 `extern "C"` 链接

```cpp
extern "C" {
    #include "legacy_c_header.h"
}
```

### 19.3.2 不兼容的方面

- C++ 有更严格的类型检查（如 `void*` 不能隐式转换为其他指针）
- C++ 关键字（如 `class`、`new`、`delete`）在 C 中不是保留字
- C++ 不支持 C99 的某些特性（如变长数组 VLA、`restrict` 关键字）
- `sizeof('a')` 在 C 中是 `sizeof(int)`，在 C++ 中是 `sizeof(char)`

### 19.3.3 从 C 迁移到 C++

- 用 `std::string` 替代 `char*`
- 用 `std::vector` 替代动态数组
- 用智能指针替代手动 `malloc`/`free`
- 用 `std::iostream` 替代 `printf`/`scanf`
- 用 RAII 管理资源
