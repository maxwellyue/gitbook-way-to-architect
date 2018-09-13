# Lock

### 锁的概念

锁是用来控制多个线程访问共享资源的方式，一般来说，一个锁可以防止多个线程同时访问共享资源（但有些锁可以允许多个线程并发的访问共享资源，如读写锁）。

在JDK1.5之前，Java是通过synchronized关键字实现锁功能的：隐式地获取锁和释放锁，但不够灵活。

在JDK1.5，`java.util.concurrent`包中新增了Lock接口以及相关实现类，用来实现锁功能。它提供了与synchronized关键字类似的同步功能，但功能更强大和灵活：获取锁和释放锁的可操作性、可中断地获取锁、超时获取锁等，见下表：

| 特性 | 描述 |
| :--- | :--- |
| 尝试非阻塞地获取锁 | 当前线程尝试获取锁，如果这个时刻锁没有被其他线程获取到，则成功获取并持有锁 |
| 能被中断地获取锁 | 获取到锁的线程能够响应中断（而synchronized则不会响应中断操作） |
| 超时获取锁 | 在指定的截止时间之前获取锁，如果在截止时间到了仍无法获取锁，则返回。 |

### **Lock接口**

Lock接口中定义的方法如下：

```java
public interface Lock {

    //获取锁：如果当前线程无法获取到锁（可能其他线程正在持有锁），则当前线程就会休眠，直到获取到锁
    void lock();

    /**
     * 可中断地获取锁
     * 
     * 如果如果当前线程无法获取到锁（可能其他线程正在持有锁），则当前线程就会休眠，
     * 直到发生下面两种情况的任意一种：
     * ①获取到了锁
     * ②被其他线程中断
     */
    void lockInterruptibly() throws InterruptedException;

    /**
     * 尝试非阻塞地获取锁
     * 
     * lock()和lockInterruptibly()在获取不到锁的时候，都会阻塞当前线程，直到获取到锁
     * 而该方法则不会阻塞线程，能立即获取到锁则返回true，获取不到则立即返回false
     * 
     * 该方法的常用方式如下：
     * 
     * Lock lock = ...;
     * if (lock.tryLock()) {
     * try {
     * // manipulate protected state
     * } finally {
     * lock.unlock();
     * }
     * } else {
     * // perform alternative actions
     * }}
     * 
     * 这种使用方式，可以保证只在获取到锁的时候才去释放锁
     */
    boolean tryLock();

    /**
     * 超时获取锁
     * 
     * 当前线程在以下三种情况下会返回：
     * ①当前线程在超时时间内获取到了锁，返回true
     * ②当前线程在超时时间内被中断，返回false（即该方法可以响应其他线程对该线程的中断操作）
     * ③超时时间结束，没有获取到锁，返回false
     */
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

    /**
     * 释放锁
     */
    void unlock();


    /**
     * 获取与该锁绑定的Condition
     * 
     * 当前线程只有在获得了锁，才能调用Condition的wait()方法（表示我已经到了某一条件），
     * 调用Condition的wait()方法之后，当前线程会释放锁
     */
    Condition newCondition();
}
```

**包`java.util.concurrent.locks`的类图**  
![](/assets/locks.png)  


其中：

* `AbstractOwnableSynchronizer`、`AbstractQueuedLongSynchronizer`、`AbstractQueuedSynchronizer`是同步器，是锁实现相关的内容。

* `ReentrantLock(重入锁)`和`ReentrantReadWriteLock（重入读写锁）`是具体的实现类。
* `LockSupport`是一个工具类，提供了基本的线程阻塞和唤醒功能。
* `Condition`是实现线程间实现多条件等待/通知模式用到的。





