## 18.2 线程

`std::thread` 表示一个执行线程。

### 18.2.1 创建线程

```cpp
#include <thread>

void f(const vector<double>& v, double* res);

void user()
{
    vector<double> v1 = {1, 2, 3, 4};
    vector<double> v2 = {5, 6, 7, 8};
    double res1, res2;

    std::thread t1 {f, std::cref(v1), &res1};  // f(v1, &res1) 在独立线程中执行
    std::thread t2 {f, std::cref(v2), &res2};  // f(v2, &res2) 在独立线程中执行

    t1.join();  // 等待 t1 完成
    t2.join();  // 等待 t2 完成

    cout << res1 << ' ' << res2 << '\n';
}
```

`std::cref` 创建 `const` 引用包装器，因为 `thread` 默认按值传递参数。

### 18.2.2 线程管理

```cpp
std::thread t {some_function};
t.join();       // 等待线程完成
t.detach();     // 分离线程（线程独立运行，不再可 join）

if (t.joinable()) { /* ... */ }  // 检查线程是否可 join
```

每个 `std::thread` 对象在销毁前必须被 `join` 或 `detach`，否则程序会调用 `std::terminate()`。
