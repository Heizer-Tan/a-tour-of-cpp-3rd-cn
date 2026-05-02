## 7.3 参数化操作

### 7.3.1 函数模板

函数模板允许我们编写独立于类型的算法：

```cpp
template<typename Container, typename Value>
Value sum(const Container& c, Value v)
{
    for (auto x : c)
        v += x;
    return v;
}

void user(const vector<int>& vi, const list<double>& ld)
{
    int x = sum(vi, 0);         // 对 int 向量求和
    double y = sum(ld, 0.0);    // 对 double 列表求和
}
```

函数模板的模板参数通常可以从函数参数中推导出来，无需显式指定。

### 7.3.2 函数对象（函子）

模板常用于定义*函数对象*（function objects，也叫*函子* functors）。函数对象是重载了 `operator()` 的类的实例：

```cpp
template<typename T>
class Less_than {
    const T val;
public:
    Less_than(const T& v) : val{v} {}
    bool operator()(const T& x) const { return x < val; }
};

Less_than<int> lti {42};        // lti(i) 比较 i < 42
Less_than<string> lts {"Backus"}; // lts(s) 比较 s < "Backus"
```

函数对象可以携带状态，这使得它们比普通函数指针更灵活。

### 7.3.3 Lambda 表达式

Lambda 表达式（[§8.3](../ch08/8-3-lambdas.md)）本质上是匿名函数对象的语法糖：

```cpp
auto less_than = [](int x, int y) { return x < y; };
```

Lambda 可以捕获局部变量，这使得它们在算法中非常方便：

```cpp
int threshold = 42;
auto pred = [&](int x) { return x < threshold; };
```
