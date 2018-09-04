# 几种方法的解释

## **Thread类中的方法**

**1、yield**

```java
public static native void yield();
```

该方法向线程调度器提示，当前线程（调用该方法的线程对象代表的线程）愿意让出处理器的使用权，但是线程调度器可能会忽视该提示。很少有适合使用该方法的地方。它可能在调试或者测试的时候，帮助重现由于竞争条件出现的bug。它还可能在设计并发控制结构的时候提供帮助，比如java.util.concurrent.locks包中的一些类。

**2、sleep**

```java
public static native void sleep(long millis) throws InterruptedException;
public static void sleep(long millis, int nanos) throws InterruptedException;
```

该方法会让当前线程（调用该方法的线程对象代表的线程）睡眠指定的一段时间（暂时停止执行），但当前线程不会失去锁的拥有权（还会继续占用锁）。上述两个方法只是指定的睡眠时间精度不同。

**3、start**

```java
public synchronized void start()
```

该方法会让当前线程（调用该方法的线程对象代表的线程）开始执行。JVM虚拟机会调用该线程的run\(\)方法。对一个线程只能start\(\)一次。

> 对同一个线程多次调用start\(\)会抛出异常：java.lang.IllegalThreadStateException，即当前状态（start）不能调用start\(\)方法。
>
> 但是可以多次调用线程的run\(\)方法！

**4、join**

```java
public final synchronized void join(long millis) throws InterruptedException
public final synchronized void join(long millis, int nanos) throws InterruptedException
public final void join() throws InterruptedException
```

该方法会让当前线程（调用该方法的线程，注意，这里并不是`thread.join()`中thread所表示的线程，而是方法`thread.join()`所在的线程）等待指定时间（如果为0，表示一直等到thread执行完），让join\(\)方法的所属线程thread执行。  
join\(\)方法内部是调用wait\(\)方法实现的。

## **Object类中的方法**

**1、wait**

```java
public final native void wait(long timeout) throws InterruptedException
public final void wait(long timeout, int nanos) throws InterruptedException
public final void wait() throws InterruptedException
```

该方法会让当前线程（该方法调用所在的线程）进入等待状态，直到被该对象（`wait()`方法的`this`对象）的`notify()`或者`notifyAll()`唤醒，或者等待时间结束（`wait(long timeout)`中的`timeout`结束）。

当一个线程执行到wait\(\)方法时，它就进入到一个和该对象相关的等待池中，同时会释放对象的锁（暂时，wait\(long timeout\)超时时间到后还需要返还对象锁），此时其他线程可以访问该对象；这里隐含了一个条件就是：wait\(\)方法只应该被拥有对象monitor的线程调用，即wiat\(\)必须放在synchronized同步块中，否则会在抛出`java.lang.IllegalMonitorStateException`异常。

如果发生下面的4种事件中的任意一件，该线程就会从该对象的等待集中移除，重新加入线程调度。此后，它就会和其他线程一样去争夺该对象的锁，一旦获取了该对象的控制权，所有该对象的同步声明就会重新加载。

* 其他线程触发了该对象的notify方法，那么该线程就可能会被唤醒（并不一定会唤醒，可能很多线程都wait该对象）；
* 其他线程触发了notifyAll方法；
* 被其他线程中断（interrupt\(\)方法）
* 设定的等待时间已到。如果是`wait(0)`或者`wait()`则会一直等待直到唤醒。

正确使用wait\(\)的方式如下：\(等待应该总是发生在轮询中：`waits should always occur in loops`\)

```java
synchronized (obj) {
       while (condition does not hold)
               obj.wait(timeout);
           ... // Perform action appropriate to condition
}
```

如果当前线程在等待之前或者等待期间被其他线程打断（Thread.interrupt\(\)），就会抛出InterruptedException异常。

**2、notify**

```java
public final native void notify();
public final native void notifyAll();
```

`notify()`唤醒等待该对象monitor的单个线程。如果多个线程都在等待该对象monitor，那么被唤醒的线程是不固定的、任意的。被唤醒的线程并不会立即执行，而是等到当前线程放弃了该对象的锁。被唤醒的线程会和其他线程去竞争对该对象的同步操作。也就是说，被唤醒的线程仅仅是被唤醒去参与竞争，并不是唤醒了就开始执行。  
该方法只应该被该对象的monitor的所有者线程调用。一个线程成为该对象的monitor的所有者有以下三种途径：  
①执行该对象的同步的实例方法  
②执行一个同步块，并且该同步了该对象  
③对Class类型的对象，执行该类的一个同步的静态方法

## 常见问题

1、**sleep和wait的区别**

* sleep是Thread类的方法，wait是Object类的方法；
* sleep只是让线程休眠，与锁机制无关，不会释放锁（假如已经拥有锁），而wait则会让所在线程放弃锁。
* wait和sleep都可以通过interrupt方法打断线程的暂停状态，从而使线程立刻抛出InterruptedException

