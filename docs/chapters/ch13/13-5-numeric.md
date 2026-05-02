## 13.5 数值算法

数值算法定义在 `<numeric>` 头文件中。

### 13.5.1 累加

```cpp
vector<int> v = {1, 2, 3, 4, 5};

int sum = std::accumulate(v.begin(), v.end(), 0);    // 15
int product = std::accumulate(v.begin(), v.end(), 1,
    std::multiplies<int>{});                          // 120
```

### 13.5.2 内积

```cpp
vector<int> a = {1, 2, 3};
vector<int> b = {4, 5, 6};

int dot = std::inner_product(a.begin(), a.end(), b.begin(), 0);
// 1*4 + 2*5 + 3*6 = 32
```

### 13.5.3 前缀和

```cpp
vector<int> v = {1, 2, 3, 4, 5};
vector<int> result(v.size());

std::partial_sum(v.begin(), v.end(), result.begin());
// result = {1, 3, 6, 10, 15}
```

### 13.5.4 相邻差

```cpp
std::adjacent_difference(v.begin(), v.end(), result.begin());
// result = {1, 1, 1, 1, 1}
```
