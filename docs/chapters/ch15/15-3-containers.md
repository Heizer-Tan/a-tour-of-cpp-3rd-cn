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
