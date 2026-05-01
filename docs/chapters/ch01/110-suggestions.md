# 1.10 建议

以下是第一章中最重要的建议，按主题整理：

1. **不要恐慌！** 随着时间推移，一切都会变得清晰；[§1.1](11-introduction.md)

2. 你不需要了解 C++ 的每一个细节就能写出好的程序；

3. 专注于编程技术，而不是语言特性；

4. 关于语言定义问题的最终答案，请查阅 ISO C++ 标准；

5. **将有意义的概念表示为类；** [§2.3](../ch02/23-class.md)

6. 将函数表示为执行某些操作的有意义的代码单元；[§1.3](13-functions.md)

7. 使用头文件来表示接口并强调逻辑结构；[§3.2](../ch03/32-separate-compilation.md)

8. 使用 `#include` 仅在需要时包含头文件；[§3.2.1](../ch03/32-separate-compilation.md#3.2.1)

9. 避免使用 `using namespace std;` 在头文件中；[§3.3](../ch03/33-namespace.md)

10. 优先使用 `import std;` 而非 `#include` 来包含标准库；[§1.2](12-program.md)

11. 使用 `{}` 初始化语法，避免窄化转换；[§1.4](14-types-variables.md)

12. 始终初始化变量；[§1.4](14-types-variables.md)

13. 使用 `auto` 避免重复类型名；[§1.4.2](14-types-variables.md#1.4.2)

14. 避免使用宏进行常量定义；使用 `const`、`constexpr` 或 `enum`；[§1.6](16-constants.md)

15. 使用 `nullptr` 而非 `0` 或 `NULL`；[§1.7](17-pointers-arrays.md)

16. 优先使用范围 `for` 而非传统的 `for` 循环；[§1.7](17-pointers-arrays.md)

17. 使用 `assert` 和 `static_assert` 来捕获错误；[§1.8](18-testing.md)

18. 理解 C++ 的基本内存模型；[§1.9](19-hardware.md)

19. 优先使用 `using` 而非 `typedef` 定义类型别名；[§1.9](19-hardware.md)

20. 避免 C 风格的类型转换；使用命名的转换操作；[§1.9](19-hardware.md)

21. 声明一个名字时，确保其作用域尽可能小；[§1.5](15-scope-lifecycle.md)

22. 避免"魔法常量"；使用符号常量；[§1.6](16-constants.md)

23. 优先使用不可变数据；[§1.6](16-constants.md)

24. 保持函数简短；[§1.3](13-functions.md)

25. 使用有意义的名称；[§1.2](12-program.md)
