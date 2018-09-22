## 使用wait\(\)和notify\(\)实现

前提是要熟悉`Object`的几个方法：

* `wait()`:当前线程释放锁，直到等到通知，再去获取锁
* `sleep()`:当前线程休眠，但不释放锁
* `notify()`:唤醒其他正在wait的线程

**参考代码**

生产者

```java
class Producer extends Thread {
    private String threadName;
    private Queue<Goods> queue;
    private int maxSize;

    public Producer(String threadName, Queue<Goods> queue, int maxSize) {
        this.threadName = threadName;
        this.queue = queue;
        this.maxSize = maxSize;
    }

    @Override
    public void run() {
        while (true) {
            //模拟生产过程中的耗时操作
            Goods goods = new Goods();
            try {
                Thread.sleep(new Random().nextInt(1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            synchronized (queue) {
                while (queue.size() == maxSize) {
                    try {
                        System.out.println("队列已满，【" + threadName + "】进入等待状态");
                        queue.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                queue.add(goods);
                System.out.println("【" + threadName + "】生产了一个商品：【" + goods.toString() + "】，目前商品数量：" + queue.size());
                queue.notifyAll();
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

    public Consumer(String threadName, Queue<Goods> queue) {
        this.threadName = threadName;
        this.queue = queue;
    }

    @Override
    public void run() {
        while (true) {
            Goods goods;
            synchronized (queue) {
                while (queue.isEmpty()) {
                    try {
                        System.out.println("队列已空，【" + threadName + "】进入等待状态");
                        queue.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                goods = queue.remove();
                System.out.println("【" + threadName + "】消费了一个商品：【" + goods.toString() + "】，目前商品数量：" + queue.size());
                queue.notifyAll();
            }
            //模拟消费过程中的耗时操作
            try {
                Thread.sleep(new Random().nextInt(1000));
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

        Thread producer1 = new Producer("生产者1", queue, maxSize);
        Thread producer2 = new Producer("生产者2", queue, maxSize);
        Thread producer3 = new Producer("生产者3", queue, maxSize);

        Thread consumer1 = new Consumer("消费者1", queue);
        Thread consumer2 = new Consumer("消费者2", queue);


        producer1.start();
        producer2.start();
        producer3.start();
        consumer1.start();
        consumer2.start();
        while (true) {
        }
    }
}
```

**几个注意的地方**  
  
①确定锁的对象是队列`queue`；  
  
②不要把生产过程和消费过程写在同步块中，这些操作无需同步，同步的仅仅是放入和取出这两个动作；  
  
③因为是持续生产，持续消费，要用`while(true){...}`的方式将【生产、放入】或【取出、消费】的操作都一直进行。  
  
④但由于是对队列使用`synchronized`的方式加锁，同一时刻，要么在放入，要么在取出，两者不能同时进行。



