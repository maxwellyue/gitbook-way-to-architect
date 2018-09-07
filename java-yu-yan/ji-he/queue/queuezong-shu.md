**Queue：元素有序，先进先出**

# java.util

**ArrayDeque**

数据结构为数组，双端队列，在队头队尾均可心插入或删除元素**。**实现了DeQueue接口。DeQueue\(Double-ended queue\)继承了Queue接口，创建双向队列，灵活性更强，可以前向或后向迭代，

**PriorityQueue**

无界队列，基于优先级堆，它的元素根据自然顺序或者通过实现Comparator接口的自定义排序方式进行排序。

> 优先级队列是不同于先进先出队列的另一种队列。每次从队列中取出的是具有最高优先权的元素。  
> （1）优先级队列不是同步的（线程安全版本为PriorityBlockingQueue），队列的获取操作如poll\(\),peek\(\)和element\(\)是访问的队列的头，保证获取的是最小的元素（根据指定的排序规则）  
> （2）返回的迭代器并不保证提供任何的有序性  
> （3）优先级队列不允许null元素，否则抛出NullPointException。

# java.util.concurrent

## BlockingQueue

阻塞队列：当队列满时，队列会阻塞插入元素的线程，直到队列不满；当队列为空时，队列会阻塞获取元素的线程，直到队列不空。阻塞队列常用于生产者和消费者模式，生产者向队列中添加元素，消费者则从队列中取出元素。

`BlockingQueue`中提供了以下方法来进行元素的插入/移除操作：

| 方法/处理方式 | 抛出异常 | 返回特殊值 | 一直阻塞 | 超时退出 |
| :--- | :--- | :--- | :--- | :--- |
| 插入 | add\(e\) | offer\(e\) | put\(e\) | offer\(e, time, unit\) |
| 移除 | remove\(\) | poll\(\) | take\(\) | poll\(time, unit\) |
| 检查 | element\(\) | peek\(\) | 不可用 | 不可用 |

具体理解如下：

**如果队列已满：**

* 使用add\(e\)添加元素，则抛出IllegalStateException\("Queue full"\)异常；

* 使用offer\(e\)添加元素，则返回false；

* 使用put\(e\)添加元素，则当前线程会一直阻塞，直到队列中出现空位或响应中断退出；

* 使用offer\(e, time, unit\)添加元素，则当前线程会阻塞一定时间，超时后如果还是满，则返回false，如果在超时前放入成功，则返回true

**如果队列为空：**

* 使用remove\(\)移除元素，则抛出NoSuchElementException异常；

* 使用poll\(\)移除元素，则返回null；

* 使用take\(\)移除元素，则当前线程会一直阻塞，直到队列中有元素插入或响应中断退出；

* 使用poll\(time, unit\)移除元素，则当前线程会阻塞一定时间，超时后如果还是为空，则返回null，如果在超时前有元素插入，则返回插入的这个元素

具体的实现类及其特性如下：

**ArrayBlockingQueue**

使用数组实现的有界阻塞队列，按照FIFO的原则对元素排序；内部使用重入锁可实现公平访问。内部使用一个重入锁来控制并发修改操作，即同一时刻，只能进行放或取中的一个操作。初始化时，必须指定容量大小。

**LinkedBlockingQueue**

使用链表实现的有界阻塞队列，按照FIFO的原则对元素排序；默认和最大长度均为`Integer.MAX_VALUE`，所以在使用的时候，要注意指定最大容量，否则可能会导致元素数量过多，内存溢出。内部使用两个重入锁来控制并发操作，即同一时刻，允许同时进行放和取。

**PriorityBlockingQueue**

支持优先级的无界阻塞队列，默认情况下元素按照自然顺序升序排列，可以自定义类实现`compareTo()`方法来指定元素的排序规则，或在初始化`PriorityBlockingQueue`时指定构造参数`Comparator`来对元素进行排序，但不能保证同优先级元素的顺序；

**DelayQueue**

支持延时获取元素的无界阻塞队列，队列中的元素必须实现Delayed接口，在创建元素时可以指定多久才能从队列中获取当前元素，只有在延迟期满后，才能从队列中获取元素。`DelayQueue`可以应用在缓存系统的设计（用`DelayQueue`保存缓存元素的有效期，使用一个线程循环查询`DelayQueue`，一旦能从`DelayQueue`中获取元素，表示缓存有效期到了）、定时任务调度（`ScheduledThreadPoolExecutor`中的`ScheduledFutureTask`类就是实现的`Delayed`接口）等场景。

**SyncronousQueue**

不存储元素的阻塞队列，每一个put操作必须等待一个take操作，否则不能继续添加元素，支持公平访问队列，非常适合传递性场景，即把生产者线程处理的数据直接传递给消费者线程，队列本身不存储任何元素。`SyncronousQueue`的吞吐量高于`ArrayBlockingQueue`和`LinkedBlockingDeque`。

**LinkedTransferQueue**

使用链表实现的无界阻塞TransferQueue，当有消费者正在等待接受元素时，队列可以通过transfer\(\)方法把生产者传入的元素立即传给消费者。

**LinkedBlockingDeque**

使用链表实现的双向阻塞队列，可以在队列的两端进行插入和移除元素。

## Concurrent Queue

**ConcurrentLinkedQueue**

适合在对性能要求相对较高，同时存在**多个线程对队列的同时读写**的场景，即如果对队列加锁的成本较高则适合使用无锁的ConcurrentLinkedQueue来替代。

> 单生产者，单消费者 用 LinkedBlockingqueue
>
> 多生产者，单消费者 用 LinkedBlockingqueue，比如任务队列
>
> 单生产者 ，多消费者 用 ConcurrentLinkedQueue
>
> 多生产者 ，多消费者 用 ConcurrentLinkedQueue ，比如消息队列

**ConcurrentLinkedDeque**

总结如下：

| 队列 | 有界性 | 锁 | 数据结构 |
| --- | --- | --- | --- |
| ArrayBlockingQueue | 有界 | 加锁 | 数组 |
| LinkedBlockingQueue | 有界/无界 | 加锁 | 单向链表 |
| ConcurrentLinkedQueue | 无界 | 无锁 | 单向链表 |
| LinkedTransferQueue | 无界 | 无锁 | 单向链表 |
| PriorityBlockingQueue | 无界 | 加锁 | 堆 |
| DelayQueue | 无界 | 加锁 | 堆 |

队列的底层一般分成三种：数组、链表和堆。其中，堆一般情况下是为了实现带有优先级特性的队列。

基于数组线程安全的队列，比较典型的是ArrayBlockingQueue，它主要通过加锁的方式来保证线程安全；

基于链表的线程安全队列分成LinkedBlockingQueue和ConcurrentLinkedQueue两大类，前者也通过锁的方式来实现线程安全，而后者以及LinkedTransferQueue都是通过原子变量compare and swap（以下简称“CAS”）这种不加锁的方式来实现的。

通过不加锁的方式实现的队列都是无界的（无法保证队列的长度在确定的范围内）；而加锁的方式，可以实现有界队列。在稳定性要求特别高的系统中，为了防止生产者速度过快，导致内存溢出，只能选择有界队列；同时，为了减少Java的垃圾回收对系统性能的影响，会尽量选择array/heap格式的数据结构。这样筛选下来，符合条件的队列就只有ArrayBlockingQueue。



扩展阅读

[Java多线程-工具篇-BlockingQueue](https://www.cnblogs.com/jackyuj/archive/2010/11/24/1886553.html)

[并发队列ConcurrentLinkedQueue和阻塞队列LinkedBlockingQueue使用场景总结](http://www.aichengxu.com/other/1959339.htm)

[优先级队列是一种什么样的数据结构](http://www.importnew.com/6510.html)

[高性能队列——Disruptor](https://tech.meituan.com/disruptor.html)



