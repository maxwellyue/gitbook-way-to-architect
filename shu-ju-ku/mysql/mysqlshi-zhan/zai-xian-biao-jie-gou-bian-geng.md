## 在线表结构变更

---

对于MySQL而言，DDL操作（比如CREATE/DROP/ALTER等）代价是非常高的，特别是在单表上千万的情况下，加个索引或改个列类型，就有可能堵塞整个表的读写。

#### 方案1

“**新表+触发器+迁移数据+rename**”，这是业内非常成熟的扩展列的方案，在互联网公司使用广泛。

以user\(uid, name, passwd\)扩展到user\(uid, name, passwd, age, sex\)为例

过程如下：

（1）先创建一个扩充字段后的新表user\_new\(uid, name, passwd, age, sex\)

（2）在原表user上创建三个触发器，对原表user进行的所有insert/delete/update操作，都会对新表user\_new进行相同的操作

（3）分批将原表user中的数据insert到新表user\_new，直至数据迁移完成

（4）删掉触发器，把原表移走（默认是drop掉）

（5）把新表user\_new重命名（rename）成原表user

整个过程不需要锁表，可以持续对外提供服务。

注意的地方：

（1）变更过程中最重要的是冲突的处理，：以触发器的新数据为准，这就要求被迁移的表必须有主键（这个要求基本都满足）；

（2）变更过程中写操作需要建立触发器，所以如果原表已经有很多触发器，方案就不行；

（3）触发器的建立，会影响原表的性能，所以这个操作建议在流量低峰期进行；

（4）确保磁盘空间充足，因为这个过程需要复制原表以及之后新增的数据。

具体实施，请查阅：**pt-online-schema-change**，它是Percona-toolkit一员，通过改进原生ddl的方式，达到不锁表在线修改表结构。

#### 方案2

**提前预留一些reserved字段，**但如果预留过多，会造成空间浪费，预留过少，不一定达得到扩展效果。

#### 方案3

**通过增加表的方式扩展列**，上游通过service来屏蔽底层的细节。

#### **方案4**

从5.6版本开始，MySQL提供了Online DDL功能，可以实现修改表结构的同时，依然允许DML操作\(select,insert,update,delete\)。

但并不是可以随心所欲地进行DDL，有一些限制：

**不允许并发DML的情况**

修改列数据类型、删除主键、变更表字符集

**可以并发DML的情况**

* 删除并添加主键
* 添加、删除列，改变列顺序
* 改变行格式ROW\_FORMAT和压缩块大小KEY\_BLOCK\_SIZE
* 改变列NULL或NOT NULL
* 优化表OPTIMIZE TABLE
* 强制 rebuild 该表

下表是根据官方[Summary of Online Status for DDL Operations](https://dev.mysql.com/doc/refman/5.6/en/innodb-create-index-overview.html)整理挑选的常用操作。

|  | In-Place? | 拷贝表 | 允许并发DML | 并发查询 |
| :--- | :--- | :--- | :--- | :--- |
| 添加索引 | Yes\* | No\* | Yes | Yes |
| 删除索引 | Yes | No | Yes | Yes |
| 对一列设置默认值 | Yes | No | Yes | Yes |
| 对一列修改auto-increment 的值 | Yes | No | Yes | Yes |
| 改变列名 | Yes\* | No\* | Yes\* | Yes |
| 添加列 | Yes\* | Yes\* | Yes\* | Yes |
| 删除列 | Yes | Yes\* | Yes | Yes |
| 修改列数据类型 | No | Yes\* | No | Yes |
| 更改列顺序 | Yes | Yes | Yes | Yes |
| 设置列属性NULL 或NOT NULL | Yes | Yes | Yes | Yes |
| 变更表字符集 | No | Yes | No | Yes |

  
TODO

## 参考

---

[pt-online-schema-change使用说明、限制与比较](http://seanlook.com/2016/05/27/mysql-pt-online-schema-change/)

这才是真正的表扩展方案

  
  


