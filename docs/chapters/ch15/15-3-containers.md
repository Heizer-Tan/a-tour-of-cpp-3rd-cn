# 15.3 容器

标准库提供了几个不能完美融入 STL 框架（第 12 章，第 13 章）的容器。例如内置数组、`array` 和 `string`。我有时称它们为“准容器”，但这并不完全公平：它们容纳元素，因此是容器，但每个都有一些限制或附加功能，使它们在 STL 的上下文中不太方便。单独描述它们也简化了 STL 的描述。

**容器**

| 类型 | 描述 |
|------|------|
| `T[N]` | 内置数组：固定大小的连续分配序列，包含 `N` 个 `T` 类型元素；隐式转换为 `T*` |
| `array<T,N>` | 固定大小的连续分配序列，包含 `N` 个 `T` 类型元素；类似于内置数组，但解决了大部分问题 |
| `bitset<N>` | 固定大小的 `N` 位序列 |
| `vector<bool>` | 在 `vector` 的特化中紧凑存储的位序列 |
| `pair<T,U>` | 两个元素，类型分别为 `T` 和 `U` |
| `tuple<T...>` | 任意数量、任意类型元素的序列 |
| `basic_string<C>` | 类型 `C` 的字符序列；提供字符串操作 |
| `valarray<T>` | 类型 `T` 的数值数组；提供数值运算 |

为什么标准要提供这么多容器？它们服务于常见但不同（常常重叠）的需求。如果标准库不提供它们，许多人将不得不自己设计并实现它们。例如：

===== 第 12 页 =====

- `pair` 和 `tuple` 是异构的；所有其他容器都是同构的（所有元素类型相同）。
- `array` 和 `tuple` 的元素是连续分配的；`list` 和 `map` 是链接结构。
- `bitset` 和 `vector<bool>` 持有位并通过代理对象访问它们；所有其他标准库容器可以持有各种类型并直接访问元素。
- `basic_string` 要求其元素是某种形式的字符，并提供字符串操作，如连接和本地环境敏感操作。
- `valarray` 要求其元素是数字，并提供数值运算。

所有这些容器都可以被视为提供了大型程序员群体所需的专门服务。没有单一的容器能够满足所有这些需求，因为有些需求是相互矛盾的，例如“能够增长” vs “保证分配在固定位置”，以及“添加元素时元素不移动” vs “连续分配”。

### 15.3.1 array

定义在 `<array>` 中的 `array` 是一个固定大小的元素序列，其中元素个数在编译时指定。因此，`array` 可以将其元素分配在栈上、对象内或静态存储中。元素在 `array` 定义所在的作用域内分配。`array` 最好被理解为一个带有固定大小的内置数组，没有隐式的、可能令人惊讶的到指针类型的转换，并提供了一些便利函数。与使用内置数组相比，使用 `array` 没有（时间或空间上的）开销。`array` 不遵循 STL 容器的“指向元素的句柄”模型。相反，`array` 直接包含其元素。它只不过是内置数组的一个更安全的版本。

这意味着 `array` 可以通过初始化列表进行初始化：

```cpp
array<int,3> a1 = {1, 2, 3};
```

===== 第 13 页 =====

初始化列表中的元素个数必须等于或小于为数组指定的元素个数。

元素个数不是可选的，元素个数必须是常量表达式，元素个数必须为正，并且元素类型必须显式指定：

```cpp
void f(int n)
{
    array<int> a0 = {1,2,3};                // 错误：未指定大小
    array<string, n> a1 = {"John's", "Queens' "}; // 错误：大小不是常量表达式
    array<string, 0> a2;                    // 错误：大小必须为正
    array<2> a3 = {"John's", "Queens' "};   // 错误：未指定元素类型
    // ...
}
```

如果你需要元素个数为变量，请使用 `vector`。

必要时，`array` 可以被显式地传递给期望指针的 C 风格函数。例如：

```cpp
void f(int* p, int sz);   // C 风格接口

void g()
{
    array<int,10> a;
    f(a, a.size());       // 错误：没有转换
    f(a.data(), a.size()); // C 风格使用
    auto p = find(a, 777); // C++/STL 风格使用（传递一个范围）
    // ...
}
```

既然 `vector` 如此灵活，为什么还要使用 `array`？`array` 不那么灵活，因此更简单。偶尔，直接访问分配在栈上的元素与在自由存储区上分配元素、通过 `vector`（一个句柄）间接访问它们然后释放它们相比，有显著的性能优势。另一方面，栈是一种有限的资源（尤其是在某些嵌入式系统上），栈溢出是棘手的。此外，有些应用领域（如安全关键的实时控制）禁止使用自由存储区分配。例如，使用 `delete` 可能导致碎片（§12.7）或内存耗尽（§4.3）。

既然可以使用内置数组，为什么还要使用 `array`？`array` 知道它的大小，因此易于与标准库算法一起使用，并且可以使用 `=` 进行拷贝。例如：

```cpp
array<int,3> a1 = {1, 2, 3};
auto a2 = a1;   // 拷贝
a2[1] = 5;
a1 = a2;        // 赋值
```

然而，我偏爱 `array` 的主要原因是它使我免于令人惊讶且糟糕的到指针的转换。考虑一个涉及类层次结构的例子：

```cpp
void h()
{
    Circle a1[10];
    array<Circle,10> a2;
    // ...
    Shape* p1 = a1;   // OK：等待发生的灾难
    Shape* p2 = a2;   // 错误：没有从 array<Circle,10> 到 Shape* 的转换（好！）
    p1[3].draw();     // 灾难
}
```

“灾难”的注释假设 `sizeof(Shape) < sizeof(Circle)`，因此通过 `Shape*` 对 `Circle` 数组进行下标会得到错误的偏移量。所有标准容器都提供了相对于内置数组的这一优势。

### 15.3.2 bitset

系统的某些方面，例如输入流的状态，通常表示为一组指示二进制条件的标志，如 good/bad、true/false、on/off。C++ 通过整数的按位操作高效地支持小标志集的概念（§1.4）。类 `bitset<N>` 泛化了这一概念，提供了对 `N` 位序列 `[0:N)` 的操作，其中 `N` 在编译时已知。对于不适合 `long long int`（通常为 64 位）的位集，使用 `bitset` 比直接使用整数方便得多。对于较小的集合，`bitset` 通常是被优化的。如果你想要为位命名而不是编号，可以使用 `set`（§12.5）或枚举（§2.4）。

`bitset` 可以用整数或字符串初始化：

```cpp
bitset<9> bs1 {"110001111"};
bitset<9> bs2 {0b1'1000'1111};   // 使用数字分隔符的二进制字面量（§1.4）
```

可以应用通常的按位运算符（§1.4）以及左移和右移运算符（`<<` 和 `>>`）：

```cpp
bitset<9> bs3 = ~bs1;   // 补码：bs3 == "001110000"
bitset<9> bs4 = bs1 & bs3;   // 全零
bitset<9> bs5 = bs1 << 2;    // 左移：bs5 = "000111100"
```

移位运算符（此处为 `<<`）会“移入”零。

操作 `to_ullong()` 和 `to_string()` 提供与构造函数相反的操作。例如，我们可以输出一个 `int` 的二进制表示：

```cpp
void binary(int i)
{
    bitset<8*sizeof(int)> b = i;   // 假设 8 位字节（另见 §17.7）
    cout << b.to_string() << '\n'; // 输出 i 的位
}
```

这将从左到右将位表示为 1 和 0，最高有效位在最左边，因此参数 123 将输出：

```
00000000000000000000000001111011
```

对于这个例子，直接使用 `bitset` 的输出运算符更简单：

```cpp
void binary2(int i)
{
    bitset<8*sizeof(int)> b = i;   // 假设 8 位字节（另见 §17.7）
    cout << b << '\n';             // 输出 i 的位
}
```

===== 第 16 页 =====

`bitset` 提供了许多用于使用和操作位集的函数，例如 `all()`、`any()`、`none()`、`count()`、`flip()`。

### 15.3.3 pair

函数返回两个值相当常见。有很多方法可以做到这一点，最简单且通常最好的是为此定义一个 `struct`。例如，我们可以返回一个值和一个成功指示器：

```cpp
struct My_res {
    Entry* ptr;
    Error_code err;
};

My_res complex_search(vector<Entry>& v, const string& s)
{
    Entry* found = nullptr;
    Error_code err = Error_code::found;
    // ... 在 v 中搜索 s ...
    return {found, err};
}

void user(const string& s)
{
    My_res r = complex_search(entry_table, s);   // 搜索 entry_table
    if (r.err != Error_code::good) {
        // ... 处理错误 ...
    }
    // ... 使用 r.ptr ...
}
```

我们可以争辩说将失败编码为尾后迭代器或 `nullptr` 更优雅，但这只能表达一种失败。通常，我们希望返回两个独立的值。为每一对值定义一个特定的具名 `struct` 通常效果很好，如果这些“值对”结构体及其成员的名称选择得当，可读性也相当高。然而，对于大型代码库，这可能导致名称和约定的激增，并且在需要一致命名的泛型代码中效果不佳。因此，标准库提供了 `pair` 作为对“值对”用例的通用支持。使用 `pair`，我们的简单示例变成：

```cpp
pair<Entry*, Error_code> complex_search(vector<Entry>& v, const string& s)
{
    Entry* found = nullptr;
    Error_code err = Error_code::found;
    // ... 在 v 中搜索 s ...
    return {found, err};
}

void user(const string& s)
{
    auto r = complex_search(entry_table, s);   // 搜索 entry_table
    if (r.second != Error_code::good) {
        // ... 处理错误 ...
    }
    // ... 使用 r.first ...
}
```

`pair` 的成员被命名为 `first` 和 `second`。从实现者的角度来看这是有意义的，但在应用程序代码中，我们可能想使用自己的名称。可以使用结构化绑定（§3.4.5）来处理这个问题：

```cpp
void user(const string& s)
{
    auto [ptr, success] = complex_search(entry_table, s);   // 搜索 entry_table
    if (success != Error_code::good) {
        // ... 处理错误 ...
    }
    // ... 使用 ptr ...
}
```

标准库的 `pair`（来自 `<utility>`）在标准库及其他地方非常频繁地用于“值对”的用例。例如，标准库算法 `equal_range` 返回一对指定子序列的迭代器，该子序列满足谓词：

```cpp
template<typename Forward_iterator, typename T, typename Compare>
pair<Forward_iterator, Forward_iterator>
equal_range(Forward_iterator first, Forward_iterator last, const T& val, Compare cmp);
```

给定已排序序列 `[first:last)`，`equal_range()` 将返回表示匹配谓词 `cmp` 的子序列的 pair。我们可以用它来在已排序的 `Record` 序列中搜索：

```cpp
auto less = [](const Record& r1, const Record& r2) { return r1.name < r2.name; };

void f(const vector<Record>& v)   // 假设 v 已按 "name" 字段排序
{
    auto [first, last] = equal_range(v.begin(), v.end(), Record{"Reg"}, less);
    for (auto p = first; p != last; ++p)   // 打印所有相等的记录
        cout << *p;   // 假设为 Record 定义了 <<
}
```

如果 `pair` 的元素支持，`pair` 会提供诸如 `=`、`==` 和 `<` 等运算符。类型推导使得创建 `pair` 变得容易，无需显式提及它的类型。例如：

```cpp
void f(vector<string>& v)
{
    pair p1 {v.begin(), 2};          // 一种方式
    auto p2 = make_pair(v.begin(), 2); // 另一种方式
    // ...
}
```

`p1` 和 `p2` 的类型都是 `pair<vector<string>::iterator, int>`。

当代码不需要泛型时，带有具名成员的简单 `struct` 通常能产生更易维护的代码。

### 15.3.4 tuple

像数组一样，标准库容器是同构的；也就是说，它们的所有元素都是单一类型的。然而，有时我们想要将不同类型元素的序列视为单个对象；也就是说，我们想要一个异构容器；`pair` 是一个例子，但并非所有此类异构序列都只有两个元素。标准库提供了 `tuple` 作为 `pair` 的推广，可以包含零个或多个元素：

```cpp
tuple t0 {};                                     // 空
tuple<string,int,double> t1 {"Shark", 123, 3.14}; // 显式指定类型
auto t2 = make_tuple(string{"Herring"}, 10, 1.23); // 类型推导为 tuple<string,int,double>
tuple t3 {"Cod"s, 20, 9.99};                     // 类型推导为 tuple<string,int,double>
```

`tuple` 的元素（成员）是独立的；它们之间不维护任何不变式（§4.3）。如果我们想要一个不变式，必须将 `tuple` 封装在一个强制该不变式的类中。

对于单个的、特定的用途，简单的 `struct` 通常是理想的，但在许多泛型用途中，`tuple` 的灵活性使我们不必定义许多 `struct`，代价是成员没有助记名称。`tuple` 的成员通过 `get` 函数模板访问。例如：

```cpp
string fish = get<0>(t1);    // 获取第一个元素："Shark"
int count = get<1>(t1);      // 获取第二个元素：123
double price = get<2>(t1);   // 获取第三个元素：3.14
```

`tuple` 的元素被编号（从零开始），并且传给 `get()` 的索引参数必须是常量。`get` 是一个函数模板，将索引作为模板值参数（§7.2.2）。

通过索引访问 `tuple` 的成员是通用的，但丑陋且有些容易出错。幸运的是，`tuple` 中具有唯一类型的元素可以通过其类型来“命名”：

```cpp
auto fish = get<string>(t1);   // 获取字符串："Shark"
auto count = get<int>(t1);     // 获取 int：123
auto price = get<double>(t1);  // 获取 double：3.14
```

我们也可以使用 `get<>` 进行写入：

```cpp
get<string>(t1) = "Tuna";   // 写入字符串
get<int>(t1) = 7;           // 写入 int
get<double>(t1) = 312;      // 写入 double
```

`tuple` 的大多数用法都隐藏在更高级别构造的实现中。例如，我们可以使用结构化绑定（§3.4.5）访问 `t1` 的成员：

```cpp
auto [fish, count, price] = t1;
cout << fish << ' ' << count << ' ' << price << '\n';   // 读取
fish = "Sea Bass";   // 写入
```

通常，这种绑定及其底层对 `tuple` 的使用用于函数调用：

```cpp
auto [fish, count, price] = todays_catch();
cout << fish << ' ' << count << ' ' << price << '\n';
```

`tuple` 的真正力量在于当您必须将数量未知、类型未知的元素作为对象存储或传递时。

显式地遍历 `tuple` 的元素有点麻烦，需要递归和函数体的编译时求值：

```cpp
template <size_t N = 0, typename... Ts>
constexpr void print(tuple<Ts...> tup)
{
    if constexpr (N < sizeof...(Ts)) {   // 还没到末尾？
        cout << get<N>(tup) << ' ';      // 打印第 N 个元素
        print<N+1>(tup);                 // 打印下一个元素
    }
}
```

这里，`sizeof...(Ts)` 给出 `Ts` 中元素的数量。使用 `print()` 很简单：

```cpp
print(t0);                                         // 无输出
print(t2);                                         // Herring 10 1.23
print(tuple{"Norah", 17, "Gavin", 14, "Anya", 9, "Courtney", 9, "Ada", 0});
```

===== 第 21 页 =====

像 `pair` 一样，如果其元素支持，`tuple` 也提供诸如 `=`、`==` 和 `<` 等运算符。此外，`pair` 与具有两个成员的 `tuple` 之间也存在转换。
