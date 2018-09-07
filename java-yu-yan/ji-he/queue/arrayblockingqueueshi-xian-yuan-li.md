ArrayBlockingQueue

有界阻塞队列，初始化的时候必须要指定队列长度，且指定长度之后不允许进行修改，内部使用数组持有元素。

它的主要属性如下

```java
// 存储队列元素的数组，是个循环数组
final Object[] items;

// 拿数据的索引，用于take，poll，peek，remove方法
int takeIndex;

// 放数据的索引，用于put，offer，add方法
int putIndex;

// 元素个数
int count;

// 可重入锁
final ReentrantLock lock;
// notEmpty条件对象，由lock创建
private final Condition notEmpty;
// notFull条件对象，由lock创建
private final Condition notFull;
```

offer\(\)方法：

```java
public boolean offer(E e) {
    checkNotNull(e); // 不允许元素为空
    final ReentrantLock lock = this.lock;
    lock.lock(); // 加锁，保证只有1个线程进行offer
    try {
        if (count == items.length) // 如果队列已满
            return false; // 直接返回false，添加失败
        else {
            insert(e); // 数组没满的话调用insert方法
            return true; // 返回true，添加成功
        }
    } finally {
        lock.unlock(); // 释放锁，让其他线程可以调用offer方法
    }
}
```

其中的insert方法如下：

```java
private void insert(E x) {
    items[putIndex] = x; // 元素添加到数组里
    putIndex = inc(putIndex); // 放数据索引+1，当索引满了变成0
    ++count; // 元素个数+1
    notEmpty.signal(); // 使用条件对象notEmpty通知，比如使用take方法的时候队列里没有数据，被阻塞。这个时候队列insert了一条数据，需要调用signal进行通知
}
```

add\(\)方法：

```java
public boolean add(E e) {
    if (offer(e))
        return true;
    else
        throw new IllegalStateException("Queue full");
}
```

put方法：

```java
public void put(E e) throws InterruptedException {
    checkNotNull(e); // 不允许元素为空
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly(); // 加锁，保证调用put方法的时候只有1个线程
    try {
        while (count == items.length) // 如果队列满了，阻塞当前线程，并加入到条件对象notFull的等待队列里
            notFull.await(); // 线程阻塞并被挂起，同时释放锁
        insert(e); // 调用insert方法
    } finally {
        lock.unlock(); // 释放锁，让其他线程可以调用put方法
    }
}
```













