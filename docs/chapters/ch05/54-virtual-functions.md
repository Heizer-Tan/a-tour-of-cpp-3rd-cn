## 5.4 虚函数

虚函数（virtual function）是 C++ 实现运行时多态（runtime polymorphism）的机制。当通过基类的指针或引用调用虚函数时，实际调用的是*派生类*中对应的覆盖版本。

### 5.4.1 虚函数表

编译器通常通过*虚函数表*（virtual function table，简称 vtbl）来实现虚函数调用。每个包含虚函数的类都有一个 vtbl，其中存储了指向实际函数实现的指针。调用虚函数的开销大约是一次额外的指针解引用。

### 5.4.2 `override` 和 `final`

- `override`：明确标记一个函数覆盖了基类的虚函数，编译器会验证签名匹配
- `final`：阻止派生类进一步覆盖该虚函数

```cpp
class Base {
public:
    virtual void f() const;
    virtual void g();
};

class Derived : public Base {
public:
    void f() const override;    // 正确：覆盖 Base::f()
    void g() override final;    // 覆盖 Base::g()，且不允许进一步覆盖
};

class More : public Derived {
public:
    void f() const override;    // 正确
    // void g() override;       // 错误：Derived::g() 是 final
};
```

### 5.4.3 虚析构函数

如果类有虚函数，其析构函数通常也应该是虚的：

```cpp
class Base {
public:
    virtual ~Base() {}  // 虚析构函数
};
```

这确保了通过基类指针删除派生类对象时，派生类的析构函数会被正确调用。
