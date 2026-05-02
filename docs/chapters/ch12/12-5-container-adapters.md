## 12.5 容器适配器

容器适配器在现有容器上提供受限的接口。

### 12.5.1 `std::stack`

后进先出（LIFO）：

```cpp
stack<int> s;
s.push(1);
s.push(2);
s.push(3);
while (!s.empty()) {
    cout << s.top() << ' ';  // 3 2 1
    s.pop();
}
```

### 12.5.2 `std::queue`

先进先出（FIFO）：

```cpp
queue<int> q;
q.push(1);
q.push(2);
q.push(3);
while (!q.empty()) {
    cout << q.front() << ' ';  // 1 2 3
    q.pop();
}
```

### 12.5.3 `std::priority_queue`

按优先级出队（默认最大优先）：

```cpp
priority_queue<int> pq;
pq.push(3);
pq.push(1);
pq.push(4);
while (!pq.empty()) {
    cout << pq.top() << ' ';  // 4 3 1
    pq.pop();
}
```
