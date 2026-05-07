
有时，线程需要等待某种外部事件，例如另一个线程完成一个任务或已经过去一段时间。最简单的“事件”仅仅是时间的流逝。使用 `<chrono>` 中的时间设施，我可以写：

```cpp
using namespace chrono;   // 见 §16.2.1

auto t0 = high_resolution_clock::now();
this_thread::sleep_for(milliseconds{20});
auto t1 = high_resolution_clock::now();

cout << duration_cast<nanoseconds>(t1 - t0).count() << " nanoseconds passed\n";
```

我甚至不需要启动一个线程；默认情况下，`this_thread` 可以指代唯一的那一个线程。

我使用 `duration_cast` 将时钟的单位调整为我想要的纳秒。

使用外部事件进行通信的基本支持由 `<condition_variable>` 中的 `condition_variable` 提供。`condition_variable` 是一种允许一个线程等待另一个线程的机制。特别地，它允许一个线程等待某个条件（通常称为**事件**）发生，该条件是其他线程工作的结果。

===== 第 11 页 =====

使用 `condition_variable` 支持多种优雅高效的共享形式，但可能相当棘手。考虑两个线程通过队列传递消息进行通信的经典示例。为简单起见，我将队列以及避免该队列上竞争条件的机制声明为生产者和消费者的全局变量：

```cpp
class Message {   // 要通信的对象
    // ...
};

queue<Message> mqueue;          // 消息队列
condition_variable mcond;       // 通信事件的变量
mutex mmutex;                   // 用于同步对 mcond 的访问
```

类型 `queue`、`condition_variable` 和 `mutex` 由标准库提供。

`consumer()` 读取并处理 `Message`：

```cpp
void consumer()
{
    while (true) {
        unique_lock lck {mmutex};                     // 获取 mmutex
        mcond.wait(lck, [] { return !mqueue.empty(); }); // 释放 mmutex 并等待，直到队列非空
        auto m = mqueue.front();                      // 获取消息
        mqueue.pop();
        lck.unlock();                                 // 释放 mmutex
        // ... 处理 m ...
    }
}
```

这里，我使用 `unique_lock` 显式地保护对队列和 `condition_variable` 的操作。等待 `condition_variable` 会释放其锁参数，直到等待结束（以便队列变为非空），然后重新获取它。显式检查条件（此处为 `!mqueue.empty()`）可以防止在唤醒后发现其他任务“抢先一步”从而导致条件不再满足的情况。

我使用 `unique_lock` 而非 `scoped_lock` 有两个原因：

1. 我们需要将锁传递给 `condition_variable` 的 `wait()`。`scoped_lock` 不可移动，但 `unique_lock` 可以。
2. 我们希望在处理消息之前解锁保护条件变量的互斥量。`unique_lock` 提供了诸如 `lock()` 和 `unlock()` 之类的操作，用于低级别的同步控制。

另一方面，`unique_lock` 只能处理单个互斥量。

相应的生产者看起来像这样：

```cpp
void producer()
{
    while (true) {
        Message m;
        // ... 填充消息 ...
        scoped_lock lck {mmutex};   // 保护操作
        mqueue.push(m);
        mcond.notify_one();         // 通知
    }   // 释放 mmutex（在作用域结束时）
}
```

## 18.5 任务间通信