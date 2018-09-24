# MySQL查询性能优化

---

对于MySQL，最简单的衡量查询开销的三个指标如下：

* 响应时间

* 扫描的行数

* 返回的行数

没有哪个指标能够完美地衡量查询的开销，但它们大致反映了MySQL在内部执行查询时需要访问多少数据，并可以大概推算出查询运行的时间。 这三个指标都会记录到MySQL的慢日志中。

## 优化数据访问

---

有些查询会请求超过实际需要的数据， 然后这些多余的数据会被应用程序丢弃。这会给MySQL服务器带来额外的负担， 并增加网络开销。另外也会消耗应用服务器的CPU 和内存资源。

**错误示例**：查询不需要的记录；多表关联时返回全部列；使用SELECT \* 取出全部列； 重复查询相同的数据（可以考虑缓存热点）

## 重构查询类型

---

（1）**考虑是否需要将一个复杂的查询分成多个简单的查询**；

（2）**切分查询**：如有必要，将一个大查询切分成小查询， 每个查询功能完全一样，每次只返回一小部分查询结果；

（3）**分解关联查询**：对每一个表进行一次单表查询， 然后将结果在应用程序中进行关联，这么做基于以下考虑：

* * 让缓存的效率更高。 应用程序便于缓存单表查询结果/MySQL的查询缓存单表更小概率缓存失效；
  * 将查询分解后，执行单个查询可以减少锁的竞争；
  * 在应用层做关联， 可以更容易对数据库进行拆分， 更容易做到高性能和可扩展；
  * 主要场景：当能够使用IN\(\)的方式替代关联查询的时候；当查询中使用同一个表的时候；当应用中使用单个查询结果缓存的时候；

## 利用查询优化器

---

该部分内容见：[《MySQL查询过程》](/shu-ju-ku/mysql/mysqlcha-xun-guo-cheng.md)

## 优化特定类型的查询

---

**优化COUNT\(\)**

* 统计某个列值的数量：COUNT\(col\_name\)，统计col\_name字段的非空行数，不统计这个字段值为null的记录
* 统计行数：COUNT\(1\)或者COUNT\(\*\)，
* * COUNT\(\*\) 的通配符＊井不会扩展成所有的列，它会忽略所有的列而直接统计所有的行数。
  * 对于MyISAM，不带任何WHERE条件的COUNT\(\*\)可以很快返回结果，因为它会为表直接维护总记录数

**优化关联查询**

MySQL执行关联查询的方式是**嵌套循环关联（**Nested Loop Join**）**：即MySQL先在一个表中循环取出单条数据，然后再嵌套循环到下一个表中寻找匹配的行，依次下去，直到找到所有表中匹配的行为止。然后根据各个表匹配的行，返回查询中需要的各个列。MySQL会尝试在最后一个关联表中找到所有匹配的行，如果最后一个关联表无法找到更多的行以后， MySQL返回到上一层次关联表， 看是否能够找到更多的匹配记录， 依此类推迭代执行。

举个例子

```sql
# 查询语句
SELECT tbl1.col1, tbl2.col2
FROM tbl1 INNER JOIN tbl2 USING(col3)
WHERE tbl1.col1 IN(5, 6)

# 查询过程用伪代码表示如下
outer_iter = iterator over tbl1 where col1 IN(5, 6)                
outer_row = outer_iter.next
while outer_row
    inner_iter = iterator over tbl32 where col3 = outer_row.col3
    inner_row = inner2_iter.next
    while inner_row
        output [outer_row.col1, inner_row.clo2]
        inner_row = inner_iter.next
    end
    outer_row = outer_iter.next
end
```

根据先获取tbl1中的数据，并以tbl1为基础，循环tbl2，这里，tbl1称为驱动表，tbl2称为被驱动表；显而易见，外层循环次数越少，查询就会越快，所以在进行连接查询的时候，**应该尽可能的使用小表做为驱动表；**此外，循环体中tbl2查询时，使用了关联字段等值条件，如果tbl2中的col3字段有索引，则查询速度就会很快，所以在进行连接查询的时候，要求关联字段必须有索引，但并不是说tbl1和tbl2上的col3都需要建索引，上面例子中，关联顺序为（tab1, tbl2），那么只需要在tbl2的col3字段上创建索引即可： 一般来说，除非有其他理由， 否则**只需要在关联顺序中的第二个表的相应列上创建索引**。

如果将上面的INNER JOIN改为LEFT JOIN，执行过程仍是适用的：

```sql
# 查询语句
SELECT tbl1.col1, tbl2.col2
FROM tbl1 LEFT JOIN tbl2 USING(col3)
WHERE tbl1.col1 IN(5, 6)

# 查询过程用伪代码表示如下
outer_iter = iterator over tbl1 where col1 IN(5, 6)                
outer_row = outer_iter.next
while outer_row
    inner_iter = iterator over tbl32 where col3 = outer_row.col3
    inner_row = inner2_iter.next
    if inner_row
        while inner_row
            output [outer_row.col1, inner_row.clo2]
            inner_row = inner_iter.next
        end
    else
        output [outer_row.col1, NULL]
    outer_row = outer_iter.next
end
```

理解了这两个执行过程，也能对INNER JOIN和OUTER JOIN的区别有更深一层的认识。

对应RIGHT JOIN，MySQL会将其改写为等价的LEFT JOIN。

此外，优化关联查询还有重要的一条：确保GROUP BY和ORDER BY中的表达式只涉及到第一个表中的列， 这样MySQL才有可能使用索引来优化这个过程。下面会详细介绍。

**优化排序**

MySQL的排序方式有两种：索引排序和文件排序。

MySQL会在以下情况使用**索引排序**：

* * ORDER BY col，且col上有单列索引
  * ORDER BY col1,  col2,  col3 ，且col1, col2, col3上有多列索引，且顺序为col1, col2, col3；
  * 对于关联查询还需要有一个额外的条件：ORDER BY后面的字段必须全部为关联查询中第一个表（驱动表）的字段。

当MySQL不能使用索引排序时，就会进行**文件排序（filesort）**:

* * 数据量小，直接在内存中进行排序（但也叫filesort）
  * 数据量大，则需要借助磁盘：先将数据分块，对每个独立的块进行排序，并将各个块的排序结果放在磁盘上，最后将各个排好序的块进行合并，返回排序结果

对于关联查询的文件排序，有以下规则：

* * 如果ORDER BY后面所有的字段都来自第一个表（驱动表），则在关联处理第一个表的时候就进行文件排序（此时，EXPLAIN的Extra字段会有"Using filesort"信息）
  * 除此之外的所有情况，MySQL都会先将关联的结果存放到一个临时表中，然后在所有的关联都结束后，再进行文件排序。（此时，EXPLAIN结果的Extra字段可以看到"Using temporaya, Using filesort"信息）

对于ORDER BY和LIMIT同时出现的查询：

* * MySQL5.6版本之前：LIMIT会在排序之后应用，所以即使需要返回较少的数据，临时表和需要排序的数据量仍然会非常大。
  * MySQL5.6及更新版本：会根据实际情况，选择抛弃不满足条件的结果，然后再进行排序。

**优化子查询**

* 尽可能使用关联查询代替子查询，但是在MySQL5.6或更新的版本或者MariaDB，可以忽略该建议。

**优化GROUP BY和DISTINCT**

* 当无法使用索引的时候， GROUP BY使用两种策略来完成：使用临时表或者文件排序来做分组。

* 如果需要对关联查询做GROUP BY，并且是按照查找表中的某个列进行分组， 那么通常采用查找表的标识列分组的效率会比其他列更高。

**优化LIMIT分页**

* 最简单的办法就是尽可能地使用索引覆盖扫描， 而不是查询所有的列。 然后根据需要做一次关联操作再返回所需的列。 对于偏移量很大的时候， 这样做的效率会提升非常大。

**优化UNION查询**

* 尽量使用UNION ALL，而不是UNION，即使有去重需求，如果数据量不大，也将去重工作交给程序，而非MySQL
* * 对于UNION查询，MySQL先将一系列的单个查询结果放到一个临时表中，然后再重新读出临时表数据来完成UNION查询。在MySQL的概念中，每个查询都是一次关联，所以读取结果临时表也是一次关联。
* * 如果没有ALL关键字，MySQL会给临时表加上DISTINCT选项，会导致对整个临时表的数据做唯一性检查，代价非常高。

---

内容来源：《高性能MySQL》：查询性能优化

