# 15.4 替代方案

标准库提供了三种表示替代方案的类型：

**替代方案**

| 类型 | 描述 |
|------|------|
| `union` | 内置类型，持有一组替代方案中的一个值（§2.5） |
| `variant<T...>` | 指定的一组替代方案中的一个（在 `<variant>` 中） |
| `optional<T>` | 类型 `T` 的值或无值（在 `<optional>` 中） |
| `any` | 无界集合的替代类型中的一个值（在 `<any>` 中） |

这些类型为用户提供了相关的功能。不幸的是，它们没有提供统一的接口。

### 15.4.1 variant

`variant<A,B,C>` 通常是显式使用 `union`（§2.5）的更安全、更方便的替代方案。最简单的例子可能是返回一个值或一个错误码：

```cpp
variant<string, Error_code> compose_message(istream& s)
{
    string mess;
    // ... 从 s 读取并组合消息 ...
    if (no_problems)
        return mess;               // 返回一个字符串
    else
        return Error_code{some_problem};   // 返回一个 Error_code
}
```

===== 第 22 页 =====

当你用一个值赋值或初始化 `variant` 时，它会记住该值的类型。之后，我们可以查询 `variant` 持有哪种类型并提取该值。例如：

```cpp
auto m = compose_message(cin);

if (holds_alternative<string>(m)) {
    cout << get<string>(m);
}
else {
    auto err = get<Error_code>(m);
    // ... 处理错误 ...
}
```

这种风格吸引了一些不喜欢异常的人（见 §4.4），但还有更有趣的用途。例如，一个简单的编译器可能需要区分具有不同表示的不同类型的节点：

```cpp
using Node = variant<Expression, Statement, Declaration, Type>;

void check(Node* p)
{
    if (holds_alternative<Expression>(*p)) {
        Expression& e = get<Expression>(*p);
        // ...
    }
    else if (holds_alternative<Statement>(*p)) {
        Statement& s = get<Statement>(*p);
        // ...
    }
    // ... Declaration 和 Type ...
}
```

这种检查替代方案以决定适当行动的模式非常常见，且相对低效，因此它值得直接支持：

```cpp
void check(Node* p)
{
    visit(overloaded{
        [](Expression& e) { /* ... */ },
        [](Statement& s) { /* ... */ },
        // ... Declaration 和 Type ...
    }, *p);
}
```

这基本上等价于虚函数调用，但可能更快。与所有关于性能的声称一样，当性能至关重要时，应通过测量来验证这种“可能更快”。对于大多数用途，性能差异微不足道。

`overloaded` 类是必需的，而且奇怪的是，它不是标准的一部分。它是一个“魔法片段”，从一组参数（通常是 lambda）构建一个重载集：

```cpp
template<class... Ts>
struct overloaded : Ts... {   // 变参模板（§8.4）
    using Ts::operator()...;
};

template<class... Ts>
overloaded(Ts...) -> overloaded<Ts...>;   // 推导指引
```

然后，“访问者” `visit` 将 `()` 应用于 `overloaded` 对象，该对象根据重载规则选择最合适的 lambda 来调用。

推导指引是一种解决微妙歧义的机制，主要用于基础库中类模板的构造函数（§7.2.3）。

如果我们试图访问一个持有与预期不同类型值的 `variant`，会抛出 `bad_variant_access`。

### 15.4.2 optional

`optional<A>` 可以看作一种特殊的 `variant`（类似于 `variant<A, nothing>`），或者看作是“`A*` 要么指向一个对象要么为 `nullptr`”这一概念的泛化。

`optional` 对于可能返回或不返回对象的函数很有用：

```cpp
optional<string> compose_message(istream& s)
{
    string mess;
    // ... 从 s 读取并组合消息 ...
    if (no_problems)
        return mess;
    return {};   // 空的 optional
}
```

===== 第 24 页 =====

有了这个，我们可以写：

```cpp
if (auto m = compose_message(cin))
    cout << *m;   // 注意解引用 (*)
else {
    // ... 处理错误 ...
}
```

这吸引了一些不喜欢异常的人（见 §4.4）。注意 `*` 的不寻常用法。`optional` 被当作指向其对象的指针而不是对象本身来对待。

`optional` 中与 `nullptr` 等价的是空对象 `{}`。例如：

```cpp
int sum(optional<int> a, optional<int> b)
{
    int res = 0;
    if (a) res += *a;
    if (b) res += *b;
    return res;
}

int x = sum(17, 19);   // 36
int y = sum(17, {});   // 17
int z = sum({}, {});   // 0
```

如果我们试图访问一个不持有值的 `optional`，结果是未定义的；不会抛出异常。因此，`optional` 不保证类型安全。不要尝试：

```cpp
int sum2(optional<int> a, optional<int> b)
{
    return *a + *b;   // 自找麻烦
}
```
