# 2.5 联合体

`union`（联合体）是一种特殊的结构体（`struct`），它的所有成员都分配在同一块内存地址上。因此，联合体所占用的空间大小取决于其中最大的那个成员变量所占用的空间。显然，联合体在任何时候只能保存其中一个成员变量的值。例如，假设有一个符号表条目，它用来存储一个名称和一个值。这个值可以是`Node*`类型的指针，也可以是`int`类型的整数。

```cpp
enum Type { ptr, num }; // 一个 Type 可以是ptr和num(§2.5)

struct Entry {
    string name;    // string是个标准库里的类型
    Type t;
    Node* p;        // 如果t==ptr，用p
    int i;          // 如果t==num，用i
};

void f(Entry* pe)
{
    if (pe->t == num)
        cout << pe->i;
    // ...
}
```

`p`和`i`这两个成员变量永远不会被同时使用，因此这种设计会浪费空间。只要指定它们都应属于同一个联合体，就能轻松解决这个问题，具体做法如下：

```cpp
union Value { 
    Node* p; 
    int i; 
};
```

现在，`Value::p`和`Value::i`被存储在每个`Value`对象相同的内存地址中。
这种空间优化方式对于那些需要处理大量数据的应用程序来说非常重要，因为在这种情况下，采用紧凑的数据存储方式就显得尤为关键。
该语言无法自动判断联合体中存储的是哪种类型的值，因此必须由程序员来负责判断。

```cpp
struct Entry { 
    string name; 
    Type t; 
    Value v;   // use v.p if t==Type::ptr; use v.i if t==Type::num 
}; 

void f(Entry* pe) 
{ 
    if (pe->t == Type::num) 
        cout << pe->v.i; 
    // ... 
}
```

保持类型字段与联合体中所存储的类型之间的对应关系其实相当容易出错。为了避免错误，我们可以将联合体及类型字段封装在一个类中，只有通过那些能够正确使用该联合体的成员函数，才能访问相关数据。在应用层面，这种抽象处理方式有助于避免错误的发生。依赖这种带有标记的联合体是一种常见且有效的做法。而使用“无标记”的联合体则应尽量避免。

标准库有个类型叫`std::variant`，使用它就可以避免绝大多数针对 `union` 的直接应用。`variant`存储一个值，该值的类型可以从一组类型中任选一个(§15.4.1)。 举个例子，`variant<Node*,int>`的值，可以是Node*或者int。

标准库中的`variant`类型可以用来避免大多数对`union`的直接使用。所谓`variant`，其实就是能够存储一组可选类型中某一种类型的值（§15.4.1）。例如，`variant<Node*,int>`这种类型可以存储`Node*`类型的值，也可以存储`int`类型的值。利用`variant`类型，原本需要用`union`来实现的代码，现在就可以用更简单的方式来实现了。比如，上面的`Entry`示例就可以这样改写：

```cpp
struct Entry {
    string name;
    variant<Node*,int> v;
};

void f(Entry* pe)
{
    if (holds_alternative<int>(pe->v))  // *pe的值是int类型吗？（参见§13.5.1）
        cout << get<int>(pe->v);        // 取（get）int值
    // ...
}
```

很多情况下，使用`variant`比`union`更简单也更安全。
