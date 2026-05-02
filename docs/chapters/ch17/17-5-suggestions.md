## 17.5 建议

1. 使用 `<cmath>` 中的标准数学函数
2. 使用 `std::complex` 进行复数运算
3. 使用 `std::valarray` 进行需要逐元素运算的数值计算
4. 对于通用数组，使用 `std::vector` 而非 `std::valarray`
5. 使用 `std::numeric_limits<T>` 获取类型的数值属性（如最大值、最小值）
6. 注意浮点运算的精度和舍入问题
