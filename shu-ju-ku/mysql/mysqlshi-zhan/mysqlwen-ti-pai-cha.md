# MySQL问题排查

---

**硬件**

问题一：cpu负载高，IO负载低

内存不够；磁盘性能差；SQL问题；IO出问题了（磁盘到临界了、raid设计不好、raid降级、锁、在单位时间内tps过高）；tps过高，大量的小数据IO；大量的全表扫描

问题二：IO负载高，cpu负载低

大量小的IO写操作：autocommit，产生大量小IO；IO/PS：磁盘的一个定值，硬件出厂的时候，厂家定义的一个每秒最大的IO次数。

大量大的IO写操作：SQL问题的几率比较大

问题三：IO和cpu负载都很高

硬件不够了或sql存在问题

**SQL**

针对突然的业务办理卡顿，无法进行正常的业务处理，需要立马解决的场景。

1、show processlist：查询每个MySQL连接（或线程）的当前状态

2、explain  sql，通过执行计划判断，索引问题（有没有、合不合理）或者语句本身问题

3、show status  like '%lock%';    \# 查询锁状态

4、kill SESSION\_ID;   \# 杀掉有问题的session

针对业务周期性的卡顿，例如在每天10-11点业务特别慢，但是还能够使用，过了这段时间就好了。

1、查看slowlog，分析slowlog，分析出查询慢的语句。

2、按照一定优先级，进行一个一个的排查所有慢语句。

3、分析优先级最高的sql，进行explain调试，查看语句执行时间。

4、调整索引或语句本身。

**工具**

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

...

---

内容来源：[MySQL 优化实施方案](#)

