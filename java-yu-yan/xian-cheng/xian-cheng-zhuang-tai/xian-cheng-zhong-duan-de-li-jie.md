# 线程中断的理解

## 中断的原理

Java中断机制是一种协作机制，也就是说通过中断并不能直接终止另一个线程，而需要被中断的线程自己处理中断。

Java中断模型也是这么简单，每个线程对象里都有一个boolean类型的标识（不一定就要是Thread类的字段，实际上也的确不是，这几个方法最终都是通过native方法来完成的），代表着是否有中断请求（该请求可以来自所有线程，包括被中断的线程本身）。

例如，当线程t1想中断线程t2，只需要在线程t1中将线程t2对象的中断标识置为true\(通过调用`t2.interrupt()`\)，然后线程2可以选择在合适的时候处理该中断请求，甚至可以不理会该请求，就像这个线程没有被中断一样。

Thread类提供了以下方法来操作这个中断状态：

| public static boolean **interrupted\(\)** | 测试当前线程是否已经中断。线程的中断状态由该方法清除。换句话说，如果连续两次调用该方法，则第二次调用将返回 false（在第一次调用已清除了其中断状态之后，且第二次调用检验完中断状态前，当前线程再次中断的情况除外）。 |
| :--- | :--- |
| public boolean **isInterrupted**\(\) | 测试线程是否已经中断。线程的中断状态不受该方法的影响。 |
| public void **interrupt**\(\) | 中断线程 |

其中，`interrupt()`方法是唯一能将中断状态设置为true的方法。静态方法interrupted会将当前线程的中断状态清除，但这个方法的命名极不直观，很容易造成误解，需要特别注意。

此外，类库中的有些类的方法也可能会调用中断，如FutureTask中的cancel方法，如果传入的参数为true，它将会在正在运行异步任务的线程上调用interrupt方法，如果正在执行的异步任务中的代码没有对中断做出响应，那么cancel方法中的参数将不会起到什么效果；又如ThreadPoolExecutor中的shutdownNow方法会遍历线程池中的工作线程并调用线程的interrupt方法来中断线程，所以如果工作线程中正在执行的任务没有对中断做出响应，任务将一直执行直到正常结束。

## 中断的处理

既然Java中断机制只是设置被中断线程的中断状态，那么被中断线程该做些什么？

**处理时机**

显然，作为一种协作机制，不会强求被中断线程一定要在某个点进行处理。实际上，被中断线程只需在合适的时候处理即可，如果没有合适的时间点，甚至可以不处理，这时候在任务处理层面，就跟没有调用中断方法一样。“合适的时候”与线程正在处理的业务逻辑紧密相关，例如，每次迭代的时候，进入一个可能阻塞且无法中断的方法之前等，但多半不会出现在某个临界区更新另一个对象状态的时候，因为这可能会导致对象处于不一致状态。

处理时机决定着程序的效率与中断响应的灵敏性。频繁的检查中断状态可能会使程序执行效率下降，相反，检查的较少可能使中断请求得不到及时响应。如果发出中断请求之后，被中断的线程继续执行一段时间不会给系统带来灾难，那么就可以将中断处理放到方便检查中断，同时又能从一定程度上保证响应灵敏度的地方。当程序的性能指标比较关键时，可能需要建立一个测试模型来分析最佳的中断检测点，以平衡性能和响应灵敏性。

**处理方式**

1、 中断状态的管理

一般说来，当可能阻塞的方法声明中有抛出InterruptedException则暗示该方法是可中断的，如BlockingQueue\#put、BlockingQueue\#take、Object\#wait、Thread\#sleep等，如果程序捕获到这些可中断的阻塞方法抛出的InterruptedException或检测到中断后，这些中断信息该如何处理？一般有以下两个通用原则：

* 如果遇到的是可中断的阻塞方法抛出InterruptedException，可以继续向方法调用栈的上层抛出该异常，如果是检测到中断，则可清除中断状态并抛出InterruptedException，使当前方法也成为一个可中断的方法。
* 若有时候不太方便在方法上抛出InterruptedException，比如要实现的某个接口中的方法签名上没有throws InterruptedException，这时就可以捕获可中断方法的InterruptedException并通过Thread.currentThread.interrupt\(\)来重新设置中断状态。如果是检测并清除了中断状态，亦是如此。

一般的代码中，尤其是作为一个基础类库时，绝不应当吞掉中断，即捕获到InterruptedException后在catch里什么也不做，清除中断状态后又不重设中断状态也不抛出InterruptedException等。因为吞掉中断状态会导致方法调用栈的上层得不到这些信息。

当然，凡事总有例外的时候，当你完全清楚自己的方法会被谁调用，而调用者也不会因为中断被吞掉了而遇到麻烦，就可以这么做。

总得来说，就是要**让方法调用栈的上层获知中断的发生**。假设你写了一个类库，类库里有个方法amethod，在amethod中检测并清除了中断状态，而没有抛出InterruptedException，作为amethod的用户来说，他并不知道里面的细节，如果用户在调用amethod后也要使用中断来做些事情，那么在调用amethod之后他将永远也检测不到中断了，因为中断信息已经被amethod清除掉了。如果作为用户，遇到这样有问题的类库，又不能修改代码，那该怎么处理？只好在自己的类里设置一个自己的中断状态，在调用interrupt方法的时候，同时设置该状态，这实在是无路可走时才使用的方法。

2、 中断的响应

程序里发现中断后该怎么响应？这就得视实际情况而定了。有些程序可能一检测到中断就立马将线程终止，有些可能是退出当前执行的任务，继续执行下一个任务。作为一种协作机制，这要与中断方协商好，当调用interrupt会发生些什么都是事先知道的，如做一些事务回滚操作，一些清理工作，一些补偿操作等。若不确定调用某个线程的interrupt后该线程会做出什么样的响应，那就不应当中断该线程。

### 使用线程中断机制结束线程或任务

离开线程有两种常用的方法：

* 抛出InterruptedException
* 用Thread.interrupted\(\)检查是否发生中断

**抛出InterruptedException异常的方式实现结束线程**

```java
public static void main(String[] args) {

    Thread thread = new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                while (true) {
                    System.out.println("i am running...");
                    //sleep方法在收到线程收到中断事件时，会抛出InterruptedException异常
                    Thread.sleep(10);
                }
            } catch (InterruptedException e) {
                System.out.println("interrupted...");
            }
        }
    });

    thread.start();
    try {
        Thread.sleep(100);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    //中断线程
    thread.interrupt();
}
//输出如下：
i am running...
i am running...
i am running...
i am running...
i am running...
i am running...
i am running...
i am running...
i am running...
i am running...
interrupted...
```

**检查中断状态的方式实现结束线程**

```java
public static void main(String[] args) {

    Thread thread = new Thread(new Runnable() {
        @Override
        public void run() {
            double d = 0.0;
            while (!Thread.interrupted()) {
                for (int i = 0; i < 900000; i++) {
                    d = d + (Math.PI + Math.E) / d;
                }
                System.out.println("i am running...");
            }
            System.out.println("interrupted...");
        }
    });

    thread.start();
    try {
        Thread.sleep(10);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    //中断线程
    thread.interrupt();
}
//输出如下：
i am running...
i am running...
interrupted...
```

### 内容来源

[详细分析Java中断机制](http://www.infoq.com/cn/articles/java-interrupt-mechanism)

[Java中的线程Thread方法之---interrupt\(\)](https://blog.csdn.net/jiangwei0910410003/article/details/19962603)

[用interrupt\(\)中断Java线程](http://hapinwater.iteye.com/blog/310558)[ ](https://blog.csdn.net/jiangwei0910410003/article/details/19962603)

[Java里一个线程调用了Thread.interrupt\(\)到底意味着什么？    
](https://www.zhihu.com/question/41048032)

