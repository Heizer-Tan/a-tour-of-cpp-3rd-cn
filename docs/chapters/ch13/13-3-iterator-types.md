# 13.3 迭代器类型

迭代器到底是什么？任何特定的迭代器都是某个类型的对象。然而，有许多不同的迭代器类型——迭代器需要持有为特定容器类型执行其工作所需的信息。这些迭代器类型可以和容器以及它们所服务的专门需求一样不同。例如，`vector` 的迭代器可以是一个普通的指针，因为指针是指向 `vector` 元素的一种相当合理的方式：

[图片描述：vector 的迭代器作为指针]

或者，`vector` 迭代器可以实现为指向 `vector` 的指针加上一个索引：

[图片描述：vector 迭代器实现为指针+索引]

使用这样的迭代器可以进行范围检查。

`list` 迭代器必须比指向元素的简单指针更复杂，因为 `list` 的元素通常不知道其下一个元素的位置。因此，`list` 迭代器可能是一个指向链接的指针：

[图片描述：list 迭代器指向链接节点]

所有迭代器的共同点是它们的语义和操作的命名。例如，对任何迭代器应用 `++` 都会产生一个指向下一个元素的迭代器。类似地，`*` 产生迭代器所指的元素。事实上，任何遵守这样几条简单规则的对象都是一个迭代器。迭代器是一个通用的概念（§8.2），不同类型的迭代器作为标准库概念提供，例如 `forward_iterator` 和 `random_access_iterator`（§14.5）。此外，用户很少需要知道特定迭代器的类型；每个容器“知道”其迭代器类型，并在常规名称 `iterator` 和 `const_iterator` 下提供它们。例如，`list<Entry>::iterator` 是 `list<Entry>` 的通用迭代器类型。我们很少需要担心该类型是如何定义的细节。

在某些情况下，迭代器不是成员类型，因此标准库提供了 `iterator_t<X>`，它在定义了 `X` 的迭代器的任何地方都能工作。

### 13.3.1 流迭代器

===== 第 10 页 =====

迭代器是处理容器中元素序列的通用且有用的概念。然而，容器并不是唯一存在元素序列的地方。例如，输入流产生一个值序列，我们将一个值序列写入输出流。因此，迭代器的概念可以有效地应用于输入和输出。

要创建 `ostream_iterator`，我们需要指定将使用哪个流以及写入该流的对象类型。例如：

```cpp
ostream_iterator<string> oo {cout};   // 将字符串写入 cout
```

对 `*oo` 赋值的效果是将该值写入 `cout`。例如：

```cpp
int main()
{
    *oo = "Hello, ";   // 相当于 cout << "Hello, "
    ++oo;
    *oo = "world!\n";  // 相当于 cout << "world!\n"
}
```

这是向标准输出写入规范消息的另一种方式。`++oo` 是为了模拟通过指针写入数组。这样，我们就可以在流上使用算法。例如：

```cpp
vector<string> v{"Hello", ", ", "World!\n"};
copy(v, oo);
```

类似地，`istream_iterator` 允许我们将输入流视为只读容器。同样，我们必须指定要使用的流和期望的值类型：

```cpp
istream_iterator<string> ii {cin};
```

输入迭代器成对使用以表示一个序列，因此我们必须提供一个 `istream_iterator` 来指示输入的结束。这是默认的 `istream_iterator`：

```cpp
istream_iterator<string> eos {};
```

===== 第 11 页 =====

通常，`istream_iterator` 和 `ostream_iterator` 不直接使用。相反，它们作为参数提供给算法。例如，我们可以编写一个简单的程序来读取一个文件，对读取的单词进行排序，消除重复项，并将结果写入另一个文件：

```cpp
int main()
{
    string from, to;
    cin >> from >> to;                     // 获取源文件和目标文件名

    ifstream is {from};                    // 文件 "from" 的输入流
    istream_iterator<string> ii {is};      // 流的输入迭代器
    istream_iterator<string> eos {};       // 输入哨兵

    ofstream os {to};                      // 文件 "to" 的输出流
    ostream_iterator<string> oo {os, "\n"}; // 输出迭代器，并带有分隔符

    vector<string> b {ii, eos};            // b 是从输入初始化的 vector
    sort(b);                               // 对缓冲区进行排序

    unique_copy(b, oo);                    // 将缓冲区复制到输出，丢弃重复项

    return !is.eof() || !os;               // 返回错误状态（§1.2.1, §11.4）
}
```

这里使用了 `sort()` 和 `unique_copy()` 的 range 版本。我也可以直接使用迭代器，例如 `sort(b.begin(), b.end())`，这在旧代码中很常见。

请记住，要同时使用标准库算法的传统迭代器版本和其范围版本，我们需要显式限定范围版本的调用，或者使用 `using` 声明（§9.3.2）：

```cpp
copy(v, oo);                    // 可能有歧义
ranges::copy(v, oo);            // OK
using ranges::copy;             // 从此处开始 copy(v, oo) 是 OK 的
copy(v, oo);                    // OK
```

===== 第 12 页 =====

`ifstream` 是可以附加到文件的 `istream`（§11.7.2），而 `ofstream` 是可以附加到文件的 `ostream`。`ostream_iterator` 的第二个参数用于分隔输出值。

实际上，这个程序比需要的要长。我们将字符串读入一个 `vector`，然后对它们进行 `sort()`，然后将它们写出，并消除重复项。一个更优雅的解决方案是根本不存储重复项。这可以通过将字符串保存在一个 `set` 中来实现，因为 `set` 不会保留重复项，并且会保持其元素有序（§12.5）。这样，我们可以用一行使用 `set` 的代码替换两行使用 `vector` 的代码，并用更简单的 `copy()` 替换 `unique_copy()`：

```cpp
set<string> b {ii, eos};   // 从输入收集字符串
copy(b, oo);               // 将缓冲区复制到输出
```

我们只使用了一次名称 `ii`、`eos` 和 `oo`，因此我们可以进一步缩减程序的大小：

```cpp
int main()
{
    string from, to;
    cin >> from >> to;   // 获取源文件和目标文件名

    ifstream is {from};  // 文件 "from" 的输入流
    ofstream os {to};    // 文件 "to" 的输出流

    set<string> b {istream_iterator<string>{is}, istream_iterator<string>{}};   // 从输入收集字符串
    copy(b, ostream_iterator<string>{os, "\n"});   // 将缓冲区复制到输出

    return !is.eof() || !os;   // 返回错误状态（§1.2.1, §11.4）
}
```

这种最后的简化是否提高了可读性，取决于个人品味和经验。

## 13.4 谓词的使用