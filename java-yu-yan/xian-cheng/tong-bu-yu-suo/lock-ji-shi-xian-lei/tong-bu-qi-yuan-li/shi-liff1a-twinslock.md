# TwinsLock

设计一个同步工具：同一个时刻，只允许两个线程同时访问，超过两个线程访问将会被阻塞。

分析：首先，确定访问模式，TwinLock能够同一时刻支持多个线程同时访问，这明显是共享式访问。这需要同步器提供的acquireShared\(int args\)方法和Share相关的方法，同时要求重写tryAcquireShared\(int args\)等方法。

其次，定义资源数量，TwinsLock在同一时刻允许两个线程同时访问资源数量为2，这样可以设置初始状态为2，当一个线程进行获取，status减1，该线程释放，则status加1。状态的合法状态为0、1、2。

* 0：无锁可用，即已经有两个线程在访问资源，此时，再有其他线程获取同步状态，该线程只能被阻塞；
* 1：已加了一把锁，还剩一把；
* 2：无锁状态；还可以加两把锁；

实现代码如下：

```java
public class TwinsLock implements Lock {

    //静态内部类继承AQS
    private static final class Sync extends AbstractQueuedSynchronizer {
        //初始化同步状态为2，表示没有线程对资源进行访问
        public Sync(int count) {
            if (count < 0) {
                throw new IllegalArgumentException("count must larger than zero.");
            }
            setState(2);
        }

        //获取同步状态：使用循环CAS操作来保证设置status的线程安全
        @Override
        protected int tryAcquireShared(int reduceCount) {
            for(;;) {
                int current = getState();
                int newCount = current - reduceCount;
                if (newCount < 0 || compareAndSetState(current, newCount)) {
                    return newCount; //结果<0,表示获取锁失败；=0，表示获取锁成功，但无剩余锁；=1，表示获取锁成功，还有剩余锁；
                }
            }
        }
        //释放同步状态：因为可能两个线程同时进行释放，需要使用循环CAS来保证安全
        @Override
        protected boolean tryReleaseShared(int returnCount) {
            for(;;) {
                int current = getState();
                int newCount = current + returnCount;
                if (compareAndSetState(current, newCount)) {
                    return true;
                }
            }
        }

        final ConditionObject newCondition() {
            return new ConditionObject();
        }

    }

    private final Sync sync = new Sync(2);

    @Override
    public void lock() {
        sync.acquireShared(1);
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    @Override
    public boolean tryLock() {
        int count = sync.tryAcquireShared(1);
        return count >= 0;
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(time));
    }

    @Override
    public void unlock() {
        sync.releaseShared(1);
    }

    @Override
    public Condition newCondition() {
        return sync.newCondition();
    }
}
```

测试

```java
public class TwinsLockExample {

    public static void main(String[] args) {

        final Lock lock = new TwinsLock();

        class Worker extends Thread {
            @Override
            public void run() {
                while (true) {
                    lock.lock();
                    try {
                        System.out.println(Thread.currentThread().getName());
                        Thread.sleep(1500);
                    } catch (InterruptedException e) {
                        System.out.println("interruptException!");
                    } finally {
                        lock.unlock();
                        break;
                    }
                }
            }
        }

        //启动10个线程
        for (int i = 0; i < 10; i++) {
            Worker worker = new Worker();
            worker.setDaemon(true);
            worker.start();
        }

        //每间隔一秒钟打印一个空行.
        for (int i = 0; i < 10; i++) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println();
        }
    }
}
//输出如下
Thread-0
Thread-1

Thread-2
Thread-3

Thread-4
Thread-5


Thread-6
Thread-7

Thread-8
Thread-9
```



