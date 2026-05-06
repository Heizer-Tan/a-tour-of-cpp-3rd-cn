# 1.5 作用域和生命周期

声明的作用是将其名称引入到相应的作用域中。

- **局部作用域**（local scope）： 声明在函数(§1.3)或lambda表达式(§7.3.2)内部的名称， 被称为局部名称（local name）。这类名称的作用域从其声明位置开始，一直延续到该名称所在代码块的结尾。代码块是由一对大括号`{ }`来标记的。函数的参数名称也属于局部名称。

- **类作用域**（class scope）：定义在类(§2.2、§2.3、第5章)中，且在任何函数(§1.3)、 lambda表达式(§7.3.2)和enum类(§2.4)之外的名称，被称为成员名称（member name）——也叫类成员名称（class member name）。其作用域从容纳它的类声明的左花括号`{`开始，到这个类声明的末尾 `}`。

- **命名空间作用域**（namespace scope）：如果名称被定义在一个命名空间（namespace）(§3.3)里，且在任何函数(§1.3)、 lambda表达式(§7.3.2)、类(§2.2、§2.3、第5章)、和enum类(§2.4)之外，就称之为命名空间成员名称（namespace member name）。其作用域从声明所在位置开始，直至命名空间结尾。

未定义于任何其它结构内的名称，被称作全局名称（global name）， 位于全局命名空间（global namespace）中。

此外，某些对象可以不具名，例如临时变量，以及通过`new` (§5.2.2)创建的对象。例如：


```cpp
vector<int> vec;      // vec is global (a global vector of integers) 

void fct(int arg)       // fct is global (names a global function) 
                                 // arg is local (names an integer argument) 
{ 
        string motto {"Who dares wins"};    // motto is local 
        auto p = new Record{"Hume"};        // p points to an unnamed Record (created b 
        // ... 
} 

struct Record { 
       string name;    // name is a member of Record (a string member) 
       // ... 
};
```

对象在使用前必须先被构造（初始化），并将在其作用域末尾被销毁。对于命名空间中的对象，其销毁的时间点位于程序的终止。对成员来说，其销毁的时间点，由持有它的对象的销毁时间点确定。经由`new`创建的对象，将“存活”至被`delete`(§5.2.2)销毁为止。
