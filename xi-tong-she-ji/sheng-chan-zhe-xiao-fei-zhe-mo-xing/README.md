## 概述

生产者消费者模型可以描述为：  
①生产者持续生产，直到仓库放满产品，则停止生产进入等待状态；仓库不满后继续生产；  
②消费者持续消费，直到仓库空，则停止消费进入等待状态；仓库不空后，继续消费；  
③生产者可以有多个，消费者也可以有多个；

![生产者消费者模型](../../.gitbook/assets/image %2814%29.png)

对应到程序中，仓库对应缓冲区，可以使用队列来作为缓冲区，并且这个队列应该是有界的，即最大容量是固定的；进入等待状态，则表示要阻塞当前线程，直到某一条件满足，再进行唤醒。

常见的实现方式主要有以下几种：

* 使用`wait()`和`notify()`
* 使用`Lock`和`Condition`
* 使用信号量`Semaphore`
* 使用`JDK`自带的阻塞队列
* 使用管道流

## 参考

[Java 实现生产者 – 消费者模型](https://link.jianshu.com/?t=http%3A%2F%2Fwww.importnew.com%2F27063.html)：各种实现方式的性能  
[高性能的生产者-消费者：无锁的实现](https://link.jianshu.com/?t=http%3A%2F%2Ffrobisher.me%2F2017%2F05%2F26%2Fjava-producer-consumer-disruptor%2F)：无锁实现  
[Java生产者和消费者模型的5种实现方式](https://link.jianshu.com/?t=http%3A%2F%2Fcdn2.jianshu.io%2Fp%2F66e8b5ab27f6)  
[生产者/消费者问题的多种Java实现方式](https://link.jianshu.com/?t=http%3A%2F%2Fblog.csdn.net%2Fmonkey_d_meng%2Farticle%2Fdetails%2F6251879)  
[Java阻塞队列ArrayBlockingQueue和LinkedBlockingQueue实现原理分析](https://link.jianshu.com/?t=https%3A%2F%2Ffangjian0423.github.io%2F2016%2F05%2F10%2Fjava-arrayblockingqueue-linkedblockingqueue-analysis%2F)：两种常用阻塞队列的区别

[完整代码示例在此](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fmaxwellyue%2FJavaLanguage%2Ftree%2Fmaster%2Fsrc%2Fmain%2Fjava%2Fproducerandconsumer)



