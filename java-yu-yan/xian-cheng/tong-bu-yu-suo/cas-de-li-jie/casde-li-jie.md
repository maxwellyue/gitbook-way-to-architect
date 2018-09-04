# 理解循环CAS

**使用循环CAS来实现原子操作**

Java中可以通过两种方式来实现原子操作：加锁和循环CAS。

下面是原子操作类AtomicInteger的incrementAndGet实现代码，很好地说明了Java中是如何使用循环CAS来实现原子操作的（最新源码已非如此，但思想是一致的）。

```java
public final int incrementAndGet() {
    for (;;) {//一直循环
        int current = get();//取出AtomicInteger当前的内存值
        int next = current + 1;//要设置的新值
        if (compareAndSet(current, next)){//用CAS操作更新AtomicInteger
            return next;
        }
    }
}
```

`boolean compareAndSet(int current, int newValue)`方法是一个原子方法，该方法首先会先检查`AtomicInteger`的当前数值（从内存中读取，即**CAS操作中内存值**）是否等于`current`（即**CAS操作中预期值**），如果等于，则说明`AtomicInteger`的值没有被其他线程修改过，则会将`AtomicInteger`的值设置为`newValue`（即**CAS操作中要修改的新值**），并返回`true`；如果不等于，说明`AtomicInteger`的值被其他线程修改过，则返回`fasle`。

**CAS实现原子操作的三个问题**

* **ABA问题**
  * 因为CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。 
  * ABA问题的解决思路就是**使用版本号**。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么A－B－A 就会变成1A-2B－3A。 
  * 从Java1.5开始JDK的atomic包里提供了一个类**AtomicStampedReference**来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。
* **②循环时间长开销大**
  * 自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。  
* **③只能保证一个共享变量的原子操作**
  * 当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。
  * 从Java1.5开始JDK提供了**AtomicReference**类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。



#### 内容来源

《Java并发编程的艺术》

[聊聊并发（五）——原子操作的实现原理](https://link.jianshu.com/?t=http://www.infoq.com/cn/articles/atomic-operation)

