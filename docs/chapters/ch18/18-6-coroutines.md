
**协程**（coroutine）是一种在调用之间保持其状态的函数。在这一点上，它有点像函数对象，但是在调用之间保存和恢复其状态是隐式且完整的。考虑一个经典示例：

```cpp
generator<long long> fib()   // 生成斐波那契数
{
    long long a = 0;
    long long b = 1;
    while (a < b) {
        auto next = a + b;
        co_yield next;   // 保存状态，返回值，然后等待
        a = b;
        b = next;
    }
    co_return 0;   // 斐波那契数太大了
}

void user(int max)
{
    for (int i = 0; i++ < max;)
        cout << fib() << ' ';
}
```

这会生成：
```
1 2 3 5 8 13 ...
```

===== 第 20 页 =====

`generator` 的返回值是协程在调用之间存储其状态的地方。当然，我们可以编写一个以相同方式工作的函数对象 `Fib`，但那样我们必须自己维护其状态。对于更大的状态和更复杂的计算，保存和恢复状态会变得繁琐、难以优化且容易出错。实际上，协程是一个在调用之间保存其栈帧的函数。`co_yield` 返回一个值并等待下一次调用。`co_return` 返回一个值并终止协程。

协程可以是同步的（调用者等待结果）或异步的（调用者做其他工作，直到它从协程中寻找结果）。斐波那契示例显然是同步的。这允许一些很好的优化。例如，一个好的优化器可以内联对 `fib()` 的调用并展开循环，只留下一系列 `<<` 调用，这些调用本身可以被优化为：

```cpp
cout << "1 2 3 5 7 12";   // fib(6)
```

协程被实现为一个极其灵活的框架，能够服务于极其广泛的潜在用途。它由专家设计，为了专家，并带有一丝委员会设计的味道。这很好，除了使简单使用变得简单的库设施在 C++20 中仍然缺失。例如，`generator` 还不是标准库的一部分。不过，有一些提案，通过 Web 搜索可以找到好的实现；[Cppcoro] 是一个例子。

### 18.6.1 协作式多任务

在《计算机程序设计艺术》的第一卷中，Donald Knuth 赞扬了协程的实用性，但也感叹很难给出简短的例子，因为协程在简化复杂系统方面最有用。这里，我将只给出一个简单的例子，以练习事件驱动仿真所需的基本元素，这是 C++ 早期成功的主要原因之一。关键思想是将一个系统表示为一个简单的任务（协程）网络，这些任务协作完成复杂的任务。基本上，每个任务都是一个执行大型工作中一小部分的参与者。有些是生成器，产生请求流（可能使用随机数生成器，可能输入真实世界的数据），有些是计算结果的网络的一部分，有些则产生输出。我个人更喜欢任务（协程）通过消息队列进行通信。组织这样一个系统的一种方式是让每个任务在产生一个结果后将自身放入一个事件队列中，等待更多的工作。然后，调度器在需要时从事件队列中选择下一个要运行的任务。这是一种协作式多任务的形式。我从 Simula [Dahl,1970] 中借用了关键思想（并致谢），形成了第一个 C++ 库的基础（§19.1.2）。

此类设计的关键在于：

- 许多不同的协程，它们在调用之间保持自己的状态。
- 一种多态形式，允许我们保持包含不同类型协程的事件列表，并独立于它们的类型调用它们。
- 一个调度器，从列表中选择下一个要运行的协程。

这里，我将只展示两个协程并交替执行它们。对于此类系统，不使用太多空间是至关重要的。这就是为什么我们不为此类应用使用进程或线程。一个线程需要一兆或两兆字节（主要是其栈），而一个协程通常只需要几十个字节。如果你需要成千上万个任务，这可能会产生巨大差异。协程之间的上下文切换也远比线程或进程之间快得多。

首先，我们需要一些运行时多态性，以便能够统一调用几十或几百种不同类型的协程：

```cpp
struct Event_base {
    virtual void operator()() = 0;
    virtual ~Event_base() {}
};

template<class Act>
struct Event : Event_base {
    Event(const string n, Act a) : name{n}, act{move(a)} { }
    string name;
    Act act;
    void operator()() override { act(); }
};
```

===== 第 22 页 =====

`Event` 只是存储一个动作并允许它被调用；该动作通常是一个协程。我添加了一个 `name`，只是为了说明一个事件通常携带的信息不仅仅是协程句柄。

这是一个简单的使用示例：

```cpp
void test()
{
    vector<Event_base*> events = {   // 创建几个事件
        new Event{ "integers ", sequencer(10) },
        new Event{ "chars ", char_seq('a') }
    };

    vector order {0, 1, 1, 0, 1, 0, 1, 0, 0};   // 选择某种顺序

    for (int x : order)   // 按顺序调用协程
        (*events[x])();

    for (auto p : events)   // 清理
        delete p;
}
```

到目前为止，这还没有任何特定于协程的内容；它只是一个传统的面向对象框架，用于在一组可能具有不同类型的对象上执行操作。然而，`sequencer` 和 `char_seq` 恰好是协程。它们在调用之间保持状态的事实对于此类框架在真实世界中的使用至关重要：

```cpp
task sequencer(int start, int step = 1)
{
    auto value = start;
    while (true) {
        cout << "value: " << value << '\n';   // 传递结果
        co_yield 0;                           // 暂停，直到有人恢复此协程
        value += step;                        // 更新状态
    }
}
```

===== 第 23 页 =====

我们可以看到 `sequencer` 是一个协程，因为它使用了 `co_yield` 在调用之间暂停自身。这意味着 `task` 必须是一个协程句柄（见下文）。

这是一个故意设计的简单协程。它所做的只是生成一个值序列并输出它们。在一个严肃的仿真中，该输出将直接或间接地成为另一个协程的输入。

`char_seq` 非常相似，但是不同的类型，以练习运行时多态性：

```cpp
task char_seq(char start)
{
    auto value = start;
    while (true) {
        cout << "value: " << value << '\n';   // 传递结果
        co_yield 0;
        ++value;
    }
}
```

“魔法”在于返回类型 `task`；它在调用之间保存协程的状态（实际上是函数的栈帧），并决定 `co_yield` 的含义。从用户的角度来看，`task` 很简单，它只是提供了一个调用协程的运算符：

```cpp
struct task {
    void operator()();
    // ... 实现细节 ...
}
```

如果 `task` 已经在一个库中（最好是标准库），那就是我们需要知道的一切，但它不是，因此这里提示一下如何实现这样的协程句柄类型。不过，有一些提案，通过 Web 搜索可以找到好的实现；[Cppcoro] 库就是一个例子。

我的 `task` 是我能想到的最小的实现，以实现我的关键示例：

```cpp
struct task {
    struct promise_type {   // 映射到语言机制
        suspend_always initial_suspend() { return {}; }
        suspend_always final_suspend() noexcept { return {}; }   // co_return
        suspend_always yield_value(int) { return {}; }            // co_yield
        auto get_return_object() { return task{ handle_type::from_promise(*this) }; }
        void return_void() {}
        void unhandled_exception() { exit(1); }
    };

    using handle_type = coroutine_handle<promise_type>;
    task(handle_type h) : coro(h) { }   // 由 get_return_object() 调用
    handle_type coro;                   // 这里是协程句柄
};
```

===== 第 24 页 =====

```cpp
struct task {
    void operator()() { coro.resume(); }
    // ...
};
```

我强烈建议你不要自己编写这样的代码，除非你是一个试图为他人省去麻烦的库实现者。如果你好奇，网上有很多解释。

## 18.7 建议