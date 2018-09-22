# synchronize关键字

**synchronized关键字**

* synchronized关键字是最基本的互斥同步手段。

* synchronized关键字编译后会在同步块的前后分别形成monitorenter和monitorexit字节码指令。这两个字节码都需要一个reference类型的参数指明要锁定和解锁的对象。如果在代码中指定synchronized的对象参数，那就是这个对象的reference。如果没有明确指定，就根据synchronized修饰的是实例方法还是类方法，取对应的对象实例或Class对象来作为锁对象。

* 在执行monitorenter指令时，首先尝试获取对象的锁，如果该对象没被锁定或者当前线程已经拥有了该对象的锁，就把锁的计数器加1；在执行monitorexit的时候，会将锁计数器减1，当计数器为0时，锁就被释放。如果获取对象锁失败，当前线程就会阻塞等待，直到对象锁被另外一个线程释放。

* synchronized对同一个线程是可重入的，不会出现自己将自己锁死的现象。

* 同步块在已进入的线程执行完之前，会阻塞后面的线程的进入。

* synchronized是重量级的操作：Java线程是映射到操作系统的原生线程之上的，阻塞或唤醒一个线程都需要操作系统的帮助，这就需要从用户态转换到核心态中，因此状态转换会耗费很多处理器时间，这个时间可能比用户代码执行的时间还长。

* 虚拟机会进行一些优化：在通知操作系统阻塞线程之前加入一段自旋等待过程，避免频繁地切入到核心态中。



扩展阅读

[What's the meaning of an object's monitor in Java? Why use this word?](https://stackoverflow.com/questions/9848616/whats-the-meaning-of-an-objects-monitor-in-java-why-use-this-word)



