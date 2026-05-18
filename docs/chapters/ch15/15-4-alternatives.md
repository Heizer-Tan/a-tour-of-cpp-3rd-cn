# 15.4 表示“多选一”的类型

标准库提供三类类型来表达“若干候选之一”：

**备选类型一览**

| 类型 | 说明 |
|------|------|
| `union` | 语言内置：保存若干候选字段之一（§2.5） |
| `variant<T...>` | `<variant>`：保存若干指定类型之一 |
| `optional<T>` | `<optional>`：要么保存 `T`，要么为空 |
| `any` | `<any>`：保存某个未知类型的值 |

它们彼此相关，遗憾的是接口并不统一。

## 15.4.1 `variant`

`variant<A, B, C>` 常常比显式 `union`（§2.5）更安全也更顺手。最简单的用法莫过于返回“要么成功值要么错误码”：

```cpp
variant<string, Error_code> compose_message(istream& s)
{
    string mess;
    // ... 从 s 读取并拼装消息 ...
    if (no_problems)
        return mess;
    else
        return Error_code{some_problem};
}
```

赋值或初始化 `variant` 时，它会记住当前持有的类型；随后可以查询并向正确的类型索取值：

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

有人会偏爱这种风格胜过异常（§4.4）；此外还有更丰富用途——例如简易编译器要把不同节点区别开来：

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
    // ... Declaration 与 Type ...
}
```

这种手写分支的模式既啰嗦也相对低效，值得单独封装——于是可以用 `visit`：

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

它在语义上接近虚函数调用，有时还可能更快；但若性能关键仍需实测。多数差异并不重要。

`overloaded` 必不可少却又诡异：**它不是标准类型**。它像魔法一样把一组候选（通常是 lambda）捏成一个可调对象：

```cpp
template<class... Ts>
struct overloaded : Ts... {
    using Ts::operator()...;
};

template<class... Ts>
overloaded(Ts...) -> overloaded<Ts...>; // 推导指引
```

随后 `visit` 会把 `()` 作用在该对象上，按常规重载决议挑出最合适的 lambda。

推导指引主要用来消解微妙的歧义，是基础库类模板构造的重要工具（§7.2.3）。

若试图取出并非当前激活类型的分支，`variant` 会抛出 `bad_variant_access`。

## 15.4.2 `optional`

可以把 `optional<A>` 看成特殊的 `variant`（类似 `variant<A, monostate>`），或是“指向 `A` 的指针但不会出现所有权问题”的泛化。

对那些“可能有返回值也可能没有”的函数很有帮助：

```cpp
optional<string> compose_message(istream& s)
{
    string mess;
    // ... 读取 ...
    if (no_problems)
        return mess;
    return {}; // 空 optional
}
```

调用方可写成：

```cpp
if (auto m = compose_message(cin))
    cout << *m;
else {
    // ...
}
```

这也迎合厌恶异常的程序员（§4.4）。注意这里的 `*`：`optional` 更像指针语义而非对象本体。

“空指针”对应 `{}`。例如：

```cpp
int sum(optional<int> a, optional<int> b)
{
    int res = 0;
    if (a)
        res += *a;
    if (b)
        res += *b;
    return res;
}

int x = sum(17, 19); // 36
int y = sum(17, {}); // 17
int z = sum({}, {}); // 0
```

倘若在没有值的时候强行访问 `optional`，结果是未定义行为——并不会抛异常。因此它谈不上百分之百类型安全；千万别写：

```cpp
int sum2(optional<int> a, optional<int> b)
{
    return *a + *b; // 自讨苦吃
}
```

## 15.4.3 `any`

`any` 容纳任意类型，并且知道自己装着谁——可以理解成“不加模板约束的 `variant`”：

```cpp
any compose_message(istream& s)
{
    string mess;
    // ...
    if (no_problems)
        return mess;
    else
        return error_number;
}
```

赋值之后同样可以按断言的类型取出：

```cpp
auto m = compose_message(cin);
string& s = any_cast<string>(m);
cout << s;
```

若类型不匹配，`any_cast` 抛出 `bad_any_access`。
