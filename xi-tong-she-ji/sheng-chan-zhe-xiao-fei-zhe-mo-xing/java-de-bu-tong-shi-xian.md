# Java的不同实现

## 概述

生产者消费者模型可以描述为：  
①生产者持续生产，直到仓库放满产品，则停止生产进入等待状态；仓库不满后继续生产；  
②消费者持续消费，直到仓库空，则停止消费进入等待状态；仓库不空后，继续消费；  
③生产者可以有多个，消费者也可以有多个；

![生产者消费者模型](../../.gitbook/assets/image %2814%29.png)

对应到程序中，仓库对应缓冲区，可以使用队列来作为缓冲区，并且这个队列应该是有界的，即最大容量是固定的；进入等待状态，则表示要阻塞当前线程，直到某一条件满足，再进行唤醒。

常见的实现方式主要有以下几种：

* 使用`wait()`和`notify()`
* 使用`Lock`和`Condition`
* 使用信号量`Semaphore`
* 使用`JDK`自带的阻塞队列
* 
## 使用wait\(\)和notify\(\)实现

前提是要熟悉`Object`的几个方法：

* `wait()`:当前线程释放锁，直到等到通知，再去获取锁
* `sleep()`:当前线程休眠，但不释放锁
* `notify()`:唤醒其他正在wait的线程

参考代码如下：

```java
public class ProducerConsumer1 {

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

> 几个注意的地方：  
>   
> ①确定锁的对象是队列`queue`；  
>   
> ②不要把生产过程和消费过程写在同步块中，这些操作无需同步，同步的仅仅是放入和取出这两个动作；  
>   
> ③因为是持续生产，持续消费，要用`while(true){...}`的方式将【生产、放入】或【取出、消费】的操作都一直进行。  
>   
> ④但由于是对队列使用`synchronized`的方式加锁，同一时刻，要么在放入，要么在取出，两者不能同时进行。

## 使用Lock和Condition实现

前提是要熟悉`Lock`接口以及常用实现类`ReentrantLock`，以及`Condition`的两个常用方法：

* `await()`:等待Condition的满足，会释放锁
* `signal()`:唤醒其他正在等待该`Condition`的线程 参考代码如下：

```java
public class ProducerConsumer2 {

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

> 要注意的地方：  
>   
> 放入和取出操作均是用的同一个锁，所以在同一时刻，要么在放入，要么在取出，两者不能同时进行。因此，与使用wait\(\)和notify\(\)实现类似，这种方式的实现并不能最大限度地利用缓冲区（即例子中的队列）。如果要实现同一时刻，既可以放入又可以取出，则要使用两个重入锁，分别控制放入和取出的操作，具体实现可以参考`LinkedBlockingQueue`。

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

## 使用jdk自带的阻塞队列实现

前提是要记住两个阻塞取放方法，因为阻塞队列提供了很多存取元素的方法，几种存取方式在队列已满/已空时采取的措施如下：

| 方法/方式处理 | 抛出异常 | 返回特殊值 | 一直阻塞 | 超时退出 |
| :--- | :--- | :--- | :--- | :--- |
| 插入 | add\(e\) | offer\(e\) | put\(e\) | offer\(e, time, unit\) |
| 移除 | remove\(\) | poll\(\) | take\(\) | poll\(time, unit\) |
| 检查 | element\(\) | peek\(\) | 不可用 | 不可用 |

所以，在这里，要选用`put()`和`take()`这两个会阻塞的方法。

参考代码如下：

```java
public class ProducerConsumer3 {

    class Producer extends Thread {
        private String threadName;
        private BlockingQueue<Goods> queue;

        public Producer(String threadName, BlockingQueue<Goods> queue) {
            this.threadName = threadName;
            this.queue = queue;
        }

        @Override
        public void run() {
            while (true){
                Goods goods = new Goods();
                try {
                    //模拟生产过程中的耗时操作
                    Thread.sleep(new Random().nextInt(100));
                    queue.put(goods);
                    System.out.println("【" + threadName + "】生产了一个商品：【" + goods.toString() + "】，目前商品数量：" + queue.size());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    class Consumer extends Thread {
        private String threadName;
        private BlockingQueue<Goods> queue;

        public Consumer(String threadName, BlockingQueue<Goods> queue) {
            this.threadName = threadName;
            this.queue = queue;
        }

        @Override
        public void run() {
            while (true){
                try {
                    Goods goods = queue.take();
                    System.out.println("【" + threadName + "】消费了一个商品：【" + goods.toString() + "】，目前商品数量：" + queue.size());
                    //模拟消费过程中的耗时操作
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
        BlockingQueue<Goods> queue = new LinkedBlockingQueue<>(maxSize);

        Thread producer1 = new ProducerConsumer3.Producer("生产者1", queue);
        Thread producer2 = new ProducerConsumer3.Producer("生产者2", queue);
        Thread producer3 = new ProducerConsumer3.Producer("生产者3", queue);

        Thread consumer1 = new ProducerConsumer3.Consumer("消费者1", queue);
        Thread consumer2 = new ProducerConsumer3.Consumer("消费者2", queue);


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

> 要注意的地方：  
>   
> 如果使用LinkedBlockingQueue作为队列实现，则可以实现：在同一时刻，既可以放入又可以取出，因为LinkedBlockingQueue内部使用了两个重入锁，分别控制取出和放入。  
> 如果使用ArrayBlockingQueue作为队列实现，则在同一时刻只能放入或取出，因为ArrayBlockingQueue内部只使用了一个重入锁来控制并发修改操作。

## 使用管道流实现

//TODO

## 无锁的缓存框架: Disruptor

BlockingQueue 实现生产者和消费者模式简单易懂，但是`BlockingQueue`并不是一个高性能的实现：它完全使用锁和阻塞来实现线程之间的同步。在高并发的场合，它的性能并不是特别的优越。（`ConconcurrentLinkedQueue`是一个高性能的队列，但并不没有实现`BlockingQueue`接口，即不支持阻塞操作）。

Disruptor是LMAX公司开发的高效的无锁缓存队列。它使用无锁的方式实现了一个环形队列，非常适合于实现生产者和消费者模式，如：事件和消息的发布。

//TODO 应用场景的代码实现

## 参考

[Java 实现生产者 – 消费者模型](https://link.jianshu.com/?t=http%3A%2F%2Fwww.importnew.com%2F27063.html)：各种实现方式的性能  
[高性能的生产者-消费者：无锁的实现](https://link.jianshu.com/?t=http%3A%2F%2Ffrobisher.me%2F2017%2F05%2F26%2Fjava-producer-consumer-disruptor%2F)：无锁实现  
[Java生产者和消费者模型的5种实现方式](https://link.jianshu.com/?t=http%3A%2F%2Fcdn2.jianshu.io%2Fp%2F66e8b5ab27f6)  
[生产者/消费者问题的多种Java实现方式](https://link.jianshu.com/?t=http%3A%2F%2Fblog.csdn.net%2Fmonkey_d_meng%2Farticle%2Fdetails%2F6251879)  
[Java阻塞队列ArrayBlockingQueue和LinkedBlockingQueue实现原理分析](https://link.jianshu.com/?t=https%3A%2F%2Ffangjian0423.github.io%2F2016%2F05%2F10%2Fjava-arrayblockingqueue-linkedblockingqueue-analysis%2F)：两种常用阻塞队列的区别

[完整代码示例在此](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fmaxwellyue%2FJavaLanguage%2Ftree%2Fmaster%2Fsrc%2Fmain%2Fjava%2Fproducerandconsumer)

