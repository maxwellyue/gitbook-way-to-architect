# Java8 JVM 参数解读

java命令的通用形式如下：

```text
java [options] classname [args]
java [options] -jar filename [args]
```

这里所说的命令参数就是options中的内容。这些options通过命令行传递给JVM，使每个JVM实例都会有自己的运行特性。每通过java命令启动一个java程序，都会创建一个JVM实例。

## 命令参数包含 {#命令参数包含}

* 标准参数（Standard Option）
* 非标准参数（Non-Standard Options）
* 高级运行时参数（Advanced Runtime Options）
* 高级JIT编译器参数（Advanced JIT Compiler Options）
* 高级服务能力参数（Advanced Serviceability Options）
* 高级垃圾回收参数（Advanced Garbage Collection Options）

### 标准参数（Standard Option） {#标准参数standard-option}

标准参数是被所有JVM实现都要支持的参数。用于做一些常规的通用的动作，比如检查版本、设置classpath等。

`-agentlib:libname[=options]`   
这个命令加载指定的native agent库。理论上这条option出现后，JVM会到本地固定路径下LD\_LIBRARY\_PATH这里加载名字为libxxx.so的库。而这样的库理论上是JVM TI的功能，具体可以参考[JVM TI规范](https://docs.oracle.com/javase/8/docs/platform/jvmti/jvmti.html)

`-agentpath:pathname[=options]`   
这个参数指定了从哪个绝对路径加载agent库。与-agentlib相同，只不过是用绝对路径来指明库文件。

`-client`   
`-server`   
指定JVM的启动模式是client模式还是server模式，具体就是 Java HotSpot Client\(Server\) VM 版本。目前64位的JDK启动，一定是server模式，会忽略这个参数。

`-d32` 和 `-d64`   
这个参数指定JVM启动的环境模式，默认是32位启动，如果系统不支持，那么会报错，-d64模式和-server模式是绑定的，也就是说-d64声明了就默认是server模式。

`-Dproperty=value`   
这个参数是最常见的了，就是设置系统属性，设置后的属性可以在代码中System.getProperty方法获取到。

`disableassertions[:[packagename]...|:classname]`   
`-da[:[packagename]...|:classname]`   
关闭指定包或类下的assertion，默认是关闭的。

`-disablesystemassertions`   
`-dsa`   
关闭系统包类下的assertion。

`-enableassertions[:[packagename]...|:classname]`   
`-ea[:[packagename]...|:classname]`   
同上，开启指定包类下的assertion。

`-enablesystemassertions`   
`-esa`   
同上，开启系统包类下的assertion。

`-javaagent:jarpath[=options]`   
加载指定的java agent，具体需要传入jar路径。

`-verbose:class`   
`-verbose:gc`   
`-verbose:jni`   
这些组合都是用来展示信息的，class展示的每个class的信息，gc展示每个GC事件的信息，jni开启展示JNI调用信息。

`-version`   
显示版本信息，然后退出

### 非标准参数（Non-Standard Options） {#非标准参数non-standard-options}

这里的非标准参数主要是针对官方JVM也就是HotSpot的，当然各家自研的JVM也有自己的非标准参数。

`-X`   
著名的-X参数，就是一个help，展示所有-X开头的参数说明。

`-Xbatch`   
禁止后台编译。将编译过程放到前台任务执行。JVM默认会将编译任务当做后台任务执行。这个参数等价于`-XX:-BackgroundCompilation`

`-Xfuture`   
强制class文件格式检查。

`-Xint`   
在 interpreted-only模式运行程序。编译为native的模式不再生效，所有的字节码都在解释器环境下解释执行。

`-Xinternalversion`   
打印一个更详细的java版本信息，执行后退出。

`-Xloggc:filename`   
设置gc日志文件，gc相关信息会重定向到该文件。这个配置如果和`-verbose:gc`同时出现，会覆盖`-verbose:gc`参数。

> 比如`-Xloggc:/home/admin/logs/gc.log`

`-Xmaxjitcodesize=size`   
为JIT编译的代码设置最大的code cache。默认的设置是240m，如果关闭了tiered compilation，那么默认大小是48m。这个参数和`-XX:ReservedCodeCacheSize`是等价的。

`-Xmixed`   
用解释器执行所有的字节码，除了被编译为native code的hot method。

`-Xmnsize`   
设置初始最大的年轻代堆大小。

> 比如-Xmn256m

`-Xmssize`   
设置初始的堆大小。

`-Xmxsize`   
设置最大的内存分配大小。一般的服务端部署，-Xms和-Xmx设置为同样大小。与`-XX:MaxHeapSize`具有同样的作用。具体设置参考\[3\]

`-Xnoclassgc`   
关闭对class的GC。这样设置可以节约一点GC的时间，不过带来的影响就是class永驻内存，不当的使用会导致OOM风险。

`-XshowSettings:category`   
查看settings信息，category可以是all、locale、properties和vm几部分。

`-Xsssize`   
设置thread stack大小，一般默认的几个系统参数如下：

> Linux/ARM \(32-bit\): 320 KB
>
> Linux/i386 \(32-bit\): 320 KB
>
> Linux/x64 \(64-bit\): 1024 KB
>
> OS X \(64-bit\): 1024 KB
>
> Oracle Solaris/i386 \(32-bit\): 320 KB
>
> Oracle Solaris/x64 \(64-bit\): 1024 KB

`-Xverify:mode`   
设置字节码校验器的模式，默认是remote，即只校验那些不是通过bootstrap类加载器加载的字节码。而还有一个模式还all，即全部都校验。虽然还有一个模式是none，但是本质上jvm不生效这个参数。因为字节码校验是非常重要的，如果关闭，将可能导致class文件格式就是错的，这对于系统稳定和安全来说有重大风险。

### 高级运行时参数（Advanced Runtime Options） {#高级运行时参数advanced-runtime-options}

这类参数控制Java HotSpot VM在运行时的行为。

`-XX:+DisableAttachMechanism`   
设置JVM的attach模式，如果启动了这个参数，jvm将不允许attach，这样类似jmap等程序就无法使用了。

`-XX:ErrorFile=filename`   
当不可恢复的错误发生时，错误信息记录到哪个文件。默认是在当前目录的一个叫做hs\_err\_pid _pid_.log的文件。如果指定的目录没有写权限，这时候文件会创建到/tmp目录下。使用如下：

> -XX:ErrorFile=/var/log/java/java\_error.log

`-XX:MaxDirectMemorySize=size`   
为NIO的direct-buffer分配时指定最大的内存大小。默认是0，意思是JVM自动选择direct-buffer的大小。使用如下：

> -XX:MaxDirectMemorySize=1m

`-XX:ObjectAlignmentInBytes=alignment`   
java对象的内存对齐大小。默认是8字节，JVM实际计算堆内存上限的方法是

> 4GB \* ObjectAlignmentInBytes

`-XX:OnError=string`   
在jvm出现错误\(不可恢复）的时候，执行哪些命令。具体例子如下：

> -XX:OnError="gcore %p;dbx - %p"

`-XX:OnOutOfMemoryError=string`   
同上，具体错误是OOM。

`-XX:-UseCompressedOops`   
禁止使用压缩命令来压缩指针引用。压缩指针是默认开启的，如果使用压缩命令压缩指针，可以在JVM内存小于32G时做到内存压缩，即在64位机器上做到内存指针对齐只占用32位而不是64位。这样对于小于32G的JVM有非常高的性能提升。该参数只在64位JVM有效。

`-XX:+UseLargePages`   
使用大页内存\[4\]，默认关闭，使用该参数后开启。

### 高级JIT编译器参数（Advanced JIT Compiler Options） {#高级jit编译器参数advanced-jit-compiler-options}

这部分的参数主要在动态just in time编译时用到。

`-XX:+BackgroundCompilation`   
后台编译，默认是开启的，如果要关闭，使用`-XX:-BackgroundCompilation`或者`-Xbatch`。

`-XX:CICompilerCount=threads`   
编译时的编译器线程数。server版的JVM默认设置为2，client版本默认设置为1。如果tiered编译开启，则会伸缩到核数个线程。

`XX:CodeCacheMinimumFreeSpace=size`   
编译使用的最小空闲空间。默认是500KB，如果最小空闲空间不足，则编译会停止。

`-XX:CompileThreshold=invocations`   
编译前解释型方法调用次数。设置这个值，在编译前，会用解释器执行方法若干次用于收集信息，从而可以更高效率的进行编译。默认这个值在JIT中是10000次。可以通过使用`-Xcomp`参数来禁止编译时的解释执行。

`-XX:+DoEscapeAnalysis`   
支持转义分析，默认是开启的。

`-XX:InitialCodeCacheSize=size`   
初始化的code cache的大小，默认500KB。这个值应该不小于系统最小内存页的大小。

`-XX:+Inline`   
编译时方法内联。默认是开启的。

`-XX:InlineSmallCode=size`   
设置最大内联方法的代码长度，默认是1000字节，只有小于这个设置的方法才会被编译内联。

`-XX:+LogCompilation`   
编译时日志输出，在编译时会有一个hotspot.log的日志输出到当前工作目录下。可以用`-XX:LogFile`指定不同的目录。默认这个参数是关闭的，即编译日志不输出。这个参数需要和`-XX:UnlockDiagnosticVMOptions`一起使用。也可以使用`-XX:+PrintCompilation`选项在控制台打印编译过程信息。

`-XX:MaxInlineSize=size`   
编译内联的方法的最大byte code大小。默认是35，高于35个字节的字节码不会被内联。

`-XX:+OptimizeStringConcat`   
字符串concat优化。默认开启。

`-XX:+PrintAssembly`   
通过使用外部的_disassembler.so_库打印汇编的字节码和native方法来辅助分析。默认是不开启的，需要和`-XX:UnlockDiagnosticVMOptions`一起使用。

`-XX:+PrintCompilation`   
将方法编译过程打印到控制台。默认不开启。

`-XX:+PrintInlining`   
将内联方法打印出来。默认不开启。

`-XX:ReservedCodeCacheSize=size`   
设置为了JIT编译代码的最大代码cache大小。这个设置默认是240MB，如果关掉了tiered编译，则大小是48MB。这个设置必须比初始化的`-XX:InitialCodeCacheSize=size`设置值大。

`-XX:-TieredCompilation`   
关闭tiered编译，默认是开启的。只有Hotspot支持这个参数。

`-XX:+UseCodeCacheFlushing`   
支持在关闭编译器之前清除code cache。默认是开启的，要关闭就把+换成-。

### 高级服务能力参数（Advanced Serviceability Options） {#高级服务能力参数advanced-serviceability-options}

这部分参数可以做系统信息收集和扩展性的debug。

`-XX:+ExtendedDTraceProbes`   
支持dtrace探测，默认是关闭的。

`-XX:+HeapDumpOnOutOfMemory`   
设置当_java.lang.OutOfMemoryError_发生时，将heap内存dump到当前目录的一个文件。默认是不开启的。

`-XX:HeapDumpPath=path`   
设置在dump heap时将文件dump到哪里。默认是当前目录下 java\_pidpid.hprof这样形式的文件。指定文件时例子如下：

> -XX:HeapDumpPath=/var/log/java/java\_heapdump.hprof

`-XX:LogFile=path`   
指定日志数据被记录在哪里，默认是在当前目录的hotspot.log下。设置例子如下：

> -XX:LogFile=/var/log/java/hotspot.log

`-XX:+PrintClassHistogram`   
支持打印类实例的直方图，在按下ctrl+c时（SIGTERM）触发。默认是关闭的。等价于运行**jmap -histo**命令或者**jcmd pid GC.class\_histogram**命令。

`-XX:+PrintConcurrentLocks`   
支持打印java.util.concurrent的锁信息，在SIGTERM时触发。默认关闭，等价于运行**jstack -l**或者**jcmd pid Thread.print -l**命令。

`-XX:+UnlockDiagnosticVMOptions`   
解锁对JVM进行诊断的选项参数。默认是关闭的，开启后支持一些特定参数对JVM进行诊断。

### 高级垃圾回收参数（Advanced Garbage Collection Options） {#高级垃圾回收参数advanced-garbage-collection-options}

这部分参数控制JVM如何进行垃圾回收（GC）

`-XX:+AggressiveHeap`   
java堆内存优化。默认是关闭的，如果开启后，针对一些长时间运行的且有密集的内存分配操作，JVM根据系统cpu和内存的配置参数来进行优化。

`-XX:+AlwaysPreTouch`   
支持在JVM启动时touch每一页，这样做会导致每页都会进入内存。可以用来模拟测试长时间运行的任务，将虚拟内存全部映射到物理内存。默认是关闭的。

`-XX:+CMSClassUnloadingEnabled`   
支持CMS垃圾回收下的类卸载。默认是开启的。

`-XX:CMSExpAvgFactor=percent`   
设置一个时间百分比，用来加权并发回收统计的指数平均的样本。默认是25%。

`-XX:CMSInitiatingOccupancyFraction=percent`   
设置一个年老代的占比，达到多少会触发CMS回收。默认是-1，任何一个负值的设定都表示了用`-XX:CMSTriggerRatio`来做真实的初始化值。设置方法如下：

> -XX:CMSInitiatingOccupancyFraction=20

`-XX:+CMSScavengeBeforeRemark`   
开启功能在CMSremark前进行Scavenge。默认是关闭的。

`-XX:CMSTriggerRatio=percent`   
设置一个在CMS开始前的内存的触发百分比，针对的是由`-XX:MinHeapFreeRatio`分配的内存。默认是80。

`-XX:ConcGCThreads=threads`   
设置支持并发GC的线程数。默认值依赖于给JVM的CPU数目。设置方法如：

> -XX:ConcGCThreads=2

`-XX:+DisableExplicitGC`   
关闭显式GC调用，即关闭**System.gc\(\)**。默认是可以调用的。

`-XX:+ExplicitGCInvokesConcurrent`   
支持通过**System.gc\(\)**来做并发的GC。默认是不支持的。该参数一定要和`-XX:+UseConcMarkSweepGC`一起使用。

`-XX:+ExplicitGCInvokesConcurrentAndUnloadsClasses`   
支持通过**System.gc\(\)**来做并发的GC并且卸载类。默认是不支持的。该参数一定要和`-XX:+UseConcMarkSweepGC`一起使用。

`-XX:G1HeapRegionSize=size`   
设置在使用G1收集器时Java堆被划分为子区域的大小。在1MB到32MB之间，默认会根据Java堆的大小自动检测。

`-XX:+G1PrintHeapRegions`   
打印出哪些region是被分配的，哪些是被G1取回的。默认是关闭打印的。

`-XX:G1ReservePercent=percent`   
设置一个堆内存的百分比用来作为false ceiling，从而降低使用G1时晋升失败的可能性。默认是10%。

`-XX:InitialHeapSize=size`   
设置初始堆内存大小，需要设置为0或者1024的倍数，设置为0说明初始堆大小等于年轻代加年老代的大小。

`-XX:InitialSurvivorRatio=ratio`   
设置初始的survivor空间占比，当使用throughput型的GC时有效（即-XX:+UseParallelGC 或 -XX:+UseParallelOldGC）。运行过程中survivor空间占比会自动根据应用运行调整，如果关闭了自适应调整策略（`-XX:-UseAdaptiveSizePolicy`），则`XX:SurvivorRatio`参数会成为survivor空间占比。计算survivor空间大小，依赖young的空间大小，计算公式如下

> S=Y/\(R+2\)

其中Y是young空间大小，R是survivor空间占比。一个例子就是如果young空间大小是2MB，而survivor默认占比是8，那么survivor的空间就是0.2MB。

`-XX:InitiatingHeapOccupancyPercent=percent`   
设置一个触发并发GC的堆占用百分比。这个值对于基于整体内存的垃圾回收器有效，比如G1。默认是45%，如果设置为0表示无停顿GC。

`-XX:MaxGCPauseMillis=time`   
设置一个最大的GC停顿时间（毫秒），这是个软目标，JVM会尽最大努力去实现它，默认没有最大值设置。

`-XX:MaxHeapSize=size`   
设置最大堆大小，这个值需要大于2MB，且是1024的整数倍。等价于-Xmx

`-XX:MaxHeapFreeRatio=percent`   
设置在一次GC后最大的堆空闲空间占比。如果空闲堆空间超过这个值，堆空间会被收缩。默认是70%。

`-XX:MaxMetaspaceSize=size`   
为类的元数据进行分配的metaspace最大native内存大小，默认情况这个值无限制。该值依赖于当前的JVM、其他在运行的JVM和系统可用内存。

`-XX:MaxNewSize=size`   
设置最大的年轻代的堆大小。默认自动检测。

`-XX:MaxTenuringThreshold=threshold`   
设置在自适应GC大小的使用占有最大阈值，默认对于parallel（throughput）的是15，对于CMS的是6。

`-XX:MetaspaceSize=size`   
设置一个metaspace的大小，第一次超出该分配后会触发GC。默认值依赖于平台，该值会在运行时增加或减少。

`-XX:MinHeapFreeRatio=percent`   
设置在一次GC后最小的空闲堆内存占比。如果空闲堆内存小于该值，则堆内存扩展。默认是40%。

`-XX:NewRatio=ratio`   
设置年轻代和年老代的比例，默认是2。

`-XX:NewSize=size`   
设置初始的年轻代的大小。年轻代是分配新对象的地方，是 GC经常发生的地方。设置太低，会频繁minor GC，设置太高的话就只会发生Full GC了。Oracle推荐设置为整体内存的一半或者1/4。该参数等价于-Xmn。

`-XX:ParallelGCThreads=threads`   
并行GC时的线程数。默认值是CPU数。

`-XX:+ParallelRefProcEnabled`   
支持并发引用处理，默认是关闭的。

`-XX:+PrintAdaptiveSizePolicy`   
打印自适应调整策略。默认关闭。

`-XX:+PrintGC`   
打印每次GC的消息，默认是关闭的。

`-XX:+PrintGCApplicationConcurrentTime`   
打印上次GC暂停到目前的时间。默认不打印。

`-XX:+PrintGCApplicationStoppedTime`   
打印GC暂停的时间长度。默认不打印。

`-XX:+PrintGCDateStamps`   
打印每个GC的日期时间戳。默认不打印。

`-XX:+PrintGCDetails`   
打印每次GC的细节信息。默认不打印。

`-XX:+PrintGCTaskTimeStamps`   
打印每个独立的GC线程任务的时间戳。默认不打印。

`-XX:+PrintGCTimeStamps`   
打印每次GC的时间戳。默认不打印。

`-XX:+PrintStringDeduplicationStatistics`   
打印细节的deduplication信息。默认不打印。

`-XX:+PrintTenuringDistribution`   
打印所在的年龄代的信息。具体例子如下：

> Desired survivor size 48286924 bytes, new threshold 10 \(max 10\)   
> - age 1: 28992024 bytes, 28992024 total   
> - age 2: 1366864 bytes, 30358888 total   
> - age 3: 1425912 bytes, 31784800 total   
> ...

其中age1是最年轻的survivor，age2存活了2代，以此类推。默认该项关闭。

`-XX:+ScavengeBeforeFullGC`   
在每次Full GC前做一次年轻代的GC。该项默认是开启的。

`-XX:SoftRefLRUPolicyMSPerMB=time`   
设置一个软引用对象在上次被引用后在堆内存中保存的时间。默认是每1MB保存1秒钟。该参数对于client模式和server模式有不同的动作，因为client模式JVM在回收时会强制flush掉软引用，然而server模式会尝试先扩容堆空间。

`-XX:StringDeduplicationAgeThreshold=threshold`   
string对象到达特定的age后会去除重复数据。默认是3，jvm中每次gc后存活的对象，age会加一。string对象在晋升为年老代之前都是去除重复数据的候选对象。

`-XX:SurvivorRatio=ratio`   
eden区和survivor区的比例。默认是8。

`-XX:TargetSurvivorRatio=percent`   
设置在YGC后的期望的survivor空间占比。默认是50%。

`-XX:TLABSize=size`   
设置thread-local allocation buffer \(TLAB\)的初始化大小。

`-XX:+UseAdaptiveSizePolicy`   
使用自适应分代大小。默认是开启的。

`-XX:+UseCMSInitiatingOccupancyOnly`   
设置使用占用值作为初始化CMS收集器的唯一条件。默认是不开启。

`-XX:+UseConcMarkSweepGC`   
设置让CMS也支持老年代的回收。默认是不开启的，如果开启，那么`-XX:+UseParNewGC`也会自动被设置。Java 8 不支持`-XX:+UseConcMarkSweepGC -XX:-UseParNewGC`这种组合。

`-XX:+UseG1GC`   
设置使用G1作为GC收集器。G1比较推荐在大堆应用场景下使用（大于6GB）。

`-XX:+UseGCOverheadLimit`   
设置一种策略用来设置一个时间比率来限制在OOM之前的GC时间。默认是开启的，并行GC时如果有多于98%以上的时间用来gc就会抛出OOM。当堆空间较小时这个参数有助于保护应用程序不至于长时间的停顿没有进展。

`-XX:+UseNUMA`   
使用NUMA\[5\]开启性能优化。默认不开启，该项只有在开启了`-XX:+UseParallelGC`后才有效。

`-XX:+UseParallelGC`   
支持并行的垃圾收集器，即throughput垃圾收集。这可以在多核处理器下提升垃圾收集性能。默认不开启，收集器由系统根据JVM和机器配置自动选择。开启后`-XX:+UseParallelOldGC`选项也自动开启。

`-XX:+UseParallelOldGC`   
支持FULL GC的并行收集器。默认不开启。

`-XX:+UseParNewGC`   
支持在年轻代用多线程进行垃圾收集。默认不开启，使用`-XX:+UseConcMarkSweepGC`时会自动被开启。

`-XX:+UseSerialGC`   
支持使用串行收集器。默认不开启。

`-XX:+UseSHM`   
在Linux环境下支持JVM使用共享内存来设置大页。

`-XX:+UseStringDeduplication`   
支持string的去重存储。默认关闭，要使用该选项，必须使用G1垃圾回收器`-XX:+UseG1GC`。

`-XX:+UseTLAB`   
在年轻代支持thread-local分配block，默认开启。



内容完全复制自：[Java8 JVM参数解读](https://www.zybuluo.com/changedi/note/975529)









