# 18.7 建议

[1] 使用并发来改善响应度或提升吞吐；[§18.1](18-1-introduction.md)。

[2] 在你力所能及的最高抽象层级上工作；[§18.1](18-1-introduction.md)。

[3] 把进程视作线程之外的备选方案；[§18.1](18-1-introduction.md)。

[4] 标准库的并发设施是类型安全的；[§18.1](18-1-introduction.md)。

[5] 内存模型意在让大多数程序员不必下降到计算机体系结构层面思考；[§18.1](18-1-introduction.md)。

[6] 内存模型使内存大体符合朴素直觉；[§18.1](18-1-introduction.md)。

[7] 原子操作使得无锁编程成为可能；[§18.1](18-1-introduction.md)。

[8] 把无锁编程留给专家；[§18.1](18-1-introduction.md)。

[9] 有时顺序解法比并发解法更简单也更快；[§18.1](18-1-introduction.md)。

[10] 避免数据竞争；[§18.1](18-1-introduction.md)，[§18.2](18-2-tasks-and-threads.md)。

[11] 相较直接使用并发，更优先考虑并行算法；[§18.1](18-1-introduction.md)，[§18.5.3](18-5-inter-task-communication.md#18.5.3)。

[12] 线程是对系统线程的类型安全封装；[§18.2](18-2-tasks-and-threads.md)。

[13] 用 `join()` 等待线程结束；[§18.2](18-2-tasks-and-threads.md)。

[14] 相较于裸 `thread`，更偏向 `jthread`；[§18.2](18-2-tasks-and-threads.md)。

[15] 尽可能避免显式共享数据；[§18.2](18-2-tasks-and-threads.md)。

[16] 相较于手动 `lock`/`unlock`，更偏向 RAII；[§18.3](18-3-shared-data.md)；[CG: CP.20]。

[17] 用 `scoped_lock` 管理互斥体；[§18.3](18-3-shared-data.md)。

[18] 用 `scoped_lock` 同时取得多把锁；[§18.3](18-3-shared-data.md)；[CG: CP.21]。

[19] 用 `shared_lock` 实现读者—写者锁；[§18.3](18-3-shared-data.md)。

[20] 把互斥体与它保护的数据一起定义；[§18.3](18-3-shared-data.md)；[CG: CP.50]。

[21] 只做很简单的共享时使用原子类型；[§18.3.2](18-3-shared-data.md#18.3.2)。

[22] 用 `condition_variable` 协调线程间通信；[§18.4](18-4-waiting-for-events.md)。

[23] 当你需要转移锁或需要更低级同步控制时，使用 `unique_lock`（而非 `scoped_lock`）；[§18.4](18-4-waiting-for-events.md)。

[24] 与 `condition_variable` 一起使用时，采用 `unique_lock`（而非 `scoped_lock`）；[§18.4](18-4-waiting-for-events.md)。

[25] 不要在无条件检查时盲目等待；[§18.4](18-4-waiting-for-events.md)；[CG: CP.42]。

[26] 最小化临界区内耗时；[§18.4](18-4-waiting-for-events.md)；[CG: CP.43]。

[27] 从并发任务的角度思考，而不是死盯着线程；[§18.5](18-5-inter-task-communication.md)。

[28] 崇尚简洁；[§18.5](18-5-inter-task-communication.md)。

[29] 相较于直接使用线程与互斥体，更偏向 `packaged_task` 与 future；[§18.5](18-5-inter-task-communication.md)。

[30] 用 `promise` 返回值，用 `future` 取结果；[§18.5.1](18-5-inter-task-communication.md#18.5.1)；[CG: CP.60]。

[31] 用 `packaged_task` 处理任务抛出的异常；[§18.5.2](18-5-inter-task-communication.md#18.5.2)。

[32] 用 `packaged_task` 与 `future` 表达对外部服务的请求并等待响应；[§18.5.2](18-5-inter-task-communication.md#18.5.2)。

[33] 用 `async()` 启动简单任务；[§18.5.3](18-5-inter-task-communication.md#18.5.3)；[CG: CP.61]。

[34] 用 `stop_token` 实现协作式终止；[§18.5.4](18-5-inter-task-communication.md#18.5.4)。

[35] 协程可比线程小得多；[§18.6](18-6-coroutines.md)。

[36] 相较于手写底层代码，更偏向协程支持库；[§18.6](18-6-coroutines.md)。
