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



## jstat

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



