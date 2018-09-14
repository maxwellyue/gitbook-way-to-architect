### **AQS中的主要方法**

同步器的主要使用方式是继承，一般作为同步器组件的静态内部类。同步器中仅定义了与状态相关的方法，且这个状态既可以独占地获取又可以共享的获取，这样就可以实现不同类型的同步组件（ReetrantLock、ReetrantReadWriteLock和CountDownLatch等）。它简化了锁的实现方式，屏蔽了同步状态管理、线程的排队、等待与唤醒等底层操作。

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



