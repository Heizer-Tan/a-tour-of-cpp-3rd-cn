# 18.5 任务间通信

标准库提供了少量设施，让程序员能在「任务」（可能要并发执行的工作）的概念层级操作，而不必直接落在更低的线程与锁层级：

- **`future` 与 `promise`**：在单独线程启动的任务之间返回值；
- **`packaged_task`**：帮助启动任务并把返回结果的机制衔接起来；
- **`async()`**：以非常类似调用函数的方式启动任务。

这些设施位于 `<future>`。

## 18.5.1

`future` 与 `promise` 的关键在于：它们能在两个任务之间传递值，而**无需显式使用锁**；「系统」会以高效方式实现传递。基本想法很简单：当任务想把值交给另一方时，把值放进 `promise`；实现会让该值出现在对应的 `future` 中，典型情况下由启动任务的一方读取。我们可以把这种关系画成示意图。

若有一个 `future<X>` 名为 `fx`，可以用 `get()` 从中取得类型 `X` 的值：

```cpp
X v = fx.get();   // 如有必要，阻塞等待值被算出
```

若值尚未就绪，线程会阻塞直到就绪。若值无法算出，`get()` 可能抛出异常（来自系统或由 `promise` 传递）。

`promise` 的主要用途是提供简单的「写入」操作（`set_value()` 与 `set_exception()`），与 `future` 的 `get()` 相呼应。「future」与「promise」之名纯属历史遗产；请勿归咎于作者。它们也是双关语的沃土。

若你手握 `promise`，要把类型 `X` 的结果送给 `future`，可以两件事择一：传值，或传异常。例如：

```cpp
void f(promise<X>& px)
{
    try {
        X res;
        // ... 计算 res ...
        px.set_value(res);
    }
    catch (...) {
        px.set_exception(current_exception());   // 把异常传给 future
    }
}
```

`current_exception()` 指代捕获到的异常。

要通过 `future` 处理传来的异常，`get()` 的调用方必须在某处准备好捕获。例如：

```cpp
void g(future<X>& fx)
{
    try {
        X v = fx.get();
        // ... 使用 v ...
    }
    catch (...) {
        // ... 处理错误：某人没能算出 v ...
    }
}
```

若错误不必由 `g()` 本身处理，代码可以极简化为：

```cpp
void g(future<X>& fx)
{
    X v = fx.get();
    // ... 使用 v ...
}
```

此时若 `fx` 关联函数抛出异常，会如同 `g()` 直接调用该函数一样隐式传播给 `g()` 的调用方。

## 18.5.2

如何把 `future` 放进需要结果的任务里，同时把对应的 `promise` 放进应该产出结果的线程？为此提供了 `packaged_task`，用于简化要在线程上运行、并与 future/promise 相关联的任务设置。`packaged_task` 提供包装代码，把任务的返回值或异常写入 `promise`（性质类似 §18.5.1 所示）。若调用 `get_future()`，`packaged_task` 会给你与其 `promise` 相对应的 `future`。

例如，我们可以设置两个任务，各自用标准库的 `accumulate()`（§17.3）累加 `vector<double>` 的一半元素：

```cpp
double accum(double* beg, double* end, double init)
{
    return accumulate(beg, end, init);   // 计算 [beg:end)，初值为 init
}

double comp2(vector<double>& v)
{
    packaged_task<double(double*, double*, double)> pt0{accum};
    packaged_task<double(double*, double*, double)> pt1{accum};

    future<double> f0{pt0.get_future()};
    future<double> f1{pt1.get_future()};

    double* first = &v[0];
    thread t1{move(pt0), first, first + v.size() / 2, 0};
    thread t2{move(pt1), first + v.size() / 2, first + v.size(), 0};
    // ...

    return f0.get() + f1.get();
}
```

`packaged_task` 的模板实参是任务的函数类型（此处为接受一对指针与初值的累加函数），构造函数实参是任务本身（此处为 `accum`）。需要 `move()`，因为 `packaged_task` 不可拷贝。原因在于它是资源句柄：拥有自己的 `promise`，并对其任务可能拥有的资源负有间接责任。

请注意这段代码**没有显式提到锁**：我们能把注意力放在要完成的工作上，而不是通信机制。两个任务会在不同线程运行，因而可能并行。

## 18.5.3

本章采用的思路是我相信最简单却仍很强的一条：**把任务视作可能与别的任务并发运行的函数**。这远非 C++ 标准库唯一支持的模型，但对很广的需求都很好用。更微妙也更棘手的模型（例如依赖共享内存的风格）可以在需要时使用。

要启动可能异步运行的任务，可使用 `async()`：

```cpp
double comp4(vector<double>& v)
{
    if (v.size() < 10'000)                     // 是否值得用并发？
        return accumulate(v.begin(), v.end(), 0.0);

    auto v0 = &v[0];
    auto sz = v.size();

    auto f0 = async(accum, v0, v0 + sz / 4, 0.0);
    auto f1 = async(accum, v0 + sz / 4, v0 + sz / 2, 0.0);
    auto f2 = async(accum, v0 + sz / 2, v0 + sz * 3 / 4, 0.0);
    auto f3 = async(accum, v0 + sz * 3 / 4, v0 + sz, 0.0);

    return f0.get() + f1.get() + f2.get() + f3.get();
}
```

根本上，`async()` 把函数调用的「调用部分」与「取得结果部分」拆开，两者又与任务的实际执行分离。使用 `async()`，你不必琢磨线程与锁；你在思考的是**可能异步算出结果**的任务。

有一条显而易见的限制：**不要**对需要加锁保护共享资源的任务使用 `async()`。用了 `async()`，你甚至不知道会用多少线程——那取决于 `async()` 根据调用时刻的系统资源所做的决策。例如，`async()` 可能在决定线程数量之前检查是否有空闲核心。

根据诸如 `v.size() < 10'000` 之类启发式来猜测计算成本相对于线程启动成本——极其粗糙，也容易在性能上大错特错。不过此处不适合展开如何管理线程的讨论；别把这类估计当成靠谱的准则。

也很少有必要手工并行化诸如 `accumulate()` 这类标准库算法，因为并行算法（例如 `reduce(par_unseq, /*...*/)`）通常做得更好（§17.3.1）。不过上述技巧具有一般性。

还要注意：`async()` 并非只做并行提速——例如也可以用它启动任务向用户索取信息，而让「主程序」同时做别的事（§18.5.3）。

## 18.5.4

有时我们想终止线程，因为我们对它的结果不再感兴趣。直接「杀掉」线程通常无法接受：线程可能拥有必须释放的资源（锁、子线程、数据库连接等）。标准库为此提供了礼貌请求线程清理并退出的机制：**`stop_token`**。若线程持有 `stop_token` 并被请求停止，就可以据此终止。

考虑并行算法 `find_any()`：它启动许多线程寻找结果。当某个线程带着答案返回时，我们希望停止其余线程。`find_any()` 启动的每个线程调用 `find()` 做真正的工作。`find()` 是非常简单的常见任务风格示例——主体循环里可以插入「继续还是停止」的测试。

此处 `!tok.stop_requested()` 判断是否有其它线程请求本线程终止。`stop_token` 是在无数据竞争前提下传达此类请求的机制。

下面是一个极简的 `find_any()`，只启动两个线程运行 `find()`：

```cpp
atomic<int> result = -1;

template<class T>
struct Range {
    T* first;
    T* last;
};

bool match(const string& a, const string& b);   // 对二者套用某种匹配准则（示意）

void find(stop_token tok, const string* base, Range<string> r, const string& target)
{
    for (string* p = r.first; p != r.last && !tok.stop_requested(); ++p)
        if (match(*p, target)) {
            result = static_cast<int>(p - base);
            return;
        }
}

void find_all(vector<string>& vs, const string& key)
{
    int mid = static_cast<int>(vs.size() / 2);
    string* pvs = &vs[0];

    stop_source ss1{};
    jthread t1(find, ss1.get_token(), pvs, Range<string>{pvs, pvs + mid}, key);

    stop_source ss2{};
    jthread t2(find, ss2.get_token(), pvs, Range<string>{pvs + mid, pvs + vs.size()}, key);

    while (result == -1)
        this_thread::sleep_for(10ms);

    ss1.request_stop();   // 已有结果：请求线程协作退出
    ss2.request_stop();

    // ... 使用 result ...
}
```

`stop_source` 产生 `stop_token`，藉此把停止请求传达给线程。

同步与返回结果我用了最简单的想法之一：把结果放进原子变量（§18.3.2），并对其自旋等待。

当然可以把示例扩展成更多搜索线程、更一般的结果返回、以及不同元素类型——但那会掩盖 `stop_source` 与 `stop_token` 的基本角色。
