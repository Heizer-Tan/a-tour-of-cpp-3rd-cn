## 16.2 时间

`std::chrono` 库提供了精确的时间处理功能。

### 16.2.1 时间点和时间间隔

```cpp
#include <chrono>
using namespace std::chrono;

auto start = steady_clock::now();           // 记录开始时间
// ... 执行一些操作 ...
auto end = steady_clock::now();

auto elapsed = duration_cast<milliseconds>(end - start);
cout << "Elapsed: " << elapsed.count() << "ms\n";
```

### 16.2.2 时间字面量

```cpp
using namespace std::chrono_literals;

auto half_sec = 500ms;
auto two_sec = 2s;
auto minute = 1min;
auto hour = 1h;

auto total = 1h + 30min + 15s;              // 时间运算
```

### 16.2.3 休眠

```cpp
std::this_thread::sleep_for(100ms);         // 休眠 100 毫秒
std::this_thread::sleep_until(
    steady_clock::now() + 2s);              // 休眠到 2 秒后
```
