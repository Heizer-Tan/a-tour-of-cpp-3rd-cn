## 13.2 非修改序列操作

这些算法不修改序列中的元素。

### 13.2.1 查找

```cpp
vector<int> v = {1, 2, 3, 4, 5, 3};

auto it = std::find(v.begin(), v.end(), 3);         // 查找第一个 3
auto it2 = std::find_if(v.begin(), v.end(),
    [](int x) { return x > 3; });                    // 查找第一个 >3 的元素
auto it3 = std::find_if_not(v.begin(), v.end(),
    [](int x) { return x < 3; });                    // 查找第一个 >=3 的元素
```

### 13.2.2 计数

```cpp
int n = std::count(v.begin(), v.end(), 3);           // 值为 3 的元素个数
int m = std::count_if(v.begin(), v.end(),
    [](int x) { return x % 2 == 0; });               // 偶数个数
```

### 13.2.3 比较

```cpp
vector<int> a = {1, 2, 3};
vector<int> b = {1, 2, 3};

bool eq = std::equal(a.begin(), a.end(), b.begin()); // true
```

### 13.2.4 搜索

```cpp
vector<int> v = {1, 2, 3, 4, 5};
vector<int> sub = {3, 4};

auto it = std::search(v.begin(), v.end(),
                      sub.begin(), sub.end());       // 查找子序列
```
