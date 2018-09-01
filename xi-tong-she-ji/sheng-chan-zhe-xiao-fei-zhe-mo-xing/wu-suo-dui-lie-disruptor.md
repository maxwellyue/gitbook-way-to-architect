# 无锁队列Disruptor

## 背景 {#-}

Disruptor是英国外汇交易公司LMAX开发的一个高性能队列，研发的初衷是解决内存队列的延迟问题（在性能测试中发现竟然与I/O操作处于同样的数量级）。基于Disruptor开发的系统单线程能支撑每秒600万订单，2010年在QCon演讲后，获得了业界关注。2011年，企业应用软件专家Martin Fowler专门撰写长文介绍。同年它还获得了Oracle官方的Duke大奖。

目前，包括Apache Storm、Camel、Log4j 2在内的很多知名项目都应用了Disruptor以获取高性能。

## Java内置队列 {#java-}

介绍Disruptor之前，我们先来看一看常用的线程安全的内置队列有什么问题。Java的内置队列如下表所示。

| 队列 | 有界性 | 锁 | 数据结构 |
| :--- | :--- | :--- | :--- |
| ArrayBlockingQueue | bounded | 加锁 | arraylist |
| LinkedBlockingQueue | optionally-bounded | 加锁 | linkedlist |
| ConcurrentLinkedQueue | unbounded | 无锁 | linkedlist |
| LinkedTransferQueue | unbounded | 无锁 | linkedlist |
| PriorityBlockingQueue | unbounded | 加锁 | heap |
| DelayQueue | unbounded | 加锁 | heap |

队列的底层一般分成三种：数组、链表和堆。其中，堆一般情况下是为了实现带有优先级特性的队列，暂且不考虑。

我们就从数组和链表两种数据结构来看，基于数组线程安全的队列，比较典型的是ArrayBlockingQueue，它主要通过加锁的方式来保证线程安全；基于链表的线程安全队列分成LinkedBlockingQueue和ConcurrentLinkedQueue两大类，前者也通过锁的方式来实现线程安全，而后者以及上面表格中的LinkedTransferQueue都是通过原子变量compare and swap（以下简称“CAS”）这种不加锁的方式来实现的。

通过不加锁的方式实现的队列都是无界的（无法保证队列的长度在确定的范围内）；而加锁的方式，可以实现有界队列。在稳定性要求特别高的系统中，为了防止生产者速度过快，导致内存溢出，只能选择有界队列；同时，为了减少Java的垃圾回收对系统性能的影响，会尽量选择array/heap格式的数据结构。这样筛选下来，符合条件的队列就只有ArrayBlockingQueue。

TODO：ArrayBlockingQueue的问题：加锁和伪共享

## Disruptor的设计方案 {#disruptor-}

Disruptor通过以下设计来解决队列速度慢的问题：

* 环形数组结构

为了避免垃圾回收，采用数组而非链表。同时，数组对处理器的缓存机制更加友好。

* 元素位置定位

数组长度2^n，通过位运算，加快定位的速度。下标采取递增的形式。不用担心index溢出的问题。index是long类型，即使100万QPS的处理速度，也需要30万年才能用完。

* 无锁设计

每个生产者或者消费者线程，会先申请可以操作的元素在数组中的位置，申请到之后，直接在该位置写入或者读取数据。

## Disruptor的使用 {#disruptor-}















## 内容来源

[高性能队列——Disruptor](https://tech.meituan.com/disruptor.html)

[高性能线程间队列DISRUPTOR简介](http://niceaz.com/%E9%AB%98%E6%80%A7%E8%83%BD%E7%BA%BF%E7%A8%8B%E9%97%B4%E9%98%9F%E5%88%97disruptor%E7%AE%80%E4%BB%8B/#more-189)

[LMAX-Exchange](https://github.com/LMAX-Exchange)/[**disruptor**](https://github.com/LMAX-Exchange/disruptor)[  
](http://niceaz.com/%E9%AB%98%E6%80%A7%E8%83%BD%E7%BA%BF%E7%A8%8B%E9%97%B4%E9%98%9F%E5%88%97disruptor%E7%AE%80%E4%BB%8B/#more-189)

