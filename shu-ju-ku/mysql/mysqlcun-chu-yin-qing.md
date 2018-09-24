# MySQL存储引擎

---

存储引擎定义了什么？

MySQL中的数据用各种不同的技术存储在文件（或者内存）中。这些技术中的每一种技术都使用不同的存储机制、索引技巧、锁定水平并且最终提供广泛的不同的功能和能力。通过选择不同的技术，你能够获得额外的速度或者功能，从而改善你的应用的整体功能。这些不同的技术以及配套的相关功能在MySQL中被称作存储引擎\(也称作表类型\)。

你可以选择适用于服务器、数据库和表格的存储引擎，以便在选择如何存储你的信息、如何检索这些信息以及你需要你的数据结合什么性能和功能的时候为你提供最大的灵活性。

选择如何存储和检索你的数据的这种灵活性是MySQL为什么如此受欢迎的主要原因。其它数据库系统\(包括大多数商业选择\)仅支持一种类型的数据存储。遗憾的是，其它类型的数据库解决方案采取的“一个尺码满足一切需求”的方式意味着你要么就牺牲一些性能，要么你就用几个小时甚至几天的时间详细调整你的数据库。使用MySQL，我们仅需要修改我们使用的存储引擎就可以了。

可以这么来理解：MySQL是数据仓库，那它是怎么存的数据，采用的什么存储机制、用的哪种索引等等，这样就是存储引擎来决定的。它规定了一个表的类型，也就是存储引擎所描述的数据单元是表。

## 存储引擎分类

---

MySQL在版本5.5以后，默认存储引擎是InnoDB，之前版本的默认存储引擎是MyISAM。所以这两种应该是最为常用的，也是最应该去了解的。

```sql
mysql> select version();
+-----------+
| version() |
+-----------+
| 5.7.18    |
+-----------+
1 row in set

mysql> show engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set
```
由于最常用的就是InnoDB，主要来看InnoDB。从上面可以看出MySQL对InnoDB的描述：支持事务、行级锁以及外键，这些都是MyISAM不支持的。

InnoDB的特性总结（Key Advantages of InnoDB）：

| 属性 | 属性值 | 属性 | 属性值 | 属性 | 属性值 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 存储限制 | 64TB | 事务 | Yes | 锁定粒度 | Row |
| 多版本并发控制 | Yes | Geospatial数据类型支持 | Yes | Geospatial索引支持 | Yes\[a\] |
| B-tree索引 | Yes | T-tree索引 | No | Hash索引 | No\[b\] |
| Full-text search索引 | Yes\[c\] | Clustered索引 | Yes | 数据缓存 | Yes |
| 索引缓存 | Yes | 数据压缩 | Yes\[d\] | 数据加密\[e\] | Yes |
| 集群数据库支持 | No | 主从复制支持\[f\] | Yes | 外键支持 | Yes |
| 备份/时间点恢复\[g\] | Yes | 查询缓存支持 | Yes | 数据字典的统计信息更新 | Yes |

## 各种引擎的特性
---

特性 | MyISAM | Memory | InnoDB | Archive | NDB
--|--|--|--|--|--|--
存储限制 | 256TB | RAM | 64TB | None | 384EB
事务 | No | No | Yes | No | Yes
锁粒度 | Table | Table | Row | Row | Row
多版本并发控制| No | No | Yes | No | No
Geospatial数据类型支持 | Yes | No | Yes | Yes | Yes
Geospatial 索引支持 | Yes | No | Yes[a] | No | No
B-tree 索引| Yes | Yes | Yes | No | No
T-tree 索引| No | No | No | No | Yes
Hash 索引| No | Yes | No[b] | No | Yes
Full-text search 索引 | Yes | No | Yes[c] | No | No
Clustered 索引| No | No | Yes | No | No
数据缓存 | No | N/A | Yes | No | Yes
索引缓存 | Yes | N/A | Yes | No | Yes
数据压缩| Yes[d] | No | Yes[e] | Yes | No
数据加密[f] | Yes | Yes | Yes | Yes | Yes
集群数据库支持 | No | No | No | No | Yes
主从复制支持[g] | Yes | Yes | Yes | Yes | Yes
外键支持 | No | No | Yes | No | Yes[h]
备份/时间点恢复[i] | Yes | Yes | Yes | Yes | Yes
查询缓存支持 | Yes | Yes | Yes | Yes | Yes
数据字典的统计信息更新 | Yes | Yes | Yes | Yes | Yes


**InnoDB和MyISAM的区别
**

* MyISAM是mysql 5.5.5 之前的默认引擎，支持 B-tree/FullText/R-tree 索引类型。锁级别为表锁，表锁优点是开销小，加锁快；缺点是锁粒度大，发生锁冲动概率较高，容纳并发能力低，这个引擎适合查询为主的业务。此引擎不支持事务，也不支持外键。MyISAM强调了快速读取操作。它存储表的行数，于是SELECT COUNT() FROM TABLE时只需要直接读取已经保存好的值而不需要进行全表扫描。

* InnoDB 存储引擎最大的亮点就是支持事务，支持回滚，它支持 Hash/B-tree 索引类型。
锁级别为行锁，行锁优点是适用于高并发的频繁表修改，高并发是性能优于 MyISAM。缺点是系统消耗较大，索引不仅缓存自身，也缓存数据，相比 MyISAM 需要更大的内存。
InnoDB中不保存表的具体行数，也就是说，执行 select count() from table时，InnoDB要扫描一遍整个表来计算有多少行。支持事务，支持外键


##  参考
---
[百度百科——存储引擎](https://link.jianshu.com/?t=http://baike.baidu.com/item/%E5%AD%98%E5%82%A8%E5%BC%95%E6%93%8E)

[官方文档(5.7版本) InnoDB的优势](https://link.jianshu.com/?t=https://dev.mysql.com/doc/refman/5.7/en/innodb-introduction.html)

[官方文档(5.7版本) 各种存储引擎的特性](https://dev.mysql.com/doc/refman/5.7/en/storage-engines.html)



