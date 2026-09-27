# 15.4 替代方案

标准库提供三种类型来表达替代：

**替代方案**

| 类型 | 说明 |
|------|------|
| `union` | 内置类型，保存一组替代中的一个（[§2.5](../ch02/2-5-union.md)） |
| `variant<T...>` | 指定的一组替代中的一个（在 `<variant>` 中） |
| `optional<T>` | 类型为 `T` 的值，或没有值（在 `<optional>` 中） |
| `any` | 无界替代类型集合中某一类型的值（在 `<any>` 中） |

这些类型为用户提供相关功能。遗憾的是，它们并不提供统一的接口。

## 15.4.1 `variant`

`variant<A,B,C>` 常常是显式使用 `union`（[§2.5](../ch02/2-5-union.md)）的更安全、更方便的替代。可能最简单的例子是返回一个值或一个错误码：

```cpp
variant<string,Error_code> compose_message(istream& s)
{
    string mess;
    // ... 从 s 读取并拼装消息 ...
    if (no_problems)
        return mess;                                               // 返回 string
    else
        return Error_code{some_problem};        // 返回 Error_code
}
```

当你用一个值赋值或初始化 `variant` 时，它会记住该值的类型。之后，我们可以查询 `variant` 持有何种类型并取出该值。例如：

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

这种风格对某些不喜欢异常的人有吸引力（见 [§4.4](../ch04/4-4-alternatives.md)），但还有更有趣的用途。例如，一个简单的编译器可能需要区分具有不同表示的不同种类的节点：

```cpp
using Node = variant<Expression,Statement,Declaration,Type>;

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
    // ... Declaration 与 Type ...
}
```

这种检查各个替代以决定适当动作的模式如此常见，又相对低效，因而值得直接支持：

```cpp
void check(Node* p)
{
    visit(overloaded {
        [](Expression& e) { /* ... */ },
        [](Statement& s) { /* ... */ },
        // ... Declaration 与 Type ...
    }, *p);
}
```

这基本上等价于虚函数调用，但可能更快。与所有关于性能的断言一样，当性能关键时，这一“可能更快”应当用测量来核实。对多数用途，性能差异并不显著。

`overloaded` 类是必要的，而且奇怪的是，它并非标准的。它是一块“魔法”，从一组实参（通常是 lambda）构造一个重载集：

```cpp
template<class... Ts>
struct overloaded : Ts... {            // 可变参数模板（[§8.4](../ch08/8-4-variadic-templates.md)）
    using Ts::operator()...;
};

template<class... Ts>
overloaded(Ts...) -> overloaded<Ts...>;    // 推导指引
```

然后“访问者”`visit` 把 `()` 作用到该重载对象上，后者按重载规则选出最合适的 lambda 来调用。

推导指引是一种消解微妙歧义的机制，主要用于基础库中类模板的构造函数（[§7.2.3](../ch07/7-2-parameterized-types.md#7.2.3)）。

若我们试图访问持有与期望类型不同的 `variant`，会抛出 `bad_variant_access`。

## 15.4.2 `optional`

可以把 `optional<A>` 看成一种特殊的 `variant`（类似 `variant<A,nothing>`），或看成“`A*` 要么指向对象要么为 `nullptr`”这一观念的推广。

`optional` 对那些可能返回对象也可能不返回对象的函数很有用：

```cpp
optional<string> compose_message(istream& s)
{
    string mess;

    // ... 从 s 读取并拼装消息 ...

    if (no_problems)
        return mess;
    return {};          // 空 optional
}
```

有了它，我们可以写：

```cpp
if (auto m = compose_message(cin))
    cout << *m;               // 注意解引用（*）
else {
    // ... 处理错误 ...
}
```

这对某些不喜欢异常的人有吸引力（见 [§4.4](../ch04/4-4-alternatives.md)）。注意对 `*` 的奇特用法。`optional` 被当作指向其对象的指针，而非对象本身。

与 `nullptr` 对应的 `optional` 是空对象 `{}`。例如：

```cpp
int sum(optional<int> a, optional<int> b)
{
    int res = 0;
    if (a) res += *a;
    if (b) res += *b;
    return res;
}

int x = sum(17, 19);         // 36
int y = sum(17, {});          // 17
int z = sum({}, {});            // 0
```

若我们试图访问不持有值的 `optional`，结果是未定义的；不会抛出异常。因此，`optional` 并不保证类型安全。不要尝试：

```cpp
int sum2(optional<int> a, optional<int> b)
{
    return *a + *b;     // 自讨苦吃
}
```

## 15.4.3 `any`

`any` 可以持有任意类型，并知道它持有何种类型（如果有的话）。它基本上是不受约束的 `variant` 版本：

```cpp
any compose_message(istream& s)
{
    string mess;

    // ... 从 s 读取并拼装消息 ...

    if (no_problems)
        return mess;                        // 返回 string
    else
        return error_number;          // 返回 int
}
```

当你用一个值赋值或初始化 `any` 时，它会记住该值的类型。之后，我们可以通过断言该值的期望类型来取出 `any` 所持有的值。例如：

```cpp
auto m = compose_message(cin);
string& s = any_cast<string>(m);
cout << s;
```

若我们试图访问持有与期望类型不同的 `any`，会抛出 `bad_any_access`。
