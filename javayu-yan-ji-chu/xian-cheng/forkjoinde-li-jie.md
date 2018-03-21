# Fork/Join框架

关键词：任务分解、结果合并、工作窃取算法、实现compute()方法

---

### Fork/Join框架用来做什么？

Fork/Join框架是Java7提供了的一个用于并行执行任务的框架， 是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。

我们再通过Fork和Join这两个单词来理解下Fork/Join框架：

* Fork就是把一个大任务切分为若干子任务并行的执行，
* Join就是合并这些子任务的执行结果，最后得到这个大任务的结果。

比如计算1+2+。。＋10000，可以分割成10个子任务，每个子任务分别对1000个数进行求和，最终汇总这10个子任务的结果。

Fork/Join的运行流程图如下：


我们只需要关注如何划分任务和组合中间结果，将剩下的事情丢给 Fork/Join 框架。

### 工作窃取算法

工作窃取（work-stealing）算法是指某个线程从其他队列里窃取任务来执行。工作窃取的运行流程图如下：



那么为什么需要使用工作窃取算法呢？假如我们需要做一个比较大的任务，我们可以把这个任务分割为若干互不依赖的子任务，为了减少线程间的竞争，于是把这些子任务分别放到不同的队列里，并为每个队列创建一个单独的线程来执行队列里的任务，线程和队列一一对应，比如A线程负责处理A队列里的任务。但是有的线程会先把自己队列里的任务干完，而其他线程对应的队列里还有任务等待处理。干完活的线程与其等着，不如去帮其他线程干活，于是它就去其他线程的队列里窃取一个任务来执行。而在这时它们会访问同一个队列，所以为了减少窃取任务线程和被窃取任务线程之间的竞争，通常会使用**双端队列**，被窃取任务线程永远从双端队列的头部拿任务执行，而窃取任务的线程永远从双端队列的尾部拿任务执行。

* 优点<br>充分利用线程进行并行计算，并减少了线程间的竞争，

* 缺点<br>在某些情况下还是存在竞争，比如双端队列里只有一个任务时。并且消耗了更多的系统资源，比如创建多个线程和多个双端队列。

### Fork/Join框架的设计

自己设计一个Fork/Join框架，该如何设计？

* 第一步**分割任务**。<br>首先我们需要有一个fork类来把大任务分割成子任务，有可能子任务还是很大，所以还需要不停的分割，直到分割出的子任务足够小。

* 第二步**执行任务并合并结果**。<br>分割的子任务分别放在双端队列里，然后几个启动线程分别从双端队列里获取任务执行。子任务执行完的结果都统一放在一个队列里，启动一个线程从队列里拿数据，然后合并这些数据。

Fork/Join使用以下两个类来完成以上两件事情：

* ForkJoinTask<br>要使用ForkJoin框架，必须首先创建一个ForkJoin任务。它提供在任务中执行fork()和join()操作的机制，通常情况下我们不需要直接继承ForkJoinTask类，而只需要继承它的子类，Fork/Join框架提供了以下两个子类：
  * RecursiveAction：用于没有返回结果的任务。
  * RecursiveTask ：用于有返回结果的任务。
* ForkJoinPool <br>ForkJoinTask需要通过ForkJoinPool来执行，任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任务。


### 使用Fork/Join框架示例

现在，假如我们需要计算1+2+3+...+ 1000：

```
//具体的任务分解与计算类，继承RecursiveTask
public class CalculateTask extends RecursiveTask<Integer> {

    //阈值：即当子任务中要计算的数字的个数大于该值时，就要继续分解
    private static final int THRESHOLD = 10;
    //从哪个数开始计算
    private int start;
    //计算到哪个数为止
    private int end;

    public CalculateTask(int start, int end) {
        this.start = start;
        this.end = end;
    }

    /**
     * 本次计算的逻辑
     * @return
     */
    @Override
    protected Integer compute() {
        //本次计算的最终结果
        int sum = 0;
        //是否要继续分解任务
        boolean shouldFork = (end - start) > THRESHOLD;
        //不需要分解，则直接计算
        if (!shouldFork) {
            for (int i = start; i <= end; i++) {
                sum += i;
            }
        } else {//需要分解，则分解为两个子任务
            int middle = (start + end) / 2;
            CalculateTask leftTask = new CalculateTask(start, middle);
            CalculateTask rightTask = new CalculateTask(middle + 1, end);
            leftTask.fork();
            rightTask.fork();
            int leftRes = leftTask.join();
            int rightRes = rightTask.join();
            sum = leftRes + rightRes;
        }
        return sum;
    }
}
```
写完任务分解与具体任务计算的逻辑，就可以真正地计算了：

```
public class Client {

    public static void main(String[] args){
        //创建线程池，默认线程个数为CPU可使用线程数
        ForkJoinPool pool = new ForkJoinPool();
        //创建任务
        CalculateTask task = new CalculateTask(1, 3);
        //提交任务
        ForkJoinTask<Integer> res = pool.submit(task);
        //打印结果
        try {
            System.out.println("最终结算结果为：" + res.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```





---

内容来源：

[JDK 7 中的 Fork/Join 模式](https://www.ibm.com/developerworks/cn/java/j-lo-forkjoin/)

[聊聊并发（八）——Fork/Join框架介绍](http://www.infoq.com/cn/articles/fork-join-introduction)






