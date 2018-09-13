# Condition

任意一个Java对象都有一组监视器方法（定义在`java.lang.Object`上）：`wait()、wait(long timeout)、notify()、notifyAll()`，这些方法与`sychronized`配合使用，可以实现等待/通知模式。  
`Condition`接口也提供了类似的监视器方法，与`Lock`配合使用，可以实现等待/通知模式。

两者的区别如下：

| 对比项 | Object Monitor Methods | Condition |
| :--- | :--- | :--- |
| 前置条件 | 获取对象的锁 | 调用Lock.lock\(\)获取锁→调用Lock.newCondition\(\)获取Condition对象 |
| 调用方式 | 直接调用，如object.wait\(\) | 直接调用，如condition.await\(\) |
| 等待队列个数 | 1个 | 多个 |
| 当前线程释放锁并进入等待状态 | 支持 | 支持 |
| 当前线程释放锁并进入等待状态，在等待状态中不响应中断 | 不支持 | 支持 |
| 当前线程释放锁并进入超时等待状态 | 支持 | 支持 |
| 当前线程释放锁并进入等待状态到将来的某个时间点 | 不支持 | 支持 |
| 唤醒等待队列中的一个线程 | 支持 | 支持 |
| 唤醒等待队列中的全部线程 | 支持 | 支持 |

Condition中的方法如下：（一般会将Condition对象作为成员变量）  
说明：当前线程调用await\(\)方法后，当前线程会释放锁并在此等候，当其他线程调用signal\(\)方法通知当前线程后，当前线程才从await\(\)方法中返回，并且在返回前已经获取了锁（`re-acquire`）。

```java
public interface Condition {

    /**
     * 当前线程进入等待状态直到被通知（signalled）或中断(interrupted)
     * 
     * 如果当前线程从该方法返回，则表明当前线程已经获取了Condition对象所对应的锁
     * 
     * @throws InterruptedException
     */
    void await() throws InterruptedException;

    /**
     * 与await()不同是：该方法对中断操作不敏感
     * 
     * 如果当前线程在等待的过程中被中断，当前线程仍会继续等待，直到被通知(signalled)，
     * 但当前线程会保留线程的中断状态值
     * 
     */
    void awaitUninterruptibly();

    /**
     * 当前线程进入等待状态，直到被通知或被中断或超时
     * 
     * 返回值表示剩余时间，
     * 如果当前线程在nanosTimeout纳秒之前被唤醒，那么返回值就是（nanosTimeout-实际耗时），
     * 如果返回值是0或者负数，则表示等待已超时
     * 
     */
    long awaitNanos(long nanosTimeout) throws InterruptedException;

    /**
     * 该方法等价于awaitNanos(unit.toNanos(time)) > 0
     */
    boolean await(long time, TimeUnit unit) throws InterruptedException;

    /**
     * 当前线程进入等待状态，直到被通知或被中断或到达时间点deadline
     * 
     * 如果在没有到达截止时间就被通知，返回true
     * 如果在到了截止时间仍未被通知，返回false
     */
    boolean awaitUntil(Date deadline) throws InterruptedException;

    /**
     * 唤醒一个等待在Condition上的线程
     * 该线程从等待方法返回前必须获得与Condition相关联的锁
     */
    void signal();

    /**
     * 唤醒所有等待在Condition上的线程
     * 每个线程从等待方法返回前必须获取Condition相关联的锁
     */
    void signalAll();
}
```

**示例**

使用Condition实现一个有界阻塞队列：当队列为空时，队列的获取操作将会阻塞当前线程，直到队列中有新增元素；当队列已满时，队列的插入操作就会阻塞插入线程，直到队列中出现空位（其实就是**简化版的**`ArrayBlockingQueue`）。

```java
public class BoundedBlockingQueue<T> {
    //使用数组维护队列
    private Object[] queue;
    //当前数组中的元素个数
    private int count = 0;
    //当前添加元素到数组的位置
    private int addIndex = 0;
    //当前移除元素在数组中的位置
    private int removeIndex = 0;

    private Lock lock = new ReentrantLock();
    private Condition notEmptyCondition = lock.newCondition();
    private Condition notFullCondition = lock.newCondition();


    private BoundedBlockingQueue() {
    }

    public BoundedBlockingQueue(int capacity) {
        queue = new Object[capacity];
    }

    public void put(T t) throws InterruptedException {
        lock.lock();//获得锁，保证内部数组修改的可见性和排他性
        try {
            //使用while，而非if：防止过早或意外的通知，
            //加入当前线程释放了锁进入等待状态，然后其他线程进行了signal，
            //则当前线程会从await()方法中返回，再次判断count == queue.length
            //todo：哪些情况下的过早或意外？？？
            while (count == queue.length) {
                notFullCondition.await();//释放锁，等待队列不满，即等待队列出现空位
            }
            queue[addIndex] = t;
            addIndex++;
            if (addIndex == queue.length) {
                addIndex = 0;
            }
            count++;
            notEmptyCondition.signal();//通知等待队列非空的线程，可以从队列中取元素了
        } finally {
            //确保会释放锁
            lock.unlock();
        }
    }

    @SuppressWarnings("unchecked")
    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) {
                notEmptyCondition.await();//释放锁，等待队列不为空，即等待队列中至少有一个元素
            }
            Object x = queue[removeIndex];
            removeIndex++;
            if (removeIndex == queue.length) {
                removeIndex = 0;
            }
            count--;
            notFullCondition.signal();//通知那些等待队列非满的线程，可以向队列中插入元素了
            return (T) x;
        } finally {
            //确保会释放锁
            lock.unlock();
        }
    }
}
```

**Condition原理**





**内容来源：**

大部分来自《Java并发编程的艺术》，部分参考JDK中的注释说明。

