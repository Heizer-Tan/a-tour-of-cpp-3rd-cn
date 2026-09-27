# 13.7 建议

[1] STL 算法对一个或多个序列进行操作；[§13.1](13-1-introduction.md)。

[2] 输入序列是半开区间，由一对迭代器界定；[§13.1](13-1-introduction.md)。

[3] 你可以为满足特定需求自定义迭代器；[§13.1](13-1-introduction.md)。

[4] 许多算法也能作用于 I/O 流；[§13.3.1](13-3-iterator-types.md#13.3.1)。

[5] 查找类算法通常用返回输入序列尾迭代器表示“未找到”；[§13.2](13-2-using-iterators.md)。

[6] 算法不会直接对其参数序列增删元素；[§13.2](13-2-using-iterators.md)，[§13.5](13-5-algorithm-overview.md)。

[7] 编写循环时，想一想是否能改写为标准算法；[§13.2](13-2-using-iterators.md)。

[8] 使用类型别名整理累赘的类型记号；[§13.2](13-2-using-iterators.md)。

[9] 借助谓词与其他函数对象，可以让标准算法表达更丰富的含义；[§13.4](13-4-predicates.md)，[§13.5](13-5-algorithm-overview.md)。

[10] 谓词不得修改其参数；[§13.4](13-4-predicates.md)。

[11] 熟悉标准库算法并优先于手写循环使用它们；[§13.5](13-5-algorithm-overview.md)。
