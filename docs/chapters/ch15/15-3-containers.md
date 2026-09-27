# 15.3 容器

标准库还提供了若干并不能完美嵌入 STL 框架（[第 12 章](../ch12/index.md)、[第 13 章](../ch13/index.md)）的容器。例子包括内置数组、`array` 与 `string`。我有时把它们叫作“准容器”，但这并不完全公平：它们保存元素，因而确实是容器，只是各自带有限制或额外设施，使它们在 STL 语境中显得别扭。把它们单独描述也有助于简化对 STL 本身的说明。

**容器**

| 类型 | 说明 |
|------|------|
| `T[N]` | 内置数组：固定大小、连续分配的 `N` 个类型为 `T` 的元素序列；隐式转换为 `T*` |
| `array<T,N>` | 固定大小、连续分配的 `N` 个类型为 `T` 的元素序列；像内置数组，但多数问题已解决 |
| `bitset<N>` | 固定大小的 `N` 个比特的序列 |
| `vector<bool>` | 在 `vector` 的特化中紧凑存放的比特序列 |
| `pair<T,U>` | 类型分别为 `T` 与 `U` 的两个元素 |
| `tuple<T...>` | 任意个数、任意类型的元素序列 |
| `basic_string<C>` | 类型为 `C` 的字符序列；提供字符串操作 |
| `valarray<T>` | 类型为 `T` 的数值数组；提供数值运算 |

为什么标准库要提供这么多容器？它们服务于常见但不同（往往重叠）的需求。若标准库不提供它们，许多人就不得不自行设计与实现。例如：

- `pair` 与 `tuple` 是异质的；所有其他容器都是同质的（所有元素类型相同）。
- `array` 与 `tuple` 的元素是连续分配的；`list` 与 `map` 是链接结构。
- `bitset` 与 `vector<bool>` 保存比特并通过代理对象访问；所有其他标准库容器可保存多种类型并直接访问元素。
- `basic_string` 要求其元素是某种形式的字符，并提供字符串操纵，例如拼接与 locale 敏感操作。
- `valarray` 要求其元素是数，并提供数值运算。

所有这些容器都可以看作在为大型程序员社群提供所需的专门服务。没有单一容器能满足所有这些需求，因为有些需求相互矛盾，例如“能够增长”对“保证分配在固定位置”，以及“添加元素时元素不移动”对“连续分配”。

## 15.3.1 `array`

`<array>` 中定义的 `array` 是给定类型元素的固定大小序列，元素个数在编译期指定。因此，`array` 可以连同其元素一起分配在栈上、对象中或静态存储中。元素分配在定义该 `array` 的作用域中。

把 `array` 理解成“尺寸牢牢附在身上的内置数组”最贴切：没有隐式的、可能令人吃惊的向指针类型的转换，并提供少量便利函数。相对使用内置数组，使用 `array` 没有时间或空间开销。`array` 并不遵循 STL 容器那种“指向元素的句柄”模型；相反，`array` 直接包含其元素。它不多不少，就是更安全的内置数组。

这意味着 `array` 可以且必须用初始化列表初始化：

```cpp
array<int,3> a1 = {1,2,3};
```

初始化器中的元素个数必须等于或少于为该 `array` 指定的元素个数。

元素个数不是可选的；元素个数必须是常量表达式；元素个数必须为正；元素类型必须显式写明：

```cpp
void f(int n)
{
    array<int> a0 = {1,2,3};                          // 错误：未指定大小
    array<string,n> a1 = {"John's", "Queens' "};      // 错误：大小不是常量表达式
    array<string,0> a2;                               // 错误：大小必须为正
    array<2> a3 = {"John's", "Queens' "};             // 错误：未写明元素类型
    // ...
}
```

若需要元素个数是变量，请使用 `vector`。

必要时，可以把 `array` 显式传给期望指针的 C 风格函数。例如：

```cpp
void f(int* p, int sz);         // C 风格接口

void g()
{
    array<int,10> a;

    f(a, a.size());             // 错误：无法转换
    f(a.data(), a.size());      // C 风格用法

    auto p = find(a, 777);      // C++/STL 风格用法（传入一个范围）
    // ...
}
```

既然 `vector` 灵活得多，我们为什么还要用 `array`？`array` 不那么灵活，因而更简单。偶尔，直接访问分配在栈上的元素，会比把元素分配在自由存储上、经 `vector`（句柄）间接访问、然后再释放它们，带来显著的性能优势。另一方面，栈是有限资源（尤其在某些嵌入式系统上），栈溢出也很讨厌。此外，在某些应用领域——例如安全攸关的实时控制——自由存储分配是被禁止的。例如，使用 `delete` 可能导致碎片化（[§12.7](../ch12/12-7-allocators.md)）或内存耗尽（[§4.3](../ch04/4-3-invariants.md)）。

既然可以用内置数组，我们为什么还要用 `array`？`array` 知道自己的大小，因而易于与标准库算法一起使用，并且可以用 `=` 拷贝。例如：

```cpp
array<int,3> a1 = {1, 2, 3};
auto a2 = a1;     // 拷贝
a2[1] = 5;
a1 = a2;          // 赋值
```

不过，我偏爱 `array` 的主要原因是：它使我免于令人吃惊且讨厌的向指针的转换。考虑一个涉及类层次结构的例子：

```cpp
void h()
{
    Circle a1[10];
    array<Circle,10> a2;
    // ...
    Shape* p1 = a1;       // OK：灾难等待发生
    Shape* p2 = a2;       // 错误：array<Circle,10> 不能转换为 Shape*（很好！）
    p1[3].draw();         // 灾难
}
```

“灾难”这一注释假定 `sizeof(Shape)<sizeof(Circle)`，因此通过 `Shape*` 对 `Circle[]` 做下标会得到错误的偏移。所有标准容器相对内置数组都提供这一优势。

## 15.3.2 `bitset`

系统的某些方面——例如输入流的状态——常常表示为一组标志，指示 good/bad、true/false、on/off 一类二元条件。C++ 通过对整数的按位运算高效支持小规模标志集合的观念（[§1.4](../ch01/1-4-types-variables.md)）。类 `bitset<N>` 把这一观念推广为对 `N` 个比特的序列 `[0:N)` 的操作，其中 `N` 在编译期已知。对装不进 `long long int`（常常是 64 位）的比特集合，使用 `bitset` 比直接使用整数方便得多。对较小的集合，`bitset` 通常也会被优化。若你想按名字而非编号指称这些比特，可以使用 `set`（[§12.5](../ch12/12-5-map.md)）或枚举（[§2.4](../ch02/2-4-enum.md)）。

`bitset` 可以用整数或字符串初始化：

```cpp
bitset<9> bs1 {"110001111"};
bitset<9> bs2 {0b1'1000'1111};       // 使用数字分隔符的二进制字面量（[§1.4](../ch01/1-4-types-variables.md)）
```

通常的按位运算符（[§1.4](../ch01/1-4-types-variables.md)）以及左移、右移运算符（`<<` 与 `>>`）都可以使用：

```cpp
bitset<9> bs3 = ~bs1;               // 取反：bs3=="001110000"
bitset<9> bs4 = bs1&bs3;            // 全零
bitset<9> bs5 = bs1<<2;             // 左移：bs5 = "000111100"
```

移位运算符（这里是 `<<`）会“移入”零。

操作 `to_ullong()` 与 `to_string()` 提供与构造函数相反的操作。例如，我们可以写出一个 `int` 的二进制表示：

```cpp
void binary(int i)
{
    bitset<8*sizeof(int)> b = i;             // 假定 8 位字节（亦见 [§17.7](../ch17/17-7-numeric-limits.md)）
    cout << b.to_string() << '\n';           // 写出 i 的各个比特
}
```

这会把比特表示为从左到右的 1 与 0，最高有效位在最左，因此实参 `123` 会给出输出

```
00000000000000000000000001111011
```

对本例而言，直接使用 `bitset` 的输出运算符更简单：

```cpp
void binary2(int i)
{
    bitset<8*sizeof(int)> b = i;      // 假定 8 位字节（亦见 [§17.7](../ch17/17-7-numeric-limits.md)）
    cout << b << '\n';                // 写出 i 的各个比特
}
```

`bitset` 还提供许多用于使用与操纵比特集合的函数，例如 `all()`、`any()`、`none()`、`count()`、`flip()`。

## 15.3.3 `pair`

函数返回两个值相当常见。做法很多，最简单且往往最好的是为此定义一个 `struct`。例如，我们可以返回一个值与一个成功指示：

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
    My_res r = complex_search(entry_table, s);      // 搜索 entry_table
    if (r.err != Error_code::good) {
        // ... 处理错误 ...
    }
    // ... 使用 r.ptr ...
}
```

我们可以争辩：把失败编码为尾迭代器或 `nullptr` 更优雅，但那只能表达一种失败。我们常常希望返回两个独立的值。为每一对值定义特定的具名 `struct` 往往效果很好，而且只要“值对”结构体及其成员的名字选得好，就相当可读。然而，在大型代码库中这可能导致名字与约定激增，并且对需要一致命名的泛型代码并不适用。因此，标准库提供 `pair`，作为“值对”用例的通用支持。使用 `pair`，我们的简单例子变成：

```cpp
pair<Entry*,Error_code> complex_search(vector<Entry>& v, const string& s)
{
    Entry* found = nullptr;
    Error_code err = Error_code::found;
    // ... 在 v 中查找 s ...
    return {found, err};
}

void user(const string& s)
{
    auto r = complex_search(entry_table, s);           // 搜索 entry_table
    if (r.second != Error_code::good) {
        // ... 处理错误 ...
    }
    // ... 使用 r.first ...
}
```

`pair` 的成员名为 `first` 与 `second`。从实现者角度看这说得通，但在应用代码中我们可能想用自己的名字。结构化绑定（[§3.4.5](../ch03/3-4-parameters.md#3.4.5)）可以用来处理这一点：

```cpp
void user(const string& s)
{
    auto [ptr, success] = complex_search(entry_table, s);      // 搜索 entry_table
    if (success != Error_code::good) {
        // ... 处理错误 ...
    }
    // ... 使用 ptr ...
}
```

标准库的 `pair`（来自 `<utility>`）在标准库及其他地方的“值对”用例中相当常见。例如，标准库算法 `equal_range` 返回一对迭代器，指明满足谓词的子序列。给定有序序列 `[first:last)`，`equal_range()` 将返回表示匹配谓词 `cmp` 的那个子序列的 `pair`。我们可以用它在已排序的 `Record` 序列中搜索：

```cpp
template<typename Forward_iterator, typename T, typename Compare>
pair<Forward_iterator, Forward_iterator>
equal_range(Forward_iterator first, Forward_iterator last, const T& val, Compare cmp);

auto less = [](const Record& r1, const Record& r2) { return r1.name < r2.name; };

void f(const vector<Record>& v)            // 假定 v 已按 name 字段排序
{
    auto [first, last] = equal_range(v.begin(), v.end(), Record{"Reg"}, less);

    for (auto p = first; p != last; ++p)                // 打印所有相等记录
        cout << *p;                                     // 假定已为 Record 定义 <<
}
```

若其元素支持，`pair` 提供诸如 `=`、`==` 与 `<` 的运算符。类型推导使我们无需显式提及类型即可轻松创建 `pair`。例如：

```cpp
void f(vector<string>& v)
{
    pair p1 {v.begin(), 2};                         // 一种写法
    auto p2 = make_pair(v.begin(), 2);              // 另一种写法
    // ...
}
```

`p1` 与 `p2` 的类型都是 `pair<vector<string>::iterator,int>`。

当代码不需要泛型时，带有具名成员的简单 `struct` 往往带来更易维护的代码。

## 15.3.4 `tuple`

与数组一样，标准库容器是同质的；也就是说，它们的所有元素都是单一类型。然而，有时我们希望把不同类型元素的序列当作单一对象来处理；也就是说，我们想要异质容器。`pair` 是一例，但并非所有这类异质序列都恰好只有两个元素。标准库提供 `tuple`，作为带有零个或更多元素的 `pair` 的推广：

```cpp
tuple t0 {};                                                      // 空
tuple<string,int,double> t1 {"Shark",123,3.14};                   // 类型显式指定
auto t2 = make_tuple(string{"Herring"},10,1.23);                  // 类型推导为 tuple<string,int,double>
tuple t3 {"Cod"s,20,9.99};                                        // 类型推导为 tuple<string,int,double>
```

`tuple` 的元素（成员）彼此独立；它们之间不维护不变式（[§4.3](../ch04/4-3-invariants.md)）。若我们想要不变式，必须把 `tuple` 封装进强制该不变式的类中。

对单一、具体的用途，简单的 `struct` 往往最理想，但在许多泛型用途中，`tuple` 的灵活性使我们免于定义许多 `struct`，代价是没有助记的成员名。`tuple` 的成员通过 `get` 函数模板访问。例如：

```cpp
string fish = get<0>(t1);              // 取得第一个元素："Shark"
int count = get<1>(t1);                // 取得第二个元素：123
double price = get<2>(t1);             // 取得第三个元素：3.14
```

`tuple` 的元素从零开始编号，传给 `get()` 的下标实参必须是常量。函数 `get` 是以该下标为模板值实参的函数模板（[§7.2.2](../ch07/7-2-parameterized-types.md#7.2.2)）。

通过下标访问 `tuple` 成员通用、难看，且有些容易出错。幸运的是，若 `tuple` 中某个元素的类型在该 `tuple` 中唯一，就可以用其类型“命名”它：

```cpp
auto fish = get<string>(t1);                // 取得 string："Shark"
auto count = get<int>(t1);                  // 取得 int：123
auto price = get<double>(t1);               // 取得 double：3.14
```

我们也可以用 `get<>` 写入：

```cpp
get<string>(t1) = "Tuna";       // 写入 string
get<int>(t1) = 7;               // 写入 int
get<double>(t1) = 312;          // 写入 double
```

多数对 `tuple` 的使用隐藏在更高层构造的实现中。例如，我们可以用结构化绑定（[§3.4.5](../ch03/3-4-parameters.md#3.4.5)）访问 `t1` 的成员：

```cpp
auto [fish, count, price] = t1;
cout << fish << ' ' << count << ' ' << price << '\n';      // 读
fish = "Sea Bass";                                                       // 写
```

通常，这种绑定及其底层对 `tuple` 的使用出现在函数调用中：

```cpp
auto [fish, count, price] = todays_catch();
cout << fish << ' ' << count << ' ' << price << '\n';
```

`tuple` 的真正力量在于：当你必须把未知个数、未知类型的元素作为对象存储或传来传去时。

显式遍历 `tuple` 的元素有些凌乱，需要递归以及对函数体的编译期求值：

```cpp
template <size_t N = 0, typename... Ts>
constexpr void print(tuple<Ts...> tup)
{
    if constexpr (N < sizeof...(Ts)) {         // 尚未到末尾？
        cout << get<N>(tup) << ' ';            // 打印第 N 个元素
        print<N + 1>(tup);                     // 打印下一个元素
    }
}
```

这里，`sizeof...(Ts)` 给出 `Ts` 中的元素个数。

使用 `print()` 很直接：

```cpp
print(t0);           // 无输出
print(t2);           // Herring 10 1.23
print(tuple{ "Norah", 17, "Gavin", 14, "Anya", 9, "Courtney", 9, "Ada", 0 });
```

与 `pair` 一样，若其元素支持，`tuple` 提供诸如 `=`、`==` 与 `<` 的运算符。在 `pair` 与带有两个成员的 `tuple` 之间也有转换。
