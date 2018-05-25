# JVM/kernel配置的最佳实践

本小节介绍RocketMQ中Broker的JVM/OS配置参数，将会指出在部署RocketMQ集群之前应该考虑的一些特定的配置项。

## JVM Options {#jvm-options}

推荐JDK 1.8版本，设置以Server模式运行，并设置8g的堆大小。同时，将Xms和Xmx的值保持一致，防止JVM对堆进行resizing。JVM 具体配置如下：

```text
-server -Xms8g -Xmx8g -Xmn4g
```

如果你不关心Broker的启动时间， 建议设置pre-touch来确保在JVM初始化时（在调用main函数之前）使用所有可用的内存分页，如下：

```text
-XX:+AlwaysPreTouch
```

禁用偏向锁可能会缩减JVM停顿时间：

```text
-XX:-UseBiasedLocking
```

在JDK 1.8中，推荐使用G1垃圾收集器：

```text
-XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25 -XX:InitiatingHeapOccupancyPercent=30
```

这些GC选项看起来有一点激进，但是在我们的生产环境中被证明有着非常好的性能。

不要把`-XX:MaxGCPauseMillis`设置的太小，否则JVM会使用非常小的年轻代来达成这一目标，而这会引发非常频繁的minor GC。

推荐使用rolling GC log file：

```text
-XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m
```

如果写GC日志文件会增加Broker的延时，考虑将GC日志文件redirect到a memory file system:

```text
-Xloggc:/dev/shm/mq_gc_%p.log
```

## Linux内核参数 {#linux-kernel-parameters}

There is a `os.sh` script that lists a lot of kernel parameters in folder `bin` which can be used for production use with minor changes. Below parameters need attention, and more details please refer to documentation for /proc/sys/vm/\*\[1\].

**vm.extra\_free\_kbytes**, tells the VM to keep extra free memory between the threshold where background reclaim \(kswapd\) kicks in, and the threshold where direct reclaim \(by allocating processes\) kicks in. RocketMQ uses this parameter to avoid high latency in memory allocation.

**vm.min\_free\_kbytes**, if you set this to lower than 1024KB, your system will become subtly broken, and prone to deadlock under high loads.

**vm.max\_map\_count**, limits the maximum number of memory map areas a process may have. RocketMQ will use mmap to load CommitLog and ConsumeQueue, so set a bigger value for this parameter is recommended.

**vm.swappiness**, define how aggressive the kernel will swap memory pages. Higher values will increase agressiveness, lower values decrease the amount of swap. 10 for this value to avoid swap latency is recommended.

**File descriptor limits**, RocketMQ needs open file descriptors for files\(CommitLog and ConsumeQueue\) and network connections. We recommend set 655350 for file descriptors.

**Disk scheduler**, the deadline I/O scheduler is recommended for RocketMQ, which attempts to provide a guaranteed latency for requests\[2\].

