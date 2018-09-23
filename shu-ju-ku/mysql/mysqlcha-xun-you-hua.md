# 高性能索引策略

**1. 如果某列建立了索引，查询时，请将其作为单独的列，不要将其放在表达式或函数中，否则不会使用索引。**

```sql
//错误示例
SELECT actor_id FROM sakila.actor WHERE actod_id + 1 = 5;
//正确写法
SELECT actor_id FROM sakila.actor WHERE actod_id = 4;

//错误示例
SELECT col_name FROM tbl_name WHERE TO_DAYS(CURRENT_DATE) - TO_DAYS(data_col) <= 10;
//正确写法
SELECT col_name FROM tbl_name WHERE data_col >= DATE_SUB(CURRENT_DATE, INTERVAL 10 DAY);
```

**2、如果要索引很长的字符串，请选择合适的前缀长度。**

如何计算这个合适的前缀长度？假设table为city\_demo，要对字段city建立索引。

有两种方法：

（1）通过观察前缀的选择性

* 第一步：找到该字段中出现次数最多的前N个值的列表，并统计其出现次数；
* 第二步：尝试设置不同的前缀长度，观察该前缀的列表，直到这个前缀的选择性接近完整列的选择性

（2）通过直接计算前缀的选择性

* 第一步：首先计算完整列的选择性
* 第二步：尝试不同的前缀长度，计算其选择性，与完整列选择性最先比较接近的长度，即为最合适的前缀长度

```sql
//计算完整列的选择性
SELECT COUNT(DISTINCT city)/COUNT(*) FROM city_demo

//尝试3/4/5/6/7这四个前缀长度，计算其选择性
SELECT  COUNT(DISTINCT LEFT(city, 3))/COUNT(*) AS sel3,
        COUNT(DISTINCT LEFT(city, 4))/COUNT(*) AS sel4,
        COUNT(DISTINCT LEFT(city, 5))/COUNT(*) AS sel5,
        COUNT(DISTINCT LEFT(city, 6))/COUNT(*) AS sel6,
        COUNT(DISTINCT LEFT(city, 7))/COUNT(*) AS sel7,
FROM city_demo
```

前缀索引的缺点：无法使用前缀索引做GROUP BY 和 ORFER BY ，也无法使用前缀索引做覆盖扫描。

TIPS：如果某个字段的前缀没有明显的选择性，但是后缀的选择性却很好，可以考虑将字符串反转后存储。

**3、在所有的列上单独创建索引，大部分情况下并不会提高查询性能，还会带来维护索引的损耗**。MySQL 5.0 及更新版本虽然提供了“索引合并”的功能，但如果发现索引合并（EXPLAIN时Extra字段出现`Using union`），请检查表的结构和查询语句是不是最优的。

索引合并：在查询时能够同时使用两个单列索引进行扫描，并将结果合并，这两个单列索引字段在WHERE中一般为OR/AND关系。

```sql
# 如果actor_id和film_id都建立了单列索引
SELECT film_id, actor_id FROM film_actor WHERE actor_id = 1 OR film_id = 1;

# 则实际会转换为以下语句来查询
SELECT film_id,actor_id FROM film_actor WHERE actor_id = 1  
UNION ALL 
SELECT film_id,actor_id FROM film_actor WHERE film_id = 1 AND actor_id <> 1
```

**4、创建多列索引时，将选择性最高的列放在索引最前列，但不要根据该索引进行排序和分组 **

TODO

**5、聚簇索引的使用**

对于InnoDB表，主键索引为聚簇索引（或叫聚集索引），应选择AUTO\_INCREMENT的自增列作为主键，并按主键顺序插入数据，这样可以保证数据行是按照顺序写入，如果使用UUID聚簇索引，因为新的行的主键值不一定比之前插入的大，所以InnoDB 无法简单的总是把新行插入到索引的最后，而是需要为新的行寻找到合适的位置--通常是已有数据的中间位置--并且分配空间，这会增加很多的额外操作。

**6、覆盖索引的使用**

如果一个索引包含了所要查询的字段的值，该索引就叫覆盖索引（该索引覆盖了字段的值）。使用覆盖索引查询，只需要查询索引就可以找到要查询的值，无需回表。EXPLAIN的结果中国extra列会出现using index就说明使用的是覆盖索引。

一般情况下，是通过建立多列索引来达到覆盖索引的效果，或者查询字段只有一个字段，而该字段上建立了单列索引，也能达到覆盖索引的效果；反之，查询的字段中含有不在索引中的字段 ，或者WHERE条件中含有对索引列的LIKE操作，都不能达到覆盖索引的效果。

**7、在设计索引时，如果一个索引既能够满足排序，又满足查询，是最好的。**

MySQL有两种方式可以生产有序的结果集：

* * 对结果集进行排序的操作，
  * 按照索引顺序扫描得出的结果自然是有序的；

如果EXPLAIN的结果中`type`列的值为`index`表示使用了索引扫描来做排序。

按照索引顺序读取数据的速度通常要比顺序地全表扫描要慢：扫描索引本身很快，因为只需要从一条索引记录移动到相邻的下一条记录。但如果索引本身不能覆盖所有需要查询的列，那么就不得不每扫描一条索引记录就回表查询一次对应的行。这个读取操作基本上是随机I/O。

只有当索引的列顺序和`ORDER BY`子句的顺序完全一致，并且所有列的排序方向也一样时，才能够使用索引来对结果做排序。如果查询需要关联多张表，则只有`ORDER BY`子句引用的字段全部为第一张表时，才能使用索引做排序。`ORDER BY`子句和查询的限制是一样的，都要满足最左前缀的要求（有一种情况例外，就是最左的列被指定为常数），其他情况下都需要执行排序操作，而无法利用索引排序。

**8、避免出现重复索引**

重复索引是指在相同的列上按照相同的顺序创建的相同类型的索引。如，已经有索引`(A,B)`，再创建索引`(A)`就是重复索引。

**9、考虑删除未使用的索引**：通过INFORMATION_SCHEMA.INDEX\__STATISTICS命令查询每个索引的使用频率。

**10、避免多个范围条件**：如有可能，可以使用多个等值条件来替代某个范围条件。





  


  


  




