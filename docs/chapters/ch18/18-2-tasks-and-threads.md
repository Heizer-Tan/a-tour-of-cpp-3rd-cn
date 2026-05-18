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
