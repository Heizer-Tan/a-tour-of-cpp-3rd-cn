## 18.1 引言

并发（concurrency）是现代编程中不可或缺的一部分。C++ 标准库提供了丰富的并发支持：

- **线程**（`std::thread`）：操作系统线程的抽象
- **互斥和锁**（`std::mutex`、`std::lock_guard`）：保护共享数据
- **条件变量**（`std::condition_variable`）：线程间通信
- **异步任务**（`std::future`、`std::async`）：启动异步操作并获取结果
- **原子操作**（`std::atomic`）：无锁的线程安全操作

本章将介绍这些工具的基本用法。
