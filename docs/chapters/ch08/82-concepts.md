## 8.2 概念

概念（concept）是对模板参数的一组编译时约束。它定义了类型必须支持哪些操作和属性。

### 8.2.1 定义概念

概念使用 `concept` 关键字定义：

```cpp
template<typename T>
concept Arithmetic = std::is_arithmetic_v<T>;

template<typename T>
concept Sortable = requires(T a, T b) {
    { a < b } -> std::convertible_to<bool>;  // a < b 必须返回可转换为 bool 的类型
    std::swap(a, b);                          // 必须支持 swap
};
```

`requires` 表达式用于指定类型必须支持的操作。

### 8.2.2 使用概念

概念可以用于约束模板参数：

```cpp
template<Arithmetic T>
T sum(const vector<T>& v)
{
    T s = 0;
    for (auto x : v) s += x;
    return s;
}

template<Sortable T>
void sort(vector<T>& v)
{
    std::sort(v.begin(), v.end());
}
```

也可以使用 `requires` 子句：

```cpp
template<typename T>
    requires Arithmetic<T>
T product(const vector<T>& v)
{
    T p = 1;
    for (auto x : v) p *= x;
    return p;
}
```

### 8.2.3 概念的优势

使用概念后，当模板参数不满足约束时，编译器会给出清晰的错误消息：

```
error: no matching function for call to 'sum'
note: candidate template ignored: constraints not satisfied
      because 'std::string' does not satisfy 'Arithmetic'
```

相比之下，不使用概念时，错误消息可能长达数十行，难以理解。

### 8.2.4 标准库概念

标准库在 `<concepts>` 头文件中提供了许多预定义概念：

- `std::integral<T>`：整数类型
- `std::floating_point<T>`：浮点类型
- `std::copyable<T>`：可拷贝的类型
- `std::movable<T>`：可移动的类型
- `std::regular<T>`：行为类似内置类型的类型
- `std::totally_ordered<T>`：支持全序比较的类型
