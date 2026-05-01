## 12.3 关联容器

关联容器基于键来组织元素，通常使用平衡二叉树（红黑树）实现。

### 12.3.1 `std::set`

`set` 存储唯一键的集合，元素自动排序。

```cpp
set<int> s = {3, 1, 4, 1, 5};   // {1, 3, 4, 5} (重复的 1 被忽略)
s.insert(2);                      // {1, 2, 3, 4, 5}
if (s.contains(3)) { /* ... */ }  // C++20
auto it = s.find(4);              // 查找: O(log n)
```

### 12.3.2 `std::map`

`map` 是键值对的集合，按键排序。

```cpp
map<string, int> ages;
ages["Alice"] = 30;
ages["Bob"] = 25;
ages["Charlie"] = 35;

for (const auto& [name, age] : ages) {  // C++17 结构化绑定
    cout << name << " is " << age << '\n';
}
```

### 12.3.3 `std::multiset` 和 `std::multimap`

允许重复键的版本：

```cpp
multiset<int> ms = {1, 1, 2, 3, 3, 3};
multimap<string, int> mm = {{"A", 1}, {"A", 2}, {"B", 3}};
```
