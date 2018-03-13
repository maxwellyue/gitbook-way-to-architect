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



