## 什么是ThreadGroup

线程组ThreadGroup表示一组线程的集合。

线程组是为了方便和统一多个线程的管理而产生的。我们知道，在一个 Java 程序运行的时候会默认创建一个线程，我们称其为主线程（即为执行 `main`方法的线程）。其实，在一个 Java 程序运行的时候也会创建一个线程组，而这个主线程正是属于这个线程中的。

```java
public class ThreadGroupExample {

    public static void main(String[] args) {
        Thread curThread = Thread.currentThread();
        ThreadGroup curGroup = curThread.getThreadGroup();
        System.out.println(String.format("thread name is [%s], threadGroup name is [%s] ", curThread.getName(), curGroup.getName()));
    }
}
//输出如下
thread name is [main], threadGroup name is [main] 
```

线程组中不仅可以包含线程，也可以包含线程组，这个有点类似于文件夹的概念，线程对应的就是文件，线程组对应的就是文件夹，文件夹中不仅可以包含文件，也可以包含文件夹。

`ThreadGroup`中主要属性：所属父线程组（parent）、名字（name）、其中线程的最大优先级（maxPriority）、是否已经被销毁（destroyed）、是否为守护线程组（daemon）。

当新建一个线程 / 线程组之后，如果你没有给这个新建的线程 / 线程组指定一个父线程组，那么其默认会将当前执行创建线程 / 线程组代码的线程所属的父线程组作为新的线程 / 线程组的父线程组。

同时，一个线程只有调用了其 `start`方法之后，其才真正算是被添加到了对应的线程组中。

## ThreadGroup 主要API

```java
// 创建一个指定名称（name）的线程组，以调用这个构造方法的线程所在的线程组作为父线程组
ThreadGroup​(String name) 

// 创建一个指定名称（name）的线程组，以 parent 线程组作为父线程组
ThreadGroup​(ThreadGroup parent, String name)

// 返回在当前线程组和子线程组中活动的线程的估计数量（注意是估计数量）
int activeCount​()

// 返回在当前线程组和子线程组中活动的线程组的估计数量（注意是估计数量）
int activeGroupCount​() 

// 判断当前执行这个方法的线程有没有权限更改当前线程组的属性，如果没有，抛出SecurityException异常
void checkAccess​()  

// 清除当前线程组和其子线程组，需要保证当前线程组和其子线程组中的所有线程都已经停止了
void destroy​()  

//相当于enumerate​(Thread[] list, true)
int enumerate​(Thread[] list)   

// 将当前线程组中的线程拷贝到参数指定的线程数组中，如果 recurse 参数为 true，
// 那么会递归将其子线程组中的线程也拷贝，
// 如果线程数组的长度小于线程组中线程的数量，那么多余的线程不会拷贝
int enumerate​(Thread[] list, boolean recurse)  

// 相当于enumerate​(ThreadGroup[] list,true) 
int enumerate​(ThreadGroup[] list)  

// 将当前线程组（不包括本身）中的子线程组拷贝到参数指定的线程组数组中，如果 recurse 参数为 true，
// 那么会递归将其子线程组中的子线程组也拷贝，
// 如果线程数组的长度小于线程组中线程的数量，那么多余的线程不会拷贝
int enumerate​(ThreadGroup[] list, boolean recurse) 

// 获取线程组中最大的线程优先级
int getMaxPriority​()   

// 获取线程组名
String getName​()  

// 获取线程组的父线程组
ThreadGroup getParent​()    

// 中断线程组中所有的线程（调用线程的 Thread.interrupt()方法）
void interrupt​()    

// 判断当前线程组是否为守护线程组
boolean isDaemon​() 

// 判断线程组是否已经销毁
boolean isDestroyed​()  

// 打印线程组的相关信息到控制台中，仅用于调试
void list​()    

// 判断当前线程组是否为线程组g的父线程组或者是祖先线程组
boolean parentOf​(ThreadGroup g)    

// 将线程组设置为守护线程组或者普通线程组
void setDaemon​(boolean daemon) 

// 设置当前线程组中线程允许的最大优先级
void setMaxPriority​(int pri)
```

## 使用场景

ThreadGroup的目的就是对一组线程进行统一管理，比如需要在某一时刻或满足某一条件时将一些线程都中断，或者对线程设置一些统一的属性，或者对一组线程中的异常进行统一处理等等，都可以考虑使用线程组。

TODO：用到的时候，再补充实例代码

## 参考

[线程组和 ThreadLocal](https://my.oschina.net/JiangTun/blog/1833826)

[Java\_多线程 \(线程组\)](https://www.jianshu.com/p/bd97f7d8811e)

[Java并发编程与技术内幕:ThreadGroup线程组应用](https://blog.csdn.net/Evankaka/article/details/51627380)

  














