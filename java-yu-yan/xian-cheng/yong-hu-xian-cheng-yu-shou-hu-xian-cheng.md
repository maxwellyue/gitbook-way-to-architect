在Java中有两类线程：用户线程\(User Thread\)、守护线程\(Daemon Thread\)。

所谓守护线程，是指在程序运行的时候在后台提供一种通用服务的线程，比如垃圾回收线程就是一个很称职的守护者，并且这种线程并不属于程序中不可或缺的部分。

用户线程和守护线程两者几乎没有区别，唯一的不同之处就在于虚拟机的离开：如果用户线程已经全部退出运行了，只剩下守护线程存在了，虚拟机也就退出了。因为没有了被守护者，守护线程也就没有工作可做了，也就没有继续运行程序的必要了。因此，当所有的非守护线程结束时，程序也就终止了，同时会杀死进程中的所有守护线程。反过来说，只要任何非守护线程还在运行，程序就不会终止。

可以通过如下方法将线程设置为守护线程：

```java
thread.setDaemon(true)
```

在使用守护线程时需要注意一下几点：

* `thread.setDaemon(true)`必须在`thread.start()`之前设置，否则会跑出一个`IllegalThreadStateException`异常。换句话说，不能把正在运行的常规线程设置为守护线程。

* 在Daemon线程中产生的新线程也是Daemon的。

* 守护线程应该永远不去访问固有资源，如文件、数据库，因为它会在任何时候甚至在一个操作的中间发生中断。

* JVM退出时，Daemon线程中的finally块并不一定会执行。所以，在构建Daemon线程的时候，不能依靠finally块中的内容来确保执行关闭或清理资源的逻辑。

* main线程不是守护线程！

```java
public class DaemonThreadExample {

    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                System.out.println("daemon finally run ");
            }
        });
        thread.setDaemon(true);
        thread.start();
        System.out.println("main thread end, jvm will close...");
    }
}
//输出如下
main thread end, jvm will close...
```









**使用守护线程的场景**

* 与系统业务同生死的基础服务（如日志采集/指标收集等）如果是采用SDK的方式，与业务系统在同一个JVM下，这些服务应该在业务线程（或者用户线程）结束后，允许JVM退出，不能因为基础服务的线程还在跑，而业务已经没有线程在跑，而程序还在运行，此时，就需要将基础服务中的相关线程设置为Daemon的。
* 其他有待发现

### 参考

[JAVA并发编程——守护线程\(Daemon Thread\)](https://www.cnblogs.com/luochengor/archive/2011/08/11/2134818.html)

