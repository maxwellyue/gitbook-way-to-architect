# ConcurrentSkipListMap

ConcurrentSkipListMap是基于跳表实现的Map，支持根据key排序。在理论上能够在O\(log\(n\)\)时间内完成查找、插入、删除操作。

注意，调用ConcurrentSkipListMap的size时，由于多个线程可以同时对映射表进行操作，所以映射表需要遍历整个链表才能返回元素个数，这个操作是个O\(log\(n\)\)的操作。



