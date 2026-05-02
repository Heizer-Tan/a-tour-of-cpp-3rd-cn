## 15.5 建议

1. 使用 `std::unique_ptr` 管理独占所有权的动态对象
2. 使用 `std::shared_ptr` 仅在确实需要共享所有权时
3. 使用 `std::make_unique` 和 `std::make_shared` 创建智能指针
4. 使用 `std::span` 作为数组参数的统一接口
5. 使用 `std::optional` 替代空指针来表示"可能没有值"
6. 使用 `std::variant` 替代联合体（union）
7. 避免在容器中存储裸指针——使用智能指针
8. 使用 `std::weak_ptr` 打破 `shared_ptr` 的循环引用
