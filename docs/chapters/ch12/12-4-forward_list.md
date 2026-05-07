# 12.4 forward_list

标准库还提供了一个称�?`forward_list` 的单向链表�?
`forward_list` 与（双向）`list` 的不同之处在于它只允许前向迭代。这样做的目的是节省空间。不需要在每个链接中保留前驱指针，并且一个空�?`forward_list` 的大小只有一个指针。`forward_list` 甚至不保存其元素个数。如果你需要元素个数，请自行计数。如果你负担不起计数的代价，那么你可能不应该使用 `forward_list`�?