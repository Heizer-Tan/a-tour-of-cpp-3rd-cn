## 18.3 共享数据和同步

### 18.3.1 互斥锁

`std::mutex` 用于保护共享数据：

```cpp
std::mutex m;
int shared_data;

void writer()
{
    std::lock_guard<std::mutex> lock {m};   // 获取锁
    shared_data = 42;
    // lock 在离开作用域时自动释放
}

void reader()
{
    std::lock_guard<std::mutex> lock {m};
    cout << shared_data << '\n';
}
```

`std::lock_guard` 是 RAII 包装器——在构造时获取锁，在析构时释放锁。这确保了即使发生异常，锁也会被释放。

### 18.3.2 更灵活的锁

`std::unique_lock` 提供比 `lock_guard` 更灵活的锁管理：

```cpp
std::mutex m;
std::unique_lock<std::mutex> lock {m, std::defer_lock};  // 延迟锁定
// ...
lock.lock();        // 手动锁定
// ...
lock.unlock();      // 手动解锁
```

### 18.3.3 条件变量

`std::condition_variable` 用于线程间的信号通知：

```cpp
std::mutex m;
std::condition_variable cv;
bool ready = false;
int data;

void producer()
{
    {
        std::lock_guard<std::mutex> lock {m};
        data = 42;
        ready = true;
    }
    cv.notify_one();    // 通知一个等待的线程
}

void consumer()
{
    std::unique_lock<std::mutex> lock {m};
    cv.wait(lock, [] { return ready; });  // 等待直到 ready 为 true
    cout << data << '\n';
}
```
