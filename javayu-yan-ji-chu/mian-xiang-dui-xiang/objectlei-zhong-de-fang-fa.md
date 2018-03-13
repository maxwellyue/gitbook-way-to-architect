首先，Object中一共包含以下12个方法：

```
private static native void registerNatives();

public final native Class<?> getClass();   这个方法可以引出有关反射，类加载机制

public native int hashCode();   这里会引出hashmap实现原理

public boolean equals(Object obj)  这里会引出hashmap实现原理

protected native Object clone() throws CloneNotSupportedException;  这里会引出设计模式

 public String toString()

public final native void notify();   这里会引出线程通信

public final native void notifyAll();  这里会引出线程通信

 public final native void wait(long timeout) throws InterruptedException;  这里会引出线程通信

 public final void wait(long timeout, int nanos) throws InterruptedException  这里会引出线程通信

public final void wait() throws InterruptedException  这里会引出线程通信

protected void finalize() throws Throwable   这里会引出垃圾回收
```

主要分为几类：

* **线程通信类：**notify\(\)，notifyAll\(\)，wait\(long timeout\)，wait\(long timeout, int nanos\)，wait\(\)

* **对象身份类：**equals\(Object obj\)，hashCode\(\)

  * SUN官方的文档中规定"如果重定义equals方法，就必须重定义hashCode方法,以便用户可以将对象插入到散列(哈希)表中"。
  
  * equals()相等，则hashCode()必须相等；equals()不相等，则hashCode()不要求一定不相等，但是不同对象的哈希值不同，可以提供哈希表性能。
  
  * hashCode()相等，equals()不一定相等
  
  * 当我们向哈希表(如HashSet、HashMap等)中添加对象object时，首先调用hashCode()方法计算object的哈希码，通过哈希码可以直接定位object在哈希表中的位置(一般是哈希码对哈希表大小取余)。如果该位置没有对象，可以直接将object插入该位置；如果该位置有对象(可能有多个，通过链表或红黑树实现)，则调用equals()方法比较这些对象与object是否相等，如果相等，则不需要保存object；如果不相等，则将该对象加入到链表中。这也就解释了为什么equals()相等，则hashCode()必须相等。如果两个对象equals()相等，则它们在哈希表(如HashSet、HashMap等)中只应该出现一次；如果hashCode()不相等，那么它们会被散列到哈希表的不同位置，哈希表中出现了不止一次。


* 对象回收类：finalize\(\)

  * Java中每一个对象都将具有finalize这种行为，其具体调用时机在：JVM准备对该对象进行垃圾回收时，将被调用。所以，此方法并不是由我们主动去调用的，而是由JVM调用。（虽然可以主动去调用，此时与其他自定义方法无异）
  

* 对象类加载器：getClass\(\)

  * 获取该Object对象的运行时类对象


* 对象输出格式：toString\(\)

* 对象复制：clone\(\)

  * 会涉及到浅拷贝和深拷贝：

* 其他：registerNatives\(\)
  * 其主要作用是将C/C++中的方法映射到Java中的native方法，实现方法命名的解耦。



---
内容来源：
[详解 equals() 方法和 hashCode() 方法](http://www.importnew.com/25783.html)

[什么时候需要重写equals方法？为什么重写equals方法，一定要重写HashCode方法？](http://blog.csdn.net/championhengyi/article/details/53490549)

[Java总结篇系列：java.lang.Object](https://www.cnblogs.com/lwbqqyumidi/p/3693015.html)
