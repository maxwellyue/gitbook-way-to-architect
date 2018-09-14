# 队列同步器

### 概念

AbstractQueuedSynchronizer**\(AQS\)**，是用来构建锁或者其他同步组件的基础框架，它使用了一个int成员变量（**status**）表示共享资源的同步状态，通过内置的**FIFO队列**来完成资源获取线程的排队工作。

* status：多线程共享某资源时，该共享资源的状态（是否已经被其他线程锁住等），使用这个status来表示。

* FIFO队列：当一些线程无法获取到共享资源，等待获取锁时，会放在一个容器中进行保存，这个容器就是FIFO队列。

在FIFO队列中，节点表示被阻塞的线程，队列节点元素有4种类型， 每种类型表示线程被阻塞的原因，这四种类型分别是：

* `CANCELLED`: 表示该线程是因为超时或者中断原因而被放到队列中
* `CONDITION`: 表示该线程是因为某个条件不满足而被放到队列中，需要等待一个条件，直到条件成立后才会出队
* `SIGNAL`: 表示该线程需要被唤醒
* `PROPAGATE`： 表示在共享模式下，当前节点执行释放`release`操作后，当前结点需要传播通知给后面所有节点

由于一个共享资源同一时间只能由一条线程持有，也可以被多个线程持有，因此AQS中存在以下两种模式：

* **独占模式**
* * 表示共享状态值state每次只能由一条线程持有，其他线程如果需要获取，则需要阻塞，如JUC中的`ReentrantLock`
* **共享模式**

  * 表示共享状态值state每次可以由多个线程持有，如JUC中的`CountDownLatch`

### AQS中的共享状态值

AQS是基于一个共享的int类型的state值来实现同步器同步的：

```java
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer implements java.io.Serializable {  

    ... ...

    private volatile int state;//*同步状态*  

    //获取当前同步状态
    protected final int getState() {
        return state;
    }
    //设置当前同步状态
    protected final void setState(int newState) {
        state = newState;
    }
    //使用CAS设置当前状态，保证“状态设置”这个操作的原子性
    protected final boolean compareAndSetState(int expect, int update) {
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }

    ... ...  
}
```

多线程共享某资源时，该共享资源的状态（是否已经被其他线程锁住等），AQS使用status来表示。为了达到多线程同步的功能，必然对该值的修改必须多线程可见，因此，state采用volatile修饰，而且`getter`和`seter`方法采用final进行修饰，目的是限制AQS的子类只能调用这两个方法对state的值进行设置和获取，而不能对其进行重写自定义设置/获取逻辑。

### **AQS中FIFO队列中节点的数据结构**

```java
    //标识节点为共享模式
    static final Node SHARED = new Node();
    //标识节点为独占模式
    static final Node EXCLUSIVE = null;

    //标识线程由于中断或超时，需要被取消，即踢出队列
    static final int CANCELLED =  1;
    //标识线程需要被唤醒
    static final int SIGNAL    = -1;
    //标识线程正在等待一个条件
    static final int CONDITION = -2;
    //标识为传播模式
    static final int PROPAGATE = -3;

    //状态值，只能从上面的CANCELLED/SIGNAL/CONDITION/PROPAGATE选择其一
    volatile int waitStatus;

    //前驱节点
    volatile Node prev;

    //后继节点
    volatile Node next;

    //节点关联的线程对象
    volatile Thread thread;

    //下一个waitStatus值为CONDITION或为共享模式的节点
    Node nextWaiter;

    //当前节点是否为共享模式
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    //返回前一个节点
    final Node predecessor() throws NullPointerException {
        ... ... 
    }

    Node() {    // Used to establish initial head or SHARED marker
    }

    Node(Thread thread, Node mode) {     // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }

    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

当前线程获取同步状态失败时，同步器会将当前线程以及等待状态等信息构造成一个Node节点，并将其加入到同步队列，同时会阻塞当前线程。当同步状态释放时，同步器会把同步队列中的首节点中的线程唤醒，使其再次尝试获取同步状态。

### AQS同步队列的维护

如下，是同步器内部队列的示例图。

图1

没有成功获取同步状态的线程将会成为节点加入该队列的尾部，这个加入队列的过程必须要保证线程安全，AQS使用CAS方法（`compareAndSetTail(Node expect, Node update)`）来完成这个操作。

同步器的首节点是成功获取同步状态的节点。首节点的线程在释放同步状态时，将会唤醒后续节点，而后续节点将会在成功获取同步状态后，将自己设置为首节点。设置首节点这步操作是由成功获取同步状态的线程来完成的，由于只有一个线程能够成功获取同步状态，所以这步操作并不需要CAS来保证。

#### 独占式获取与释放同步状态

若要独占式地获取同步状态，可以通过同步器的tryAcquire\(int arg\) 方法实现：

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





### **AQS中的主要方法**

同步器的主要使用方式是继承，一般作为同步器组件的静态内部类，在同步器中仅定义了与状态相关的方法，且这个状态既可以独占地获取又可以共享的获取，这样就可以实现不同类型的同步组件（ReetrantLock、ReetrantReadWriteLock和CountDownLatch等）。它简化了锁的实现方式，屏蔽了同步状态管理、线程的排队、等待与唤醒等底层操作。

**同步器中可重写的方法如下：**

| 方法 | 说明 |
| :--- | :--- |
| boolean tryAcquire\(int arg\) | 独占地获取同步状态，实现该方法需要检查当前状态并判断同步状态是否符合预期，然后再进行CAS设置状态 |
| boolean tryRelease\(int arg\) | 独占式释放同步状态，等待获取同步状态的线程有机会获取同步状态 |
| int tryAcquireShared\(int arg\) | 共享式获取同步状态，返回值≥0则表示获取成功，否则失败 |
| boolean tryReleaseShared\(int arg\) | 共享式释放同步状态 |
| boolean isHeldExclusively\(\) | 当前同步器是否在独占模式下被线程占用，一般该方法表示是否被当前线程所独占 |

**同步器中的模板方法如下：**

| 方法 | 说明 |
| --- | --- |
| void acquire\(int arg\) | 独占式获取同步状态，如果当前线程获取同步状态成功，则由该方法返回，否则，将会进入同步队列等待，该方法将会调用重写的tryAcquire\(int arg\) 方法。 |
| void acquireInterruptibly\(int arg\) | 与acquire\(int arg\) 相同，但是该方法响应中断，当前线程未获取到同步状态而进入同步队列中，如果当前被中断，则该方法会抛出InterruptedException并返回。 |
| boolean tryAcquireNanos\(int arg,long nanos\) | 在acquireInterruptibly基础上增加了超时限制，如果当前线程在超时时间内没有获取到同步状态，那么将会返回false，获取到了返回true。 |
| boolean release\(int arg\) | 独占式的释放同步状态，该方法会在释放同步状态之后，将同步队列中第一个节点包含的线程唤醒 |
| void acquireShared\(int arg\) | 共享式获取同步状态，如果当前线程未获取到同步状态，将会进入同步队列等待。与独占式的不同是同一时刻可以有多个线程获取到同步状态。 |
| void acquireSharedInterruptibly\(int arg\) | 与acquire\(int arg\) 相同，但是该方法响应中断，当前线程未获取到同步状态而进入同步队列中，如果当前被中断，则该方法会抛出InterruptedException并返回。 |
| boolean tryAcquireSharedNanos\(int arg,long nanos\) | 在acquireSharedInterruptibly基础上增加了超时限制，如果当前线程在超时时间内没有获取到同步状态，那么将会返回false，获取到了返回true |
| boolean releaseShared\(int arg\) | 共享式释放同步状态 |
| Collection&lt;Thread&gt; getQueuedThreads\(\) | 获取等待在同步队列上的线程集合 |

参考内容

[Java 并发编程 ----- AQS（抽象队列同步器）](https://juejin.im/post/5afb9ab3f265da0b736dd1e1)

[队列同步器（AQS）详解](https://blog.csdn.net/summer_yuxia/article/details/71452310)

[Java中的锁——队列同步器](http://rolinyin.iteye.com/blog/2340159)

《Java并发编程的艺术》：Java中的锁

