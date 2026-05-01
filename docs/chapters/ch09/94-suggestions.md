## 9.4 建议

1. 优先使用标准库组件，而非自己实现——它们经过充分测试和优化
2. 使用 `std::vector` 作为默认容器，除非有明确理由选择其他容器
3. 使用标准库算法（如 `sort`、`find`）而非手写循环
4. 使用 `std::string` 而非 C 风格字符串
5. 使用智能指针（`std::unique_ptr`、`std::shared_ptr`）管理动态内存
6. 使用 `std::chrono` 进行时间相关的操作
7. 使用 `std::format`（C++20）进行类型安全的字符串格式化
8. 了解标准库提供的设施，避免重复造轮子
9. 在头文件中避免 `using namespace std;`
10. 参考 cppreference.com 获取标准库的详细文档
