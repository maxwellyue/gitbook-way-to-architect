# MySQL中的B+Tree

通常我们所说的索引是指`B-Tree`索引，它是目前关系型数据库中查找数据最为常用和有效的索引，大多数存储引擎都支持这种索引。使用`B-Tree`这个术语，是因为MySQL在`CREATE TABLE`或其它语句中使用了这个关键字，如下：

```sql
CREATE [UNIQUE|FULLTEXT|SPATIAL] INDEX index_name
    [index_type]
    ON tbl_name (index_col_name,...)

index_type:
    USING {BTREE | HASH}
```

但实际上不同的存储引擎可能使用不同的数据结构，比如NDB使用的是`T-Tree`，MyISAM和InnoDB使用的是`B+Tree`。

## InnoDB索引实现

---

在InnoDB中，表的数据文件本身就是按B+Tree组织的一个索引结构，这棵树的叶节点的key是数据表的主键，data是与该key对应的完整的行记录。

假如有表employee如下：

| id | name | age |
| :--- | :--- | :--- |
| 15 | Bob | 24 |
| 19 | Alice | 32 |
| 20 | JIm | 10 |
| 35 | Eric | 45 |
| 49 | Tom | 14 |
| 60 | Rose | 30 |

则employee表的主键索引如下所示：

![](/assets/屏幕快照 2018-09-23 下午1.19.20.png)

可以看到叶节点包含了完整的数据记录。这种索引叫做聚集索引。因为InnoDB的数据文件本身要按主键聚集，所以InnoDB要求表必须有主键。如果没有显式指定，则MySQL会自动选择一个可以唯一标识数据记录的列作为主键，如果不存在这种列，则MySQL自动为该表生成一个隐含字段作为主键。

对于辅助索引，data域存储相应记录主键的值而不是地址。即InnoDB的所有辅助索引都引用主键作为data域。因此，辅助索引搜索需要检索两遍索引：首先检索辅助索引获得主键，然后根据主键到主索引中检索获得记录。

比如，我们为表employee的name字段设置索引，则其索引示意图如下：

![](/assets/屏幕快照 2018-09-23 下午1.19.43.png)

## MyISAM索引实现

---

虽然MyISAM存储引擎同样使用B+Tree这种索引结构，但与InnoDB不同的是，其主索引（Primary Key）和辅助索引（Secondary key）在结构上没有任何区别，**叶节点中的data域都是存放的行记录的地址**，只是主索引要求key是唯一的，而辅助索引的key可以重复。

因此，MyISAM中索引检索的算法为：首先按照B+Tree搜索算法找到指定的Key，取出其data域的值，然后以data域的值为地址，读取相应数据记录。



## 总结

---



了解不同存储引擎的索引实现方式对于正确使用和优化索引都非常有帮助，例如知道了InnoDB的索引实现后，就很容易明白为什么不建议使用过长的字段作为主键，因为所有辅助索引都引用主索引，过长的主索引会令辅助索引变得过大。再例如，用非单调的字段作为主键在InnoDB中不是个好主意，因为InnoDB数据文件本身是一颗B+Tree，非单调的主键会造成在插入新记录时数据文件为了维持B+Tree的特性而频繁的分裂调整，十分低效，而使用自增字段作为主键则是一个很好的选择。  


**内容来源**

[MyISAM索引实现](https://www.kancloud.cn/kancloud/theory-of-mysql-index/41852)    

[InnoDB索引实现](https://www.kancloud.cn/kancloud/theory-of-mysql-index/41850)

在原文基础上，改动了结构，重画了示意图。

