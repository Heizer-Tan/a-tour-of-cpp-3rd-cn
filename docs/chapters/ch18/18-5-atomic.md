## 18.5 原子操作

`std::atomic` 提供了无锁的线程安全操作：

```cpp
#include <atomic>

std::atomic<int> counter {0};

void increment()
{
    for (int i = 0; i < 1000000; ++i)
        ++counter;                          // 原子递增
}

// 多个线程可以安全地同时调用 increment()
```

原子操作比互斥锁更轻量，但只适用于简单的数据类型和操作。对于复杂的临界区，仍需要使用互斥锁。

常用原子操作：
- `load()` / `store()`：读取/写入
- `exchange()`：交换值
- `compare_exchange_weak()` / `compare_exchange_strong()`：比较并交换（CAS）
- `fetch_add()` / `fetch_sub()`：原子加减
