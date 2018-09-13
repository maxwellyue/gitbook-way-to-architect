# LockSupport

`LockSupport`定义了一组公共静态方法，这些方法提供了最基本的线程阻塞和唤醒功能，是构建同步组件的基础工具，它主要有两类方法：

* 以`park`开头的方法：阻塞当前线程
* 以`unpark`开头的方法：唤醒被阻塞的线程

```java
void park()
阻塞当前线程，只有当前线程被中断或其他线程调用unpark(Thread thread)，才能从park()方法返回

void parkNanos(long nanos)
阻塞当前线程，最长不超过nanos纳秒，返回条件在park()的基础上增加了超时返回

void parkUntil(long deadline)
阻塞当前线程，直到deadline这个时间点（从1970年开始到deadline时间的毫秒数）

void unpark(Thread thread)
唤醒处于阻塞状态的thread线程
```

在JDK1.6中，该类增加了`void park(Object blocker)`、`void parkNanos(Object blocker, long nanos)`、`void parkUntil(Object blocker, long deadline)`方法，相比之前的park方法，多了一个blocker对象，该对象用来标识

**当前线程在等待的对象（阻塞对象）**，主要用来问题排查和系统监控（对线程dump时，可以提供阻塞对象的信息），可以用来代替原有的park方法。  


