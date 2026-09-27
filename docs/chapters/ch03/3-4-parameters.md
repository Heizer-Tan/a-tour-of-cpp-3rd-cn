# 3.4 函数参数和返回值

在程序各部分之间传递信息，首要且推荐的方式是通过函数调用。执行任务所需的信息作为参数传入函数，产生的结果作为返回值传回。例如：

```cpp
int sum(const vector<int>& v)
{
    int s = 0;
    for (const int i : v)
        s += i;
    return s;
}

vector fib = {1, 2, 3, 5, 8, 13, 21};

int x = sum(fib);               // x 变成 53
```

函数之间还有其他传递信息的途径，例如全局变量（[§1.5](../ch01/1-5-scope-lifecycle.md)）以及类对象中的共享状态（[第5章](../ch05/index.md)）。全局变量是公认的错误来源，强烈不建议使用；状态通常应只在共同实现某个清晰抽象的函数之间共享（例如类的成员函数；[§2.3](../ch02/2-3-class.md)）。

既然向函数传入、传出信息如此重要，实现方式多种多样也就不足为奇。关键问题包括：

- 对象是被复制，还是被共享？
- 若被共享，是否可变？
- 对象是否被移动，从而留下一个“空对象”（[§6.2.2](../ch06/6-2-copy-move.md#6.2.2)）？

参数传递与返回值的默认行为都是“做一份拷贝”（[§1.9](../ch01/1-9-hardware.md)），但许多拷贝可以被隐式优化为移动。

在 `sum()` 例子中，结果 `int` 会从 `sum()` 中拷贝出来；但把可能非常大的 `vector` 拷贝进 `sum()` 既低效也无意义，因此参数按引用传递（用 `&` 表示；[§1.7](../ch01/1-7-pointers-arrays.md)）。

`sum()` 没有理由修改其实参。这种不可变性通过把 `vector` 参数声明为 `const`（[§1.6](../ch01/1-6-constants.md)）来表达，于是该 `vector` 按 const 引用传递。

## 3.4.1 参数传递

先考虑如何把值传入函数。默认是拷贝（“按值传递”）；若要引用调用者环境中的对象，则使用引用（“按引用传递”）。例如：

```cpp
void test(vector<int> v, vector<int>& rv)   // v 按值传递；rv 按引用传递
{
    v[1] = 99;          // 修改 v（局部变量）
    rv[2] = 66;         // 修改 rv 所引用的对象
}

int main()
{
    vector fib = {1, 2, 3, 5, 8, 13, 21};
    test(fib,fib);
    cout << fib[1] << ' ' << fib[2] << '\n';    // 打印 2 66
}
```

在乎性能时，我们通常对小值按值传递，对较大者按引用传递。这里的“小”指“拷贝起来确实很便宜的东西”。何谓“小”取决于机器体系结构，但“不超过两三个指针的大小”是个不错的经验法则。若对你的性能可能有显著影响，就去测量。

若因性能原因要按引用传递，又不需要修改实参，则像 `sum()` 例子那样按 const 引用传递。在普通好代码里，这是最常见的情况：既快又不易出错。

函数参数常常有一个默认值，也就是被认为更可取或最常见的值。我们可以通过默认函数参数来指定它。例如：

```cpp
void print(int value, int base =10);   // 以进制 base 打印 value

print(x,16);       // 十六进制
print(x,60);       // 六十进制（苏美尔人）
print(x);          // 使用默认：十进制
```

这在记法上比重载更简洁：

```cpp
void print(int value, int base);          // 以进制 base 打印 value

void print(int value)                     // 以十进制打印 value
{
    print(value,10);
}
```

使用默认参数意味着函数只有一个定义。这通常更利于理解和减小代码体积。当我们需要为不同类型用不同代码实现相同语义时，可以使用重载。

## 3.4.2 返回值

一旦算出了结果，就需要把它从函数中拿出来交还给调用者。返回值的默认方式同样是拷贝；对小对象来说这很理想。只有当我们想让调用者访问*并非*函数局部的东西时，才“按引用”返回。例如，`Vector` 可以通过下标运算把某个元素交给用户访问：

```cpp
class Vector {
public:
    // ...
    double& operator[](int i) { return elem[i]; }   // 返回对第 i 个元素的引用
private:
    double* elem;   // elem 指向含有 sz 个元素的数组
    // ...
};
```

`Vector` 的第 i 个元素独立于下标运算符的这次调用而存在，因此我们可以返回对它的引用。

另一方面，局部变量在函数返回时就会消失，因此不应返回指向它的指针或引用：

```cpp
int& bad()
{
    int x;
    // ...
    return x;  // 糟糕：返回对局部变量 x 的引用
}
```

幸运的是，所有主流 C++ 编译器都会捕捉 `bad()` 中这种明显错误。

返回引用或“小”类型的值是高效的，但如何把大量信息传出函数呢？考虑：

```cpp
Matrix operator+(const Matrix& x, const Matrix& y)
{
    Matrix res;
    // ... 对所有 res[i,j]，res[i,j] = x[i,j]+y[i,j] ...
    return res;
}

Matrix m1, m2;
// ...
Matrix m3 = m1+m2;         // 无拷贝
```

即便在现代硬件上，`Matrix` 也可能非常大、拷贝代价很高。因此我们并不拷贝，而是给 `Matrix` 一个移动构造函数（[§6.2.2](../ch06/6-2-copy-move.md#6.2.2)），廉价地把 `Matrix` 移出 `operator+()`。即便没有定义移动构造函数，编译器也常常能优化掉这次拷贝（省略拷贝），并在最终需要它的位置直接构造 `Matrix`。这称为*拷贝省略*（copy elision）。

我们不应倒退回去使用手工内存管理：

```cpp
Matrix* add(const Matrix& x, const Matrix& y)   // 复杂且易错的 20 世纪风格
{
    Matrix* p = new Matrix;
    // ... 对所有 *p[i,j]，*p[i,j] = x[i,j]+y[i,j] ...
    return p;
}

Matrix m1, m2;
// ...
Matrix* m3 = add(m1,m2);        // 只是拷贝一个指针
// ...
delete m3;                      // 很容易忘记
```

遗憾的是，通过返回指针来返回大对象在旧代码中很常见，也是难以定位的错误的主要来源。不要写这种代码。`Matrix` 的 `operator+()` 至少与 `Matrix` 的 `add()` 一样高效，却远更容易定义、更容易使用，也更不易出错。

若函数无法完成其应有任务，可以抛出异常（[§4.2](../ch04/4-2-exceptions.md)）。这有助于避免代码被“异常问题”的错误码检查弄得到处都是。

## 3.4.3 返回类型推导

函数的返回类型可以从其返回值推导出来。例如：

```cpp
auto mul(int i, double d) { return i*d; }   // 这里的 "auto" 表示“推导返回类型”
```

这很方便，尤其对泛型函数（[§7.3.1](../ch07/7-3-parameterized-operations.md#7.3.1)）和 lambda（[§7.3.3](../ch07/7-3-parameterized-operations.md#7.3.3)）而言；但应谨慎使用，因为推导出的类型并不提供稳定接口：改变函数（或 lambda）的实现就可能改变其类型。

## 3.4.4 后缀返回类型

为什么返回类型写在函数名和参数之前？原因主要是历史性的——Fortran、C 和 Simula 就是这么做的（现在仍是）。然而，有时我们需要先看参数才能确定结果的类型。返回类型推导是一例，但并非唯一。在本书范围之外的例子中，这个问题还会在命名空间（[§3.3](3-3-namespace.md)）、lambda（[§7.3.3](../ch07/7-3-parameterized-operations.md#7.3.3)）和概念（[§8.2](../ch08/8-2-concepts.md)）相关处出现。因此，当我们想显式写出返回类型时，允许把它加在参数列表之后。此时 `auto` 的含义是“返回类型稍后写出，或由推导得到”。例如：

```cpp
auto mul(int i, double d) -> double { return i*d; }   // 返回类型是 "double"
```

与变量一样（[§1.4.2](../ch01/1-4-types-variables.md#1.4.2)），我们可以用这种记法把名字排得更整齐。把这种后缀返回类型的写法，与 [§1.3](../ch01/1-3-functions.md) 中的版本对照：

```cpp
auto next_elem() -> Elem*;
auto exit(int) -> void;
auto sqrt(double) -> double;
```

我觉得后缀返回类型记法比传统前缀写法更合乎逻辑，但由于绝大多数代码仍用传统记法，本书也沿用传统写法。

## 3.4.5 结构化绑定

函数只能返回一个值，但该值可以是带有许多成员的类对象。这让我们能优雅地返回多个值。例如：

```cpp
struct Entry {
    string name;
    int value;
};

Entry read_entry(istream& is)   // 朴素的读取函数（更好的版本见 §11.5）
{
    string s;
    int i;
    is >> s >> i;
    return {s,i};
}

auto e = read_entry(cin);

cout << "{ " << e.name << " , " << e.value << " }\n";
```

这里用 `{s,i}` 构造 `Entry` 返回值。更好的读取写法见 [§11.5](../ch11/11-5-user-defined-io.md)。类似地，我们可以把 `Entry` 的成员“拆包”到局部变量中：

```cpp
auto [n,v] = read_entry(is);
cout << "{ " << n << " , " << v << " }\n";
```

`auto [n,v]` 声明了两个局部变量 `n` 和 `v`，其类型从 `read_entry()` 的返回类型推导。这种为类对象成员赋予局部名字的机制称为*结构化绑定*（structured binding）。

再看一个例子：

```cpp
map<string,int> m;
// ... 填充 m ...
for (const auto [key,value] : m)
    cout << "{" << key << "," << value << "}\n";
```

照例，我们可以给 `auto` 加上 `const` 和 `&`。例如：

```cpp
void incr(map<string,int>& m)   // 递增 m 中每个元素的值
{
    for (auto& [key,value] : m)
        ++value;
}
```

对没有私有数据的类使用结构化绑定时，绑定方式很直观：绑定中定义的名字个数必须与类对象的数据成员个数相同，且绑定引入的每个名字对应相应成员。与显式使用复合对象相比，目标代码质量没有差别。尤其是，使用结构化绑定并不意味着对 `struct` 做拷贝。此外，返回简单 `struct` 很少涉及拷贝，因为简单返回类型可以直接在需要它的位置构造（[§3.4.2](3-4-parameters.md#3.4.2)）。结构化绑定关乎的是如何最好地表达想法。

也可以处理通过成员函数访问的类。例如：

```cpp
complex<double> z = {1,2};
auto [re,im] = z+2;                   // re=3; im=2
```

`complex` 有两个数据成员，但其接口由访问函数（如 `real()` 和 `imag()`）组成。把 `complex<double>` 映射到两个局部变量（如 `re` 和 `im`）是可行且高效的，但具体做法超出了本书范围。
