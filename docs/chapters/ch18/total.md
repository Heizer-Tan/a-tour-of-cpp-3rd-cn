===== 第 1 页 =====

18

# 并发

保持简单：尽可能简单，但不要更简单。 – A. 爱因斯坦

- 引言
- 任务与线程
  - 传递参数；返回结果
- 共享数据
  - 互斥量与锁；原子操作
- 等待事件
- 任务间通信
  - future 与 promise；packaged_task；async()；停止线程
- 协程

===== 第 2 页 =====

### 18.1 引言

**并发**——同时执行多个任务——被广泛用于提高吞吐量（通过使用多个处理器进行单一计算）或提高响应性（允许程序的一部分在执行的同时，另一部分等待响应）。所有现代编程语言都为此提供支持。C++ 标准库提供的支持是 C++ 中已使用超过 20 年的可移植、类型安全的变体，并且几乎得到现代硬件的普遍支持。标准库的支持主要面向系统级并发，而不是直接提供复杂的高级并发模型；这些高级模型可以作为库使用标准库设施构建。

标准库直接支持单个地址空间内多个线程的并发执行。为此，C++ 提供了合适的内存模型和一组原子操作。原子操作允许无锁编程 [Dechev,2010]。内存模型确保只要程序员避免数据竞争（对可变数据的无控制并发访问），一切都会如人们天真的预期那样工作。然而，大多数用户只会从标准库及其之上构建的库的角度看待并发。本节简要介绍主要的标准库并发支持设施的示例：线程、互斥量、`lock()` 操作、`packaged_task` 和 `future`。这些特性直接构建在操作系统提供的基础之上，并且与它们相比没有性能损失。它们也不保证相比操作系统提供显著的性能提升。

不要将并发视为万能药。如果一个任务可以顺序完成，那么顺序完成通常更简单、更快。将信息从一个线程传递到另一个线程可能会出奇地昂贵。

作为使用显式并发特性的替代方案，我们通常可以使用并行算法来利用多个执行引擎以获得更好的性能（§13.6，§17.3.1）。

===== 第 3 页 =====

最后，C++ 支持**协程**；即在调用之间保持状态的函数（§18.6）。

## 18.2 任务与线程

我们将可能与其他计算并发执行的计算称为**任务**。**线程**是程序中任务的系统级表示。通过构造一个以任务为参数的 `thread`（位于 `<thread>` 中）来启动一个与其他任务并发执行的任务。任务可以是一个函数或一个函数对象：

```cpp
void f();               // 函数

struct F {              // 函数对象
    void operator()();  // F 的调用运算符（§7.3.2）
};

void user()
{
    thread t1 {f};      // f() 在单独的线程中执行
    thread t2 {F{}};    // F{}() 在单独的线程中执行

    t1.join();          // 等待 t1
    t2.join();          // 等待 t2
}
```

`join()` 确保在线程完成之前不会退出 `user()`。“加入”一个线程意味着“等待该线程终止”。

很容易忘记调用 `join()`，结果通常很糟糕，因此标准库提供了 `jthread`，这是一个“可加入的线程”，它通过其析构函数调用 `join()` 来遵循 RAII：

```cpp
void user()
{
    jthread t1 {f};     // f() 在单独的线程中执行
    jthread t2 {F{}};   // F{}() 在单独的线程中执行
}
```

===== 第 4 页 =====

析构函数完成加入操作，因此顺序与构造相反。这里，我们在 `t1` 之前等待 `t2`。

一个程序中的线程共享单个地址空间。在这方面，线程不同于通常不直接共享数据的进程。由于线程共享地址空间，它们可以通过共享对象进行通信（§18.3）。这种通信通常由锁或其他机制控制，以防止数据竞争（对变量的无控制并发访问）。

编程并发任务可能非常棘手。考虑任务 `f`（函数）和 `F`（函数对象）的可能实现：

```cpp
void f()
{
    cout << "Hello ";
}

struct F {
    void operator()() { cout << "Parallel World!\n"; }
};
```

这是一个严重错误的例子：这里，`f` 和 `F{}` 都使用 `cout` 对象，没有任何形式的同步。结果的输出将是不可预测的，可能在程序的不同执行之间变化，因为两个任务中各个操作的执行顺序是未定义的。程序可能产生“奇怪的”输出，例如：
`PaHerallllel o World!`

标准中一个特定的保证使我们免于 `ostream` 定义内的数据竞争（该竞争可能导致崩溃）。

为了避免输出流的此类问题，要么只让一个线程使用流，要么使用 `osyncstream`（§11.7.5）。

在定义并发程序的任务时，我们的目标是让任务完全分离，除非它们以简单明了的方式进行通信。考虑并发任务的最简单方式是将其视为一个恰好与其调用者并发运行的函数。为此，我们只需要传递参数、获取返回结果，并确保之间没有共享数据的使用（没有数据竞争）。

#### 18.2.1 传递参数

通常，任务需要操作的数据。我们可以轻松地将数据（或指向数据的指针或引用）作为参数传递。考虑：

```cpp
void f(vector<double>& v);               // 函数：对 v 进行操作

struct F {                               // 函数对象：对 v 进行操作
    vector<double>& v;
    F(vector<double>& vv) : v{vv} { }
    void operator()();                   // 应用运算符；§7.3.2
};

int main()
{
    vector<double> some_vec {1, 2, 3, 4, 5, 6, 7, 8, 9};
    vector<double> vec2 {10, 11, 12, 13, 14};

    jthread t1 {f, ref(some_vec)};       // f(some_vec) 在单独的线程中执行
    jthread t2 {F{vec2}};                // F(vec2)() 在单独的线程中执行
}
```

`F{vec2}` 将对参数向量的引用保存在 `F` 中。`F` 现在可以使用该向量，并且希望没有其他任务在 `F` 执行时访问 `vec2`。按值传递 `vec2` 将消除该风险。

使用 `{f, ref(some_vec)}` 进行初始化时，使用了线程的变参模板构造函数，它可以接受任意序列的参数（§8.4）。`ref()` 是来自 `<functional>` 的类型函数，不幸的是，它需要告诉变参模板将 `some_vec` 视为引用，而不是对象。如果没有 `ref()`，`some_vec` 将按值传递。编译器会检查在给定后续参数的情况下第一个参数是否可以被调用，并构建必要的函数对象传递给线程。因此，如果 `F::operator()()` 和 `f()` 执行相同的算法，那么对这两个任务的处理大致相当：在两种情况下，都会为线程构造一个函数对象来执行。

#### 18.2.2 返回结果

===== 第 6 页 =====

在 §18.2.1 的示例中，我通过非 `const` 引用传递参数。只有当我期望任务修改所引用数据的值时，我才这样做（§1.7）。这是一种有点隐蔽但并非不常见的返回结果的方式。一种不太晦涩的技术是通过 `const` 引用传递输入数据，并将存放结果的位置作为单独的参数传递：

```cpp
void f(const vector<double>& v, double* res);   // 从 v 获取输入；将结果放在 *res 中

class F {
public:
    F(const vector<double>& vv, double* p) : v{vv}, res{p} { }
    void operator()();                          // 将结果放在 *res 中
private:
    const vector<double>& v;                    // 输入源
    double* res;                                // 输出目标
};

double g(const vector<double>&);                // 使用返回值

void user(vector<double>& vec1, vector<double> vec2, vector<double> vec3)
{
    double res1;
    double res2;
    double res3;

    jthread t1 {f, cref(vec1), &res1};          // f(vec1, &res1) 在单独的线程中执行
    jthread t2 {F{vec2, &res2}};                // F{vec2, &res2}() 在单独的线程中执行
    jthread t3 { [&]() { res3 = g(vec3); } };   // 通过引用捕获局部变量

    t1.join();   // 在使用结果之前加入
    t2.join();
    t3.join();

    cout << res1 << ' ' << res2 << ' ' << res3 << '\n';
}
```

这里，`cref(vec1)` 将 `vec1` 的 `const` 引用作为参数传递给 `t1`。

这种方法可行且非常常见，但我认为通过引用返回结果并不是特别优雅，因此我在 §18.5.1 中会回到这个话题。

## 18.3 共享数据

有时任务需要共享数据。在这种情况下，必须同步访问，以便一次最多只有一个任务可以访问。有经验的程序员会认识到这是一种简化（例如，许多任务同时读取不可变数据没有问题），但考虑如何确保一次最多只有一个任务可以访问一组给定的对象。

#### 18.3.1 互斥量与锁

**互斥量**（mutex，即“互斥对象”）是线程之间共享数据的关键元素。线程使用 `lock()` 操作获取互斥量：

```cpp
mutex m;      // 控制互斥量
int sh;       // 共享数据

void f()
{
    scoped_lock lck {m};   // 获取互斥量
    sh += 7;               // 操作共享数据
}   // 隐式释放互斥量
```

`lck` 的类型被推导为 `scoped_lock<mutex>`（§7.2.3）。`scoped_lock` 的构造函数获取互斥量（通过调用 `m.lock()`）。如果另一个线程已经获取了该互斥量，当前线程会等待（“阻塞”），直到另一个线程完成其访问。一旦某个线程完成对共享数据的访问，`scoped_lock` 释放互斥量（通过调用 `m.unlock()`）。当互斥量被释放时，等待它的线程恢复执行（“被唤醒”）。互斥和锁定设施位于 `<mutex>` 中。

注意使用 RAII（§6.3）。使用资源句柄（如 `scoped_lock` 和 `unique_lock`（§18.4））比显式锁定和解锁互斥量更简单、更安全。

===== 第 8 页 =====

共享数据与互斥量之间的对应关系依赖于约定：程序员必须知道哪个互斥量应该对应哪个数据。显然，这容易出错，同样明显的是，我们试图通过各种语言手段使这种对应关系清晰。例如：

```cpp
class Record {
public:
    mutex rm;
    // ...
};
```

不难猜到，对于一个名为 `rec` 的 `Record`，你应当在访问 `rec` 的其余部分之前获取 `rec.rm`，尽管注释或更好的名称可能对读者更有帮助。

有时需要同时访问多个资源以执行某个操作。这可能导致**死锁**。例如，如果线程 1 获取了 `mutex1` 然后试图获取 `mutex2`，而线程 2 获取了 `mutex2` 然后试图获取 `mutex1`，那么这两个任务都将永远无法继续。`scoped_lock` 通过允许我们同时获取多个锁来提供帮助：

```cpp
void f()
{
    scoped_lock lck {mutex1, mutex2, mutex3};   // 获取所有三个锁
    // ... 操作共享数据 ...
}   // 隐式释放所有互斥量
```

这个 `scoped_lock` 只有在获取了其所有互斥量参数后才会继续，并且不会在持有互斥量时阻塞（“进入睡眠”）。`scoped_lock` 的析构函数确保线程离开作用域时释放互斥量。

通过共享数据进行通信是非常低级的。特别是，程序员必须设计出方法来了解各个任务已经完成和尚未完成哪些工作。在这方面，使用共享数据不如调用和返回的概念。另一方面，有些人认为共享一定比拷贝参数和返回值更高效。在处理大量数据时确实可能如此，但锁定和解锁是相对昂贵的操作。另一方面，现代机器非常擅长拷贝数据，尤其是紧凑的数据，例如 `vector` 的元素。因此，不要因为“效率”的考虑而在没有思考的情况下选择共享数据进行通信，最好是在没有测量的情况下也不要选择。

基本的互斥量一次只允许一个线程访问数据。最常见的共享数据方式之一是多个读取者和单个写入者。这种“读者-写者锁”惯用法由 `shared_mutex` 支持。读取者将“共享”地获取互斥量，以便其他读取者仍然可以访问，而写入者则需要独占访问。例如：

```cpp
shared_mutex mx;   // 一个可以共享的互斥量

void reader()
{
    shared_lock lck {mx};   // 愿意与其他读取者共享访问
    // ... 读取 ...
}

void writer()
{
    unique_lock lck {mx};   // 需要独占（唯一）访问
    // ... 写入 ...
}
```

#### 18.3.2 原子操作

互斥量是一个相当重量级的机制，涉及操作系统。它允许在无数据竞争的情况下完成任意数量的工作。然而，有一种更简单、更便宜的机制来处理少量工作：**原子变量**。例如，这里是一个经典的双检查锁定的简单变体：

```cpp
mutex mut;
atomic<bool> init_x;   // 初始为 false
X x;                   // 需要非平凡初始化的变量

if (!init_x) {
    lock_guard lck {mut};
    if (!init_x) {
        // ... 对 x 进行非平凡初始化 ...
        init_x = true;
    }
}

// ... 使用 x ...
```

===== 第 10 页 =====

原子操作使我们免除了大多数使用昂贵互斥量的情况。如果 `init_x` 不是原子类型，那么初始化会非常罕见地失败，导致神秘且难以发现的错误，因为 `init_x` 上会发生数据竞争。

这里，我使用了 `lock_guard` 而不是 `scoped_lock`，因为我只需要一个互斥量，所以最简单的锁（`lock_guard`）就足够了。

## 18.4 等待事件

有时，线程需要等待某种外部事件，例如另一个线程完成一个任务或已经过去一段时间。最简单的“事件”仅仅是时间的流逝。使用 `<chrono>` 中的时间设施，我可以写：

```cpp
using namespace chrono;   // 见 §16.2.1

auto t0 = high_resolution_clock::now();
this_thread::sleep_for(milliseconds{20});
auto t1 = high_resolution_clock::now();

cout << duration_cast<nanoseconds>(t1 - t0).count() << " nanoseconds passed\n";
```

我甚至不需要启动一个线程；默认情况下，`this_thread` 可以指代唯一的那一个线程。

我使用 `duration_cast` 将时钟的单位调整为我想要的纳秒。

使用外部事件进行通信的基本支持由 `<condition_variable>` 中的 `condition_variable` 提供。`condition_variable` 是一种允许一个线程等待另一个线程的机制。特别地，它允许一个线程等待某个条件（通常称为**事件**）发生，该条件是其他线程工作的结果。

===== 第 11 页 =====

使用 `condition_variable` 支持多种优雅高效的共享形式，但可能相当棘手。考虑两个线程通过队列传递消息进行通信的经典示例。为简单起见，我将队列以及避免该队列上竞争条件的机制声明为生产者和消费者的全局变量：

```cpp
class Message {   // 要通信的对象
    // ...
};

queue<Message> mqueue;          // 消息队列
condition_variable mcond;       // 通信事件的变量
mutex mmutex;                   // 用于同步对 mcond 的访问
```

类型 `queue`、`condition_variable` 和 `mutex` 由标准库提供。

`consumer()` 读取并处理 `Message`：

```cpp
void consumer()
{
    while (true) {
        unique_lock lck {mmutex};                     // 获取 mmutex
        mcond.wait(lck, [] { return !mqueue.empty(); }); // 释放 mmutex 并等待，直到队列非空
        auto m = mqueue.front();                      // 获取消息
        mqueue.pop();
        lck.unlock();                                 // 释放 mmutex
        // ... 处理 m ...
    }
}
```

这里，我使用 `unique_lock` 显式地保护对队列和 `condition_variable` 的操作。等待 `condition_variable` 会释放其锁参数，直到等待结束（以便队列变为非空），然后重新获取它。显式检查条件（此处为 `!mqueue.empty()`）可以防止在唤醒后发现其他任务“抢先一步”从而导致条件不再满足的情况。

我使用 `unique_lock` 而非 `scoped_lock` 有两个原因：

1. 我们需要将锁传递给 `condition_variable` 的 `wait()`。`scoped_lock` 不可移动，但 `unique_lock` 可以。
2. 我们希望在处理消息之前解锁保护条件变量的互斥量。`unique_lock` 提供了诸如 `lock()` 和 `unlock()` 之类的操作，用于低级别的同步控制。

另一方面，`unique_lock` 只能处理单个互斥量。

相应的生产者看起来像这样：

```cpp
void producer()
{
    while (true) {
        Message m;
        // ... 填充消息 ...
        scoped_lock lck {mmutex};   // 保护操作
        mqueue.push(m);
        mcond.notify_one();         // 通知
    }   // 释放 mmutex（在作用域结束时）
}
```

## 18.5 任务间通信

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

[1] 使用并发来提高响应性或提高吞吐量；§18.1。
[2] 在你能够承受的最高抽象级别上工作；§18.1。
[3] 考虑将进程作为线程的替代方案；§18.1。
[4] 标准库的并发设施是类型安全的；§18.1。
[5] 内存模型的存在是为了让大多数程序员不必考虑计算机的机器架构级别；§18.1。
[6] 内存模型使内存看起来大致符合天真的预期；§18.1。
[7] 原子操作允许无锁编程；§18.1。
[8] 将无锁编程留给专家；§18.1。
[9] 有时，顺序解决方案比并发解决方案更简单、更快；§18.1。
[10] 避免数据竞争；§18.1，§18.2。
[11] 相对于直接使用并发，优先使用并行算法；§18.1，§18.5.3。

===== 第 25 页 =====

[12] 线程是系统线程的类型安全接口；§18.2。
[13] 使用 `join()` 等待线程完成；§18.2。
[14] 相对于 `thread`，优先使用 `jthread`；§18.2。
[15] 只要可能，避免显式共享数据；§18.2。
[16] 相对于显式的 `lock/unlock`，优先使用 RAII；§18.3；[CG: CP.20]。
[17] 使用 `scoped_lock` 管理互斥量；§18.3。
[18] 使用 `scoped_lock` 获取多个锁；§18.3；[CG: CP.21]。
[19] 使用 `shared_lock` 实现读者-写者锁；§18.3。
[20] 将互斥量与其保护的数据一起定义；§18.3；[CG: CP.50]。
[21] 对于非常简单的共享，使用原子操作；§18.3.2。
[22] 使用 `condition_variable` 管理线程间的通信；§18.4。
[23] 当你需要拷贝锁或需要更低级别的同步控制时，使用 `unique_lock`（而不是 `scoped_lock`）；§18.4。
[24] 与 `condition_variable` 一起使用时，使用 `unique_lock`（而不是 `scoped_lock`）；§18.4。
[25] 不要在没有条件的情况下等待；§18.4；[CG: CP.42]。
[26] 最小化在临界区中花费的时间；§18.4；[CG: CP.43]。
[27] 从并发任务的角度思考，而不是直接在线程的角度思考；§18.5。
[28] 重视简单性；§18.5。
[29] 相对于直接使用线程和互斥量，优先使用 `packaged_task` 和 `future`；§18.5。
[30] 使用 `promise` 返回结果，并从 `future` 获取结果；§18.5.1；[CG: CP.60]。
[31] 使用 `packaged_task` 处理任务抛出的异常；§18.5.2。
[32] 使用 `packaged_task` 和 `future` 向外部服务发出请求并等待其响应；§18.5.2。
[33] 使用 `async()` 启动简单任务；§18.5.3；[CG: CP.61]。
[34] 使用 `stop_token` 实现协作式终止；§18.5.4。
[35] 协程可以比线程小得多；§18.6。
[36] 相对于手工编写的代码，优先使用协程支持库；§18.6。