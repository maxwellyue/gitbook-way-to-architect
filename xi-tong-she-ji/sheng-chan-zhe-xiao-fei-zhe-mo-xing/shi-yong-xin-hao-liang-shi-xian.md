## 使用信号量Semaphore实现

前提是熟悉信号量`Semaphore`的使用方式，尤其是`release()`方法，`Semaphore`在`release`之前不必一定要先`acquire`\(如果不熟悉`Semaphore`，可以参考阅读[【多线程与并发】Java并发工具类](https://www.jianshu.com/p/738d1ddd6731)\)。

> There is no requirement that a thread that releases a permit must have acquired that permit by calling acquire.Correct usage of a semaphore is established by programming convention in the application.

参考代码如下：

```java
public class ProducerConsumer4 {


    class Producer extends Thread {
        private String threadName;
        private Queue<Goods> queue;
        private Semaphore queueSizeSemaphore;
        private Semaphore concurrentWriteSemaphore;
        private Semaphore notEmptySemaphore;

        public Producer(String threadName, Queue<Goods> queue, Semaphore concurrentWriteSemaphore, Semaphore queueSizeSemaphore, Semaphore notEmptySemaphore) {
            this.threadName = threadName;
            this.queue = queue;
            this.concurrentWriteSemaphore = concurrentWriteSemaphore;
            this.queueSizeSemaphore = queueSizeSemaphore;
            this.notEmptySemaphore = notEmptySemaphore;
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

                try {
                    queueSizeSemaphore.acquire();//获取队列未满的信号量
                    concurrentWriteSemaphore.acquire();//获取读写的信号量
                    queue.add(goods);
                    System.out.println("【" + threadName + "】生产了一个商品：【" + goods.toString() + "】，目前商品数量：" + queue.size());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    concurrentWriteSemaphore.release();
                    notEmptySemaphore.release();
                }
            }
        }
    }

    class Consumer extends Thread {
        private String threadName;
        private Queue<Goods> queue;
        private Semaphore queueSizeSemaphore;
        private Semaphore concurrentWriteSemaphore;
        private Semaphore notEmptySemaphore;

        public Consumer(String threadName, Queue<Goods> queue, Semaphore concurrentWriteSemaphore, Semaphore queueSizeSemaphore, Semaphore notEmptySemaphore) {
            this.threadName = threadName;
            this.queue = queue;
            this.concurrentWriteSemaphore = concurrentWriteSemaphore;
            this.queueSizeSemaphore = queueSizeSemaphore;
            this.notEmptySemaphore = notEmptySemaphore;
        }

        @Override
        public void run() {
            while (true) {
                Goods goods;
                try {
                    notEmptySemaphore.acquire();
                    concurrentWriteSemaphore.acquire();
                    goods = queue.remove();
                    System.out.println("【" + threadName + "】生产了一个商品：【" + goods.toString() + "】，目前商品数量：" + queue.size());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    concurrentWriteSemaphore.release();
                    queueSizeSemaphore.release();
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


    @Test
    public void test() {

        int maxSize = 5;
        Queue<Goods> queue = new LinkedList<>();
        Semaphore concurrentWriteSemaphore = new Semaphore(1);
        Semaphore notEmptySemaphore = new Semaphore(0);
        Semaphore queueSizeSemaphore = new Semaphore(maxSize);


        Thread producer1 = new ProducerConsumer4.Producer("生产者1", queue, concurrentWriteSemaphore, queueSizeSemaphore, notEmptySemaphore);
        Thread producer2 = new ProducerConsumer4.Producer("生产者2", queue, concurrentWriteSemaphore, queueSizeSemaphore, notEmptySemaphore);
        Thread producer3 = new ProducerConsumer4.Producer("生产者3", queue, concurrentWriteSemaphore, queueSizeSemaphore, notEmptySemaphore);

        Thread consumer1 = new ProducerConsumer4.Consumer("消费者1", queue, concurrentWriteSemaphore, queueSizeSemaphore, notEmptySemaphore);
        Thread consumer2 = new ProducerConsumer4.Consumer("消费者2", queue, concurrentWriteSemaphore, queueSizeSemaphore, notEmptySemaphore);
        Thread consumer3 = new ProducerConsumer4.Consumer("消费者3", queue, concurrentWriteSemaphore, queueSizeSemaphore, notEmptySemaphore);


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

> 要注意的地方：  
>   
> ①理解代码中的三个信号量的含义  
> **queueSizeSemaphore**：（其中的许可证数量，可以理解为队列中可以再放入多少个元素），该信号量的许可证初始数量为仓库大小，即`maxSize`；生产者每放置一个商品，则该信号量-1，即执行`acquire()`，表示队列中已经添加了一个元素，要减少一个许可证；消费者每取出一个商品，该信号量+1，即执行`release()`，表示队列中已经少了一个元素，再给你一个许可证。  
> **notEmptySemaphore**：（其中的许可证数量，可以理解为队列中可以取出多少个元素），该信号量的许可证初始数量为0；生产者每放置一个商品，则该信号量+1，即执行`release()`，表示队列中添加了一个元素；消费者每取出一个商品，该信号量-1，即执行`acquire()`，表示队列中已经少了一个元素，要减少一个许可证；  
> **concurrentWriteSemaphore**，相当于一个写锁，在放入或取出商品的时候，都需要先获取再释放许可证。  
>   
> ②由于实现中，使用了`concurrentWriteSemaphore`实现了对队列并发写的控制，在同一时刻，只能对队列进行一种操作：放入或取出。假如把`concurrentWriteSemaphore`中的信号量初始化为2或者2以上的值，就会出现多个生产者同时放入或多个消费者同时消费的情况，而使用的`LinkedList`是不允许并发进行这种修改的，否则会出现溢出或取空的情况。所以，`concurrentWriteSemaphore`只能设置为1，也就导致性能与使用`wait() / notify()`方式类似，性能不高。
