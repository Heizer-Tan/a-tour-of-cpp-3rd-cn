# 16.2 时间

在 `<chrono>` 中，标准库提供了处理时间的设施：

- `clock`、`time_point` 和 `duration`，用于测量某个操作花费的时间，并作为与时间相关的任何事物的基础。
- `day`、`month`、`year` 和 `weekday`，用于将 `time_point` 映射到我们的日常生活中。
- `time_zone` 和 `zoned_time`，用于处理全球时间报告的差异。几乎所有主要系统都会处理这些实体中的一部分。

### 16.2.1 时钟

以下是对某个操作进行计时的基本方法：

```cpp
using namespace std::chrono;   // 在子命名空间 std::chrono 中；见 §3.3

auto t0 = system_clock::now();
do_work();
auto t1 = system_clock::now();

cout << t1 - t0 << "\n";                           // 默认单位
cout << duration_cast<milliseconds>(t1-t0).count() << "ms\n";   // 指定单位
cout << duration_cast<nanoseconds>(t1-t0).count() << "ns\n";    // 指定单位
```

===== 第 3 页 =====

时钟返回一个 `time_point`（时间点）。两个 `time_point` 相减得到一个 `duration`（一段时间）。`duration` 的默认 `<<` 会添加一个表示所用单位的后缀。不同的时钟以不同的时间单位（“时钟滴答”）给出结果（我使用的时钟以百纳秒为单位），因此通常最好将 `duration` 转换为适当的单位。这就是 `duration_cast` 的作用。

时钟对于快速测量很有用。不要在没有进行时间测量的情况下对代码的“效率”做出断言。关于性能的猜测是最不可靠的。快速、简单的测量总比没有测量好，但现代计算机的性能是一个棘手的课题，因此我们必须小心，不要过于重视少数简单的测量。始终重复测量，以降低被罕见事件或缓存效应蒙蔽的机会。

命名空间 `std::chrono_literals` 定义了时间单位后缀（§6.6）。例如：

```cpp
this_thread::sleep_for(10ms + 33us);   // 等待 10 毫秒 33 微秒
```

传统的符号名称极大地提高了可读性，并使代码更易于维护。

#### 16.2.2 日历

在处理日常事件时，我们很少使用毫秒；我们使用年、月、日、小时、秒和星期几。标准库支持这一点。例如：

```cpp
auto spring_day = April/7/2018;
cout << weekday(spring_day) << '\n';                // Sat
cout << format("{:%A}\n", weekday(spring_day));     // Saturday
```

===== 第 4 页 =====

`Sat` 是我电脑上星期六的默认字符表示。我不喜欢那个缩写，因此我使用了 `format`（§11.6.2）来获得更长的名称。出于晦涩的原因，`%A` 表示“写出星期几的全名”。自然，`April` 是一个月份；更准确地说，是 `std::chrono::Month`。我们也可以这样说：

```cpp
auto spring_day = 2018y/April/7;
```

后缀 `y` 用于将年份与普通的 `int`（用于表示 1 到 31 的日期）区分开。

可以表达无效的日期。如果有疑问，可以使用 `ok()` 进行检查：

```cpp
auto bad_day = January/0/2024;
if (!bad_day.ok())
    cout << bad_day << " is not a valid day\n";
```

显然，`ok()` 对从计算得到的日期最有用。

日期通过重载 `operator/`（斜杠）与类型 `year`、`month` 和 `int` 来组合。得到的 `year_month_day` 类型与 `time_point` 之间具有转换关系，以允许涉及日期的精确高效计算。例如：

```cpp
sys_days t = sys_days{February/25/2022};        // 获取一个 time_point，精度为 days
t += days{7};                                   // 2022 年 2 月 25 日之后一周
auto d = year_month_day(t);                     // 将 time_point 转换回日历日期

cout << d << '\n';                              // 2022-03-04
cout << format("{:%B}/{}/{}\n", d.month(), d.day(), d.year()); // March/04/2022
```

这个计算需要月份的变化和关于闰年的知识。默认情况下，实现以 ISO 8601 标准格式给出日期。要拼出“March”这样的月份名称，我们必须分解日期的各个字段并深入了解格式细节（§11.6.2）。出于晦涩的原因，`%B` 表示“写出月份的全名”。

这样的操作通常可以在编译时完成，因此出人意料地快：

```cpp
static_assert(weekday(April/7/2018) == Saturday);   // true
```

===== 第 5 页 =====

日历是复杂而微妙的。这对于为“普通人”设计的“系统”来说是典型且合适的，这些系统经过了几百年的演变，而不是由程序员为了简化编程而设计的。标准库的日历系统可以（并且已经）被扩展以处理儒略历、伊斯兰历、泰历和其他历法。

#### 16.2.3 时区

与时间相关的最棘手的问题之一就是时区。它们是如此随意，以至于难以记忆，并且它们的各种变化方式在全球范围内没有标准化。例如：

```cpp
auto tp = system_clock::now();                  // tp 是一个 time_point
cout << tp << '\n';                             // 2021-11-27 21:36:08.2085095

zoned_time ztp { current_zone(), tp };
cout << ztp << '\n';                            // 2021-11-27 16:36:08.2085095 EST

const time_zone est {"Europe/Copenhagen"};
cout << zoned_time{ &est, tp } << '\n';         // 2021-11-27 22:36:08.2085095 GMT+1
```

`time_zone` 是相对于 `system_clock` 使用的标准时间（称为 GMT 或 UTC）的时间。标准库与一个全局数据库（IANA）同步以得到正确的答案。这种同步可以由操作系统自动完成，或者由系统管理员控制。时区的名称是形式为“洲/主要城市”的 C 风格字符串，例如 `"America/New_York"`、`"Asia/Tokyo"`、`"Africa/Nairobi"`。`zoned_time` 是一个 `time_zone` 加上一个 `time_point`。

像日历一样，时区涉及一系列我们应该交给标准库处理的问题，而不是依赖我们自己手工编写的代码。想一想：2024 年 2 月最后一天，纽约的什么时间，新德里的日期会发生改变？2020 年美国科罗拉多州丹佛市的夏令时何时结束？下一个闰秒何时发生？标准库“知道”。

===== 第 6 页 =====
