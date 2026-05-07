# 12.6 unordered_map

`map` 查找的成本是 O(log(n))，其�?`n` �?`map` 中元素的数量。这相当不错。例如，对于一个包�?1,000,000 个元素的 `map`，我们只需大约 20 次比较和间接寻址就能找到一个元素。然而，在许多情况下，我们可以使用哈希查找而不是使用排序函数（�?`<`）进行比较，从而做得更好。标准库的哈希容器被称为“无序的”，因为它们不需要排序函数�?
例如，我们可以使�?`<unordered_map>` 中的 `unordered_map` 来实现我们的电话簿：

```cpp
unordered_map<string, int> phone_book {
    {"David Hume", 123456},
    {"Karl Popper", 234567},
    {"Bertrand Arthur William Russell", 345678}
};
```

�?`map` 类似，我们可以对 `unordered_map` 使用下标操作�?
```cpp
int get_number(const string& s)
{
    return phone_book[s];
}
```

标准库为字符串以及其他内置类型和标准库类型提供了默认的哈希函数。如有必要，我们可以提供自己的哈希函数。可能最常见的需要自定义哈希函数的情况是当我们需要一个包含我们自己类型的无序容器时。哈希函数通常实现为函数对象（§7.3.2）。例如：

```cpp
struct Record {
    string name;
    int product_code;
    // ...
};

struct Rhash {   // �?Record 定义的哈希函�?    size_t operator()(const Record& r) const
    {
        return hash<string>{}(r.name) ^ hash<int>{}(r.product_code);
    }
};

unordered_set<Record, Rhash> my_set;   // 使用 Rhash 进行查找�?Record 集合
```

设计好的哈希函数是一门艺术，通常需要了解将应用于哪些数据。通过使用异或（`^`）组合现有哈希函数来创建新的哈希函数，既简单又通常非常有效。然而，要小心确保参与哈希的每个值确实有助于区分不同的值。例如，除非你可以为同一个产品代码有多个名称（或为同一个名称有多个产品代码），否则组合两个哈希值不会带来任何好处�?
我们可以通过将哈希操作定义为标准�?`hash` 的特化来避免显式传递它�?
```cpp
namespace std {   // �?Record 创建一个哈希函�?    template<> struct hash<Record> {
        using argument_type = Record;
        using result_type = size_t;

        result_type operator()(const Record& r) const
        {
            return hash<string>{}(r.name) ^ hash<int>{}(r.product_code);
        }
    };
}
```

注意 `map` �?`unordered_map` 之间的区别：

- `map` 需要一个排序函数（默认�?`<`），并产生一个有序序列�?- `unordered_map` 需要一个相等函数（默认�?`==`）；它不维护其元素之间的顺序�?
给定一个好的哈希函数，`unordered_map` 对于大型容器�?`map` 快得多。然而，如果哈希函数不佳，`unordered_map` 的最坏情况行为远�?`map` 糟糕�?