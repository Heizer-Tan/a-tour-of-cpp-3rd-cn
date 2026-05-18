# 10.5 建议

[1] 使用 `std::string` 拥有字符序列；[§10.2](10-2-strings.md)；[CG: SL.str.1]。
[2] 优先使用字符串操作，而非 C 风格字符串函数；[§10.1](10-1-introduction.md)。
[3] 使用 `string` 声明变量和成员，而非作为基类；[§10.2](10-2-strings.md)。
[4] 按值返回字符串（依赖移动语义和拷贝省略）；[§10.2](10-2-strings.md)，[§10.2.1](10-2-strings.md#1021-string-的实现)；[CG: F.15]。
[5] 直接或间接使用 `substr()` 读取子串，使用 `replace()` 写入子串；[§10.2](10-2-strings.md)。
[6] 字符串可以根据需要增长和收缩；[§10.2](10-2-strings.md)。
[7] 当你需要范围检查时，使用 `at()` 而非迭代器或 `[]`；[§10.2](10-2-strings.md)，[§10.3](10-3-string-view.md)。
[8] 当你需要优化速度时，使用迭代器和 `[]` 而非 `at()`；[§10.2](10-2-strings.md)，[§10.3](10-3-string-view.md)。
[9] 使用范围 `for` 以安全地减少范围检查；[§10.2](10-2-strings.md)，[§10.3](10-3-string-view.md)。
[10] 读入 `string` 不会溢出；[§10.2](10-2-strings.md)，[§11.3](../ch11/11-3-input.md)。
[11] 仅在必须时，使用 `c_str()` 或 `data()` 生成字符串的 C 风格表示；[§10.2](10-2-strings.md)。
[12] 使用 `stringstream` 或通用值提取函数（如 `to<X>`）进行字符串的数值转换；[§11.7.3](../ch11/11-7-streams.md#1173-字符串流)。
[13] `basic_string` 可用于构造任意字符类型的字符串；[§10.2.1](10-2-strings.md#1021-string-的实现)。
[14] 对表示标准库字符串的字符串字面量使用 `s` 后缀；[§10.2](10-2-strings.md)；[CG: SL.str.12]。（亦见 [§6.6](../ch06/6-6-user-defined-literals.md)。）
[15] 将 `string_view` 作为需要以多种方式**读取**字符序列的函数的参数；[§10.3](10-3-string-view.md)；[CG: SL.str.2]。
[16] 将 `string_span<char>`（或等价的可写字符区间类型）作为需要以多种方式**写入**字符序列的函数的参数；[§10.3](10-3-string-view.md)；[CG: SL.str.2] [CG: SL.str.11]。
[17] 将 `string_view` 视为一种带长度的指针；它不拥有其字符；[§10.3](10-3-string-view.md)。
[18] 对表示标准库 `string_view` 的字符串字面量使用 `sv` 后缀；[§10.3](10-3-string-view.md)。
[19] 对大多数常规正则表达式用途使用 `regex`；[§10.4](10-4-regular-expressions.md)。
[20] 除最简单的模式外，优先用原始字符串字面量来表达模式；[§10.4](10-4-regular-expressions.md)。
[21] 使用 `regex_match()` 匹配完整输入；[§10.4](10-4-regular-expressions.md)，[§10.4.2](10-4-regular-expressions.md#1042-正则表达式记法)。
[22] 使用 `regex_search()` 在输入流中搜索模式；[§10.4.1](10-4-regular-expressions.md#1041-搜索)。
[23] 正则表达式记号可调整以匹配各种标准；[§10.4.2](10-4-regular-expressions.md#1042-正则表达式记法)。
[24] 默认正则表达式记号是 ECMAScript（JavaScript）所基于的 ECMA 变体；[§10.4.2](10-4-regular-expressions.md#1042-正则表达式记法)。
[25] 有所节制；正则表达式很容易变成只写语言；[§10.4.2](10-4-regular-expressions.md#1042-正则表达式记法)。
[26] 注意反向引用 `\1`、`\2` 等可用来引用先前的子模式；[§10.4.2](10-4-regular-expressions.md#1042-正则表达式记法)。
[27] 使用后缀 `?` 使重复为“惰性”（最短匹配）；[§10.4.2](10-4-regular-expressions.md#1042-正则表达式记法)。
[28] 使用 `regex_iterator` 在字符序列上迭代查找模式；[§10.4.3](10-4-regular-expressions.md#1043-迭代器regex_iterator)。
