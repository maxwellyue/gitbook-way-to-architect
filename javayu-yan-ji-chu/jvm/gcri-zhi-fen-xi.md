通过配置一下参数让JVM输出日志：

```
-XX:+PrintGCDetails     打印GC详情
-XX:+PrintGCDateStamps  打印GC发生的时间
-XX:+PrintGCTimeStamps  打印GC停顿的时间
```

比如：

```
2018-03-10T14:45:37.987-0200: 151.126: 
  [GC (Allocation Failure) 151.126:
    [DefNew: 629119K->69888K(629120K), 0.0584157 secs]
    1619346K->1273247K(2027264K), 0.0585007 secs] 
  [Times: user=0.06 sys=0.00, real=0.06 secs]
```

其中，

2018-03-10T14:45:37.987-0200：GC事件\(GC event\)开始的时间点

151.126：GC时间的开始时间，相对于JVM的启动时间，单位是秒

GC：说明这次垃圾收集的停顿类型，可能为GC或者Full GC，如果有Full，说明发生了Stop The World，而不是用来区分新生代GC还是老年代GC。

Allocation Failure：引起垃圾回收的原因，本次GC是因为年轻代中没有任何合适的区域能够存放需要分配的数据结构而触发的.

DefNew：表示GC发生的区域，这里显示的区域名与使用的GC收集器相关，这个例子中，使用的是Serial收集器中的新生代名为“Default New Generation”，所以显示的是DefNew。如果采用ParNew收集器，新生代名称就是ParNew，意为“Parallel New Generation”，如果采用的是Parallel Scavenge收集器，它的新生代叫“PSYoungGen”。老年代和永久代同理，名称也是由收集器决定的。

629119K-&gt;69888K\(629120K\)表示：GC前该内存区域已使用的容量-&gt;GC后该内存区域已使用容量（该区域内存总容量）。

0.0584157 secs：该区域GC耗时，单位是秒。

1619346K-&gt;1273247K\(2027264K\)表示：GC前堆已使用的容量-&gt;GC后堆已使用容量（Java堆内存总容量）。

0.0585007 secs：GC事件的总的持续时间，单位是秒。

\[Times: user=0.06 sys=0.00, real=0.06 secs\]： 与Linux的time命令输出的时间含义一致。

* **user**
  – 此次垃圾回收，垃圾收集线程消耗的CPU时间
* **sys**
  – 操作系统调用\(OS call\) 以及等待系统事件的时间
* **real**
  – 应用程序暂停的时间。 由于串行垃圾收集器\(Serial Garbage Collector\)只会使用单个线程, 所以 real time 等于 user 以及 system time 的总和.



