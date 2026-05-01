## 12.4 无序容器

无序容器基于哈希表实现，提供平均 O(1) 的查找时间。

### 12.4.1 `std::unordered_set`

```cpp
unordered_set<int> us = {3, 1, 4, 1, 5};
us.insert(2);
if (us.contains(3)) { /* ... */ }  // C++20
auto it = us.find(4);              // 平均 O(1)
```

### 12.4.2 `std::unordered_map`

```cpp
unordered_map<string, int> scores;
scores["Alice"] = 95;
scores["Bob"] = 87;

for (const auto& [name, score] : scores) {
    cout << name << ": " << score << '\n';
}
```

### 12.4.3 有序 vs 无序

| 特性 | 有序容器 | 无序容器 |
|------|---------|---------|
| 查找 | O(log n) | 平均 O(1) |
| 元素顺序 | 排序 | 任意 |
| 内存开销 | 较小 | 较大（哈希表） |
| 需要 `<` 运算符 | 是 | 否（需要 `==` 和哈希函数） |

当需要有序遍历时使用有序容器；当只需要快速查找时使用无序容器。
