# 并发

> **保持简单：尽可能简单，但不要更简单。**
>
> —— A. Einstein

# 18.1 引言

并发——同时进行多项任务的执行——广泛用于提升吞吐（让单次计算使用多个处理器）或改善响应度（在一部分程序等待响应时让另一部分继续前进）。现代编程语言普遍为此提供支持。C++ 标准库所提供的支持是一种可移植、类型安全的变体，其理念已在 C++ 中沿用二十多年，并且几乎被所有现代硬件普遍支持。标准库的支持主要面向**系统层并发**，而非直接提供复杂的高层并发模型；后者可作为构建于标准库设施之上的库来提供。

标准库直接支持在单一地址空间中并发执行多个线程。为此，C++ 提供合适的内存模型与一组原子操作。原子操作允许无锁编程 [Dechev,2010]。只要程序员避免数据竞争（对可变数据的失控并发访问），内存模型就能确保一切都按朴素直觉那样工作。不过大多数用户只会通过标准库及其之上的库接触并发。本节简要示例标准库并发支持的主要设施：线程、`mutex`、`lock()` 操作、`packaged_task` 与 `future` 等。这些特性直接构建在操作系统提供的能力之上，相对二者不会造成额外性能损失；但也不能保证相较操作系统必然带来显著性能提升。

不要把并发当成万金油。若任务可以顺序完成，通常更简单也更快。把信息从一个线程传到另一个线程的开销也可能出人意料地大。

除了显式并发特性之外，我们常常能用并行算法让多个执行引擎参与并获得更好性能（§13.6、§17.3.1）。

最后，C++ 支持协程；也就是在调用之间保存状态的函数（§18.6）。

# 18.2 任务与线程

我们把可能与其它计算并发执行的计算称为**任务**。**线程**是程序中任务在系统层的表征。要让任务与其它任务并发运行，可用任务作为实参构造一个线程对象（位于 `<thread>`）。任务可以是函数，也可以是函数对象：

```cpp
void f();                       // 函数

struct F {                     // 函数对象
    void operator()();          // F 的调用运算符（§7.3.2）
};

void user()
{
    thread t1{f};               // f() 在独立线程执行
    thread t2{F{}};             // F{}() 在独立线程执行

    t1.join();                  // 等待 t1
    t2.join();                  // 等待 t2
}
```

`join()` 保证在线程结束之前不会退出 `user()`。**汇合（join）**线程意思是**等待该线程终止**。

忘记调用 `join()` 很容易，而后果通常很糟，于是标准库提供 `jthread`——一种遵循 RAII 的「会自动汇合的线程」，其析构函数会执行 `join()`：

```cpp
void user()
{
    jthread t1{f};
    jthread t2{F{}};
}
```

汇合由析构函数完成，因此顺序与构造顺序相反。此处会先等待 `t2`，再等待 `t1`。

程序的线程共享单一地址空间。在这一点上线程不同于进程——进程通常不直接共享数据。正因为共享地址空间，线程能通过共享对象通信（§18.3）。这类通信通常由锁或其它机制控制，以防止数据竞争（对变量的失控并发访问）。

并发任务的编程可能非常棘手。考虑任务 `f`（函数）与 `F`（函数对象）的可能实现：

```cpp
void f()
{
    cout << "Hello ";
}

struct F {
    void operator()() { cout << "Parallel World!\n"; }
};
```

这是一种糟糕的错误：`f` 与 `F{}` 都在没有任何同步的情况下使用 `cout`。由于两项任务里各个操作的执行顺序未定义，最终输出不可预测，可能在多次运行之间变化，从而产生怪异输出，例如：

```
PaHerallllel o World!
```

标准里对某些保证使我们免于 `ostream` 定义内部的数据竞争（那可能导致崩溃）。

要避免输出流的此类问题，要么只允许一个线程使用某个流，要么使用 `osyncstream`（§11.7.5）。

定义并发程序的任务时，目标是除了在简单而明确的通信点之外**彼此分离**。思考并发任务的最简单方式是：把它视作碰巧与其调用者并发运行的函数。要做到这点，只要传入参数、收回结果，并确保其间不使用共享数据（没有数据竞争）。

## 18.2.1

通常任务需要数据来工作。我们可以很方便地把数据（或指针或引用）作为实参传入。考虑：

```cpp
void f(vector<double>& v);

struct F {
    vector<double>& v;
    F(vector<double>& vv) : v{vv} { }
    void operator()();           // 调用运算符；§7.3.2
};

int main()
{
    vector<double> some_vec{1, 2, 3, 4, 5, 6, 7, 8, 9};
    vector<double> vec2{10, 11, 12, 13, 14};

    jthread t1{f, ref(some_vec)};   // f(some_vec) 在独立线程运行
    jthread t2{F{vec2}};             // F{vec2}() 在独立线程运行
}
```

`F{vec2}` 在 `F` 内保存实参向量的引用。`F` 于是可使用该向量；理想情况下不应有其它任务在 `F` 执行期间访问 `vec2`。若以值传递 `vec2`，可消除这一风险。

`{f, ref(some_vec)}` 的初始化采用线程的变参模板构造函数，能接受任意实参序列（§8.4）。`ref()` 来自 `<functional>`，不幸的是需要用这一类型函数告诉变参模板把 `some_vec` **当作引用**，而不是对象本身。若没有 `ref()`，`some_vec` 会以值传递。编译器会检查第一个实参能否在给定后续实参的情况下调用，并构造必要的函数对象交给线程。因而若 `F::operator()()` 与 `f()` 做的是同一种算法，两种任务处理方式大体等价：都会构造供线程执行的函数对象。

## 18.2.2

在 §18.2.1 的例子里，我通过非常量引用传递实参。仅当我期望任务修改所指数据的值时才这么做（§1.7）。这是有点取巧但并不少见的结果返回方式。不那么费解的技巧是按常量引用传入输入数据，另外传入单独的结果写入位置：

```cpp
void f(const vector<double>& v, double* res);

class F {
public:
    F(const vector<double>& vv, double* p) : v{vv}, res{p} { }
    void operator()();                     // 把结果写入 *res
private:
    const vector<double>& v;
    double* res;
};

double g(const vector<double>&);

void user(vector<double>& vec1, vector<double> vec2, vector<double> vec3)
{
    double res1;
    double res2;
    double res3;

    thread t1{f, cref(vec1), &res1};
    thread t2{F{vec2, &res2}};
    thread t3{[&]() { res3 = g(vec3); }};   // 以引用捕获局部变量

    t1.join();      // 使用结果前先汇合
    t2.join();
    t3.join();

    cout << res1 << ' ' << res2 << ' ' << res3 << '\n';
}
```

此处 `cref(vec1)` 把指向 `vec1` 的常量引用作为 `t1` 的实参传入。这一写法可行也很常见，但我并不特别欣赏经由引用返回结果的做法；我们会在 §18.5.1 回到这个话题。

# 18.3 共享数据

有时任务需要共享数据。此时访问必须经过同步，使得任一时刻至多只有一个任务访问。有经验的程序员会承认这是简化表述（例如多任务同时读不可变数据就没有问题），但不妨想想如何保证任一时刻至多只有一个任务访问某一对象集合。

## 18.3.1

**互斥体（mutex，mutual exclusion object）** 是在线程之间共享数据时的核心构件。线程通过 `lock()` 操作取得互斥体：

```cpp
mutex m;      // 控制用的互斥体
int sh;       // 共享数据

void f()
{
    scoped_lock lck{m};   // 取得互斥体
    sh += 7;              // 操作共享数据
}                         // 隐式释放互斥体
```

`lck` 的类型被推导引 `scoped_lock<mutex>`（§7.2.3）。`scoped_lock` 的构造函数取得互斥体（调用 `m.lock()`）。若另一线程已经占有互斥体，本线程就等待（「阻塞」）直到对方结束访问。线程完成对共享数据的访问后，`scoped_lock` 释放互斥体（调用 `m.unlock()`）。互斥体一旦释放，等待它的线程就会恢复执行（「被唤醒」）。互斥与加锁设施在 `<mutex>` 中。

注意这里使用了 RAII（§6.3）。使用诸如 `scoped_lock` 与 `unique_lock`（§18.4）之类的资源句柄，远比手动反复 `lock()`/`unlock()` 更简单也更安全。

共享数据与互斥体的对应关系基于约定：程序员必须知道哪个互斥体保护哪些数据。显然这容易出错；我们也会用语言手段尽可能把这种对应弄得显而易见。例如：

```cpp
class Record {
public:
    mutex rm;
    // ...
};
```

用不着天才也能猜到：对名为 `rec` 的 `Record`，应当在访问其余成员前先取得 `rec.rm`，尽管注释或更好的命名可能对读者更有帮助。

同时访问多项资源才能完成某个动作的情形并不少见，这可能引出**死锁**。例如：线程 1 先取得 `mutex1` 再试图取得 `mutex2`，线程 2 先取得 `mutex2` 再试图取得 `mutex1`，于是两项任务都无法继续前进。`scoped_lock` 能帮我们同时取得多个锁：

```cpp
void f()
{
    scoped_lock lck{mutex1, mutex2, mutex3};   // 同时取得三把锁
    // ... 操作共享数据 ...
}                                              // 隐式释放所有互斥体
```

只有当 **`lck`** 的所有互斥体实参都已到手之后，`scoped_lock` 才会继续；并且在握有一把互斥体时绝不会阻塞（睡眠）。离开作用域时，`scoped_lock` 析构函数确保释放互斥体。

经由共享数据通信相当底层。尤其程序员必须自己设法弄清各项任务已完成什么、未完成什么。就这点而言，共享数据不如「调用—返回」那样清晰。另一方面，有些人坚信共享一定比拷贝参数与返回值更高效；在大数据量时确实可能如此，但加锁与解锁是相对昂贵的操作。反过来，现代机器拷贝数据——尤其是紧凑数据如向量元素——往往非常快。**不要为了想象中的效率不经思索就选择共享数据，最好先有度量依据。**

基本互斥体一次只允许一个线程访问数据。很常见的一种共享情形是许多读者与单个写者。这种「读写锁」惯用法由 `shared_mutex` 支持。读者可以「共享」方式取得互斥体，其它读者仍可进入；写者则需要独占访问。例如：

```cpp
shared_mutex mx;

void reader()
{
    shared_lock lck{mx};   // 愿与其它读者共享
    // ... 读取 ...
}

void writer()
{
    unique_lock lck{mx};   // 需要独占（unique）访问
    // ... 写入 ...
}
```

## 18.3.2

互斥体是相当重量级的机制，牵涉操作系统。它允许在无数据竞争的前提下完成任意数量的工作。然而若只做一点点工作，还有更简单也更便宜的机制：**原子变量**。例如，下面是经典双重检查锁定的一种简单变体：

```cpp
mutex mut;
atomic<bool> init_x;    // 初始为 false
X x;

if (!init_x) {
    lock_guard lck{mut};
    if (!init_x) {
        // ... 对 x 做非平凡初始化 ...
        init_x = true;
    }
}

// ... 使用 x ...
```

原子变量使我们免于大量动用昂贵得多的互斥体。若 `init_x` 不是原子的，初始化就可能以极低概率失败，留下神秘难寻的错误，因为在 `init_x` 上会存在数据竞争。

此处我用 `lock_guard` 而不是 `scoped_lock`，因为我只需要一把互斥体，最简单的锁就够用了。

# 18.4 等待事件

有时线程需要等待某种外部事件：例如另一线程完成任务，或经过一段时间。最简单的「事件」只是时间流逝。借助 `<chrono>` 的时间设施可以写成：

```cpp
using namespace chrono;

auto t0 = high_resolution_clock::now();
this_thread::sleep_for(milliseconds{20});
auto t1 = high_resolution_clock::now();

cout << duration_cast<nanoseconds>(t1 - t0).count() << " nanoseconds passed\n";
```

我甚至不必另起线程；默认情况下 `this_thread` 指的就是这一个线程。

我用 `duration_cast` 把时钟单位调整到想要的纳秒。

使用外部事件通信的基本支持由 `<condition_variable>` 中的 `condition_variable` 提供。`condition_variable` 是一种让一个线程等待另一线程的机制。特别是，它允许线程等待某个条件（常称为事件）因其它线程的工作而成为真。

使用条件变量能实现多种优雅高效的共享方式，但也颇为棘手。考虑经典的两线程经由队列传消息的例子。为简单起见，我把队列以及避免队列竞争的机制声明在生产者与消费者的全局作用域：

```cpp
class Message {
    // ...
};

queue<Message> mqueue;
condition_variable mcond;
mutex mmutex;
```

类型 `queue`、`condition_variable` 与 `mutex` 均由标准库提供。

`consumer()` 读取并处理 `Message`：

```cpp
void consumer()
{
    while (true) {
        unique_lock lck{mmutex};                           // 取得 mmutex
        mcond.wait(lck, [] { return !mqueue.empty(); });    // 释放 mmutex 并等待；
                                                              // 唤醒时重新取得 mmutex
        auto m = mqueue.front();
        mqueue.pop();
        lck.unlock();                                      // 释放 mmutex
        // ... 处理 m ...
    }
}
```

此处我用 `unique_lock` 明确保护对队列以及对 `condition_variable` 的操作。在条件变量上等待时会**释放**其锁实参，直到等待结束（队列非空）再重新取得它。对条件的显式检查——此处 `!mqueue.empty()`——可防止「醒来后发现别的任务抢先一步」导致条件不再成立。

我用 `unique_lock` 而非 `scoped_lock`，原因有二：

- 需要把锁传给条件变量的 `wait()`。`scoped_lock` 不能移动，而 `unique_lock` 可以。
- 希望在处理消息**之前**解锁保护条件变量的互斥体。`unique_lock` 提供 `lock()`、`unlock()` 等低级同步控制。

另一方面，`unique_lock` 只能管理单个互斥体。

对应的 `producer()` 如下：

```cpp
void producer()
{
    while (true) {
        Message m;
        // ... 填充消息 ...
        scoped_lock lck{mmutex};    // 保护操作
        mqueue.push(m);
        mcond.notify_one();         // 通知
    }                               // 作用域末尾释放 mmutex
}
```

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

# 18.6 协程

**协程**是在调用之间保持状态的函数。在这一点上它有点像函数对象，但在调用之间保存与恢复状态是隐式且完整的。考虑经典示例：

```cpp
generator<long long> fib()
{
    long long a = 0;
    long long b = 1;
    while (true) {
        auto next = a + b;
        co_yield next;           // 保存状态，返回值并等待
        a = b;
        b = next;
    }
}

void user(int max)
{
    auto g = fib();
    for (int i = 0; i < max; ++i)
        cout << g() << ' ';
}
```

这会生成：

```
1 2 3 5 8 13 ...
```

（具体调用语法取决于所用的 `generator` 设施；关键是保存发生器对象并在其上前进。）

`generator` 返回类型用来存放协程在调用之间的状态。我们当然可以手写函数对象 `Fib` 达成类似效果，但那就必须亲自维护状态。状态更大、计算更复杂时，手写保存/恢复既乏味又难优化，也容易出错。实际上，协程会在调用之间保存其栈帧。`co_yield` 返回值并等待下一次调用。`co_return` 返回值并终止协程。

协程可以是同步的（调用者等待结果）或异步的（调用者先做别的工作，再在稍后取用结果）。上面的 Fibonacci 示例显然是同步的，这也让某些优化成为可能——例如，优秀的优化器可以把对 `fib()` 的调用内联并展开循环，最终只留下一连串 `<<`，再继续优化成：

```cpp
cout << "1 2 3 5 8 13";   // fib(6)，示意
```

协程实现为极度灵活的框架，足以涵盖很广的潜在用途；它由专家设计并服务于专家，又带点委员会设计的痕迹。这没问题——唯独 **C++20 的标准库尚未提供足够设施把简单用法简单化**。例如，`generator` 还不在标准库里（至少在写作本文时如此）。不过提案已有；在网上检索可以找到稳妥的实现，`[Cppcoro]` 便是一例。

## 18.6.1

在《计算机程序设计艺术》第一卷里，高德纳肯定了协程的用处，但也慨叹难以给出简短示例——协程最擅长简化复杂系统。在此我只举一个把玩 primitive 的极简示例，用以演示早年促成 C++ 成功的一类事件驱动仿真所需的想法。关键想法是把系统表示成一个由简单任务（协程）组成的网络，它们协作完成复杂任务。基本上每个任务都是一个参与者（actor），承担宏大工作中很小的一块：有些是发生器，源源不断地产出请求（可能用到随机数，也可能接入真实数据）；有些是网络片段，计算结果；还有些负责产出输出。我个人更倾向于任务（协程）经由消息队列通信。组织此类系统的一种方式是让每个任务在产出结果之后把自己放回事件队列等待更多工作；需要时再由上层的调度器从事件队列挑选下一个任务运行。这是**协作式多任务**。我曾致谢借用 Simula [Dahl,1970] 的关键想法，把它们化作最早的 C++ 库的基础之一（§19.1.2）。

此类设计的要点包括：

- 大量彼此独立的协程在调用之间保持状态；
- 某种多态机制：使我们能把不同类型协程放进事件表里并独立调用；
- 调度器：从表中选出下一次运行的协程。

这里我只展示两个协程并轮流执行它们。此类系统务必不能使用过多内存——因此我们不用进程或线程来做这类应用。**线程往往占用一两 MB（多半是给栈）**；协程常常只占几十字节。若你需要成千上万个任务，差别非常大。**协程之间上下文切换也比线程或进程快得多。**

首先，我们需要运行期多态，以便能以统一方式调用数十乃至上百种协程：

```cpp
struct Event_base {
    virtual void operator()() = 0;
    virtual ~Event_base() = default;
};

template<class Act>
struct Event : Event_base {
    Event(std::string n, Act a) : name{move(n)}, act{move(a)} {}
    std::string name;
    Act act;
    void operator()() override { act(); }
};
```

一个 `Event` 存储动作并允许调用它；动作通常将是协程。我加了 `name` 只是想说明事件常常携带的不止于协程句柄。

下面是一个很琐细的用法：

```cpp
void test()
{
    vector<Event_base*> events = {
        new Event{"integers ", sequencer(10)},
        new Event{"chars ", char_seq('a')}
    };

    vector order{0, 1, 1, 0, 1, 0, 1, 0, 0};

    for (int x : order)
        (*events[x])();

    for (auto p : events)
        delete p;
}
```

到目前为止尚无特异于协程之处——它只是常规的面向对象框架：在一组潜在不同类型对象上执行操作。然而 `sequencer` 与 `char_seq` 碰巧是协程：它们在调用之间保持状态，这对现实用途必不可少：

```cpp
task sequencer(int start, int step = 1)
{
    auto value = start;
    while (true) {
        cout << "value: " << value << '\n';    // 输出结果
        co_yield 0;                             // 挂起，直到再次被唤起
        value += step;                          // 更新状态
    }
}
```

能看出这是协程，是因为它使用了 `co_yield` 在调用间隔挂起自己，这也意味着返回类型 `task` 必须是协程句柄（见下文）。

这是刻意的极简协程：它只做递增序列并打印。在严肃的仿真里，这类输出会直接或间接成为其它协程的输入。

`char_seq` 与之很像，但类型不同，用以操练运行期多态：

```cpp
task char_seq(char start)
{
    auto value = start;
    while (true) {
        cout << "value: " << value << '\n';
        co_yield 0;
        ++value;
    }
}
```

「魔力」在返回类型 `task`：它保存协程状态（实际上就是函数的栈帧）并在调用之间延续；它也决定了 `co_yield` 的含义。从使用者角度看，`task` 理应极简——只需提供一个运算符用来唤起协程：

```cpp
struct task {
    void operator()();
    // ... 实现细节 ...
};
```

若 `task` 来自库（最好是标准库），我们通常只知道这么多就够；但事实并非如此，所以下面略微暗示如何实现此类「协程句柄」类型。提案在路上；网上同样有可参考的实现，`[Cppcoro]` 便是其中之一。

我把 `task` 写成能实现示例所需的极简版本：

```cpp
struct task {
    void operator()() { coro.resume(); }
    // ... promise_type、coroutine_handle 等细节 ...
};
```

除非你是一名库实现者，想让旁人免于折腾，否则我强烈不建议自己动手写这类代码。若你只是好奇，网上有大量讲解可读。

# 18.7 建议

[1] 使用并发来改善响应度或提升吞吐；§18.1。

[2] 在你力所能及的最高抽象层级上工作；§18.1。

[3] 把进程视作线程之外的备选方案；§18.1。

[4] 标准库的并发设施是类型安全的；§18.1。

[5] 内存模型意在让大多数程序员不必下降到计算机体系结构层面思考；§18.1。

[6] 内存模型使内存大体符合朴素直觉；§18.1。

[7] 原子操作使得无锁编程成为可能；§18.1。

[8] 把无锁编程留给专家；§18.1。

[9] 有时顺序解法比并发解法更简单也更快；§18.1。

[10] 避免数据竞争；§18.1，§18.2。

[11] 相较直接使用并发，更优先考虑并行算法；§18.1，§18.5.3。

[12] 线程是对系统线程的类型安全封装；§18.2。

[13] 用 `join()` 等待线程结束；§18.2。

[14] 相较于裸 `thread`，更偏向 `jthread`；§18.2。

[15] 尽可能避免显式共享数据；§18.2。

[16] 相较于手动 `lock`/`unlock`，更偏向 RAII；§18.3；[CG: CP.20]。

[17] 用 `scoped_lock` 管理互斥体；§18.3。

[18] 用 `scoped_lock` 同时取得多把锁；§18.3；[CG: CP.21]。

[19] 用 `shared_lock` 实现读者—写者锁；§18.3。

[20] 把互斥体与它保护的数据一起定义；§18.3；[CG: CP.50]。

[21] 只做很简单的共享时使用原子类型；§18.3.2。

[22] 用 `condition_variable` 协调线程间通信；§18.4。

[23] 当你需要转移锁或需要更低级同步控制时，使用 `unique_lock`（而非 `scoped_lock`）；§18.4。

[24] 与 `condition_variable` 一起使用时，采用 `unique_lock`（而非 `scoped_lock`）；§18.4。

[25] 不要在无条件检查时盲目等待；§18.4；[CG: CP.42]。

[26] 最小化临界区内耗时；§18.4；[CG: CP.43]。

[27] 从并发任务的角度思考，而不是死盯着线程；§18.5。

[28] 崇尚简洁；§18.5。

[29] 相较于直接使用线程与互斥体，更偏向 `packaged_task` 与 future；§18.5。

[30] 用 `promise` 返回值，用 `future` 取结果；§18.5.1；[CG: CP.60]。

[31] 用 `packaged_task` 处理任务抛出的异常；§18.5.2。

[32] 用 `packaged_task` 与 `future` 表达对外部服务的请求并等待响应；§18.5.2。

[33] 用 `async()` 启动简单任务；§18.5.3；[CG: CP.61]。

[34] 用 `stop_token` 实现协作式终止；§18.5.4。

[35] 协程可比线程小得多；§18.6。

[36] 相较于手写底层代码，更偏向协程支持库；§18.6。
