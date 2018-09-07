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

可见，ArrayBlockingQueue只有1个锁，同一时刻，要么添加数据，要么删除数据，两者不能并行执行。

## 插入元素

offer\(\)方法：如果队列已满，直接返回fasle

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

add\(\)方法：如果队列已满，抛出IllegalStateException\("Queue full"\)异常；

```java
public boolean add(E e) {
    if (offer(e))
        return true;
    else
        throw new IllegalStateException("Queue full");
}
```

put\(\)方法：如果队列已满，当前线程会一直阻塞，直到队列中出现空位或响应中断退出

```java
public void put(E e) throws InterruptedException {
    checkNotNull(e); // 不允许元素为空
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly(); // 加锁(但可相应中断)，保证调用put方法的时候只有1个线程
    try {
        while (count == items.length) // 如果队列满了，阻塞当前线程，并加入到条件对象notFull的等待队列里
            notFull.await(); // 线程阻塞并被挂起，同时释放锁
        insert(e); // 调用insert方法
    } finally {
        lock.unlock(); // 释放锁，让其他线程可以调用put方法
    }
}
```

## 移除元素

poll\(\)方法：如果队列为空，直接返回null

```java
public E poll() {
    final ReentrantLock lock = this.lock;
    lock.lock(); // 加锁，保证调用poll方法的时候只有1个线程
    try {
        return (count == 0) ? null : extract(); // 如果队列里没元素了，返回null，否则调用extract方法
    } finally {
        lock.unlock(); // 释放锁，让其他线程可以调用poll方法
    }
}
```

poll\(\)方法内部调用extract\(\)方法：

```java
private E extract() {
    final Object[] items = this.items;
    E x = this.<E>cast(items[takeIndex]); // 得到取索引位置上的元素
    items[takeIndex] = null; // 对应取索引上的数据清空
    takeIndex = inc(takeIndex); // 取数据索引+1，当索引满了变成0
    --count; // 元素个数-1
    notFull.signal(); // 使用条件对象notFull通知，比如使用put方法放数据的时候队列已满，被阻塞。这个时候消费了一条数据，队列没满了，就需要调用signal进行通知
    return x; // 返回元素
}
```

take\(\)方法：如果队列为空，当前线程会一直阻塞，直到队列中有元素插入或响应中断退出；

```java
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly(); // 加锁，保证调用take方法的时候只有1个线程
    try {
        while (count == 0) // 如果队列空，阻塞当前线程，并加入到条件对象notEmpty的等待队列里
            notEmpty.await(); // 线程阻塞并被挂起，同时释放锁
        return extract(); // 调用extract方法
    } finally {
        lock.unlock(); // 释放锁，让其他线程可以调用take方法
    }
}
```

remove\(\)方法：如果队列为空，则抛出NoSuchElementException异常

```java
public boolean remove(Object o) {
    if (o == null) return false;
    final Object[] items = this.items;
    final ReentrantLock lock = this.lock;
    lock.lock(); // 加锁，保证调用remove方法的时候只有1个线程
    try {
        for (int i = takeIndex, k = count; k > 0; i = inc(i), k--) { // 遍历元素
            if (o.equals(items[i])) { // 两个对象相等的话
                removeAt(i); // 调用removeAt方法
                return true; // 删除成功，返回true
            }
        }
        return false; // 删除成功，返回false
    } finally {
        lock.unlock(); // 释放锁，让其他线程可以调用remove方法
    }
}
```

内容来源：[Java阻塞队列ArrayBlockingQueue和LinkedBlockingQueue实现原理分析](https://fangjian0423.github.io/2016/05/10/java-arrayblockingqueue-linkedblockingqueue-analysis/)

