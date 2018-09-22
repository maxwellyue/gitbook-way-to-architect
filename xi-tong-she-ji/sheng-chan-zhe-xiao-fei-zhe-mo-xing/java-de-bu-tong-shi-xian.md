

## 无锁的缓存框架: Disruptor

BlockingQueue 实现生产者和消费者模式简单易懂，但是`BlockingQueue`并不是一个高性能的实现：它完全使用锁和阻塞来实现线程之间的同步。在高并发的场合，它的性能并不是特别的优越。（`ConconcurrentLinkedQueue`是一个高性能的队列，但并不没有实现`BlockingQueue`接口，即不支持阻塞操作）。

Disruptor是LMAX公司开发的高效的无锁缓存队列。它使用无锁的方式实现了一个环形队列，非常适合于实现生产者和消费者模式，如：事件和消息的发布。

//TODO 应用场景的代码实现

## 参考

[Java 实现生产者 – 消费者模型](https://link.jianshu.com/?t=http%3A%2F%2Fwww.importnew.com%2F27063.html)：各种实现方式的性能  
[高性能的生产者-消费者：无锁的实现](https://link.jianshu.com/?t=http%3A%2F%2Ffrobisher.me%2F2017%2F05%2F26%2Fjava-producer-consumer-disruptor%2F)：无锁实现  
[Java生产者和消费者模型的5种实现方式](https://link.jianshu.com/?t=http%3A%2F%2Fcdn2.jianshu.io%2Fp%2F66e8b5ab27f6)  
[生产者/消费者问题的多种Java实现方式](https://link.jianshu.com/?t=http%3A%2F%2Fblog.csdn.net%2Fmonkey_d_meng%2Farticle%2Fdetails%2F6251879)  
[Java阻塞队列ArrayBlockingQueue和LinkedBlockingQueue实现原理分析](https://link.jianshu.com/?t=https%3A%2F%2Ffangjian0423.github.io%2F2016%2F05%2F10%2Fjava-arrayblockingqueue-linkedblockingqueue-analysis%2F)：两种常用阻塞队列的区别

[完整代码示例在此](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fmaxwellyue%2FJavaLanguage%2Ftree%2Fmaster%2Fsrc%2Fmain%2Fjava%2Fproducerandconsumer)

