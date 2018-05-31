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

