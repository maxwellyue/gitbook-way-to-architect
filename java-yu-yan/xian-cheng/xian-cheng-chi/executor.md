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

#### **ThreadPoolExecutor**

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

（3）**CachedThreadPool**：根据需要创建新的线程

```java
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
}
```

①使用无容量队列SynchronousQueue，但maxmumPoolSize无界。如果提交任务的速度大于线程处理任务的速度，将会不断创建新线程，极端情况会因为创建过多线程而耗尽CPU资源。  
②keepAliveTime为60s，空闲线程超过该时间将会终止。  
③执行完任务的某线程会执行SynchronousQueue.poll\(\)从队列中取任务，这个取的动作会持续60s，如果在60s内有新的任务，则执行新的任务，没有任务则终止线程。因此长时间保持空闲的CachedThreadPool不会占用任何资源。  
④当有任务提交时，a.如果当前线程池为空或者已创建的线程都正在处理任务，则CachedThreadPool会创建新线程来执行该任务。b.如果当前线程池有空闲的线程（正在执行阻塞方法SynchronousQueue.poll\(\)），则将任务交给该等待任务的空闲线程来执行。**CachedThreadPool适用于执行很多的短期异步任务的小程序或者是负载较轻的服务器。**

#### **ScheduledThreadPoolEecutor**

用来执行定期任务或者在给定延迟时间之后执行任务。TODO

### 获取异步结果

主要是通过接口Future和实现类FutureTask。

**Future**

Future代表了一个异步计算的结果。

```java
public interface Future<V> {
//取消当前任务，如果任务已经完成，就会取消失败，返回false;
//如果取消成功，并且在调用该方法之前对应的任务还没有开始，
//则该任务永远也不会执行。如果任务正在执行，
//参数mayInterruptIfRunning设为true则表示将正在执行该任务的线程终止，
//参数mayInterruptIfRunning设为false则表示会等待任务完成。
//该方法返回true或false之后，之后的isDone()方法会返回true；
//如果该方法返回true，之后的isCancelled()方法也会返回true，
boolean cancel(boolean mayInterruptIfRunning);

//对应的任务是否在完成之前就取消了
boolean isCancelled();

//任务是否完成
boolean isDone();

//获取计算结果，如果任务还没执行完成，则会阻塞当前线程（调用该方法所在的线程），直到任务完成
V get() throws InterruptedException, ExecutionException;

//最多等待timeout就尝试取回结果
V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

JDK文档上给出的示例

```java
interface ArchiveSearcher { String search(String target); }
class App {
       ExecutorService executor = ...
       ArchiveSearcher searcher = ...
       public void showSearch(final String target)throws InterruptedException {
             Future<String> future= executor.submit(new Callable<String>() {
                  public String call() {
                      return searcher.search(target);
                  }
              });
             displayOtherThings(); // do other things while searching
             try {
                  displayText(future.get()); // use future
             } catch (ExecutionException ex) { cleanup(); return; }
       }
}
```

**FutureTask**

FutureTask就像它的名字一样，既有Future的特点（实现Future接口），又具有任务的特点（实现Runnable接口）。更直白的理解是，FutureTask就是一种特殊的任务的描述类，利用FutureTask创建的任务可以获取计算结果。

FutureTask表示一个可取消的异步计算，并通过实现Future接口来开始或取消一个计算、查看计算是否完成、获取计算结果。如果计算还没有完成，调用FutureTask的get方法会阻塞当前线程（调用get方法所在的线程）。

FutureTask可以用来包装Callable和Runnable对象，因为实现了Runnable接口，所以FutureTask可以提交给Executor来执行（不提交就调用自己的run方法，也可以执行计算）。

```java
FutureTask<String> future = new FutureTask<String>(new Callable<String>() {
    public String call() {
        return searcher.search(target);
    }
});
executor.execute(future);
```

作为一个独立的类，该类提供了很多protected的方法，以便创建你自己的定制任务类。

### 内容来源

大多数摘抄自《Java并发编程的艺术》







