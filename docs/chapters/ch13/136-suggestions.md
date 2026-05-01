## 13.6 建议

1. 优先使用标准库算法，而非手写循环——它们更清晰、更不易出错
2. 使用 lambda 表达式来定制算法的行为
3. 使用 `std::sort` 进行排序，`std::binary_search` 进行二分查找
4. 记住 `remove` 不会改变容器大小——配合 `erase` 使用
5. 使用 `std::accumulate` 和 `std::inner_product` 进行数值计算
6. 使用 `std::copy_if` 和 `std::transform` 进行条件复制和变换
7. 对于已排序的序列，使用 `std::lower_bound` 和 `std::upper_bound`
8. 使用 C++20 的范围算法（`std::ranges::sort` 等）简化代码
