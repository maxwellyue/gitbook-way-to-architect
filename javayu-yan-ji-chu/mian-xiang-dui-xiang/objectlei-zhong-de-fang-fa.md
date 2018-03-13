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

  * 获取class之后可以利用反射特性进行操作：
  ```
this.getClass().newInstance(); //用缺省构造函数创建一个该类的对象
this.getClass().getInterfaces(); //获得此类实现的接口信息
this.getClass().getMethods();//获得此类实现的所有公有方法
```
  * 除此之外，还有另外两种获取Class对象的方法：①类.class；②Class类的静态方法Class.forName("包名.类路径")。
  

* 对象输出格式：toString\(\)

* 对象复制：clone\(\)

  * new可以创建一个对象，但该对象的属性都会是初始值；假如现在已经有一个对象Person，且此时Person对象的name属性和age属性都已被赋值，此时，想要得到一个与该对象的拷贝，就可以使用clone方法。
  
  * 假如对象Person中有一个集合属性List<Pet> petList，petList中有两个元素Dog和Cat，那使用clone方法会不会将petList中的元素Dog和Cat也复制一份，还是仅仅复制元素的引用：这就是浅拷贝和深拷贝的问题。默认是浅拷贝，即只会复制petList中的元素Dog和Cat的引用。
  
  * 如果想要深拷贝一个对象， 这个对象必须要实现Cloneable接口，实现clone方法，并且在clone方法内部，把该对象引用的其他对象也要clone一份，这就要求这个被引用的对象必须也要实现Cloneable接口并且实现clone方法。

* 其他：registerNatives\(\)
  * 其主要作用是将C/C++中的方法映射到Java中的native方法，实现方法命名的解耦。



---
内容来源：
[详解 equals() 方法和 hashCode() 方法](http://www.importnew.com/25783.html)

[什么时候需要重写equals方法？为什么重写equals方法，一定要重写HashCode方法？](http://blog.csdn.net/championhengyi/article/details/53490549)

[Java总结篇系列：java.lang.Object](https://www.cnblogs.com/lwbqqyumidi/p/3693015.html)

[详解Java中的clone方法 -- 原型模式](http://blog.csdn.net/zhangjg_blog/article/details/18369201)
