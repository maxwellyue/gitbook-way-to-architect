## 使用Lock和Condition实现

前提是要熟悉`Lock`接口以及常用实现类`ReentrantLock`，以及`Condition`的两个常用方法：

* `await()`:等待Condition的满足，会释放锁
* `signal()`:唤醒其他正在等待该`Condition`的线程 

**参考代码**

生产者

```java
class Producer extends Thread {

    private String threadName;
    private Queue<Goods> queue;
    private Lock lock;
    private Condition notFullCondition;
    private Condition notEmptyCondition;
    private int maxSize;

    public Producer(String threadName, Queue<Goods> queue, Lock lock, Condition notFullCondition, Condition notEmptyCondition, int maxSize) {
        this.threadName = threadName;
        this.queue = queue;
        this.lock = lock;
        this.notFullCondition = notFullCondition;
        this.notEmptyCondition = notEmptyCondition;
        this.maxSize = maxSize;

    }

    @Override
    public void run() {
        while (true) {
            //模拟生产过程中的耗时操作
            Goods goods = new Goods();
            try {
                Thread.sleep(new Random().nextInt(100));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            lock.lock();
            try {
                while (queue.size() == maxSize) {
                    try {
                        System.out.println("队列已满，【" + threadName + "】进入等待状态");
                        notFullCondition.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                queue.add(goods);
                System.out.println("【" + threadName + "】生产了一个商品：【" + goods.toString() + "】，目前商品数量：" + queue.size());
                notEmptyCondition.signalAll();

            } finally {
                lock.unlock();
            }
        }
    }
}
```

消费者

```java
class Consumer extends Thread {
    private String threadName;
    private Queue<Goods> queue;
    private Lock lock;
    private Condition notFullCondition;
    private Condition notEmptyCondition;

    public Consumer(String threadName, Queue<Goods> queue, Lock lock, Condition notFullCondition, Condition notEmptyCondition) {
        this.threadName = threadName;
        this.queue = queue;
        this.lock = lock;
        this.notFullCondition = notFullCondition;
        this.notEmptyCondition = notEmptyCondition;
    }

    @Override
    public void run() {
        while (true) {
            Goods goods;
            lock.lock();
            try {
                while (queue.isEmpty()) {
                    try {
                        System.out.println("队列已空，【" + threadName + "】进入等待状态");
                        notEmptyCondition.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                goods = queue.remove();
                System.out.println("【" + threadName + "】消费了一个商品：【" + goods.toString() + "】，目前商品数量：" + queue.size());
                notFullCondition.signalAll();

            } finally {
                lock.unlock();
            }

            //模拟消费过程中的耗时操作
            try {
                Thread.sleep(new Random().nextInt(100));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```




测试

```java
public class ProducerConsumer {
    @Test
    public void test() {

        int maxSize = 5;
        Queue<Goods> queue = new LinkedList<>();
        Lock lock = new ReentrantLock();
        Condition notEmptyCondition = lock.newCondition();
        Condition notFullCondition = lock.newCondition();

        Thread producer1 = new ProducerConsumer2.Producer("生产者1", queue, lock, notFullCondition, notEmptyCondition, maxSize);
        Thread producer2 = new ProducerConsumer2.Producer("生产者2", queue, lock, notFullCondition, notEmptyCondition, maxSize);
        Thread producer3 = new ProducerConsumer2.Producer("生产者3", queue, lock, notFullCondition, notEmptyCondition, maxSize);

        Thread consumer1 = new ProducerConsumer2.Consumer("消费者1", queue, lock, notFullCondition, notEmptyCondition);
        Thread consumer2 = new ProducerConsumer2.Consumer("消费者2", queue, lock, notFullCondition, notEmptyCondition);
        Thread consumer3 = new ProducerConsumer2.Consumer("消费者3", queue, lock, notFullCondition, notEmptyCondition);


        producer1.start();
        producer2.start();
        producer3.start();
        consumer1.start();
        consumer2.start();
        consumer3.start();
        while (true) {

        }
    }
}
```

要注意的地方
   
放入和取出操作均是用的同一个锁，所以在同一时刻，要么在放入，要么在取出，两者不能同时进行。因此，与使用wait\(\)和notify\(\)实现类似，这种方式的实现并不能最大限度地利用缓冲区（即例子中的队列）。如果要实现同一时刻，既可以放入又可以取出，则要使用两个重入锁，分别控制放入和取出的操作，具体实现可以参考`LinkedBlockingQueue`。



