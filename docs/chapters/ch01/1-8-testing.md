# 1.8 测试

C++提供了一组用于实现条件判断和循环处理的常用语句，比如`if`语句、`switch`语句、`while`循环和`for`循环。例如，下面是一个简单的函数：它会提示用户输入信息，然后返回一个布尔值来表示用户的回答是什么。

```cpp
bool accept()
{
    cout << "Do you want to proceed (y or n)?\n";   // 输出问题
    char answer = 0;                                // 初始化一个值，无需显示
    cin >> answer;                                  // 读取回应
    if (answer == 'y')
        return true;
    return false;
}
```


与`<<`输出运算符相对应，`>>`运算符则用于输入操作；`cin`是标准输入流（第11章）。
`>>`运算符右侧的操作数类型决定了可以输入的数据类型，而该操作数本身就是输入操作的目标。
输出字符串末尾的`\n`字符表示换行符（§1.2.1节）。

请注意，`answer`的定义会出现在其应有的位置上（而不会提前出现）。声明可以出现在任何适合陈述出现的地方。

如果考虑到答案为“否”的情况，那么这个例子就可以进一步完善了。

```cpp
bool accept2()
{
    cout << "Do you want to proceed (y or n)?\n";   // 输出问题
    char answer = 0;                                // 初始化一个值，无需显示
    cin >> answer;                                  // 读取回应

    switch (answer) {
    case 'y':
        return true;
    case 'n':
        return false;
    default:
        cout << "I'll take that for a no.\n";
        return false;
    }
}
```

`switch`-语句的作用是將某个值与一组常量进行比较。这些常量被称为“`case`-标签”，它们必须各不相同。如果被比较的值与这些案例标签都不匹配，则会采用默认值。如果该值与所有案例标签都不匹配，且没有设置默认值，那么就不会有任何操作被执行。

我们不必通过从包含 `switch` 语句的函数中返回来结束该函数的执行。通常，我们只想让程序继续执行 `switch` 语句之后的代码。这时，可以使用 `break` 语句来实现。例如，可以考虑这样一个设计得相当巧妙，但同时又相当简单的解析器，它用于处理某种简单命令型视频游戏中的输入数据。

```cpp
void action()
{
    while (true) {
        cout << "enter action:\n";  // 询问动作
        string act;
        cin >> act;                 // 把字符串读入一个string
        Point delta {0,0};          // Point里存有一个{x,y}对

        for (char ch : act) {
            switch (ch) {
            case 'u':   // 上（up）
            case 'n':   // 北（north）
                ++delta.y;
                break;
            case 'r':   // 右（right）
            case 'e':   // 东（east）
                ++delta.x;
                break;
            // ... more actions ...
            default:
                cout << "I freeze!\n";
            }
            move(current+delta*scale);
            update_display();
        }
    }
}
```

与“`for`语句（S1.7）类似，`if`语句也可以用来声明一个变量并对该变量进行测试。例如：

```cpp
void do_something(vector<int>& v)
{
    if (auto n = v.size(); n!=0) {
        // ... 如果 n!=0 就走到这 ...
    }
    // ...
}
```


在这里，整数`n`被定义为在`if`语句中使用的变量，其初始值为`v.size()`。分号之后，会立即用`n!=0`这个条件来检测`n`是否不为零。在`if`语句中，如果在条件表达式中定义了某个变量，那么该变量在`if`语句的两个分支中都是有效的。

与`for`语句类似，在`if`语句的条件部分声明变量的目的也是为了限制变量的作用范围，从而提高代码的可读性并减少错误的发生。最常见的情况是测试某个变量是否等于`o`（或`nullptr`）。要做到这一点，只需省略对条件的明确表述即可。例如：

```cpp
void do_something(vector<int>& v)
{
    if (auto n = v.size()) {
        // ... 如果 n!=0 就走到这 ...
    }
    // ...
}
```

只要有可能，就尽量使用这种更简洁、更简单的表达方式吧。
