# 11.9 文件系统

大多数系统都有文件系统的概念，提供对存储为文件的持久信息的访问。不幸的是，文件系统的属性和操作它们的方式差异很大。为了解决这个问题，`<filesystem>` 中的文件系统库为大多数文件系统的大多数设施提供了统一的接口。使用 `<filesystem>`，我们可以可移植地：

- 表达文件系统路径并在文件系统中导航
- 检查文件类型及其关联的权限

文件系统库可以处理 unicode，但解释如何做超出了本书的范围。我推荐 cppreference [Cppreference] 和 Boost 文件系统文档 [Boost] 以获取详细信息。

## 11.9.1 路径

考虑一个例子：

```cpp
path f = "dir/hypothetical.cpp";   // 命名一个文件

assert(exists(f));                 // f 必须存在

if (is_regular_file(f))            // f 是普通文件吗？
    cout << f << " is a file; its size is " << file_size(f) << '\n';
```

注意，操作文件系统的程序通常与其他程序一起在计算机上运行。因此，文件系统的内容可能在两个命令之间发生变化。例如，即使我们首先仔细断言了 `f` 存在，当我们在下一行询问 `f` 是否是普通文件时，这可能不再为真。

`path` 是一个相当复杂的类，能够处理许多操作系统的不同字符集和约定。特别地，它可以处理来自 `main()` 呈现的命令行中的文件名；例如：

```cpp
int main(int argc, char* argv[])
{
    if (argc < 2) {
        cerr << "arguments expected\n";
        return 1;
    }

    path p {argv[1]};                // 从命令行创建一个路径
    cout << p << " " << exists(p) << '\n';   // 注意：path 可以像字符串一样打印
    // ...
}
```

路径在被使用之前不会检查其有效性。即使在那时，其有效性也取决于程序运行所在系统的约定。

自然地，路径可用于打开文件：

```cpp
void use(path p)
{
    ofstream f {p};
    if (!f) error("bad file name: ", p);
    f << "Hello, file!";
}
```

除了 `path`，`<filesystem>` 还提供了用于遍历目录和查询所找到文件属性的类型：

[表：文件系统类型（部分）]

考虑一个简单但并非完全不现实的例子：

```cpp
void print_directory(path p)   // 打印 p 中所有文件的名称
try {
    if (is_directory(p)) {
        cout << p << ":\n";
        for (const directory_entry& x : directory_iterator{p})
            cout << "  " << x.path() << '\n';
    }
}
catch (const filesystem_error& ex) {
    cerr << ex.what() << '\n';
}
```

字符串可以隐式转换为 `path`，因此我们可以这样运行 `print_directory`：

```cpp
void use()
{
    print_directory(".");   // 当前目录
    print_directory("..");  // 父目录
    print_directory("/");   // Unix 根目录
    print_directory("c:");  // Windows 卷 C

    for (string s; cin >> s; )
        print_directory(s);
}
```

如果我还想列出子目录，我会使用 `recursive_directory_iterator(p)`。如果我想按字典顺序打印条目，我会将路径复制到一个 `vector` 中，并在打印前对其进行排序。

类 `path` 提供了许多常见且有用的操作：

[表：路径操作（部分）]

例如：

```cpp
void test(path p)
{
    if (is_directory(p)) {
        cout << p << ":\n";
        for (const directory_entry& x : directory_iterator(p)) {
            const path& f = x;   // 引用目录条目的路径部分
            if (f.extension() == ".exe")
                cout << f.stem() << " is a Windows executable\n";
            else {
                string n = f.extension().string();
                if (n == ".cpp" || n == ".C" || n == ".cxx")
                    cout << f.stem() << " is a C++ source file\n";
            }
        }
    }
}
```

我们将 `path` 用作字符串（例如 `f.extension()`），并且可以从 `path` 中提取各种类型的字符串（例如 `f.extension().string()`）。

命名约定、自然语言和字符串编码非常复杂。标准库的文件系统抽象提供了可移植性和极大的简化。

## 11.9.2 文件与目录

自然，文件系统提供的操作非常多，不同操作系统提供的集合也不尽相同。标准库只包含少数能在多种系统上合理实现的操作。

**文件系统操作（部分）**（`p`、`p1`、`p2` 为路径；`e` 为 `error_code`；`b` 为表示成功与否的 `bool`）

| 操作 | 说明 |
|------|------|
| `exists(p)` | `p` 是否指向已存在的文件系统对象？ |
| `copy(p1,p2)` | 将文件或目录从 `p1` 复制到 `p2`；以异常报告错误 |
| `copy(p1,p2,e)` | 复制文件或目录；以错误码报告错误 |
| `b=copy_file(p1,p2)` | 将文件内容从 `p1` 复制到 `p2`；以异常报告错误 |
| `b=create_directory(p)` | 创建名为 `p` 的新目录；`p` 上的所有中间目录必须已存在 |
| `b=create_directories(p)` | 创建名为 `p` 的新目录；必要时创建 `p` 上的所有中间目录 |
| `p=current_path()` | `p` 为当前工作目录 |
| `current_path(p)` | 将当前工作目录设为 `p` |
| `s=file_size(p)` | `s` 为 `p` 的字节数 |
| `b=remove(p)` | 若 `p` 是文件或空目录则删除 |

许多操作还有接受额外参数（例如操作系统权限）的重载。其处理方式远超出本书范围，需要时请自行查阅。

与 `copy()` 一样，每个操作通常都有两种用法：

- **基本版本**（如上表所列），例如 `exists(p)`：操作失败时抛出 `filesystem_error`。
- **带 `error_code` 的版本**，例如 `exists(p, e)`：通过检查 `e` 判断成功与否。

当操作在正常使用中预计会**经常**失败时，使用错误码版本；当失败应被视为**异常**时，使用抛出版本。

通常，使用查询函数是检查文件属性最简单直接的做法。`<filesystem>` 了解几类常见文件，并将其余归为“其他”：

**文件类型**（`f` 为 `path` 或 `file_status`）

| 谓词 | 含义 |
|------|------|
| `is_block_file(f)` | `f` 是否为块设备？ |
| `is_character_file(f)` | `f` 是否为字符设备？ |
| `is_directory(f)` | `f` 是否为目录？ |
| `is_empty(f)` | `f` 是否为空文件或空目录？ |
| `is_fifo(f)` | `f` 是否为命名管道？ |
| `is_other(f)` | `f` 是否为其他类型文件？ |
| `is_regular_file(f)` | `f` 是否为普通（常规）文件？ |
| `is_socket(f)` | `f` 是否为命名 IPC 套接字？ |
| `is_symlink(f)` | `f` 是否为符号链接？ |
| `status_known(f)` | `f` 的文件状态是否已知？ |
