# 指针和容器

> **教育是关于做什么、何时做以及为何做；训练是关于如何做。**
>
> —— Richard Hamming

# 15.1 引言

C++ 提供简单而内置的低级类型来保存并引用数据：对象与数组保存数据；指针与引用关联到这些数据。然而现实代码往往需要更专门或更一般的方式来持有并使用数据。举例来说，标准库容器（第 12 章）与迭代器（§13.3）就是为通用算法而设计的。

容器与指针这几类抽象的关键共同点在于：**只有把数据连同访问与操纵它的一组操作一起封装起来，才有可能正确又高效地使用它们**。指针是极其通用且高效的“机器地址”抽象，但要用它来刻画资源所有权已被证明极其困难。因此标准库提供了资源管理指针：即用类封装裸指针，并提供更易正确使用的一组操作。

这些标准库抽象封装内置类型，并且在时间与空间上应当与不犯错误的手工用法同样高效。

它们也没有什么“魔法”。我们也可以沿用标准库的技法设计与实现自己的智能指针与专用容器。

# 15.2 指针一览

指针这一笼统观念指的是：允许我们引用对象并按其类型去访问它的某种手段。内置指针例如 `int*` 只是一种特例；此外还有许许多多形态。

**若干指针与指针类抽象**

| 记号 | 含义 |
|------|------|
| `T*` | 内置指针：指向类型 `T` 的对象，或指向一段连续的 `T` 数组 |
| `T&` | 内置引用：引用类型 `T` 的对象；相当于隐式解引用的一次性指针（§1.7） |
| `unique_ptr<T>` | 独占所有权：指向 `T` |
| `shared_ptr<T>` | 共享所有权：多个 `shared_ptr` 共同拥有一颗对象 |
| `weak_ptr<T>` | 观测由 `shared_ptr` 拥有的对象；访问前通常要先转成 `shared_ptr` |
| `span<T>` | 指向一段连续的 `T` 序列（§15.2.2） |
| `basic_string_view<C>` | 指向常量字符串片段（§10.3） |
| `X_iterator<C>` | 来自容器 `C` 的一段迭代序列，`X` 指明迭代器类别（§13.3） |

同一个对象可以同时被多个指针指到。**拥有型指针**负责最终销毁所指对象；**非拥有指针**（如 `T*`、`span`）则可能悬空——指向早已销毁或离开作用域之处。

经由悬空指针读写是最棘手的一类错误之一：语义上是未定义行为。实践中常常表现为碰巧读写到了别处占用的一块内存：读出任意比特，写入破坏无关数据结构。最体面的下场往往是崩溃——而这常常好过静默的错误。

《C++ 核心准则》[CG] 给出了如何避免悬空指针的规则与可做静态检查的建议。下面是几条实操层面的防线：

- 不要在对象销毁后继续持有指向栈对象的指针；尤其不要从函数返回指向局部对象的指针，也不要把来路不明的指针塞进长寿数据结构。系统化地使用容器与算法（第 12、13 章）往往能避开最容易引出悬空指针的写法。
- 对在自由存储上分配的对象使用拥有型指针。
- 指向静态存储期对象（如全局变量）的指针不会悬空。
- 把指针算术留给资源句柄实现去做（例如 `vector`、`unordered_map`）。
- 记住 `string_view` 与 `span` 都属于非拥有指针。

## 15.2.1 资源管理与智能指针

任何严肃的程式都需要管理资源。**资源**指必须先获取再最终释放的东西——不论是显式还是隐式释放。例子包括内存、锁、套接字、线程句柄、文件句柄等。长跑程序若迟迟不释放资源（泄漏），可能造成严重的性能退化（§12.7），甚至更糟糕的崩溃；哪怕是小程序也可能尴尬地把运行时间拖垮好几个数量级。

标准库组件被设计成尽量不泄漏资源：它们依赖构造函数/析构函数成对的 RAII 技术来保证资源不会比负责它的对象活得更久。`Vector` 用构造函数/析构函数管理元素生命周期的示例（§5.2.2）就是一例；各容器实现大体同理。更重要的是它与基于异常的报错协作无间——例如标准库的锁类：

```cpp
mutex m; // 用于保护共享数据

void f()
{
    scoped_lock lck {m}; // 构造阶段占有互斥量 m
    // ... 操纵共享数据 ...
}
```

线程在 `lck` 构造完成之前不会继续前进（§18.3）；对应的析构函数释放互斥量。于是在该例里，不论函数是通过返回、`return`、末尾坠落抑或抛出异常离开，`scoped_lock` 都会在收尾阶段解锁。

这便是 **RAII**（资源获取即初始化；§5.2.2）。它在地道的 C++ 资源管理中根深蒂固。容器（例如 `vector`、`map`）、`string`、`iostream` 也以同类方式管理自己的缓冲或文件句柄等资源。

到目前为止的例子关注的是栈对象——它们在离开作用域时释放所获资源；那自由存储上的对象呢？在 `<memory>` 里标准库提供两颗常用的智能指针来帮助管理：

- `unique_ptr`：独占所有权（析构时销毁对象）；
- `shared_ptr`：共享所有权（最后一颗 `shared_ptr` 析构时销毁对象）。

最基本的用途是避免因粗心大意导致的内存泄漏：

```cpp
void f(int i, int j) // X* 对比 unique_ptr<X>
{
    X* p = new X;
    unique_ptr<X> sp {new X};
    // ...

    if (i < 99)
        throw Z{};      // 可能抛异常
    if (j < 77)
        return;         // 可能提前返回
    // ... 使用 p 与 sp ...
    delete p;           // 销毁 *p
}
```

我们会“忘记”在 `i<99` 或 `j<77` 的路径上调用 `delete p`。反观 `unique_ptr`，不论我们从何处退出 `f()`，对象都能得到妥善销毁。

颇具讽刺意味的是：若根本不需要指针语义，局部变量就够了：

```cpp
void f(int i, int j)
{
    X x;
    // ...
}
```

不幸的是，`new`（连同指针与引用）被滥用似乎与日俱增。

当你确实需要指针语义时，`unique_ptr` 相对正确使用内置指针而言几乎没有额外的时空开销；它也常用于把自由存储分配的对象搬进搬出函数：

```cpp
unique_ptr<X> make_X(int i) // 创建 X 并立刻交给 unique_ptr
{
    // ... 校验 i 等等 ...
    return unique_ptr<X> {new X{i}};
}
```

可以把 `unique_ptr` 理解为对单个对象（或数组）的句柄，有点像 `vector` 之于一串元素：二者都用 RAII 控制下层对象生命周期，并借助拷贝删除或移动语义让返回值简单又划算（§6.2.2）。

`shared_ptr` 与 `unique_ptr` 相似，差别在于拷贝：`shared_ptr` 可复制；指向同一对象的若干 `shared_ptr` 共享所有权，直至最后一个销毁才把对象删掉：

```cpp
void f(shared_ptr<fstream>);
void g(shared_ptr<fstream>);

void user(const string& name, ios_base::openmode mode)
{
    shared_ptr<fstream> fp {new fstream(name, mode)};
    if (!*fp)                   // 确认文件确实打开成功
        throw No_file{};

    f(fp);
    g(fp);
    // ...
}
```

`fp` 构造阶段打开的文件，会在最后一个仍持有副本的函数（显式或隐式）销毁其 `shared_ptr` 时关闭。注意 `f()` 或 `g()` 可能保存副本，使对象活得比 `user()` 更长。于是 `shared_ptr` 提供一种尊重析构式资源管理的“微型垃圾回收”。它既不是零成本，也未必昂贵，但会让共享对象的寿命更难预判。**只在确有共享所有权需求时再使用 `shared_ptr`。**

先在自由存储创建对象再把指针交给智能指针略显啰嗦，也容易出错——忘记交给 `unique_ptr`、把栈对象的指针塞进 `shared_ptr` 等等。为此 `<memory>` 还提供工厂函数 `make_shared()`、`make_unique()`：

```cpp
struct S {
    int i;
    string s;
    double d;
    // ...
};

auto p1 = make_shared<S>(1, "Ankh Morpork", 4.65); // shared_ptr<S>
auto p2 = make_unique<S>(2, "Oz", 7.62);         // unique_ptr<S>
```

此时 `p2` 指向一颗值为 `{2, "Oz"s, 7.62}` 的 `S`。

`make_shared()` 不仅写法省事：相较分别 `new` 再交给 `shared_ptr`，通常也更高效，因为它往往不必为引用计数单独再做一次分配。

有了 `unique_ptr` 与 `shared_ptr`，许多程序足以贯彻“禁止裸 `new`”（§5.2.2）。但它们终究是概念上的指针，只能排在资源管理的第二顺位——优先选用容器及其他更高层抽象的句柄。尤其是，`shared_ptr` 并不会告诉你哪位所有者可以读、哪位可以写；消灭资源泄漏并不等于消灭数据竞争（§18.5）或其它混淆。

什么时候选用智能指针（如 `unique_ptr`），什么时候选用语义贴合资源的句柄（如 `vector`、`thread`）？答案是：**当你确实需要指针语义时**。

- 共享对象需要指针或引用才可把所有使用者连在一起——多半就该用 `shared_ptr`（除非显然只有一个所有者）。
- 在传统面向对象代码里引用多态对象（§5.5）也需要指针或引用——多半就该用 `unique_ptr`，因为我们事先并不知道确切动态类型（乃至尺寸）。
- 共享的多态对象往往需要 `shared_ptr`。
- 要从函数返回一组对象时并不需要指针：容器作为资源句柄即可依靠拷贝省略（§3.4.2）与移动（§6.2.2）简洁高效地完成。

## 15.2.2 `span`

长久以来，越界访问一直是 C/C++ 严重缺陷的主要来源之一：轻则错误答案，重则崩溃与安全漏洞。容器（第 12 章）、算法（第 13 章）与范围 `for` 显著缓解了问题，但仍有改进余地。常见症结是人们传入指针（裸的或智能的），再用默契约定去猜测背后有几枚元素。对资源句柄之外的代码，最佳实践往往假定“至多指向一个对象”[CG: F.22]，但若缺少工具支撑这条守则就很难落地。

标准库的 `string_view`（§10.3）有帮助，但它只读且面向字符。多数程序员需要更通用的抽象——例如在偏底层的缓冲区读写之间既要高性能又要规避缓冲区溢出。`<span>` 提供的 `span` 本质上就是 `{指针, 长度}` 二元组：

`span` 让我们得以访问一段连续元素。元素可能来源于 `vector`、内置数组等多种载体。与指针类似，`span` **不拥有**所指字符（这一点很像 `string_view`，也像 STL 的一对迭代器；§13.3）。

考虑常见的接口风格：

```cpp
void fpn(int* p, int n)
{
    for (int i = 0; i < n; ++i)
        p[i] = 0;
}
```

我们假定 `p` 指向 `n` 个整数——但这只是约定俗成，既不能据此写出范围 `for`，编译器也难廉价地做彻底检查。更何况假设可能是错的：

```cpp
void use(int x)
{
    int a[100];
    fpn(a, 100);   // OK
    fpn(a, 1000);  // 糟糕：手滑让 fpn 越界
    fpn(a + 10, 100); // fpn 内部仍会越界
    fpn(a, x);     // 看起来人畜无害，实则可疑
}
```

改用 `span` 会好很多：

```cpp
void fs(span<int> p)
{
    for (int& x : p)
        x = 0;
}

void use(int x)
{
    int a[100];
    fs(a);             // 编译器推出 span<int>{a, 100}
    fs(a, 1000);       // 错误：span 期望另一种构造方式
    fs({a + 10, 100}); // 显式写出边界：依旧可能在 fs 内越界，但更醒目
    fs({a, x});        // 可疑意图更明显
}
```

最常见的路径——直接从数组生成 `span`——既安全（编译器计算长度）又省事；其余情形虽说不能完全杜绝错误，至少迫使程序员显式拼出 `span`，从而降低疏忽概率。

在函数之间层层转发 `span` 也比 `(指针, 计数)` 组合更简单，而且无需额外检查：

```cpp
void f1(span<int> p);

void f2(span<int> p)
{
    // ...
    f1(p);
}
```

与容器类似：`span` 的下标 `r[i]` 不做范围检查，越界访问仍是未定义行为。实现当然可以把这种未定义行为实现成抛出异常，但遗憾的是鲜有实现这么做。《C++ 核心准则》支撑库中的原始 `gsl::span` 则会进行范围检查（参见 [CG]）。

# 15.3 若干容器与近邻

标准库还提供了一批并不能完美嵌入 STL 框架（第 12、13 章）的容器：内置数组、`array`、`string` 等等。我偶尔把它们叫作“准容器”，这其实不公平——它们确实保存元素，因而当然是容器；只是各自附带限制或额外功能，使得在 STL 语境里显得有些别扭。把它们单独说明也有助于把 STL 本体讲清楚。

**常用容器与“准容器”一览**

| 类型 | 说明 |
|------|------|
| `T[N]` | 内置数组：在静态存储期、栈或对象体内连续存放 `N` 个 `T`；会退化（decay）成 `T*` |
| `array<T, N>` | 固定长度的连续序列：语义接近内置数组，但规避了大多数经典的坑 |
| `bitset<N>` | 固定长度、`N` 个二进制位的序列 |
| `vector<bool>` | `vector` 的特化：紧凑存放比特并通过代理对象访问 |
| `pair<T, U>` | 一枚 `T` 与一枚 `U` |
| `tuple<T...>` | 任意个数、任意类型的序列 |
| `basic_string<C>` | 类型为 `C` 的字符序列；附带字符串操作 |
| `valarray<T>` | 数值数组：额外提供一批数值运算 |

为何要有这么多容器？因为它们对应常见却各不相同（往往互相重叠）的需求；缺了它们，人们就不得不各自再造轮子。

- `pair` 与 `tuple` 是异质的：其余容器多半是同质元素。
- `array`、`tuple` 的元素在连续内存里；`list`、`map` 多是链表结构。
- `bitset`、`vector<bool>` 存放比特并通过代理访问；其它容器通常可直接摸到元素对象。
- `basic_string` 要求元素是某种字符并提供拼接、依赖 locale 的操作等。
- `valarray` 要求元素是数值并提供批量数值运算。

可以把它们视为服务庞大程序员社群的专项工具。不存在单一容器能吞下全部需求——有些需求彼此冲突，例如“能够增长” vs “布局固定在已知地址”、“插入时不搬动旧元素” vs “内存连续”。

## 15.3.1 `array`

`<array>` 里的 `array` 是一种编译期确定长度的元素序列，长度必须是常量表达式，因而可以把元素放在栈上、对象体内或静态存储区——生命周期随定义它的作用域而定。

把它理解成“带上尺寸的更安全内置数组”最贴切：没有偷偷摸摸退化成龙指针的陷阱，还附带少量便利函数；相较内置数组不会多出时空开销。`array` 并不遵循 STL 容器那种“句柄指向缓冲区”的模型——它直接内含元素，只不过是更难误用的内置数组。

这也意味着必须用初始化列表填充：

```cpp
array<int, 3> a1 = {1, 2, 3};
```

初始化列表的元素个数必须小于等于模板指定的容量。

元素个数不是可选参数：必须是正的常量表达式，元素类型也必须写得明明白白：

```cpp
void f(int n)
{
    array<int> a0 = {1, 2, 3};                     // 错误：未给出大小
    array<string, n> a1 = {"John's", "Queens'"};   // 错误：大小不是常量表达式
    array<string, 0> a2;                           // 错误：大小必须为正
    array<2> a3 = {"John's", "Queens'"};           // 错误：元素类型未写明
    // ...
}
```

若长度要到运行期才知道，请改用 `vector`。

必要时也能显式把 `array` 交给期望指针的 C 接口：

```cpp
void f(int* p, int sz); // C 风格接口

void g()
{
    array<int, 10> a;

    f(a, a.size());       // 错误：无法转换
    f(a.data(), a.size()); // C 风格用法

    auto p = find(a, 777); // C++/STL 风格（把整个范围传进去）
    // ...
}
```

既然 `vector` 灵活得多，为什么还要 `array`？——正因为不那么灵活，它更简单。偶尔把元素直接铺在栈上会比经由 `vector` 句柄访问自由存储更快；但栈空间有限（嵌入式尤其捉襟见肘），栈溢出也相当难看。另有一些领域（安全攸关的实时控制）干脆禁止使用自由存储：`delete` 可能造成碎片化（§12.7）或耗尽内存（§4.3）。

相较于内置数组，`array` 知道自己的长度：更容易套用标准库算法，也能整体赋值。例如：

```cpp
array<int, 3> a1 = {1, 2, 3};
auto a2 = a1; // 拷贝
a2[1] = 5;
a1 = a2;      // 赋值
```

我自己偏爱 `array` 的主要原因是它能挡住令人瞠目的指针退化。例如涉及类层次时：

```cpp
void h()
{
    Circle a1[10];
    array<Circle, 10> a2;
    // ...

    Shape* p1 = a1; // 可以编译：灾难已在酝酿
    Shape* p2 = a2; // 错误：array<Circle,10> 不会偷偷转成 Shape*（太好了）
    p1[3].draw();   // 灾难
}
```

注释里的“灾难”假定 `sizeof(Shape) < sizeof(Circle)`：经由 `Shape*` 去给 `Circle[]` 做下标会得到错误跨度。所有标准容器相对内置数组都有这层优势。

## 15.3.2 `bitset`

系统状态（例如输入流状态）常用一组二元标志表示。C++ 允许通过对整数做位运算处理小规模集合（§1.4）。`bitset<N>` 则把这些操作推广到编译期固定的 `N` 个位 `[0:N)`；若位数多到难以塞进 `long long`（常见为 64 位），`bitset` 往往比手工摆弄整数轻松得多；位数较少时也常被专门优化。

如果想按名字而非编号区分比特，可以用 `set`（§12.5）或枚举（§2.4）。

`bitset` 可用整数或字符串初始化：

```cpp
bitset<9> bs1 {"110001111"};
bitset<9> bs2 {0b1'1000'1111}; // 使用数字分隔符的二进制字面量（§1.4）
```

常规的按位运算符（§1.4）以及移位运算符 `<<`、`>>` 都可直接使用：

```cpp
bitset<9> bs3 = ~bs1;      // 按位取反：bs3 == "001110000"
bitset<9> bs4 = bs1 & bs3; // 全零
bitset<9> bs5 = bs1 << 2;  // 左移补零：bs5 == "000111100"
```

此处的 `<<` 向低位填充 0。

`to_ullong()`、`to_string()` 可视作与构造函数互逆。若想打印整数的二进制展开，可以：

```cpp
void binary(int i)
{
    bitset<8 * sizeof(int)> b = i; // 假定字节长为 8（亦见 §17.7）
    cout << b.to_string() << '\n';
}
```

输出从左到右依次为高位到低位；例如传入 `123` 会打印其二进制模式。

## 15.3.3 `pair`

函数返回两个值司空见惯；最简单的做法往往是定义专用 `struct`。例如返回指针并附带错误码：

```cpp
struct My_res {
    Entry* ptr;
    Error_code err;
};

My_res complex_search(vector<Entry>& v, const string& s)
{
    Entry* found = nullptr;
    Error_code err = Error_code::found;
    // ... 在 v 中查找 s ...
    return {found, err};
}

void user(const string& s)
{
    My_res r = complex_search(entry_table, s);
    if (r.err != Error_code::good) {
        // ... 处理错误 ...
    }
    // ... 使用 r.ptr ...
}
```

你也可以争辩：把失败编码成尾迭代器或 `nullptr` 或许更优雅，但那通常只能表达一类失败。现实里往往需要真正独立的两个返回值；只要命名得当，专用结构非常清晰。然而在巨型代码库里这会催生姓名爆炸；泛型代码又需要一致的命名——于是标准库提供了通用的 `pair`：

```cpp
pair<Entry*, Error_code> complex_search(vector<Entry>& v, const string& s)
{
    Entry* found = nullptr;
    Error_code err = Error_code::found;
    // ...
    return {found, err};
}

void user(const string& s)
{
    auto r = complex_search(entry_table, s);
    if (r.second != Error_code::good) {
        // ...
    }
    // ... 使用 r.first ...
}
```

成员名 `first`、`second` 对实现者友好，在应用代码里却未必称心；结构化绑定（§3.4.5）能改善可读性：

```cpp
void user(const string& s)
{
    auto [ptr, success] = complex_search(entry_table, s);
    if (success != Error_code::good) {
        // ...
    }
    // ... 使用 ptr ...
}
```

`<utility>` 里的 `pair` 在标准库与其余代码中都非常常见。例如算法 `equal_range()` 会返回一对迭代器，指明有序区间里满足比较关系的子序列：

```cpp
template<typename Forward_iterator, typename T, typename Compare>
pair<Forward_iterator, Forward_iterator>
equal_range(Forward_iterator first, Forward_iterator last, const T& val, Compare cmp);

auto less = [](const Record& r1, const Record& r2) { return r1.name < r2.name; };

void f(const vector<Record>& v) // 假定 v 已按 name 排序
{
    auto [first, last] = equal_range(v.begin(), v.end(), Record{"Reg"}, less);

    for (auto p = first; p != last; ++p)
        cout << *p; // 假定已为 Record 定义 <<
}
```

若成员类型支持，`pair` 也会自动生成 `=`、`==`、`<` 等运算符。模板实参推导让我们可以轻松写出：

```cpp
void f(vector<string>& v)
{
    pair p1 {v.begin(), 2};
    auto p2 = make_pair(v.begin(), 2);
    // ...
}
```

`p1`、`p2` 的类型都是 `pair<vector<string>::iterator, int>`。

一旦无需泛化，具名成员的结构体通常更易维护。

## 15.3.4 `tuple`

与数组一样，多数标准容器是**同质**的：元素彼此类型相同。但有时我们希望把不同类型的序列视作单一对象——这就需要异质容器；`pair` 只是一例，却不局限于两项。

标准库的 `tuple` 可视作 `pair` 的推广：允许零个或更多成员。

各成员彼此独立，并不会自动维持跨字段不变式（§4.3）；若要 invariant，必须把 `tuple` 封装进自定义类里强制执行。

针对单个具体用途，`struct` 往往最理想；但在大量泛型场合，`tuple` 让我们免于声明海量小型类型——代价是失去助记成员名。访问 `tuple` 主要靠 `get` 函数模板：

```cpp
string fish = get<0>(t1);   // 第一个元素："Shark"
int count = get<1>(t1);     // 第二个元素：123
double price = get<2>(t1);  // 第三个元素：3.14
```

元素按下标编号（从 0 开始），传给 `get` 的下标必须是编译期常量；这是模板形参为值的典范用法（§7.2.2）。

按下标访问通用却难看且容易抄错。好在若某个类型在 `tuple` 中唯一，还可以通过类型“点名”：

```cpp
auto fish = get<string>(t1);
auto count = get<int>(t1);
auto price = get<double>(t1);
```

`get` 也能用于写入：

```cpp
tuple t0 {};                                     // 空 tuple
tuple<string, int, double> t1 {"Shark", 123, 3.14};
auto t2 = make_tuple(string{"Herring"}, 10, 1.23);
tuple t3 {"Cod"s, 20, 9.99};

get<string>(t1) = "Tuna";
get<int>(t1) = 7;
get<double>(t1) = 312;
```

多数 `tuple` 用法隐藏在更高层抽象背后；结构化绑定（§3.4.5）能把读取写得直白：

```cpp
auto [fish, count, price] = t1;
cout << fish << ' ' << count << ' ' << price << '\n';
fish = "Sea Bass";
```

典型场景还包括函数返回：

```cpp
auto [fish, count, price] = todays_catch();
cout << fish << ' ' << count << ' ' << price << '\n';
```

真正凸显 `tuple` 威力的是：必须把未知个数、未知类型的值打包成一个对象时。

若想遍历 `tuple` 的元素，代码会啰嗦不少——往往需要递归配合编译期求值：

```cpp
template<size_t N = 0, typename... Ts>
constexpr void print(tuple<Ts...> tup)
{
    if constexpr (N < sizeof...(Ts)) {
        cout << get<N>(tup) << ' ';
        print<N + 1>(tup);
    }
}
```

`sizeof...(Ts)` 给出模板包 `Ts` 的元素个数。

使用很直接：

```cpp
print(t0); // 无输出
print(t2); // Herring 10 1.23
print(tuple{"Norah", 17, "Gavin", 14, "Anya", 9, "Courtney", 9, "Ada", 0});
```

与 `pair` 类似：若元素类型支持，`tuple` 也能得到 `=`、`==`、`<` 等运算符；还可以在两项 `tuple` 与 `pair` 之间转换。

# 15.4 表示“多选一”的类型

标准库提供三类类型来表达“若干候选之一”：

**备选类型一览**

| 类型 | 说明 |
|------|------|
| `union` | 语言内置：保存若干候选字段之一（§2.5） |
| `variant<T...>` | `<variant>`：保存若干指定类型之一 |
| `optional<T>` | `<optional>`：要么保存 `T`，要么为空 |
| `any` | `<any>`：保存某个未知类型的值 |

它们彼此相关，遗憾的是接口并不统一。

## 15.4.1 `variant`

`variant<A, B, C>` 常常比显式 `union`（§2.5）更安全也更顺手。最简单的用法莫过于返回“要么成功值要么错误码”：

```cpp
variant<string, Error_code> compose_message(istream& s)
{
    string mess;
    // ... 从 s 读取并拼装消息 ...
    if (no_problems)
        return mess;
    else
        return Error_code{some_problem};
}
```

赋值或初始化 `variant` 时，它会记住当前持有的类型；随后可以查询并向正确的类型索取值：

```cpp
auto m = compose_message(cin);

if (holds_alternative<string>(m)) {
    cout << get<string>(m);
}
else {
    auto err = get<Error_code>(m);
    // ... 处理错误 ...
}
```

有人会偏爱这种风格胜过异常（§4.4）；此外还有更丰富用途——例如简易编译器要把不同节点区别开来：

```cpp
using Node = variant<Expression, Statement, Declaration, Type>;

void check(Node* p)
{
    if (holds_alternative<Expression>(*p)) {
        Expression& e = get<Expression>(*p);
        // ...
    }
    else if (holds_alternative<Statement>(*p)) {
        Statement& s = get<Statement>(*p);
        // ...
    }
    // ... Declaration 与 Type ...
}
```

这种手写分支的模式既啰嗦也相对低效，值得单独封装——于是可以用 `visit`：

```cpp
void check(Node* p)
{
    visit(overloaded {
        [](Expression& e) { /* ... */ },
        [](Statement& s) { /* ... */ },
        // ... Declaration 与 Type ...
    }, *p);
}
```

它在语义上接近虚函数调用，有时还可能更快；但若性能关键仍需实测。多数差异并不重要。

`overloaded` 必不可少却又诡异：**它不是标准类型**。它像魔法一样把一组候选（通常是 lambda）捏成一个可调对象：

```cpp
template<class... Ts>
struct overloaded : Ts... {
    using Ts::operator()...;
};

template<class... Ts>
overloaded(Ts...) -> overloaded<Ts...>; // 推导指引
```

随后 `visit` 会把 `()` 作用在该对象上，按常规重载决议挑出最合适的 lambda。

推导指引主要用来消解微妙的歧义，是基础库类模板构造的重要工具（§7.2.3）。

若试图取出并非当前激活类型的分支，`variant` 会抛出 `bad_variant_access`。

## 15.4.2 `optional`

可以把 `optional<A>` 看成特殊的 `variant`（类似 `variant<A, monostate>`），或是“指向 `A` 的指针但不会出现所有权问题”的泛化。

对那些“可能有返回值也可能没有”的函数很有帮助：

```cpp
optional<string> compose_message(istream& s)
{
    string mess;
    // ... 读取 ...
    if (no_problems)
        return mess;
    return {}; // 空 optional
}
```

调用方可写成：

```cpp
if (auto m = compose_message(cin))
    cout << *m;
else {
    // ...
}
```

这也迎合厌恶异常的程序员（§4.4）。注意这里的 `*`：`optional` 更像指针语义而非对象本体。

“空指针”对应 `{}`。例如：

```cpp
int sum(optional<int> a, optional<int> b)
{
    int res = 0;
    if (a)
        res += *a;
    if (b)
        res += *b;
    return res;
}

int x = sum(17, 19); // 36
int y = sum(17, {}); // 17
int z = sum({}, {}); // 0
```

倘若在没有值的时候强行访问 `optional`，结果是未定义行为——并不会抛异常。因此它谈不上百分之百类型安全；千万别写：

```cpp
int sum2(optional<int> a, optional<int> b)
{
    return *a + *b; // 自讨苦吃
}
```

## 15.4.3 `any`

`any` 容纳任意类型，并且知道自己装着谁——可以理解成“不加模板约束的 `variant`”：

```cpp
any compose_message(istream& s)
{
    string mess;
    // ...
    if (no_problems)
        return mess;
    else
        return error_number;
}
```

赋值之后同样可以按断言的类型取出：

```cpp
auto m = compose_message(cin);
string& s = any_cast<string>(m);
cout << s;
```

若类型不匹配，`any_cast` 抛出 `bad_any_access`。

# 15.5 建议

[1] 库不见得庞大复杂才有价值；§16.1。

[2] 凡是必须先获取再释放的东西都是资源；§15.2.1。

[3] 用资源句柄（RAII）管理资源；§15.2.1；[CG: R.1]。

[4] `T*` 的问题是它能代表太多语义，因而难以辨别裸指针的真实意图；§15.2.1。

[5] 对多态类型对象使用 `unique_ptr`；§15.2.1；[CG: R.20]。

[6] 仅在确有共享所有权时使用 `shared_ptr`；§15.2.1；[CG: R.20]。

[7] 更偏爱语义贴合具体资源的句柄，而不是笼统的智能指针；§15.2.1。

[8] 能用局部对象解决的场景不要用智能指针；§15.2.1。

[9] 默认倾向 `unique_ptr` 而非 `shared_ptr`；§6.3，§15.2.1。

[10] 仅在需要转移所有权职责时，才把 `unique_ptr` / `shared_ptr` 用作参数或返回值；§15.2.1；[CG: F.26] [CG: F.27]。

[11] 用 `make_unique()` 构造 `unique_ptr`；§15.2.1；[CG: R.22]。

[12] 用 `make_shared()` 构造 `shared_ptr`；§15.2.1；[CG: R.23]。

[13] 更偏爱智能指针而非垃圾回收；§6.3，§15.2.1。

[14] 优先使用 `span`，而不是“指针 + 元素个数”接口；§15.2.2；[CG: F.24]。

[15] `span` 支持范围 `for`；§15.2.2。

[16] 若序列长度是编译期常量，可选用 `array`；§15.3.1。

[17] 默认倾向 `array` 胜过内置数组；§15.3.1；[CG: SL.con.2]。

[18] 当你需要 `N` 个比特且 `N` 未必等于某个内置整数类型的宽度时，用 `bitset`；§15.3.2。

[19] 不要滥用 `pair` 与 `tuple`：具名成员的结构体往往更易读；§15.3.3。

[20] 使用 `pair` 时配合模板实参推导或 `make_pair()`，避免重复书写类型；§15.3.3。

[21] 使用 `tuple` 时配合模板实参推导或 `make_tuple()`，避免啰嗦的类型清单；§15.3.3；[CG: T.44]。

[22] 默认倾向 `variant`，不要轻易手写 `union`；§15.4.1；[CG: C.181]。

[23] 借助 `variant` 在若干备选之间分流时，可以考虑 `visit()` 配合 `overloaded()`；§15.4.1。

[24] 面对 `variant`、`optional` 或 `any`，凡是可能存在多个备选的情况，访问前先确认标签/状态；§15.4。
