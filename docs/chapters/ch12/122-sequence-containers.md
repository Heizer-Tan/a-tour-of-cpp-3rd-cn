## 12.2 序列容器

序列容器按线性顺序存储元素。

### 12.2.1 `std::vector`

`vector` 是动态数组，元素在内存中连续存储。它是默认的容器选择。

```cpp
vector<int> v = {1, 2, 3, 4, 5};
v.push_back(6);             // 在末尾添加
v.pop_back();               // 删除末尾元素
int x = v[2];               // 随机访问: O(1)
v.insert(v.begin(), 0);     // 在开头插入: O(n)
```

- 随机访问：O(1)
- 末尾插入/删除：均摊 O(1)
- 中间插入/删除：O(n)

### 12.2.2 `std::list`

`list` 是双向链表，支持在任意位置高效插入和删除。

```cpp
list<int> l = {1, 2, 3, 4, 5};
l.push_front(0);            // 在开头添加: O(1)
l.push_back(6);             // 在末尾添加: O(1)
auto it = l.begin();
++it; ++it;
l.insert(it, 99);           // 在指定位置插入: O(1)
```

- 任意位置插入/删除：O(1)
- 不支持随机访问
- 内存开销较大（每个节点需要两个指针）

### 12.2.3 `std::deque`

`deque`（双端队列）支持在两端高效插入和删除。

```cpp
deque<int> d = {1, 2, 3};
d.push_front(0);            // 前端插入: O(1)
d.push_back(4);             // 后端插入: O(1)
int x = d[2];               // 随机访问: O(1)
```

### 12.2.4 `std::array`（C++11）

`array` 是固定大小的数组，大小在编译时确定。

```cpp
std::array<int, 5> a = {1, 2, 3, 4, 5};
int x = a[2];               // 随机访问: O(1)
size_t n = a.size();        // 5
```

### 12.2.5 `std::forward_list`（C++11）

`forward_list` 是单向链表，内存开销最小。

```cpp
forward_list<int> fl = {1, 2, 3};
fl.push_front(0);           // 前端插入: O(1)
```
