## 13.3 修改序列操作

这些算法会修改序列中的元素。

### 13.3.1 复制

```cpp
vector<int> src = {1, 2, 3, 4, 5};
vector<int> dst(src.size());

std::copy(src.begin(), src.end(), dst.begin());
std::copy_if(src.begin(), src.end(), dst.begin(),
    [](int x) { return x % 2 == 0; });               // 只复制偶数
```

### 13.3.2 变换

```cpp
vector<int> v = {1, 2, 3, 4, 5};
vector<int> result(v.size());

std::transform(v.begin(), v.end(), result.begin(),
    [](int x) { return x * x; });                    // result = {1, 4, 9, 16, 25}
```

### 13.3.3 替换

```cpp
std::replace(v.begin(), v.end(), 3, 33);             // 将所有 3 替换为 33
std::replace_if(v.begin(), v.end(),
    [](int x) { return x < 3; }, 0);                 // 将 <3 的元素替换为 0
```

### 13.3.4 填充

```cpp
std::fill(v.begin(), v.end(), 42);                   // 全部填充为 42
std::fill_n(v.begin(), 3, 99);                       // 前 3 个填充为 99
```

### 13.3.5 移除

```cpp
auto new_end = std::remove(v.begin(), v.end(), 3);   // 移除所有 3
v.erase(new_end, v.end());                           // 实际删除

// 或使用 erase-remove 惯用法
v.erase(std::remove_if(v.begin(), v.end(),
    [](int x) { return x % 2 == 0; }), v.end());
```

注意：`remove` 不会改变容器大小——它只是将不需要的元素移到末尾，返回新的逻辑末尾迭代器。需要配合 `erase` 来实际删除。
