## 15.2 智能指针

智能指针是管理动态分配对象生命周期的 RAII 包装器。

### 15.2.1 `std::unique_ptr`

`unique_ptr` 表示独占所有权。它不能被拷贝，只能被移动。

```cpp
auto p = std::make_unique<Circle>(Point{0, 0}, 10);
p->draw();                          // 通过 -> 访问成员

auto q = std::move(p);              // 转移所有权给 q
// p 现在为 nullptr

vector<unique_ptr<Shape>> shapes;
shapes.push_back(std::make_unique<Circle>(Point{0, 0}, 10));
shapes.push_back(std::make_unique<Triangle>(Point{1, 1}, Point{2, 2}, Point{3, 1}));
```

`unique_ptr` 是零开销的——它的大小与裸指针相同，且没有运行时开销。

### 15.2.2 `std::shared_ptr`

`shared_ptr` 表示共享所有权，通过引用计数管理生命周期。

```cpp
auto p = std::make_shared<Circle>(Point{0, 0}, 10);
auto q = p;                         // 引用计数变为 2
// 当最后一个 shared_ptr 被销毁时，对象被删除
```

`shared_ptr` 有额外的内存和性能开销（引用计数需要原子操作），仅在确实需要共享所有权时使用。

### 15.2.3 `std::weak_ptr`

`weak_ptr` 是对 `shared_ptr` 管理对象的非拥有引用：

```cpp
auto p = std::make_shared<int>(42);
std::weak_ptr<int> w = p;

if (auto sp = w.lock()) {           // 尝试获取 shared_ptr
    cout << *sp << '\n';            // 如果对象还存在
}
p.reset();                          // 释放对象
// w.lock() 现在返回 nullptr
```

`weak_ptr` 常用于打破 `shared_ptr` 的循环引用。
