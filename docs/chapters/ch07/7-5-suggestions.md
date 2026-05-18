# 7.5 建议

[1] 使用模板来表达适用于多种参数类型的算法；[§7.1](7-1-introduction.md)；[CG: T.2]。
[2] 使用模板来表达容器；[§7.2](7-2-parameterized-types.md)；[CG: T.3]。
[3] 使用模板来提高代码的抽象层次；[§7.2](7-2-parameterized-types.md)；[CG: T.1]。
[4] 模板是类型安全的，但对于无约束模板，检查发生得太晚；[§7.2](7-2-parameterized-types.md)。
[5] 让构造函数或函数模板推导类模板实参类型；[§7.2.3](7-2-parameterized-types.md)。
[6] 使用函数对象作为算法的参数；[§7.3.2](7-3-parameterized-operations.md)；[CG: T.40]。
[7] 如果你只需要在一个地方使用简单的函数对象，请使用 lambda；[§7.3.2](7-3-parameterized-operations.md)。
[8] 虚成员函数不能是成员函数模板；[§7.3.1](7-3-parameterized-operations.md)。
[9] 使用 `finally()` 为缺少析构函数、却需要“善后动作”的类型提供类 RAII 行为；[§7.3.3.3](7-3-parameterized-operations.md)。
[10] 使用模板别名来简化记法并隐藏实现细节；[§7.4.2](7-4-template-mechanisms.md)。
[11] 使用 `if constexpr` 在不付出运行时开销的前提下提供不同实现；[§7.4.3](7-4-template-mechanisms.md)。
