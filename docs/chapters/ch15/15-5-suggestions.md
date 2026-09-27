# 15.5 建议

[1] 库不见得庞大或复杂才有用；[§16.1](../ch16/16-1-introduction.md)。

[2] 资源是任何必须先获取再（显式或隐式）释放的东西；[§15.2.1](15-2-pointers.md#15.2.1)。

[3] 用资源句柄管理资源（RAII）；[§15.2.1](15-2-pointers.md#15.2.1)；[CG: R.1]。

[4] `T*` 的问题是它可以被用来表示任何东西，因而我们难以确定一个“裸”指针的用途；[§15.2.1](15-2-pointers.md#15.2.1)。

[5] 用 `unique_ptr` 指称多态类型的对象；[§15.2.1](15-2-pointers.md#15.2.1)；[CG: R.20]。

[6] （仅）用 `shared_ptr` 指称共享对象；[§15.2.1](15-2-pointers.md#15.2.1)；[CG: R.20]。

[7] 偏爱具有特定语义的资源句柄，而非智能指针；[§15.2.1](15-2-pointers.md#15.2.1)。

[8] 局部变量就够用时，不要用智能指针；[§15.2.1](15-2-pointers.md#15.2.1)。

[9] 偏爱 `unique_ptr` 而非 `shared_ptr`；[§6.3](../ch06/6-3-resource-mgmt.md)，[§15.2.1](15-2-pointers.md#15.2.1)。

[10] 仅在需要转移所有权职责时，才把 `unique_ptr` 或 `shared_ptr` 用作实参或返回值；[§15.2.1](15-2-pointers.md#15.2.1)；[CG: F.26] [CG: F.27]。

[11] 用 `make_unique()` 构造 `unique_ptr`；[§15.2.1](15-2-pointers.md#15.2.1)；[CG: R.22]。

[12] 用 `make_shared()` 构造 `shared_ptr`；[§15.2.1](15-2-pointers.md#15.2.1)；[CG: R.23]。

[13] 偏爱智能指针而非垃圾回收；[§6.3](../ch06/6-3-resource-mgmt.md)，[§15.2.1](15-2-pointers.md#15.2.1)。

[14] 偏爱 `span`，而非指针加计数接口；[§15.2.2](15-2-pointers.md#15.2.2)；[CG: F.24]。

[15] `span` 支持范围 `for`；[§15.2.2](15-2-pointers.md#15.2.2)。

[16] 需要具有 `constexpr` 大小的序列时，使用 `array`；[§15.3.1](15-3-containers.md#15.3.1)。

[17] 偏爱 `array` 而非内置数组；[§15.3.1](15-3-containers.md#15.3.1)；[CG: SL.con.2]。

[18] 若你需要 `N` 个比特，且 `N` 未必等于某个内置整数类型的比特数，使用 `bitset`；[§15.3.2](15-3-containers.md#15.3.2)。

[19] 不要过度使用 `pair` 与 `tuple`；具名 `struct` 往往带来更可读的代码；[§15.3.3](15-3-containers.md#15.3.3)。

[20] 使用 `pair` 时，用模板实参推导或 `make_pair()` 避免多余的类型说明；[§15.3.3](15-3-containers.md#15.3.3)。

[21] 使用 `tuple` 时，用模板实参推导或 `make_tuple()` 避免多余的类型说明；[§15.3.3](15-3-containers.md#15.3.3)；[CG: T.44]。

[22] 偏爱 `variant`，而非显式使用 `union`；[§15.4.1](15-4-alternatives.md#15.4.1)；[CG: C.181]。

[23] 用 `variant` 在一组替代中做选择时，考虑使用 `visit()` 与 `overloaded()`；[§15.4.1](15-4-alternatives.md#15.4.1)。

[24] 若 `variant`、`optional` 或 `any` 可能有多个替代，访问前先检查标签；[§15.4](15-4-alternatives.md)。
