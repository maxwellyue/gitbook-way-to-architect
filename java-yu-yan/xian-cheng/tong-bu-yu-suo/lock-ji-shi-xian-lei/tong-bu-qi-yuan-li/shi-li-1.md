# Mutex

Mutex是AQS的注释中出现的一个示例，它是一个独占锁：在同一时刻，只允许一个线程占有锁。

```java
public class Mutex implements Lock, java.io.Serializable {

    //使用静态内部类的方式，实现AQS
    private static class Sync extends AbstractQueuedSynchronizer {
        // 当前同步器是否在独占模式下被线程占用，即是否被当前线程独占
        @Override
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        // 当status = 0时，获取锁
        @Override
        public boolean tryAcquire(int acquires) {
            assert acquires == 1; // Otherwise unused
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        // 通过将状态设置为0，来释放锁
        @Override
        protected boolean tryRelease(int releases) {
            assert releases == 1; // Otherwise unused
            if (getState() == 0) throw new IllegalMonitorStateException();
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        // Provides a Condition
        Condition newCondition() {
            return new ConditionObject();
        }

        // Deserializes properly
        private void readObject(ObjectInputStream s)
                throws IOException, ClassNotFoundException {
            s.defaultReadObject();
            setState(0); // reset to unlocked state
        }
    }

    //将复杂逻辑都交给同步器去处理，实现类只需要使用即可
    private final Sync sync = new Sync();

    @Override
    public void lock() {
        sync.acquire(1);
    }

    @Override
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    @Override
    public void unlock() {
        sync.release(1);
    }

    @Override
    public Condition newCondition() {
        return sync.newCondition();
    }

    public boolean isLocked() {
        return sync.isHeldExclusively();
    }

    public boolean hasQueuedThreads() {
        return sync.hasQueuedThreads();
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    @Override
    public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }
}
```

**Mutex使用**

```java
public class Counter {

    private int num;

    private Mutex mutex = new Mutex();

    public Counter(int num){
        this.num = num;
    }

    public int getNum() {
        return num;
    }

    public void increase(){
        mutex.lock();
        num++;
        mutex.unlock();
    }
}

public class MutexExample {

    public static void main(String[] args) throws InterruptedException {
        Counter counter = new Counter(0);
        CountDownLatch latch = new CountDownLatch(1000);
        ExecutorService threadPool = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 1000; i++) {
            threadPool.execute(() -> {
                counter.increase();
                latch.countDown();
            });
        }
        latch.await();
        System.out.println(counter.getNum());
    }
}
//输出如下：
1000
```



