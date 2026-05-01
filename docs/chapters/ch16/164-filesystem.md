## 16.4 文件系统

`std::filesystem`（C++17）提供了跨平台的文件系统操作。

### 16.4.1 路径

```cpp
#include <filesystem>
namespace fs = std::filesystem;

fs::path p = "docs/chapters/ch01/index.md";
cout << p.filename() << '\n';               // "index.md"
cout << p.stem() << '\n';                   // "index"
cout << p.extension() << '\n';              // ".md"
cout << p.parent_path() << '\n';            // "docs/chapters/ch01"
```

### 16.4.2 目录操作

```cpp
fs::create_directory("output");             // 创建目录
fs::create_directories("a/b/c");            // 递归创建目录

for (const auto& entry : fs::directory_iterator("docs")) {
    cout << entry.path() << '\n';           // 遍历目录
}

for (const auto& entry : fs::recursive_directory_iterator("docs")) {
    cout << entry.path() << '\n';           // 递归遍历
}
```

### 16.4.3 文件操作

```cpp
if (fs::exists("config.json")) {
    auto size = fs::file_size("config.json");
    cout << "Size: " << size << " bytes\n";
}

fs::copy("src.txt", "dst.txt");             // 复制文件
fs::rename("old.txt", "new.txt");           // 重命名
fs::remove("temp.txt");                     // 删除文件
fs::remove_all("temp_dir");                 // 递归删除目录
```
