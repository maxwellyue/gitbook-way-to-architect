JDK中提供了两类并发工具类：

* 并发流程控制相关：`CountDownLatch`、`CyclicBarrier`、`Semaphore`
* 线程间交换数据相关：`Exchanger`

### CountDownLatch

* 作用：允许一个或多个线程等待其他线程完成操作
* 使用步骤：
①定义一个CountDownLatch（称为计数器），并指定等待次数；
②在合适的时机将计数器减1；
③在需要等待所有任务结束的位置，调用await()方法；

根据JDK中的说明文档整理的两个例子：
例子1：

```
public class CountDownLatchLearning {

    public void doSomething() {

        CountDownLatch startSignal = new CountDownLatch(1);
        CountDownLatch doneSignal = new CountDownLatch(10);

        //创建并启动线程
        for (int i = 0; i < 10; ++i) {
            new Thread(new Worker(startSignal, doneSignal)).start();
        }

        doSomeThingBeforeAllThreadsProcess();
        startSignal.countDown();      //让之前for循环创建的线程开始真正工作
        try {
            doneSignal.await();       // 等待之前for循环创建的线程执行结束
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        doSomeThingAfterAllThreadsProcess();
    }

    private void doSomeThingAfterAllThreadsProcess() {
        //所有任务开始前，做一些准备工作
    }

    private void doSomeThingBeforeAllThreadsProcess() {
        //所有任务开始后，做一些其他工作，如合并结果等等
    }

    class Worker implements Runnable {

        private final CountDownLatch startSignal;
        private final CountDownLatch doneSignal;

        Worker(CountDownLatch startSignal, CountDownLatch doneSignal) {
            this.startSignal = startSignal;
            this.doneSignal = doneSignal;
        }

        @Override
        public void run() {
            try {
                startSignal.await();//等待，开始信号为0再继续向下进行
                doWork();
            } catch (InterruptedException ex) {
                ex.printStackTrace();
            }finally {
                doneSignal.countDown();//完成后，将结束信号减1
            }
        }

        void doWork() {
            //这里是真正有意义的任务
        }
    }
}
```

例子2：

```
public class CountDownLatchLearning1 {

    public void doSomething() {

        CountDownLatch doneSignal = new CountDownLatch(100);

        Executor e = Executors.newFixedThreadPool(10);
        
        for (int i = 0; i < 100; ++i) {
            e.execute(new WorkerRunnable(doneSignal, i));
        }

        try {
            doneSignal.await();           // 等待所有任务结束
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
    }

    private void doSomeThingAfterAllThreadsProcess() {
        //所有任务开始前，做一些准备工作
    }

    private void doSomeThingBeforeAllThreadsProcess() {
        //所有任务开始后，做一些其他工作，如合并结果等等
    }

    class WorkerRunnable implements Runnable {

        private final CountDownLatch doneSignal;
        private final int i;

        WorkerRunnable(CountDownLatch doneSignal, int i) {
            this.doneSignal = doneSignal;
            this.i = i;
        }

        @Override
        public void run() {
            doWork(i);
            doneSignal.countDown();
        }

        void doWork(int i) {
            //这里是真正的有意义的任务
        }
    }
}
```

### CyclicBarrier

* 作用：让一组线程等待至某个状态之后再全部同时执行，适用于多线程计算数据，最后合并计算结果的场景。
* 使用：

```
①构造：
public CyclicBarrier(int parties, Runnable barrierAction) {}
public CyclicBarrier(int parties) {}
其中：
parties指让多少个线程或者任务等待至barrier状态；
barrierAction指当这些线程都达到barrier状态时会执行的内容；

②在合适的时机调用await方法，告诉CyclicBarrier我（当前线程）已经达到了屏障，然后当前线程被阻塞
public int await();
public int await(long timeout, TimeUnit unit);
返回当前线程到达屏障的次序（ 0 ~ getParties() - 1）

③其他有用的方法
getNumberWaiting()：获取CyclicBarrier阻塞的线程数量
isBroken()：阻塞线程（一个或多个）是否被中断
reset()：重置CyclicBarrier
```

举个例子：

```
public class CyclicBarriarExapmle {

    private Map<String, Integer> map = new ConcurrentHashMap<>();
    
    CyclicBarrier barrier = new CyclicBarrier(10, ()->{
        //线程全部到达屏障后，执行的任务
        System.out.println("我是线程全部到达屏障后，执行的任务");
        int result = 0;
        for(Map.Entry<String, Integer> entry : map.entrySet()){
            result += entry.getValue();
        }
        System.out.println("最终计算结果：" + result);
    });

    private void calculate(){
        for(int i = 0; i < 10; i++){
            final int j = i;
            new Thread(()->{
                //执行计算，假如计算结果是，计算完成后，放入map中
                System.out.println("当前计算结果：" + j);
                map.put(Thread.currentThread().getName(), j);
                try {
                    barrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }

    public static void main(String[] args){
        CyclicBarriarExapmle exapmle = new CyclicBarriarExapmle();
        exapmle.calculate();
    }
}
```

程序最终输出：

```
当前计算结果：1
当前计算结果：4
当前计算结果：3
当前计算结果：2
当前计算结果：6
当前计算结果：0
当前计算结果：7
当前计算结果：5
当前计算结果：8
当前计算结果：9
我是线程全部到达屏障后，执行的任务
最终计算结果：45
```

**CountDownLatch与CyclicBarrier的区别**：二者都可以用来让一组线程等待其他线程，但CyclicBarrier功能更强大，可以重复使用，并可以设置优先任务。

### Semaphore
* 作用：控制同时访问特定资源的线程数量，进行流量控制
* 使用：①创建Semaphore，根据资源特性，指定可以同时访问该资源的线程数量；②在具体使用资源的时候，首先从Semaphore获取许可证，使用完资源之后，释放资源
* 值得注意的是：在一个线程release之前，并不一定要acquire。可以根据程序需要，自行控制。

```
/**
 * Releases a permit, returning it to the semaphore.
 *
 * <p>Releases a permit, increasing the number of available permits by
 * one.  If any threads are trying to acquire a permit, then one is
 * selected and given the permit that was just released.  That thread
 * is (re)enabled for thread scheduling purposes.
 *
 * <p>There is no requirement that a thread that releases a permit must
 * have acquired that permit by calling {@link #acquire}.
 * Correct usage of a semaphore is established by programming convention
 * in the application.
 */
public void release() {
    sync.releaseShared(1);
}
```

举个例子：
```
public class SemaphoreExample {
    private Semaphore semaphore = new Semaphore(10);
    private Executor executor = Executors.newFixedThreadPool(30);
    public void calculate(){
        for(int i = 0; i < 30; i++){
            executor.execute(()->{
                try {
                    //获取许可证
                    semaphore.acquire();
                    //执行计算
                    System.out.println("使用资源，执行任务");
                    //释放许可证
                    semaphore.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
    }
}
```

### Exchanger
* 作用：线程间数据交换，它提供一个同步点，两个线程可以交换彼此的数据，这两个线程通过exchange()方法交换数据，如果第一个线程先执行该方法，它会一直等待第二个线程也执行该方法，当两个线程都到达同步点的时候，这两个线程就可以交换数据。

```
public class ExchangerExample {

    public static void main(String[] args){
        Exchanger<String> exchanger = new Exchanger<>();
        new Thread(()->{
            String resultOne = "A";
            try {
                String exchangeResult = exchanger.exchange(resultOne);
                System.out.println("我的计算结果是：" + resultOne + "，与我交换数据的那个线程计算的结果是：" + exchangeResult);

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        new Thread(()->{
            String resultTwo = "B";
            try {
                String exchangeResult = exchanger.exchange(resultTwo);
                System.out.println("我的计算结果是：" + resultTwo + "，与我交换数据的那个线程计算的结果是：" + exchangeResult);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```


### 参考
《Java并发编程的艺术》，有适当更改



