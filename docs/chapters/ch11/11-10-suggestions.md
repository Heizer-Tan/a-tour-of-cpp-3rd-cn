# 11.10 建议

[1] `iostream` 具有类型安全、类型敏感且可扩展的特点；[§11.1](11-1-introduction.md)。
[2] 仅在必要时才使用基于字符的输入；[§11.3](11-3-input.md)；[CG: SL.io.1]。
[3] 读取时始终考虑格式错误的输入；[§11.3](11-3-input.md)；[CG: SL.io.2]。
[4] 避免使用 `endl`（如果你不知道 `endl` 是什么，你并没有错过什么）；[CG: SL.io.50]。
[5] 为具有有意义的文本表示的用户定义类型定义 `<<` 和 `>>`；[§11.1](11-1-introduction.md)，[§11.2](11-2-output.md)，[§11.3](11-3-input.md)。
[6] 使用 `cout` 作常规输出，使用 `cerr` 报告错误；[§11.1](11-1-introduction.md)。
[7] 既有面向普通 `char` 的 `iostream`，也有面向宽字符的类型，并且你可以为任意字符种类定义 `iostream`；[§11.1](11-1-introduction.md)。
[8] 支持二进制 I/O；[§11.1](11-1-introduction.md)。
[9] 标准 `iostream` 覆盖标准 I/O、文件与字符串等场景；[§11.2](11-2-output.md)，[§11.3](11-3-input.md)，[§11.7.2](11-7-streams.md#1172-文件流)，[§11.7.3](11-7-streams.md#1173-字符串流)。
[10] 将多个 `<<` 串联以获得更简洁的写法；[§11.2](11-2-output.md)。
[11] 将多个 `>>` 串联以获得更简洁的写法；[§11.3](11-3-input.md)。
[12] 读入 `string` 不会溢出；[§11.3](11-3-input.md)。
[13] 默认情况下 `>>` 会跳过开头的空白；[§11.3](11-3-input.md)。
[14] 使用流状态的 `fail` 处理可能可恢复的 I/O 错误；[§11.4](11-4-io-status.md)。
[15] 我们可以为自己的类型定义 `<<` 和 `>>`；[§11.5](11-5-user-defined-io.md)。
[16] 添加新的 `<<` 和 `>>` 不必修改 `istream` 或 `ostream`；[§11.5](11-5-user-defined-io.md)。
[17] 使用操纵符或 `format()` 控制格式化；[§11.6.1](11-6-output-formatting.md#1161-流格式化)，[§11.6.2](11-6-output-formatting.md#1162-printf-风格的格式化)。
[18] `precision()` 的设置会影响其后所有浮点输出操作；[§11.6.1](11-6-output-formatting.md#1161-流格式化)。
[19] 浮点格式操纵符（如 `scientific`）会影响其后所有浮点输出操作；[§11.6.1](11-6-output-formatting.md#1161-流格式化)。
[20] 使用标准操纵符时请 `#include <ios>` 或 `#include <iostream>`；[§11.6](11-6-output-formatting.md)。
[21] 流格式化操纵符对流上的许多值具有“粘性”；[§11.6.1](11-6-output-formatting.md#1161-流格式化)。
[22] 使用带参数的标准操纵符时请 `#include <iomanip>`；[§11.6](11-6-output-formatting.md)。
[23] 可以以标准格式输出时间、日期等；[§11.6.1](11-6-output-formatting.md#1161-流格式化)，[§11.6.2](11-6-output-formatting.md#1162-printf-风格的格式化)。
[24] 不要试图拷贝流：流只能移动；[§11.7](11-7-streams.md)。
[25] 使用文件流之前，记住确认它已正确关联到文件；[§11.7.2](11-7-streams.md#1172-文件流)。
[26] 在内存中格式化时，使用字符串流或内存流；[§11.7.3](11-7-streams.md#1173-字符串流)，[§11.7.4](11-7-streams.md#1174-内存流)。
[27] 可以为任意两种都能表示为字符串的类型定义相互转换；[§11.7.3](11-7-streams.md#1173-字符串流)。
[28] C 风格 I/O 不是类型安全的；[§11.8](11-8-c-style-io.md)。
[29] 除非你使用 `printf` 家族函数，否则应调用 `ios_base::sync_with_stdio(false)`；[§11.8](11-8-c-style-io.md)；[CG: SL.io.10]。
[30] 优先使用 `<filesystem>`，而不是直接使用平台相关接口；[§11.9](11-9-file-system.md)。
