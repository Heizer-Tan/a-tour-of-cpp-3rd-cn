# 11.4 I/O 状态

`iostream` 有一个状态，我们可以检查它以确定操作是否成功。最常见的用法是读取一个值序列：

```cpp
vector<int> read_ints(istream& is)
{
    vector<int> res;
    for (int i; is >> i; )
        res.push_back(i);
    return res;
}
```

这从 `is` 读取，直到遇到不是整数的东西。那个东西通常是输入的结尾。这里发生的是：操作 `is >> i` 返回对 `is` 的引用，而测试 `iostream` 在流准备好进行下一个操作时返回 `true`。

===== 第 7 页 =====

一般来说，I/O 状态保存了读取或写入所需的所有信息，例如格式化信息（§11.6.2）、错误状态（例如是否已到达输入结尾？）以及使用何种缓冲。特别地，用户可以设置状态以反映发生了错误（§11.5），并且如果错误不严重，可以清除状态。例如，我们可以想象一个 `read_ints()` 的版本，它接受一个终止字符串：

```cpp
vector<int> read_ints(istream& is, const string& terminator)
{
    vector<int> res;
    for (int i; is >> i; )
        res.push_back(i);

    if (is.eof())               // 没问题：文件结束
        return res;
    if (is.fail()) {            // 读取整数失败；是否是终止字符串？
        is.clear();             // 将状态重置为 good()
        string s;
        if (is >> s && s == terminator)
            return res;
        is.setstate(ios_base::failbit);   // 向 is 的状态添加 fail()
    }
    return res;
}

auto v = read_ints(cin, "stop");
```
