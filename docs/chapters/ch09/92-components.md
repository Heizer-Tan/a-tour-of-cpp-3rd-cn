## 9.2 标准库组件

标准库的主要组件包括：

### 9.2.1 容器

容器（containers）用于存储和管理对象集合：

- **序列容器**：`vector`、`list`、`deque`、`array`、`forward_list`
- **关联容器**：`set`、`map`、`multiset`、`multimap`
- **无序关联容器**：`unordered_set`、`unordered_map` 等
- **容器适配器**：`stack`、`queue`、`priority_queue`

```cpp
vector<int> v = {1, 2, 3, 4, 5};
map<string, int> ages = {{"Alice", 30}, {"Bob", 25}};
```

### 9.2.2 算法

算法（algorithms）提供对序列的通用操作：

- 非修改操作：`find`、`count`、`search`、`equal`
- 修改操作：`copy`、`transform`、`replace`、`fill`
- 排序操作：`sort`、`stable_sort`、`partial_sort`
- 数值操作：`accumulate`、`inner_product`

```cpp
std::sort(v.begin(), v.end());
auto it = std::find(v.begin(), v.end(), 3);
```

### 9.2.3 迭代器

迭代器（iterators）是算法和容器之间的桥梁。主要类别包括：

- 输入迭代器（input iterator）
- 输出迭代器（output iterator）
- 前向迭代器（forward iterator）
- 双向迭代器（bidirectional iterator）
- 随机访问迭代器（random-access iterator）

### 9.2.4 字符串和正则表达式

- `std::string`：动态字符串
- `std::string_view`（C++17）：字符串的只读视图
- `std::regex`：正则表达式匹配和替换

### 9.2.5 输入/输出

- `std::iostream`：控制台 I/O
- `std::fstream`：文件 I/O
- `std::stringstream`：字符串 I/O
- `std::format`（C++20）：类型安全的格式化

### 9.2.6 并发

- `std::thread`：操作系统线程
- `std::mutex`、`std::lock_guard`：互斥和锁
- `std::future`、`std::async`：异步任务
- `std::atomic`：原子操作

### 9.2.7 数值

- `std::complex`：复数
- `std::valarray`：数值数组
- `std::random`：随机数生成
- `std::chrono`：时间工具
