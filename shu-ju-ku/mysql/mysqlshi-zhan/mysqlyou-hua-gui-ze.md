## MySQL的优化思路

---

MySQL的优化，其实是一个非常大的话题。因为，MySQL的性能的影响因素很多，通常可以总结为以下几个方面：

* **硬件**性能，如硬盘类型，CPU核数和主频，内存大小
* **网络**
* **操作系统配置**，如IO规划与配置，Swap交换分区，内核参数
* **应用**，如连接池的管理，表结构设计，索引的设计，SQL的质量，是否需要分库分表等
* **MySQL配置**

一般来说，优化成本由高到低为硬件&gt;系统配置&gt;表结构/SQL语句/索引，而优化效果则与之相反：硬件&lt;系统配置&lt;表结构/SQL语句/索引。

以上几个方面，拿其中任何一个方向来讲也都是一个非常大的话题。作为开发人员，一般只需要关注应用层面，即保证使用MySQL方式的合理性。但是，在业务初期，就应该考虑到业务的数据量，访问量等，以便DBA可以根据我们提供的这些信息，在合适的硬件或网络资源中创建数据库实例，并做好操作系统或MySQL配置。更进一步，甚至可以考虑有没有更好的存储方案比MySQL更适合，比如MongoDB，Redis等。











## 优化工具

---

### **数据库层面**

检查问题常用工具

```sql
msyqladmin                                 mysql客户端，可进行管理操作
mysqlshow                                  功能强大的查看shell命令
show [SESSION | GLOBAL]variables           查看数据库参数信息
SHOW [SESSION | GLOBAL]STATUS              查看数据库的状态信息
information_schema                         获取元数据的方法
SHOW ENGINE INNODB STATUS                  Innodb引擎的所有状态
SHOW PROCESSLIST                           查看当前所有连接session状态
explain                                    获取查询语句的执行计划
show index                                 查看表的索引信息
slow-log                                   记录慢查询语句
mysqldumpslow                              分析slowlog文件的
```

不常用但好用的工具

```
zabbix                  监控主机、系统、数据库（部署zabbix监控平台）
pt-query-digest         分析慢日志
mysqlslap               分析慢日志
sysbench                压力测试工具
mysql profiling         统计数据库整体状态工具    
Performance Schema      mysql性能状态统计的数据
workbench               管理、备份、监控、分析、优化工具（比较费资源）
```

**应急调优的思路（**针对突然的业务办理卡顿，无法进行正常的业务处理，需要立马解决的场景）

1、show processlist：查询每个MySQL连接（或线程）的当前状态

2、explain  sql，通过执行计划判断，索引问题（有没有、合不合理）或者语句本身问题

3、show status  like '%lock%';    \# 查询锁状态

4、kill SESSION\_ID;   \# 杀掉有问题的session

**常规调优思路（**针对业务周期性的卡顿，例如在每天10-11点业务特别慢，但是还能够使用，过了这段时间就好了）

1、查看slowlog，分析slowlog，分析出查询慢的语句。

2、按照一定优先级，进行一个一个的排查所有慢语句。

3、分析优先级最高的sql，进行explain调试，查看语句执行时间。

4、调整索引或语句本身。

### 系统层面

cpu

```bash
vmstat 
sar top
htop
nmon
mpstat
```

内存

```
free
ps -aux 
```

IO设备（磁盘、网络）

```
iostat
ss
netstat
iptraf
iftop
lsof
```

问题一：cpu负载高，IO负载低

内存不够；磁盘性能差；SQL问题；IO出问题了（磁盘到临界了、raid设计不好、raid降级、锁、在单位时间内tps过高）；tps过高，大量的小数据IO；大量的全表扫描

问题二：IO负载高，cpu负载低

大量小的IO写操作：autocommit，产生大量小IO；IO/PS：磁盘的一个定值，硬件出厂的时候，厂家定义的一个每秒最大的IO次数。

大量大的IO写操作：SQL问题的几率比较大

问题三：IO和cpu负载都很高

硬件不够了或sql存在问题

## 具体优化 

---

首先定位问题点：硬件 --&gt; 系统 --&gt; 应用 --&gt; 数据库 --&gt; 架构（高可用、读写分离、分库分表）

### 硬件优化

主机

> 根据数据库类型，主机CPU选择、内存容量选择、磁盘选择
>
>   平衡内存和磁盘资源
>
>   随机的I/O和顺序的I/O
>
>   主机 RAID卡的BBU\(Battery Backup Unit\)关闭

cpu的选择：

> cpu的两个关键因素：核数、主频
>
>     根据不同的业务类型进行选择：
>
>     cpu密集型：计算比较多，OLTP     主频很高的cpu、核数还要多
>
>     IO密集型：查询比较，OLAP         核数要多，主频不一定高的

内存的选择：

> OLAP类型数据库，需要更多内存，和数据获取量级有关。
>
>     OLTP类型数据一般内存是cpu核心数量的2倍到4倍，没有最佳实践。

存储方面：

> 根据存储数据种类的不同，选择不同的存储设备
>
> 配置合理的RAID级别\(raid5、raid10、热备盘\)
>
> 对与操作系统来讲，不需要太特殊的选择，最好做好冗余（raid1）（ssd、sas 、sata）
>
> _**raid卡：主机raid卡选择：**_
>
>     　　实现操作系统磁盘的冗余（raid1）
>
> 　　　平衡内存和磁盘资源
>
> 　　　随机的I/O和顺序的I/O
>
> 　　　主机 RAID卡的BBU\(Battery Backup Unit\)要关闭。

网络设备方面：

> 使用流量支持更高的网络设备（交换机、路由器、网线、网卡、HBA卡）

注意：以上这些规划应该在初始设计系统时就应该考虑好。

### 系统优化

Cpu：

> 基本不需要调整，在硬件选择方面下功夫即可。

内存：

> 基本不需要调整，在硬件选择方面下功夫即可。

SWAP：

> MySQL尽量避免使用swap。阿里云的服务器中默认swap为0

IO ：

> raid、no lvm、 ext4或xfs、ssd、IO调度策略

**Swap调整：**不使用swap分区

```
临时：/proc/sys/vm/swappiness的内容改成0
永久：/etc/sysctl.conf上添加vm.swappiness=0
```

这个参数决定了Linux是倾向于使用swap，还是倾向于释放文件系统cache。在内存紧张的情况下，数值越低越倾向于释放文件系统cache。当然，这个参数只能减少使用swap的概率，并不能避免Linux使用swap。

**修改MySQL的配置参数innodb\_flush\_method，开启O\_DIRECT模式。**

这种情况下，InnoDB的buffer pool会直接绕过文件系统cache来访问磁盘，但是redo log依旧会使用文件系统cache。

值得注意的是，Redo log是覆写模式的，即使使用了文件系统的cache，也不会占用太多

**IO调度策略：修改为deadline**

```
临时修改：#echo deadline>/sys/block/sda/queue/scheduler  
永久修改：将/boot/grub/grub.conf更改为 kernel /boot/vmlinuz-2.6.18-8.el5 ro root=LABEL=/ elevator=deadline rhgb quiet
```

### 系统参数调整

Linux系统内核参数优化

```

```

用户限制参数（mysql可以不设置以下配置）

```

```

### 应用优化

业务应用和数据库应用独立,

防火墙：iptables、selinux等其他无用服务\(关闭\)：



数据库优化

SQL优化方向：

　　执行计划、索引、SQL改写

架构优化方向：

　　高可用架构、高性能架构、分库分表

### 数据库参数优化

调整：

　　实例整体（高级优化，扩展）：

```

```

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

SQL层（基础优化）

```
查询缓存   

  OLAP类型数据库,需要重点加大此内存缓存，
                                        但是一般不会超过GB
                                        对于经常被修改的数据，缓存会立马失效。
                                        我们可以实用内存数据库（redis、memecache），替代他的功能。
```

### 1.6.2 存储引擎层（innodb基础优化参数）

```
default-storage-engine
innodb_buffer_pool_size       # 没有固定大小，50%测试值，看看情况再微调。但是尽量设置不要超过物理内存70%
innodb_file_per_table=(1,0)
innodb_flush_log_at_trx_commit=(0,1,2) # 1是最安全的，0是性能最高，2折中
binlog_sync
Innodb_flush_method=(O_DIRECT, fdatasync)
innodb_log_buffer_size        # 100M以下
innodb_log_file_size          # 100M 以下
innodb_log_files_in_group     # 5个成员以下,一般2-3个够用（iblogfile0-N）
innodb_max_dirty_pages_pct   # 达到百分之75的时候刷写 内存脏页到磁盘。
log_bin
max_binlog_cache_size         # 可以不设置
max_binlog_size               # 可以不设置
innodb_additional_mem_pool_size    #小于2G内存的机器，推荐值是20M。32G内存以上100M
```





内容来源：

