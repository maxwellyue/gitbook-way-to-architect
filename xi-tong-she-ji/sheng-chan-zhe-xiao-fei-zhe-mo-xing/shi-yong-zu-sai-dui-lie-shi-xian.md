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
