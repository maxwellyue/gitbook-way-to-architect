LinkedBlockingQueue

使用链表（**单向**）完成队列操作的阻塞队列。

它的主要属性如下：

```java
// 容量大小
private final int capacity;

// 元素个数，因为有2个锁，存在竞态条件，使用AtomicInteger
private final AtomicInteger count = new AtomicInteger(0);

// 头结点
private transient Node<E> head;

// 尾节点
private transient Node<E> last;

// 拿锁
private final ReentrantLock takeLock = new ReentrantLock();

// 拿锁的条件对象
private final Condition notEmpty = takeLock.newCondition();

// 放锁
private final ReentrantLock putLock = new ReentrantLock();

// 放锁的条件对象
private final Condition notFull = putLock.newCondition();
```

LinkedBlockingQueue有2个锁（takeLock和putLock），添加数据和删除数据是可以并行进行的，当然添加数据和删除数据的时候只能有1个线程各自执行。

## 插入元素

offer\(\)方法：如果队列已满，直接返回fasle

```java
public boolean offer(E e) {
    if (e == null) throw new NullPointerException(); // 不允许空元素
    final AtomicInteger count = this.count;
    if (count.get() == capacity) // 如果容量满了，返回false
        return false;
    int c = -1;
    Node<E> node = new Node(e); // 容量没满，以新元素构造节点
    final ReentrantLock putLock = this.putLock;
    putLock.lock(); // 放锁加锁，保证调用offer方法的时候只有1个线程
    try {
        if (count.get() < capacity) { // 再次判断容量是否已满，因为可能拿锁在进行消费数据，没满的话继续执行
            enqueue(node); // 节点添加到链表尾部
            c = count.getAndIncrement(); // 元素个数+1
            if (c + 1 < capacity) // 如果容量还没满
                notFull.signal(); // 在放锁的条件对象notFull上唤醒正在等待的线程，表示可以再次往队列里面加数据了，队列还没满
        }
    } finally {
        putLock.unlock(); // 释放放锁，让其他线程可以调用offer方法
    }
    if (c == 0) // 由于存在放锁和拿锁，这里可能拿锁一直在消费数据，count会变化。这里的if条件表示如果队列中还有1条数据
        signalNotEmpty(); // 在拿锁的条件对象notEmpty上唤醒正在等待的1个线程，表示队列里还有1条数据，可以进行消费
    return c >= 0; // 添加成功返回true，否则返回false
}
```

抛出IllegalStateException\("Queue full"\)异常；

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
    if (e == null) throw new NullPointerException(); // 不允许空元素
    int c = -1;
    Node<E> node = new Node(e); // 以新元素构造节点
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    putLock.lockInterruptibly(); // 放锁加锁，保证调用put方法的时候只有1个线程
    try {
        while (count.get() == capacity) { // 如果容量满了
            notFull.await(); // 阻塞并挂起当前线程
        }
        enqueue(node); // 节点添加到链表尾部
        c = count.getAndIncrement(); // 元素个数+1
        if (c + 1 < capacity) // 如果容量还没满
            notFull.signal(); // 在放锁的条件对象notFull上唤醒正在等待的线程，表示可以再次往队列里面加数据了，队列还没满
    } finally {
        putLock.unlock(); // 释放放锁，让其他线程可以调用put方法
    }
    if (c == 0) // 由于存在放锁和拿锁，这里可能拿锁一直在消费数据，count会变化。这里的if条件表示如果队列中还有1条数据
        signalNotEmpty(); // 在拿锁的条件对象notEmpty上唤醒正在等待的1个线程，表示队列里还有1条数据，可以进行消费
}
```

> 在消费的时候进行唤醒插入阻塞的线程，同时在插入的时候如果容量还没满，也会唤醒插入阻塞的线程

## 移除元素

poll\(\)方法：如果队列为空，直接返回null

```java
public E poll() {
    final AtomicInteger count = this.count;
    if (count.get() == 0) // 如果元素个数为0
        return null; // 返回null
    E x = null;
    int c = -1;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lock(); // 拿锁加锁，保证调用poll方法的时候只有1个线程
    try {
        if (count.get() > 0) { // 判断队列里是否还有数据
            x = dequeue(); // 删除头结点
            c = count.getAndDecrement(); // 元素个数-1
            if (c > 1) // 如果队列里还有元素
                notEmpty.signal(); // 在拿锁的条件对象notEmpty上唤醒正在等待的线程，表示队列里还有数据，可以再次消费
        }
    } finally {
        takeLock.unlock(); // 释放拿锁，让其他线程可以调用poll方法
    }
    if (c == capacity) // 由于存在放锁和拿锁，这里可能放锁一直在添加数据，count会变化。这里的if条件表示如果队列中还可以再插入数据
        signalNotFull(); // 在放锁的条件对象notFull上唤醒正在等待的1个线程，表示队列里还能再次添加数据
                return x;
}
```

take\(\)方法：如果队列为空，当前线程会一直阻塞，直到队列中有元素插入或响应中断退出；

```java
public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly(); // 拿锁加锁，保证调用take方法的时候只有1个线程
    try {
        while (count.get() == 0) { // 如果队列里已经没有元素了
            notEmpty.await(); // 阻塞并挂起当前线程
        }
        x = dequeue(); // 删除头结点
        c = count.getAndDecrement(); // 元素个数-1
        if (c > 1) // 如果队列里还有元素
            notEmpty.signal(); // 在拿锁的条件对象notEmpty上唤醒正在等待的线程，表示队列里还有数据，可以再次消费
    } finally {
        takeLock.unlock(); // 释放拿锁，让其他线程可以调用take方法
    }
    if (c == capacity) // 由于存在放锁和拿锁，这里可能放锁一直在添加数据，count会变化。这里的if条件表示如果队列中还可以再插入数据
        signalNotFull(); // 在放锁的条件对象notFull上唤醒正在等待的1个线程，表示队列里还能再次添加数据
    return x;
}
```

remove\(\)方法：如果队列为空，则抛出NoSuchElementException异常

```java
public boolean remove(Object o) {
    if (o == null) return false;
    fullyLock(); // remove操作要移动的位置不固定，2个锁都需要加锁
    try {
        for (Node<E> trail = head, p = trail.next; // 从链表头结点开始遍历
             p != null;
             trail = p, p = p.next) {
            if (o.equals(p.item)) { // 判断是否找到对象
                unlink(p, trail); // 修改节点的链接信息，同时调用notFull的signal方法
                return true;
            }
        }
        return false;
    } finally {
        fullyUnlock(); // 2个锁解锁
    }
}
```

内容来源：[Java阻塞队列ArrayBlockingQueue和LinkedBlockingQueue实现原理分析](https://fangjian0423.github.io/2016/05/10/java-arrayblockingqueue-linkedblockingqueue-analysis/)

