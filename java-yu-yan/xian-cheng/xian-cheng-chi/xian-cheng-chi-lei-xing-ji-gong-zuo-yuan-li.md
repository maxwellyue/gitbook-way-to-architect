# 线程池工作原理及创建

## 为什么需要线程池

* **降低资源消耗**：对象的创建和销毁是非常耗时的操作（线程也是一个对象）。通过重复利用已创建的线程降低线程创建和销毁造成的消耗；
* **提高响应速度**：当任务到达时，任务可以不需要等待线程创建就能立即执行；
* **提高线程的可管理性**：线程是稀缺资源，如果无限制地创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一分配、调优和监控。

## 工作原理

线程池的工作流程如下所示。

### ![](https://upload-images.jianshu.io/upload_images/1932449-7b4da1c878b378cb.png?imageMogr2/auto-orient/strip|imageView2/2/w/604/format/webp)

## 线程池的创建

```java
//ThreadPoolExecutor的构造器
public ThreadPoolExecutor(int corePoolSize,                                
                          int maximumPoolSize,                             
                          long keepAliveTime,                              
                          TimeUnit unit,                                   
                          BlockingQueue<Runnable> workQueue,               
                          ThreadFactory threadFactory,                     
                          RejectedExecutionHandler handler) {              
    if (corePoolSize < 0 ||                                                
        maximumPoolSize <= 0 ||                                            
        maximumPoolSize < corePoolSize ||                                  
        keepAliveTime < 0)                                                 
        throw new IllegalArgumentException();                              
    if (workQueue == null || threadFactory == null || handler == null)     
        throw new NullPointerException();                                  
    this.corePoolSize = corePoolSize;                                      
    this.maximumPoolSize = maximumPoolSize;                                
    this.workQueue = workQueue;                                            
    this.keepAliveTime = unit.toNanos(keepAliveTime);                      
    this.threadFactory = threadFactory;                                    
    this.handler = handler;                                                
}
```

参数解释

* **corePoolSize**（线程池的基本大小）：当提交一个任务时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，直到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreTreads\(\)方法，线程池就会提前创建并启动所有基本线程。
* **maximumPoolSize**（线程最大数量）：线程池允许创建的最大线程数。如果队列已满，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。但如果使用了无解的任务队列，该参数没有效果。
* **keepAliveTime**（线程活动保持时间）：线程池的工作线程空闲后，保持存活的时间。如果任务很多，且每个任务执行时间较短，可调大该值。
* **TimeUnit**（线程活动保持时间的单位）：keepAliveTime的时间度量单位。可选天、小时、分钟、毫秒、微妙、纳秒。
* **BlockingQueue**&lt;Runnable&gt;（任务队列）：用于保存等待执行的任务的阻塞嘟列，可以选择以下几个阻塞队列
  * ArrayBlockingQueue：基于数组结构的有界阻塞队列
  * LinkedBlockingQueue：基于链表机构的阻塞队列，吞吐量通常高于ArrayBlockingQueue
  * SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除，否则插入操作一直处于阻塞状态，吞吐量通常高于LinkedBlockingQueue
  * PriorityBlockingQueue：具有优先级的无限阻塞队列
* **ThreadFactory**：创建线程的工厂。
* **RejectedExecutionHandler**：饱和策略，即队列和线程池都满了，对于新提交的任务无法执行，这时采取的处理新来的任务的方法，有4种策略可选（也可以**自定义策略：实现RejectedExecutionHandler接口，如记录日志或持久化不能处理的任务**）
  * AbortPolicy：直接抛出`RejectedExecutionException`异常。（默认策略）
  * DiscardPolicy：对新任务直接丢弃，不做任何事情
  * DiscardOldestPolicy：丢掉队列里最近（`the oldest unhandled`）的一个任务，并执行当前新任务。
  * CallerRunsPolicy：使用调用者所在的线程来运行任务。

## 提交任务

有两种方式将任务提交给线程池来执行

**execute\(\)**

用于提交不需要返回值的任务，所以无法判断任务是否被线程执行成功。

**submit\(\)**  
提交需要返回值的任务。线程池会返回一个Future类型的对象，通过这个对象可以判断任务是否执行成功，并且可以通过Future对象的get\(\)方法来获取返回值。但get\(\)方法会阻塞当前线程直到任务完成，使用get\(long timeout, TimeUnit unit\)方法会阻塞当前线程一段时间后立即返回（此时任务可能还没有执行完）。

## **关闭线程池**

调用线程池的两个方法来关闭shutdown\(\)或者shutdownNow\(\)：遍历线程池中的工作线程，然后逐个调用线程的interupt\(\)方法中断线程，所以无法响应中断的任务可能永远无法终止。

* shutdownNow\(\) 不允许添加新的任务。立刻关闭线程池。不管池中是否还存在正在运行的任务。关闭顺序是先尝试关闭当前正在运行的任务。然后返回待完成任务的清单。已经运行的任务则不返回。（首先将线程池的状态设置为STOP，然后尝试终止所有的线程（包括正在执行任务或暂停任务的），并返回等待执行任务的列表；）
* shutdown\(\) 不允许添加新的任务，等池中所有的任务执行完毕之后再关闭线程池。 （只是将线程池的状态设置为SHUTDOWN，然后中断所有没有正在执行任务的线程。）

_//todo 有待验证shutdown\(\)和shutdownNow\(\)的区别_





























