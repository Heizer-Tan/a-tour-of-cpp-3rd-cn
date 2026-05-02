## 13.4 排序和搜索

### 13.4.1 排序

```cpp
vector<int> v = {5, 3, 1, 4, 2};

std::sort(v.begin(), v.end());                       // 升序: {1, 2, 3, 4, 5}
std::sort(v.begin(), v.end(), std::greater<int>{});  // 降序: {5, 4, 3, 2, 1}

// 自定义比较
std::sort(v.begin(), v.end(),
    [](int a, int b) { return abs(a) < abs(b); });
```

### 13.4.2 稳定排序

```cpp
std::stable_sort(v.begin(), v.end());                // 保持相等元素的相对顺序
```

### 13.4.3 部分排序

```cpp
std::partial_sort(v.begin(), v.begin() + 3, v.end()); // 前 3 个最小元素就位
```

### 13.4.4 二分搜索

```cpp
// 前提：序列已排序
bool found = std::binary_search(v.begin(), v.end(), 3);
auto it = std::lower_bound(v.begin(), v.end(), 3);   // 第一个 >=3 的位置
auto it2 = std::upper_bound(v.begin(), v.end(), 3);  // 第一个 >3 的位置
```

### 13.4.5 合并

```cpp
vector<int> a = {1, 3, 5};
vector<int> b = {2, 4, 6};
vector<int> result(6);

std::merge(a.begin(), a.end(), b.begin(), b.end(), result.begin());
// result = {1, 2, 3, 4, 5, 6}
```
