## 10.3 字符串视图

`std::string_view`（C++17）是一个字符串的只读视图。它不拥有数据，只是指向已有字符序列的指针和长度。

```cpp
string s = "Hello, World!";
string_view sv = s;                 // 不拷贝，只是引用

string_view sub = sv.substr(7, 5);  // 不拷贝，只是调整指针和长度
cout << sub << '\n';                // "World"
```

`string_view` 的优势：

- 零拷贝：作为函数参数时避免不必要的内存分配
- 灵活性：可以指向 `string`、`const char*` 或字符数组
- 高效：`substr` 是 O(1) 操作

```cpp
void print(string_view sv)          // 接受任何字符串类型
{
    cout << sv << '\n';
}

print("Hello");                     // const char*
print(string("World"));             // std::string
print("C++"sv);                     // std::string_view
```

注意：`string_view` 不保证以 null 结尾，且不拥有数据——确保底层数据在 `string_view` 使用期间保持有效。
