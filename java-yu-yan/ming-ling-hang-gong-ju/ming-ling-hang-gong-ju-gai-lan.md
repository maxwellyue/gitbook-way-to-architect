# 命令行工具概览

Java命令行工具主要有以下几种：

| **名称** | **主要功能** |
| --- | --- | --- | --- | --- | --- | --- |
| jps | JVM Process Status Tool，显示指定系统内所有HotSpot虚拟机进程 |
| jstat | JVM Statistics Minitoring Tool，用于收集HotSpot虚拟机各方面的运行数据 |
| jinfo | Configuration Info for Java，显示虚拟机配置信息 |
| jmap | Memory Map for Java，生成虚拟机的内存转储快照（heapdump）文件 |
| jhat | JVM Heap Dump Browser，用于分析heapdump文件，它会建立一个HTTP/HTML服务器，让用户可以在浏览器上查看分析结果 |
| jstack | Stack Trace for Java，显示虚拟机的线程快照 |

## **jps：查看虚拟机进程**

**列出正在运行的虚拟机进程，并显示主类（main\(\)函数所在的类）的名称，以及这些进程的本地虚拟机的唯一ID（LVMID，Local Virtual Machine Identifier，**LVMID与操作系统的进程ID即PID是一致的**）**。当一台主机上有多个Java程序（虚拟机进程）时，无法根据进程名称定位时，就可以用jps命令显示主类来进行区分。

```bash
root@arch:~# jps -mlVv
2245 com.maxwell.xxxx.service.Launcher -XX:+HeapDumpOnOutOfMemoryError  -Xms1G -Xmx1G -XX:MaxMetaspaceSize=256M -Xss512K -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n 
14091 sun.tools.jps.Jps -mlVv -Denv.class.path=:/usr/java/jdk1.8.0_51/lib/dt.jar:/usr/java/jdk1.8.0_51/lib/tools.jar:/jre/lib -Dapplication.home=/usr/java/jdk1.8.0_51 -Xms8m
```

可以看出，jps会显示pid，主类名，启动参数JAVA\_OPS

jps 参数

| -m | 输出虚拟机进程启动时传递给主类的main\(\)函数的参数 |
| --- | --- | --- | --- |
| -l | 输出主类的全名，如果进程执行的是jar包，输出jar路径 |
| -v | 输出JVM启动参数 |
| -V | 通过由标志文件（.hotspotrc文件或由-XX：Flags = &lt; 文件名 &gt;参数指定的文件）传递给JVM的启动参数。 |

注意：**jps只能显示当前用户的java进程**，要显示其他用户的还是只能用unix/linux的ps命令。

## jstat：虚拟机统计信息监控工具

jstat（JVM Statistics Monitoring Tool）是用于监控虚拟机各种运行状态信息的命令行工具。它可以显示本地或远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据，在没有GUI图像界面，只提供了纯文本控制台环境的服务器上，它是运行期定位虚拟机性能问题的首选工具。

jstat命令格式：

```text
jstat [option vmid [interval[s|ms] [count]] ]
```

> 如果是本地虚拟机进程，VMID和LVMID是一致的，如果是远程虚拟机进程，那VMID的格式应当是：`[protocol:][//] lvmid [@hostname[:port]/servername]`。
>
> --interval：查询间隔
>
> --count：查询次数
>
> 如果省略这两个参数，说明只查询一次。假设需要每250毫秒查询一次进程5828垃圾收集状况，一共查询5次，那命令行如下：`$ jstat -gc 5828 250 5`

选项option代表用户查询的虚拟机信息，主要分为3类：类装载、垃圾收集和运行期编译状况，具体选项及作用参见下表：

| **选项** | **作用**  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| -class | 监视类装载、卸载数量、总空间及类装载所耗费的时间 |
| -gc | 监视Java堆状况，包括Eden区、2个Survivor区、老年代、永久代等的容量 |
| -gccapacity | 监视内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大和最小空间 |
| -gcutil | 监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比 |
| -gccause | 与-gcutil功能一样，但是会额外输出导致上一次GC产生的原因 |
| -gcnew | 监视新生代GC的状况 |
| -gcnewcapacity | 监视内容与-gcnew基本相同，输出主要关注使用到的最大和最小空间 |
| -gcold | 监视老年代GC的状况 |
| -gcoldcapacity | 监视内容与——gcold基本相同，输出主要关注使用到的最大和最小空间 |
| -gcpermcapacity | 输出永久代使用到的最大和最小空间 |
| -compiler | 输出JIT编译器编译过的方法、耗时等信息 |
| -printcompilation | 输出已经被JIT编译过的方法 |

举例

```bash
$ jstat -gc 2245
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
2048.0 2048.0  0.0   1792.0 345088.0 203408.5  699392.0   76042.0   48768.0 46450.5 5248.0 4871.2   2191   11.221   2      0.142   11.363
# 说明(单位为kB)
# 年轻代中第一个survivor的总容量(S0C)为2048.0，目前已使用(SOC)0.0
# 年轻代中第二个survivor的总容量(S1C)为2048.0，目前已使用(S1C)1792.0
# 年轻代中Ede区的总容量（EC）为345088.0，目前已使用(EU)203408.5
# 老年代总容量(EO)为699392.0，目前已使用(EU)76042.0
# 方法区总容量(MO)为48768.0，目前已使用(MU)46450.5
# 压缩类空间大小(CCSC)为5248.0，目前已使用4871.2
# 年轻代发生GC次数(YGC)为2191，总耗时(YGCT)为11.221s
# 发生FGC次数(FGC)为2，总耗时(FGCT)为0.142s
# GC总耗时11.363s

$ jstat -class 2245
Loaded  Bytes  Unloaded  Bytes     Time
  7280 13670.8        0     0.0       7.62
# 共加载了7248个类，占用13670.8bytes空间，总耗时7.62s
```

## **jinfo： Java配置信息工具**

jinfo（Configuration Info for Java）的作用是**查看和调整虚拟机的各项参数**。

> 使用jps的命令的-v参数可以查看虚拟机启动时显示指定的参数列表，但如果想知道未被显示指定的参数的系统默认值，可以①使用jinfo的-flag选项进行查询了；或使用②java -XX:+PrintFlagsFinal查看参数默认值。

jinfo可以使用`-sysprops`选项把虚拟机进程的`System.getProperties()`的内容打印出来，并可以使用`-flag[+|-]name`或`-flag name=valule`修改一部分运行期可写的虚拟机参数值。

jinfo命令格式：

```text
jinfo [option] pid
```

| -flag name | 输出name属性的值 |
| --- | --- | --- | --- | --- |
| -flag \[+\|-\]name | 启用或禁用name功能 |
| -flag name=value | 设置name属性的值为value |
| -flags | 输出JVM使用的属性及值 |
| -sysprops | 输出Java系统属性和值 |

示例

```bash
$ jinfo -flag NewSize 2245
-XX:NewSize=357564416

$ jinfo -flags 2245
Attaching to process ID 2245, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.51-b03
Non-default VM flags: 
    -XX:CICompilerCount=2 
    -XX:+HeapDumpOnOutOfMemoryError 
    -XX:HeapDumpPath=null 
    -XX:InitialHeapSize=1073741824 
    -XX:MaxHeapSize=1073741824 
    -XX:MaxMetaspaceSize=268435456 
    -XX:MaxNewSize=357564416 
    -XX:MinHeapDeltaBytes=524288 
    -XX:NewSize=357564416 
    -XX:OldSize=716177408 
    -XX:ThreadStackSize=512 
    -XX:+UseCompressedClassPointers 
    -XX:+UseCompressedOops 
    -XX:+UseParallelGC
Command line:  
    -XX:+HeapDumpOnOutOfMemoryError 
    -XX:HeapDumpPath=/xxxx/xxxx/xxxx/xxxxx_gc.log 
    -Xms1G -Xmx1G -XX:MaxMetaspaceSize=256M -Xss512K 
    -Xdebug -Xnoagent -Djava.compiler=NONE 
    -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5066  
    -Dfile.encoding=UTF-8 
```

## **jmap： Java内存映像工具**

jmap（Memory Map for Java）命令用于输出堆转储快照（一般称为heapdump或dump文件）。

> 如果不使用jmap命令，要想获取Java堆转储快照还有一些比较”暴力“的手段，比如：
>
> ①-XX:+HeapDumpOnOutOfMemoryError：让虚拟机在OOM异常出现之后自动生生成dump文件；
>
> ②-XX:+HeapDumpOnCtrlBreak：可以使用\[Ctrl\]+\[Break\]键让虚拟机生成dump文件
>
> ③在Linux系统下通过Kill -3命令发送进程退出信号”恐吓“一下虚拟机，也能拿到dump文件。

jmap的作用并不仅仅是为了获取dump文件，它还可以查询finalize执行队列，Java堆和永久代的详细信息，如空间使用率、当前用的是那种收集器等。

jmap命令格式：

```text
jmap [option] vmid
```

option选项合法值与具体含义如下：

| **选项** | **作用** |
| --- | --- | --- | --- | --- | --- | --- |
| -dump | 生成Java堆转储快照。格式为：`-dump:[live,]format=b,file=<filename>`，其中live子参数说明是否只dump出存活的对象 |
| -finalizerinfo | 显示在F-Queue中等待Finalizer线程执行finalize\(\)方法的对象。 |
| -heap | 显示Java堆详细信息，如使用哪种回收器、参数配置、分代状况等。 |
| -histo\[:live\] | 显示堆中对象统计信息，包括类、实例数量和合计容量 |
| -permstat | 以ClassLoader为统计口径显示永久代内存状态。 |
| -F | 当虚拟机进程对-dump选项没有响应时，可使用这个选项强制生成dump快照。 |

示例

```bash
# 查看堆信息
$ jmap -heap 2245
using thread-local object allocation.
Parallel GC with 2 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 1073741824 (1024.0MB)
   NewSize                  = 357564416 (341.0MB)
   MaxNewSize               = 357564416 (341.0MB)
   OldSize                  = 716177408 (683.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 268435456 (256.0MB)
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 351272960 (335.0MB)
   used     = 84463000 (80.5501937866211MB)
   free     = 266809960 (254.4498062133789MB)
   24.04483396615555% used
From Space:
   capacity = 3145728 (3.0MB)
   used     = 1851440 (1.7656707763671875MB)
   free     = 1294288 (1.2343292236328125MB)
   58.855692545572914% used
To Space:
   capacity = 3145728 (3.0MB)
   used     = 0 (0.0MB)
   free     = 3145728 (3.0MB)
   0.0% used
PS Old Generation
   capacity = 716177408 (683.0MB)
   used     = 86170368 (82.178466796875MB)
   free     = 630007040 (600.821533203125MB)
   12.03198635386164% used

19527 interned Strings occupying 1987528 bytes.

# 生产dump文件
$ jmap -dump:format=b,file=xxxxx.bin 2245
Dumping heap to /root/xxxxx.bin ...
Heap dump file created
```

小技巧：如果执行jmap -histo &lt;pid&gt; ，会发现有非常多的类信息，可以使用如下命令查看某个pid的java服务占用内存排名前n的类：

```text
jmap -histo <pid> | head -n
```

## **jhat：虚拟机堆转储快照分析工具**

Sun JDK提供了jhat（JVM Heap Analysis Tool）命令与jmap搭配使用，来分析jmap生成的堆转储快照。jhat内置了一个微型的HTTP/HTML服务器，生成dump文件的分析结果后，可以在浏览器中查看，不过实事求是地说，在实际工作中，除非真的没有别的工具可用，否则一般不会去直接使用jhat命令来分析dump文件，主要原因有二：一是一般不会在部署应用程序的服务器上直接分析dump文件，即使可以这样做，也会尽量将dump文件拷贝到其他机器上进行分析，因为分析工作时一个耗时且消耗硬件资源的过程，既然都要在其他机器上进行，就没必要收到命令行工具的限制了。另外一个原因是jhat的分析功能相对来说很简陋，VisualVM以及专门分析dump文件的Eclipse Memory Analyzer、IBM HeapAnalyzer等工具，都能实现比jhat更强大更专业的分析功能。

综上，不用jhat，而是使用VisualVM、Eclipse Memory Analyzer、IBM HeapAnalyzer等工具进行分析。

## jstack： Java堆栈跟踪工具

jstack（Stack Trace for Java）命令用于生成虚拟机当前时刻的线程快照（一般称为threaddump或javacore文件）。

线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等都是导致线程长时间停顿的常见原因。线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做些什么事情，或者等待着什么资源。

jstack命令格式：

```text
jstack [option] vmid
```

option选项的合法值与具体意义如下：

| **选项** | **作用** |
| --- | --- | --- | --- |
| -F | 强制输出`threaddump`（当`jstack <pid>`没有响应时，即pid进程hung住了） |
| -l | 除堆栈外，显示关于锁的附加信息 |
| -m | 如果调用到本地方法的话，可以显示C/C++的堆栈 |

一般就是`jstack <pid>` ，如：

```text
jstack 2245
```

## 参考

[java高分局之jstat命令使用](https://blog.csdn.net/maosijunzi/article/details/46049117)

[Oracle Java Platform, Standard Edition Tools Reference: ****jstat](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html)

[《深入理解Java虚拟机》——JDK自带命令行工具](https://my.oschina.net/daidetian/blog/304383)

[Java命令学习系列（一）——Jps](http://www.hollischuang.com/archives/105)

[记一次Java内存过大排查](https://jeffinbao.github.io/2016/04/24/20160424-research-on-java-memory-overweighted/)

[jstack\(查看线程\)、jmap\(查看内存\)和jstat\(性能分析\)命令](http://guafei.iteye.com/blog/1815222)

