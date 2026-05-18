# 13.3 迭代器类型

迭代器究竟是什么？任何一个迭代器都是某个类型的对象。但迭代器的类型很多：它需要携带在某个容器上做本职工作所需的足够信息。这些迭代器类型之间的差别可以像容器及其特定用途的差异那么大。

举例来说，`vector` 的迭代器完全可以是普通指针——指针恰好是一种很自然地指向 `vector` 元素的引用：

[图片描述：把 vector 的迭代器实现成指针]

或者，`vector` 迭代器也可以实现成「指向 `vector` 的指针 + 下标」：

[图片描述：把 vector 的迭代器实现成指针加索引]

使用此类迭代器就便于做范围检查。

相较之下，`list` 迭代器往往比一个指向节点的裸指针更复杂：链表中的一个节点通常并不知道表中下一个节点在哪儿，于是 `list` 迭代器可能表现为指向某个链接（link）的指针：

[图片描述：list 迭代器指向链表结点链接]

对所有迭代器共同的，是其语义以及操作的命名。例如对任一迭代器做 `++` 都得到指向下一元素的迭代器；`*` 则给出迭代器所指的元素。实际上，只要遵循几条诸如此类的规则的对象都可视作迭代器。迭代器是一种宽泛的观念（概念，§8.2）；不同类别的迭代器在标准库里也以概念的形式给出，例如 `forward_iterator`、`random_access_iterator`（§14.5）。

此外，使用者也很少真的关心某个迭代器的具体类型；每种容器都“知道”自己的迭代器类型，并用常规的别名把它们提供给外部：`iterator` 与 `const_iterator`。例如 `list<Entry>::iterator` 就是 `list<Entry>` 的一般迭代器类型；我们通常无须深究它的底层定义。

有些情形下迭代器并不是嵌套的成员类型，此时标准库提供 `iterator_t<X>`：凡是能为 `X` 定义迭代器的地方，`iterator_t<X>` 都好用。

## 13.3.1 输入迭代器与输出迭代器

迭代器是把容器中元素序列当作序列来处理时一个非常通用而有用的抽象；然而序列不只存在于容器里。输入流会产生一串值，而我们会向输出流写出一串值。因而迭代器的观念同样可以应用于输入与输出。

若要构造 `ostream_iterator`，需要指明要写往哪一个流，以及写入对象值的类型。例如：

```cpp
ostream_iterator<string> oo {cout}; // 把字符串写到 cout
```

对 `*oo` 赋值的效果就是把相应值写入 `cout`。例如：

```cpp
int main()
{
    *oo = "Hello, ";   // 相当于 cout << "Hello, "
    ++oo;
    *oo = "world!\n"; // 相当于 cout << "world!\n"
}
```

这又是写出那句经典问候语的另一种写法。这里的 `++oo` 是为了模仿通过指针向数组写入，从而让算法也能施加于流。例如：

```cpp
vector<string> v {"Hello", ", ", "World!\n"};
copy(v, oo);
```

类似地，`istream_iterator` 让我们把输入流当成只读容器来面对；同样需要指明输入源以及读取的值类型：

```cpp
istream_iterator<string> ii {cin};
```

输入迭代器总是成对出现来表示序列，因此还需要第二个迭代器标记输入结束；默认构造的 `istream_iterator` 就扮演这个角色：

```cpp
istream_iterator<string> eos {};
```

实践中，`istream_iterator` / `ostream_iterator` 通常并非手写直接用，而是作为参数传给算法。例如可以先读取文件，对读到的单词排序、去掉相邻重复，再写入另一个文件：

```cpp
int main()
{
    string from, to;
    cin >> from >> to;                     // 源文件名与目标文件名

    ifstream is {from};                    // 来自文件 `from` 的输入流
    istream_iterator<string> ii {is};      // 针对流的输入迭代器
    istream_iterator<string> eos {};       // 输入哨兵

    ofstream os {to};                      // 写入文件 `to` 的输出流
    ostream_iterator<string> oo {os, "\n"}; // 输出迭代器，带分隔串

    vector<string> b {ii, eos};            // 用输入初始化缓冲区 b
    sort(b);                               // 排序缓冲区
    unique_copy(b, oo);                    // 写到输出并去掉相邻重复

    return !is.eof() || !os;               // 返回错误状态（§1.2.1，§11.4）
}
```

这里用的是 `sort()` 与 `unique_copy()` 的范围版本；也可以写成 `sort(b.begin(), b.end())`——在老代码里更常见。

请记住（§9.3.2）：若要在同一作用域里同时使用传统迭代器版本与其范围版本，要么显式限定命名空间（例如 `ranges::copy`），要么通过 `using` 声明消除歧义：

```cpp
copy(v, oo);           // 可能产生歧义
ranges::copy(v, oo);   // OK

using std::ranges::copy;
copy(v, oo);           // OK（在本作用域中指范围版本）
```

`ifstream` 是可附着到文件的输入流（§11.7.2），`ofstream` 是可附着到文件的输出流。`ostream_iterator` 的第二个参数用来分隔输出的各个值。

坦诚地说，这个示例还可以写得更短：我们先读入 `vector`、排序，再写出并去掉重复。更优雅的思路是根本不必保存重复——把它们放进 `set` 即可：`set` 既不含重复元素，又自动保持有序（§12.5）。于是可以把原先两行基于 `vector` 的逻辑换成一行基于 `set`，并把 `unique_copy()` 换成更直接的 `copy()`：

```cpp
set<string> b {ii, eos}; // 从输入收集字符串
copy(b, oo);             // 写出缓冲区
```

`ii`、`eos`、`oo` 若只用一次，还可以进一步压缩：

```cpp
int main()
{
    string from, to;
    cin >> from >> to;

    ifstream is {from};
    ofstream os {to};

    set<string> b {istream_iterator<string>{is}, istream_iterator<string>{}};
    copy(b, ostream_iterator<string>{os, "\n"});

    return !is.eof() || !os;
}
```

是否要把程序压到这么短，归根结底关乎品味与经验。
