# 9.4 建议

[1] 不要重复造轮子；使用库；[§9.1](9-1-introduction.md)；[CG: SL.1]。
[2] 当有选择时，优先选择标准库而非其他库；[§9.1](9-1-introduction.md)；[CG: SL.2]。
[3] 不要认为标准库对一切都理想；[§9.1](9-1-introduction.md)。
[4] 如果不使用模块，记得 `#include` 适当的头文件；[§9.3.1](9-3-standard-library-organization.md)。
[5] 记住标准库设施定义在命名空间 `std` 中；[§9.3.1](9-3-standard-library-organization.md)；[CG: SL.3]。
[6] 使用范围时，记得显式限定算法名称；[§9.3.2](9-3-standard-library-organization.md)。
[7] 优先导入模块，而非 `#include` 头文件（[§9.3.3](9-3-standard-library-organization.md)）。
