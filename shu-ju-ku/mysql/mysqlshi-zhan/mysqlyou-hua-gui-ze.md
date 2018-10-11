## MySQL的优化思路

---

MySQL的优化，其实是一个非常大的话题。因为，MySQL的性能的影响因素很多，通常可以总结为以下几个方面：

* **硬件**性能，如硬盘类型和RAID级别，CPU核数和主频，内存大小，网络设备的传输速率等
* **操作系统配置**，如IO规划与配置，Swap交换分区，内核参数
* **应用**，如连接池的管理，表结构设计，索引的设计，SQL的质量，是否需要读写分离，是否需要分库分表等
* **MySQL配置**

一般来说，优化成本由高到低为硬件&gt;系统配置&gt;表结构/SQL语句/索引，而优化效果则与之相反：硬件&lt;系统配置&lt;表结构/SQL语句/索引。

以上几个方面，拿其中任何一个方向来讲也都是一个非常大的话题。作为开发人员，一般只需要关注应用层面，即保证使用MySQL方式的合理性。但是，在业务初期，就应该考虑到业务的数据量，访问量等，以便DBA可以根据我们提供的这些信息，在合适的硬件或网络资源中创建数据库实例，并做好操作系统或MySQL配置。更进一步，甚至可以考虑有没有更好的存储方案比MySQL更适合，比如MongoDB，Redis等。

虽然除了应用层之外，我们不需要做过多关注，但是了解其他方面的内容和细节，对于更好地使用MySQL或者解决一些实际问题还是很有帮助的。所以下面就简单说一下硬件、网络、操作系统配置，MySQL配置。

## 硬件与操作系统

---

### 硬件

**CPU：**CPU的两个关键因素就是核数、主频。根据不同的业务类型进行选择：

* CPU密集型，即计算比较多，则要求主频高且核数多

* IO密集型：查询比较多，则要求核数要多，主频不一定高

**内存**：越大越好

**硬盘**

硬盘性能的主要指标有IO读写时延、IOPS和吞吐量。

* IOPS：硬盘每秒进行读写的操作次数。
* 吞吐量：硬盘每秒成功传送的数据量，即读取和写入的数据量。
* IO读写时延：硬盘连续两次进行读写操作所需要的最小时间间隔。

而不同的硬盘类型，其性能差异较大，固态硬盘SSD的性能要远高于传统硬盘HDD。

此外，不同的RAID级别的硬盘组合，其读写性能也不同。

> 常见的RAID级别
>
> RAID0：也就是常说的数据条带化\(Data Stripping\)，数据被分散存放在阵列中的各个物理磁盘上，需要2块及以上的硬盘，成本低，性能和容量随硬盘数递增，在所有的RAID级别中，RAID 0的速度是最快的，但是RAID 0没有提供冗余或错误修复能力，如果一个磁盘（物理）损坏，则所有的数据都无法使用。
>
> RAID1：也就是常说的数据镜像\(Data Mirroring\)，2块及以上的硬盘\(偶数个\)，被分为2组，数据在每组磁盘中各有一份，若其中一组有磁盘损坏，另一组可以保证数据访问不会中断。RAID1同RAID0一样，有很好的读取速度，但是写的速度，有所下降。
>
> RAID 5： 一种数据安全、性能、容量、成本、可行性都相对兼顾的解决方案，可以理解为是RAID 0和RAID 1的折衷方案，有和RAID 0相近似的数据读取速度，有比RAID1低的容灾能力，写入数据的速度比RAID1慢。

**网络设备**：使用流量支持更高的网络设备（交换机、路由器、网线、网卡、HBA卡等）

### 操作系统

主要是避免使用SWAP分区，并调整IO调度策略。

**避免使用SWAP分区**：对于频繁进行读写操作的系统而言，数据看似在内存而实际上在磁盘是非常糟糕的，响应时间的增长很可能直接拖垮整个系统。而Linux不会因为MySQL很重要就避免将分配给MySQL的地址空间映射到swap上，所以直接不使用swap分区。

**IO调度策略调整为Deadline模式：**目前主流Linux发行版本使用三种I/O调度器：DeadLine、CFQ、NOOP，通常来说Deadline适用于大多数环境,特别是写入较多的文件服务器，从原理上看，DeadLine是一种以提高机械硬盘吞吐量为思考出发点的调度算法，尽量保证在有I/O请求达到最终期限的时候进行调度，非常适合业务比较单一并且I/O压力比较重的业务，比如Web服务器，数据库应用等。CFQ 为所有进程分配等量的带宽,适用于有大量进程的多用户系统，CFQ是一种比较通用的调度算法，它是一种以进程为出发点考虑的调度算法，保证大家尽量公平，为所有进程分配等量的带宽,适合于桌面多任务及多媒体应用。NOOP 对于闪存设备和嵌入式系统是最好的选择。对于固态硬盘来说使用NOOP是最好的，DeadLine次之，而CFQ效率最低。

## MySQL配置

---

**使用InnoDB存储引擎基础优化参数**

```bash
innodb_buffer_pool_size       # 没有固定大小，50%测试值，看看情况再微调。但是尽量设置不要超过物理内存70%
innodb_file_per_table=(1,0)
innodb_flush_log_at_trx_commit=(0,1,2) # 1是最安全的，0是性能最高，2折中
binlog_sync
Innodb_flush_method=(O_DIRECT, fdatasync)
innodb_log_buffer_size        # 100M以下
innodb_log_file_size          # 100M 以下
innodb_log_files_in_group     # 5个成员以下,一般2-3个够用（iblogfile0-N）
innodb_max_dirty_pages_pct    # 达到百分之75的时候刷写 内存脏页到磁盘。
log_bin
max_binlog_cache_size         # 可以不设置
max_binlog_size               # 可以不设置
innodb_additional_mem_pool_size    #小于2G内存的机器，推荐值是20M。32G内存以上100M
```

具体解释请参考：[基本配置](/shu-ju-ku/mysql/ji-ben-pei-zhi.md)

## MySQL使用

---



SQL优化方向：

执行计划、索引、SQL改写

架构优化方向：

高可用架构、高性能架构、分库分表

连接层（基础优化）

设置合理的连接客户和连接方式

```
    max_connections           # 最大连接数，看交易笔数设置    
    max_connect_errors        # 最大错误连接数，能大则大
    connect_timeout           # 连接超时
    max_user_connections      # 最大用户连接数
    skip-name-resolve         # 跳过域名解析
    wait_timeout              # 等待超时
    back_log                  # 可以在堆栈中的连接数量
```

### 

## 参考

---

[RAID在数据库存储上的应用](https://www.cnblogs.com/seusoftware/p/3195594.html)

[MySQL 优化实施方案](http://clsn.io/clsn/lx287.html)

[OLTP和OLAP浅析](http://blog.51cto.com/76287/885475)

[MySQL如何避免使用swap](https://blog.csdn.net/John_Chang11/article/details/52043830)

[调整 Linux I/O 调度器优化系统性能](https://www.ibm.com/developerworks/cn/linux/l-lo-io-scheduler-optimize-performance/index.html)

