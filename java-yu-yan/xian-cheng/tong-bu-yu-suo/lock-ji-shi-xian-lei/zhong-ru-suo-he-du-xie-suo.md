# ReentrantLock

ReentrantLock，重入锁，支持重新进入的锁：即某线程在获取到锁之后，可以再次获取锁而不会被阻塞。

其主要特性如下：

* **支持重入**：通过组合自定义同步器来来实现重入特性
* * `synchronized`关键字也隐式地支持重进入，比如`synchronized`修饰的递归方法，在方法执行时，执行线程在获取了锁之后仍能连续多次地获得该锁，不会出现阻塞自己的情况；

* **支持公平地获取锁：**获取锁的顺序与请求锁的顺序是相同的，等待时间最长的线程最优先获取到锁。
* **支持绑定多个Condition**。

**非公平获取锁重进入的实现**

```java
final boolean nonfairTryAcquire(int acquires) {
      final Thread current = Thread.currentThread();
      int c = getState();
      if (c == 0) {
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }else if (current == getExclusiveOwnerThread()) {
        //如果是当前持有锁的线程再次获取锁，则将同步值进行增加并返回true
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
}
```

**公平锁重进入的实现**

```java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

与非公平获取锁的方法`nonfairTryAcquire(int acquires)`相比，多了一个`hasQueuedPredecessors()`判断：同步队列中当前节点（当前想要获取锁的线程）是否有前驱节点，如果该方法返回true，则表示有线程比当前线程更早地请求获取锁，因此需要前驱线程获取并释放锁之后才能继续获取锁。

公平锁保证了锁的获取按照FIFO原则，而代价是进行大量的线程切换；  
非公平锁虽然可能造成线程“饥饿”（即某线程可能需要等很久才得到锁），但线程切换极少，可以保证更大的吞吐量。

---

### ReentrantReadWriteLock

`ReentrantLock`在同一时刻，只允许一个线程进行访问（无论读还是写）。而读写锁是指：在同一时刻，允许多个线程进行读操作，而写操作则会阻塞其他所有的线程（无论是读还是写，都会被阻塞）。读写锁维护了一对锁：读锁和写锁，通过分离读锁和写锁，使得并发性能相比一般的排他锁有了很大的提升。

它的特性如下：

* 重进入；
* 公平性选择
* 锁降级：写锁可以降级为读锁，

此外，ReentrantReadWriteLock还提供了一些便于外界监控其内部状态的方法，如下：

```java
int getReadLockCount()
返回当前读锁被获取的次数
注意：该次数并不等于获取读锁的线程数，
因为同一线程可以连续获得多次读锁，获取一次，返回值就加1，
比如，仅一个线程，它连续获得了n次读锁，那么占据读锁的线程数是1，但该方法返回n

int getReadHoldCount()
返回当前线程获取读锁的次数

boolean isWriteLock()
判断读锁是否被获取

int getWriteHoldCount()
返回当前写锁被获取的次数
```

**示例：使用ReentrantReadWriteLock实现Cache**

```java
public class Cache{

    //非线程安全的HashMap
    private static Map<String, Object> map = new HashMap<>();
    //读写锁
    private static ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();
    //读锁
    private static Lock readLock = reentrantReadWriteLock.readLock();
    //写锁
    private static Lock writeLock = reentrantReadWriteLock.writeLock();

    /**
     * 获取key对应的value
     * 
     * 使用读锁，使得并发访问该方法时不会被阻塞
     */
    public static final Object get(String key){
        readLock.lock();
        try{
            return map.get(key);
        }finally {
            readLock.unlock();
        }
    }

    /**
     * 设置key对应的value
     * 
     * 当有线程对map进行put操作时，使用写锁，阻塞其他线程的读、写操作，
     * 只有在写锁被释放后，其他读写操作才能继续
     */
    public static Object put(String key, Object value){
        writeLock.lock();
        try {
            return map.put(key, value);
        }finally {
            writeLock.unlock();
        }
    }

    /**
     * 清空map
     *
     * 当有线程对map进行清空操作时，使用写锁，阻塞其他线程的读、写操作，
     * 只有在写锁被释放后，其他读写操作才能继续
     */
    public static void clear(){
        writeLock.lock();
        try {
            map.clear();
        }finally {
            writeLock.unlock();
        }
    }
}
```

TODO:读写锁的实现原理













