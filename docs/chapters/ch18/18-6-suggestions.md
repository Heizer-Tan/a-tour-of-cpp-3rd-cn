## 18.6 建议

1. 使用 `std::thread` 创建并发执行的任务
2. 使用 `std::mutex` 和 `std::lock_guard` 保护共享数据
3. 优先使用 `std::async` 和 `std::future` 进行异步操作
4. 使用 `std::atomic` 进行简单的无锁线程安全操作
5. 避免数据竞争（data race）——多个线程同时访问同一数据且至少有一个是写操作
6. 使用 `std::condition_variable` 进行线程间通信
7. 确保每个 `std::thread` 在销毁前被 `join` 或 `detach`
8. 优先使用 RAII 锁（`lock_guard`、`unique_lock`）而非手动锁定/解锁
9. 注意死锁（deadlock）——使用 `std::lock` 同时锁定多个互斥锁
10. 并发编程很困难——尽可能使用高层次的抽象
