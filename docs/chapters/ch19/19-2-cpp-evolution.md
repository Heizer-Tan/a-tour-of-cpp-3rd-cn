# 19.2 C++ 特性演变

在这里，我列出了为 C++11、C++14、C++17 和 C++20 标准添加到 C++ 的语言特性和标准库组件。

## 19.2.1 C++11 语言特性

看着一列语言特性可能会令人相当困惑。请记住，语言特性不是为了孤立地使用。特别是，C++11 中的大多数新特性在脱离旧特性提供的框架时是没有意义的。

[1] 使用 `{}` 列表的统一和通用初始化（[§1.4.2](../ch01/1-4-types-variables.md#1.4.2)，[§5.2.3](../ch05/5-2-concrete-types.md#5.2.3)）
[2] 从初始化器推导类型：`auto`（[§1.4.2](../ch01/1-4-types-variables.md#1.4.2)）
[3] 防止窄化转换（[§1.4.2](../ch01/1-4-types-variables.md#1.4.2)）
[4] 泛化且保证的常量表达式：`constexpr`（[§1.6](../ch01/1-6-constants.md)）
[5] 范围 `for` 语句（[§1.7](../ch01/1-7-pointers-arrays.md)）
[6] 空指针关键字：`nullptr`（[§1.7.1](../ch01/1-7-pointers-arrays.md#1.7.1)）
[7] 作用域和强类型枚举：`enum class`（[§2.4](../ch02/2-4-enum.md)）
[8] 编译时断言：`static_assert`（[§4.5.2](../ch04/4-5-assertions.md#4.5.2)）
[9] 语言将 `{}` 列表映射到 `std::initializer_list`（[§5.2.3](../ch05/5-2-concrete-types.md#5.2.3)）
[10] 右值引用，启用移动语义（[§6.2.2](../ch06/6-2-copy-move.md#6.2.2)）
[11] Lambda 表达式（[§7.3.3](../ch07/7-3-parameterized-operations.md#7.3.3)）
[12] 变参模板（[§7.4.1](../ch07/7-4-template-mechanisms.md#7.4.1)）
[13] 类型和模板别名（[§7.4.2](../ch07/7-4-template-mechanisms.md#7.4.2)）
[14] Unicode 字符
[15] `long long` 整数类型
[16] 对齐控制：`alignas` 和 `alignof`
[17] 使用表达式的类型作为声明中的类型的能力：`decltype`
[18] 原始字符串字面量（[§10.4](../ch10/10-4-regular-expressions.md)）
[19] 后缀返回类型语法（[§3.4.4](../ch03/3-4-parameters.md#3.4.4)）
[20] 属性的语法和两个标准属性：`[[carries_dependency]]` 和 `[[noreturn]]`
[21] 阻止异常传播的方式：`noexcept` 说明符（[§4.4](../ch04/4-4-alternatives.md)）
[22] 测试表达式中可能抛出异常：`noexcept` 运算符
[23] C99 特性：扩展整数类型；窄/宽字符串的连接；`__STDC_HOSTED__`；`_Pragma(X)`；变参宏和空宏参数
[24] `__func__` 作为保存当前函数名称的字符串的名称
[25] 内联命名空间
[26] 委托构造函数
[27] 类内成员初始化器（[§6.1.3](../ch06/6-1-introduction.md#6.1.3)）
[28] 控制默认：`default` 和 `delete`（[§6.1.1](../ch06/6-1-introduction.md#6.1.1)）
[29] 显式转换运算符
[30] 用户定义字面量（[§6.6](../ch06/6-6-user-defined-literals.md)）
[31] 对模板实例化的更显式控制：`extern` 模板
[32] 函数模板的默认模板参数
[33] 继承构造函数（[§12.2.2](../ch12/12-2-vector.md#12.2.2)）
[34] 重写控制：`override`（[§5.5](../ch05/5-5-hierarchies.md)）和 `final`
[35] 更简单、更通用的 SFINAE（替换失败不是错误）规则
[36] 内存模型（[§18.1](../ch18/18-1-introduction.md)）
[37] 线程局部存储：`thread_local`

关于 C++98 在 C++11 中的变化的更完整描述，请参见 [Stroustrup,2013]。

## 19.2.2 C++14 语言特性

[1] 函数返回类型推导（[§3.4.3](../ch03/3-4-parameters.md#3.4.3)）
[2] 改进的 `constexpr` 函数，例如允许 `for` 循环（[§1.6](../ch01/1-6-constants.md)）
[3] 变量模板（[§7.4.1](../ch07/7-4-template-mechanisms.md#7.4.1)）
[4] 二进制字面量（[§1.4](../ch01/1-4-types-variables.md)）
[5] 数字分隔符（[§1.4](../ch01/1-4-types-variables.md)）
[6] 泛型 lambda（[§7.3.3.1](../ch07/7-3-parameterized-operations.md#7.3.3.1)）
[7] 更一般的 lambda 捕获
[8] `[[deprecated]]` 属性
[9] 一些其他的小扩展

## 19.2.3 C++17 语言特性

[1] 保证的拷贝省略（[§6.2.2](../ch06/6-2-copy-move.md#6.2.2)）
[2] 过度对齐类型的动态分配
[3] 更严格的求值顺序（[§1.4.1](../ch01/1-4-types-variables.md#1.4.1)）
[4] UTF-8 字面量（`u8`）
[5] 十六进制浮点字面量（[§11.6.1](../ch11/11-6-output-formatting.md#11.6.1)）
[6] 折叠表达式（[§8.4.1](../ch08/8-4-variadic-templates.md#8.4.1)）
[7] 泛型值模板参数（`auto` 模板参数；[§8.2.5](../ch08/8-2-concepts.md#8.2.5)）
[8] 类模板参数类型推导（[§7.2.3](../ch07/7-2-parameterized-types.md#7.2.3)）
[9] 编译时 `if`（[§7.4.3](../ch07/7-4-template-mechanisms.md#7.4.3)）
[10] 带有初始化器的选择语句（[§1.8](../ch01/1-8-testing.md)）
[11] `constexpr` lambda
[12] 内联变量
[13] 结构化绑定（[§3.4.5](../ch03/3-4-parameters.md#3.4.5)）
[14] 新标准属性：`[[fallthrough]]`、`[[nodiscard]]` 和 `[[maybe_unused]]`
[15] `std::byte` 类型（[§16.7](../ch16/16-7-bitwise.md)）
[16] 用其底层类型的值初始化枚举（[§2.4](../ch02/2-4-enum.md)）
[17] 一些其他的小扩展

## 19.2.4 C++20 语言特性

[1] 模块（[§3.2.2](../ch03/3-2-separate-compilation.md#3.2.2)）
[2] 概念（[§8.2](../ch08/8-2-concepts.md)）
[3] 协程（[§18.6](../ch18/18-6-coroutines.md)）
[4] 指定初始化器（C99 特性的略微受限版本）
[5] `<=>`（“宇宙飞船运算符”）三路比较（[§6.5.1](../ch06/6-5-conventional-operations.md#6.5.1)）
[6] `[*this]` 按值捕获当前对象（[§7.3.3](../ch07/7-3-parameterized-operations.md#7.3.3)）
[7] 标准属性 `[[no_unique_address]]`、`[[likely]]` 和 `[[unlikely]]`
[8] 在 `constexpr` 函数中允许更多设施，包括 `new`、`union`、`try-catch`、`dynamic_cast` 和 `typeid`。
[9] 保证编译时求值的 `consteval` 函数（[§1.6](../ch01/1-6-constants.md)）
[10] 保证静态（而非运行时）初始化的 `constinit` 变量（[§1.6](../ch01/1-6-constants.md)）
[11] 使用作用域枚举（[§2.4](../ch02/2-4-enum.md)）
[12] 一些其他的小扩展

## 19.2.5 C++11 标准库组件

C++11 对标准库的增补有两种形式：新组件（例如正则表达式匹配库）和对 C++98 组件的改进（例如容器的移动构造函数）。

[1] 容器的 `initializer_list` 构造函数（[§5.2.3](../ch05/5-2-concrete-types.md#5.2.3)）
[2] 容器的移动语义（[§6.2.2](../ch06/6-2-copy-move.md#6.2.2)，[§13.2](../ch13/13-2-using-iterators.md)）
[3] 单向链表：`forward_list`（[§12.3](../ch12/12-3-list.md)）
[4] 哈希容器：`unordered_map`、`unordered_multimap`、`unordered_set` 和 `unordered_multiset`（[§12.6](../ch12/12-6-unordered-map.md)，[§12.8](../ch12/12-8-container-overview.md)）
[5] 资源管理指针：`unique_ptr`、`shared_ptr` 和 `weak_ptr`（[§15.2.1](../ch15/15-2-pointers.md#15.2.1)）
[6] 并发支持：`thread`（[§18.2](../ch18/18-2-tasks-and-threads.md)）、互斥量和锁（[§18.3](../ch18/18-3-shared-data.md)）、条件变量（[§18.4](../ch18/18-4-waiting-for-events.md)）
[7] 更高级的并发支持：`packaged_task`、`future`、`promise` 和 `async()`（[§18.5](../ch18/18-5-inter-task-communication.md)）
[8] `tuple`（[§15.3.4](../ch15/15-3-containers.md#15.3.4)）
[9] 正则表达式：`regex`（[§10.4](../ch10/10-4-regular-expressions.md)）
[10] 随机数：分布和引擎（[§17.5](../ch17/17-5-random-numbers.md)）
[11] 整数类型名称，如 `int16_t`、`uint32_t` 和 `int_fast64_t`（[§17.8](../ch17/17-8-type-aliases.md)）
[12] 固定大小的连续序列容器：`array`（[§15.3](../ch15/15-3-containers.md)）
[13] 拷贝和重新抛出异常（[§18.5.1](../ch18/18-5-inter-task-communication.md#18.5.1)）
[14] 使用错误码报告错误：`system_error`
[15] 容器的 `emplace()` 操作（[§12.8](../ch12/12-8-container-overview.md)）
[16] 广泛使用 `constexpr` 函数
[17] 系统化使用 `noexcept` 函数
[18] 改进的函数适配器：`function` 和 `bind()`（[§16.3](../ch16/16-3-function-adapters.md)）
[19] 字符串到数值的转换
[20] 作用域分配器
[21] 类型特征，如 `is_integral` 和 `is_base_of`（[§16.4.1](../ch16/16-4-type-functions.md#16.4.1)）
[22] 时间工具：`duration` 和 `time_point`（[§16.2.1](../ch16/16-2-time.md#16.2.1)）
[23] 编译时有理数算术：`ratio`
[24] 放弃进程：`quick_exit`（[§16.8](../ch16/16-8-program-exit.md)）
[25] 更多算法，如 `move()`、`copy_if()` 和 `is_sorted()`（[第 13 章](../ch13/index.md)）
[26] 垃圾回收 API；后来被废弃（[§19.2.9](19-2-cpp-evolution.md#19.2.9)）
[27] 低级并发支持：原子操作（[§18.3.2](../ch18/18-3-shared-data.md#18.3.2)）
[28] 一些其他的小扩展

## 19.2.6 C++14 标准库组件

[1] `shared_mutex` 和 `shared_lock`（[§18.3](../ch18/18-3-shared-data.md)）
[2] 用户定义字面量（[§6.6](../ch06/6-6-user-defined-literals.md)）
[3] 按类型的元组寻址（[§15.3.4](../ch15/15-3-containers.md#15.3.4)）
[4] 关联容器异构查找
[5] 一些其他的小扩展

## 19.2.7 C++17 标准库组件

[1] 文件系统（[§11.9](../ch11/11-9-file-system.md)）
[2] 并行算法（[§13.6](../ch13/13-6-parallel-algorithms.md)，[§17.3.1](../ch17/17-3-numeric-algorithms.md#17.3.1)）
[3] 数学特殊函数（[§17.2](../ch17/17-2-math-functions.md)）
[4] `string_view`（[§10.3](../ch10/10-3-string-view.md)）
[5] `any`（[§15.4.3](../ch15/15-4-alternatives.md#15.4.3)）
[6] `variant`（[§15.4.1](../ch15/15-4-alternatives.md#15.4.1)）
[7] `optional`（[§15.4.2](../ch15/15-4-alternatives.md#15.4.2)）
[8] 一种调用任何对于给定参数集可调用对象的方式：`invoke()`
[9] 基本字符串转换：`to_chars()` 和 `from_chars()`
[10] 多态分配器（[§12.7](../ch12/12-7-allocators.md)）
[11] `scoped_lock`（[§18.3](../ch18/18-3-shared-data.md)）
[12] 一些其他的小扩展

## 19.2.8 C++20 标准库组件

[1] 范围、视图和管道（[§14.1](../ch14/14-1-introduction.md)）
[2] `printf` 风格格式化：`format()` 和 `vformat()`（[§11.6.2](../ch11/11-6-output-formatting.md#11.6.2)）
[3] 日历（[§16.2.2](../ch16/16-2-time.md#16.2.2)）和时区（[§16.2.3](../ch16/16-2-time.md#16.2.3)）
[4] 用于对连续数组进行读写访问的 `span`（[§15.2.2](../ch15/15-2-pointers.md#15.2.2)）
[5] `source_location`（[§16.5](../ch16/16-5-source-location.md)）
[6] 数学常数，例如 `pi` 和 `ln10e`（[§17.9](../ch17/17-9-math-constants.md)）
[7] 对原子操作的许多扩展（[§18.3.2](../ch18/18-3-shared-data.md#18.3.2)）
[8] 等待多个线程的方式：`barrier` 和 `latch`。
[9] 特性测试宏
[10] `bit_cast<>`（[§16.7](../ch16/16-7-bitwise.md)）
[11] 位操作（[§16.7](../ch16/16-7-bitwise.md)）
[12] 更多标准库函数变为 `constexpr`
[13] 标准库中许多使用 `<=>` 的地方
[14] 许多其他小扩展

## 19.2.9 移除和废弃的特性

世界上有数十亿行 C++ 代码，没有人确切知道哪些特性正处于关键使用中。因此，ISO 委员会只有在经过多年的警告后才不情愿地移除旧特性。然而，有时一些麻烦的特性会被移除或废弃。

通过废弃一个特性，标准委员会表达了希望该特性消失的意愿。然而，委员会无权立即移除一个广泛使用的特性——无论它多么冗余或危险。因此，废弃是一个强烈提示，要避免使用该特性。它可能会在未来消失。废弃特性的列表在标准 [C++,2020] 的附录 D 中。编译器很可能会对废弃特性的使用发出警告。然而，废弃特性是标准的一部分，历史表明，由于兼容性原因，它们往往会“永远”得到支持。即使最终被移除的特性，由于用户对实现者的压力，往往也会在实现中继续存在。

- **已移除**：异常规范：`void f() throw(X, Y);` // C++98；现在是一个错误
- **已移除**：异常规范的支持设施：`unexpected_handler`、`set_unexpected()`、`get_unexpected()` 和 `unexpected()`。改用 `noexcept`（[§4.2](../ch04/4-2-exceptions.md)）。
- **已移除**：三字符组（Trigraphs）。
- **已移除**：`auto_ptr`。改用 `unique_ptr`（[§15.2.1](../ch15/15-2-pointers.md#15.2.1)）。
- **已移除**：存储说明符 `register` 的使用。
- **已移除**：对 `bool` 使用 `++`。
- **已移除**：C++98 的 `export` 特性。它很复杂并且没有被主要供应商提供。取而代之，`export` 被用作模块的关键字（[§3.2.2](../ch03/3-2-separate-compilation.md#3.2.2)）。
- **已废弃**：为带有析构函数的类生成拷贝操作（[§6.2.1](../ch06/6-2-copy-move.md#6.2.1)）。
- **已移除**：将字符串字面量赋值给 `char*`。改用 `const char*` 或 `auto`。
- **已移除**：一些 C++ 标准库函数对象和相关函数。大多与参数绑定有关。改用 lambda 和 `function`（[§16.3](../ch16/16-3-function-adapters.md)）。
- **已废弃**：枚举值与来自不同枚举或浮点值的比较。
- **已废弃**：两个数组之间的比较。
- **已废弃**：下标中的逗号操作（例如 `[a,b]`）。为允许用户定义带多个参数的 `operator[]` 腾出空间。
- **已废弃**：lambda 表达式中的隐式 `*this` 捕获。改用 `[=, this]`（[§7.3.3](../ch07/7-3-parameterized-operations.md#7.3.3)）。
- **已移除**：垃圾回收器的标准库接口。C++ 垃圾回收器不使用该接口。
- **已废弃**：`strstream`；改用 `spanstream`（[§11.7.4](../ch11/11-7-streams.md#11.7.4)）。
