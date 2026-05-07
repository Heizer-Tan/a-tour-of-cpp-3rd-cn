# 12.7 分配器

默认情况下，标准库容器使用 `new` 分配空间。运算符 `new` 和 `delete` 提供了一个通用的自由存储区（也称为动态内存或堆），可以容纳任意大小的对象，并具有用户控制的生命周期。这意味着时间和空间开销，在许多特殊情况下可以消除。因此，标准库容器提供了在需要时安装具有特定语义的分配器的机会。这已被用于解决各种与性能相关的问题（例如池分配器）、安全性（在删除时清理内存的分配器）、每线程分配以及非统一内存架构（在特定内存中分配并匹配指针类型）。这里不是讨论这些重要但非常专业且通常是高级技术的地方。然而，我将给出一个由真实世界问题驱动的例子，其中池分配器是解决方案。

一个重要的长期运行系统使用了一个事件队列（见 §18.4），将向量作为事件，以 `shared_ptr` 传递。这样，事件的最后一个使用者会隐式删除它：

```cpp
struct Event {
    vector<int> data = vector<int>(512);
};

list<shared_ptr<Event>> q;

void producer()
{
    for (int n = 0; n != LOTS; ++n) {
        lock_guard lk {m};               // m 是一个互斥量；见 §18.3
        q.push_back(make_shared<Event>());
        cv.notify_one();                 // cv 是一个条件变量；见 §18.4
    }
}
```

从逻辑角度来看，这工作得很好。逻辑上简单，因此代码健壮且可维护。不幸的是，这导致了大量的内存碎片。在 16 个生产者和 4 个消费者之间传递了 100,000 个事件之后，消耗了超过 6GB 的内存。

解决碎片问题的传统方法是重写代码以使用池分配器。池分配器是一种管理单个固定大小对象的分配器，它一次为许多对象分配空间，而不是使用单独的分配。幸运的是，C++ 直接支持这一点。池分配器定义在 `std` 的 `pmr`（“多态内存资源”）子命名空间中：

```cpp
pmr::synchronized_pool_resource pool;   // 创建一个池

struct Event {
    vector<int> data = vector<int>{512, &pool};   // 让 Events 使用池
};

list<shared_ptr<Event>> q {&pool};                // 让 q 使用池

void producer()
{
    for (int n = 0; n != LOTS; ++n) {
        scoped_lock lk {m};                       // m 是一个互斥量（§18.3）
        q.push_back(allocate_shared<Event, pmr::polymorphic_allocator<Event>>(pmr::polymorphic_allocator<Event>{&pool}));
        cv.notify_one();
    }
}
```

===== 第 17 页 =====

现在，在 16 个生产者和 4 个消费者之间传递了 100,000 个事件之后，消耗的内存不到 3MB。这大约提升了 2000 倍！当然，实际使用的内存量（相对于浪费在碎片上的内存）没有变化。消除碎片后，内存使用随时间保持稳定，因此系统可以运行数月。

这类技术从 C++ 的早期就被应用并取得了良好的效果，但通常需要重写代码以使用专门的容器。现在，标准容器可以选择性地接受分配器参数。默认情况下，容器使用 `new` 和 `delete`。其他多态内存资源包括：

- `unsynchronized_polymorphic_resource`：类似于 `polymorphic_resource`，但只能由一个线程使用。
- `monotonic_polymorphic_resource`：一个快速的分配器，仅在销毁时释放其内存，并且只能由一个线程使用。

多态资源必须派生自 `memory_resource` 并定义成员 `allocate()`、`deallocate()` 和 `is_equal()`。其理念是让用户构建自己的资源来调整代码。
