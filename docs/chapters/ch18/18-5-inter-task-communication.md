
标准库提供了一些设施，允许程序员在任务的（可并发执行的工作）概念层面上进行操作，而不是直接在线程和锁的较低层面上操作：

- `future` 和 `promise`：用于从在单独线程上生成的任务返回值
- `packaged_task`：帮助启动任务并连接返回结果的机制
- `async()`：以一种与调用函数非常相似的方式启动任务

这些设施位于 `<future>` 中。

===== 第 13 页 =====

#### 18.5.1 future 与 promise

关于 `future` 和 `promise` 的重要之处在于，它们可以在两个任务之间传递一个值，而无需显式使用锁；“系统”高效地实现了这种传递。基本思想很简单：当一个任务想要向另一个任务传递一个值时，它会将该值放入一个 `promise` 中。不知何故，实现会使得该值出现在对应的 `future` 中，然后可以从 `future` 中读取该值（通常由任务的启动者读取）。我们可以用图形表示：

[图片描述：promise 和 future 示意图]

如果我们有一个名为 `fx` 的 `future<X>`，我们可以从中 `get()` 一个 `X` 类型的值：

```cpp
X v = fx.get();   // 如果需要，等待值被计算出来
```

如果值还没有准备好，我们的线程会被阻塞，直到值到达。如果值无法计算，`get()` 可能会抛出一个异常（来自系统或从 `promise` 传递过来的）。

`promise` 的主要目的是提供简单的“放入”操作（称为 `set_value()` 和 `set_exception()`）以匹配 `future` 的 `get()`。“future”和“promise”这些名称是历史产物；请不要归功或归咎于我。它们是双关语的又一丰富来源。

如果你有一个 `promise` 并且需要将一个 `X` 类型的结果发送给一个 `future`，你可以做两件事之一：传递一个值或传递一个异常。例如：

```cpp
void f(promise<X>& px)   // 任务：将结果放入 px
{
    // ...
    try {
        X res;
        // ... 为 res 计算一个值 ...
        px.set_value(res);
    }
    catch (...) {   // 糟糕：无法计算 res
        px.set_exception(current_exception());   // 将异常传递给 future
    }
}
```

===== 第 14 页 =====

`current_exception()` 引用被捕获的异常。

为了处理通过 `future` 传递的异常，`get()` 的调用者必须准备在某个地方捕获它。例如：

```cpp
void g(future<X>& fx)   // 任务：从 fx 获取结果
{
    // ...
    try {
        X v = fx.get();   // 如果需要，等待值被计算出来
        // ... 使用 v ...
    }
    catch (...) {   // 糟糕：有人无法计算 v
        // ... 处理错误 ...
    }
}
```

如果错误不需要由 `g()` 本身处理，代码可以简化为最少形式：

```cpp
void g(future<X>& fx)   // 任务：从 fx 获取结果
{
    // ...
    X v = fx.get();   // 如果需要，等待值被计算出来
    // ... 使用 v ...
}
```

现在，从 `fx` 的函数（`f()`）抛出的异常会隐式地传播到 `g()` 的调用者，就像 `g()` 直接调用 `f()` 一样。

#### 18.5.2 packaged_task

我们如何将 `future` 传递给需要结果的任务，并将对应的 `promise` 传递给应该产生该结果的线程？标准库提供了 `packaged_task` 类型来简化设置与 `future` 和 `promise` 连接的任务以在线程上运行。`packaged_task` 提供包装代码，将任务的返回值或异常放入 `promise` 中（就像 §18.5.1 中显示的代码）。如果你通过调用 `get_future()` 来询问，`packaged_task` 会给你对应于其 `promise` 的 `future`。例如，我们可以设置两个任务，每个任务使用标准库的 `accumulate()`（§17.3）计算一个 `vector<double>` 的一半元素的和：

```cpp
double accum(vector<double>::iterator beg, vector<double>::iterator end, double init)
// 计算 [beg:end) 的和，从初始值 init 开始
{
    return accumulate(&*beg, &*end, init);
}

double comp2(vector<double>& v)
{
    packaged_task pt0 {accum};   // 包装任务
    packaged_task pt1 {accum};

    future<double> f0 {pt0.get_future()};   // 获取 pt0 的 future
    future<double> f1 {pt1.get_future()};   // 获取 pt1 的 future

    double* first = &v[0];

    jthread t1 {move(pt0), first, first + v.size()/2, 0};   // 为 pt0 启动线程
    jthread t2 {move(pt1), first + v.size()/2, first + v.size(), 0};   // 为 pt1 启动线程

    return f0.get() + f1.get();   // 获取结果
}
```

===== 第 15 页 =====

`packaged_task` 模板以任务的类型作为其模板参数（此处为 `double(double*, double*, double)`），并以任务作为其构造函数参数（此处为 `accum`）。需要 `move()` 操作，因为 `packaged_task` 不能被拷贝。`packaged_task` 不能被拷贝的原因是它是一个资源句柄：它拥有其 `promise`，并（间接地）对其任务可能拥有的任何资源负责。

请注意，这段代码中没有显式提到锁：我们能够专注于要完成的任务，而不是用于管理它们通信的机制。这两个任务将在单独的线程上运行，因此有可能并行运行。

#### 18.5.3 async()

我在本章中坚持的思考方式是我认为最简单但仍然最强大的方式之一：将一个任务视为一个可能与其他任务并发运行的函数。这远非 C++ 标准库支持的唯一模型，但它能满足广泛的需求。更微妙、更棘手的模型（例如，依赖共享内存的编程风格）可以在需要时使用。

为了启动可能异步运行的任务，我们可以使用 `async()`：

```cpp
double comp4(vector<double>& v)   // 如果 v 足够大，则生成许多任务
{
    if (v.size() < 10'000)        // 值得使用并发吗？
        return accum(v.begin(), v.end(), 0.0);

    auto v0 = &v[0];
    auto sz = v.size();

    auto f0 = async(accum, v0, v0 + sz/4, 0.0);          // 第一段
    auto f1 = async(accum, v0 + sz/4, v0 + sz/2, 0.0);   // 第二段
    auto f2 = async(accum, v0 + sz/2, v0 + sz*3/4, 0.0); // 第三段
    auto f3 = async(accum, v0 + sz*3/4, v0 + sz, 0.0);   // 第四段

    return f0.get() + f1.get() + f2.get() + f3.get();    // 收集并组合结果
}
```

基本上，`async()` 将函数调用的“调用部分”与“获取结果部分”分离开来，并将两者与实际执行任务分离开。使用 `async()`，你不需要考虑线程和锁。相反，你从可能异步计算结果的任务的角度思考。有一个明显的限制：甚至不要考虑将 `async()` 用于需要锁来共享资源的任务。使用 `async()` 时，你甚至不知道将使用多少线程，因为这由 `async()` 根据调用时它对可用系统资源的了解来决定。例如，`async()` 可能会在决定使用多少线程之前检查是否有空闲内核（处理器）可用。

===== 第 17 页 =====

使用对计算成本相对于启动线程成本的猜测（例如 `v.size() < 10'000`）是非常原始的，并且容易在性能方面犯大错误。然而，这里不是讨论如何管理线程的合适场合。不要将这个估计视为比一个简单的、可能很差的猜测更多的东西。

很少需要手动并行化标准库算法（如 `accumulate()`），因为并行算法（例如 `reduce(par_unseq, /*...*/)` 通常在这方面做得更好（§17.3.1）。然而，这种技术是通用的。

请注意，`async()` 不仅仅是一种专门为提高性能而进行并行计算的机制。例如，它也可以用于生成一个从用户获取信息的任务，而让“主程序”忙于其他事情。

#### 18.5.4 停止线程

有时，我们想要停止一个线程，因为我们不再对其结果感兴趣。仅仅“杀死”它通常是不可接受的，因为一个线程可能拥有必须释放的资源（例如，锁、子线程和数据库连接）。相反，标准库提供了一种礼貌地请求线程清理并退出的机制：`stop_token`。一个线程可以被编程为，如果它有一个 `stop_token` 并被请求停止，则终止。

考虑一个并行算法 `find_any()`，它生成许多线程来寻找结果。当一个线程返回一个答案时，我们希望停止其余的线程。每个由 `find_any()` 生成的线程调用 `find()` 来完成真正的工作。这个 `find()` 是一个非常简单的例子，代表了一种常见的任务风格，其中有一个主循环，我们可以在其中插入一个测试来决定是继续还是停止：

```cpp
atomic<int> result = -1;   // 放置结果索引的位置

template<class T>
struct Range { T* first; T* last; };   // 传递范围的一种方式

void find(stop_token tok, const string* base, const Range<string> r, const string& target)
{
    for (string* p = r.first; p != r.last && !tok.stop_requested(); ++p)
        if (match(*p, target)) {   // match() 将某种匹配条件应用于两个字符串
            result = p - base;     // 找到的元素的索引
            return;
        }
}
```

这里，`!tok.stop_requested()` 测试是否有其他线程请求此线程终止。`stop_token` 是安全（无数据竞争）地传达此类请求的机制。

这是一个简单的 `find_any()`，它只生成两个运行 `find()` 的线程：

```cpp
void find_all(vector<string>& vs, const string& key)
{
    int mid = vs.size() / 2;
    string* pvs = &vs[0];

    stop_source ss1{};
    jthread t1(find, ss1.get_token(), pvs, Range{pvs, pvs + mid}, key);

    stop_source ss2{};
    jthread t2(find, ss2.get_token(), pvs, Range{pvs + mid, pvs + vs.size()}, key);

    while (result == -1)
        this_thread::sleep_for(10ms);

    ss1.request_stop();   // 我们得到了结果：停止所有线程
    ss2.request_stop();

    // ... 使用 result ...
}
```

===== 第 19 页 =====

`stop_source` 产生 `stop_token`，通过它向线程传达停止请求。

同步和返回结果的方式是我能想到的最简单的方式：将结果放在一个原子变量（§18.3.2）中，并在其上执行自旋循环。

当然，我们可以将这个简单的例子扩展为使用许多搜索线程，使结果的返回更通用，并使用不同的元素类型。然而，那样会掩盖 `stop_source` 和 `stop_token` 的基本作用。

## 18.6 协程