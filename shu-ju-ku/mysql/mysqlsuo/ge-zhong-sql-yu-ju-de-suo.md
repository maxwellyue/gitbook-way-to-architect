## SQL中的锁

---

**一、普通select**

\(1\)在读未提交\(Read Uncommitted\)，读提交\(Read Committed, RC\)，可重复读\(Repeated Read, RR\)这三种事务隔离级别下，普通select使用快照读\(snpashot read\)，不加锁，并发非常高；

\(2\)在串行化\(Serializable\)这种事务的隔离级别下，普通select会升级为select ... in share mode;

**二、加锁select**

加锁select主要是指：

* select ... for update

* select ... in share mode

\(1\)如果在唯一索引\(unique index\)上使用唯一的查询条件\(unique search condition\)，会使用行锁\(record lock\)，而不会封锁记录之间的间隔，即不会使用间隙锁\(gap lock\)与临键锁\(next-key lock\)；

\(2\)其他的查询条件和索引条件，InnoDB会封锁被扫描的索引范围，并使用间隙锁与临键锁，避免索引范围区间插入记录；

举个例子，假设有InnoDB表：t\(id PK, name\);

表中有三条记录：

1, wangwu

2, zhangsan

3, lisi

SQL语句`select * from t where id=1 for update;`只会封锁记录，而不会封锁区间。

**三、update与delete**

\(1\)和加锁select类似，如果在唯一索引上使用唯一的查询条件来update/delete。

例如`update t set name=xxx where id=1;`只加行锁；

\(2\)否则，**符合查询条件**的索引记录之前，都会加排他临键锁\(exclusive next-key lock\)，来封锁索引记录与之前的区间；

\(3\)尤其需要特殊说明的是，如果update的是聚集索引\(clustered index\)记录，则对应的普通索引\(secondary index\)记录也会被隐式加锁，这是由InnoDB索引的实现机制决定的：普通索引存储PK的值，检索普通索引本质上要二次扫描聚集索引。

**四、insert**

同样是写操作，insert和update与delete不同，它会用排它锁封锁被插入的索引记录，而不会封锁记录之前的范围。

同时，会在插入区间加插入意向锁\(insert intention lock\)，但这个并不会真正封锁区间，也不会阻止相同区间的不同KEY插入。

## **案例分析**

---

### **case1**

MySQL，InnoDB，默认的隔离级别\(RR\)，假设有数据表：t\(id PrimaryKey, name\);

数据表中有数据：  
10, wangwu  
20, zhangsan  
30, lisi

事务A先执行，还未提交：`insert into t values(11, xxx);`

事务B后执行：`insert into t values(12, ooo);`

问：事务B会不会被阻塞？

答：虽然事务隔离级别是RR，虽然是同一个索引，虽然是同一个区间，但插入的记录并不冲突，所以：**使用的是插入意向锁，并不会阻塞事务B。**

### **Case 2**

MySQL，InnoDB，默认的隔离级别\(RR\)，假设有数据表：t\(id AUTO\_INCREMENT, name\);

数据表中有数据：  
1, wangwu  
2, zhangsan  
3, lisi

事务A先执行，还未提交：  
insert into t\(name\) values\(xxx\);

事务B后执行：  
insert into t\(name\) values\(ooo\);

问：事务B会不会被阻塞？

答：自增锁是一种特殊的表级别锁（table-level lock），专门针对事务插入AUTO\_INCREMENT类型的列。最简单的情况，如果一个事务正在往表中插入记录，所有其他事务的插入必须等待，以便第一个事务插入的行，是连续的主键值。所以：**使用的是自增锁，会阻塞事务B。**





