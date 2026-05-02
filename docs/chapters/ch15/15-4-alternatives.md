## 15.4 替代方案

### 15.4.1 `std::optional`（C++17）

`optional<T>` 表示"可能存在也可能不存在"的值，是空指针的安全替代：

```cpp
std::optional<int> find_even(const vector<int>& v)
{
    for (int x : v)
        if (x % 2 == 0)
            return x;
    return {};  // 或 return std::nullopt;
}

if (auto result = find_even({1, 3, 5, 6, 7})) {
    cout << "Found: " << *result << '\n';  // 6
} else {
    cout << "No even number found\n";
}
```

### 15.4.2 `std::variant`（C++17）

`variant<A, B, C>` 可以持有多种类型中的一种，是类型安全的联合体：

```cpp
std::variant<int, string, double> v;

v = 42;
int i = std::get<int>(v);           // 42

v = "hello"s;
string s = std::get<string>(v);     // "hello"

// 使用 visit 安全地访问
std::visit([](auto&& arg) {
    cout << arg << '\n';
}, v);
```

### 15.4.3 `std::any`（C++17）

`any` 可以持有任意可拷贝类型的值：

```cpp
std::any a = 42;
a = "hello"s;
a = 3.14;

// 需要知道类型才能取出
double d = std::any_cast<double>(a);
```
