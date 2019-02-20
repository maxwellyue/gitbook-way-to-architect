# JVM常用配置

常用的JVM设置主要从以下几个方面：堆、垃圾收集器、GC日志等。

### 堆配置

包括：堆大小、年轻代和年老代的比例、年轻代中Eden区与Survivor区的比例。

再解释为什么堆大小不能太大（每次GC时间太长），也不能太小（GC频率太高）。

```text
-Xms:初始堆大小
-Xmx：最大堆大小
-XX:NewSize=n:设置年轻代大小
-XX:NewRatio=n:设置年轻代和年老代的比值。如：为3表示年轻代和年老代比值为1：3，年轻代占整个年轻代年老代和的1/4
-XX:SurvivorRatio=n:年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如3表示Eden： 3 Survivor：2，一个Survivor区占整个年轻代的1/5
-XX:MaxPermSize=n:设置持久代大小 //该配置在JDK1.8及更高的版本中，无效，没有该区域了
```

> 说明：  
> 1、一般初始堆和最大堆设置一样。
>
> 因为：现在内存不是什么稀缺的资源，但是如果不一样，从初始堆到最大堆的过程会有一定的性能开销，所以一般设置为初始堆和最大堆一样。
>
> 64位系统理论上可以设置为无限大，但是一般设置为**4G**,因为如果再大，JVM进行垃圾回收出现的暂停时间会比较长，这样全GC过长，影响JVM对外提供服务，所以不能太大。一般设置为4G。

### 收集器设置

即设置使用哪种垃圾收集器，一般会为年轻代设置一种，为年老代设置一种

```text
-XX:+UseSerialGC     设置串行收集器
-XX:+UseParallelGC   设置并行收集器
-XX:+UseParalledlOldGC   设置并行年老代收集器
-XX:+UseConcMarkSweepGC  设置并发收集器
```

这里，选择了一种垃圾回收器之后，该类型的垃圾回收器还会有一些具体的配置信息。

### **打印GC回收的过程日志信息**

```text
-XX:+PrintGC   打印GC日志
-XX:+PrintGCDetails  打印GC日志详情
-XX:+PrintGCTimeStamps  发生GC时，从JVM启动以来经过的秒数
```

## JVM参数一览

1、使用`java -XX:+PrintFlagsFinal`来查看所有 XX 参数和值。

2、查看进程24684的参数

```bash
$ jcmd 24684 VM.flags
  24684:
  -XX:InitialHeapSize=98566144 -XX:MaxHeapSize=1547698176 \
  -XX:MaxNewSize=515899392 -XX:MinHeapDeltaBytes=524288 \
  -XX:NewSize=1572864 -XX:OldSize=96993280 \
  -XX:+UseCompressedClassPointers \
  -XX:+UseCompressedOops -XX:+UseParallelGC 
```







