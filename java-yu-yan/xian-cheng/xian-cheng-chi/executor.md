### 线程池框架Executor

### 概述

Executor是（since）JDK1.5实现的线程池技术，先看Executor框架的主要类与接口



Executor主要可以分为3个部分：

* **任务对象的创建**
  ：实现`Runnable`接口或实现`Callable`接口
* **任务的执行**
  ：接口ExecutorService、两个实现类`ThreadPoolExecutor`和`ScheduledThreadPoolExecutor`
* **异步计算的结果**
  ：接口Future以及实现类FutureTask

### 任务对象

通过两种方式创建任务对象：实现`Runnable`接口或实现`Callable`接口。

* Runnable不会返回结果，Callable可以返回结果；
* Callable的call方法可抛出异常，而Runnable的run方法不能抛出异常

可以通过Executors工具类提供的两个方法将Runnable包装为Callable：

```java
public static <T> Callable<T> callable(Runnable task, T result);
public static Callable<Object> callable(Runnable task);
```

### 任务执行

任务的执行是由两个实现类完成的：`ThreadPoolExecutor`和`ScheduledThreadPoolExecutor`。

前面介绍线程池工作过程就是以`ThreadPoolExecutor`为例进行的。在实际使用中，通常使用工具类`Executors`创建不同类型的

`ThreadPoolExecutor`。而`ScheduledThreadPoolExecutor`是`ThreadPoolExecutor`类的子类，相当于特定功能的扩展：在给定的延迟之后运行任务或者定期执行任务。它与Timer的功能类似，但更强大、更灵活。Timer对应的是单个后台线程，而

`ScheduledThreadPoolExecutor`可以指定多个线程数。

**ThreadPoolExecutor**

工具类`Executors`可以创建3中类型的`ThreadPoolExecutor`，分别如下。

（1）**FixedThreadPool：**可重用、固定线程数的线程池

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
}
```

可以看出，①FixedThreadPool的corePoolSize和maxmumPoolSize都被设置为nThreads；②keepAliveTime设为0，表示某工作线程一旦空闲，就立即关闭该工作线程。③使用无界队列LinkedBlockingQueue，当线程池中的线程数达到corePoolSize后，新任务将会在无界队列中等待，因此线程数永远不会超过corePoolSize。**FixedThreadPool适用于为了满足资源管理的需求，而需要限制当前线程数量的场景，比如负载较重的服务器。**

（2）**SingleThreadExecutor**

```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
}
```

参数与FixedThreadPool的区别仅在于corePoolSize和maxmumPoolSize均为1，keepAliveTime和使用的阻塞队列都一样，特性类似，可以概括为：当有新任务时，如果线程池中没有线程，则创建一个线程，之后来的任务都存储在无界队列LinkedBlockingQueue中。该线程一直从队列中取任务执行。假如任务都执行完毕，立即终止该线程。**SingleThreadExecutor适用于需要保证顺序地执行各个任务，并且在任意时间点，不会有多个线程是活动的场景。**



















  








