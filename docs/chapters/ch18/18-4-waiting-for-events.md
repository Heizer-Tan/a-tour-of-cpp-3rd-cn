# 18.4 等待事件

有时线程需要等待某种外部事件：例如另一线程完成任务，或经过一段时间。最简单的「事件」只是时间流逝。借助 `<chrono>` 的时间设施可以写成：

```cpp
using namespace chrono;

auto t0 = high_resolution_clock::now();
this_thread::sleep_for(milliseconds{20});
auto t1 = high_resolution_clock::now();

cout << duration_cast<nanoseconds>(t1 - t0).count() << " nanoseconds passed\n";
```

我甚至不必另起线程；默认情况下 `this_thread` 指的就是这一个线程。

我用 `duration_cast` 把时钟单位调整到想要的纳秒。

使用外部事件通信的基本支持由 `<condition_variable>` 中的 `condition_variable` 提供。`condition_variable` 是一种让一个线程等待另一线程的机制。特别是，它允许线程等待某个条件（常称为事件）因其它线程的工作而成为真。

使用条件变量能实现多种优雅高效的共享方式，但也颇为棘手。考虑经典的两线程经由队列传消息的例子。为简单起见，我把队列以及避免队列竞争的机制声明在生产者与消费者的全局作用域：

```cpp
class Message {
    // ...
};

queue<Message> mqueue;
condition_variable mcond;
mutex mmutex;
```

类型 `queue`、`condition_variable` 与 `mutex` 均由标准库提供。

`consumer()` 读取并处理 `Message`：

```cpp
void consumer()
{
    while (true) {
        unique_lock lck{mmutex};                           // 取得 mmutex
        mcond.wait(lck, [] { return !mqueue.empty(); });    // 释放 mmutex 并等待；
                                                              // 唤醒时重新取得 mmutex
        auto m = mqueue.front();
        mqueue.pop();
        lck.unlock();                                      // 释放 mmutex
        // ... 处理 m ...
    }
}
```

此处我用 `unique_lock` 明确保护对队列以及对 `condition_variable` 的操作。在条件变量上等待时会**释放**其锁实参，直到等待结束（队列非空）再重新取得它。对条件的显式检查——此处 `!mqueue.empty()`——可防止「醒来后发现别的任务抢先一步」导致条件不再成立。

我用 `unique_lock` 而非 `scoped_lock`，原因有二：

- 需要把锁传给条件变量的 `wait()`。`scoped_lock` 不能移动，而 `unique_lock` 可以。
- 希望在处理消息**之前**解锁保护条件变量的互斥体。`unique_lock` 提供 `lock()`、`unlock()` 等低级同步控制。

另一方面，`unique_lock` 只能管理单个互斥体。

对应的 `producer()` 如下：

```cpp
void producer()
{
    while (true) {
        Message m;
        // ... 填充消息 ...
        scoped_lock lck{mmutex};    // 保护操作
        mqueue.push(m);
        mcond.notify_one();         // 通知
    }                               // 作用域末尾释放 mmutex
}
```
