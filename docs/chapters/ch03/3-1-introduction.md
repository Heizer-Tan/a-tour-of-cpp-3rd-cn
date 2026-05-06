# 3.1 引言

C++程序由许多独立开发的组成部分构成，比如函数（§1.2.1）、用户自定义类型（第2章内容）、类的层级（§5.5）以及模板（第7章）。要妥善管理这些复杂的组成部分，关键在于明确界定它们之间的交互关系。首要且最重要的步骤就是区分某个组件的接口与其实现方式。在C++语言中，接口是通过声明来体现的。一个声明就足以说明使用某个函数或类型时所需的一切信息。例如：

```cpp
double sqrt(double);    // 平方根函数接收double参数，返回double值

class Vector {
public:
    Vector(int s);
    double& operator[](int i);
    int size();

private:
    double* elem;   // elem指向一个数组，该数组承载sz个double
    int sz;
};
```
这里的要点是：函数体(函数的定义)可以放在“其他地方”。在这个例子中，我们希望`Vector`的定义也放在“其他地方”，不过这个问题我们稍后再讨论（抽象类型；§5.3）。`sqrt()`的定义如下：

```cpp
double sqrt(double d)   // sqrt()的定义
{
    // ... 数学课本里的算法 ...
}
```

对于`Vector`来说，我们需要定义全部三个成员函数：

```cpp
Vector::Vector(int s)               // 构造函数的定义
    :elem{new double[s]}, sz{s}     // 初始化成员变量
{
}

double& Vector::operator[](int i)   // 取下标运算符的定义
{
    return elem[i];
}

int Vector::size()                  // size()的定义
{
    return sz;
}
```

我们必须自行定义`Vector`相关的函数，而`sqrt()`则不必如此，因为它属于标准库的组成部分。不过，这其实并没有什么区别：所谓“标准库”，不过是用与我们相同的编程语言编写而成的“其他代码”罢了。

一个实体，比如一个函数，可以有多个声明方式，但定义却只能有一个。
