# 13.4 谓词

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

此前的例子中，算法对每个元素要做什么往往是“内置固定”的；但我们常常希望把这项动作参数化。譬如 `find`（§13.2，§13.5）提供了查找特定值的便捷途径；更一般的变体则是查找满足某个要求的元素——这类要求称为**谓词**（predicate）。

例如我们可能想在 `map` 里寻找第一个值大于 `42` 的项。`map` 的元素序列可以视作一串 `(key, value)` 对，于是能在 `map<string,int>` 上搜索第一个满足“`int` 部分大于 `42`”的 `pair<const string,int>`：

```cpp
void f(map<string, int>& m)
{
    auto p = find_if(m, Greater_than{42});
    // ...
}
```

这里的 `Greater_than` 是函数对象（§7.3.2），内部保存要与 `map` 元素（类型为 `pair<string,int>`）比较的阈值：

```cpp
struct Greater_than {
    int val;
    Greater_than(int v) : val{v} {}
    bool operator()(const pair<string, int>& r) const { return r.second > val; }
};
```

等价地，也可以用 lambda（§7.3.3）：

```cpp
auto p = find_if(m, [](const auto& r) { return r.second > 42; });
```

谓词不应修改它所作用的元素。
