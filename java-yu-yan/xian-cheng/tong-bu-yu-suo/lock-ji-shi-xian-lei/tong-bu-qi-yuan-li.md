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



## 参考内容

[Java 并发编程 ----- AQS（抽象队列同步器）](https://juejin.im/post/5afb9ab3f265da0b736dd1e1)

[队列同步器（AQS）详解](https://blog.csdn.net/summer_yuxia/article/details/71452310)

[Java中的锁——队列同步器](http://rolinyin.iteye.com/blog/2340159)

《Java并发编程的艺术》：Java中的锁

