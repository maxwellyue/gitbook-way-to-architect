##### 1、Java是否可以GC直接内存？

可以，不过GC只会在老年区满了触发Full GC时，才会去顺便清理直接内存的废弃对象。

#####  2、eden survivor区的比例，为什么是这个比例，eden survivor的工作过程？

复制算法中，将新生代分为一个Eden和两个Survivor，每次使用Eden和一个Survivor，当回收时，将Eden和这个Survivor中还存活的对象复制到另一个Survivor中，再清理掉Eden和之前用到的Survivor。

HotSpot默认的比例为8：1：1，每次新生代中可用内存空间为整个新生代容量的90%，只有10%的内存会被“浪费”。假如在复制时，发现Survivor不够用，会让老年代进行分配担保。所以也可以看出，复制算法更适用于新生代，即对象存活时间较短的区域。

##### 3、内存分配策略？

* 对象优先在Eden分配

* 大对象直接进入老年代 
  通过设置参数`-XX:PretenureSizeThreshold`参数，令大于该设置值的对象直接在老年代分配。这样做的目的是为了避免在Eden区和两个Suvivor区之间发生大量的内存复制。

* 长期存活的对象将进入老年代
  虚拟机给每个对象定义一个对象年龄计数器（在对象头中）。如果对象在Eden出生并且经过第一次Monior GC后仍然存活，并且被Survivor容纳的话，就将该对象的年龄设为1。对象在Survivor区中每熬过依次Monior GC，年龄就增加1。当该对象的年龄增加到一定程度（默认为15），就会进入老年代。可以通过参数`-XX:MaxTenuringThreshold`设置年龄阈值。

  * 动态对象年龄判定  
  如果在Survivor区间中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄&gt;=该年龄的对象就可以直接进入老年代，无需等到参数`-XX:MaxTenuringThreshold`设置的年龄。


* 空间分配担保
<br>在发生Monior GC之前，虚拟机会先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果大于，则Monior GC可以确保是安全的。如果小于，则虚拟机会查看`HandlePromotionFailure`设置值是否允许担保失败，如果允许，那么会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，就尝试进行一次Monior GC（有风险），如果小于或者`HandlePromotionFailure`设置不允许冒险，则此时改为进行一次Full GC。



