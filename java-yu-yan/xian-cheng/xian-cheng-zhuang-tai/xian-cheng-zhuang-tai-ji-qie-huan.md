# 线程状态及切换

## 线程状态

```java
/**
 * 一个线程在给定的某个时刻，只能处在下面的6中状态中的一种。
 *
 * 这些状态是相对于虚拟机而言的，并不能反映出任何操作系统中线程状态。
 *
 */
public enum State {
    /**
     *
     * 新建状态
     *
     * 创建后还没有启动的线程处在NEW的状态；
     *
     * 而启动线程只有start()方法。也就是说还未调用start()方法的线程处在NEW的状态
     */
    NEW,

    /**
     *
     * 运行状态
     *
     * 处在此状态的线程有可能正在运行，也有可能正在等待CPU为它分配执行时间
     * 对于虚拟机而言，处在RUNNABLE状态的线程就是正在执行。
     *
     */
    RUNNABLE,

    /**
     *
     * 阻塞状态
     *
     * 处在阻塞状态的线程正在等待一个锁
     * 
     * 在程序等待进入同步区域的时候，线程将进入这种状态。
     * 
     *
     */
    BLOCKED,

    /**
     * 
     * 无限期等待状态
     * 
     * 线程进入无限期等待状态的原因是调用了下面三种方法之一：
     * ①没有设置Timeout参数的Object.wait()方法
     * ②没有设置Timeout参数的Thread.join()方法
     * ③LockSupport.park()方法
     * 
     * 
     */
    WAITING,
    /**
     * 
     * 限期等待状态
     * 
     * 线程进入限期等待状态的原因是调用了下面五种方法之一：
     * 
     * ①设置Timeout参数的Object.wait()方法
     * ②设置Timeout参数的Thread.join()方法
     * ③LockSupport.parkNanos()方法
     * ④LockSupport.parkUntil
     * ⑤Thread.sleep()方法
     * 
     */
    TIMED_WAITING,

    /**
     * 
     * 结束状态
     * 
     * 线程已经完成执行。
     */
    TERMINATED;
}
```

## 线程状态切换

![](../../../.gitbook/assets/import.png)

