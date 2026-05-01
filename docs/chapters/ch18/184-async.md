## 18.4 异步任务

`std::async` 和 `std::future` 提供了高层次的异步操作接口。

### 18.4.1 `std::async`

```cpp
#include <future>

double compute(int x);

void user()
{
    auto fut = std::async(compute, 42);     // 异步启动 compute(42)
    // ... 做其他事情 ...
    double result = fut.get();              // 获取结果（如果需要则等待）
    cout << result << '\n';
}
```

### 18.4.2 `std::future` 和 `std::promise`

```cpp
void producer(std::promise<int>& prom)
{
    // 计算...
    prom.set_value(42);                     // 设置结果
}

void consumer(std::future<int>& fut)
{
    int result = fut.get();                 // 获取结果
    cout << result << '\n';
}

void user()
{
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();

    std::thread t1 {producer, std::ref(prom)};
    std::thread t2 {consumer, std::ref(fut)};

    t1.join();
    t2.join();
}
```

### 18.4.3 `std::packaged_task`

`packaged_task` 将可调用对象包装为异步操作：

```cpp
std::packaged_task<double(int)> task {compute};
auto fut = task.get_future();
std::thread t {std::move(task), 42};
double result = fut.get();
t.join();
```
