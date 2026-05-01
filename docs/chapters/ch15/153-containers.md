## 15.3 容器和指针

### 15.3.1 在容器中存储指针

容器可以存储指针，但需要注意所有权：

```cpp
vector<Shape*> shapes1;                     // 裸指针：不管理所有权
vector<unique_ptr<Shape>> shapes2;          // 独占所有权
vector<shared_ptr<Shape>> shapes3;          // 共享所有权
```

优先使用 `vector<unique_ptr<Shape>>`——它清晰地表达了所有权，且自动管理生命周期。

### 15.3.2 `std::span`（C++20）

`span` 是数组或连续内存区域的视图。它不拥有数据，类似于 `string_view` 之于字符串。

```cpp
void process(std::span<int> data)           // 接受任何连续整数序列
{
    for (int& x : data)
        x *= 2;
}

vector<int> v = {1, 2, 3, 4, 5};
std::array<int, 5> a = {1, 2, 3, 4, 5};
int c_arr[] = {1, 2, 3, 4, 5};

process(v);      // OK
process(a);      // OK
process(c_arr);  // OK
```

`span` 的优势：
- 统一接口：接受 `vector`、`array`、C 数组等
- 安全：携带大小信息，支持边界检查
- 零开销：只是一个指针 + 大小
