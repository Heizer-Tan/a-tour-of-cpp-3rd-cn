# 16.2 时间

在 `<chrono>` 中，标准库提供了与时间有关的设施：时钟、`time_point` 与 `duration`，用于度量某项操作耗时，并作为一切与时间相关计算的基础；`day`、`month`、`year` 以及工作日，用于把 `time_point` 映射到日常生活；`time_zone` 与 `zoned_time`，用于处理全球各地在时间报告上的差异。几乎所有主要系统都会涉及这些实体中的若干种。

## 16.2.1

度量某项操作耗时的基本写法如下：

```cpp
using namespace std::chrono;         // 位于 std::chrono 子命名空间；见 §3.3

auto t0 = system_clock::now();
do_work();
auto t1 = system_clock::now();
```

时钟返回的是 `time_point`（时间点）。两个 `time_point` 相减得到 `duration`（时间段）。`duration` 默认的 `<<` 会以后缀形式附带所用单位的提示。各种时钟以不同的时间单位给出结果——「时钟滴答」（我所用的时钟以百分之一纳秒为单位度量），因此通常最好把 `duration` 转换成合适的单位，这正是 `duration_cast` 的作用。

时钟便于做快速测量。在没有先做计时测量之前，不要对代码的「效率」下结论。对性能的猜测大多不可靠。快速而简单的测量总比没有测量好，但现代计算机的性能是个棘手话题，因此要小心，别把寥寥几次简单测量看得过重。应反复测量，以降低被罕见事件或缓存效应蒙蔽的概率。

命名空间 `std::chrono_literals` 定义了时间单位后缀（§6.6）。例如可以写出 `200ms`、`24h` 一类字面量。相较无名魔术常量，这类符号名字大大提高可读性，也使代码更易维护。

## 16.2.2

处理日常事件时，我们很少用毫秒；我们用年、月、日、小时、秒以及星期几。标准库对此提供支持。例如：

```cpp
auto spring_day = April/7/2018;
cout << weekday(spring_day) << '\n';                            // Sat
cout << format("{:%A}\n", weekday(spring_day));                 // Saturday

cout << t1 - t0 << "\n";                                        // 默认单位：……
cout << duration_cast<milliseconds>(t1 - t0).count() << "ms\n"; // 指定单位
cout << duration_cast<nanoseconds>(t1 - t0).count() << "ns\n"; // 指定单位
this_thread::sleep_for(10ms + 33us);                            // 等待 10 毫秒加 33 微秒
```

`Sat` 是我这台计算机上星期六的默认字符表示。我不喜欢这种缩写，于是用 `format`（§11.6.2）得到更长名称。出于晦涩的历史原因，`%A` 表示「写出星期几的全名」。自然，`April` 是一个月份；更确切地说，它是 `std::chrono::month`。我们也可以写成：

```cpp
auto spring_day = 2018y/April/7;
```

后缀 `y` 用来把年份与普通 `int`（表示月中从 1 到 31 的日序）区分开。

有可能写出非法日期。若有疑虑，用 `ok()` 检查：

```cpp
auto bad_day = January/0/2024;
if (!bad_day.ok())
    cout << bad_day << " is not a valid day\n";
```

显然，对于通过计算得到的日期，`ok()` 最有用。

日期通过对类型 `year`、`month` 与 `int` 重载运算符 `/`（斜杠）拼合而成。得到的 `year_month_day` 类型可与 `time_point` 相互转换，从而能对日期进行准确而高效的计算；这其中往往需要跨月并正确处理闰年。默认情况下，实现会以 ISO 8601 标准格式给出日期。若要把月份拼写成「March」，就必须拆出日期的各个字段并涉足格式化细节（§11.6.2）。出于晦涩原因，`%B` 表示「写出月份全名」。

此类操作常常可在编译期完成，因而快得出奇：

```cpp
sys_days t = sys_days{February/25/2022};     // 取得某日期的 time_point（精度为……）
t += days{7};                                   // 2022 年 2 月 25 日之后一周
auto d = year_month_day(t);                   // 再把时间点换回日历日期

cout << d << '\n';                            // 2022-03-04
cout << format("{:%B}/{}/{}\n", d.month(), d.day(), d.year()); // March/04/2022

static_assert(weekday(April/7/2018) == Saturday);   // 成立
```

日历复杂而微妙。这对专为「普通人」历经数百年演化而成的「体系」而言很典型，也很恰当——并非程序员为了简化编程而设计。标准库的日历系统可以（并且已经）扩展以应对儒略历、伊斯兰历、泰国历等其他历法。

## 16.2.3

与时间相关的议题里，最难做对之一是时区。它们任意性强，难以记忆，而且会以种种方式在各地并非标准化的时刻发生变化。

`time_zone` 表示相对于某种标准（常称为 GMT 或 UTC）的时间偏移，供 `system_clock` 使用。标准库与全球数据库（IANA）同步以给出正确结果；同步可以由操作系统自动完成，也可由系统管理员控制。时区名字是 C 风格字符串，形如「洲／主要城市」，例如 `"America/New_York"`、`"Asia/Tokyo"`、`"Africa/Nairobi"`。`zoned_time` 则是 `time_zone` 与一个 `time_point` 的组合。

与日历一样，时区涉及我们应交给标准库、而非靠自己手写代码去对付的一组关注点。想一想：2024 年 2 月最后一天纽约当地的什么时间，新德里的日期会更替？2020 年美国科罗拉多州丹佛的夏令时何时结束？下一次闰秒何时发生？标准库「知道」这些答案。

```cpp
auto tp = system_clock::now();              // tp 为 time_point
cout << tp << '\n';                         // 例如：2021-11-27 21:36:08.2085095

zoned_time ztp{current_zone(), tp};        // 例如：2021-11-27 16:36:08.2085095 EST
cout << ztp << '\n';

const time_zone est{"Europe/Copenhagen"};
cout << zoned_time{&est, tp} << '\n';      // 例如：2021-11-27 22:36:08.2085095 GMT+……
```
