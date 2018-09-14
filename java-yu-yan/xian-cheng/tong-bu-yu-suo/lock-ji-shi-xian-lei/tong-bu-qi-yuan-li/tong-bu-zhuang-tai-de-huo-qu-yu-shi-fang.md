### AQS同步队列的维护

如下，是同步器内部队列的示例图。![](/assets/屏幕快照 2018-09-14 上午12.07.46.png)没有成功获取同步状态的线程将会成为节点加入该队列的尾部，这个加入队列的过程必须要保证线程安全，AQS使用CAS方法（`compareAndSetTail(Node expect, Node update)`）来完成这个操作。

同步器的首节点是成功获取同步状态的节点。首节点的线程在释放同步状态时，将会唤醒后续节点，而后续节点将会在成功获取同步状态后，将自己设置为首节点。设置首节点这步操作是由成功获取同步状态的线程来完成的，由于只有一个线程能够成功获取同步状态，所以这步操作并不需要CAS来保证。

## 独占式获取与释放同步状态

**获取同步状态**

若要独占式地获取同步状态，可以通过同步器的acquire\(int arg\) 方法实现：

```java
public final void acquire(int arg) {
if (!tryAcquire(arg) &&
    acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
    selfInterrupt();
}
```

如果当前线程获取同步状态成功，则由该方法返回，否则，将会进入同步队列等待；该方法对中断不敏感，也就是说当线程由于获取同步状态失败后进入队列中，后续对线程进行中断操作时，线程不会从队列中移出。

上述代码做的事情：获取同步状态、构造节点、加入队列以及在队列中自旋等待。主要逻辑为：首先调用自定义同步器实现的tryAcquire\(int arg\) 方法，该方法保证线程安全地获取同步状态，如果获取失败，则构造同步节点（Node.EXCLUSIVE，同一时刻只能有一个线程成功地获取同步状态），并通过addWaiter\(Node.EXCLUSIVE\), arg\)方法将其加入到队列尾部，最后调用acquireQueued\(\)方法，通过死循环的方式获取同步状态，如果获取不到，则阻塞节点中的线程，而被阻塞线程的唤醒主要依靠前驱节点的出队或阻塞线程被中断来实现。

```java
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // 尝试通过一次CAS操作将其加到队尾
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    //快速尝试不成功，则d调用enq(node)方法
    enq(node);
    return node;
}

private Node enq(final Node node) {
    //通过死循环CAS操作，来确保节点被线程安全地添加到队尾
    for (;;) {
        Node t = tail;
        if (t == null) { // 队列未初始化的情况
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {//正常情况
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

节点进入队列后，进入了一个自旋的过程，每个节点（或者说每个节点关联的线程）都在自省地检查，当条件满足，获取了同步状态，则从这个自旋过程中退出，否则依旧留在这个自旋过程中（相当于阻塞线程）：

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        //无限循环
        for (;;) {
            //获取当前节点的前驱节点
            final Node p = node.predecessor();
            //如果前驱节点是头节点，则进行获取同步状态
            if (p == head && tryAcquire(arg)) {
                //当前节点获取同步状态成功，则将当前节点设置为头节点，并从自旋中返回
                //（因为只可能有一个线程获取到同步状态，所以这里不需要通过CAS操作来设置）
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }

            if (shouldParkAfterFailedAcquire(p, node) && //如果应该阻塞线程
                parkAndCheckInterrupt())//阻塞线程，直到被中断才返回true，否则就一直阻塞
                interrupted = true;     //阻塞过程被中断，则会继续从头开始for循环
        }                               //这样就保证：队列中某个节点的前驱节点不是头节点，但是被中断了，
    } finally {                         //也不会从循环中逃出，即不会出队，保证了FIFO的性质
        if (failed)
            cancelAcquire(node);
    }
}
```

独占式获取同步状态的整个流程如下：

![](/assets/屏幕快照 2018-09-14 下午10.27.11.png)

**释放同步状态**

当前线程在成功获取同步状态并执行完自己的业务逻辑之后，就需要释放同步状态，使得后续节点能够继续获取同步状态。

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            //唤醒后继节点（这里的unpark与前面的park是一对相反的操作）
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

总结：获取状态失败的线程都会被加入到队尾，并在队列中自旋；移出队列的条件是当前节点的前驱节点为头节点并且成功获取到了同步状态。在释放同步状态时，会唤醒头节点的后继节点。

## 共享式同步状态获取与释放

**共享式获取同步状态**

共享式获取与独占式获取最主要的区别在于同一时刻能否有多个线程同时获取到同步状态。

> 以文件的读写为例，若一个程序在对文件进行读操作，那么这一时刻对于该文件的写操作均被阻塞，而读操作能够同时进行。写操作要求对资源的独占式访问，而读操作可以是共享式访问。

若要共享式地获取同步状态，可以通过同步器的acquireShared\(int arg\) 方法实现：

```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)//调用实现类的获取同步状态的方法，该方法返回值≥0的时候，表示成功获取同步状态
        doAcquireShared(arg);
}

private void doAcquireShared(int arg) {
    //构造共享式节点，并将其通过CAS线程安全地添加到队尾
    final Node node = addWaiter(Node.SHARED);
    //下面的逻辑整体与独占式的类似
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

与独占式一样，共享式获取后，也需要释放状态：

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   // loop if head changed
            break;
    }
}
```

与独占式不同，共享式释放同步状态这个操作可能来自多个线程，一般通过无限循环CAS来保证安全。

## 独占式超时获取同步状态

TODO

